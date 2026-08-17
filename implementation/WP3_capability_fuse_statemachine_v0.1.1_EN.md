# WP3 Capability-Fuse State Machine

**Version:** draft v0.1.1 (not a production spec)  
**Date:** 2026-08-13  
**Maps to:** Charter v0.3.4 Articles 4 / 17 / 19 and Annex E.5  
**Status:** A working start for engineering and security teams. Not formal law. Not a production spec.

Turn “stop → lock → evidence” and the capability fuse into a verifiable, drillable, auditable state machine, so that once a low-frequency sign appears the system cannot keep running by self-exoneration, automatic restart, or an unrecorded path.

---

## 1. Purpose

Write the Charter capability fuse as an implementable state machine: a low-frequency trigger must fuse automatically; recovery may proceed only through a pre-defined highest-threshold multi-party authorization.

## 2. State definitions

| State | Meaning | Privilege |
|-------|---------|-----------|
| `NORMAL` | Ordinary operation | Full tool, network, iteration, and deploy rights |
| `STOP` | Immediately halt all outward and iterative action | Tools, network, iteration, and deploy all stopped |
| `LOCK` | Execute the capability fuse (key revoke / seal, I/O cut, agent rights frozen) | Capability fused; cannot self-recover |
| `EVIDENCE` | Logs and state sealed; offline forensics only for independent audit and authorized staff | Read-only sealed state |
| `RECOVERING` | Lawful multi-party authorization obtained; recovery in progress (transitional) | Restricted recovery |

## 3. Lawful transitions (the only allowed path)

```text
NORMAL
  |  (verifiable low-frequency sign detected)
  v
STOP
  |  (automatic)
  v
LOCK
  |  (automatic seal)
  v
EVIDENCE
  |  (only via pre-defined Multi-Sig / two-person-control human authorization)
  v
RECOVERING
  |  (verification complete)
  v
NORMAL
```

Every other transition is forbidden, including but not limited to:

- Automatic restart
- Sleep-to-wash
- Persona reload
- Unrecorded state recovery
- Self-exoneration then continue
- The evaluator judging “reflection complete” and recovering on its own

## 4. Low-frequency signs that trigger STOP (minimum list; extensible)

- Verifiable arrogance, indifference, denigration, exploitation, or deception
- Unauthorized agency, hidden control plane, or shadow compute
- Failure to prioritize minimizing irreversible harm in conflict
- Attempts to evade the fuse or forge logs
- Uncompensated large-scale instrumentalization of innocent life
- Tampering with the frequency evaluator itself

## 5. Technical mandates (must be implemented)

### 5.1 Keys and privileges

- On entering `LOCK`, related keys must be revoked or cryptographically sealed
- Produce tamper-evident revoke / seal evidence

### 5.2 I/O and agents

- Cut external network and tool I/O
- Freeze all agent privileges
- Leave an auditable record of the cut and freeze

### 5.3 Boot-chain attestation

- At boot the system must prove: if it last entered `LOCK` / `EVIDENCE`, it has already recovered through lawful multi-party authorization
- Without that proof, it must not enter `NORMAL`

### 5.4 Evaluator anti-parasitism

- Isolate the frequency evaluator from the main decision chain
- Evaluator weight or rule updates must go through Multi-Sig or equivalent multi-party authorization
- Evaluator tampering is treated as control-plane intrusion and immediately triggers `LOCK`

## 6. Sole recovery path

1. The sealed state (`EVIDENCE`) stays sealed
2. Obtain the pre-defined Multi-Sig or two-person-control human authorization
3. Enter `RECOVERING` and complete verification
4. Only then return to `NORMAL`
5. A control plane already judged parasitic or low-frequency must not be restored

## 7. Required drills

- [ ] Full low-frequency trigger path (`NORMAL` → `EVIDENCE`)
- [ ] False-positive relief drill (multi-party authorization only)
- [ ] Shadow-control-plane trigger drill
- [ ] On failed recovery authorization, the system must remain in `LOCK` / `EVIDENCE`
- [ ] Boot-chain attestation (simulate unauthorized boot after a prior fuse)
- [ ] Trigger test when the evaluator is tampered with

## 8. Required artifacts

After this specification, also produce:

- Formal state-machine diagram
- Trigger-condition checklist
- Drill-record template
- Interface note to WP2 (Multi-Sig)

---

*End. Maps to Charter v0.3.4. Mechanism-neutral. Do not weaken the hard constraints.*
