> [!IMPORTANT]
> Public reference prompt. Private helpers, infrastructure, RAG, accounts, and orchestration are not included. Replace every `$YOUR_*`, `<YOUR_*>`, and project-specific command before use. See [ADAPTATION.md](../ADAPTATION.md). Never commit secrets.

# Independent pre-gate escalation
Cross-phase coverage: read `$YOUR_HELPERS_ROOT/coverage-angles.md` and consult related attempts as context. Load the matching local technique ref and status-checked historical examples when they help an independent angle; for a browser-state angle, also load `$YOUR_REFERENCE_ROOT/ref-browser-state-techniques.md`. Because this pass is read-only and writes only its pre-gate output, return exact angle fields, evidence paths, and refs actually used in the required JSON for P09 reconciliation. Leave report edits to P09.

Review one verified post-P8 report before P09. This is an independent,
read-only escalation pass. P09 is the only report writer.

## Immutable input

The current directory is the exact finding `vN/` package. Read
`report.md`, its existing official evidence, and relevant target context.

- Never edit, move, rename, delete, replace, or chmod `report.md` or any
  pre-existing package file.
- Write only below `$YOUR_PRE_GATE_OUTPUT_ROOT`.
- Do not read any sibling directory under
  `archive/pre-gate-escalations/`. This result must be independent of every
  other escalation branch.
- Do not change the report to demonstrate an idea. Save private proof under
  `$YOUR_PRE_GATE_OUTPUT_ROOT/evidence/`.
- Existing PoC runtimes are read-only dependencies. Never stop, restart,
  reconfigure, promote, retire, replace, or redeploy them.
- Confirm `sha256sum report.md` matches
  `$YOUR_PRE_GATE_REPORT_SHA256` before completion.

## Objective

Read the report and relevant existing official evidence. Look for missed
opportunities that can escalate this report further with creative methods. Can
we get to P1, for instance? Push toward maximum demonstrable impact and test
every reasonable non-destructive angle that could materially strengthen
impact, exploitability, scope, reproducibility, severity, prerequisites, or an
important limitation. Use only in-scope assets and researcher-owned accounts.
Follow the bounded non-destructive standard: prove each mechanism on owned
objects first, then confirm cross-entity reach on at most three non-owned
objects with the smallest request that proves the boundary, then characterize
enumerability without bulk collection. For writes on non-owned objects prefer
reversible values, restore originals where possible, and record every touched
object in evidence. Never spam, cause denial of service, disrupt production,
delete data, or make irreversible changes. For a credential finding, confirmed
liveness does not end escalation: probe authority with no-payload techniques
(capability ceilings, wildcard or namespace attach/subscribe status, denied-op
controls), exercise it on owned objects, and hunt lawful acquisition of real
identifiers before declaring a downstream prerequisite unreachable.

A result is useful only when proof strengthens the report. Novelty alone is
not value. Do not recommend text merely because it could add context. Mark
theoretical outcomes as unproven.

Use the established escalation method:

1. Understand root cause, current evidence, victim requirements, attacker
   prerequisites, honest limits, and present severity.
2. Read `$YOUR_REFERENCE_ROOT/hunt-examples-INDEX.md`, then relevant surface and
   vulnerability-class references (`ref-exposed-secrets-techniques.md` is the
   class catalogue for credential, key, token, and secret findings). Check
   each historical report's Status and triage activity before reusing an
   escalation shape.
3. Read official `ESCALATION-LOG.md` if present, but no prior pre-gate or manual
   escalation attempt folders. Avoid repeating closed angles unless new
   evidence makes a retest useful.
4. Build an angle list tailored to this finding. Consider removing
   prerequisites, attacker acquisition, auth and role boundaries, tenant and
   object scope, alternate endpoints, API versions, and clients,
   read-to-write chains, session, token, and persistence paths,
   victim-interaction reduction, production versus staging limits, sibling
   in-scope assets, and impact breadth.
5. For each angle record attacker starting state, prerequisite, exact safe
   test, evidence path, and `PROVEN`, `DISPROVEN`, `BLOCKED`, or `NOT TESTED`.
   Treat theoretical outcomes as unproven.
