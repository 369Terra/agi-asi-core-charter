# Implementation starter pack（初稿・非生產規格）

對應憲章 v0.3.4　第 4／17／19 條、附錄 D／E.5

This folder is a **first-draft starter pack**, not production spec and not formal law.  
It is for teams who want to refine and implement the Charter.  
Do not weaken the hard constraints. See the Charter Hand-off note.

歡迎在不弱化硬約束的前提下繼續細化。

## Read in this order

1. [WP3_capability_fuse_statemachine_v0.1.1.pdf](./WP3_capability_fuse_statemachine_v0.1.1.pdf) — STOP → LOCK → EVIDENCE → RECOVERING  
2. [WP2_Multi-Sig根金鑰政策_v0.1.1.pdf](./WP2_Multi-Sig根金鑰政策_v0.1.1.pdf) — Multi-Sig thresholds  
3. [WP2_WP3_integration_v0.1.pdf](./WP2_WP3_integration_v0.1.pdf) — how they meet  
4. [WP2_WP3_介面契約_v0.1.md](./WP2_WP3_介面契約_v0.1.md) / [WP2_WP3_interface_contract_v0.1.md](./WP2_WP3_interface_contract_v0.1.md) — minimum interface contract

## Closed loop (must not break)

Low-frequency trigger → WP3 fuses automatically (no Multi-Sig required first).  
Recovery → only via WP2 highest-threshold multi-sig.  
Otherwise the system must not return to `NORMAL`.
