# izolater documentation

Planning and design docs for the **izolater client** — a multiplatform Zig binary that isolates an endpoint and reverses isolation only with a password.

| Document | Purpose |
| --- | --- |
| [izolater-plan.md](izolater-plan.md) | Concise user-facing plan (start here) |
| [overview.md](overview.md) | Product & technical overview |
| [features.md](features.md) | Prioritized features and functionality |
| [architecture.md](architecture.md) | Client architecture sketch |
| [threat-model.md](threat-model.md) | Threat model sketch |
| [validation.md](validation.md) | What must be proven before coding |
| [non-goals.md](non-goals.md) | Explicit out-of-scope |
| [directory-structure.md](directory-structure.md) | Proposed repo layout |
| [adr/](adr/) | Architectural Decision Records |

**Status:** planning only. No product Zig implementation yet.

**Related (out of repo):** izolater server / control plane is a separate project. Client-side contracts and assumptions live under [`../contracts/`](../contracts/).