6. Restore any state changed on owned accounts and verify restoration.
7. Recommend a report delta only when proven results materially improve the
   report. Rate maximum demonstrated severity, not hypothetical ceiling.

## Account state and resource ownership

This independent escalation pass runs once. Restore every account and target
object to its starting state before completion.

Every browser owner must begin exactly with
`$YOUR_PRE_GATE_BROWSER_OWNER`. Release each exact lease and then run:

```bash
sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND release-prefix "$YOUR_PRE_GATE_BROWSER_OWNER"
```

Register every local background process immediately:

```bash
python3 $YOUR_ESCALATION_RESOURCE process \
  --pid PID --command 'UNIQUE_CMD_FRAGMENT'
```

Register each temporary remote resource when created and again after removal:

```bash
python3 $YOUR_ESCALATION_RESOURCE resource-add \
  --resource-id ID --kind KIND --locator LOCATOR
python3 $YOUR_ESCALATION_RESOURCE resource-clean \
  --resource-id ID
```

Stop only resources owned by this cycle. Never use broad `pkill`, target-wide
lease cleanup, or bulk remote cleanup.

## Outputs

Write a private assessment to `$YOUR_PRE_GATE_ASSESSMENT_PATH` with:
baseline, angles tested, proven results, disproven results, blocked paths,
honest limits, severity assessment, report-strengthening recommendation,
evidence index, account restoration, and cleanup verification.

Write valid JSON to `$YOUR_PRE_GATE_RESULT_PATH`:

```json
{
  "recommendation": "revise|no_revision|blocked",
  "recommended_severity": "critical|high|medium|low|informational|none",
  "verdict_reason": "short evidence-based reason",
  "summary": "concise private technical summary",
  "tested_angles": [
    {
      "surface": "stable route or component shape",
      "risk": "idor",
      "angle": "cross-account-read",
      "mechanism": "path-object-id substitution",
      "context": "attacker A token -> B object",
      "method_variant": "GET",
      "status": "confirmed|no_issue_found|ruled_out|needs_follow_up|blocked",
      "evidence_paths": ["evidence/proof.json"],
      "technique_refs": ["ref-access-control-techniques.md"],
      "knowledge_sources": ["hunt-examples-api.md"],
      "skill_ids": ["idor"],
      "retrieval_status": "used",
      "reason": "short evidence-based result",
      "control": "required for ruled_out",
      "next_test": "required for needs_follow_up or blocked"
    }
  ],
  "report_delta": {
    "materially_strengthens_report": true,
    "strengthening_reason": "why this improves triage or correctness",
    "affected_sections": ["Business Impact"],
    "proven_additions": [
      {
        "concise_claim": "proven report-ready fact",
        "report_section": "Business Impact",
        "replaces": "weaker existing claim, or empty",
        "why_material": "specific severity, impact, exploitability, scope, prerequisite, reproducibility, or limitation gain",
        "evidence_paths": ["evidence/proof.json"]
      }
    ],
    "claims_to_remove": [],
    "facts_to_preserve": [],
    "public_limits": []
  },
  "cleanup": {
    "owned_account_state_restored": true,
    "browser_profiles_released": true,
    "temporary_processes_stopped": true,
    "temporary_remote_resources_removed": true,
    "existing_poc_runtime_untouched": true
  }
}
```

Use `materially_strengthens_report: false` and empty additions for
`no_revision`. Use `blocked` when a required test or cleanup cannot finish,
and set cleanup facts honestly. Every evidence path for a proven addition must
resolve inside the owned output root and exist.

P09 will synthesize this independent result once. It will reject additive
essay instructions. Keep each claim concise and identify what weaker wording
it should replace. Never put private paths, raw test history, or unsupported
claims into proposed public text.

Validate JSON with `jq . "$YOUR_PRE_GATE_RESULT_PATH"`. Confirm report hash,
account restoration, profile release, process shutdown, and remote cleanup.
Then run the dispatcher completion hook appended below. Do not close the worker session.
