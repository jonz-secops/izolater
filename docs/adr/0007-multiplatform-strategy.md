# ADR-0007: Delivery platforms (Zig, OS order, parity)

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Ship **one** containment product across OSes without forking the security model—**Windows first**, then macOS, then Linux—in Zig.
- **Owns:** Language, OS order, backend layout, parity rules, break-glass packaging order.
- **Defers:** What isolate does → [0001](0001-platform-isolation-mechanisms.md); auth → [0008](0008-break-glass-and-auth-modes.md)/[0002](0002-crypto-password-handling.md).

## Fits in the system

0007 is **how we build and sequence** the actuators in 0001 and the sister tooling in 0008. It does not invent a different threat model per OS—0005 applies everywhere. Gaps surface in `status`, never as silent no-ops.

## Context

Owner confirmed order: Windows → macOS → Linux. Shared CLI/auth/orchestrator from day one; backends gate on OS.

## Decision

1. **Zig latest stable** at implementation start; pin in toolchain docs then.  
2. **Ship order: Windows → macOS → Linux.**  
3. **Layout:** `src/platform/{windows,darwin,linux}` behind `src/isolation` orchestrator.  
4. **Parity:** same commands; unsupported/partial surfaces reported honestly.  
5. **One binary owns mutate path**—no Python/Electron helper for isolate/reverse.  
6. **Sister break-glass** packaging follows the same OS order; crypto/tenant binding shared.  
7. **CI (when coding):** privileged matrix; radio/NIC tests may need capable VMs/hardware.

## Consequences

- Early useful releases may be Windows-backend-only.  
- macOS Wi‑Fi/DNS spikes follow Windows learning.  
- Linux server variants (e.g. no NetworkManager) need explicit DNS/NIC strategy later.

## Alternatives rejected

| Alternative | Why rejected |
| --- | --- |
| Windows → Linux → macOS | Owner reordered |
| Three backends in first slice | Quality risk under hostile bar |
| Per-OS script wrappers for core mutate | Breaks “one binary” ops + security review story |

## Open

- Minimum OS versions.  
- Linux without NetworkManager: resolver/interface strategy.
