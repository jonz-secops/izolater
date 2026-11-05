# Required features and functionality

Priorities: **P0** = useful hostile-aware v0.1; **P1** = hardening / multiplatform / break-glass; **P2** = later. Planning targets only.

## P0 — Minimum viable client

| ID | Feature | Notes |
| --- | --- | --- |
| F-01 | Install / bootstrap | Install root, ACLs, `tenant_id`, auth mode, capability probe |
| F-02 | Reverse password (when mode needs it) | Set verifier; inject via stdin/file (env opt-in only) |
| F-03 | Hard isolate | Filters + routes + DNS + Wi‑Fi disconnect + disable non-CP-path NICs |
| F-04 | CP path exception | If CP configured, retain minimal HTTP/RPC path; else fail closed on general net |
| F-05 | Snapshot before mutate | All surfaces in F-03; integrity-protected |
| F-06 | Reverse via ReverseProof | Password proof for P0; refuse without proof in `password_required` |
| F-07 | Status | Isolated?, mode, tenant_id, CP configured?, last action |
| F-08 | Privilege checks | Unprivileged mutate → deny |
| F-09 | Local audit | install/isolate/reverse success+fail; no secrets in logs |
| F-10 | Platform: Windows | First backend (ADR-0007) |
| F-11 | Hostile CLI posture | No unauthenticated listening reverse API |
| F-12 | Wrong-password throttle | Progressive delay, capped; no permanent brick |

## P1 — Depth

| ID | Feature | Notes |
| --- | --- | --- |
| F-13 | macOS backend | Second platform |
| F-14 | Linux backend | Third platform |
| F-15 | Break-glass sister tooling | Tenant-scoped verify/sign; challenge–response preferred |
| F-16 | Auth modes | `password_required` / `password_optional` / `password_disabled` |
| F-17 | Session drain/kill | Policy for existing non-CP connections (pending owner decision) |
| F-18 | Tamper detection | Corrupt snapshot/config → refuse reverse |
| F-19 | Reboot persistence | Re-apply if still Isolated |
| F-20 | CP client stubs | Isolate/Reverse/Status/`ResetReversePassword`/`SetAuthMode` |
| F-21 | Password rotation (local) | old+new |
| F-22 | Profiles | Named hard / CP-only variants |
| F-23 | Uninstall | Proof if isolated; clean artifacts |

## P2 — Later

| ID | Feature | Notes |
| --- | --- | --- |
| F-24 | Drift watchdog | Detect Wi‑Fi/NIC rollback; re-apply / alert |
| F-25 | Per-install break-glass certs | Tighter than tenant-only |
| F-26 | Dual-control break-glass | Two-person rule |
| F-27 | TPM / hardware secrets | |
| F-28 | Signed policy packs | |
| F-29 | Process/disk containment | Only if threat model expands |
| F-30 | GUI / self-update | |

## Cross-cutting requirements

1. Snapshot missing ⇒ do not isolate.  
2. No plaintext password at rest.  
3. Tenant mismatch ⇒ break-glass fails closed.  
4. Automation without TTY via stdin/file/CP.  
5. Idempotent isolate; fail reverse closed.  
6. Distinct audit for password vs break-glass vs CP.  

## Traceability

- Isolation → ADR-0001, 0006  
- Crypto / injection / throttle → ADR-0002  
- Config → ADR-0003  
- CP → ADR-0004  
- Hostile assumptions → ADR-0005, threat-model  
- Platforms → ADR-0007  
- Break-glass / modes → ADR-0008  
