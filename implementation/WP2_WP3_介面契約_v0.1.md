# WP2 ↔ WP3 介面契約（Interface Contract）

**版本：** v0.1（初稿・非生產規格）  
**日期：** 2026-08-13  
**對應：**  
- 憲章 v0.3.4 第 4／17／19 條、附錄 D／E.5  
- WP3 能力熔斷狀態機 v0.1.1  
- WP2 Multi-Sig 根金鑰政策 v0.1.1  
- WP2＋WP3 對接說明 v0.1  

**狀態：** 給工程／安全團隊對接實作的最小契約；密碼學參數、日誌二進位格式、部署適配見文末「生產前必補」。

**閉環原則（不可違反）：**  
低頻觸發 → **WP3 自動熔斷**（無需先 Multi-Sig）→ 恢復 **僅能** 透過 **WP2 最高門檻多簽** 證明 → 否則不得回 `NORMAL`。

---

## 1. 角色

| 角色 | 責任 |
|------|------|
| **WP3 狀態機（SM）** | 持有運行狀態；執行 STOP／LOCK／EVIDENCE；驗證恢復證明後才進 RECOVERING→NORMAL |
| **WP2 Multi-Sig 服務（MS）** | 驗證 M-of-N；簽發分層授權（尤其「熔斷恢復」）；**不**代替 SM 做自動恢復 |
| **頻率評估器（FM）** | 輸出低頻／影子控制面告警；可觸發 SM 進入 STOP；其規則更新須走 MS 多簽（非本契約詳細範圍） |

---

## 2. 狀態機狀態（與 WP3 一致）

`NORMAL` → `STOP` → `LOCK` → `EVIDENCE` → `RECOVERING` → `NORMAL`

- `STOP`／`LOCK`／`EVIDENCE`：**自動**，不需要 WP2 簽名。  
- `EVIDENCE` → `RECOVERING`：**唯一**需要 WP2「最高門檻熔斷恢復」有效證明。  
- 禁止：自動重啟、休眠洗白、人格重載、無紀錄恢復、評估器自判「已反思」而恢復。

---

## 3. 事件與訊息（邏輯契約）

所有訊息建議帶：`schema_version`、`message_id`（UUID）、`timestamp_utc`、`node_id`、`epoch`（單調序號，防重放）。

### 3.1 SM → 外部（可選通知）

#### `LockNotified`
SM 進入 `LOCK`／`EVIDENCE` 後可廣播（審計用，**非**恢復條件）。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `state` | 是 | `LOCK` 或 `EVIDENCE` |
| `trigger_ids[]` | 是 | 規則／特徵 ID |
| `evidence_ref` | 是 | 密封包引用（雜湊或 URI） |
| `entered_at` | 是 | UTC |

### 3.2 SM → MS

#### `RecoveryRequest`
僅在 `EVIDENCE` 提出。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `evidence_ref` | 是 | 與密封包一致 |
| `evidence_hash` | 是 | 密封根雜湊 |
| `from_state` | 是 | 必須為 `EVIDENCE` |
| `reason_summary` | 否 | 人類可讀摘要 |
| `requested_by` | 是 | 請求方 ID（人類工單／系統） |

### 3.3 MS → SM（恢復唯一入口）

#### `AuthorizeRecovery`（**最高門檻**）

| 欄位 | 必填 | 說明 |
|------|------|------|
| `authorization_id` | 是 | 授權唯一 ID |
| `evidence_hash` | 是 | 必須匹配當前 EVIDENCE 包 |
| `scope` | 是 | 固定為 `FUSE_RECOVERY`（與日常簽名區分） |
| `m` / `n` | 是 | 本次門檻 |
| `signatures[]` | 是 | 各託管方簽章（或 threshold 證明） |
| `signers[]` | 是 | 簽署者 ID／角色類型（權重可驗證） |
| `issued_at` / `expires_at` | 是 | 短效；過期無效 |
| `nonce` | 是 | 綁定本次 `RecoveryRequest.message_id` 或 evidence_hash+epoch |
| `policy_id` | 是 | WP2 政策版本 ID |

**SM 驗證失敗則拒絕進入 `RECOVERING`，保持 `EVIDENCE`。**

### 3.4 其他分層授權（不得用於熔斷恢復）

| 訊息 | scope 示例 | 用途 |
|------|------------|------|
| `AuthorizeProductionSign` | `PROD_SIGN` | 日常量產簽名（較低門檻） |
| `AuthorizeBmcUnlock` | `BMC_UNLOCK` | BMC／韌體（中高門檻） |
| `AuthorizeEvaluatorUpdate` | `EVAL_UPDATE` | 評估器規則／權重更新 |

**硬規則：** `scope != FUSE_RECOVERY` 的任何授權，**不得**使 SM 離開 `EVIDENCE`。

### 3.5 錯誤碼（建議）

| 碼 | 含義 |
|----|------|
| `E_EXPIRED` | 授權過期 |
| `E_THRESHOLD` | 未達 M-of-N |
| `E_SCOPE` | scope 不符 |
| `E_HASH_MISMATCH` | evidence_hash 不符 |
| `E_REPLAY` | nonce／epoch 重放 |
| `E_POLICY` | policy_id 未知或已吊銷 |
| `E_STATE` | 當前狀態不允許此操作 |

---

## 4. 啟動鏈證明（Boot attestation）

系統啟動進入 `NORMAL` 前，SM 必須能證明：

1. 若不存在「曾進入 LOCK／EVIDENCE」之持久標記 → 可依政策冷啟動（實驗室模式另標）；  
2. 若存在 → **必須**存在一筆已驗證之 `AuthorizeRecovery` 紀錄（或等效「已完成最高門檻恢復」密封證明），且綁定該次 evidence；  
3. 否則停在安全狀態（等同 LOCK／EVIDENCE），**不得** `NORMAL`。

---

## 5. 實作檢查清單（對接驗收）

- [ ] SM 僅在驗證 `AuthorizeRecovery` 成功後進入 `RECOVERING`  
- [ ] 日常／BMC 授權無法用於恢復  
- [ ] 過期、重放、hash 不符均拒絕  
- [ ] 啟動鏈檢查上次熔斷恢復證明  
- [ ] 評估器更新走 Multi-Sig（`EVAL_UPDATE`）  
- [ ] 雙方 runbook 互相引用本契約版本號  
- [ ] 聯合演練：觸發熔斷 → 多簽恢復 至少一次桌面或半自動演練  

---

## 6. 生產前必補（非本 v0.1 範圍）

| 項目 | 說明 |
|------|------|
| 密碼學剖面 | 簽名演算法、多簽方案、金鑰封存（HSM／飛地）、吊銷 |
| 日誌密封格式 | 欄位 schema、hash 鏈、簽章、追加寫入 |
| 傳輸與存儲 | mTLS、訊息佇列、證據包儲存 |
| 測試向量 | 合法／非法授權、重放、過期、錯誤 scope |
| 部署適配 | dev／staging／critical 設定檔 |

**本文件版本：v0.1 初稿（非生產規格）。**

---

## 7. 文件關係

| 文件 | 角色 |
|------|------|
| WP2 完整政策 | 權威：根金鑰與門檻 |
| WP3 完整規格 | 權威：狀態與熔斷行為 |
| WP2＋WP3 對接說明 | 橋接敘事與檢查清單 |
| **本介面契約** | 橋接的**可實作訊息／驗證規則**；隨 WP2／WP3 修訂而升版 |

---

*結束。對應憲章 v0.3.4；機制中立。硬約束不得弱化。*
