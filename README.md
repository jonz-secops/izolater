# izolater

Quick, reversible **endpoint containment** client for **hostile** hosts — multiplatform Zig (planned).

Cuts network/routes/DNS/Wi‑Fi/extra NICs; proof-gated reverse (password, tenant break-glass, future CP). Server is a separate project.

## Status

**Planning / design only.** ADRs are **Proposed**, not accepted.

## Branch policy

| Branch | Role |
| --- | --- |
| `master` | Protected; do not land agent experiments here directly |
| `agentic` | Integration base for agentic / Cloud Agent worktrees and follow-on branches |

Start here: **[docs/izolater-plan.md](docs/izolater-plan.md)**

| Doc | Description |
| --- | --- |
| [docs/overview.md](docs/overview.md) | High-level overview |
| [docs/features.md](docs/features.md) | Prioritized features |
| [docs/threat-model.md](docs/threat-model.md) | Hostile threat model |
| [docs/adr/](docs/adr/) | ADRs 0001–0008 |
| [docs/validation.md](docs/validation.md) | Prove-before-code gates |
| [contracts/](contracts/) | Control-plane client contract |

## License

Apache License 2.0 — see [LICENSE](LICENSE).
