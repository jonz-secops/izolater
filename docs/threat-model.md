# Threat model sketch

Status: **draft for validation**. Assumptions are locked in [ADR-0005](adr/0005-threat-model-assumptions.md). This doc is the working sketch operators and reviewers should challenge before coding.

## Assets

| Asset | Why it matters |
| --- | --- |
| Reverse password / verifier | Controls ability to restore network; compromise enables silent unisolate or DoS via lockout policies |
| Restore snapshot | Integrity required for safe reverse; tampering can brick connectivity or leave holes |
| Isolation enforcement | Effectiveness of containment against malware C2 / exfil |
| Audit log | Forensics and accountability |
| Binary & install root | Integrity of the tool itself |
| Future CP credentials | Lateral trust into fleet control (client-side only for now) |

## Actors

| Actor | Capability (assumed) |
| --- | --- |
| A1 Operator / IR | Legitimate isolate/reverse; knows or can retrieve password via out-of-band process |
| A2 Local malware (user) | Runs as non-admin user; can use network if not isolated |
| A3 Local malware (admin/SYSTEM/root) | Can often disable host firewalls, kill processes, read many files |
| A4 Remote attacker (pre-isolate) | Network reachability to host services |
| A5 Remote attacker (post-isolate) | Should be cut off except allowlisted paths |
| A6 Malicious insider with password | Can reverse isolation |
| A7 Compromised future CP | Can issue client commands if channel/auth weak |

## Trust boundaries

1. **Process boundary** — izolater CLI vs other local processes  
2. **Privilege boundary** — admin/root required for mutate ops  
3. **Secret boundary** — password material in memory / injection channels  
4. **Network boundary** — host vs external network (isolation goal)  
5. **Future CP boundary** — authenticated control channel (contract only)

## Security goals

| ID | Goal |
| --- | --- |
| G1 | Unauthorized party without reverse secret cannot reverse isolation via izolater’s supported interfaces |
| G2 | Isolate meaningfully reduces default network connectivity for attacker C2/exfil |
| G3 | Reverse restores prior connectivity posture (no silent residual deny/allow) |
| G4 | Password not recoverable from on-disk verifier under stated KDF parameters |
| G5 | Failed reverse attempts do not weaken isolation |
| G6 | Client remains usable offline without CP |

## Non-goals of protection (honest limits)

Against **A3 (admin-equivalent malware)**, no userspace tool can guarantee enforcement: the attacker can disable firewalls, unload drivers, or patch the binary. izolater **raises cost** and provides **operator-controlled, auditable** containment — it is not a hypervisor or MDM root of trust.

We still design so that:

- casual admin mistakes don’t dump plaintext secrets;  
- malware without the password cannot use *izolater* as a clean reverse API;  
- tampering with snapshots is detected rather than silently trusted.

## STRIDE-oriented notes

| Category | Example risks | Mitigations (planned) |
| --- | --- | --- |
| Spoofing | Fake CP commands | mTLS + pinned trust (future); local CLI auth for reverse |
| Tampering | Corrupt snapshot / config | Checksums/HMAC with key derived from install secret; refuse reverse |
| Repudiation | No record of reverse | Local audit log; future CP aggregation |
| Info disclosure | Password in world-readable config | Verifier only; restrictive ACLs; no plaintext |
| DoS | Isolate breaks IR path permanently | Reversible snapshots; break-glass process documented; optional CP allowlist |
| Elevation | Unprivileged isolate/reverse | Hard privilege checks |

## Abuse cases to design against

1. Attacker calls `izolater reverse` without password → fail, no change.  
2. Attacker replaces snapshot with “empty allow-all” → integrity check fails.  
3. Attacker reads config looking for password → finds verifier only.  
4. Operator isolates without snapshot capacity → isolate aborted.  
5. Host reboots while isolated → posture persists / re-applies.  
6. Automation leaks password via process list → prefer stdin/file with tight ACLs; document env risk.

## Open risks (must stay visible)

See [validation.md](validation.md) for proof gates. Highlights:

- **R1** Admin malware defeats host firewalls — product positioning must stay honest.  
- **R2** Password injection via environment is visible to co-resident processes.  
- **R3** Platform API differences may leave bypass paths (IPv6, WSL, VM switches, containers).  
- **R4** Break-glass if password lost — policy decision needed (see open questions).  
- **R5** Future CP becomes highest-value target — client must pin/limit trust.

## Open questions for skrappy0x4a

Recorded also in the user-facing plan:

1. Is **network-only** isolation sufficient for v1, or is process/disk containment in scope later?  
2. **Break-glass**: if password is lost while isolated, is reimage / offline recovery the only path, or is a sealed recovery key required?  
3. Primary launch OS order confirmation: **Windows → Linux → macOS**?  
4. Acceptable password injection channels for automation (env vs stdin vs named pipe vs CP-only)?  
5. Should wrong-password reverse attempts lock out / delay, and can that create an IR DoS?
