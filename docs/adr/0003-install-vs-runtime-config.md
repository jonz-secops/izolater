# ADR-0003: Install vs runtime configuration

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Hostile hosts and multi-proof reverse require durable identity (`tenant_id`), auth mode, and CP/break-glass trust material—separate from ephemeral passwords and dynamic isolation state.

## Decision

1. **Install config (relatively static):** schema version, `install_id`, `tenant_id`, auth mode, password verifier (if any), Argon2 params, break-glass public trust (tenant pubkey/CA), CP endpoints + trust anchors, default profile, feature flags, ACL expectations, install-scoped integrity key material (not the user password).  
2. **Runtime state (dynamic):** isolation enum, active profile, snapshot reference, timestamps, reboot-reapply flag, throttle counters (opaque).  
3. **Ephemeral never persisted:** passwords, one-shot CLI secrets, break-glass private keys (those belong in sister tooling / vault—not on the endpoint).  
4. **Install** mandatory before isolate; restrictive ACLs (admin/SYSTEM or root-only) — assume malware will *try* to read these files.  
5. **Password** only when mode requires/allows setting one; CP may later reset verifier or change mode (ADR-0004).  
6. **Schema version** on both stores.  
7. **Automation surface:** document stdin/file/CP; env only if explicitly enabled at install.

## Consequences

- Tenant identity exists even before the server project.  
- Endpoint never holds tenant break-glass private keys.  
- Backups of install config are sensitive (trust anchors, verifiers) but not plaintext passwords.

## Open questions

- Default install paths per OS.  
- Operator-generated vs CP-assigned `tenant_id` pre-server.
