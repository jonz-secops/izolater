# Feature inventory (major + supporting)

**Purpose:** Explicit catalog of product capabilities.  
**Relationship:** [features.md](features.md) = prioritized delivery backlog (thin). **This file** = exhaustive inventory derived from plan + ADRs.  
**ADR status:** all **Proposed** — inventory tracks intent, not accepted law.  
**IDs:** `INV-<AREA>-<NN>` stable; map to `F-xx` where a delivery ticket already exists.

Legend: **M** = major capability · **S** = supporting · **P** = priority theme (0/1/2) · **ADR** = primary decision record(s)

---

## 1. Install, identity, configuration

| ID | Kind | Capability | P | ADR / docs | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-CFG-01 | M | Install bootstrap (dirs, ACLs, schema version) | 0 | 0003 | F-01 |
| INV-CFG-02 | M | Enrollment modes: `standalone` \| `tenant` | 0 | 0003, tenancy | F-01 |
| INV-CFG-03 | M | `install_id` (UUID) always present | 0 | tenancy, 0003 | F-01 |
| INV-CFG-04 | M | `tenant_id` + `tenant_name` when tenant | 0 | tenancy | F-02 |
| INV-CFG-05 | S | Install vs runtime config split | 0 | 0003 | |
| INV-CFG-06 | S | Ephemeral secrets never persisted | 0 | 0002, 0003 | |
| INV-CFG-07 | S | Example non-secret configs in repo | 1 | config/ | |
| INV-CFG-08 | S | Config schema version / migrations story | 1 | 0003 | |
| INV-CFG-09 | S | Default install paths per OS | 1 | 0003 open Q | |
| INV-CFG-10 | M | Feature flags store (tenant default, install override) | 1 | 0001, 0003 | F-14 |

## 2. Authentication & reverse authorization

| ID | Kind | Capability | P | ADR | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-AUTH-01 | M | Auth modes: `password_required` \| `optional` \| `disabled` | 0 | 0008 | F-08 |
| INV-AUTH-02 | M | Argon2id verifier + salt + params at rest | 0 | 0002 | F-03 |
| INV-AUTH-03 | M | ReverseProof abstraction (password / break-glass / CP) | 0 | 0008, 0006 | |
| INV-AUTH-04 | M | Admin-alone reverse when password off | 0 | 0008 | F-08 |
| INV-AUTH-05 | M | Password injection: stdin ≻ file ≻ TTY; env opt-in | 0 | 0002 | |
| INV-AUTH-06 | S | No argv password recommendation | 0 | 0002 | |
| INV-AUTH-07 | S | Memory zeroization best-effort | 1 | 0002 | |
| INV-AUTH-08 | M | Wrong-password progressive throttle (no permanent brick) | 0 | 0002 | F-18 |
| INV-AUTH-09 | M | Local password rotation (old+new) | 1 | 0002 | F-21 area |
| INV-AUTH-10 | M | CP reset password without old password | 1 | 0004 | F-16 |
| INV-AUTH-11 | M | CP SetAuthMode (incl. remove password requirement) | 1 | 0004 | F-16 |
| INV-AUTH-12 | S | Constant-time verify | 0 | 0002 | |
| INV-AUTH-13 | S | Break-glass OR vs AND policy flag | ? | 0008 | **Owner pending** (rec: OR) |
| INV-AUTH-14 | S | Password length/complexity policy | 2 | 0002 open Q | |

## 3. Break-glass sister tooling

| ID | Kind | Capability | P | ADR | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-BG-01 | M | Sister break-glass tooling (separate artifact) | 1 | 0008 | F-13 |
| INV-BG-02 | M | Scope `tenant` | 1 | 0008, tenancy | |
| INV-BG-03 | M | Scope `tenant_and_host` (tenant_id + install_id) | 1 | 0008, tenancy | highest security |
| INV-BG-04 | M | Scope `install` (standalone) | 1 | 0008 | |
| INV-BG-05 | M | Cross-tenant fail closed | 0 | 0008, 0005 | S8/S12 |
| INV-BG-06 | S | Challenge–response proofs; private key off endpoint | 1 | 0008 | recommended |
| INV-BG-07 | S | Short TTL / single-use nonce | 1 | 0008 | |
| INV-BG-08 | S | Offline USB sister workflow | 1 | 0008 | **confirm GA** |
| INV-BG-09 | S | HSM / YubiKey key backend | 2 | 0008 | |
| INV-BG-10 | S | Dual-control break-glass | 2 | 0008 | F-20 area |
| INV-BG-11 | S | Distinct audit for break-glass reverse | 0 | 0008 | |

