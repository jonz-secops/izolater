# Client ↔ control-plane contracts

Client-side assumptions only — **no server implementation**.

Normative draft: [docs/adr/0004-control-plane-client-contract.md](../docs/adr/0004-control-plane-client-contract.md).

## Draft commands

| Command | Notes |
| --- | --- |
| `Isolate` | profile + request_id; ensure CP allowlist path |
| `Reverse` | `ReverseProof` (CP capability) |
| `Status` | state, auth mode, tenant_id |
| `ResetReversePassword` | set new verifier without old password (CP-auth) |
| `SetAuthMode` | password_required / optional / disabled |
| `RotateBreakGlassTrust` | update tenant public trust |
| `ApplyProfile` | swap profile while isolated |

Transport: mutually authenticated (mTLS or equivalent); client dial-out preferred. While isolated, **only** these CP endpoints should remain remotely reachable (ADR-0001).

Hostile note: unauthenticated or MITM CP is treated as fleet-scale compromise — pinning/trust anchors are mandatory in the design.
