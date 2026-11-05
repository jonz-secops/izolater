# Example configuration (non-secret)

Future home for sample `izolater` config and JSON Schema.

**Rules:**

- No real passwords, verifiers, or customer endpoints in git.  
- Examples use placeholders only.  
- Install vs runtime split is defined in [docs/adr/0003-install-vs-runtime-config.md](../docs/adr/0003-install-vs-runtime-config.md).

Placeholder example (illustrative, not loaded by any binary yet):

```toml
# izolater.example.toml
schema_version = 1
default_profile = "hard"
# control_plane_endpoints = ["https://cp.example.invalid"]
```