## 4. Isolation / containment

| ID | Kind | Capability | P | ADR | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-ISO-01 | M | Hard isolate orchestrator (snapshot→apply→restore) | 0 | 0001, 0006 | F-04/F-07 |
| INV-ISO-02 | M | Packet filter deny-most | 0 | 0001 | |
| INV-ISO-03 | M | Route surgery (keep CP path only if configured) | 0 | 0001 | |
| INV-ISO-04 | M | DNS policy / resolver override | 0 | 0001 | |
| INV-ISO-05 | M | Kill established non-CP TCP sessions | 0 | 0001 | F-05 |
| INV-ISO-06 | M | Flush DNS + ARP/neighbor-related caches | 0 | 0001 | F-06 |
| INV-ISO-07 | M | Disconnect / disable Wi‑Fi | 0 | 0001 | |
| INV-ISO-08 | M | Disable non–CP-path NICs | 0 | 0001 | |
| INV-ISO-09 | M | CP allowlist path exception (HTTPS/mTLS endpoints) | 0 | 0001, 0004 | F-04 |
| INV-ISO-10 | M | Feature flag: disable Bluetooth | 1 | 0001 | F-14 |
| INV-ISO-11 | M | Feature flag: disable cellular/WWAN | 1 | 0001 | F-14 |
| INV-ISO-12 | M | USB **not** disabled | 0 | 0001, non-goals | F-15 |
| INV-ISO-13 | S | Idempotent re-isolate | 0 | 0006 | |
| INV-ISO-14 | S | Fail closed if snapshot cannot be taken | 0 | 0006 | |
| INV-ISO-15 | S | Partial-apply rollback | 0 | 0006 | |
| INV-ISO-16 | S | Named isolation profiles | 1 | 0001 | F-22 area |
| INV-ISO-17 | S | `isolate --plan` dry-run | 1 | 0001 suggestion | |
| INV-ISO-18 | S | Bypass inventory / mitigations (WSL, VPN, …) | 0 | 0001, validation | S5 |
| INV-ISO-19 | S | UDP/QUIC explicit handling beyond filter | ? | 0001 open Q | |
| INV-ISO-20 | S | Apply ordering to shrink C2 race window | 1 | 0001 | |

## 5. Reversibility & integrity

| ID | Kind | Capability | P | ADR | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-REV-01 | M | Snapshot-before-mutate | 0 | 0006 | F-07 |
| INV-REV-02 | M | Restore pre-isolate posture (not flush-all) | 0 | 0006 | |
| INV-REV-03 | M | Snapshot integrity MAC/sign (install key ≠ user password) | 0 | 0006, 0002 | S3 |
| INV-REV-04 | M | Refuse reverse on tamper | 0 | 0006 | F-17 |
| INV-REV-05 | S | Drift warning on restore | 1 | 0006 | |
| INV-REV-06 | S | Snapshot retention (1 active + backup suggested) | 1 | 0006 open Q | |
| INV-REV-07 | M | Reboot persistence / re-apply if Isolated | 1 | 0006 | F-17 |
| INV-REV-08 | S | Drift watchdog re-apply/alert | 2 | plan, features | F-19; rec P2 |

## 6. Platforms

| ID | Kind | Capability | P | ADR | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-PLT-01 | M | Shared CLI/auth/orchestrator across OS | 0 | 0007 | |
| INV-PLT-02 | M | Windows isolation backend | 0 | 0007, 0001 | F-10 |
| INV-PLT-03 | M | macOS isolation backend | 1 | 0007 | F-12 |
| INV-PLT-04 | M | Linux isolation backend | 1 | 0007 | F-12 |
| INV-PLT-05 | S | Clear error on unsupported OS | 0 | 0007 | |
| INV-PLT-06 | S | Capability probe at install | 0 | 0003 | F-01 |
| INV-PLT-07 | S | Min OS version matrix | 1 | 0007 open Q | |
| INV-PLT-08 | S | Zig latest stable toolchain pin (when coding) | 0 | 0007 | |

## 7. CLI / UX / observability

