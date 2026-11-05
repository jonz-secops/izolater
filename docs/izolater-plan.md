# izolater — user-facing plan

**Status:** planning complete for review · **Code:** none yet (Zig client deferred until validation gates pass)  
**Repo:** [jonz-secops/izolater](https://github.com/jonz-secops/izolater)

## 1. High-level overview

**izolater** is a multiplatform **Zig** client binary that **isolates an endpoint** (network containment) **quickly**, then **reverses** that isolation only when a **password** is presented. The password is set at **install time** and can also be supplied by automation or a future control plane. The client is **local-first** and **control-plane-ready**; the **izolater server** is a **separate project** and out of scope here.

Design pillars: reversible by design · password-gated reverse · secure secret handling · multiplatform backends behind one orchestrator · prove security direction before coding.

Deeper read: [overview.md](overview.md) · [architecture.md](architecture.md)

## 2. Required features (summary)

| Priority | Themes |
| --- | --- |
| **P0** | Install + password verifier · isolate · snapshot · password reverse · status · local audit · Windows backend · CLI |
| **P1** | Linux/macOS backends · profiles · CP allowlist while isolated · rotation · reboot persistence · CP client stubs · uninstall |
| **P2** | Process/disk containment · TPM secrets · signed policies · GUI · self-update |

Full list: [features.md](features.md)

## 3. ADRs (major choices)

| ADR | Topic |
| --- | --- |
| [0001](adr/0001-platform-isolation-mechanisms.md) | Network isolation via OS firewall/pf/nft; snapshot-capable backends |
| [0002](adr/0002-crypto-password-handling.md) | Argon2id verifier; no plaintext; stdin/file/env injection |
| [0003](adr/0003-install-vs-runtime-config.md) | Split install config vs runtime state; ephemeral secrets |
| [0004](adr/0004-control-plane-client-contract.md) | Client command contract; mTLS assumptions; no server in-repo |
| [0005](adr/0005-threat-model-assumptions.md) | Admin malware limits; privilege model; honest failure |
| [0006](adr/0006-reversibility-model.md) | Snapshot-before-mutate; integrity; password gate; no backdoor |
| [0007](adr/0007-multiplatform-strategy.md) | Zig latest; Windows → Linux → macOS |

Index: [adr/README.md](adr/README.md)

## 4. Proposed repo layout

Planning scaffolds under `docs/`, `contracts/`, `src/*`, `tests/*`, `config/`, `scripts/` — see [directory-structure.md](directory-structure.md). No `build.zig` until implementation.

## 5. Security validation

- Threat model sketch: [threat-model.md](threat-model.md)  
- Prove-before-code gates: [validation.md](validation.md)  
- Open risks: admin bypass of host firewalls · env password leakage · platform bypass paths (WSL/VPN/IPv6/…) · lost-password break-glass · future CP as high-value target  

**Do not start product Zig** until ADRs are accepted (or deferred), bypass inventory for OS #1 exists, and owner open questions below are answered.

## 6. Non-goals

**izolater server**, EDR replacement, guaranteed enforcement vs SYSTEM/root malware, mobile/IoT, and (this phase) any product implementation. Full list: [non-goals.md](non-goals.md).

## Decisions needed from skrappy0x4a

1. **Break-glass** if reverse password is lost while isolated — reimage/OOB only, or sealed recovery key design?  
2. Confirm **OS ship order**: Windows → Linux → macOS?  
3. **Password injection**: allow env var in default policy, or stdin/file/CP only?  
4. **Wrong-password throttling** — acceptable IR lockout tradeoff?  
5. Is **network-only** isolation enough for v1 positioning?  
6. May future **CP reverse bypass local password** in enterprise mode?  
7. Accept ADRs 0001–0007 as Proposed → Accepted (or note edits)?

## Doc map

[docs/README.md](README.md) · [contracts/](../contracts/)
