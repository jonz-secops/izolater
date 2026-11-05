# Security-focused validation — prove before coding

Gates before Zig product implementation. ADRs remain Proposed until owner accepts.

## Direction

| # | Proof | Done when |
| --- | --- | --- |
| V1 | Hard-isolate semantics clear per OS | Pass/fail observables agreed (filter/routes/DNS/Wi‑Fi/NIC/CP) |
| V2 | Reverse exact | Round-trip restore on clean VM |
| V3 | Auth modes + injection channels accepted | Owner signs off tradeoffs |
| V4 | CP seam doesn’t force server | `contracts/` review |

## Hostile / security proofs

| # | Proof | Done when |
| --- | --- | --- |
| S1 | Threat model challenged (malware, sockets, rogue AI) | Open risks accepted or mitigated in ADRs |
| S2 | Secret storage + injection ranking | No plaintext; env default off |
| S3 | Snapshot integrity independent of user password | Tamper ⇒ refuse reverse |
| S4 | Privilege model | Unprivileged mutate denied |
| S5 | Bypass inventory OS#1 (Windows) | WSL/VPN/Hyper-V/IPv6/tether listed |
| S6 | Fail matrix | No silent success |
| S7 | Adversarial reverse | Wrong password / no proof / rogue script → no change |
| S8 | Tenant break-glass mismatch | Cross-tenant proof refused |
| S9 | C2 path cut | Test socket to public IP fails after isolate (CP still works if configured) |
| S10 | Throttle IR tradeoff | Caps delay; break-glass still usable |

## Platform spikes (pre-product)

| Order | OS | Must show |
| --- | --- | --- |
| 1 | Windows | Hard isolate + restore + reboot story |
| 2 | macOS | Same |
| 3 | Linux | Same |

## Before product Zig

1. Owner answers clarifying questions in [izolater-plan.md](izolater-plan.md).  
2. ADRs accepted or explicitly deferred.  
3. Windows bypass inventory drafted.  
4. CLI + exit codes stable enough for tests.  
5. Break-glass crypto sketch reviewed (ADR-0008).  

## Planning-complete checklist

- [x] User-facing plan published (living doc)  
- [x] Features prioritized  
- [x] ADRs filed (0001–0008) — **acceptance pending**  
- [x] Hostile threat model sketched  
- [x] Directory scaffolds present  
- [ ] Owner clarifying questions answered  
- [ ] ADR acceptance wave  
