# Non-goals and out of scope

Clarity here protects scope creep and keeps security claims honest.

## Explicitly out of scope (this repository)

| Item | Notes |
| --- | --- |
| **izolater server / control plane** | Separate project. No fleet UI, multi-tenant policy engine, or server APIs implemented here. Only client hooks and contracts. |
| **EDR / AV replacement** | izolater contains; it does not detect malware. |
| **Full host lockdown** | Not a kiosk mode, not BitLocker management, not account lockout product. |
| **Guaranteed enforcement vs admin malware** | Userspace network isolation can be disabled by equivalent privilege; we document limits. |
| **Cross-host orchestration** | Fan-out isolate of many endpoints lives with the future server / SOAR. |
| **Legal intercept / lawful monitoring** | Not a monitoring agent. |
| **Mobile / IoT / RTOS** | Desktop/server Windows, Linux, macOS only for planned platforms. |
| **Zig product implementation (this phase)** | Planning artifacts and scaffolds only until validation gates pass. |

## Deferred (may become goals later, not now)

- Process or filesystem containment beyond network  
- TPM-backed secrets  
- Signed remote policy packs  
- GUI  
- Automatic self-update  
- Cloud-hosted break-glass escrow (would touch server project)

## In-scope boundaries (reminders)

- Local CLI isolate / reverse / status / install  
- Password verifier and injection channels  
- Platform isolation backends behind one abstraction  
- Local audit  
- Client-side control-plane **contract** and stubs  

If a proposal needs a running control plane to be useful, it belongs in the server project or is at most a client stub here.
