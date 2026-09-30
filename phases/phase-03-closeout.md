> [!IMPORTANT]
> Public reference prompt. Private helpers, infrastructure, RAG, accounts, and orchestration are not included. Replace every `$YOUR_*`, `<YOUR_*>`, and project-specific command before use. See [ADAPTATION.md](../ADAPTATION.md). Never commit secrets.

# Phase 3: Close-out

Cross-phase coverage: read `$YOUR_HELPERS_ROOT/coverage-angles.md`, run its `summary`, and carry unresolved angle IDs, next tests, reference citations, and any `pending` or `blocked` retrieval status into the handoff. Do not hunt, fetch new material solely for closeout, or mark a gap clean here. The Phase 02 matrix remains the sweep gate.

Short phase. Audit what the sweep covered, validate that what it wrote is well-formed, hand the gaps forward, release the profiles.

**You do not triage here.** The So-What gate lives downstream in P06, and the live re-reproduction gate lives in P09. Both do it properly. A third, weaker triage at this point is how a corpus ends up with hundreds of reports that nothing downstream accepts. Your job is to make the handoff clean, not to judge merit.

You also do not hunt. If the audit surfaces an untested Tier 1 shape, it goes in the notes for the goal cycles. It does not get tested now.

## Resources
`$YOUR_RESOURCE_GUIDE` for tooling. You need almost none of it.

**Read `$YOUR_HELPERS_ROOT/report-format.md` now.** It is the canonical public workflow format. Close-out validates that format and preserves original finding attribution; it never claims that Phase 03 discovered a finding.

## Target
- **Domain**: {{target}}
- **Scope**: {{scope}}

## Phase tracking

```bash
$YOUR_HUNT_BIN/hunt-phase-event start \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-03-closeout true
```

## User intervention notify

```bash
$YOUR_HUNT_BIN/hunt-intervention wait \
  --platform '{{platform}}' --handle '{{handle}}' \
  --target '{{target_norm}}' --phase phase-03-closeout \
  --title '<cause and evidence, max 120 chars>' \
  --action '<specific user action, max 120 chars>'
```

Discussion does not resolve a wait. When user input gives you an actionable next attempt, run `$YOUR_HUNT_BIN/hunt-intervention continue` immediately before resuming tools. If still blocked, fire `wait` again. Each round pauses only this phase's execution limit while wall and wait time continue recording.

## Setup

```bash
set +u
: "${YOUR_WORKSPACE_ROOT:?runner did not provide isolated workspace}"
: "${YOUR_TARGET_ROOT:?runner did not provide target root}"
: "${YOUR_HELPERS_ROOT:?runner did not provide frozen helpers}"
[ "$YOUR_TARGET_ROOT" = "$YOUR_WORKSPACE_ROOT" ] || { echo "ERROR: isolated target root mismatch"; exit 1; }
BASE="$YOUR_WORKSPACE_ROOT"
TARGET="{{target_norm}}"
APEX="$TARGET"; APEX="${APEX#wild.}"
export PHASE=phase-03-closeout
EP=$YOUR_HELPERS_ROOT/bin/endpoints
cd "$YOUR_WORKSPACE_ROOT"
R="$YOUR_TARGET_ROOT/raw"
mkdir -p "$R/eligibility"
REPORT_SPEC=$YOUR_HELPERS_ROOT/report-format.md
REPORT_VALIDATOR=$YOUR_HELPERS_ROOT/phase-03-closeout/validate-reports.py
MISSING_REPORTS=$YOUR_HELPERS_ROOT/phase-03-closeout/find-missing-reports.py
COVERAGE_MATRIX=$YOUR_HELPERS_ROOT/phase-02-sweep/coverage_matrix.py
ORPHAN_AUDIT=$YOUR_HELPERS_ROOT/phase-03-closeout/audit-orphans.py
MOBILE_HANDOFF_HELPER=$YOUR_HELPERS_ROOT/mobile-handoff-results.py
THREAT_MODEL_HELPER=$YOUR_HELPERS_ROOT/threat-model.py

for required in hunt/phase-01-access.md "$REPORT_SPEC" "$REPORT_VALIDATOR" "$MISSING_REPORTS" "$COVERAGE_MATRIX" "$ORPHAN_AUDIT" "$MOBILE_HANDOFF_HELPER" "$THREAT_MODEL_HELPER"; do
  [ -r "$required" ] || { echo "ERROR: required close-out input missing: $required"; exit 1; }
done

ACCESS_MODE=$(awk '/^## ACCESS MODE/{getline; while($0 ~ /^$/) getline; print $1; exit}' hunt/phase-01-access.md 2>/dev/null)
case "$ACCESS_MODE" in
  RICH|PARTIAL|UNAUTH) ;;
  *) echo "ERROR: invalid or missing ACCESS MODE in hunt/phase-01-access.md"; exit 1 ;;
esac
PROFILE_A=$(awk -F'|' '/^\| A \|/{gsub(/[^0-9]/,"",$6); print $6; exit}' hunt/phase-01-access.md)
PROFILE_B=$(awk -F'|' '/^\| B \|/{gsub(/[^0-9]/,"",$6); print $6; exit}' hunt/phase-01-access.md)
ACCESS_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_RUN_ID:0:12}-access:"
SWEEP_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_RUN_ID:0:12}-sweep:"
echo "ACCESS_MODE=$ACCESS_MODE"
python3 "$THREAT_MODEL_HELPER" . validate > "$R/closeout-threat-model-validation.txt" || {
  cat "$R/closeout-threat-model-validation.txt"
  exit 1
}
```

