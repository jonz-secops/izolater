# ADR-0004: Control-plane client contract

- Status: Proposed
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

A future **izolater server** will orchestrate fleets. This repo must not implement that server, but the client must not paint itself into a corner. We need **hooks, contracts, and assumptions** only.

## Decision

1. **Separate `contracts/` tree** describes versioned client capabilities and message shapes (Markdown now; JSON Schema/protobuf later if needed).  
2. **Command set** the client can eventually receive (names indicative):  
   - `Isolate { profile, request_id }`  
   - `Reverse { proof, request_id }`  
   - `Status`  
   - `RotateReverseSecret { … }` (carefully gated)  
   - `ApplyProfile { … }`  
3. **Local CLI remains authoritative for v1** — CP transport may be compiled but dormant.  
4. **Auth assumption:** future channel is mutually authenticated (mTLS or equivalent); client stores trust anchors in install config; **no** “trust any HTTPS.”  
5. **Reverse proof:** CP may supply either (a) operator-approved capability token, or (b) instruct local password entry — client auth module stays abstract (`ReverseProof`).  
6. **Allowlist while isolated:** profiles may permit only configured CP endpoints + DNS as needed — so remote reverse remains possible.  
7. **Server non-goals remain absolute:** no server binary, no multi-tenant policy engine, no cloud account system in this repo.

## Consequences

- Client can ship value offline.  
- Server team (later) designs against a published contract.  
- Risk: over-specifying the contract too early — keep messages minimal until first server spike.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Ignore CP until server exists | Likely forces breaking CLI redesign |
| Embed full agent framework now | Scope explosion |
| Shell-out RPC only | Weak typing and auth story |

## Open questions

- Push (client dials out) vs pull (CP connects) — prefer client dial-out for NAT, confirm later.  
- Whether reverse via CP can bypass local password entirely in enterprise mode (major policy decision).
