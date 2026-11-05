# Client architecture (planning sketch)

## Goals of this sketch

Define module boundaries for the Zig client **before** implementation. No runtime code lives here yet; directories under `src/` are scaffolds only.

## Logical components

```
                    ┌──────────── CLI ────────────┐
                    │ parse args, exit codes, I/O │
                    └─────────────┬───────────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         ▼                        ▼                        ▼
   ┌───────────┐          ┌──────────────┐         ┌──────────────┐
   │  config   │          │    auth      │         │ controlplane │
   │ install + │◄────────►│  password    │         │ client hooks │
   │ runtime   │          │  verify/KDF  │         │ (future)     │
   └─────┬─────┘          └──────┬───────┘         └──────┬───────┘
         │                       │                        │
         └───────────┬───────────┴────────────┬───────────┘
                     ▼                        ▼
              ┌─────────────┐          ┌─────────────┐
              │  isolation  │◄────────►│  platform   │
              │ orchestrator│          │  backends   │
              │ snapshot +  │          │ win/linux/  │
              │ apply/restore│         │ mac         │
              └─────────────┘          └─────────────┘
                     │
                     ▼
              ┌─────────────┐
              │ local audit │
              └─────────────┘
```

## Component responsibilities

| Component | Responsibility | Must not |
| --- | --- | --- |
| CLI | UX, flags, prompting | Own crypto or firewall APIs directly |
| config | Paths, policy refs, install metadata | Store plaintext passwords |
| auth | KDF verify, rotation, injection channels | Talk to network stack |
| isolation | Snapshot → apply → restore state machine | Embed OS-specific syscalls |
| platform | OS firewall / net APIs | Know passwords |
| controlplane | Transport + command contract stubs | Implement server logic |
| audit | Local event append | Block reverse on log failure without policy |

## Isolation state machine (conceptual)

```
  NotInstalled → Installed → Ready
                                │
                     isolate()  ▼
                            Isolated
                     reverse()  │
                                ▼
                              Ready
```

Transitions require:

- **install** — privileges + password establishment  
- **isolate** — privileges + writable snapshot slot + healthy backend  
- **reverse** — privileges + valid password + intact snapshot  

## Config layers

See ADR-0003. Summary:

1. **Install-time** — verifier, install id, default profile, optional CP endpoints  
2. **Runtime** — current isolation state pointer, last action, active profile  
3. **Ephemeral** — password from env/stdin for one command (never written)

## Control-plane seam

The client exposes a **command contract** (isolate, reverse, status, rotate-secret, apply-profile) and a **transport assumption** (authenticated channel). Implementation of the server is out of scope; see `contracts/` and ADR-0004.

## Error model (planning)

- Distinct exit codes: auth failure, privilege, already-isolated, corrupt state, unsupported platform.  
- Reverse auth failure must be indistinguishable in timing as far as practical (constant-time verify).  
- Partial isolate failure → attempt rollback using snapshot; if rollback fails → loud degraded state.

## Testing architecture (pre-code)

- Platform backends behind a narrow interface → mockable for unit tests.  
- Integration tests on real VMs per OS (see [validation.md](validation.md)).  
- Security tests: password guess resistance locally, snapshot integrity, privilege drop checks.