Stamp this phase too, so the ledger records who closed the target out. Same helper as the sweep, specified in `$YOUR_HELPERS_ROOT/report-format.md`:

```bash
. $YOUR_HELPERS_ROOT/bin/provenance.sh && hunt_provenance
```

## Step 1: Coverage audit

Rebuild the catalog, then measure what the sweep actually reached.

```bash
$EP rebuild
$EP stats

if python3 "$COVERAGE_MATRIX" . validate > "$R/closeout-coverage-validation.txt"; then
  COVERAGE_MATRIX_OK=1
else
  COVERAGE_MATRIX_OK=0
fi
cat "$R/closeout-coverage-validation.txt"

IDENTITY_LANE_OK=0
if jq -e '.identity_lane == "dual" or .identity_lane == "single" or .identity_lane == "unauth"' \
  "$R/phase02-identity-lane.json" > "$R/closeout-identity-lane-validation.txt" 2>&1; then
  IDENTITY_LANE_OK=1
else
  echo "missing or invalid Phase 02 identity lane record" \
    > "$R/closeout-identity-lane-validation.txt"
fi

DISRUPTIVE_TAIL_OK=0
if jq -e '
  (.status == "completed" or .status == "partial" or .status == "not-applicable" or .status == "policy-excluded")
  and (.reason | length > 0)
  and (.checks | type == "array" and length > 0)
  and all(.checks[];
    ((.name | type) == "string" and (.name | length) > 0)
    and (.status == "completed" or .status == "partial" or .status == "not-applicable" or .status == "policy-excluded")
    and ((.reason // "") | length > 0))
  and (if .status == "completed" then all(.checks[]; (.applicable != true) or .status == "completed") else true end)
  and (if .identity_lane == "dual" then
    any(.checks[]; .name == "password-email-change")
    and any(.checks[]; .name == "reset-completion")
    and (((.reason // "") | test("preserv.*(closeout|close-out)"; "i")) | not)
    and all(.checks[]; (((.reason // "") | test("preserv.*(closeout|close-out)"; "i")) | not))
  else true end)
' \
  "$R/phase02-disruptive-tail.json" > "$R/closeout-disruptive-tail-validation.txt" 2>&1; then
  DISRUPTIVE_TAIL_OK=1
else
  echo "missing or invalid Phase 02 disruptive tail record" \
    > "$R/closeout-disruptive-tail-validation.txt"
fi
cat "$R/closeout-identity-lane-validation.txt"
cat "$R/closeout-disruptive-tail-validation.txt"

$EP list --untested > $R/closeout-untested.txt
UNTESTED_N=$(wc -l < $R/closeout-untested.txt)
TOTAL_N=$($EP list --format shape 2>/dev/null | wc -l)
echo "shapes: $TOTAL_N total, $UNTESTED_N with zero tests"

# Per-class eligible remainder, using the same vocabulary and eligibility files
# the sweep worked from. Raw `endpoints pending` is only a superset.
TESTS=(ssrf ssti xxe deser jndi lfi redirect upload
       idor idor-x unauth-access id-walk path-bypass method-override api-version func-level graphql-authz
       jwt session oauth pwreset token-leak siwe preauth-bypass
       numeric massassign race idempotency workflow method-swap entitlement
       domxss cswsh cache header-inj second-order api-abuse
       injection)
: > $R/closeout-pending.txt
for t in "${TESTS[@]}"; do
  eligible="$R/eligibility/$t-eligible.txt"
  if [ ! -f "$eligible" ]; then
    echo "$t MISSING_ELIGIBILITY" >> $R/closeout-pending.txt
    continue
  fi
  comm -12 <($EP pending --test "$t" 2>/dev/null | sort -u) <(sort -u "$eligible") \
    > "$R/eligibility/$t-remaining.txt"
  n=$(wc -l < "$R/eligibility/$t-remaining.txt")
  [ "$n" -gt 0 ] && echo "$t $n" >> $R/closeout-pending.txt
done
cat $R/closeout-pending.txt

# High-value operation candidates must enter eligibility, receive a test event,
# or carry a schema-valid exclusion in raw/orphan-exclusions.jsonl. Static
# candidates do not disappear because bounded browser walk did not trigger them.
if python3 "$ORPHAN_AUDIT" . validate > "$R/closeout-orphan-audit.txt"; then
  ORPHAN_AUDIT_OK=1
else
  ORPHAN_AUDIT_OK=0
fi
cat "$R/closeout-orphan-audit.txt"
```

Cross-reference the untested shapes against `raw/tier1-hosts.txt`. **A Tier 1 host with untested shapes is the single most useful thing you can hand forward**, because the goal cycles pick their own angles and will not know it was missed unless you say so.

Do not test it now. Write it down. High-value orphan candidates block clean completion and must return to Phase 02 eligibility. Closeout does not invent eligibility or exclusions.

`raw/orphan-exclusions.jsonl` is limited to four provable categories:

```json
{"shape":"POST https://target/path","category":"duplicate|policy-excluded|third-party|parser-artifact","covered_by_shape":"POST https://target/canonical-path","policy_source":"program rule or scope record","parser_basis":"exact extraction defect","reason":"specific reason","evidence":["raw/existing-proof.txt"]}
```

- `duplicate` requires `covered_by_shape` already present in eligibility or test events.
- `policy-excluded` requires exact `policy_source`.
- `third-party` requires candidate host to be absent from canonical `raw/scope-hosts.txt`.
- `parser-artifact` requires exact `parser_basis` and file-backed extraction evidence.
- Every exclusion must match a real high-value operation candidate and name at least one existing target-relative evidence file.
- Static-only discovery, no live request, missing context, missing identity, missing object/state, or blocked prerequisite never qualify. Route exact candidate to its declared classes and retain it as PARTIAL.

