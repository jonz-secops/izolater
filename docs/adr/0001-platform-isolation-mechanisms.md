# ADR-0001: Platform isolation mechanisms

- Status: Proposed
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

izolater must **quickly isolate an endpoint** in a way that is **reversible** and implementable from a Zig binary across Windows, Linux, and macOS. “Isolate” in v1 means **network containment**: reduce the host’s ability to reach (and be reached by) untrusted networks while preserving a documented recovery path.

Different OSes expose different enforcement points (Windows Filtering Platform / firewall rules, nftables/iptables, pf). Choosing poorly creates bypasses or irreversible messes.

## Decision

1. **Scope of isolation (v1):** network layer only (host firewall / packet filter). Process, disk, and identity lockdown are out of scope (see non-goals).  
2. **Architecture:** a single **isolation orchestrator** calls a **platform backend** interface: `probe`, `snapshot`, `apply(profile)`, `restore(snapshot)`, `status`.  
3. **Default backends (intent):**  
   - **Windows:** Windows Firewall with Advanced Security / WFP-oriented APIs as available from userspace with admin rights; prefer documented, restorable rule sets over ad-hoc `netsh` string soup. Exact API binding chosen during spike; must support export/import or equivalent snapshot.  
   - **Linux:** **nftables** preferred; iptables-legacy only as fallback if detected.  
   - **macOS:** **pf** with izolater-owned anchors so restore does not clobber unrelated rules.  
4. **Profiles:** isolate applies a named **profile** (default: deny-most with optional CP/IR allowlist). Profiles are data, not hard-coded per OS — OS backends translate profile intents.  
5. **Bypasses:** known bypass classes (WSL, containers, Hyper-V switches, VPNs, IPv6) are **inventoried** and either mitigated in-profile, documented as limitations, or deferred explicitly — never ignored.

## Consequences

- Positive: one mental model; testable per backend; reversible if snapshot API is honest.  
- Negative: userspace firewall changes are defeatable by admin malware (accepted in ADR-0005).  
- Spike work required before coding to confirm snapshot/restore fidelity on Windows first.

## Alternatives considered

| Alternative | Why not (for now) |
| --- | --- |
| Kill NIC / disable adapters | Brutal, often breaks IR, messy reverse, hardware-specific |
| Network namespace only (Linux) | Doesn’t isolate the host’s other namespaces; weak multiplatform story |
| Driver / kernel module | High maintenance, signing hell, overkill for v1 |
| Cloud NAC / 802.1X quarantine | Requires infra; not a local binary |

## Open questions

- Confirm Windows API preference (Firewall Policy COM vs WFP vs helper service).  
- Should “allow DHCP/DNS to local resolvers” be default in hard profile?  
- Is blocking localhost lateral (container bridges) in scope for v1?
