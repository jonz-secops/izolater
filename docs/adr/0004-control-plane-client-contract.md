# ADR-0004: Control-plane client seam

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Define the **client-side** contract for a future izolater server so isolation allowlisting and reverse/password powers are designed now—without implementing the server here.
- **Owns:** Transport choice (HTTPS+mTLS primary), dial-out posture, command set, CP as ReverseProof issuer and password/mode authority.
- **Defers:** Server implementation → other project / [non-goals](../non-goals.md); network allowlist mechanics → [0001](0001-platform-isolation-mechanisms.md); proof verification policy → [0008](0008-break-glass-and-auth-modes.md).

## Fits in the system

While isolated, **0001** may leave a hole **only** for CP endpoints named in **0003**. That hole is useless unless **0004** authenticates the peer. CP-issued capabilities are a first-class **0008** proof. Password reset/clear must not break **0006** integrity keys (**0002** separation).

## Context

Owner: CP should eventually reset reverse password or remove password requirement. Transport should be secure and flexible for later SaaS/on-prem orchestration—without custom raw RPC as the only bet.

## Decision

1. **Primary transport: HTTPS (HTTP/2) + mutual TLS**; client **dials out**. Payloads: versioned JSON first; protobuf-over-HTTPS optional.  
2. **`contracts/` is transport-agnostic** so a later gRPC *binding* can reuse the same commands.  
3. **Reject custom raw RPC as primary**—worse auth/framing footguns; poorer enterprise ingress story.  
4. **Command set (client must anticipate):**  
   `Isolate` · `Reverse` · `Status` · `ResetReversePassword` · `SetAuthMode` · `RotateBreakGlassTrust` · `ApplyProfile`  
5. **CP Reverse** satisfies 0008 without also needing the local password (CP proof stands alone under OR recommendation).  
6. **High-impact commands** (`ResetReversePassword`, `SetAuthMode`) require successful mTLS + authorization roles (server-side; client enforces TLS and audits).  
7. **Offline CLI remains valid** with CP dormant (0005).  
8. **No server code in this repository.**

## Consequences

- TLS/trust-anchor work is real P1 client scope.  
- Orchestrators can use ordinary HTTPS tooling.  
- Isolation without reachable CP still needs local password and/or sister path (0008).

## Alternatives rejected

| Alternative | Why rejected |
| --- | --- |
| Ignore CP until server exists | Would force allowlist/auth redesign later |
| CP required for every reverse | Breaks air-gap / CP outage IR |
| gRPC-only from day one | Narrower deploy surface; HTTPS binding covers more first |
| Custom binary RPC only | Security/ops flexibility loss |

## Open

- URL/API versioning shape (`/v1/...`).  
- Role model details for mode/password changes (server project; client placeholders OK).  
- Owner accept of HTTPS+mTLS primary (plan recommendation).