Orphan audit rejects loose reasons and invalid categories. Phase 03 cannot mass-write exclusions to make closeout pass.

## Step 2: Validate the reports are well-formed

Format only. You are not judging whether a finding is real, severe, or worth submitting; downstream does that. You are checking that downstream can parse it.

Capture report-set digest before any structural repair. Any content edit, provenance recovery, report recovery, or duplicate merge invalidates prior Stage C proof.

```bash
REPORT_SHA_AT_CLOSEOUT_START=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' -print0 \
  | sort -z | xargs -0 -r sha256sum | sha256sum | awk '{print $1}')
printf '%s\n' "$REPORT_SHA_AT_CLOSEOUT_START" > "$R/closeout-report-start.sha256"
```

The spec is `$YOUR_HELPERS_ROOT/report-format.md`. Use the canonical validator so exact headings, order, content, severity, and provenance fields cannot drift independently inside this prompt.

```bash
if python3 "$REPORT_VALIDATOR" reports > "$R/closeout-report-validation.txt"; then
  REPORT_FORMAT_OK=1
else
  REPORT_FORMAT_OK=0
fi
cat "$R/closeout-report-validation.txt"
```

Audit confirmed medium-or-above angles and leads for missing reports. This detects findings that never made it from working memory into `reports/`.

```bash
python3 "$MISSING_REPORTS" . > "$R/closeout-missing-reports.txt" || true
cat "$R/closeout-missing-reports.txt"
```

Fix only structural defects supported by existing files: add an omitted heading, normalise a severity line, or fill `Component findings` from its angle file. **Do not invent Impact, evidence, attribution, or finding content.** If Impact is weak, flag it in the ledger under `Needs evidence before triage`.

If a confirmed medium-or-above finding has no report, reconstruct it only when existing angle and evidence files support every canonical section. Original attribution is mandatory:

- `raw/phase02-provenance.json` must provide Phase 02 central-writer phase, provider, model, and effort.
- For orchestrator findings, use the recorded stage from the angle file.
- For delegated findings, include the lead ID and its `determined_by` identity from `raw/leads.jsonl` verbatim in `Found by`.
- Add `report recovered by phase-03-closeout` to `Found by` so recovery is transparent.
- If exact source attribution is unavailable, do not guess and do not stamp Phase 03. Leave the item in `closeout-missing-reports.txt`, notify the user, and record the blocker in `CLOSEOUT.md`.

After writing the complete missing report, stamp it from the saved Phase 02 identity in a subshell. Replace `<source attribution>` with the exact orchestrator stage or delegated lead identity:

```bash
REPORT_TO_RECOVER="reports/REPORT-NN.md"
SOURCE_ATTRIBUTION="<source attribution>; report recovered by phase-03-closeout"
PROV="$R/phase02-provenance.json"

if jq -e '.phase=="phase-02-sweep" and (.provider|length)>0 and (.model|length)>0 and (.effort|length)>0' "$PROV" >/dev/null 2>&1; then
  (
    export PHASE=$(jq -r '.phase' "$PROV")
    export YOUR_PROVIDER=$(jq -r '.provider' "$PROV")
    export YOUR_MODEL=$(jq -r '.model' "$PROV")
    export YOUR_EFFORT=$(jq -r '.effort' "$PROV")
    . $YOUR_HELPERS_ROOT/bin/provenance.sh
    stamp_report "$REPORT_TO_RECOVER" "$SOURCE_ATTRIBUTION"
  )
else
  echo "ERROR: exact Phase 02 provenance unavailable; do not attribute report to Phase 03"
fi
```

For an existing report with missing provenance, use the same recovery rule only when its component angle or lead proves it came from Phase 02. Leave existing `Model: unknown` unchanged; never guess it. Re-run both audits after repairs:

```bash
python3 "$REPORT_VALIDATOR" reports > "$R/closeout-report-validation.txt"
REPORT_FORMAT_OK=$?
cat "$R/closeout-report-validation.txt"
python3 "$MISSING_REPORTS" . > "$R/closeout-missing-reports.txt" || true
cat "$R/closeout-missing-reports.txt"
```

Three things to check by reading, not grepping:

- **Impact answers the question.** It must say what an attacker actually does. If it says a control is missing and stops there, flag it in the ledger under `Needs evidence before triage`. Do not delete the report and do not rewrite the claim.
- **No duplicates.** Two reports on the same root cause and endpoint should be one report with an `## Escalation` section. Merge them; that is bookkeeping, not triage. Preserve earliest report's immutable provenance and record merged source files or lead IDs under `Component findings`.
- **Severity is internally consistent.** A report claiming CRITICAL whose Impact describes a single-record read is inconsistent with itself. Flag it. Do not recalibrate it: P06 and P09 own severity and both will look at it with fresh eyes.

## Step 3: Confirm or rerun Stage C

The sweep's chaining stage is where a set of mediums becomes a high, and nothing downstream does it: each goal cycle starts cold and sees one angle at a time.

```bash
CURRENT_REPORTS=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' | wc -l)
CURRENT_REPORT_SHA256=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' -print0 \
  | sort -z | xargs -0 -r sha256sum | sha256sum | awk '{print $1}')
REPORT_SHA_AT_CLOSEOUT_START=$(cat "$R/closeout-report-start.sha256" 2>/dev/null)
STAGE_C_OK=0
if jq -e --argjson current "$CURRENT_REPORTS" --arg digest "$CURRENT_REPORT_SHA256" '
  (.phase=="phase-02-sweep" or .phase=="phase-03-closeout")
  and .all_confirmed_reviewed==true
  and .report_count==$current
  and .report_sha256==$digest
  and (.completed_at|length)>0
' "$R/stage-c-complete.json" >/dev/null 2>&1; then
  STAGE_C_OK=1
  cat "$R/stage-c-complete.json"
else
  echo "GAP: Stage C completion is missing, invalid, or predates current report set"
fi
```

