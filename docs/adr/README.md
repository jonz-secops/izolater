# Architectural Decision Records — izolater client

**Status of the set:** all **Proposed**, not accepted.  
These eight records are meant to read as **one architecture**, not eight disconnected essays.

## The story they tell together

izolater is a **local Zig containment client** for **hostile** endpoints. It must **cut remote control** hard, stay **reversible**, and **authorize reverse** without a global backdoor—while staying ready for a **future control plane** that lives in another repo.

```
                    ADR-0005  Threat posture (hostile by default)
                              │
                              ▼
   ADR-0003 Identity & stores ◄──────────────────────────────┐
         │                                                   │
         ▼                                                   │
   ADR-0001 What “isolate” mutates ──► ADR-0006 How reverse  │
         │                              restores + integrity │
         │                                     ▲             │
         │                                     │             │
         └──────────► ADR-0008 Who may reverse ┤             │
                         ▲         ▲           │             │
                         │         │           │             │
                   ADR-0002   ADR-0004    (password /        │
                   password    CP proofs   break-glass /     │
                   material    + allowlist  admin-if-off)    │
                         │         │                         │
                         └────┬────┘                         │
                              ▼                              │
                        ADR-0007 How we ship it              │
                        (Zig; Win→macOS→Linux;               │
                         same orchestrator) ─────────────────┘
```

| If you care about… | Read first | Then |
| --- | --- | --- |
| Cutting C2 / what isolate does | **0005 → 0001** | 0006 |
| Unisolate without stranding IR | **0006 → 0008** | 0002, 0004 |
| Tenancy / sister binary | **0008 + tenancy.md** | 0003 |
| Future server hooks | **0004** | 0001 allowlist, 0008 proofs |
| Where code lives / OS order | **0007** | 0001 backends |

## Index

| ADR | Intent (one line) | Status |
| --- | --- | --- |
| [0001](0001-platform-isolation-mechanisms.md) | Define the **hard-isolate mutation sequence** and what we refuse to touch (USB). | Proposed |
| [0002](0002-crypto-password-handling.md) | How the **quarantine password** is stored, injected, throttled, and kept separate from integrity keys. | Proposed |
| [0003](0003-install-vs-runtime-config.md) | What is **durable identity** vs **runtime quarantine state** vs never-on-disk. | Proposed |
| [0004](0004-control-plane-client-contract.md) | How the client talks to a **future CP** (HTTPS+mTLS) and which powers CP may wield. | Proposed |
| [0005](0005-threat-model-assumptions.md) | **Hostile-by-default** assumptions and honest enforcement ceiling. | Proposed |
| [0006](0006-reversibility-model.md) | **Snapshot-before-mutate**, integrity, restore semantics—the twin of 0001. | Proposed |
| [0007](0007-multiplatform-strategy.md) | **Zig + Win→macOS→Linux** with one orchestrator and honest parity. | Proposed |
| [0008](0008-break-glass-and-auth-modes.md) | **Who is allowed to reverse**: modes, sister scopes, proofs—not the password KDF itself. | Proposed |

## Design cohesion rules (for editors)

1. **One owner per concern.** Password KDF stays in 0002; *who may reverse* stays in 0008; *what isolate changes* stays in 0001; *how restore works* stays in 0006.  
2. **Every ADR states Owns / Defers** so agents don’t re-decide the same thing twice.  
3. **Complement, don’t restate.** Link; don’t paste the hard-isolate table into four files.  
4. **Intentional > generic.** Prefer izolater-specific constraints (USB kept, TCP killed, tenant blast radius) over textbook security boilerplate.  
5. **Open questions only if blocking.** Resolved owner decisions leave the “Open” section.

## Related non-ADR docs

- [izolater-plan.md](../izolater-plan.md) · [feature-inventory.md](../feature-inventory.md) · [threat-model.md](../threat-model.md) · [tenancy.md](../tenancy.md)
