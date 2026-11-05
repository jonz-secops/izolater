# ADR-0006: Reversibility model

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)
- Related: ADR-0001 (expanded isolation steps), ADR-0008 (proofs)

## Context

Isolation now mutates **filters, routes, DNS, Wi‑Fi, and interface state**. Reverse must restore pre-isolate posture. Reverse authorization uses **ReverseProof** (password / break-glass / CP), not a single hard-coded password path.

## Decision

1. **Snapshot-before-mutate:** no containment changes until a restore snapshot is written and integrity-protected. Snapshot includes all ADR-0001 step categories that will change.  
2. **Integrity:** MAC/sign with an **install-scoped key** independent of the user password (so CP password clear does not invalidate integrity design). Tamper ⇒ refuse reverse.  
3. **Reverse gate:** validate `ReverseProof` per auth mode (ADR-0008); then restore.  
4. **Restore semantics:** restore **pre-isolate** baseline for filters, routes, DNS, Wi‑Fi, interfaces — not “enable everything.” Warn on drift.  
5. **Idempotent isolate:** keep original pre-isolate snapshot by default.  
6. **Persistence across reboot:** re-apply hard profile if state is Isolated (service/task design later).  
7. **Break-glass:** tenant-scoped sister binary / signed proof (ADR-0008) — **not** a global backdoor.  
8. **Failure:** partial apply attempts rollback from snapshot; if rollback fails → loud degraded state + guidance.

## Consequences

- Snapshot format is richer and OS-specific sections grow.  
- Testing matrix expands (Wi‑Fi restore, NIC admin state).  
- Lost password is recoverable via break-glass/CP without reimage (when those are provisioned).

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Flush all network config on reverse | Destroys org baselines |
| Password-only forever | Owner requested break-glass + mode options |

## Open questions

- One active snapshot + one backup, or more history?  
- Reboot persistence mechanism per OS (service vs launchd vs systemd).
