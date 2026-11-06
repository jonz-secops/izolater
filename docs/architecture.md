# Client architecture (planning sketch)

## Goals

Module boundaries before implementation. Scaffolds only under `src/`.

## Logical components

```
                         ┌──────────── CLI ────────────┐
                         │ no open reverse socket API  │
                         └─────────────┬───────────────┘
                                       │
          ┌────────────────────────────┼────────────────────────────┐
          ▼                            ▼                            ▼
    ┌───────────┐               ┌──────────────┐            ┌──────────────┐
    │  config   │               │    auth      │            │ controlplane │
    │ tenant +  │◄─────────────►│ ReverseProof │            │ mTLS stubs   │
    │ modes     │               │ pwd/bg/CP    │            │              │
    └─────┬─────┘               └──────┬───────┘            └──────┬───────┘
          │                            │                           │
          └──────────────┬─────────────┴─────────────┬─────────────┘
                         ▼                           ▼
                  ┌─────────────┐             ┌─────────────┐
                  │  isolation  │◄───────────►│  platform   │
                  │ snapshot +  │             │ win/darwin/ │
                  │ apply/restore│            │ linux       │
                  └──────┬──────┘             └─────────────┘
                         ▼
                  ┌─────────────┐     ┌─────────────────────┐
                  │ local audit │     │ breakglass tooling  │
                  └─────────────┘     │ (sister artifact)   │
                                      └─────────────────────┘
```

## Responsibilities

| Component | Does | Must not |
| --- | --- | --- |
| CLI | Commands, prompts, exit codes | Listen unauthenticated for reverse |
| config | tenant_id, modes, trust material | Store plaintext passwords or BG private keys |
| auth | Verify password / break-glass / CP proofs | Talk to NICs directly |
| isolation | State machine; multi-surface apply | Embed OS syscalls |
| platform | Filters, routes, DNS, Wi‑Fi, NICs | Know secrets |
| controlplane | Future dial-out mTLS | Implement server |
| breakglass | Tenant-scoped sign/respond | Work cross-tenant |
| audit | Append events | Log secrets |

## Isolation surfaces (orchestrated)

Per ADR-0001: snapshot → DNS → routes → packet filter → Wi‑Fi disconnect → disable non-CP NICs → (optional) session drain.

## ReverseProof

See ADR-0008. Auth module accepts password, break-glass signature, or CP capability depending on mode and policy.

## Hostile notes

- Prefer oneshot privileged CLI over a fat localhost daemon.  
- If a service is required for reboot re-apply / watchdog, it must not expose unauthenticated reverse.  
- Apply path should minimize time C2 sockets remain usable (race window).  

## Error model

Distinct exit codes: auth failure, privilege, already-isolated, corrupt state, tenant mismatch, unsupported platform. Reverse auth failures: capped throttle, no permanent lockout.
