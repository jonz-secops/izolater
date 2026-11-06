# ADR-0005: Hostile-by-default threat posture

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Make “hostile endpoint” the **default design context** and set an honest enforcement ceiling every other ADR must respect.
- **Owns:** Adversary classes, security goals/non-goals of protection, product posture language.
- **Defers:** Concrete isolate steps → [0001](0001-platform-isolation-mechanisms.md); reverse authorization → [0008](0008-break-glass-and-auth-modes.md); snapshot integrity → [0006](0006-reversibility-model.md). Detail sketch → [threat-model.md](../threat-model.md).

## Fits in the system

0005 is the **frame**. 0001/0006 implement containment+restore under that frame. 0002/0008/0004 implement *authorized* reverse so izolater is not a convenience unisolate API for malware or rogue agents. 0007 ships the same posture on each OS.

## Context

izolater will run on machines that may already host malware, socket-based RAT/C2, or compromised automation (including tool-using AI). Lab-clean assumptions produce the wrong product.

## Decision

1. **Hostile is default**, not an edge case. Design against: surviving C2 sockets, re-opening egress, calling reverse without secrets, stealing injection channels, corrupting snapshots, killing the tool.  
2. **In-scope adversaries:** remote C2 operators; local malware (user and admin-equivalent); rogue/compromised local agents that can shell out.  
3. **Success for isolate:** the remote controller **loses the channel** (see 0001)—not “a firewall rule was inserted.”  
4. **izolater is not a public RPC service.** Mutate paths = privileged local CLI and/or mutually authenticated CP only. No unauthenticated reverse listener.  
5. **Privilege gate:** isolate/reverse require admin/root; unprivileged callers fail closed.  
6. **Honest ceiling:** SYSTEM/root malware can fight the OS network stack outside izolater. We still deny *clean* reverse without a valid proof (0008), integrity-check snapshots (0006), avoid plaintext secrets (0002), audit loudly, and forbid global unlock keys (0008).  
7. **Weak auth modes** (`password_optional` / `password_disabled`) are allowed as product options but **incompatible with strong hostile claims**—docs must warn (0008).  
8. **Operators** own out-of-band protection of passwords and tenant break-glass keys.  
9. **CP is untrusted until mTLS** (0004); absence of CP is normal.

## We do not assume

Secure boot/ELAM as v1 requirements · env vars as secrets · users never lose passwords · that izolater can stop out-of-band manual undoing of OS network state by admin malware.

## Consequences

- Positioning: **contain and raise cost**, not “immune to SYSTEM.”  
- Validation must include adversarial cases (validation.md).  
- Convenience features that widen local abuse surface are rejected by default.

## Alternatives rejected

| Alternative | Why it fails izolater |
| --- | --- |
| Lab-clean host assumption | Owner requires hostile prep |
| “Guaranteed vs SYSTEM” | Needs stronger root of trust than planned client |
| Localhost reverse daemon for DX | Rogue agents / malware abuse |

## Open

- Reboot re-apply: oneshot+OS task vs long-lived service (feeds 0006/0007 spikes)—service must still expose **no** unauthenticated reverse.
