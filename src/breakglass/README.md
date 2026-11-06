# `src/breakglass` (scaffold)

Future **sister tooling** for tenant-scoped recovery when the reverse password is lost or unsuitable.

- Verifies/signs `BreakGlassProof` bound to `tenant_id` (+ recommended `install_id`)
- Must **fail closed** cross-tenant
- Prefer challenge–response; private keys stay in vault/HSM — not on endpoints
- See [docs/adr/0008-break-glass-and-auth-modes.md](../../docs/adr/0008-break-glass-and-auth-modes.md)

No product code in this planning phase.
