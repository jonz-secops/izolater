# ADR-0001: Platform isolation mechanisms

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Hard containment on hostile endpoints: cut remote control paths while remaining reversible. Owner refinements: **kill existing TCP sessions**, **clear DNS/ARP-related caches**, optional **Bluetooth / cellular** disable via feature flags, **never disable USB**.

## Decision

1. **Default hard profile** (all reversible via snapshot):

   | Step | Intent |
   | --- | --- |
   | A. Snapshot | Filters, routes, DNS, interface/Wi‑Fi/BT/cellular admin state as applicable |
   | B. DNS policy | Override/block resolvers except what CP allowlist needs |
   | C. Routes | Remove general egress; keep CP path if configured |
   | D. Packet filter | Deny most traffic; allow CP endpoints only (when configured) |
   | E. **Kill non-CP TCP** | Actively terminate established TCP sessions that are not on the CP allowlist |
   | F. **Flush caches** | Clear DNS resolver caches and ARP/neighbor caches (and OS equivalents) so stale resolutions/L2 entries cannot bridge policy |
   | G. Wi‑Fi | Disconnect / disable Wi‑Fi used for general connectivity |
   | H. Other NICs | Disable interfaces not on the retained CP/uplink path |
   | I. Bluetooth | **If feature flag enabled** (tenant default or install override): disable BT radios / PAN |
   | J. Cellular / WWAN (“smart call”) | **If feature flag enabled**: disable cellular/WWAN interfaces |
   | K. USB | **Do not disable** USB as part of isolation |

2. **Feature flags** (tenant default, overridable per install): `disable_bluetooth`, `disable_cellular` (names indicative). Default for hostile templates: recommend `true` once backends are reliable; until then document cautiously.  

3. **CP exception:** minimal path for CP (HTTPS/mTLS — ADR-0004). No CP configured ⇒ no general remote path.  

4. **Orchestrator + platform backends** (`probe` / `snapshot` / `apply` / `restore` / `status`) on Windows → macOS → Linux.  

5. **Bypasses** inventoried (WSL, VPN, Hyper-V, IPv6, tether, etc.) — mitigate, document, or defer explicitly.  

6. **Success metric:** remote controller loses the channel (sockets die / unroutable), not merely “a rule exists.”

## Consequences

- Stronger C2 interruption; more complex restore (session state is not “restored,” only network posture is).  
- Cache flush reduces stealthy reuse of poisoned DNS/ARP after policy apply.  
- USB left up preserves local IR media / break-glass USB workflows.  
- BT/cellular flags avoid surprising tenants that rely on those radios for out-of-band IR (they must opt policy consciously).

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Filter-only, leave TCP up | Owner rejected; RAT sessions can linger |
| Disable USB | Owner rejected; breaks local IR/break-glass media |
| Always kill BT/cellular with no flag | Too blunt; owner wants per-tenant/install control |

## Open questions

- Exact OS APIs for TCP kill and ARP/DNS flush per platform (spike).  
- Whether UDP “sessions” / QUIC need explicit handling beyond filter+route.  
- Default flag values for `disable_bluetooth` / `disable_cellular` in hostile template.
