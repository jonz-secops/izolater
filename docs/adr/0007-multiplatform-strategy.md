# ADR-0007: Multiplatform strategy

- Status: Proposed
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

The binary must be **multiplatform Zig (latest)**. IR reality often prioritizes Windows endpoints; Linux servers and macOS exist too. Spreading thin across three backends early risks weak isolation everywhere.

## Decision

1. **Language:** Zig **latest stable** at implementation start; pin version in toolchain docs when coding begins.  
2. **Ship order:** **Windows → Linux → macOS**, unless owner reorders. CLI and auth are shared from day one; isolation backends gate on OS.  
3. **Code layout:** `src/platform/{windows,linux,darwin}` behind a shared interface in `src/isolation`.  
4. **CI (future):** matrix builds per target; integration tests on VMs/runners with privileges.  
5. **Feature parity policy:** shared commands work everywhere; backend-specific limitations surface in `status` and docs rather than silent no-ops. Unsupported OS ⇒ clear error.  
6. **libc / linking:** prefer Zig-friendly static or predictable linking per target; avoid heavy third-party runtimes.  
7. **No electron / .NET / Python helper** for core isolate path — keep one binary responsible for mutate ops (helpers for install packaging OK later).

## Consequences

- Faster learning on one OS; others follow a proven orchestrator.  
- Temporary Windows-only usefulness in early releases — communicate clearly.  
- Darwin/Linux experts needed before claiming GA on those platforms.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| All three backends in first commit | Quality risk |
| Wrapper scripts per OS | Defeats “one binary” ops story |
| Go/Rust rewrite | Counter to stated Zig intent |

## Open questions

- Minimum OS versions (e.g. Windows 10/Server 2019+, specific Linux distros, macOS versions).  
- Whether Windows is always first for *your* deployment mix (confirm).
