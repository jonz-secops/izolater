# izolater — user-facing plan

**Status:** planning in review (ADRs **not** accepted) · **Code:** none yet  
**Repo:** [jonz-secops/izolater](https://github.com/jonz-secops/izolater) · **Agent base branch:** `agentic` (`master` protected)

## 1. High-level overview

**izolater** is a multiplatform **Zig** client that **quickly contains a hostile endpoint** and reverses containment under a clear auth policy: reverse password when required, tenant/install **break-glass**, future **CP**, or **local admin alone** when password mode is off.

**Hostile by default:** malware, socket C2/RAT, rogue/compromised automation (incl. AI agents).

**Hard isolate:** packet filters, routes, DNS, clear DNS/ARP-related caches, **kill existing non-CP TCP sessions**, disconnect Wi‑Fi, disable non–CP-path NICs. Optional (feature-flagged per tenant or install): disable **Bluetooth** and **cellular/WWAN (“smart call”)** radios. **Do not disable USB.** Sole intentional remote exception: control-plane over **HTTPS + mTLS** (see recommendation below).

**Recovery:** auth modes; sister break-glass scoped per rules in [tenancy.md](tenancy.md); CP can reset/remove password requirement. Server is a separate project.

Deeper: [overview.md](overview.md) · [architecture.md](architecture.md) · [threat-model.md](threat-model.md) · [tenancy.md](tenancy.md)

## 2. Required features (summary)

| Priority | Themes |
| --- | --- |
| **P0** | Install (standalone or tenant) · auth modes · hard isolate + TCP kill + DNS/ARP cache clear · snapshot/restore · ReverseProof / admin-if-password-off · audit · Windows · no open reverse socket |
| **P1** | macOS→Linux · CP HTTPS/mTLS stubs · break-glass sister (tenant / tenant+host / install) · BT/cellular feature flags · reboot persistence · tamper detection |
| **P2** | Optional drift watchdog · TPM · dual-control · signed policies |

Full list: [features.md](features.md)

## 3. ADRs (all still **Proposed**)

| ADR | Topic |
| --- | --- |
| [0001](adr/0001-platform-isolation-mechanisms.md) | Hard isolate + TCP kill + cache flush; BT/cellular flags; USB untouched |
| [0002](adr/0002-crypto-password-handling.md) | Argon2id; stdin/file/CP; capped throttle |
| [0003](adr/0003-install-vs-runtime-config.md) | Install vs runtime; tenancy fields |
| [0004](adr/0004-control-plane-client-contract.md) | CP over HTTPS+mTLS; password reset / mode change |
| [0005](adr/0005-threat-model-assumptions.md) | Hostile assumptions |
| [0006](adr/0006-reversibility-model.md) | Rich snapshot; proof-gated reverse |
| [0007](adr/0007-multiplatform-strategy.md) | Windows → macOS → Linux |
| [0008](adr/0008-break-glass-and-auth-modes.md) | Sister scopes + admin-alone when password off |

## 4. Repo layout

See [directory-structure.md](directory-structure.md). Tenancy examples: [tenancy.md](tenancy.md), [`config/tenants.example.toml`](../config/tenants.example.toml).

## 5. Security validation

[threat-model.md](threat-model.md) · [validation.md](validation.md) — ADRs remain Proposed.

## 6. Non-goals

[non-goals.md](non-goals.md) — no server impl; no USB disable; no global unlock.

## Owner decisions captured

| Topic | Decision |
| --- | --- |
| OS order | Windows → macOS → Linux |
| Isolation | Filters + routes + DNS + Wi‑Fi + non-CP NICs; **kill non-CP TCP**; **clear DNS/ARP-related caches**; CP path exception |
| Bluetooth / cellular | **Disable allowed**, but **feature flag** per tenant or per install |
| USB | **Do not disable** |
| Password off | **Local admin alone may reverse** |
| Break-glass scopes | Tenant install: **per-tenant** *or* **per-tenant + per-host**; non-tenant: **per-install sister** |
| Tenancy pattern | UUID `tenant_id` + slug `tenant_name`; fictitious samples in [tenancy.md](tenancy.md) |
| Password injection | stdin/file/CP; env off by default |
| Wrong password | Progressive delay, no permanent brick |
| Future CP | Reset password or remove password requirement |
| ADR accept | Still **not** accepting ADRs |
| Hostile env | Malware, rogue AI, socket remote control |

## Recommendations (where you were unsure)

### Break-glass OR vs AND

**Recommend default: OR** (password **or** break-glass **or** CP proof).  
Reason: AND fails the whole point of lost-password recovery—if the password is gone, AND cannot unisolate. Offer optional `break_glass_requires_password = true` later for dual-control high-assurance tenants. **Pending your explicit accept.**

### Drift watchdog (what it is)

After isolate, malware with admin rights can flip Wi‑Fi/NIC back on *outside* izolater. A **drift watchdog** would be a privileged helper that notices “we’re supposed to be isolated but Wi‑Fi is up again” and **re-applies** quarantine / alerts.  
**Recommend:** not required for first useful release; schedule as **P2** (or late P1) once hard isolate + restore are solid. It improves hostile resilience but adds a service surface we must keep reverse-API-free.

### CP: HTTP vs RPC

**Recommend: HTTPS (HTTP/2) + mTLS as the primary control channel**, with versioned JSON (or protobuf-in-HTTPS) command payloads.  

| Option | Security | Flexibility later |
| --- | --- | --- |
| **HTTPS + mTLS** (recommended) | Mature PKI, load balancers, WAF, standard auditing | Best fit for SaaS, on-prem, gateways, mesh; easy to add async jobs |
| gRPC (HTTP/2) + mTLS | Equally strong if mTLS done right | Great streaming; harder for some enterprise proxies; can be a **later binding** of the same contracts |
| Custom raw RPC | Easy to get auth/framing wrong | Locks orchestration options |

Design `contracts/` **transport-agnostic** so a gRPC binding can appear later without changing Isolate/Reverse semantics. Prefer **client dial-out** to the CP.

## Remaining open questions

1. Accept **OR** as default break-glass composition (recommendation above)?  
2. Accept **drift watchdog = P2**?  
3. Accept **HTTPS + mTLS** as primary CP transport (gRPC optional later)?  
4. Offline break-glass required for GA (USB sister on isolated host) — still recommended **yes**; confirm?  
5. Minimum OS versions per platform?

## Suggestions

1. Challenge–response break-glass; keys in vault/HSM.  
2. `isolate --plan` dry-run.  
3. Order apply to cut egress before slower radio steps; then kill TCP; then flush DNS/ARP caches.  
4. Feature flags: `disable_bluetooth`, `disable_cellular` at tenant default with install override.  
5. Never authorize on `tenant_name` alone.

## Doc map

[docs/README.md](README.md) · [contracts/](../contracts/) · [tenancy.md](tenancy.md)
