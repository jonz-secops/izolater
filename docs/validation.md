# Security-focused validation — prove before coding

This checklist gates the start of Zig product implementation. Items are **must-prove** unless marked advisory.

## Direction strength (product)

| # | Proof | How | Done when |
| --- | --- | --- | --- |
| V1 | Isolation semantics are clear | Written acceptance tests per OS for “isolated” | Reviewer agrees pass/fail observables |
| V2 | Reverse is exact | Snapshot/restore table per backend | Round-trip demo on clean VM |
| V3 | Password model fits automation + human IR | Walkthrough of install + reverse channels | Owner accepts threat tradeoffs |
| V4 | CP seam doesn’t force premature server | `contracts/` review | No server code required to ship CLI |

## Security proofs (pre-code / design)

| # | Proof | How | Done when |
| --- | --- | --- | --- |
| S1 | Threat model reviewed | Challenge session on [threat-model.md](threat-model.md) | Open risks accepted or mitigated in ADRs |
| S2 | Secret storage design | ADR-0002 + parameter selection | No plaintext path on disk |
| S3 | Integrity of restore state | Design for MAC/checksum tied to install secret | Tamper → refuse reverse |
| S4 | Privilege model | Document required rights per OS | Unprivileged mutate denied |
| S5 | Bypass inventory | Enumerate IPv6, VPN, WSL, Hyper-V, Docker, iCloud Private Relay, etc. | Explicit allow/deny/ defer per bypass |
| S6 | Fail behavior | Matrix: isolate fail, reverse fail, corrupt state | No silent success |

## Platform proofs (spike-level OK, still pre-product)

Spikes may use throwaway scripts **outside** product `src/` if needed later; this planning phase records *what* to spike, not the spikes themselves.

| OS | Must demonstrate |
| --- | --- |
| Windows | Block egress to public IP; restore prior firewall posture; survive reboot story |
| Linux | Equivalent with nftables or chosen API; persist across reboot |
| macOS | Equivalent with pf or chosen API; persist across reboot |

## What must be true before writing product Zig

1. ADRs 0001–0007 accepted or explicitly superseded.  
2. Open questions in the plan answered or explicitly deferred with owners.  
3. Bypass inventory (S5) drafted for the first OS.  
4. CLI command set and exit codes frozen enough for tests.  
5. Contract stubs for CP reviewed so client won’t paint into a corner.

## Ongoing validation (once coding starts — foreshadow only)

- Property tests: isolate → status isolated → reverse → status clear.  
- Negative tests: wrong password, truncated snapshot, dropped privileges.  
- Timing: reverse verify roughly constant-time.  
- External review of crypto/KDF parameters.

## Exit criteria for “planning complete”

- [ ] User-facing plan published  
- [ ] Features prioritized  
- [ ] ADRs filed  
- [ ] Threat model + open risks listed  
- [ ] Directory scaffolds present  
- [ ] Owner decisions on break-glass, OS order, injection channels  
