# ADR-0008: Break-glass sister binary and reverse auth modes

- Status: Proposed (**not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)
- Related: ADR-0002, ADR-0004, ADR-0005, ADR-0006, [tenancy.md](../tenancy.md)

## Context

Lost reverse password must not imply global backdoor or mandatory reimage. Owner decisions:

- If **password is off** (`password_optional` without using a password, or `password_disabled`): **local admin alone may reverse**.  
- **Tenant installs:** break-glass scope option **`tenant`** or **`tenant_and_host`** (highest security).  
- **Non-tenant (standalone) installs:** **per-install** sister only.  
- OR vs AND for password⊕break-glass: owner unsure — **recommendation recorded below, not accepted**.

## Decision

### Auth modes

| Mode | Local reverse |
| --- | --- |
| `password_required` (default for hostile) | Password **or** break-glass **or** CP proof (OR composition recommended) |
| `password_optional` | Admin alone **or** password if supplied **or** break-glass/CP |
| `password_disabled` | **Admin alone** (no verifier); still audit. Break-glass/CP optional extras |

### Break-glass scopes

| Enrollment | Allowed `break_glass_scope` |
| --- | --- |
| `tenant` | `tenant` — sister/key covers all hosts in `tenant_id` |
| `tenant` | `tenant_and_host` — proof must match `tenant_id` **and** `install_id` |
| `standalone` | `install` — per-install sister only; no cross-host tenant key |

Cross-tenant or wrong `install_id` (when required) → **fail closed**. Max blast radius for tenant keys = that tenant.

### Cryptographic sketch

- Tenant (or install) keypair/CA; **private key never on endpoint**.  
- Endpoint stores public trust + `tenant_id` / `install_id` as applicable.  
- Challenge–response short-lived `BreakGlassProof` preferred.  
- See [tenancy.md](../tenancy.md) for ID pattern and fictitious samples.

### OR vs AND (recommendation — awaiting owner)

**Default OR** so lost-password recovery works. Optional future policy `break_glass_requires_password` for AND/dual-control. Until owner accepts, document both; implementer must not assume AND.

### Audit

Distinct high-severity events for admin-alone reverse, password reverse, break-glass, CP reverse/mode/password changes.

## Consequences

- `password_disabled` + admin reverse is **weak under hostile admin malware** — explicit product warning.  
- Standalone and tenant codepaths both first-class.  
- `tenant_and_host` increases ops cost (per-host enrollment material).

## Open questions

- Owner accept of OR default?  
- Offline USB sister required for GA? (recommended yes)  
- Who operates per-host cert issuance pre-CP?
