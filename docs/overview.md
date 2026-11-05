# High-level overview

## What izolater is

**izolater** is a small, multiplatform **client binary** (Zig, latest stable) that can **quickly isolate an endpoint** during incident response or containment, then **reverse that isolation** only when a **password** is provided.

Isolation means cutting the host off from untrusted network paths while preserving a deliberate, audited path back to a recoverable state. Reversal is a first-class operation — not an afterthought — and is gated by a secret established at install time (or supplied by a trusted peer process / future control plane).

## Why it exists

Endpoint containment tools often either:

- lock an operator out of a reversible path, or
- make “unisolate” too easy for an attacker who already has host access, or
- are tied to a vendor control plane from day one.

izolater aims for a **fast local binary** with a **password-gated reverse**, designed so a **future control plane** can drive isolate/unisolate without reinventing the client security model.

## Core user journey

1. **Install** — set reverse password (interactive or supplied by installer/orchestration); persist verifier + isolation policy; optionally register control-plane endpoints (stubs only for now).
2. **Isolate** — apply platform-native network containment; record pre-state for exact reverse; emit audit events.
3. **Operate while isolated** — host remains usable for local IR; network egress/ingress restricted per policy.
4. **Reverse** — present password (CLI / env / stdin / future CP command); verify; restore recorded state; confirm connectivity posture.

## Design pillars

| Pillar | Meaning |
| --- | --- |
| Reversible by design | Every isolate action produces a restore plan; reverse is a first-class command. |
| Password-gated reverse | Unisolate requires proof of the install-time (or rotated) secret. |
| Local-first, CP-ready | Works offline as a CLI; contracts assume a future izolater server without implementing it. |
| Multiplatform | Windows primary for many IR shops; Linux and macOS in the same abstraction model. |
| Secure defaults | Hash secrets, minimize privilege window, fail closed on reverse, fail safe on isolate where possible. |
| Validate before code | Threat model, ADRs, and proof gates land before Zig product code. |

## System context (this repo vs server)

```
┌─────────────────────────────┐         ┌──────────────────────────┐
│  izolater client (this repo)│◄───────►│  izolater server (OOS)   │
│  isolate / reverse / local  │ future  │  control plane, policy,  │
│  auth verifier, platform I/O│  mTLS   │  fleet, audit aggregation│
└─────────────────────────────┘         └──────────────────────────┘
         ▲
         │ password / config injection
┌────────┴────────┐
│ installer / EDR │
│ / automation    │
└─────────────────┘
```

Only the left box and its **client contracts** are in scope here. See [non-goals.md](non-goals.md).

## Reading order

1. This overview  
2. [features.md](features.md)  
3. [adr/](adr/) (especially isolation, crypto, reversibility)  
4. [threat-model.md](threat-model.md) + [validation.md](validation.md)  
5. [directory-structure.md](directory-structure.md)  

Concise entry point for stakeholders: [izolater-plan.md](izolater-plan.md).
