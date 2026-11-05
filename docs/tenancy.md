# Tenancy identity pattern

Planning-only. No runtime code. Used by install config, break-glass binding, and future CP.

## Enrollment kinds

| Kind | When | Break-glass binding |
| --- | --- | --- |
| `standalone` | No tenant / single host or unmanaged install | **Per-install** sister only (`install_id`) |
| `tenant` | Fleet / org managed | **Per-tenant** *or* **per-tenant + per-host** (highest security) |

## Identifiers

| Field | Type | Rules |
| --- | --- | --- |
| `tenant_id` | UUID v4 string | Canonical lowercase + hyphens (`xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`). Stable forever for that tenant. Never reuse across orgs. |
| `tenant_name` | slug | Human label: `^[a-z0-9]+(-[a-z0-9]+)*$`, length 3–63. **Not** a security boundary — display / ops only. Renamable. |
| `install_id` | UUID v4 string | Unique per endpoint install. Always present. |
| `break_glass_scope` | enum | `standalone` installs: always `install`. Tenant installs: `tenant` **or** `tenant_and_host`. |

### Security boundary

Cryptographic checks bind to **`tenant_id`** and, when scope is `tenant_and_host` or standalone, also **`install_id`**. Never authorize on `tenant_name` alone.

## Fictitious examples (safe for docs/tests)

These are **fake**. Do not treat as real customers.

### Tenant A — Northwind Security

```toml
enrollment = "tenant"
tenant_id = "a1b2c3d4-e5f6-4789-a012-3456789abcde"
tenant_name = "northwind-security"
# break_glass_scope chosen per policy:
# break_glass_scope = "tenant"            # one sister covers all Northwind hosts
# break_glass_scope = "tenant_and_host"   # sister proof must also match install_id
```

### Tenant B — Contoso IR Lab

```toml
enrollment = "tenant"
tenant_id = "f00dcafe-0001-4000-8000-00000000beef"
tenant_name = "contoso-ir-lab"
break_glass_scope = "tenant_and_host"
```

### Standalone install (no tenant)

```toml
enrollment = "standalone"
# tenant_id omitted or null
install_id = "11111111-2222-4333-8444-555555555555"
break_glass_scope = "install"
```

Machine-readable copy: [`../config/tenants.example.toml`](../config/tenants.example.toml).

## CP / ops notes

- Pre-server: operator supplies `tenant_id` + `tenant_name` at install (or generates UUID).  
- Future CP is source of truth for tenant directory; clients keep a local copy of `tenant_id` + break-glass public trust.  
- Cross-tenant proofs **fail closed**.
