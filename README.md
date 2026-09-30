# Bug Bounty Hunting Prompts

Reusable prompt files for building a structured, evidence-first web bug bounty
hunting workflow.

This repository contains prompts adapted from a private hunting workflow. It
does not contain the original helpers, browser fleet, proxy or email services,
dashboard, proof-of-concept runtime, credentials, private operating rules, or
reference corpus. The prompts are intentionally not runnable without adapting
those dependencies to your own environment.

## Provided as-is

This prompt pack is provided as-is, without setup, integration, debugging, or
usage support. We will not provide the private helpers, infrastructure,
accounts, RAG corpus, orchestration services, or equivalent replacements used
in our own workflow. Users are responsible for adapting the prompts, supplying
their own tools and resources, validating results, and operating within their
own authorization and program rules. Please do not expect maintainers to
reproduce private environments or answer individual implementation questions.

Start with [`ADAPTATION.md`](ADAPTATION.md). It is the replacement index for
every private integration represented by a `YOUR_*` variable or an angle-bracket
placeholder. Do not paste credentials, cookies, tokens, API keys, or private
infrastructure details into these prompt files.

## Prompt sequence

The public release contains this sequence:

| Phase | File | Purpose |
| --- | --- | --- |
| 0 | [`phase-00-recon.md`](phases/phase-00-recon.md) | Lightweight surface map |
| 1 | [`phase-01-access.md`](phases/phase-01-access.md) | Account setup and authenticated access |
| 2 | [`phase-02-sweep.md`](phases/phase-02-sweep.md) | Broad endpoint and vulnerability-class sweep |
| 3 | [`phase-03-closeout.md`](phases/phase-03-closeout.md) | Validate the sweep handoff and release resources |
| 4a | [`phase-04a-scope.md`](phases/phase-04a-scope.md) | Scope-only context |
| 4b | [`phase-04b-browser-walk.md`](phases/phase-04b-browser-walk.md) | Authenticated browser walk |
| 5a | [`phase-05a-scope.md`](phases/phase-05a-scope.md) | Scope-only context for the critical hunt |
| 5b | [`phase-05b-critical.md`](phases/phase-05b-critical.md) | Critical-impact hunting |
| 6 | [`phase-06-triage.md`](phases/phase-06-triage.md) | Initial triage and deduplication |
| 7 | [`phase-07-validate-escalate.md`](phases/phase-07-validate-escalate.md) | Validate and escalate findings |
| 8 | [`phase-08-verify-escalate.md`](phases/phase-08-verify-escalate.md) | Independent verification and escalation |
| 8a | [`phase-08a-independent-escalation.md`](phases/phase-08a-independent-escalation.md) | One read-only pre-gate escalation pass |
| 9 | [`phase-09-triage-gate.md`](phases/phase-09-triage-gate.md) | Independent live triage gate |
| 10 | [`phase-10-self-duplicate-check.md`](phases/phase-10-self-duplicate-check.md) | Same-program duplicate convergence |

The duplicate second critical branch and duplicate second independent escalation
branch are not included in this public sequence. The browser-walk phase remains
available as an optional browser-focused stage.

## How to use the prompts

These are prompt source files, not an application or orchestration engine. A
typical adaptation is:

1. Choose the phases that match your workflow.
2. Replace the runtime variables and integration commands listed in
   [`ADAPTATION.md`](ADAPTATION.md).
3. Supply current target scope, researcher-owned accounts, and local evidence
   paths at runtime.
4. Provide your agent with one phase at a time and preserve the phase handoff
   artifacts it requests.
5. Run only against authorized, in-scope assets. Use bounded, non-destructive
   proofs and restore reversible test changes.

You can remove integrations you do not use. For example, an unauthenticated
workflow can omit account setup, browser leases, email, SMS, and authenticated
walks while retaining the recon, sweep, triage, and reporting principles.

## Safety and disclosure

Use these prompts only for authorized security research. Do not target real
users, submit or publish findings automatically, collect data in bulk, or leave
state changes behind. Keep secrets outside the repository and report findings
through the program's responsible disclosure channel.

## Status

This is a reference prompt pack. It is expected to evolve as hunters adapt it
to different agents, tools, and program rules.