Completion record proves Stage C reviewed exact current report bytes. Report provenance belongs at creation time. Closeout may recover missing provenance only from saved Phase 02 metadata, and must preserve original finding attribution.

If `CURRENT_REPORT_SHA256` differs from `REPORT_SHA_AT_CLOSEOUT_START`, closeout changed report content. It must invalidate old completion record, validate reports, rerun Stage C over current reports and existing Phase 02 evidence, recompute digest, then rewrite `raw/stage-c-complete.json`. Do not send new target traffic. Review every current report against all confirmed/partial angles, `raw/leads.jsonl`, chain events, identity-object map, and existing scale evidence. Each report needs concrete `## Chain value`, including `standalone` when no supported combination exists. If existing evidence cannot answer a chain question, record gap in `RECON-NOTES.md`; do not invent result or completion record.

```bash
if [ "$CURRENT_REPORT_SHA256" != "$REPORT_SHA_AT_CLOSEOUT_START" ]; then
  rm -f "$R/stage-c-complete.json"
  python3 "$REPORT_VALIDATOR" reports > "$R/closeout-report-validation.txt" || exit 1

  # Perform Stage C review here using current reports and existing evidence only.
  # Do not continue until every report has a non-empty Chain value section.
  STAGE_C_READY=1
  for report in reports/REPORT-*.md; do
    [ -e "$report" ] || continue
    chain_value=$(awk '
      /^## Chain value$/ {in_section=1; next}
      in_section && /^## / {exit}
      in_section && NF {print; exit}
    ' "$report")
    [ -n "$chain_value" ] || { echo "MISSING Chain value: $report"; STAGE_C_READY=0; }
  done

  if [ "$STAGE_C_READY" -eq 1 ]; then
    CURRENT_REPORTS=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' | wc -l)
    CURRENT_REPORT_SHA256=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' -print0 \
      | sort -z | xargs -0 -r sha256sum | sha256sum | awk '{print $1}')
    STAGE_C_CHAINS=$(jq -r 'select(.kind=="chain") | .chain_id' "$R/endpoints-events.jsonl" 2>/dev/null | sort -u | wc -l)
    STAGE_C_ESCALATIONS=$(grep -rl 'Escalation of:' hunt/*.md 2>/dev/null | wc -l)
    jq -n \
      --arg phase "phase-03-closeout" \
      --arg review_reason "report content changed during structural closeout repair" \
      --arg reviewed_by "$YOUR_STAMP" \
      --arg completed_at "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
      --arg report_sha256 "$CURRENT_REPORT_SHA256" \
      --argjson report_count "$CURRENT_REPORTS" \
      --argjson chain_count "$STAGE_C_CHAINS" \
      --argjson escalation_count "$STAGE_C_ESCALATIONS" \
      '{phase:$phase,review_reason:$review_reason,reviewed_by:$reviewed_by,completed_at:$completed_at,
        all_confirmed_reviewed:true,report_count:$report_count,report_sha256:$report_sha256,
        chain_count:$chain_count,escalation_count:$escalation_count}' \
      > "$R/stage-c-complete.json"
    STAGE_C_OK=1
  else
    STAGE_C_OK=0
  fi
fi
```

If closeout did not change reports and Stage C is missing or stale, record gap. Do not create proof for a review that never ran.

## Step 4: Write RECON-NOTES.md

This is the handoff. The goal cycles that follow choose their own angles, so anything you do not write here is invisible to them.

Write `hunt/RECON-NOTES.md`:

```markdown
# Recon notes, $TARGET
Date: {date}
Access mode at close: {RICH | PARTIAL | UNAUTH}
Coverage matrix: `hunt/phase-02-coverage.md` ({PASS | INCOMPLETE})

## Untested Tier 1 shapes
Shapes on a Tier 1 host with zero tests. Highest-value follow-up.
| Shape | Host | Why it was missed |
|---|---|---|
| `METHOD https://host/path` | host | out of budget / blocked / needed an identity we lacked |

## Classes with a pending remainder
| Class | Shapes pending | Reason |
|---|---|---|
| ssrf | 12 | budget |
| idor-x | 30 | no second identity |

## High-value orphan operation candidates
From `raw/orphan-candidates.jsonl`. These appeared in static, documented, route-manifest, browser, or mitm sources but entered neither eligibility nor exact-shape test evidence and lack explicit exclusion.
| Shape | Source | Why high value | Required routing |
|---|---|---|---|
| `POST https://host/path` | source-map | status transition with tenant ID | Phase 02 workflow and IDOR eligibility |

## Blocked, retry when conditions change
Anything that failed for a reason that might not hold next time.
- Auth-gated tests that were skipped because ACCESS MODE was UNAUTH. State which classes.
- Endpoints behind a paywall, KYC wall, or a plan we could not reach.
- A WAF or bot wall that blocked a class outright, and which vendor.
- A rate limit that made a race or brute-force class untestable.

## Partial findings worth another pass
From `hunt/` files with Status `PARTIAL`. These are half-proven and cheap to finish.
| Angle file | What is proven | What is missing |
|---|---|---|

## Chains not attempted
Combinations Stage C identified but could not complete, and why.

