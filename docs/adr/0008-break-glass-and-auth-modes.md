# ADR-0008: Break-glass sister binary and reverse auth modes

- Status: Proposed (**not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)
- Related: ADR-0002, ADR-0004, ADR-0005 (hostile), ADR-0006

## Context

Operators can lose the reverse (quarantine) password while a host is isolated. A silent vendor backdoor is unacceptable; reimage-only is too harsh for many IR cases. Owner direction:

- Optional **sister binary**, unique to a **tenant** (maximum blast radius) or possibly a particular installation.
- Cryptographically based (certificate / key), ideally composed with **tenant ID** and, when policy requires, the reverse password.
- Explicit install/policy option: **with or without** reverse-password requirement.
- Future control plane may **reset** the reverse password or **remove** the password requirement.

## Decision (proposed model)

### Auth modes (install- or policy-selectable)

| Mode | Local reverse requires | Notes |
| --- | --- | --- |
| `password_required` (default) | Valid reverse password **or** valid break-glass / CP capability | Strong default for containment |
| `password_optional` | Admin privileges alone **or** password if provided | Faster IR; weaker against admin malware — document honestly |
| `password_disabled` | Admin privileges alone (no password verifier maintained) | Equivalent to “no reverse secret”; still audit |

Future CP may transition modes (`password_required` → `password_disabled`) and may **rotate/reset** the verifier without local knowledge of the old password when the CP is mutually authenticated (server project; client contract only here).

### ReverseProof (unified gate)

Reverse succeeds only if **one** accepted proof validates (unless a tenant policy later requires dual control — open question):

1. **PasswordProof** — Argon2id verify (when mode has a verifier and proof is password).  
2. **BreakGlassProof** — signature / cert-based proof bound to `tenant_id` (+ recommended `install_id`).  
3. **ControlPlaneProof** — future capability from izolater server (out of scope to implement; client must accept the shape).

When `password_required` and only admin rights are present → **deny**.

### Sister binary (break-glass artifact)

**Intent:** a separate artifact (`izolater-breakglass` working name) that can authorize reverse when the password is lost, without unlocking *all* izolater deployments worldwide.

**Cryptographic sketch (recommended):**

1. Each **tenant** holds a long-lived **break-glass keypair** (or tenant CA). Private key stays in a sealed ops vault / HSM / offline ceremony — **not** on endpoints.  
2. At **install**, the endpoint stores: `tenant_id`, **tenant break-glass public key** (or CA cert), `install_id`, and auth mode.  
3. Sister binary is **tenant-scoped**: it can load/use that tenant’s private key material (or is built/configured only for that tenant). It **refuses** installs whose `tenant_id` does not match.  
4. Preferred online protocol: **challenge–response** — endpoint emits a nonce + `install_id` + `tenant_id`; sister binary (or offline signer) returns a short-lived signed `BreakGlassProof`. Private key need not be permanently embedded in a copy-pasteable binary if loaded from sealed storage.  
5. Optional binding: proof may also require knowledge of the reverse password (**AND** composition) for high-assurance tenants — policy flag `break_glass_requires_password`. Default suggestion: **OR** (password *or* break-glass), so lost-password recovery works.  
6. **Blast radius ceiling:** tenant. A sister binary / key for Tenant A must not reverse Tenant B. Cross-tenant proofs fail closed.  
7. **Per-install uniqueness (optional stronger mode):** issue an install-scoped cert at enroll; sister tooling signs only for that `install_id`. Heavier ops; offer as tier-2.

**What we explicitly reject:**

- Global vendor master key that unisolates every customer.  
- Plaintext recovery password stored beside the verifier.  
- Sister binary that skips `tenant_id` checks.

### Audit

Every break-glass or CP-driven reverse/mode-change emits a **distinct, high-severity local audit event** (and later CP event) so it cannot be confused with normal password reverse.

## Consequences

- Planning must treat “main binary” and “break-glass tooling” as related products with different trust storage.  
- Tenant identity becomes a first-class install field even before the server exists (can be operator-supplied UUID).  
- `password_optional` / `password_disabled` weaken containment against admin-equivalent malware — product copy must say so.  
- CP password reset / mode change becomes a contract requirement (ADR-0004).

## Alternatives considered

| Alternative | Why weaker |
| --- | --- |
| Reimage only | Too costly for many IR playbooks |
| Symmetric recovery code printed at install | Easy to lose/photograph; still need tenant binding |
| Same binary + hidden flag | Encourages malware reuse; muddies trust story |
| Per-host unique sister binary only | Ops nightmare at fleet scale; tenant-scoped is the blast-radius sweet spot |

## Suggestions (improvements)

1. Prefer **challenge–response** over shipping private keys inside the sister binary PE/Mach-O.  
2. Support **YubiKey / HSM** as the sister-binary key backend for SOC vaults.  
3. Add optional **dual control** (two break-glass officers) as a tenant policy later.  
4. Mint proofs with **short TTL** and single-use nonces to limit replay.  
5. Keep break-glass path **usable offline** (USB to isolated host) because isolation may cut the network except CP — if CP is down, local sister binary still matters.

## Open questions for owner

1. Default composition: break-glass **OR** password (recommended), or **AND**?  
2. When `password_optional` / `password_disabled`, is **local admin enough** to reverse, or must sister/CP still authorize?  
3. Sister artifact: **per-tenant** (recommended default) vs **per-install** as GA requirement?  
4. Must break-glass work **fully offline** (no CP), **CP-mediated only**, or both?  
5. Who generates `tenant_id` pre-server — operator UUID at install?
