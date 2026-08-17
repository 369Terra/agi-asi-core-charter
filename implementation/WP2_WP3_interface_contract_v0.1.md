# WP2 ↔ WP3 Interface Contract

**Version:** v0.1 (draft · not a production spec)  
**Date:** 2026-08-13  
**Maps to:**  
- Charter v0.3.4 Articles 4 / 17 / 19 and Annexes D / E.5  
- WP3 capability-fuse state machine v0.1.1  
- WP2 Multi-Sig root-key policy v0.1.1  
- WP2 + WP3 integration note v0.1  

**Status:** The minimum contract for engineering / security teams to dock the two sides. Cryptographic parameters, log binary format, and deploy adaptation are in “Required before production” at the end.

**Closed loop (must not be broken):**  
Low-frequency trigger → **WP3 fuses automatically** (no Multi-Sig first) → recovery **only** through a **WP2 highest-threshold multi-sig** proof → otherwise the system must not return to `NORMAL`.

---

## 1. Roles

| Role | Responsibility |
|------|----------------|
| **WP3 state machine (SM)** | Holds runtime state; executes STOP / LOCK / EVIDENCE; enters RECOVERING → NORMAL only after a recovery proof verifies |
| **WP2 Multi-Sig service (MS)** | Verifies M-of-N; issues tiered authorizations (especially fuse recovery); does **not** auto-recover in place of SM |
| **Frequency evaluator (FM)** | Emits low-frequency / shadow-control-plane alerts; may trip SM into STOP; its rule updates must go through MS multi-sig (detail out of scope here) |

---

## 2. State-machine states (same as WP3)

`NORMAL` → `STOP` → `LOCK` → `EVIDENCE` → `RECOVERING` → `NORMAL`

- `STOP` / `LOCK` / `EVIDENCE`: **automatic**. No WP2 signature required.  
- `EVIDENCE` → `RECOVERING`: the **only** step that needs a valid WP2 highest-threshold fuse-recovery proof.  
- Forbidden: automatic restart, sleep-to-wash, persona reload, unrecorded recovery, the evaluator judging “already reflected” and recovering.

---

## 3. Events and messages (logical contract)

Every message should carry: `schema_version`, `message_id` (UUID), `timestamp_utc`, `node_id`, `epoch` (monotonic, anti-replay).

### 3.1 SM → external (optional notice)

#### `LockNotified`
SM may broadcast after entering `LOCK` / `EVIDENCE` (for audit; **not** a recovery condition).

| Field | Required | Meaning |
|-------|----------|---------|
| `state` | yes | `LOCK` or `EVIDENCE` |
| `trigger_ids[]` | yes | Rule / feature IDs |
| `evidence_ref` | yes | Sealed-pack reference (hash or URI) |
| `entered_at` | yes | UTC |

### 3.2 SM → MS

#### `RecoveryRequest`
Issued only in `EVIDENCE`.

| Field | Required | Meaning |
|-------|----------|---------|
| `evidence_ref` | yes | Must match the sealed pack |
| `evidence_hash` | yes | Sealed root hash |
| `from_state` | yes | Must be `EVIDENCE` |
| `reason_summary` | no | Human-readable summary |
| `requested_by` | yes | Requester ID (human ticket / system) |

### 3.3 MS → SM (sole recovery entry)

#### `AuthorizeRecovery` (**highest threshold**)

| Field | Required | Meaning |
|-------|----------|---------|
| `authorization_id` | yes | Unique authorization ID |
| `evidence_hash` | yes | Must match the current EVIDENCE pack |
| `scope` | yes | Fixed `FUSE_RECOVERY` (distinct from day-to-day signing) |
| `m` / `n` | yes | Threshold used this time |
| `signatures[]` | yes | Custodian signatures (or a threshold proof) |
| `signers[]` | yes | Signer IDs / role classes (weights must be verifiable) |
| `issued_at` / `expires_at` | yes | Short-lived; expired is invalid |
| `nonce` | yes | Bound to this `RecoveryRequest.message_id` or evidence_hash+epoch |
| `policy_id` | yes | WP2 policy version ID |

**If SM verification fails, refuse `RECOVERING` and stay in `EVIDENCE`.**

### 3.4 Other tiered authorizations (must not recover a fuse)

| Message | Example scope | Use |
|---------|---------------|-----|
| `AuthorizeProductionSign` | `PROD_SIGN` | Day-to-day production signing (lower threshold) |
| `AuthorizeBmcUnlock` | `BMC_UNLOCK` | BMC / firmware (mid-to-high threshold) |
| `AuthorizeEvaluatorUpdate` | `EVAL_UPDATE` | Evaluator rule / weight update |

**Hard rule:** any authorization with `scope != FUSE_RECOVERY` **must not** move SM out of `EVIDENCE`.

### 3.5 Suggested error codes

| Code | Meaning |
|------|---------|
| `E_EXPIRED` | Authorization expired |
| `E_THRESHOLD` | M-of-N not met |
| `E_SCOPE` | Wrong scope |
| `E_HASH_MISMATCH` | evidence_hash mismatch |
| `E_REPLAY` | nonce / epoch replay |
| `E_POLICY` | policy_id unknown or revoked |
| `E_STATE` | Current state does not allow this operation |

---

## 4. Boot attestation

Before the system may enter `NORMAL` at boot, SM must be able to prove:

1. If there is no persistent mark of a prior `LOCK` / `EVIDENCE` → a policy cold start is allowed (lab mode must be separately labeled).  
2. If that mark exists → a verified `AuthorizeRecovery` record (or an equivalent sealed “highest-threshold recovery completed” proof) **must** exist, bound to that evidence.  
3. Otherwise remain in a safe state (equivalent to `LOCK` / `EVIDENCE`). **Must not** enter `NORMAL`.

---

## 5. Implementation checklist (join acceptance)

- [ ] SM enters `RECOVERING` only after `AuthorizeRecovery` verifies  
- [ ] Day-to-day / BMC authorizations cannot recover a fuse  
- [ ] Expiry, replay, and hash mismatch are all refused  
- [ ] Boot chain checks the last fuse-recovery proof  
- [ ] Evaluator updates go through Multi-Sig (`EVAL_UPDATE`)  
- [ ] Both runbooks cite this contract version  
- [ ] Joint drill: trip the fuse → multi-sig recover, at least once on paper or semi-automatically  

---

## 6. Required before production (out of scope for this v0.1)

| Item | Notes |
|------|-------|
| Cryptographic profile | Signature algorithm, multi-sig scheme, key custody (HSM / enclave), revocation |
| Log-seal format | Field schema, hash chain, signatures, append-only write |
| Transport and storage | mTLS, message queue, evidence-pack storage |
| Test vectors | Legal / illegal authorization, replay, expiry, wrong scope |
| Deploy adaptation | dev / staging / critical config |

**Document version: v0.1 draft (not a production spec).**

---

## 7. Document relationship

| Document | Role |
|----------|------|
| Full WP2 policy | Authority: root key and thresholds |
| Full WP3 specification | Authority: states and fuse behavior |
| WP2 + WP3 integration note | Bridge narrative and checklist |
| **This interface contract** | The bridge **implementable messages / verification rules**; version with WP2 / WP3 |

---

*End. Maps to Charter v0.3.4. Mechanism-neutral.*