## Notes for the goal cycles
Anything a fresh agent would waste time rediscovering: quirks of the auth flow, endpoints
that look interesting and are not, a host that always 403s, a parameter that silently
ignores input. Dead ends belong here as much as leads.
```

The last section matters more than it looks. A goal cycle that rediscovers the same dead end burns hours you already spent.

## Step 5: Write the ledger

Write `hunt/CLOSEOUT.md`.

Not `hunt/phase-03-*.md`. The sweep's angle files run `phase-02-<slug>`, `phase-03-<slug>` and upward, so anything this phase writes into that numbered range collides with one of them. `CLOSEOUT.md` and `RECON-NOTES.md` are reserved non-numbered names for exactly this reason.

```markdown
# Close-out, $TARGET
Date: {date}
Access mode: {RICH | PARTIAL | UNAUTH}
Closed out by: {$YOUR_STAMP}

## Coverage
| Metric | Value |
|---|---|
| Shapes in catalog | {TOTAL_N} |
| Shapes with zero tests | {UNTESTED_N} |
| Untested on a Tier 1 host | {n} |
| Endpoint classes terminal | {n} of 38 |
| Endpoint classes fully complete | {n} of 38 |
| Coverage matrix terminal rows | {n} of 47 |
| Coverage matrix fully complete rows | {n} of 47 |
| Coverage matrix validation | {PASS | INCOMPLETE; see hunt/phase-02-coverage.md} |
| Cross-phase angle ledger | {N current angles; M open or follow-up} |
| High-value orphan operations | {n; clean completion requires 0} |
| Identity lane | {dual | single | unauth | INVALID} |
| Disruptive tail | {completed | partial | not-applicable | policy-excluded | INVALID} |
| Classes with an eligible remainder | {list} |
| Classes missing eligibility review | {list, or none} |

## Reports
| ID | Title | Severity | Component findings | File |
|---|---|---|---|---|
| REPORT-01 | {title} | HIGH | phase-02-sweep, L07 | reports/REPORT-01.md |

Severity distribution: Critical {n}, High {n}, Medium {n}, Low {n}, Info {n}.

**Not triaged here.** These go to P06 for the So-What gate and to P09 for live
re-reproduction. Counts above are what the sweep produced, not what will survive.

## Report health
| Check | Result |
|---|---|
| Malformed (missing sections or unparseable severity) | {n, fixed / listed} |
| Duplicates merged | {n} |
| Needs evidence before triage | {list, or none} |
| Missing provenance | {n, recovered from exact source metadata / listed} |
| Severity internally inconsistent | {list, or none} |

## Stage C
{Ran in Phase 02 against current digest | Reran in Phase 03 after report repair, N chains recorded, M escalations | Missing or stale, gap noted}

## Angle files
{n} total. Status distribution: CONFIRMED {n}, PARTIAL {n}, DEAD {n}, SKIPPED {n}.

The DEAD count is the number the goal cycles will not have to re-test. It is a
deliverable, not a failure count.

## Sensitive findings
{N entries in sensitive/SENSITIVE-FINDINGS.md; M elevated to a report; remainder
supporting evidence or credentials.}

## Cleanup
- Profiles expected from access contract: A p{A or none}, B p{B or none}
- Phase 02 listener cleanup: {VERIFIED | FAILED}; evidence `raw/phase02-oast-cleanup.txt` and `raw/phase02-owned-listeners.md` when present
- Browser profile release: PENDING, finalized in Step 6
- Release evidence: `raw/closeout-profile-release.txt` and `raw/closeout-profile-list.txt`
- mitm flows preserved at `$YOUR_BROWSER_FLOW_DIR/p{N}/current.flow`; warmed profile storage is not wiped

## Handoff
`hunt/RECON-NOTES.md` lists untested Tier 1 shapes, pending classes, blocked retries,
partial findings, and dead ends the goal cycles should not repeat.

Also include the latest `python3 "$YOUR_HELPERS_ROOT/coverage-angles.py" --target "$YOUR_TARGET_ROOT" summary` output or its path. Preserve open angle attempt IDs, exact keys, evidence paths, and `next_test` values. A later phase may retest an exact angle only with a documented `--retest-reason`; different angle keys remain eligible.
```

## Step 6: Release the browser profiles

**Last operational action.** First recover cleanup for any target-owned Phase 02 listener left by an interrupted sweep. Then release active Chrome and mitm services and return leases to the pool. This does not wipe warmed profile cookies or storage, and it does not delete captured flows.

```bash
: "${YOUR_WORKSPACE_ROOT:?runner did not provide isolated workspace}"
: "${YOUR_RUN_ID:?runner did not provide run identity}"
BASE="$YOUR_WORKSPACE_ROOT"
TARGET="{{target_norm}}"
cd "$YOUR_TARGET_ROOT"
R="$YOUR_TARGET_ROOT/raw"
ACCESS_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_RUN_ID:0:12}-access:"
SWEEP_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_RUN_ID:0:12}-sweep:"
REPORT_VALIDATOR=$YOUR_HELPERS_ROOT/phase-03-closeout/validate-reports.py
MISSING_REPORTS=$YOUR_HELPERS_ROOT/phase-03-closeout/find-missing-reports.py
COVERAGE_MATRIX=$YOUR_HELPERS_ROOT/phase-02-sweep/coverage_matrix.py
ORPHAN_AUDIT=$YOUR_HELPERS_ROOT/phase-03-closeout/audit-orphans.py
PRE_RELEASE_OK=1

