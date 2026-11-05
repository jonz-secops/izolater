# ADR-0006: Reversibility model

- Status: Proposed
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Isolation that cannot be cleanly reversed is an outage generator. Reversal that ignores integrity is an attacker’s unisolate button. We need a crisp model.

## Decision

1. **Snapshot-before-mutate:** no firewall changes until a **restore snapshot** is successfully written and integrity-protected.  
2. **Snapshot contents:** enough backend-specific state to restore prior posture (exported rules, izolater anchor contents, metadata: schema version, platform, timestamp, profile applied).  
3. **Integrity:** snapshot MAC or signature using a key derived at install (from high-entropy install key and/or KDF material — exact derivation in implementation ADR follow-up). Tamper ⇒ reverse refused.  
4. **Reverse gate:** successful **ReverseProof** (password verify for v1) required before restore.  
5. **Restore semantics:** restore **pre-isolate** posture, not “delete all firewall rules.” Non-izolater changes made during isolation may be tricky — document conflict policy (prefer: restore snapshot baseline; warn if drift detected).  
6. **Idempotency:** isolate while isolated refreshes metadata but does not destroy the original pre-isolate snapshot unless explicitly re-based (default: keep original).  
7. **Persistence:** isolation intent survives reboot; on boot, agent/task re-applies profile if state says Isolated (implementation may be a scheduled task/service — design later, requirement now).  
8. **Break-glass:** if proof unavailable, izolater does not embed a backdoor. Recovery is operational (reimage, offline password reset with physical access policy, or future sealed recovery key if owner approves).

## Consequences

- Isolate path is slightly slower (snapshot first) — acceptable.  
- Need storage for snapshots with tight ACLs.  
- Drift during isolation is an operational edge case to test.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| “Just flush firewall on reverse” | Destroys pre-existing org rules |
| Always-on dual rule tables without snapshot | Harder multiplatform |
| Hidden vendor unlock | Unacceptable trust model |

## Open questions

- Keep N historical snapshots or exactly one? (Lean: one active + one backup.)  
- Service vs oneshot login hook for reboot persistence on each OS.
