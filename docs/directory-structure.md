# Proposed project directory structure

Planning-phase layout. Directories exist as **README scaffolds**; no product Zig sources yet.

```
izolater/
├── README.md                 # project entry
├── LICENSE
├── .gitignore
├── docs/                     # durable design docs (this tree)
│   ├── README.md
│   ├── izolater-plan.md      # concise user-facing plan
│   ├── overview.md
│   ├── features.md
│   ├── architecture.md
│   ├── threat-model.md
│   ├── validation.md
│   ├── non-goals.md
│   ├── directory-structure.md
│   └── adr/                  # Architectural Decision Records
│       ├── README.md
│       └── 000N-*.md
├── contracts/                # client↔server assumptions (no server impl)
│   └── README.md
├── config/                   # example config schemas / samples (non-secret)
│   └── README.md
├── scripts/                  # future maintainer scripts (empty of product logic)
│   └── README.md
├── src/                      # future Zig modules (scaffolds only)
│   ├── README.md
│   ├── cli/
│   ├── auth/
│   ├── isolation/
│   ├── platform/
│   └── controlplane/
└── tests/                    # future tests
    ├── README.md
    ├── integration/
    └── security/
```

## Mapping to architecture

| Path | Future contents |
| --- | --- |
| `src/cli` | Argument parsing, human/JSON output |
| `src/auth` | KDF, verify, rotation |
| `src/isolation` | State machine, snapshot format |
| `src/platform` | OS backends (`windows`, `linux`, `darwin`) |
| `src/controlplane` | Client transport stubs implementing `contracts/` |
| `contracts/` | Versioned message/command schemas (Markdown/JSON Schema later) |
| `config/` | Example `izolater.toml` / JSON schema without secrets |
| `tests/security` | Negative auth, tamper, privilege tests |

## Intentionally absent for now

- `build.zig` / `build.zig.zon` — added when implementation starts (Zig latest).  
- Vendor folders, CI release pipelines — after first spike.  
- Server tree — never in this repo.

## Evolution rules

1. Behavior changes that affect security → update ADR or file a new one.  
2. New top-level dirs need a one-line README stating purpose.  
3. Do not place secrets or real password material under `config/`.  
