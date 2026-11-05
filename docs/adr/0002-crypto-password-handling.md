# ADR-0002: Crypto and password handling

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)
- Related: ADR-0008 (auth modes / break-glass), ADR-0004 (CP reset)

## Context

Reverse may require a **reverse quarantine password**, depending on auth mode (ADR-0008). The password is set at install when the mode uses one, and must support **scripted install** plus a **future control plane**. Owner ask: choose the **safest injection approach** that still enables automation and CP.

## Decision

1. **Never store plaintext passwords** on disk. Store an **Argon2id** verifier + unique salt + parameter set when a password is configured.  
2. **Modes without a password** (`password_optional` without setting one, or `password_disabled`) store mode flags only — no verifier.  
3. **Verify:** constant-time compare; failure ⇒ no state change; apply throttling policy (below).  
4. **Password injection — security preference order (normative intent):**  

   | Rank | Channel | Use |
   | --- | --- | --- |
   | 1 | **Future CP authenticated channel** | Remote set/reset/reverse authorization |
   | 2 | **Stdin** (piped, no echo) | Scripted install/reverse; preferred local automation |
   | 3 | **Restricted secret file** (`--password-file`, mode 0600 / ACL admin-only) | Installers that cannot hold stdin open |
   | 4 | **Interactive TTY prompt** | Human IR |
   | 5 | **Environment variable** | **Disabled by default**; opt-in only via explicit install flag — process-list / crash-dump exposure |

5. **Do not** recommend embedding passwords on argv (`--password secret`) — visible in process listings; if a flag exists for broken installers, document as insecure and prefer file/stdin.  
6. **Memory hygiene:** zeroize password buffers after use (best effort).  
7. **Rotation / reset:** local rotation with old+new; **CP may reset verifier or clear password requirement** without old password once CP trust is established (client contract; server OOS).  
8. **Wrong-password / IR tradeoff (accepted direction):**  

   - Progressive delay after failures (e.g. ~0s, 1s, 2s, 4s … capped at a modest ceiling such as 15–30s).  
   - **No permanent local lockout** that bricks IR when break-glass/CP is unavailable.  
   - Optional soft threshold that requires break-glass or delay ceiling — never irreversible lock without ADR-0008 path.  
   - All failures audited.  

9. **Break-glass and CP proofs** are not passwords; see ADR-0008. Snapshot integrity keys should be **independent install keys**, not derived solely from the user password (so CP password clear does not weaken snapshot MAC).

## Consequences

- Scripted setup remains practical without defaulting to env vars.  
- CP becomes the best remote secret channel when it exists.  
- Throttling slows online guessing without creating a self-DoS dead end.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Env-var-first automation | Too easy to leak via process/environ |
| Hard lockout after N tries | Unacceptable IR risk if password lost mid-incident |
| Reversible encrypted password at rest | Raises theft blast radius |

## Open questions

- Minimum length / complexity: enforce vs warn?  
- Exact delay curve numbers (tune in spike).  
- Whether `password_optional` still *allows* setting a verifier for “password if you have it.”