# Phase 02 normally stops its own <YOUR_OOB_TOOL> process. Recover safely after an
# interrupted sweep, but never kill a PID whose command does not match target.
OAST_PID_FILE="$R/phase02-oast.pid"
OAST_CLEANUP="$R/phase02-oast-cleanup.txt"
: > "$OAST_CLEANUP"
if [ -s "$OAST_PID_FILE" ]; then
  OAST_PID=$(head -n 1 "$OAST_PID_FILE")
  OAST_PID_VALID=1
  case "$OAST_PID" in
    ''|*[!0-9]*)
      echo "FAILED: invalid <YOUR_OOB_TOOL> PID file" | tee -a "$OAST_CLEANUP"
      PRE_RELEASE_OK=0
      OAST_PID_VALID=0
      OAST_PID=""
      ;;
  esac
  if [ -n "$OAST_PID" ] && kill -0 "$OAST_PID" 2>/dev/null; then
    OAST_CMD=$(tr '\0' ' ' < "/proc/$OAST_PID/cmdline" 2>/dev/null)
    case "$OAST_CMD" in
      *<YOUR_OOB_TOOL>-client*"$YOUR_TEMP_ROOT/oast-interactions.json"*)
        kill -TERM "$OAST_PID"
        for _ in 1 2 3 4 5; do
          kill -0 "$OAST_PID" 2>/dev/null || break
          sleep 1
        done
        ;;
      *)
        echo "FAILED: PID $OAST_PID does not match this target: $OAST_CMD" | tee -a "$OAST_CLEANUP"
        PRE_RELEASE_OK=0
        ;;
    esac
  fi
  if [ "$OAST_PID_VALID" -eq 1 ] && kill -0 "$OAST_PID" 2>/dev/null; then
    echo "FAILED: target <YOUR_OOB_TOOL> PID $OAST_PID remains active" | tee -a "$OAST_CLEANUP"
    PRE_RELEASE_OK=0
  elif [ "$OAST_PID_VALID" -eq 1 ]; then
    echo "VERIFIED: target <YOUR_OOB_TOOL> PID ${OAST_PID:-unknown} is stopped" | tee -a "$OAST_CLEANUP"
  fi
else
  echo "VERIFIED: no <YOUR_OOB_TOOL> PID was recorded" | tee -a "$OAST_CLEANUP"
fi
if [ -s "$R/phase02-owned-listeners.md" ] \
   && grep -qE '^ACTIVE[[:space:]]*\|' "$R/phase02-owned-listeners.md"; then
  echo "ERROR: target-owned listener cleanup remains ACTIVE"
  PRE_RELEASE_OK=0
fi

# Do not release until every non-cleanup completion gate is already satisfied.
[ -s hunt/RECON-NOTES.md ] || { echo "ERROR: RECON-NOTES.md missing"; PRE_RELEASE_OK=0; }
[ -s hunt/CLOSEOUT.md ] || { echo "ERROR: CLOSEOUT.md missing"; PRE_RELEASE_OK=0; }
if ! python3 "$REPORT_VALIDATOR" reports > "$R/closeout-report-validation.txt"; then
  cat "$R/closeout-report-validation.txt"
  PRE_RELEASE_OK=0
fi
if ! python3 "$MISSING_REPORTS" . > "$R/closeout-missing-reports.txt"; then
  cat "$R/closeout-missing-reports.txt"
  PRE_RELEASE_OK=0
fi
if ! python3 "$COVERAGE_MATRIX" . validate > "$R/closeout-coverage-validation.txt"; then
  cat "$R/closeout-coverage-validation.txt"
  PRE_RELEASE_OK=0
fi
if ! python3 "$ORPHAN_AUDIT" . validate > "$R/closeout-orphan-audit.txt"; then
  cat "$R/closeout-orphan-audit.txt"
  PRE_RELEASE_OK=0
fi
if ! python3 "$MOBILE_HANDOFF_HELPER" validate > "$R/closeout-mobile-handoff-validation.txt"; then
  echo "ERROR: imported mobile hypotheses are not terminal"
  cat "$R/closeout-mobile-handoff-validation.txt"
  PRE_RELEASE_OK=0
fi
if ! jq -e '.identity_lane == "dual" or .identity_lane == "single" or .identity_lane == "unauth"' \
    "$R/phase02-identity-lane.json" >/dev/null 2>&1; then
  echo "ERROR: invalid Phase 02 identity lane record"
  PRE_RELEASE_OK=0
