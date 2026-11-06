# ADR-0008: Reverse authorization (modes, proofs, sister scopes)

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Answer **who may unisolate** on a hostile host without inventing a global unlock—and without stranding IR when the password is lost.
- **Owns:** Auth modes, `ReverseProof` kinds, break-glass scopes, admin-alone-when-password-off, audit distinctions for reverse paths.
- **Defers:** Password storage/KDF/injection → [0002](0002-crypto-password-handling.md); CP transport → [0004](0004-control-plane-client-contract.md); restore mechanics → [0006](0006-reversibility-model.md); IDs → [tenancy.md](../tenancy.md).

## Fits in the system

0008 sits **in front of** 0006 restore. 0002 supplies one proof type (password). 0004 supplies another (CP capability). Sister tooling supplies break-glass. 0005 forbids turning izolater into an unauthenticated unisolate API. 0003 persists mode, scope, and public trust—not private keys.

## Context

Owner wants: password optional/required modes; if password is **off**, **admin alone may reverse**; tenant sister scopes `tenant` or `tenant_and_host`; standalone → per-install sister; max blast radius = tenant.

## Decision

### Auth modes

| Mode | Who may reverse locally | Hostile note |
| --- | --- | --- |
| `password_required` (default for hostile templates) | Valid password proof **or** break-glass **or** CP proof | Strong default |
| `password_optional` | Admin alone, **or** password if set, **or** BG/CP | Weaker; document |
| `password_disabled` | **Admin alone** (no verifier) | Weakest vs admin malware; document |

### ReverseProof (OR composition — recommendation)

Accepted proof kinds (any one unless a future dual-control policy says otherwise):

1. **PasswordProof** — 0002 verify  
2. **BreakGlassProof** — signed capability bound per scope below  
3. **ControlPlaneProof** — 0004 capability  

**Recommended default: OR** (password ∨ break-glass ∨ CP). AND would defeat lost-password recovery. Optional later: `break_glass_requires_password` for high-assurance tenants. **Owner has not accepted OR yet**—treat as recommendation.

### Break-glass scopes

| Enrollment | `break_glass_scope` | Binding |
| --- | --- | --- |
| `tenant` | `tenant` | `tenant_id` only — one sister covers the tenant |
| `tenant` | `tenant_and_host` | `tenant_id` **and** `install_id` — highest security |
| `standalone` | `install` | `install_id` only |

Cross-tenant or wrong host binding → **fail closed**. **No global vendor key.**

### Sister tooling principles

- Separate artifact; private key in vault/HSM—**never on the endpoint**.  
- Endpoint holds public trust only (0003).  
- Prefer challenge–response, short-lived, single-use proofs.  
- Offline USB path should work when CP is down (USB not disabled—0001).

### Audit

Distinct high-severity events: password reverse · break-glass · CP · **admin-alone**.

## Consequences

- Product can offer convenience modes without lying about hostile strength.  
- Tenancy becomes mandatory thinking even before the server exists.  
- `tenant_and_host` costs more ops (per-host material).

## Alternatives rejected

| Alternative | Why rejected |
| --- | --- |
| Global unlock | Violates tenant blast-radius ceiling |
| Reimage-only recovery | Too harsh for IR; owner asked for sister path |
| Same binary + hidden flag | Malware-reuse and unclear trust |

## Open

- Accept OR default?  
- Offline sister required for GA? (recommend yes)  
- Pre-CP issuance process for `tenant_and_host` certs.
