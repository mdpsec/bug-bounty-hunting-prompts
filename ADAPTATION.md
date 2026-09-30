# Adaptation and replacement index

The prompts are deliberately portable placeholders. Replace these values before
running any command. A prompt that still contains `YOUR_*`, `<YOUR_*>`, or an
unresolved private integration is not ready to execute.

## 1. Required runtime values

Replace the template fields supplied by your runner:

| Placeholder | Supply |
| --- | --- |
| `{{target}}` / `{{target_display}}` | The in-scope target name or hostname |
| `{{scope}}` / `{{scope_full}}` | The complete current program scope and exclusions |
| `{{platform}}` / `{{handle}}` | Your local program identifier, if useful |
| `{{target_norm}}` | A filesystem-safe target identifier |
| `{{report_format_path}}` | Your report template or format specification |
| `{{classification_catalog_path}}` / `{{classification_name}}` | Your platform taxonomy, if applicable |

Do not infer scope from discovery. Treat the program brief as authoritative.

## 2. Workspace and helper roots

The private workflow used several internal roots. Provide your own equivalents,
or remove the dependent command block:

| Public placeholder | What it represents |
| --- | --- |
| `$YOUR_WORKSPACE_ROOT` | One run's isolated workspace |
| `$YOUR_TARGET_ROOT` | Target-scoped artifacts, reports, raw data, and sensitive files |
| `$YOUR_HELPERS_ROOT` | Your scripts for endpoint catalogs, validation, provenance, and leases |
| `$YOUR_REFERENCE_ROOT` | Optional local technique references and historical examples |
| `$YOUR_RESOURCE_GUIDE` | Your inventory and rules for browser, email, OOB, VPS, and other tools |
| `$YOUR_PROJECT_ROOT` / `$YOUR_PROJECT_RULES` | Your own project instructions and policy files |
| `$YOUR_HUNT_BIN` | Your phase start, finish, intervention, or dispatcher commands |
| `$YOUR_SECRET_STORE` | A protected environment or secret manager. Never commit its contents |
| `$YOUR_DIG_ROOT` | Your finding queue and escalation tree |
| `$YOUR_LEASE_OWNER_PREFIX` | Your run-scoped browser/resource owner prefix. Replace the example owner format in each phase |
| `$YOUR_BROWSER_USER` | The local OS account that owns your browser profiles, if your tooling needs one |
| `$YOUR_CDP_HOST` | The host where your browser driver's CDP endpoint listens |
| `$YOUR_CDP_BASE_PORT` | Your profile-to-CDP port mapping, if profiles use numbered local ports |
| `$YOUR_MITM_HOST` | The host where your traffic-capture proxy listens |
| `$YOUR_MITM_BASE_PORT` | Your profile-to-proxy port mapping, if traffic capture uses numbered local ports |
| `$YOUR_FORWARDED_INBOX` | Researcher-owned destination used to receive platform-alias mail, if applicable |
| `$YOUR_NOTIFICATION_ENDPOINT` | Optional orchestration callback URL. Remove callback blocks if you do not have one |
| `$YOUR_EMAIL_API_KEY_COMMAND` | A protected command that prints the email API key without storing it in this repository |

Any other `$YOUR_*` variable in a prompt is a run-specific value. Define it,
rename it, or remove the surrounding integration.

## 3. Integrations to replace

The prompts describe capabilities, not required vendors. Choose one safe,
auditable implementation for each capability you keep:

- Browser control: CDP-capable Chrome or another browser driver, with your own
  profile lifecycle and isolated ownership rules.
- Traffic capture: an approved proxy or HAR capture path. Replace browser flow
  paths and `mitmdump` commands with your equivalent.
- Email verification: a researcher-owned mailbox or test inbox. Query only the
  exact address and keep API keys in your secret store.
- SMS and MFA: researcher-owned numbers or test factors within program rules.
  Do not copy provider names, numbers, recovery codes, or seeds into prompts.
- CAPTCHA and anti-bot handling: your approved solver or manual handoff. A
  solver is optional and must not become an excuse to exceed program limits.
- OOB callbacks: an approved interaction service or a controlled listener.
  Record ownership and clean up listeners after each bounded test.
- VPS or PoC hosting: your managed, finding-owned runtime. Replace all runtime
  registration, health-check, promotion, and cleanup commands.
- Evidence capture: your genuine screenshot, terminal capture, HAR, and mail
  viewing tools. Never synthesize evidence.
- Reporting and provenance: your report validator, classification mapping,
  finding provenance, and severity workflow.
- Phase orchestration: your own event logger, queue, agent runner, and user
  intervention path. These prompts do not ship a dispatcher or dashboard.

## 4. Optional reference corpus

The prompts mention technique references and historical examples. Those are
calibration material, not proof. You can:

1. Build a small local reference library with one file per vulnerability class.
2. Replace the reference paths with your own notes or approved documentation.
3. Remove RAG instructions if your workflow does not use retrieval.

Never include private reports, customer data, credentials, or unpublished target
details in a public reference corpus.

## 5. Accounts and secrets

