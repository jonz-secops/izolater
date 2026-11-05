# izolater

Quick, reversible **endpoint isolation** client — multiplatform Zig binary (planned).

Isolation is **password-gated to reverse**. Password is set at install time and can be supplied by automation or a future control plane. The **izolater server** is a separate project.

## Status

**Planning / design only.** No product implementation yet.

Start here: **[docs/izolater-plan.md](docs/izolater-plan.md)**

| Doc | Description |
| --- | --- |
| [docs/overview.md](docs/overview.md) | High-level overview |
| [docs/features.md](docs/features.md) | Prioritized features |
| [docs/adr/](docs/adr/) | Architectural Decision Records |
| [docs/threat-model.md](docs/threat-model.md) | Threat model sketch |
| [docs/validation.md](docs/validation.md) | Prove-before-code gates |
| [docs/non-goals.md](docs/non-goals.md) | Out of scope |
| [contracts/](contracts/) | Future control-plane client contract |

## License

Apache License 2.0 — see [LICENSE](LICENSE).
