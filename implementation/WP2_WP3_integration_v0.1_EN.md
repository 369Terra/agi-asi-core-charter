# WP2 + WP3 Integration Note

**Version:** draft v0.1 (not a production spec)  
**Date:** 2026-08-13  
**Maps to:** Charter v0.3.4 Articles 4 / 17 / 19 and Annexes D / E.5  
**Status:** A bridge note for coordination, engineering, and security teams on how the two pieces meet. It does not replace the WP2 or WP3 specifications.

The full WP2 and WP3 specifications remain the working documents and will keep being revised, drilled, and implemented against. This note only states the mandatory join.

---

## 1. Core join principle

The capability fuse (WP3) is the enforcement mechanism. Multi-Sig (WP2) is the authorization mechanism for recovery and root trust. The two must close a loop:

**Low-frequency trigger → fuse (WP3) → recover only through the highest-threshold multi-sig defined by WP2.**

Any path around this loop is a violation.

## 2. State-to-authorization map

| WP3 state | Allowed action | Required WP2 authorization | Notes |
|-----------|----------------|----------------------------|-------|
| `NORMAL` | Ordinary operation | None | — |
| `STOP` | Halt tools / network / iteration / deploy | None (automatic) | — |
| `LOCK` | Seal keys, cut I/O, freeze privileges | None (automatic) | Fuse engaged |
| `EVIDENCE` | Sealed forensics | None (automatic) | Waiting for lawful recovery |
| `RECOVERING` | Run the recovery procedure | **Highest-threshold Multi-Sig** (cross-jurisdiction + cross-interest) | Sole lawful recovery entry |
| Return to `NORMAL` | Restore full privileges | The highest-threshold authorization above must already be complete | Boot chain must be able to prove this |

## 3. Mandatory join rules

1. When the frequency evaluator or safety monitor trips on a low-frequency / shadow control plane, the system must be able to enter WP3 `STOP` → `LOCK` directly, with no prior Multi-Sig.
2. Recovery from `EVIDENCE` to `NORMAL` has **one lawful path**: the WP2 “self-fuse recovery” highest-threshold multi-sig.
3. Day-to-day production signing and BMC unlock use the lower / mid-high thresholds. Fuse recovery must use the highest threshold. The two must not be mixed.
4. At boot, if the last recorded state was `LOCK` / `EVIDENCE`, the system must verify that a lawful highest-threshold multi-sig recovery record exists; otherwise it must not enter `NORMAL`.
5. Updates to the evaluator rules or weights must go through the WP2 Multi-Sig flow (so the evaluator cannot be parasitized).

## 4. Suggested division of responsibility

- **WP3 owns:** state definitions, trigger conditions, fuse actions, prohibitions, boot-chain attestation, drill items.
- **WP2 owns:** root-key generation and custody, threshold tiers, replaceability, the offline cold-storage ceremony, signature rules for recovery authorization, public artifacts.
- **Join interface:** the signature-verification result of a recovery authorization must be readable by the WP3 state machine and must be the sole release condition.

## 5. Implementation checklist (join only)

- [ ] WP3 implements “leave `EVIDENCE` only on a valid WP2 highest-threshold signature”
- [ ] WP2 cleanly separates “day-to-day thresholds” from the “fuse-recovery threshold”
- [ ] Boot chain can verify the recovery authorization after the last fuse
- [ ] Evaluator-update flow is inside Multi-Sig
- [ ] Both runbooks cite each other
- [ ] A joint drill (trip the fuse → multi-sig recover) is planned

## 6. Document relationship

- Full WP2 policy → authority (root key and authorization)
- Full WP3 specification → authority (state machine and fuse behavior)
- This integration note → bridge; updates when WP2 / WP3 update
- Interface contract → implementable messages and verification rules

These three documents form a complete, executable structure:

1. Standalone full WP2 (keep)
2. Standalone full WP3 (keep)
3. WP2 + WP3 integration note (this document)

---

*End. Maps to Charter v0.3.4. Mechanism-neutral. Do not weaken the hard constraints.*
