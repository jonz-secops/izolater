# Agent brief (izolater)

Read this before changing the repo.

## Product

izolater = multiplatform **Zig client** (not started) for **hostile-endpoint containment** with proof-gated reverse. **izolater server is out of scope** here.

## Branching

- Base all agent work on **`agentic`**.  
- **Do not commit to `master`.**  
- One vertical slice per PR into `agentic`.

## Authority

1. [docs/izolater-plan.md](docs/izolater-plan.md)  
2. [docs/feature-inventory.md](docs/feature-inventory.md) (`INV-*` IDs)  
3. [docs/adr/](docs/adr/) — all **Proposed**, **not accepted** unless owner says so  
4. [docs/multi-agent-dev.md](docs/multi-agent-dev.md)  
5. [docs/non-goals.md](docs/non-goals.md)

If chat conflicts with these docs, **docs win**. Inference provider does not change scope.

## Hard rules

- Planning/docs until owner accepts ADRs or explicitly waives.  
- No unauthenticated reverse socket/API designs.  
- No global unlock key; break-glass blast radius ≤ tenant (or per-install if standalone).  
- Do not disable USB in isolate design.  
- No secrets in examples or commits.

## Before coding a slice

State: owned paths, must-not-touch paths, `INV-*` / ADR refs, acceptance checks.
