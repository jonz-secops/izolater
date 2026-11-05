# ADR-0002: Crypto and password handling

- Status: Proposed
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Reversal of isolation **requires a password**. The password is set at **install time** and must also be supplyable by **another app** (automation, future cloud service, helper binary). Storing or handling this secret incorrectly would let attackers unisolate or recover the password offline.

## Decision

1. **Never store plaintext passwords** on disk. Store a **password verifier** produced by a memory-hard KDF.  
2. **KDF:** **Argon2id** (preferred) with parameters selected for interactive-ish verify on IR hardware; parameters recorded beside the verifier for future bumps. If Zig std/lib constraints force a temporary substitute during early spikes, document it — production path remains Argon2id.  
3. **Salt:** unique per install, cryptographically random, stored with verifier.  
4. **Verify:** constant-time compare of KDF output; on failure, no isolation state change.  
5. **Injection channels (supported):**  
   - interactive prompt (TTY)  
   - stdin (piped)  
   - file path flag (operator-managed secret file)  
   - environment variable (documented as **weaker**; optional, may be disabled by policy)  
6. **Memory hygiene:** zeroize buffers holding password material after use (best effort in Zig).  
7. **Rotation:** supported via old+new password; writes new verifier; audits event.  
8. **No password recovery from verifier** — lost password ⇒ break-glass policy (owner decision), not “export plaintext.”  
9. **Future CP:** may deliver an **authorization token** that *authorizes reverse* without embedding the human password in every message — design auth module so “proof of reverse right” can be password **or** CP-issued capability later, without weakening local password mode.

## Consequences

- Positive: offline brute-force costly; automation still possible; clear story for auditors.  
- Negative: env-var injection remains a footgun if enabled; operators must be educated.  
- Argon2 parameters need calibration on low-end endpoints so IR isn’t blocked by 10s verifies.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Store encrypted password (reversible) | Increases blast radius if key recoverable locally |
| OS keychain only | Good complement later; poor headless/automation story alone |
| Asymmetric unwrap with local private key | Heavier; still needs protecting the private key |
| Hash with SHA-256 only | Too fast for password offline attack |

## Open questions

- Minimum password length / complexity policy — enforce or recommend?  
- Local attempt throttling vs IR lockout DoS (see threat model).  
- Whether to derive a separate **HMAC key** from the password for snapshot integrity (see ADR-0006) vs independent install key.
