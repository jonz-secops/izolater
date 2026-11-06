# ADR-0002: Quarantine password material

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Handle the **quarantine password** so it supports humans, scripts, and future CP—without becoming malware-readable ambient state or collapsing snapshot integrity when the password is cleared.
- **Owns:** Verifier format (Argon2id), injection channel ranking, throttle/IR tradeoff, separation from install integrity keys.
- **Defers:** Whether a password is required at all → [0008](0008-break-glass-and-auth-modes.md); CP reset command → [0004](0004-control-plane-client-contract.md); where verifier lives on disk → [0003](0003-install-vs-runtime-config.md).

## Fits in the system

0002 implements **one** `ReverseProof` kind. It does not decide *policy* for admin-alone or break-glass—that’s 0008. It deliberately does **not** own snapshot MACs (0006) so CP password reset cannot gut restore integrity.

## Context

Password may be set at install and must be injectable by automation and CP. Owner: safest channels that still enable scripts; wrong-password behavior must not permanently brick IR.

## Decision

1. **No plaintext at rest.** When a password exists: Argon2id verifier + unique salt + recorded parameters.  
2. **No verifier** when mode is password-disabled (or optional with none set)—mode flags only (0008).  
3. **Verify:** constant-time compare; failure → no 0006 restore; throttle below.  
4. **Injection ranking (safest practical first):**

   | Rank | Channel | Role |
   | --- | --- | --- |
   | 1 | CP mTLS channel | Remote set/reset/authorize (0004) |
   | 2 | stdin (no echo) | Preferred local automation |
   | 3 | ACL-restricted password file | Installers without durable stdin |
   | 4 | TTY prompt | Human IR |
   | 5 | Environment variable | **Off by default**; explicit opt-in only |

5. **Avoid argv secrets** (process listing).  
6. **Zeroize** password buffers after use (best effort).  
7. **Local rotate** with old+new; **CP may reset/clear** without old password once trusted (0004)—integrity keys unaffected.  
8. **Throttle:** progressive delay, modest cap (≈15–30s); **no permanent local lockout**; failures audited. Break-glass/CP remain available (0008).  
9. **Install integrity key ≠ password-derived** (feeds 0006).

## Consequences

- Scripts work without teaching env-var password culture.  
- Rogue agents without the secret can’t casually satisfy PasswordProof.  
- IR remains recoverable via 0008 paths if password is lost mid-throttle.

## Alternatives rejected

| Alternative | Why rejected |
| --- | --- |
| Env-first automation | Hostile/rogue-agent readable |
| Hard lockout after N tries | Bricks IR when password lost |
| Reversible encrypted password at rest | Theft blast radius |
| Derive snapshot MAC only from password | Breaks on CP password clear |

## Open

- Enforce vs warn on password complexity.  
- Exact delay curve (tune in spike).  
- Confirm `password_optional` may carry an unused-capable verifier (“password if you have it”).
