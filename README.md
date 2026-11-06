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

Multi-agent / multi-provider: **[docs/multi-agent-dev.md](docs/multi-agent-dev.md)** · **[AGENTS.md](AGENTS.md)**

Start here: **[docs/izolater-plan.md](docs/izolater-plan.md)**

| Doc | Description |
| --- | --- |
| [docs/overview.md](docs/overview.md) | High-level overview |
| [docs/feature-inventory.md](docs/feature-inventory.md) | Exhaustive major + supporting features |
| [docs/features.md](docs/features.md) | Prioritized delivery backlog |
| [docs/multi-agent-dev.md](docs/multi-agent-dev.md) | Multi-agent readiness (no orchestrator required) |
| [docs/threat-model.md](docs/threat-model.md) | Hostile threat model |
| [docs/adr/](docs/adr/) | ADRs 0001–0008 (Proposed) |
| [docs/validation.md](docs/validation.md) | Prove-before-code gates |
| [contracts/](contracts/) | Control-plane client contract |

## License

Apache License 2.0 — see [LICENSE](LICENSE).
