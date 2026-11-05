# ADR-0005: Threat model assumptions

- Status: Proposed (revised; **not accepted**)
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)
- Related: [threat-model.md](../threat-model.md), ADR-0001, ADR-0008

## Context

izolater is an **incident-containment** tool. It must be designed for a **hostile endpoint**: malware, unauthorized remote control (including socket-based C2 / RAT), and **rogue or compromised automation** (including “rogue AI” agents that can spawn processes, call CLIs, or open sockets). Security claims must stay honest about privilege ceilings while still maximizing attacker cost and minimizing clean abuse of izolater itself.

## Decision — environment we prepare for

1. **Hostile host is the default design context**, not an edge case. Assume active adversaries try to: keep C2 sockets alive, re-open egress, call `izolater reverse`, steal secrets, corrupt snapshots, kill izolater, or socially/automatically abuse install tooling.  
2. **Adversary classes explicitly in scope for design:**  
   - Remote network attackers and socket-based C2 / RAT channels  
   - Local malware (user and admin-equivalent)  
   - Rogue or compromised local agents (scripts, EDR plugins, AI tooling) that can execute commands as some principal  
3. **Isolation goal under hostility:** cut general network paths (filters, routes, DNS, Wi‑Fi, extraneous NICs) so remote control sockets **lose their path**, with the only intentional remote exception being authenticated **CP** endpoints when configured (ADR-0001).  
4. **izolater is not a trustworthy RPC server for the world:** mutate operations are local CLI / future mutually authenticated CP only — no unauthenticated local socket API that malware can casually drive.  
5. **Privilege:** isolate/reverse require administrator / root. Unprivileged callers always fail closed.  
6. **Admin-equivalent adversary ceiling (honest):** a process with SYSTEM/root can often disable host firewalls, re-enable NICs, or patch binaries. We still:  
   - prevent *clean* reverse via izolater without a valid `ReverseProof`  
   - integrity-check snapshots/config  
   - avoid plaintext secrets  
   - make silent abuse noisy via audit  
   - avoid shipping global unlock keys  
7. **Rogue AI / automation:** treat as a local principal with whatever OS rights it has. Password injection must not default to world-readable channels; CP and break-glass proofs must not be forgeable by “just ask the CLI.” Prefer designs where a compromised script **without** the password/tenant key cannot unisolate.  
8. **Operators** protect reverse password and tenant break-glass keys out-of-band.  
9. **Honest failure:** refuse isolate if snapshot/restore cannot be guaranteed; refuse reverse on tamper.  
10. **CP untrusted until mutually authenticated**; absence of CP is normal.  
11. **Auth modes** (`password_required` default) exist because some tenants accept weaker local reverse; hostile-environment deployments should prefer `password_required` + break-glass + CP (ADR-0008).

## Decision — we explicitly do not assume

- Secure boot / measured boot / ELAM as required for v1  
- That packet filters are the only path packets can take (VM switches, etc.)  
- That environment variables are secret  
- That users never lose the password  
- That izolater can stop a determined admin-equivalent attacker from *manually* undoing OS network state outside our tool  

## Consequences

- Product positioning: **contain and raise cost**, not “immune to SYSTEM malware.”  
- Feature work includes anti-abuse of the reverse API, tamper detection, and cutting socket C2 paths — not only “set a firewall rule.”  
- `password_optional` / `password_disabled` are **incompatible with strong hostile-host claims**; docs must warn.  
- Validation must include adversarial scenarios (malware calling reverse, corrupting snapshot, trying to keep a socket alive across isolate).

## Alternatives considered

| Stance | Why rejected |
| --- | --- |
| “Lab-clean hosts only” | Owner requires hostile preparation |
| “Guarantee enforcement vs SYSTEM” | Needs stronger root of trust than planned v1 |
| Open local daemon socket for convenience | Large malware/rogue-agent abuse surface |

## Suggestions

1. No listening reverse API on `0.0.0.0` / unauthenticated localhost by default.  
2. Optional later: authenticated local IPC with explicit allowlist of caller binaries — still hostile-aware.  
3. Consider binary + config integrity checks (signature) before mutate.  
4. Document “race”: isolate should terminate or freeze non-CP sockets where the OS allows (connection drain / block) as part of apply.

## Open questions

- Should v1 attempt to **reset/kill existing TCP sessions** (not only block new packets)?  
- Is a persistently running izolater service required for reboot re-apply in hostile conditions, or is oneshot + OS persistence enough?  
- Formal ADR acceptance still pending.
