# Source (scaffold)

Future Zig modules for the izolater **client** and related **break-glass** tooling.

| Subdir | Role |
| --- | --- |
| `cli/` | Commands; no unauthenticated reverse listener |
| `auth/` | ReverseProof: password / break-glass / CP |
| `isolation/` | Multi-surface snapshot/apply/restore |
| `platform/` | windows, darwin, linux backends |
| `controlplane/` | mTLS client stubs |
| `breakglass/` | Tenant sister tooling |

**No product `.zig` files in this planning phase.**
