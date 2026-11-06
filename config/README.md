# Example configuration (non-secret)

**Rules:** no real passwords/verifiers/customer endpoints in git.

| File | Purpose |
| --- | --- |
| [tenants.example.toml](tenants.example.toml) | Fictitious `tenant_id` / `tenant_name` samples |
| (future) `izolater.example.toml` | Full install config shape |

Tenancy rules: [docs/tenancy.md](../docs/tenancy.md). Install vs runtime: [ADR-0003](../docs/adr/0003-install-vs-runtime-config.md).

```toml
# izolater.example.toml (illustrative)
schema_version = 1
enrollment = "tenant"
tenant_id = "a1b2c3d4-e5f6-4789-a012-3456789abcde"
tenant_name = "northwind-security"
break_glass_scope = "tenant"
auth_mode = "password_required"
default_profile = "hard"
disable_bluetooth = true
disable_cellular = true
# USB is never disabled by izolater
# control_plane_endpoints = ["https://cp.example.invalid"]
```
