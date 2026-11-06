# ADR-0003: Identity stores and config layers

- Status: Proposed (**not accepted**)
- Date: 2026-11-05 · Revised: 2026-11-06
- Deciders: skrappy0x4a (pending)
- **Intent:** Split **who this install is** from **whether it is quarantined now**, and keep secrets that don’t belong on the endpoint off disk.
- **Owns:** Install config vs runtime state vs ephemeral; tenancy/break-glass public fields; feature-flag persistence; ACL posture assumption.
- **Defers:** Auth mode semantics → [0008](0008-break-glass-and-auth-modes.md); verifier format → [0002](0002-crypto-password-handling.md); CP endpoint meaning → [0004](0004-control-plane-client-contract.md); isolate surfaces → [0001](0001-platform-isolation-mechanisms.md).

## Fits in the system

0003 is the **filing cabinet** the other ADRs write into. 0008/0002/0004/0001 decisions become fields here. Private break-glass keys never appear—only public trust—because 0005 assumes malware will read whatever it can.

## Context

Hostile hosts + multi-proof reverse need durable `tenant_id` / `install_id` before a server exists ([tenancy.md](../tenancy.md)).

## Decision

### Three layers

| Layer | Lifetime | Examples |
| --- | --- | --- |
| **Install config** | Stable across isolate cycles | schema version, `enrollment`, `install_id`, optional `tenant_id`/`tenant_name`, `break_glass_scope`, auth mode, verifier+params (if any), BG **public** trust, CP endpoints+trust anchors, default profile, flags (`disable_bluetooth`, `disable_cellular`), **install integrity key material** |
| **Runtime state** | Changes with quarantine | Isolated?, active profile, snapshot ref, timestamps, reboot-reapply bit, throttle counters |
| **Ephemeral** | Never written | passwords, one-shot CLI secrets, BG **private** keys |

### Rules

1. **Install before isolate.**  
2. **ACLs:** admin/SYSTEM or root-only—assume malware tries to read.  
3. **Schema version** on both persistent stores.  
4. Automation channels documented per 0002; env only if install opt-in.  
5. CP may rewrite verifier/mode later (0004) without rewriting integrity key casually.

## Consequences

- Standalone and tenant installs share one schema with optional tenant fields.  
- Backups of install config are sensitive (trust, verifiers) but not plaintext passwords.  
- Agents/implementers know where each ADR’s data lives.

## Alternatives rejected

| Alternative | Why rejected |
| --- | --- |
| Single mutable blob for everything | Mixes secrets, identity, and quarantine churn |
| Store BG private key on endpoint | Violates 0008/0005 |
| No install step | Nowhere for verifier/tenant trust |

## Open

- Default install paths per OS.  
- Pre-server: operator-minted vs future-CP-assigned `tenant_id` (operator UUID OK for now).
