# ADR-0001: Platform isolation mechanisms

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)
- Supersedes prior draft scope: “firewall-only”

## Context

izolater must **quickly isolate an endpoint** in a way that is **reversible** and implementable from a Zig binary across Windows, macOS, and Linux. Owner direction expands isolation beyond packet filters alone:

- Network filtering  
- Routes  
- DNS  
- Shut down **non–internet-connected** interfaces  
- **Disconnect Wi‑Fi**  
- **Sole exception:** reachability to the future **control-plane** service (HTTP and/or RPC)

Process/disk lockdown remain out of scope for now.

## Decision

1. **Isolation profile (default hard profile)** applies a coordinated multi-step containment plan, all reversible via snapshot:

   | Step | Intent |
   | --- | --- |
   | A. Snapshot | Capture firewall/pf/nft rules, routes, DNS config, interface admin state, Wi‑Fi association metadata needed to restore |
   | B. DNS | Pin/override resolver behavior so name resolution cannot bypass policy (block or redirect per profile; CP hostnames resolvable if allowlisted) |
   | C. Routes | Remove/replace routes that enable general egress; retain only path(s) required for CP allowlist (if configured) |
   | D. Packet filter | Deny most ingress/egress; allow CP endpoints (and minimal supporting traffic if proven necessary, e.g. ARP/NDP on the CP path) |
   | E. Wi‑Fi | Disconnect / disable Wi‑Fi radios or profiles used for general connectivity |
   | F. Other NICs | Administratively disable interfaces that are **not** on the path to the CP / not providing the retained uplink |

2. **CP exception:** when CP endpoints are configured, isolation **must leave** a minimal path (filter + route + DNS) so HTTP/RPC to those endpoints can work. When CP is not configured, fail closed on general network (local reverse / break-glass still apply).  

3. **Architecture:** isolation orchestrator + platform backend interface: `probe`, `snapshot`, `apply(profile)`, `restore(snapshot)`, `status`. Backends implement OS-specific operations for filters, routes, DNS, interface, and Wi‑Fi.  

4. **Default backends (intent):**  
   - **Windows:** Firewall/WFP + route table + DNS client settings + interface/`netsh`/appropriate APIs + WLAN disconnect.  
   - **macOS:** pf anchors + routes + DNS (scutil/resolver config as appropriate) + interface/Wi‑Fi control.  
   - **Linux:** nftables + routes + resolvers (resolvectl/NetworkManager as detected) + ip link / nmcli Wi‑Fi.  

5. **Profiles** encode the steps above as data; OS backends translate.  

6. **Bypasses** (WSL, Hyper-V, containers, VPNs, IPv6, tether, Thunderbolt bridges, etc.) are inventoried per OS and mitigated, documented, or explicitly deferred — never ignored.

7. **Reversibility:** every mutate step is covered by the snapshot; partial failure rolls back (ADR-0006).

## Consequences

- Much stronger IR containment story; also **higher restore complexity** and more privilege surface.  
- Wi‑Fi disconnect + NIC disable can strand a host if CP path and local reverse are both wrong — break-glass (ADR-0008) is mandatory product thinking.  
- “Internet vs non-internet interface” classification needs a clear probe heuristic (default route membership, operstate, medium type) — must be validated on real hardware.  
- Spikes required on **Windows first**, then macOS, then Linux (ADR-0007).

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Firewall-only | Owner rejected as insufficient from day one |
| Pull cable / disable *all* NICs | Breaks CP exception and often local IR |
| Kernel driver quarantine | Too heavy for v1 |

## Hostile-environment note

Assume active **socket C2 / RAT** and malware that will race the apply window. Isolation must make general remote sockets unroutable as early as practical; optional **session kill/drain** for non-CP connections is an open owner decision. Success is measured by “remote controller loses the channel,” not by “a rule was inserted.”

## Suggestions

1. Treat “CP path” as a first-class object: `{endpoints[], transport: http|rpc, dns_names[]}`.  
2. Prefer **disable Wi‑Fi** over deleting profiles so restore is clean.  
3. Record **before/after** interface inventory in audit for forensics.  
4. Add a dry-run / `isolate --plan` that prints the intended mutations without applying.  
5. Order mutations to cut egress early (e.g. routes/filter before slower Wi‑Fi/NIC steps) to shrink the C2 race window.

## Open questions

- Exact heuristic for “non–internet-connected interface” (see clarifying questions in the plan).  
- Is Bluetooth PAN / USB tether in scope for disable?  
- Minimum traffic allowed to reach CP (DNS to public resolvers vs only static CP IPs)?  
- Windows API preference once spikes run (WFP vs Firewall Policy COM, etc.).
