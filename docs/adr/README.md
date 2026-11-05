# Architectural Decision Records

ADRs capture **significant, durable choices** for the izolater client. Each record is immutable in spirit: supersede with a new ADR rather than silently rewriting history.

## Index

| ADR | Title | Status |
| --- | --- | --- |
| [0001](0001-platform-isolation-mechanisms.md) | Platform isolation mechanisms | Proposed |
| [0002](0002-crypto-password-handling.md) | Crypto and password handling | Proposed |
| [0003](0003-install-vs-runtime-config.md) | Install vs runtime configuration | Proposed |
| [0004](0004-control-plane-client-contract.md) | Control-plane client contract | Proposed |
| [0005](0005-threat-model-assumptions.md) | Threat model assumptions | Proposed |
| [0006](0006-reversibility-model.md) | Reversibility model | Proposed |
| [0007](0007-multiplatform-strategy.md) | Multiplatform strategy | Proposed |

## Template

```markdown
# ADR-NNNN: Title

- Status: Proposed | Accepted | Superseded by ADR-XXXX
- Date: YYYY-MM-DD
- Deciders: …

## Context
## Decision
## Consequences
## Alternatives considered
## Open questions
```

## When to write an ADR

- Cross-cutting security or platform choice  
- Anything expensive to reverse in code  
- Contract with the future izolater server  
