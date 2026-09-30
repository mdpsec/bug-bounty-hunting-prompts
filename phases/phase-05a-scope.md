> [!IMPORTANT]
> Public reference prompt. Private helpers, infrastructure, RAG, accounts, and orchestration are not included. Replace every `$YOUR_*`, `<YOUR_*>`, and project-specific command before use. See [ADAPTATION.md](../ADAPTATION.md). Never commit secrets.

# Scope context for {{target_display}}
The next testing phase uses the shared cross-phase angle ledger. This scope-only phase does not read, write, or test ledger entries.
Below is the full program scope (in and out). Read it and hold it in context. Do NOT start any testing, recon, or tooling yet. This is reference only.

- Primary focus: {{target_display}}
- A `*.` prefix means wildcard: any subdomain of that apex is in scope unless explicitly listed Out of Scope below.
- Operational target hosts also include evidence-backed first-party functional siblings recorded in `$YOUR_TARGET_ROOT/raw/scope-hosts.txt`. A sibling directly used by the listed application for auth, account, application UI, API, or billing is part of the same target surface. Explicit named exclusions override; passive DNS names and third-party providers do not qualify.

{{scope_full}}

Read only. Take no action. Wait for your next prompt, which contains your actual task.
