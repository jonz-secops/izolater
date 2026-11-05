# High-level overview

## What izolater is

**izolater** is a small, multiplatform **client binary** (Zig, latest stable) that can **quickly contain a hostile endpoint**, then **reverse** that containment only when a valid **`ReverseProof`** is presented: reverse quarantine password, **tenant-scoped break-glass** proof, and/or a future **control-plane** capability.

Containment is broader than “a firewall rule”: **packet filters, routes, DNS, Wi‑Fi disconnect, and disabling interfaces** that are not needed for the sole remote exception—the future **control-plane** service (HTTP and/or RPC).

## Hostile environment

The product assumes contested hosts: **malware**, **socket-based remote control**, and **rogue or compromised automation** (including AI agents that can run tools). Goals: cut C2 paths fast, deny clean unisolate without proof, keep recovery possible for IR without a global backdoor. See [threat-model.md](threat-model.md) and [ADR-0005](adr/0005-threat-model-assumptions.md).

## Why it exists

Many containment tools either lock operators out, make unisolate too easy for an on-host adversary, or require a vendor control plane from day one. izolater aims for a **fast local binary**, **proof-gated reverse**, **tenant-bounded break-glass**, and **CP-ready contracts** without implementing the server here.

## Core user journey

1. **Install** — `tenant_id`, auth mode (`password_required` default), optional password verifier, CP endpoints, break-glass public trust material.  
2. **Isolate** — snapshot; apply hard profile (filter/routes/DNS/Wi‑Fi/NICs); leave CP path if configured; audit.  
3. **Operate while isolated** — local IR; remote only to CP (if any); C2 sockets should lose path.  
4. **Reverse** — password and/or break-glass sister tooling and/or CP proof → restore snapshot.  
5. **Recover secrets (future CP)** — reset password or change auth mode remotely under mTLS.

## Design pillars

| Pillar | Meaning |
| --- | --- |
| Hostile-ready | Design for malware, sockets, rogue agents—not clean labs |
| Reversible | Snapshot-before-mutate; exact restore of containment surfaces |
| Proof-gated reverse | Password / break-glass / CP — not an open admin convenience API |
| Tenant-bounded recovery | Sister break-glass blast radius ≤ tenant |
| Local-first, CP-ready | Offline CLI; server is a separate project |
| Multiplatform | Windows → macOS → Linux |
| Validate before code | ADRs stay Proposed until accepted |

## System context

```
┌─────────────────────────────┐         ┌──────────────────────────┐
│  izolater client (this repo)│◄───────►│  izolater server (OOS)   │
│  isolate / reverse / local  │ future  │  control plane, policy,  │
│  auth + break-glass verify  │  mTLS   │  password reset / modes  │
└─────────────────────────────┘         └──────────────────────────┘
         ▲
         │ stdin/file/CP (not env by default)
┌────────┴────────┐     ┌─────────────────────┐
│ installer / EDR │     │ tenant sister       │
│ / automation    │     │ break-glass tooling │
└─────────────────┘     └─────────────────────┘
```

## Reading order

1. [izolater-plan.md](izolater-plan.md)  
2. This overview → [features.md](features.md)  
3. ADRs 0001, 0005, 0008, then the rest  
4. [threat-model.md](threat-model.md) + [validation.md](validation.md)  