fi
if ! jq -e '
  (.status == "completed" or .status == "partial" or .status == "not-applicable" or .status == "policy-excluded")
  and (.reason | length > 0)
  and (.checks | type == "array" and length > 0)
  and all(.checks[];
    ((.name | type) == "string" and (.name | length) > 0)
    and (.status == "completed" or .status == "partial" or .status == "not-applicable" or .status == "policy-excluded")
    and ((.reason // "") | length > 0))
  and (if .status == "completed" then all(.checks[]; (.applicable != true) or .status == "completed") else true end)
  and (if .identity_lane == "dual" then
    any(.checks[]; .name == "password-email-change")
    and any(.checks[]; .name == "reset-completion")
    and (((.reason // "") | test("preserv.*(closeout|close-out)"; "i")) | not)
    and all(.checks[]; (((.reason // "") | test("preserv.*(closeout|close-out)"; "i")) | not))
  else true end)
' \
    "$R/phase02-disruptive-tail.json" >/dev/null 2>&1; then
  echo "ERROR: invalid Phase 02 disruptive tail record"
  PRE_RELEASE_OK=0
fi
CURRENT_REPORTS=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' | wc -l)
CURRENT_REPORT_SHA256=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' -print0 \
  | sort -z | xargs -0 -r sha256sum | sha256sum | awk '{print $1}')
if ! jq -e --argjson current "$CURRENT_REPORTS" --arg digest "$CURRENT_REPORT_SHA256" '
  (.phase=="phase-02-sweep" or .phase=="phase-03-closeout")
  and .all_confirmed_reviewed==true
  and .report_count==$current
  and .report_sha256==$digest
' "$R/stage-c-complete.json" >/dev/null 2>&1; then
  echo "ERROR: Stage C does not cover current report digest"
  PRE_RELEASE_OK=0
fi
if [ "$PRE_RELEASE_OK" -ne 1 ]; then
  $YOUR_HUNT_BIN/hunt-intervention wait \
    --platform '{{platform}}' --handle '{{handle}}' \
    --target '{{target_norm}}' --phase phase-03-closeout \
    --title 'Close-out blocked | pre-release gate failed' \
    --action 'Finish ledgers and resolve report validation before releasing profiles'
  echo "Profiles not released; fix close-out artifacts first"
  exit 1
fi

: > "$R/closeout-profile-release.txt"
RELEASE_OK=1
for prefix in "$ACCESS_PREFIX" "$SWEEP_PREFIX"; do
  if ! sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND release-prefix "$prefix" \
      >> "$R/closeout-profile-release.txt" 2>&1; then
    RELEASE_OK=0
  fi
done
if ! sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND list > "$R/closeout-profile-list.txt" 2>&1; then
  echo "ERROR: could not verify browser profile list" | tee -a "$R/closeout-profile-release.txt"
  RELEASE_OK=0
fi
cat "$R/closeout-profile-release.txt"

if grep -F "$ACCESS_PREFIX" "$R/closeout-profile-list.txt" >/dev/null \
   || grep -F "$SWEEP_PREFIX" "$R/closeout-profile-list.txt" >/dev/null; then
  echo "ERROR: one or more target leases remain" | tee -a "$R/closeout-profile-release.txt"
  RELEASE_OK=0
fi

if [ "$RELEASE_OK" -ne 1 ]; then
  $YOUR_HUNT_BIN/hunt-intervention wait \
    --platform '{{platform}}' --handle '{{handle}}' \
    --target '{{target_norm}}' --phase phase-03-closeout \
    --title 'Close-out blocked | browser lease release failed' \
    --action 'Review raw/closeout-profile-release.txt and release remaining target leases'
  echo "Browser profile release: FAILED. Do not stamp phase finish."
else
  echo "Browser profile release: VERIFIED"
fi
```

That covers both the profiles Phase 1 leased for accounts A and B, and any profile the sweep acquired for itself in UNAUTH mode. Release by prefix, never by slot number: another target may be holding the adjacent slot.

After successful verification, edit the Phase 02 listener and browser release cleanup lines in `hunt/CLOSEOUT.md` to `VERIFIED`. On failure, change the affected line to `FAILED` and stop. Do not perform more hunting, testing, or browsing after release.

---

## What NOT to do
- Do NOT triage. No So-What test, no REPORT / RECON NOTE / DROP buckets, no severity recalibration. P06 and P09 own all of it and both do it better with fresh eyes.
- Do NOT rewrite a weak Impact section into a stronger claim. Flag it and move on.
- Do NOT delete or downgrade a report. If it is wrong, downstream will say so, and a report deleted here leaves no record that the angle was ever worked.
- Do NOT stamp a finding as Phase 03. Recover original Phase 02 attribution from recorded metadata or leave it blocked.
- Do NOT test anything. An untested Tier 1 shape is a note, not an invitation.
- Do NOT manufacture Stage C proof when sweep skipped it and closeout did not change reports. If closeout changes report content, rerun Stage C against existing evidence as required above; never send new target traffic.
- Do NOT release profiles before `CLOSEOUT.md` and `RECON-NOTES.md` are both written.
- Do NOT write the ledger as `hunt/phase-NN-*.md`. That range belongs to the sweep's angle files.
- Do NOT release by slot number, only by owner prefix.
- Do NOT delete the mitm flows or anything under `raw/`. Later phases read both.

---

## Phase complete: stamp finish time

```bash
# Hard completion gates. Failure means notify, fix the close-out artifact, and do
# not call phase-runs/finish.
: "${YOUR_WORKSPACE_ROOT:?runner did not provide isolated workspace}"
: "${YOUR_RUN_ID:?runner did not provide run identity}"
BASE="$YOUR_WORKSPACE_ROOT"
TARGET="{{target_norm}}"
cd "$YOUR_TARGET_ROOT"
R="$YOUR_TARGET_ROOT/raw"
EP=$YOUR_HELPERS_ROOT/bin/endpoints
REPORT_VALIDATOR=$YOUR_HELPERS_ROOT/phase-03-closeout/validate-reports.py
MISSING_REPORTS=$YOUR_HELPERS_ROOT/phase-03-closeout/find-missing-reports.py
COVERAGE_MATRIX=$YOUR_HELPERS_ROOT/phase-02-sweep/coverage_matrix.py
ORPHAN_AUDIT=$YOUR_HELPERS_ROOT/phase-03-closeout/audit-orphans.py
ACCESS_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_RUN_ID:0:12}-access:"
SWEEP_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_RUN_ID:0:12}-sweep:"
FINAL_OK=1
[ -s hunt/RECON-NOTES.md ] || { echo "ERROR: RECON-NOTES.md missing"; FINAL_OK=0; }
[ -s hunt/CLOSEOUT.md ] || { echo "ERROR: CLOSEOUT.md missing"; FINAL_OK=0; }
[ -s "$R/closeout-profile-release.txt" ] || { echo "ERROR: profile release evidence missing"; FINAL_OK=0; }
[ -s "$R/phase02-oast-cleanup.txt" ] || { echo "ERROR: Phase 02 listener cleanup evidence missing"; FINAL_OK=0; }
grep -qF 'VERIFIED:' "$R/phase02-oast-cleanup.txt" \
  || { echo "ERROR: Phase 02 <YOUR_OOB_TOOL> cleanup not verified"; FINAL_OK=0; }
if [ -s "$R/phase02-owned-listeners.md" ] \
   && grep -qE '^ACTIVE[[:space:]]*\|' "$R/phase02-owned-listeners.md"; then
  echo "ERROR: target-owned listener cleanup remains ACTIVE"
  FINAL_OK=0
fi

if ! python3 "$REPORT_VALIDATOR" reports > "$R/closeout-report-validation.txt"; then
  cat "$R/closeout-report-validation.txt"
  FINAL_OK=0
fi
if ! python3 "$MISSING_REPORTS" . > "$R/closeout-missing-reports.txt"; then
  echo "ERROR: confirmed medium-or-above findings still lack reports"
  cat "$R/closeout-missing-reports.txt"
  FINAL_OK=0
fi
if ! python3 "$COVERAGE_MATRIX" . validate > "$R/closeout-coverage-validation.txt"; then
  cat "$R/closeout-coverage-validation.txt"
  FINAL_OK=0
fi
if ! python3 "$ORPHAN_AUDIT" . validate > "$R/closeout-orphan-audit.txt"; then
  cat "$R/closeout-orphan-audit.txt"
  FINAL_OK=0
fi
if ! jq -e '.identity_lane == "dual" or .identity_lane == "single" or .identity_lane == "unauth"' \
    "$R/phase02-identity-lane.json" >/dev/null 2>&1; then
  echo "ERROR: invalid Phase 02 identity lane record"
  FINAL_OK=0
fi
if ! jq -e '
  (.status == "completed" or .status == "partial" or .status == "not-applicable" or .status == "policy-excluded")
  and (.reason | length > 0)
  and (.checks | type == "array" and length > 0)
  and all(.checks[];
    ((.name | type) == "string" and (.name | length) > 0)
    and (.status == "completed" or .status == "partial" or .status == "not-applicable" or .status == "policy-excluded")
    and ((.reason // "") | length > 0))
  and (if .status == "completed" then all(.checks[]; (.applicable != true) or .status == "completed") else true end)
  and (if .identity_lane == "dual" then
    any(.checks[]; .name == "password-email-change")
    and any(.checks[]; .name == "reset-completion")
    and (((.reason // "") | test("preserv.*(closeout|close-out)"; "i")) | not)
    and all(.checks[]; (((.reason // "") | test("preserv.*(closeout|close-out)"; "i")) | not))
  else true end)
' \
    "$R/phase02-disruptive-tail.json" >/dev/null 2>&1; then
  echo "ERROR: invalid Phase 02 disruptive tail record"
  FINAL_OK=0
fi
CURRENT_REPORTS=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' | wc -l)
CURRENT_REPORT_SHA256=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' -print0 \
  | sort -z | xargs -0 -r sha256sum | sha256sum | awk '{print $1}')
if ! jq -e --argjson current "$CURRENT_REPORTS" --arg digest "$CURRENT_REPORT_SHA256" '
  (.phase=="phase-02-sweep" or .phase=="phase-03-closeout")
  and .all_confirmed_reviewed==true
  and .report_count==$current
  and .report_sha256==$digest
' "$R/stage-c-complete.json" >/dev/null 2>&1; then
  echo "ERROR: Stage C does not cover current report digest"
  FINAL_OK=0
fi
if ! sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND list > "$R/closeout-profile-list.txt" 2>&1; then
  echo "ERROR: post-release profile list failed"
  FINAL_OK=0
fi
[ -s "$R/closeout-profile-list.txt" ] || { echo "ERROR: post-release profile list missing"; FINAL_OK=0; }
if grep -F "$ACCESS_PREFIX" "$R/closeout-profile-list.txt" >/dev/null \
   || grep -F "$SWEEP_PREFIX" "$R/closeout-profile-list.txt" >/dev/null; then
  echo "ERROR: browser profile release not verified"
  FINAL_OK=0
fi
grep -qF 'Browser profile release: VERIFIED' hunt/CLOSEOUT.md \
  || { echo "ERROR: CLOSEOUT.md cleanup status not finalized"; FINAL_OK=0; }

if [ "$FINAL_OK" -ne 1 ]; then
  $YOUR_HUNT_BIN/hunt-intervention wait \
    --platform '{{platform}}' --handle '{{handle}}' \
    --target '{{target_norm}}' --phase phase-03-closeout \
    --title 'Close-out incomplete | hard gate failed' \
    --action 'Review closeout validation files and finish required artifacts'
  echo "Phase 03 not finished"
  exit 1
fi

# Nothing has appended to the catalog since Step 1, so no rebuild is needed.
$EP stats
ls reports/*.md 2>/dev/null | wc -l | xargs echo "reports:"
for f in hunt/phase-*.md; do
  [ -f "$f" ] || continue
  tail -5 "$f" | grep -qE 'Status: (DEAD|PARTIAL|SKIPPED|CONFIRMED)|^(DEAD|PARTIAL|SKIPPED|CONFIRMED-[A-Z]+)$' && echo "$f"
done | wc -l | xargs echo "angle files with terminal status:"
echo "RECON-NOTES.md written"
echo "CLOSEOUT.md written"
echo "browser profile release verified"

$YOUR_HUNT_BIN/hunt-phase-event finish \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-03-closeout true
```
