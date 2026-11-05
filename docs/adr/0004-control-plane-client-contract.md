# ADR-0004: Control-plane client contract

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

A future **izolater server** orchestrates fleets. This repo implements **client hooks/contracts only**. Owner additions:

- While isolated, **only** CP services (HTTP/RPC) remain reachable (plus local reverse/break-glass).  
- CP should eventually **reset** the reverse quarantine password or **remove** the password requirement.

## Decision

1. Keep versioned contracts under `contracts/` (Markdown now; schema later).  
2. **Command set** (indicative):

   | Command | Purpose |
   | --- | --- |
   | `Isolate` | Apply profile; ensure CP allowlist path |
   | `Reverse` | Restore; `ReverseProof` may be CP capability |
   | `Status` | State, mode, tenant_id, versions |
   | `ResetReversePassword` | Set new verifier **without** old password (CP-auth only) |
   | `SetAuthMode` | e.g. `password_required` → `password_disabled` / `password_optional` |
   | `RotateBreakGlassTrust` | Update tenant public key / CA material |
   | `ApplyProfile` | Swap isolation profile |

3. **Transport assumption:** mutually authenticated channel (mTLS or equivalent); client dial-out preferred for NAT; trust anchors in install config.  
4. **Isolation allowlist:** CP endpoints from install/runtime config are the **only** intentional remote exception in the hard profile (ADR-0001).  
5. **Local CLI** remains usable offline; CP may be dormant.  
6. **No server implementation** in this repository.

## Consequences

- Client auth module must treat CP as a first-class `ReverseProof` and config authority for password/mode.  
- Resetting password or disabling password requirement via CP is a **high-impact** control — must be tightly authenticated and audited.  
- Hard isolation without reachable CP still needs local password and/or sister break-glass (ADR-0008).

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Ignore CP until server exists | Would force breaking redesign of allowlist + auth modes |
| CP always required for reverse | Breaks air-gapped / CP-outage IR |

## Open questions

- HTTP vs custom RPC vs both for v1 CP protocol (owner said “probably via http or rpc”).  
- Can CP reverse with capability **alone** when mode is `password_required`? (**Suggested: yes**, CP proof satisfies the gate; password not also required.)  
- Authorization model for `SetAuthMode` (which CP roles)?
