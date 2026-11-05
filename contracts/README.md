# Client ↔ control-plane contracts

This directory holds **client-side assumptions and command contracts** for a future **izolater server**.

- **In scope:** versioned command names, auth expectations, allowlist semantics while isolated, error shapes.  
- **Out of scope:** server implementation, fleet policy UI, multi-tenant cloud.

Until schemas are formalized, treat [docs/adr/0004-control-plane-client-contract.md](../docs/adr/0004-control-plane-client-contract.md) as the normative draft.

## Draft command list (indicative)

| Command | Direction | Notes |
| --- | --- | --- |
| `Isolate` | CP → client | Includes profile id + request id |
| `Reverse` | CP → client | Carries `ReverseProof` (capability or password-driven flow) |
| `Status` | CP ↔ client | Isolation state, versions, health |
| `RotateReverseSecret` | CP → client | Highly sensitive; may remain local-only at first |
| `ApplyProfile` | CP → client | Swap isolation profile while isolated |

Transport assumption: mutually authenticated channel (mTLS or equivalent); client dial-out preferred for NAT.

## Versioning

When the first schema file lands, use `contracts/vN/` and never break `vN` in place — add `vN+1`.
