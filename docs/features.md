# Required features and functionality

Planning targets only. ADRs Proposed.

> **Thin delivery backlog.** For every major + supporting capability (with ADR traceability), use **[feature-inventory.md](feature-inventory.md)**. ADRs alone do **not** enumerate the full feature set.

## P0

| ID | Feature | Notes |
| --- | --- | --- |
| F-01 | Install | `standalone` or `tenant`; `install_id`; auth mode; ACLs |
| F-02 | Tenancy fields | `tenant_id` / `tenant_name` when tenant; see [tenancy.md](tenancy.md) |
| F-03 | Password when required | Argon2id; stdin/file (env opt-in) |
| F-04 | Hard isolate | Filters, routes, DNS, Wi‑Fi, non-CP NICs |
| F-05 | Kill non-CP TCP | Tear down established sessions outside CP allowlist |
| F-06 | Flush DNS/ARP-related caches | After policy apply |
| F-07 | Snapshot / restore | All mutated surfaces |
| F-08 | Reverse policy | Proof when password on; **admin alone when password off** |
| F-09 | Status / audit / privileges | |
| F-10 | Windows backend | First OS |
| F-11 | No open reverse socket API | Hostile posture |

## P1

| ID | Feature | Notes |
| --- | --- | --- |
| F-12 | macOS / Linux backends | Order: macOS then Linux |
| F-13 | Break-glass sister | Scopes: `tenant` \| `tenant_and_host` \| `install` |
| F-14 | Feature flags | `disable_bluetooth`, `disable_cellular` (tenant default, install override) |
| F-15 | USB | Explicitly **not** disabled by isolate |
| F-16 | CP HTTPS+mTLS stubs | Reset password / SetAuthMode / Reverse |
| F-17 | Tamper detection / reboot persistence | |
| F-18 | Wrong-password throttle | Capped delays |

## P2

| ID | Feature | Notes |
| --- | --- | --- |
| F-19 | Drift watchdog | Re-apply/alert if isolation surfaces undone out-of-band — see plan |
| F-20 | Dual-control / TPM / signed policies | |

## Traceability

ADR-0001 isolation · 0002 crypto · 0003 config · 0004 CP · 0005 hostile · 0006 reverse · 0007 platforms · 0008 break-glass/modes · [tenancy.md](tenancy.md)
