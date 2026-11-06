# ADR-0001: Hard-isolate mutation sequence

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Define **exactly what izolater changes** on isolate so a remote controller loses the channel—while leaving USB and a future CP path intact.
- **Owns:** Mutation sequence, radio/NIC policy, CP allowlist *as a network exception*, success metric for isolate.
- **Defers:** Snapshot/restore/integrity → [0006](0006-reversibility-model.md); threat ceiling → [0005](0005-threat-model-assumptions.md); CP protocol → [0004](0004-control-plane-client-contract.md); OS ship order → [0007](0007-multiplatform-strategy.md).

## Fits in the system

0001 is the **actuator**. 0006 is its twin (nothing in this sequence applies without a restorable snapshot). 0005 demands the sequence kill C2, not decorate firewalls. 0004 names which endpoints may remain in the allowlist. 0003 holds the feature flags that gate Bluetooth/cellular.

## Context

Firewall-only quarantine is too weak for izolater’s hostile brief. Owner direction: filters **and** routes **and** DNS **and** TCP kill **and** cache flush **and** Wi‑Fi/NIC actions; optional BT/cellular; **never USB**.

## Decision

### Hard profile sequence (all reversible via 0006)

Apply in an order that **cuts egress early**, then cleans residual state:

| Step | Mutation | Why it’s here |
| --- | --- | --- |
| 0 | **Snapshot** (0006) | No mutate without restore plan |
| 1 | DNS policy | Stop name-based bypass of allowlist |
| 2 | Routes | Remove general egress; keep CP path if configured |
| 3 | Packet filter | Deny-most; allow only CP endpoints when configured |
| 4 | **Kill non-CP TCP** | Established RAT sessions must die, not linger |
| 5 | **Flush DNS + ARP/neighbor caches** | Stale resolver/L2 entries must not bridge policy |
| 6 | Wi‑Fi disconnect/disable | Kill common alternate uplink |
| 7 | Disable non–CP-path NICs | Shrink lateral/physical paths |
| 8 | Bluetooth *(flag)* | Optional PAN/radio cut |
| 9 | Cellular/WWAN *(flag)* | Optional “smart call” cut |
| — | **USB** | **Explicit non-goal:** do not disable |

### Policy knobs

- Flags `disable_bluetooth`, `disable_cellular`: **tenant default, install override** (stored per 0003).  
- **CP exception:** if CP endpoints configured, retain minimal path for them (transport per 0004). If none → no intentional remote path.  
- **Bypasses** (WSL, VPN, Hyper-V, IPv6, tether, …): inventory per OS; mitigate, document, or explicitly defer—never ignore.

### Success metric

Isolate **succeeds** when a pre-existing non-CP remote channel fails and new general egress fails—not when “rules exist.”

### Architecture hook

One orchestrator calls platform backends: `probe` / `snapshot` / `apply` / `restore` / `status` (0007 places backends).

## Consequences

- Restore complexity rises with surfaces touched (0006).  
- USB stays up → local IR and offline sister media still work (0008).  
- TCP kill is best-effort per OS API; document gaps, don’t pretend perfect.

## Alternatives rejected

| Alternative | Why rejected for izolater |
| --- | --- |
| Filter-only, leave TCP up | Owner: RAT sessions linger |
| Disable all NICs / pull cable | Breaks CP exception and local IR |
| Always kill BT/cellular with no flag | Too blunt; tenant/install control required |
| Disable USB | Owner: keep local IR / break-glass media |

## Open

- Per-OS APIs for TCP kill and cache flush (spike on Windows first).  
- UDP/QUIC beyond filter+route—whether extra handling is required.  
- Hostile-template defaults for BT/cellular flags once backends are reliable.
