# Required features and functionality

Priorities: **P0** = must ship for useful v0.1 client; **P1** = strong follow-on; **P2** = deferred / nice-to-have. All items are planning targets — not commitments of current code.

## P0 — Minimum viable client

| ID | Feature | Notes |
| --- | --- | --- |
| F-01 | Install / bootstrap | Create install root, write config, establish reverse-password verifier, record platform capability probe. |
| F-02 | Set reverse password | Interactive prompt and non-interactive supply (env var / file / stdin) for automation and peer binaries. |
| F-03 | Isolate endpoint | Apply network isolation via platform backend; default policy: deny most egress/ingress, allow documented exceptions. |
| F-04 | Capture restore state | Snapshot firewall/rules/state needed for exact reverse before mutation. |
| F-05 | Reverse (unisolate) | Require password; verify; restore snapshot; report success/failure clearly. |
| F-06 | Status | Show isolated vs not, policy id/version, last action time, whether CP is configured (even if unused). |
| F-07 | Privilege handling | Detect insufficient privileges; refuse mutate ops with actionable error. |
| F-08 | Audit log (local) | Append-only local events for install, isolate, reverse attempts (success/fail), config changes. |
| F-09 | Platform: Windows | First target OS for isolation backend (see ADR-0001). |
| F-10 | CLI surface | `install`, `isolate`, `reverse`/`unisolate`, `status`, `version`; stable exit codes. |

## P1 — Hardening and multiplatform depth

| ID | Feature | Notes |
| --- | --- | --- |
| F-11 | Linux isolation backend | nftables/iptables (or equivalent) behind same trait as Windows. |
| F-12 | macOS isolation backend | pf (or documented alternative) behind same trait. |
| F-13 | Password rotation | Rotate verifier with old+new password; audit event. |
| F-14 | Isolation profiles | Named profiles (e.g. hard / IR-allow-CP / allow-local-subnet) selected at isolate time. |
| F-15 | Safe allowlist for control plane | While isolated, optionally keep path to configured CP endpoints only. |
| F-16 | Tamper signals | Detect missing/corrupt restore snapshot or config; refuse silent reverse. |
| F-17 | Crash / reboot persistence | Isolation and restore metadata survive reboot; re-apply if needed. |
| F-18 | Control-plane client stubs | TLS client skeleton, command enum, auth hook — no real server required. |
| F-19 | Uninstall | Reverse if isolated (password), remove artifacts; document residual OS state. |

## P2 — Later

| ID | Feature | Notes |
| --- | --- | --- |
| F-20 | Disk / process containment | Beyond network (e.g. process restrictions) — only if threat model expands. |
| F-21 | Hardware-backed secrets | TPM / Secure Enclave storage for verifier material. |
| F-22 | Signed policy packs | Externally signed isolation profiles. |
| F-23 | GUI / tray | Not required for IR binary; optional later. |
| F-24 | Self-update | Out of band with CP project. |

## Functional requirements (cross-cutting)

1. **Reversibility** — Isolate without a restorable snapshot must fail closed (do not apply).  
2. **Password never stored in plaintext** — only KDF verifier (or OS secret reference). See ADR-0002.  
3. **Automation-friendly** — password and config injectable without TTY.  
4. **Idempotent isolate** — re-running isolate on already-isolated host is safe and logged.  
5. **Fail reverse closed** — wrong password → no state change; rate-limit / backoff locally.  
6. **Observable** — human-readable status + machine-parseable JSON mode (P1).  

## Non-functional targets (planning)

| Area | Target (aspirational until validated) |
| --- | --- |
| Isolate latency | Seconds, not minutes, on a healthy endpoint |
| Binary size | Small static-ish Zig release build |
| Dependencies | Prefer Zig std + audited crypto; minimize FFI |
| Languages | Zig latest stable only for product code (when coding starts) |
| Docs | ADRs + threat model kept current with behavior |

## Traceability

Features map into ADRs:

- Isolation mechanics → ADR-0001, ADR-0006  
- Password / crypto → ADR-0002, ADR-0003  
- Control plane readiness → ADR-0004  
- Threat assumptions → ADR-0005, [threat-model.md](threat-model.md)  
- Multiplatform → ADR-0007  
