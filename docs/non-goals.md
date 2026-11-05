# Non-goals and out of scope

## Explicitly out of scope (this repository)

| Item | Notes |
| --- | --- |
| **izolater server / control plane impl** | Contracts and client stubs only |
| **EDR / AV replacement** | Containment, not detection |
| **Guaranteed enforcement vs SYSTEM/root** | Hostile design raises cost; cannot stop all out-of-band stack edits |
| **Global vendor unlock key** | Break-glass max blast radius = tenant |
| **Unauthenticated network reverse API** | Would help malware / rogue agents |
| **Full host lockdown / kiosk / BitLocker mgmt** | |
| **Mobile / IoT / RTOS** | Windows, macOS, Linux desktops/servers |
| **Zig product implementation (this phase)** | Planning only until gates pass |

## Deferred

- Process/filesystem containment beyond network stack surfaces  
- TPM-backed secrets, dual-control break-glass  
- GUI, self-update  
- Cloud escrow of break-glass keys (server project)

## In scope reminders

- Hostile-aware hard isolate + proof-gated reverse  
- Tenant sister break-glass design  
- Auth modes; CP contract for password reset / mode change  
- Local audit  
