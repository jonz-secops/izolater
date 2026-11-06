# Multi-agent development readiness

**Status:** planning guidance · **ADRs:** still Proposed (not accepted)  
**Goal:** make izolater workable for several agents (and several model/providers) **without** requiring an orchestration layer yet.

## Honest assessment (today)

| Area | State | Gap |
| --- | --- | --- |
| Shared product truth | Strong start: plan, ADRs, threat model, tenancy | ADRs not accepted; some open owner decisions |
| Feature inventory | Thematic P0/P1/P2 list | Not yet exhaustive major+supporting catalog (see [feature-inventory.md](feature-inventory.md)) |
| Branch / merge base | `agentic` protects `master` | Need per-slice feature branches off `agentic` |
| Module boundaries | `src/*` scaffolds + architecture sketch | No Zig yet; interfaces not coded |
| Agent operating manual | This doc (new) | No `AGENTS.md` at repo root until you want Cursor/other tools to auto-load it |
| Task packets | Cursor Project task IZO-1 only | Need slice tickets with owner files + acceptance checks |
| Multi-provider prompts | None | Providers differ; repo docs must be the contract, not chat memory |
| Orchestration layer | Explicitly out of scope for now | Correct — don’t wait on one |

**Bottom line:** the repo is *directionally* agent-friendly (docs-first, `agentic` base) but **not yet** set up for parallel multi-agent delivery. The missing pieces are **work slicing**, **ownership**, and an **exhaustive feature catalog**—not more model vendors.

## What “multi-agent friendly” means here (no orchestrator)

You do **not** need LangGraph/Crew/etc. first. You need:

1. **One durable source of truth in git** (this `docs/` tree + later code).  
2. **Small vertical slices** one agent can finish in a PR.  
3. **Non-overlapping file ownership** per slice (or explicit sequential merge).  
4. **Stable IDs** (`F-…`, `ADR-…`, `INV-…`) every agent cites.  
5. **Acceptance checks** copied into the PR body.  
6. **Merge only into `agentic`**, never straight to `master`.

Inference provider (Cursor / Claude / GPT / local / …) becomes a **worker implementation detail**. If two providers disagree, **docs + ADR + inventory win**.

## Recommended operating model (v0)

```
master          ← protected; human promote only
  └── agentic   ← integration base for all agents
        ├── feat/<slice>-<id>     agent A
        ├── feat/<slice>-<id>     agent B
        └── docs/<topic>-<id>     docs-only agents
```

### Rules of engagement

| Rule | Why |
| --- | --- |
| Branch from latest `agentic` | Avoid divergent “truth” |
| One slice per PR | Reviewable; less cross-agent thrash |
| Touch only owned paths | Parallelism without merge hell |
| Update inventory/ADR when behavior changes | Providers don’t share chat history |
| No product Zig until ADR acceptance wave (or explicit waiver) | Prevents thrashing implementation |
| Secrets never in prompts or example configs | Hostile + multi-agent leak risk |

### Slice shape (copy into every agent assignment)

```markdown
## Slice: <INV-id or F-id> — <title>
Base: agentic
Owns: <paths>
Must not touch: <paths>
Reads: docs/izolater-plan.md, ADR-####, feature-inventory.md rows …
Done when:
- [ ] … acceptance checks …
Out of scope: …
Provider: <optional; irrelevant to contract>
```

## Parallelism map (planning → early impl)

Safe to parallelize **now** (docs/contracts only):

| Lane | Owner paths | Depends on |
| --- | --- | --- |
| A — Isolation semantics | `docs/adr/0001*`, inventory isolate rows | Owner decisions on UDP/QUIC if any |
| B — Auth / break-glass | `docs/adr/0002*`, `0008*`, `docs/tenancy.md` | OR-default accept |
| C — CP contract | `contracts/`, `docs/adr/0004*` | HTTPS+mTLS accept |
| D — Validation / threat | `docs/validation.md`, `docs/threat-model.md` | — |
| E — Multi-agent hygiene | this doc, inventory, AGENTS.md | — |

**Not** parallel until interfaces exist: Windows backend + macOS backend coding the same `src/isolation` types without a frozen trait.

## Multi-provider tips (without orchestration)

1. **Put the assignment in the repo or PR**, not only in a vendor chat.  
2. Prefer **short system preamble**: “Follow `docs/izolater-plan.md`; ADRs Proposed; do not invent server scope.”  
3. Ask each provider to **list files it will touch** before editing.  
4. If two agents must share a file, **serialize** (merge A before B starts).  
5. Use **inventory IDs** in commit messages (`INV-ISO-TCP`, `INV-AUTH-MODE`).  
6. Treat provider-specific “memory” as disposable.

## Still missing (backlog to become truly ready)

| Item | Purpose | Priority |
| --- | --- | --- |
| Root `AGENTS.md` | Auto-discovered agent brief for Cursor and others | P0 for multi-agent |
| Exhaustive [feature-inventory.md](feature-inventory.md) | Every major + supporting capability | P0 (added) |
| Slice ticket template under `docs/agent-slices/` | Paste-ready assignments | P1 |
| CODEOWNERS-style path map | Human + agent ownership | P1 |
| Doc link CI (lychee/markdown lint) | Catch broken cross-links | P1 |
| Frozen Zig module interfaces (when coding starts) | True parallel backends | P0 at impl time |
| Optional later: orchestration | Only after slices + CI hurt | P2 |

## What not to do yet

- Don’t build a multi-agent orchestrator for izolater itself.  
- Don’t encode provider-specific APIs into product design.  
- Don’t let agents accept ADRs or merge to `master`.  
- Don’t start Zig until the inventory + open decisions are tight enough (or you explicitly waive).

## Related

- [izolater-plan.md](izolater-plan.md)  
- [feature-inventory.md](feature-inventory.md)  
- [directory-structure.md](directory-structure.md)  
- Branch policy in repo `README.md`