| ID | Kind | Capability | P | ADR / arch | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-CLI-01 | M | Commands: install, isolate, reverse, status, version | 0 | architecture | F-09/F-11 area |
| INV-CLI-02 | M | Stable exit codes (auth, privilege, corrupt, …) | 0 | architecture | |
| INV-CLI-03 | M | No unauthenticated listening reverse API | 0 | 0005 | F-11 |
| INV-CLI-04 | S | Human-readable status | 0 | features | F-09 |
| INV-CLI-05 | S | JSON status mode | 1 | features | |
| INV-CLI-06 | M | Local append-only audit log | 0 | 0005, features | F-09 |
| INV-CLI-07 | S | Audit never logs secrets | 0 | 0002 | |
| INV-CLI-08 | S | Privilege preflight / actionable errors | 0 | features | F-09 |
| INV-CLI-09 | M | Uninstall (proof if isolated) | 1 | features | F-23 area |

## 8. Control plane (client only)

| ID | Kind | Capability | P | ADR | Notes / F-map |
| --- | --- | --- | --- | --- | --- |
| INV-CP-01 | M | HTTPS + mTLS dial-out client stubs | 1 | 0004 | F-16; **rec pending accept** |
| INV-CP-02 | M | Transport-agnostic contracts (gRPC later OK) | 1 | 0004 | |
| INV-CP-03 | M | Commands: Isolate, Reverse, Status | 1 | 0004 | |
| INV-CP-04 | M | ResetReversePassword | 1 | 0004 | |
| INV-CP-05 | M | SetAuthMode | 1 | 0004 | |
| INV-CP-06 | S | RotateBreakGlassTrust | 1 | 0004 | |
| INV-CP-07 | S | ApplyProfile | 1 | 0004 | |
| INV-CP-08 | S | Trust anchors in install config | 1 | 0003, 0004 | |
| INV-CP-09 | M | Offline CLI without CP | 0 | 0004, 0005 | |
| INV-CP-10 | — | Server implementation | — | non-goals | **OOS** |

## 9. Hostile-environment / security posture

| ID | Kind | Capability | P | ADR | Notes |
| --- | --- | --- | --- | --- | --- |
| INV-SEC-01 | M | Design for malware / socket C2 / rogue AI | 0 | 0005 | |
| INV-SEC-02 | M | Raise cost vs admin malware; no false “SYSTEM-proof” claim | 0 | 0005 | |
| INV-SEC-03 | S | Adversarial test cases (security tests scaffold) | 1 | validation | |
| INV-SEC-04 | S | Binary/config signature checks | 2 | 0005 suggestion | |
| INV-SEC-05 | S | No global vendor unlock | 0 | 0008, non-goals | |

## 10. Multi-agent / engineering enablement (repo meta)

| ID | Kind | Capability | P | Docs | Notes |
| --- | --- | --- | --- | --- | --- |
| INV-ENG-01 | M | `agentic` integration branch; `master` protected | 0 | README | done |
| INV-ENG-02 | M | Durable docs + ADR set | 0 | docs/ | in progress |
| INV-ENG-03 | M | Exhaustive feature inventory | 0 | this file | **this revision** |
| INV-ENG-04 | M | Multi-agent operating guide | 0 | multi-agent-dev.md | **this revision** |
| INV-ENG-05 | S | Root `AGENTS.md` | 0 | — | **todo** |
| INV-ENG-06 | S | Slice ticket templates | 1 | — | todo |
| INV-ENG-07 | S | Path ownership map | 1 | — | todo |
| INV-ENG-08 | S | Doc link lint in CI | 1 | — | todo |

---

## Coverage verdict

| Question | Answer |
| --- | --- |
| Do we have a feature list? | **Yes, two layers:** thin delivery list ([features.md](features.md)) + **this inventory**. |
| Do ADRs alone list every feature? | **No.** ADRs decide *how*; they imply capabilities but do not enumerate every major/supporting item. Use **this inventory** as the explicit list; ADRs as normative rationale. |
| Gaps still visible | Owner pending: break-glass OR, CP HTTPS accept, drift watchdog P2, offline sister GA, UDP/QUIC, min OS versions, password complexity. |

## How to extend

1. New capability → add `INV-*` row here **and** link ADR (or file ADR if cross-cutting).  
2. When scheduling work → add/adjust `F-xx` in [features.md](features.md).  
3. Agents cite `INV-*` in PRs.