Use only researcher-owned accounts and objects. Keep credentials, cookies, API
keys, OTPs, recovery codes, private keys, and target data in a protected store
outside the prompt repository. The prompts may ask you to record evidence paths,
but they should point to protected local files, not embed secret values in a
commit.

## 6. Phase-specific removal rules

- If you cannot provide authenticated accounts, run the public or unauthenticated
  branches and record the access limitation. Do not invent credentials.
- If you do not have a browser, skip the browser-walk phase and any browser-only
  angle. Do not treat the skipped work as tested.
- If you do not have OOB or PoC hosting, mark those angles `BLOCKED` and preserve
  the exact missing proof. Do not replace a blocked result with a negative result.
- If you do not have a report validator or platform taxonomy, use a small local
  schema and state its limitations in the report.

## 7. Minimum portable setup

The smallest useful adaptation can use:

- a target scope file;
- a run workspace with `raw/`, `reports/`, `hunt/`, and `sensitive/` directories;
- a normal browser or HTTP client;
- researcher-owned test accounts when needed;
- a local technique-notes directory, or no RAG at all; and
- a human review step before every state-changing or externally hosted action.

The prompts are intentionally not a turnkey scanner. Their value is the phase
boundaries, evidence discipline, handoffs, and triage gates.

## 8. Exact public variable inventory

Use this as a preflight checklist. Some values are required only by the phases
that reference them.

Workspace and files:

- `$YOUR_WORKSPACE_ROOT`
- `$YOUR_TARGET_ROOT`
- `$YOUR_TEMP_ROOT`
- `$YOUR_DIG_ROOT`
- `$YOUR_READY_ROOT`
- `$YOUR_ACCOUNTS_FILE`
- `$YOUR_PHASE_HUNT_ROOT`
- `$YOUR_PHASE_RAW_ROOT`
- `$YOUR_PHASE_REPORT_ROOT`

Tools and integrations:

- `$YOUR_HELPERS_ROOT`
- `$YOUR_REFERENCE_ROOT`
- `$YOUR_RESOURCE_GUIDE`
- `$YOUR_PROJECT_RULES`
- `$YOUR_HUNT_BIN`
- `$YOUR_SECRET_STORE`
- `$YOUR_ESCALATION_RESOURCE`
- `$YOUR_PROCESS_REGISTRY`
- `$YOUR_BROWSER_PROFILE_COMMAND`
- `$YOUR_BROWSER_HOME`
- `$YOUR_BROWSER_FLOW_DIR`
- `$YOUR_BROWSER_LEASE_DIR`
- `$YOUR_MITMDUMP`
- `$YOUR_HTTPX`
- `$YOUR_STEALTH_FEATURES`
- `$YOUR_STEALTH_MODE`
- `$YOUR_RESEARCH_HOST`

Evidence commands used in the examples:

- `your-terminal-capture`
- `your-browser-capture`
- `your-console-capture`
- `your-mail-viewer`
- `your-page-capture`

Runner and model metadata:

- `$YOUR_RUN_ID`
- `$YOUR_PROVIDER`
- `$YOUR_MODEL`
- `$YOUR_EFFORT`
- `$YOUR_STAMP`
- `$YOUR_SOFT_DEADLINE_AT`
- `$YOUR_HARD_DEADLINE_AT`

Independent escalation and final result paths:

- `$YOUR_PRE_GATE_OUTPUT_ROOT`
- `$YOUR_PRE_GATE_REPORT_SHA256`
- `$YOUR_PRE_GATE_BROWSER_OWNER`
- `$YOUR_PRE_GATE_ASSESSMENT_PATH`
- `$YOUR_PRE_GATE_RESULT_PATH`
- `$YOUR_P10_RESULT_PATH`
- `$YOUR_LEASE_CYCLE`
- `$YOUR_CALLBACK_TOKEN`
- `$YOUR_CYCLE_RUN_ID`

Credential and provider markers:

- `<ACCOUNT_PASSWORD>`
- `<EMAIL_API_HOST>`
- `<CATCH_ALL_DOMAIN_1>` through `<CATCH_ALL_DOMAIN_4>` (or your configured catch-all domain count)
- `<YOUR_SOCIAL_PROFILE_SET>` and `<YOUR_FLOW_PROFILE_SET>` for your own browser pools, if used
- `<YOUR_GEO_PROFILE_POOLS>` for any profile-to-region allocation owned by your browser manager
- `<PLATFORM_ALIAS_BASE>`
- `<HACKERONE_ALIAS_DOMAIN>` and `<BUGCROWD_ALIAS_DOMAIN>` when you are
  authorized to use platform researcher aliases
- `<YOUR_PROXY_PROVIDER>` and `<YOUR_ALTERNATE_PROXY_PROVIDER>`
- `<YOUR_CAPTCHA_SOLVER>`
- `<YOUR_SMS_PROVIDER>`
- `<YOUR_OOB_TOOL>`

Other angle-bracket values such as `<METHOD>`, `<SEVERITY>`, `<VERDICT>`,
`<UTC>`, and `<JSON>` are per-run output fields, not infrastructure settings.
