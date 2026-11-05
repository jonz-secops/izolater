# ADR-0003: Install vs runtime configuration

- Status: Proposed
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Some settings are established once (password verifier, install identity, default profile, optional CP endpoints). Others change as the host is isolated and reversed (active state, snapshot handles, last action). Mixing them invites accidental secret persistence and upgrade pain.

## Decision

1. **Two persistent stores** (logical; may be two files or sections):  
   - **Install config** — relatively static: install id, verifier+params, default profile name, CP endpoint list, feature flags, ACL expectations.  
   - **Runtime state** — dynamic: isolation enum, active profile, snapshot reference/path, timestamps, reboot-reapply flag.  
2. **Ephemeral inputs** never persisted: password, one-shot flags, CLI overrides for a single command.  
3. **Install command** is mandatory before isolate; creates directories with restrictive ACLs (admin/SYSTEM or root-only).  
4. **Password at install** required; may be provided non-interactively. Changing password later is **rotation**, not a silent rewrite.  
5. **Human-editable config** allowed for non-secret fields (profiles, endpoints); verifier blob is opaque and should not be hand-edited.  
6. **Schema version** field on both stores for migrations.  
7. **Automation variable surface** documented in one place (`config/` examples + man-ish docs later): e.g. `IZOLATER_PASSWORD`, `IZOLATER_CONFIG`, profile name — exact names frozen at implementation start.

## Consequences

- Clear upgrade and backup story (back up install config carefully; runtime state is host-local).  
- Operators can ship install config via management tools without shipping passwords in the file.  
- Need careful Windows ACL and Linux mode bits from day one.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Single mutable config file | Easy to leak secrets into backups; harder migrations |
| Registry-only (Windows) | Weak multiplatform unity; still need files for snapshots |
| No install step (stateless binary) | No place for verifier; contradicts password-at-install |

## Open questions

- Default install paths per OS (`ProgramData` vs `/var/lib` vs `/Library/Application Support`).  
- Whether install config should be signed/HMAC’d with install key.
