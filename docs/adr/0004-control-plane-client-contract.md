# ADR-0004: Control-plane client contract

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Future izolater server orchestrates fleets. Client needs a transport that is **secure** and **flexible** for SaaS, on-prem, and varied orchestration later.

## Decision

1. **Primary transport: HTTPS (HTTP/2) + mutual TLS.** Versioned command payloads (JSON first; protobuf-over-HTTPS optional). Client **dials out** to CP.  
2. **Contracts are transport-agnostic** under `contracts/` so a **gRPC binding can be added later** without changing Isolate/Reverse semantics.  
3. **Why not custom raw RPC as primary:** higher chance of weak auth/framing; poorer fit for load balancers, API gateways, mesh, and enterprise inspection.  
4. **Why HTTPS before gRPC-only:** broadest deployment surface; same mTLS security bar; easiest multi-environment orchestration. gRPC remains a fine *later* option for streaming/high-chatter agents.  
5. **Command set** (indicative): `Isolate`, `Reverse`, `Status`, `ResetReversePassword`, `SetAuthMode`, `RotateBreakGlassTrust`, `ApplyProfile`.  
6. While isolated, **only** configured CP endpoints remain the intentional remote allowlist (ADR-0001).  
7. **No server implementation** in this repo.

## Consequences

- Client TLS stack and trust-anchor story become P1-critical.  
- Orchestrators can use ordinary HTTPS tooling.  
- Password reset / mode change over CP remain high-impact, mTLS-gated, heavily audited.

## Open questions

- Exact URL layout / API versioning (`/v1/...`).  
- CP role model for `SetAuthMode` / password reset.  
- Confirm owner accept of HTTPS+mTLS primary (recommended in plan).
