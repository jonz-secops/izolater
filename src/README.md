# Source (scaffold)

Future Zig modules for the izolater **client** binary.

| Subdir | Role |
| --- | --- |
| `cli/` | Commands, flags, output |
| `auth/` | Password KDF / verify / rotation |
| `isolation/` | Snapshot + apply + restore orchestrator |
| `platform/` | OS backends |
| `controlplane/` | Client stubs honoring `contracts/` |

**No product `.zig` files in this planning phase.** `build.zig` will appear when implementation starts (Zig latest stable).
