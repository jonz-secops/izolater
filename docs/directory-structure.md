# Proposed project directory structure

```
izolater/
├── README.md
├── LICENSE
├── .gitignore
├── docs/                      # durable design docs
│   ├── izolater-plan.md
│   ├── overview.md
│   ├── features.md
│   ├── architecture.md
│   ├── threat-model.md
│   ├── validation.md
│   ├── non-goals.md
│   ├── directory-structure.md
│   └── adr/                   # ADR-0001 … 0008
├── contracts/                 # client↔CP assumptions
├── config/                    # example non-secret samples
├── scripts/
├── src/
│   ├── cli/
│   ├── auth/
│   ├── isolation/
│   ├── platform/              # windows, darwin, linux
│   ├── controlplane/
│   └── breakglass/            # sister tooling (scaffold)
└── tests/
    ├── integration/
    └── security/              # adversarial cases
```

No `build.zig` until implementation. No server tree ever in this repo.
