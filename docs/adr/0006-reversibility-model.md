# ADR-0006: Reversibility and snapshot integrity

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Guarantee that every 0001 mutation is **undoable** to the pre-isolate posture, and that undo is **integrity-checked** and **authorization-gated**.
- **Owns:** Snapshot-before-mutate, restore semantics, integrity key separation, rollback/degraded behavior, reboot re-apply *requirement*.
- **Defers:** What gets mutated → [0001](0001-platform-isolation-mechanisms.md); who may reverse → [0008](0008-break-glass-and-auth-modes.md); password KDF → [0002](0002-crypto-password-handling.md).

## Fits in the system

0006 and 0001 are a **pair**: 0001 without 0006 strands the host; 0006 without 0001 is an empty restore machine. Authorization is *not* defined here—0008 supplies `ReverseProof`. Integrity keys must survive CP password clear (0002/0004), so they are **not** derived from the user password.

## Context

izolater’s hard profile touches many OS surfaces. “Delete all firewall rules on reverse” would destroy org baselines. Lost-password recovery must not require a hidden vendor backdoor (0008).

## Decision

1. **Snapshot-before-mutate.** If snapshot write or integrity seal fails → **do not isolate**.  
2. **Snapshot contents** cover every 0001 surface that will change (filters, routes, DNS, Wi‑Fi/NIC/optional radio state, metadata: schema, platform, profile, timestamps).  
3. **Integrity:** MAC/sign with an **install-scoped key** stored in install config (0003), independent of the quarantine password. Tamper → refuse reverse.  
4. **Gate:** restore runs only after 0008 accepts a proof (or admin-alone when mode allows).  
5. **Restore = pre-isolate baseline**, not “enable everything.” Warn if drift detected.  
6. **TCP sessions are not restored**—only network posture; killed connections stay dead (intentional).  
7. **Idempotent isolate:** keep the original pre-isolate snapshot by default (don’t rebase on every re-isolate).  
8. **Reboot:** if runtime state says Isolated, re-apply hard profile after boot (mechanism per OS—spike under 0007).  
9. **Partial apply:** attempt rollback from snapshot; failure → loud degraded state.  
10. **Break-glass is not a backdoor in this ADR**—it’s an authorized proof type defined in 0008.

## Consequences

- Richer, OS-specific snapshot format.  
- CP can clear passwords without orphaning snapshot integrity (key separation).  
- Drift watchdog (optional later) complements but does not replace this model.

## Alternatives rejected

| Alternative | Why rejected |
| --- | --- |
| Flush-all on reverse | Destroys pre-existing org network policy |
| Password-derived-only integrity | Breaks when CP resets/clears password |
| Isolate without snapshot | Irreversible outages |

## Open

- Retention: one active + one backup (lean yes) vs longer history.  
- Reboot mechanism choice per OS (task/service/launchd/systemd) without exposing reverse APIs (0005).
