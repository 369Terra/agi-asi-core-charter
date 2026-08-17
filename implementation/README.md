# Implementation starter pack（初稿・非生產規格）

對應憲章 v0.3.4　第 4／17／19 條、附錄 D／E.5

This folder is a **first-draft starter pack**, not production spec and not formal law.  
It is for teams who want to refine and implement the Charter.  
Do not weaken the hard constraints. See the Charter Hand-off note.

歡迎在不弱化硬約束的前提下繼續細化。

一般讀者看憲章兩份 PDF 即可。這裡是給要動手的人。

## Read in this order

1. WP3 capability-fuse state machine — STOP → LOCK → EVIDENCE → RECOVERING  
   - [中文 PDF](./WP3_capability_fuse_statemachine_v0.1.1.pdf) · [English PDF](./WP3_capability_fuse_statemachine_v0.1.1_EN.pdf)  
   - [中文 MD](./WP3_capability_fuse_statemachine_v0.1.1.md) · [English MD](./WP3_capability_fuse_statemachine_v0.1.1_EN.md)
2. WP2 Multi-Sig root-key policy — thresholds and cold-storage ceremony  
   - [中文 PDF](./WP2_Multi-Sig根金鑰政策_v0.1.1.pdf) · [English PDF](./WP2_Multi-Sig_root_key_policy_v0.1.1_EN.pdf)  
   - [中文 MD](./WP2_Multi-Sig根金鑰政策_v0.1.1.md) · [English MD](./WP2_Multi-Sig_root_key_policy_v0.1.1_EN.md)
3. WP2–WP3 integration — how they meet  
   - [中文 PDF](./WP2_WP3_integration_v0.1.pdf) · [English PDF](./WP2_WP3_integration_v0.1_EN.pdf)  
   - [中文 MD](./WP2_WP3_integration_v0.1.md) · [English MD](./WP2_WP3_integration_v0.1_EN.md)
4. Minimum interface contract  
   - [中文 MD](./WP2_WP3_介面契約_v0.1.md) · [English MD](./WP2_WP3_interface_contract_v0.1.md)

中文與英文是對等分冊，不是同一頁上英下中。術語（`NORMAL`、`LOCK`、`M-of-N`、`BMC`）兩冊相同。

## Closed loop (must not break)

Low-frequency trigger → WP3 fuses automatically (no Multi-Sig required first).  
Recovery → only via WP2 highest-threshold multi-sig.  
Otherwise the system must not return to `NORMAL`.

## Root custody (WP2)

The root seed / root CA must be generated and held in an **offline cold-storage** ceremony (or equivalent high-assurance offline procedure).  
A single cloud account or a single software **hot wallet** is forbidden as root.  
Cold is the requirement. Hot is the thing that is banned.

根種子／根 CA 必須走**線下冷儲存**儀式。單雲／單軟體**熱錢包**不得為根。冷是要求，熱是禁令。
