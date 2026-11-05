# ADR-0005: Threat model assumptions

- Status: Proposed
- Date: 2026-11-05
- Deciders: skrappy0x4a (pending acceptance)

## Context

Security claims must rest on explicit assumptions. See also [threat-model.md](../threat-model.md).

## Decision — we assume

1. **Operators** can protect the reverse password out-of-band (password manager, sealed IR runbook, future CP escrow).  
2. **v1 adversary focus:** remote attackers and non-admin local malware; **admin-equivalent malware** can disable host firewalls — we mitigate misuse of izolater APIs and secret exposure, not kernel integrity.  
3. **Privilege:** isolate/reverse require OS administrator / root.  
4. **Clock:** local clock may be skewed; security must not depend on perfect time except for audit readability.  
5. **Single-host tool:** izolater does not assume a TPM or secure boot, though may use them later.  
6. **Network isolation is the containment primitive** for v1.  
7. **Honest failure:** better to refuse isolate than to apply irreversible or unknown rules.  
8. **CP is untrusted until authenticated**; absence of CP is normal.

## Decision — we explicitly do not assume

- Secure boot / measured boot  
- That firewall rules are the only path packets can take (virtualization edges exist)  
- That env vars are a secret channel  
- That users will never lose the password  

## Consequences

- Marketing and docs must state admin-malware limits.  
- Validation effort concentrates on reverse auth, snapshot integrity, and bypass documentation.  
- Break-glass policy becomes an owner decision, not a crypto miracle.

## Alternatives considered

| Stance | Why rejected |
| --- | --- |
| “Prevent all unisolate even against SYSTEM malware” | Requires stronger root of trust than planned |
| “Password optional” | Violates product intent |

## Open questions

- Formal acceptance of this ADR by skrappy0x4a before implementation.
