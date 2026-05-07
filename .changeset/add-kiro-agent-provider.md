---
"@ai-hero/sandcastle": patch
---

Add Kiro CLI agent provider. Use `kiro(model, options?)` to run Kiro CLI's headless mode (`kiro-cli chat --no-interactive --trust-all-tools`) inside a Sandcastle sandbox; authenticate via the `KIRO_API_KEY` environment variable. `sandcastle init` now offers `kiro` as a selectable agent and scaffolds a Dockerfile that installs the CLI from `https://cli.kiro.dev/install`.
