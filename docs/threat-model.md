# Threat model sketch

Status: **draft for validation — not an accepted ADR**. Assumptions: [ADR-0005](adr/0005-threat-model-assumptions.md). Break-glass: [ADR-0008](adr/0008-break-glass-and-auth-modes.md).

## Design posture: hostile by default

izolater must assume the endpoint is **already or soon contested**:

| Hostile factor | Example |
| --- | --- |
| Malware | Ransomware staging, stealers, living-off-the-land |
| Remote control via socket | RAT / C2 over TCP/UDP/HTTP(S)/HTTP2/WebSocket/named tunnels |
| Rogue or compromised AI / automation | Agents that can shell out, write files, or call izolater as a user or admin |
| Insider / stolen creds | Operator password or break-glass key mishandling |

Containment aims to **deny remote control paths** and **deny clean unisolate** without cryptographic proof — while remaining reversible for legitimate IR.

## Assets

| Asset | Why it matters |
| --- | --- |
| Reverse password / verifier | Clean unisolate via izolater |
| Tenant break-glass key material | Tenant-wide unisolate capability — protect like crown jewels |
| Restore snapshot | Integrity ⇒ safe reverse; tampering ⇒ wrong posture or refuse |
| Isolation enforcement state | Filters, routes, DNS, Wi‑Fi, NIC admin state |
| CP trust anchors + allowlist | Forgeable CP = remote unisolate / mode change |
| Audit log | Accountability under hostility |
| Binary & install root | Replacement binary could skip auth |

## Actors

| ID | Actor | Notes |
| --- | --- | --- |
| A1 | IR operator | Legitimate isolate/reverse; holds password or break-glass process |
| A2 | Non-admin malware | Wants egress/C2; may try to invoke izolater |
| A3 | Admin / SYSTEM / root malware | Can fight OS network stack directly; may abuse or kill izolater |
| A4 | Remote C2 operator | Maintains sockets; tries to survive isolate |
| A5 | Rogue AI / compromised automation | Local process with some rights; high speed, tool-using |
| A6 | Malicious insider with password or tenant key | Can reverse within blast radius |
| A7 | Compromised future CP | Fleet-scale impact if client trust is weak |
| A8 | Cross-tenant attacker with stolen sister binary | Must fail closed outside tenant |

## Security goals

| ID | Goal |
| --- | --- |
| G1 | Without a valid `ReverseProof`, izolater’s supported interfaces will not reverse isolation |
| G2 | Hard isolate removes general remote socket paths (filters/routes/DNS/Wi‑Fi/extra NICs); only CP allowlist remains if configured |
| G3 | Reverse restores pre-isolate posture (no silent residual holes or permanent blackhole) |
| G4 | Password not recoverable from on-disk verifier under stated KDF params |
| G5 | Break-glass proofs are tenant-bound; no global unlock |
| G6 | Rogue local automation without secrets cannot unisolate via izolater |
| G7 | Tampered snapshot/config ⇒ refuse reverse (fail closed) |
| G8 | Client usable offline without CP |

## Non-goals of protection (honest limits)

Against **A3**, izolater cannot guarantee the OS network stack stays quarantined. We optimize for:

- denying izolater as a **convenient unisolate API** without proof  
- integrity and audit  
- cutting *default* C2 paths quickly  
- tenant-scoped recovery  

## Abuse cases (hostile)

1. RAT open socket → `isolate` → socket path should die / be unroutable except CP.  
2. Malware runs `izolater reverse` with no password → fail, throttle, audit.  
3. Rogue agent reads `IZOLATER_PASSWORD` from environ → **mitigate by disabling env by default** (ADR-0002).  
4. Adversary replaces snapshot with allow-all → integrity check fails.  
5. Stolen Tenant A sister binary against Tenant B host → `tenant_id` mismatch, refuse.  
6. Attacker spoofs CP over HTTP → mTLS/pinning must reject.  
7. Admin malware toggles Wi‑Fi outside izolater → outside guarantee; detect drift on status if possible.  
8. Compromised automation with admin rights + `password_disabled` → can reverse — **mode warning**.

## STRIDE notes

| Category | Hostile example | Planned direction |
| --- | --- | --- |
| Spoofing | Fake CP, fake break-glass | mTLS; signed proofs bound to tenant/install |
| Tampering | Snapshot/config/binary | MAC/sign; refuse; optional binary signature later |
| Repudiation | Silent reverse | Distinct audit for password vs break-glass vs CP |
| Info disclosure | Password on argv/env | stdin/file/CP preference; no plaintext at rest |
| DoS | Lockout via bad passwords; isolate without reverse path | Capped throttle; break-glass; CP exception |
| Elevation | Unprivileged mutate | Hard privilege checks |

## Open risks

| ID | Risk |
| --- | --- |
| R1 | Admin malware undoes NIC/firewall outside tool |
| R2 | Bypass paths (WSL, VPN, Hyper-V, IPv6, tether) keep sockets alive |
| R3 | Existing TCP sessions may survive until reset — need decide kill/drain |
| R4 | `password_optional` / `disabled` weaken hostile claims |
| R5 | Tenant break-glass key compromise = tenant-wide blast radius (accepted ceiling) |
| R6 | Race: C2 exfils during the isolate apply window |
| R7 | Rogue AI with break-glass key file access |

## Clarifying questions (hostile + prior)

See also the user-facing [izolater-plan.md](izolater-plan.md).

1. Should isolate **actively terminate** established non-CP connections, or only block new packets/routes?  
2. Persist a **privileged service** that re-applies quarantine if malware undoes rules (best-effort watchdog)?  
3. Any requirement to detect/report **drift** (Wi‑Fi back on, NIC re-enabled) to CP/local audit?
