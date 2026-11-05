# izolater — user-facing plan

**Status:** planning in review (ADRs **not** accepted) · **Code:** none yet  
**Repo:** [jonz-secops/izolater](https://github.com/jonz-secops/izolater)

## 1. High-level overview

**izolater** is a multiplatform **Zig** client that **quickly contains a hostile endpoint** and **reverses** containment only with a valid **cryptographic proof** (reverse password, tenant break-glass, and/or future control-plane capability).

**Hostile by default:** design for malware, socket-based remote control (RAT/C2), and rogue/compromised automation (including tool-using AI agents)—not lab-clean hosts.

**Containment (from day one):** packet filters, routes, DNS, disconnect Wi‑Fi, disable non-uplink / non–CP-path interfaces. **Sole intentional remote exception:** future control-plane services (HTTP and/or RPC).

**Recovery:** password modes (required / optional / disabled); tenant-scoped **sister break-glass** binary/tooling (max blast radius = tenant); future CP can reset password or remove password requirement. **izolater server** remains a separate project.

Deeper: [overview.md](overview.md) · [architecture.md](architecture.md) · [threat-model.md](threat-model.md)

## 2. Required features (summary)

| Priority | Themes |
| --- | --- |
| **P0** | Install + tenant_id + auth mode · password verifier when required · full hard-isolate plan · snapshot/restore · ReverseProof gate · audit · Windows · hostile-aware CLI (no open reverse socket) |
| **P1** | macOS then Linux backends · CP allowlist path · break-glass sister tooling · reboot persistence / re-apply · tamper detection · CP stubs (`ResetReversePassword`, `SetAuthMode`) · connection drain/kill policy |
| **P2** | Watchdog vs drift · TPM · signed policies · per-install certs · dual-control break-glass |

Full list: [features.md](features.md)

## 3. ADRs (all still **Proposed**)

| ADR | Topic |
| --- | --- |
| [0001](adr/0001-platform-isolation-mechanisms.md) | Hard isolate: filter + routes + DNS + Wi‑Fi + NIC; CP exception |
| [0002](adr/0002-crypto-password-handling.md) | Argon2id; stdin/file ≻ env; capped reverse throttle |
| [0003](adr/0003-install-vs-runtime-config.md) | Install vs runtime; tenant + auth mode fields |
| [0004](adr/0004-control-plane-client-contract.md) | CP commands including password reset / mode change |
| [0005](adr/0005-threat-model-assumptions.md) | Hostile host, malware, sockets, rogue AI |
| [0006](adr/0006-reversibility-model.md) | Rich snapshot; proof-gated reverse |
| [0007](adr/0007-multiplatform-strategy.md) | Windows → **macOS** → Linux |
| [0008](adr/0008-break-glass-and-auth-modes.md) | Sister binary; tenant blast radius; auth modes |

## 4. Repo layout

Scaffolds under `docs/`, `contracts/`, `src/*` (incl. break-glass), `tests/*` — [directory-structure.md](directory-structure.md). No product Zig yet.

## 5. Security validation

- [threat-model.md](threat-model.md) · [validation.md](validation.md)  
- Prove before code: adversarial reverse attempts, snapshot tamper, C2-path cut, tenant mismatch on break-glass, bypass inventory (WSL/VPN/IPv6/…)  
- ADRs remain **Proposed** until you accept them

## 6. Non-goals

izolater **server** implementation, EDR replacement, guaranteed enforcement vs SYSTEM/root malware rewriting the stack outside izolater, global vendor unlock, mobile/IoT. Details: [non-goals.md](non-goals.md).

## Owner decisions captured

| Topic | Decision |
| --- | --- |
| OS order | Windows → macOS → Linux |
| Isolation scope | Network + routes + DNS + Wi‑Fi down + non-CP NICs down; CP HTTP/RPC exception |
| Break-glass | Tenant-scoped sister binary / key-cert design; max blast radius = tenant |
| Password requirement | Option: required / not required (modes) |
| Password injection | Safest channels that still allow scripts + future CP (stdin/file/CP; env off by default) |
| Wrong password | Acceptable IR tradeoff: progressive delay, no permanent brick |
| Future CP | Can reset reverse password or remove password requirement |
| ADR accept | **Not accepting 0001–0007 yet** (0008 also Proposed) |
| Environment | Prepare for hostile: malware, rogue AI, socket remote control |

## Clarifying questions (need your input)

1. **Break-glass composition:** password **OR** sister proof (recommended for lost-password), or **AND**?  
2. When mode is `password_optional` / `password_disabled`, can **local admin alone** reverse, or still require sister/CP? (Hostile posture strongly favors still requiring a proof.)  
3. Sister artifact default: **per-tenant** (recommended) vs **per-install** required for GA?  
4. Must break-glass work **fully offline** (USB on isolated host), CP-only, or both? (Recommend both.)  
5. Isolate should **kill/drain existing non-CP TCP sessions**, or only change filter/routes/DNS/interfaces?  
6. **Watchdog service** that re-applies quarantine if malware flips Wi‑Fi/NIC back on — in scope for v1 or P1?  
7. Pre-server `tenant_id`: operator-supplied UUID at install OK?  
8. CP protocol preference to design toward: **HTTPS**, custom **RPC**, or both?  
9. Bluetooth PAN / USB tether: treat like Wi‑Fi (disable) in hard profile?

## Suggestions (improvements)

1. Challenge–response break-glass so tenant private keys never sit on the endpoint.  
2. Short-lived, single-use signed proofs; optional HSM/YubiKey for sister tooling.  
3. No unauthenticated localhost reverse daemon — rogue agents love those.  
4. `isolate --plan` dry-run before mutate.  
5. Independent install MAC key (not derived only from user password) so CP password clear doesn’t break snapshot integrity.  
6. Loud, distinct audit events for break-glass and CP mode/password changes.  
7. Default new installs to `password_required` in any “hostile” template.

## Doc map

[docs/README.md](README.md) · [contracts/](../contracts/)
