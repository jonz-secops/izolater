# ADR-0007: Multiplatform strategy

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Multiplatform Zig (latest) client. Owner confirmed ship order.

## Decision

1. **Language:** Zig **latest stable** at implementation start; pin when coding begins.  
2. **Ship order:** **Windows → macOS → Linux**. Shared CLI/auth/orchestrator first; backends gate on OS.  
3. **Layout:** `src/platform/{windows,darwin,linux}` behind `src/isolation`.  
4. **CI (future):** privileged matrix; Wi‑Fi/NIC tests may need hardware or suitably capable VMs.  
5. **Parity policy:** same commands; OS limitations visible in `status` / docs; no silent no-ops.  
6. **Linking:** predictable Zig-friendly builds; no heavy helper runtimes on the mutate path.  
7. **Sister break-glass tooling** follows the same OS order for packaging, but crypto/tenant binding is shared.

## Consequences

- Early releases may be Windows-only for isolation backends.  
- macOS Wi‑Fi/DNS APIs need early spikes after Windows.  
- Linux third — servers still covered, just later.

## Alternatives considered

| Alternative | Why not |
| --- | --- |
| Windows → Linux → macOS | Owner reordered to put macOS second |
| All three at once | Quality risk |

## Open questions

- Minimum OS versions for each platform.  
- Whether Linux server images without NetworkManager need a distinct DNS/interface strategy.
