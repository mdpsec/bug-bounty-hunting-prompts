> [!IMPORTANT]
> Public reference prompt. Private helpers, infrastructure, RAG, accounts, and orchestration are not included. Replace every `$YOUR_*`, `<YOUR_*>`, and project-specific command before use. See [ADAPTATION.md](../ADAPTATION.md). Never commit secrets.

# Phase 2: Sweep

Cross-phase coverage: read `$YOUR_HELPERS_ROOT/coverage-angles.md` and use its shared angle ledger for each concrete vulnerability hypothesis. Before testing a class, load the matching local technique ref and, when a historical shape helps, the relevant `hunt-examples-INDEX.md` section. Record the exact files consulted and retrieval status on the angle. Keep existing endpoint test events and the 47-row matrix as Phase 02 completion authority. A prior class verdict does not close an untested mechanism, identity, or method.

One contiguous session with subagent fan-out. You work the endpoint catalog by class, running session-bound classes yourself and delegating stateless ones, until every shape carries a verdict.

## Resources
Read `$YOUR_RESOURCE_GUIDE`. OOB domain, hosted callback, email accounts, browser manager, crypto wallets, and installed tools.

**Read `$YOUR_HELPERS_ROOT/report-format.md` now.** It is the canonical public workflow format for `reports/` and `SENSITIVE-FINDINGS.md`. Use its exact headings and provenance rules for every finding written in this phase. Do not wait until the first confirmed finding to load it.

**Technique detail lives in the refs, not here.** `$YOUR_REFERENCE_ROOT/ref-<class>-techniques.md` is how to test each class: infrastructure, ssrf, access-control, auth, business-logic, injection, file-upload, graphql, xxe-deserialization. For parser, proxy, normalization, lifecycle, or cross-runtime differentials, also load `$YOUR_REFERENCE_ROOT/ref-semantic-confusion-techniques.md`. For a confirmed Next.js target, also load `$YOUR_REFERENCE_ROOT/ref-nextjs-runtime-techniques.md`. For browser-dependent classes (`domxss`, `oauth`, `cache`, `csrf`, or postMessage), also load `$YOUR_REFERENCE_ROOT/ref-browser-state-techniques.md`. `hunt-examples-INDEX.md` is the historical report corpus by surface, including rejected reports. Check each entry's Status and triage activity before reusing its claim. Skim the relevant class ref before working that class; skim the INDEX when stuck.

**Coverage matrix helper:** `$YOUR_HELPERS_ROOT/phase-02-sweep/coverage_matrix.py`. It renders the 47-row human checklist at `hunt/phase-02-coverage.md` from machine-readable eligibility, test events, and target-scoped records. The matrix is not self-attested prose; helper validation decides whether Phase 02 may finish.

## Target
- **Domain**: {{target}}
- **Scope**: {{scope}}

## Your Role: orchestrator

**Give every eligible endpoint shape a relevant verdict, and file what confirms.**

This is a coverage phase, not a bug-count phase. The goal cycles that follow chase depth on whatever looks promising and leave the map unannotated. Your job is the opposite: breadth, with relevant tests recorded across the mapped surface, so the cycles start from a map that says what has already been tried and what died. Do not multiply every class across every static asset or obviously unrelated route.

### Operational target surface

`raw/scope-hosts.txt` is the canonical host list for this target. It includes the listed asset plus evidence-backed first-party functional siblings that the application directly uses for signup, login, account management, application UI, API, or billing. Test those siblings normally. `account.X`, `auth.X`, or `api.X` is not out of scope merely because program display text names `www.X` when the listed application routes normal product traffic there.

`raw/functional-sibling-hosts.tsv` supplies role and evidence for additions. Do not broaden from passive subdomain names. Third-party IdPs, captcha, analytics, CDN, support SaaS, and payment providers remain external. An explicit program exclusion naming a host overrides the local list.

You are the single controller for every browser and account. Phase 1 may have left zero, one, or two authenticated identities. Derive the lane from actual auth state, not from the mode label alone: two identities enable bidirectional cross-account work, one identity enables single-account work, and zero identities route to UNAUTH coverage. Everything independent and stateless goes to background subagents while you keep controller-only work serial.

**Browser lifecycle contract:** successful Phase 1 profiles remain leased under
their exact `...-access:` owners until Phase 3 Closeout. Never start a profile
with direct `systemctl`, and never run bare or untagged `browser-profile-command acquire`,
including against a known profile number. Do not repair, resume, or relabel an
unexpectedly inactive Phase 1 profile. Use saved auth for bounded request replay
where possible, mark browser-dependent coverage partial, and continue. Every new
lease allowed by this prompt must use `browser-lease.py` with a run-scoped owner.

### Route on ACCESS MODE

Read it from `hunt/phase-01-access.md`. It is the first line under `## ACCESS MODE`.

| Mode | Controller lane | Stateless subagent lane |
|---|---|---|
| **RICH** | Normally two identities. Verify both, then run bidirectional cross-account and single-account work. | Full |
| **PARTIAL** | Derive from auth files: two identities run bidirectional work, one runs single-account work, zero routes to UNAUTH behavior. | Full, filtered by available auth |
| **UNAUTH** | No account work. Run browser-safe DOM XSS, postMessage, open redirect confirmation, and cache deception with a clean profile. | Full and reprioritised. See UNAUTH mode below. |

UNAUTH is a productive mode, not a degraded one. 31 submitted findings including 13 criticals came from programs where no account ever existed. Do not treat it as a thin run.

## Phase tracking

Run this first.

```bash
$YOUR_HUNT_BIN/hunt-phase-event start \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-02-sweep true
```

## User intervention notify

Fire before asking the user, then ask and wait. Only you fire it; subagents have no user channel.

```bash
$YOUR_HUNT_BIN/hunt-intervention wait \
  --platform '{{platform}}' --handle '{{handle}}' \
  --target '{{target_norm}}' --phase phase-02-sweep \
  --title '<blocked task | cause and evidence, max 120 chars>' \
  --action '<specific user action, max 120 chars>'
```

Discussion does not resolve a wait. When user input gives you an actionable next attempt, run `$YOUR_HUNT_BIN/hunt-intervention continue` immediately before resuming tools. If still blocked, fire `wait` again. Each round pauses only this phase's execution limit while wall and wait time continue recording.

## Setup

```bash
set +u
: "${ACCESS_MODE:=UNAUTH}"
: "${PROFILE_A:=}" "${PROFILE_B:=}" "${CDP_A:=}" "${CDP_B:=}"

: "${YOUR_WORKSPACE_ROOT:?runner did not provide isolated workspace}"
: "${YOUR_TARGET_ROOT:?runner did not provide target root}"
: "${YOUR_HELPERS_ROOT:?runner did not provide frozen helpers}"
[ "$YOUR_TARGET_ROOT" = "$YOUR_WORKSPACE_ROOT" ] || { echo "ERROR: isolated target root mismatch"; exit 1; }
BASE="$YOUR_WORKSPACE_ROOT"
TARGET="{{target_norm}}"
APEX="$TARGET"; APEX="${APEX#wild.}"
export PHASE=phase-02-sweep
EP=$YOUR_HELPERS_ROOT/bin/endpoints
cd "$YOUR_WORKSPACE_ROOT"
R="$YOUR_TARGET_ROOT/raw"
mkdir -p $R/responses hunt reports sensitive
SCOPE_HOSTS_FILE="$R/scope-hosts.txt"
[ -s "$SCOPE_HOSTS_FILE" ] || { echo "ERROR: missing canonical scope hosts: $SCOPE_HOSTS_FILE"; exit 1; }
COVERAGE_MATRIX=$YOUR_HELPERS_ROOT/phase-02-sweep/coverage_matrix.py
OP_CANDIDATE_HELPER=$YOUR_HELPERS_ROOT/operation-candidates.py
API_CONTRACT_HELPER=$YOUR_HELPERS_ROOT/api-contract.py
THREAT_MODEL_HELPER=$YOUR_HELPERS_ROOT/threat-model.py
FLOW_REPLAY_HELPER=$YOUR_HELPERS_ROOT/bin/flow-replay.py
RESOURCE_GRAPH_VALIDATOR=$YOUR_HELPERS_ROOT/phase-01-access/validate-resource-graph.py
ORPHAN_AUDIT=$YOUR_HELPERS_ROOT/phase-03-closeout/audit-orphans.py
[ -r "$COVERAGE_MATRIX" ] || { echo "ERROR: missing Phase 02 coverage helper: $COVERAGE_MATRIX"; exit 1; }
[ -r "$OP_CANDIDATE_HELPER" ] || { echo "ERROR: missing operation candidate helper"; exit 1; }
[ -r "$API_CONTRACT_HELPER" ] || { echo "ERROR: missing API contract helper"; exit 1; }
[ -r "$THREAT_MODEL_HELPER" ] || { echo "ERROR: missing threat model helper"; exit 1; }
[ -r "$FLOW_REPLAY_HELPER" ] || { echo "ERROR: missing flow replay helper"; exit 1; }
[ -r "$RESOURCE_GRAPH_VALIDATOR" ] || { echo "ERROR: missing resource graph validator"; exit 1; }
[ -r "$ORPHAN_AUDIT" ] || { echo "ERROR: missing operation orphan audit"; exit 1; }
echo "Operational target hosts:"
cat "$SCOPE_HOSTS_FILE"

# Static, documented, browser, and mitm candidates are catalog inputs. Validate
# Phase 01 graph before routing so placeholders and silent prerequisite gaps do
# not become false coverage.
for spec in "$R"/api-specs/*.json "$R"/api-specs/*.yaml "$R"/api-specs/*.yml; do
  [ -f "$spec" ] || continue
  python3 "$API_CONTRACT_HELPER" . --spec "$spec" \
    || printf '%s\n' "$spec" >> "$R/api-contract-import-errors.txt"
done
if [ -s "$R/api-contract-import-errors.txt" ]; then
  echo "API contract imports needing review:"
  sed -n '1,20p' "$R/api-contract-import-errors.txt"
fi
python3 "$OP_CANDIDATE_HELPER" . seed || exit 1
$EP rebuild
python3 "$RESOURCE_GRAPH_VALIDATOR" . || exit 1

MOBILE_HANDOFF_HELPER=$YOUR_HELPERS_ROOT/mobile-handoff-results.py
[ -r "$MOBILE_HANDOFF_HELPER" ] || { echo "ERROR: missing mobile handoff result helper"; exit 1; }
MOBILE_HANDOFF_COUNT=$(python3 "$MOBILE_HANDOFF_HELPER" list | tee "$R/mobile-handoff-work.jsonl" | wc -l)
echo "Imported mobile validation hypotheses: $MOBILE_HANDOFF_COUNT"

# Read the access contract. These drive everything below.
ACC=hunt/phase-01-access.md
ACCESS_MODE=$(awk '/^## ACCESS MODE/{getline; while($0 ~ /^$/) getline; print $1; exit}' "$ACC")
PROFILE_A=$(awk -F'|' '/^\| A \|/{gsub(/[^0-9]/,"",$6); print $6; exit}' "$ACC")
PROFILE_B=$(awk -F'|' '/^\| B \|/{gsub(/[^0-9]/,"",$6); print $6; exit}' "$ACC")
CDP_A=$((YOUR_CDP_BASE_PORT + ${PROFILE_A:-0})); CDP_B=$((YOUR_CDP_BASE_PORT + ${PROFILE_B:-0}))
[ -n "$ACCESS_MODE" ] || { echo "ERROR: ACCESS MODE missing from $ACC"; exit 1; }
python3 "$THREAT_MODEL_HELPER" . derive --access-mode "$ACCESS_MODE" --phase phase-02-sweep
python3 "$THREAT_MODEL_HELPER" . validate || exit 1

AUTH_A_OK=$(jq -r '.authenticated // false' "$R/auth-a.json" 2>/dev/null || echo false)
AUTH_B_OK=$(jq -r '.authenticated // false' "$R/auth-b.json" 2>/dev/null || echo false)
ACCESS_LEASE_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_RUN_ID:0:12}-access:"
lease_matches_access() {
  local profile=$1 lease owner
  lease="$YOUR_BROWSER_LEASE_DIR/$profile"
  [ -r "$lease" ] || return 1
  IFS=$'\t' read -r _ owner < "$lease"
  [[ "$owner" == "$ACCESS_LEASE_PREFIX"* ]]
}
LEASE_A_OK=false; LEASE_B_OK=false
[ "${PROFILE_A:-0}" -gt 0 ] 2>/dev/null && lease_matches_access "$PROFILE_A" && LEASE_A_OK=true
[ "${PROFILE_B:-0}" -gt 0 ] 2>/dev/null && lease_matches_access "$PROFILE_B" && LEASE_B_OK=true
LIVE_A=false; LIVE_B=false
if [ "$AUTH_A_OK" = true ] && [ "${PROFILE_A:-0}" -gt 0 ] 2>/dev/null \
  && [ "$LEASE_A_OK" = true ] \
  && curl -fsS "http://$YOUR_CDP_HOST:$CDP_A/json/version" >/dev/null 2>&1; then LIVE_A=true; fi
if [ "$AUTH_B_OK" = true ] && [ "${PROFILE_B:-0}" -gt 0 ] 2>/dev/null \
  && [ "$LEASE_B_OK" = true ] \
  && curl -fsS "http://$YOUR_CDP_HOST:$CDP_B/json/version" >/dev/null 2>&1; then LIVE_B=true; fi

IDENTITY_COUNT=0
[ "$AUTH_A_OK" = true ] && IDENTITY_COUNT=$((IDENTITY_COUNT + 1))
[ "$AUTH_B_OK" = true ] && IDENTITY_COUNT=$((IDENTITY_COUNT + 1))
if [ "$ACCESS_MODE" = UNAUTH ]; then
  IDENTITY_LANE=unauth
elif [ "$IDENTITY_COUNT" -ge 2 ]; then
  IDENTITY_LANE=dual
elif [ "$IDENTITY_COUNT" -eq 1 ]; then
  IDENTITY_LANE=single
else
  IDENTITY_LANE=unauth
fi

echo "ACCESS_MODE=$ACCESS_MODE  IDENTITY_LANE=$IDENTITY_LANE"
echo "A: auth=$AUTH_A_OK lease=$LEASE_A_OK live=$LIVE_A p${PROFILE_A:-none} CDP=$CDP_A"
echo "B: auth=$AUTH_B_OK lease=$LEASE_B_OK live=$LIVE_B p${PROFILE_B:-none} CDP=$CDP_B"
jq -n \
  --arg access_mode "$ACCESS_MODE" --arg lane "$IDENTITY_LANE" \
  --argjson auth_a "$AUTH_A_OK" --argjson auth_b "$AUTH_B_OK" \
  --argjson lease_a "$LEASE_A_OK" --argjson lease_b "$LEASE_B_OK" \
  --argjson live_a "$LIVE_A" --argjson live_b "$LIVE_B" \
  --arg profile_a "${PROFILE_A:-}" --arg profile_b "${PROFILE_B:-}" \
  '{access_mode:$access_mode,identity_lane:$lane,auth_a:$auth_a,auth_b:$auth_b,
    lease_a:$lease_a,lease_b:$lease_b,live_a:$live_a,live_b:$live_b,
    profile_a:$profile_a,profile_b:$profile_b}' \
  > "$R/phase02-identity-lane.json"

$EP stats
```

### Provenance stamp

Run this now. Every report you write gets stamped with the phase and the model that produced it, and it is detected rather than guessed.

```bash
. $YOUR_HELPERS_ROOT/bin/provenance.sh && hunt_provenance

# Preserve this phase's detected central-writer identity. Close-out may use it
# only to reconstruct a missing Phase 02 report without claiming Phase 03 origin.
jq -nc \
  --arg phase "$PHASE" \
  --arg provider "$YOUR_PROVIDER" \
  --arg model "$YOUR_MODEL" \
  --arg effort "$YOUR_EFFORT" \
  --arg recorded_at "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  '{phase:$phase,provider:$provider,model:$model,effort:$effort,recorded_at:$recorded_at}' \
  > "$R/phase02-provenance.json"

# Clear Phase 02 completion and lane proof at phase start. If this sweep stops,
# close-out must not accept records left by an earlier attempt.
rm -f "$R/stage-c-complete.json" "$R/phase02-disruptive-tail.json" \
  "$R/identity-health-before.json" "$R/identity-object-map.json"
```

```bash
# Canonical test-name vocabulary. Every endpoint-scoped class in the matrix maps to
# exactly one of these. Both the objective check and the finish check read this array,
# so there is one source of truth for what "covered" means.
TESTS=(ssrf ssti xxe deser jndi lfi redirect upload
       idor idor-x unauth-access id-walk path-bypass method-override api-version func-level graphql-authz
       jwt session oauth pwreset token-leak siwe preauth-bypass
       numeric massassign race idempotency workflow method-swap entitlement
       domxss cswsh cache header-inj second-order api-abuse
       injection)
echo "${#TESTS[@]} endpoint-scoped classes"

mkdir -p "$R/eligibility" "$R/coverage-target"
sha256sum "$COVERAGE_MATRIX" > "$R/phase02-coverage-helper-sha256.txt"
python3 "$COVERAGE_MATRIX" . render
```

First render intentionally shows `PENDING`. Re-render after completing any class or target-scoped row. Never hand-edit `hunt/phase-02-coverage.md`; helper overwrites it from active ledgers.

### OOB listener

Several Stage B classes need an out-of-band callback: SSRF, XXE, deserialization, SSTI, JNDI, blind injection. Stand it up once here and hand `$OAST_URL` to any subagent whose class needs it.

```bash
OAST_PID_FILE="$R/phase02-oast.pid"
OAST_CLEANUP="$R/phase02-oast-cleanup.txt"
mkdir -p "$YOUR_TEMP_ROOT"
<YOUR_OOB_TOOL>-client -json -o "$YOUR_TEMP_ROOT/oast-interactions.json" -sf "$YOUR_TEMP_ROOT/oast-session.json" \
  -poll-interval 3 > "$YOUR_TEMP_ROOT/oast.log" 2>&1 &
OAST_PID=$!
printf '%s\n' "$OAST_PID" > "$OAST_PID_FILE"
printf '{"pid":%s,"owner":"%s","command":"<YOUR_OOB_TOOL>-client","evidence":"%s","started_at":"%s"}\n' \
  "$OAST_PID" "$YOUR_RUN_ID" "$YOUR_TEMP_ROOT/oast-interactions.json" "$(date -u +%Y-%m-%dT%H:%M:%SZ)" >> "$YOUR_PROCESS_REGISTRY"
sleep 3
OAST_URL=$(grep -oP '[a-z0-9]+\.oast\.(pro|live|site|online|fun|me)' "$YOUR_TEMP_ROOT/oast.log" | head -1)
echo "OAST_URL=$OAST_URL"
```

Poll `$YOUR_TEMP_ROOT/oast-interactions.json` for callbacks; one line per hit. If the target's WAF blocks public <YOUR_OOB_TOOL>, switch to a VPS listener (`$YOUR_RESOURCE_GUIDE`). Use the VPS whenever you need to control the response, not just observe the callback: redirect chains, SSRF escalation to metadata, serving a crafted payload.

The VPS may listen and serve controlled responses only. Never SSH into it to run
`nmap`, `naabu`, TLS enumeration, CIDR iteration, service discovery, or any other
outbound reconnaissance. All recon commands run on `$YOUR_RESEARCH_HOST`.

Every local listener, tunnel, background process, VPS backend, and temporary vhost started by this phase is phase-owned. Record non-<YOUR_OOB_TOOL> resources in `raw/phase02-owned-listeners.md` as `ACTIVE | kind | identifier | exact teardown command`. Tear them down after the final callback poll, replace `ACTIVE` with `VERIFIED`, and append teardown evidence. Never stop a process or vhost unless its recorded identifier matches this target. Closing the model or terminal is not cleanup because background processes can detach.

If `ACCESS_MODE` comes back empty, read the contract by hand before proceeding. Do not default to UNAUTH silently: running unauth-mode on a target that has working accounts throws away the 3.5x that accounts buy on auth-required classes.

## Testing Policy

**No custom headers. No researcher-identifying headers. No program-mandated rate limits.** Non-negotiable; overrides any program scope instruction telling you otherwise.

Dependency-confusion and public package-namespace claiming are retired from public workflow. Do not enumerate candidate package names for takeover, query registry claimability, attempt to create or claim public package organizations or names, publish placeholder packages, or open findings for this class. Historical examples and helpers are reference artifacts only.

A real attacker does not send `X-Bug-Bounty: <handle>`, does not add `X-Forwarded-For: 127.0.0.1` "just in case", and does not throttle to 1 RPS because the program asked. Findings that only fire with researcher headers are known issues, not attack surface.

Auth headers (`Cookie`, `Authorization: Bearer ...`) and explicit test-payload headers (Host injection, XFF for SSRF) are the payload itself, allowed.

### Availability and account safety

Do not run generic login, password, OTP, MFA, reset, resend, or account-lockout spraying. Rate-limit absence is not a standalone finding. A bounded rate or scale check is allowed only at the end when a confirmed flaw needs repeatability to prove impact.

Identity A is the clean control. In a dual-identity lane, finish all cross-account work first, then use B for disruptive session and authentication checks. Never transfer a lockout test from B to A, rotate identities or IPs to evade a counter, or wait through a cooldown. In a single-identity lane, preserve the only account until every other authenticated check is terminal.

Stop the disruptive block at the first availability signal: `429`, `Retry-After`, remaining-attempt warning, captcha or MFA step-up, resend cooldown, unexpected `401` from previously valid state, session revocation, password-change effect, or lockout email. Record the affected class `partial` or `skipped` with exact evidence and continue work that does not need the affected identity.

403 / WAF-block on a direct request: retry that request once through the configured proxy wrapper, then once through the configured alternate proxy profile if still blocked. Do not change headers or payload. If all three paths block, document it and move on. Browser challenges remain browser work.

### Bounded non-destructive standard

Non-destructive means no deletion, lockout, denial of service, irreversible change, or spam. It does not mean read-only or own-records-only: GET, POST, PUT, PATCH, and other state-changing requests are in bounds when they are the finding's mechanism. Prove each mechanism on researcher-owned objects first (own account, record, domain), then confirm cross-entity reach on at most three non-owned objects with the smallest request that proves the boundary, then characterize enumerability without bulk collection. For writes on non-owned objects prefer reversible values, restore originals where possible, and record every touched object in evidence. Program text such as "stop testing once a flaw could modify data" means stop after this bounded proof and submit; it does not forbid the proof and never requires vendor pre-authorization.

Web and API assets only. Mobile apps are a separate pipeline even when in scope, with one exception in UNAUTH mode below.

## Inputs

1. `hunt/phase-01-access.md`: ACCESS MODE, account table, profile numbers and ports, resource graph, auth flow, authenticated route coverage, top auth-only endpoints.
2. `hunt/phase-00-recon.md` - tech stack, tier table, auth surface.
3. **`raw/endpoints.json` via the `endpoints` CLI** - the work list. This is the phase's spine.
4. `raw/recon-endpoints-merged.txt`, `raw/recon-shapes-merged.txt`, `raw/tier1-hosts.txt`.
5. `raw/js/`, `raw/source-maps-authwalk.txt`, `raw/operation-candidates.jsonl`, `raw/route-manifest-candidates.txt`, and `raw/auth-route-coverage.jsonl`: static, documented, and actually navigated candidates.
6. `raw/auth-a.json`, `raw/auth-b.json` - copied creds for subagents.
7. `hunt/account-creation-process.md` - saved login and route context only. Do not use it to re-establish, restart, resume, or relabel a dead browser session in this phase.
8. `raw/scope-hosts.txt` and `raw/functional-sibling-hosts.tsv` - canonical target hosts and evidence for functional sibling additions.
9. `raw/resource-graph.json`: A and B objects, ownership proof, reversible state, and explicit missing child prerequisites.
9a. `raw/threat-model.json`: shared attacker starting state, observed and inferred
    trust boundaries, and attributed amendments. Correct it when Phase 2 proves
    an earlier inference wrong with `threat-model.py amend --change
    '{"text":"<observed boundary correction>","evidence":["raw/<evidence>"]}'
    --phase phase-02-sweep`. Scope remains governed by program text and
    `raw/scope-hosts.txt`.
10. `raw/mobile-handoff/import-manifest.json`, `raw/mobile-handoff/packages/`,
    `raw/mobile-handoff/seeds/`, and `raw/mobile-handoff-work.jsonl` when present.
    Every imported candidate is mandatory P02 work, not optional inspiration.
11. `sensitive/SENSITIVE-FINDINGS.md` - P01 (and any earlier phase) logs secrets,
    credentials, tokens, internal URLs, and anomaly signals here with the note
    "Phase 2 owns all of it." That handoff is real: every `UNTESTED` or
    `UNKNOWN`-access secret, key, or token becomes a `P01-SECRET-<n>` lead in
    `raw/leads.jsonl`, validated with the minimal liveness check in
    `ref-exposed-secrets-techniques.md`; a confirmed live credential gets a
    normal `REPORT-NN.md`. Internal endpoints and admin URLs become endpoint
    catalog candidates. Anomaly signals that imply a bug class (mass-assignment
    flags, other-user PII, guessable verification tokens) become leads under
    that class. Logged-but-never-tested is a drop, not a disposition.

## Captured request search and bounded replay

The staged `flow-replay.py` helper searches saved mitmproxy captures and replays
one selected request. Search and export redact credential headers. Replay is
explicit, host-scoped, does not follow redirects, and requires `--confirm` plus
`--include-sensitive`.

```bash
FLOW=""
if [ "$LIVE_A" = true ] && [ -r "$YOUR_BROWSER_FLOW_DIR/p${PROFILE_A}/current.flow" ]; then
  FLOW="$YOUR_BROWSER_FLOW_DIR/p${PROFILE_A}/current.flow"
elif [ "$LIVE_B" = true ] && [ -r "$YOUR_BROWSER_FLOW_DIR/p${PROFILE_B}/current.flow" ]; then
  FLOW="$YOUR_BROWSER_FLOW_DIR/p${PROFILE_B}/current.flow"
fi
if [ -n "$FLOW" ]; then
  python3 "$FLOW_REPLAY_HELPER" "$FLOW" list --scope-hosts "$SCOPE_HOSTS_FILE" \
    --method POST --path '/api/' --limit 40
  python3 "$FLOW_REPLAY_HELPER" "$FLOW" export --index <flow-index> \
    --scope-hosts "$SCOPE_HOSTS_FILE" --output "$R/responses/replay-request.json"
  python3 "$FLOW_REPLAY_HELPER" "$FLOW" replay --index <flow-index> \
    --include-sensitive --confirm --scope-hosts "$SCOPE_HOSTS_FILE" \
    --set-json '{"body":"<small reversible test body>"}' \
    --output "$R/responses/replay-result.json"
else
  echo "No lease-verified current mitm capture is available. Skip replay and record the capture-mode limitation."
fi
```

Treat replay output as sensitive evidence. Record the flow index, operation,
modifications, response, and any restored object state in the normal endpoint
event. Replay follows the two-account, bounded cross-entity, and reversible
state rules.

## Imported mobile hypothesis lane

Process every row returned by `python3 "$MOBILE_HANDOFF_HELPER" list`. Add one
`MOBILE-<candidate_id>` entry to `raw/leads.jsonl` before work and dedupe its
operations against normal coverage. Use the P01 identities, objects, endpoint
catalog, and account recipe. Do not create a separate run or fresh account ledger.

Each candidate must finish with exactly one result:

- `confirmed`: live evidence proves medium-or-higher impact. Write a normal
  sequential `reports/REPORT-NN.md`, preserve mobile origin in its provenance,
  and record the report path.
- `disproved`: controlled live evidence disproves the exact hypothesis.
- `partial`: bounded work completed, but a technical prerequisite or proof step
  remains unavailable.
- `blocked`: an external resource such as payment, KYC, SMS, enterprise access,
  or special account state prevented validation.

Record it immediately:

```bash
python3 "$MOBILE_HANDOFF_HELPER" record <candidate-id> <confirmed|disproved|partial|blocked> \
  --evidence raw/<existing-evidence-file> \
  --report reports/REPORT-NN.md \
  --note '<exact result or blocker>'
```

Omit `--report` unless status is `confirmed`. A terminal result always needs an
existing evidence file and exact note. Disproved mobile hypotheses do not enter
`reports/`, P06, or P07.

## The objective, stated mechanically

Not "find N bugs". The phase is done when every class applicable to this access mode has been worked across every shape eligible for that class.

```bash
# `pending` is an upper bound, not the work list. It includes static assets and
# shapes with no input or state relevant to a given class.
mkdir -p "$R/eligibility"
for t in "${TESTS[@]}"; do
  $EP pending --test "$t" 2>/dev/null | sort -u > "$R/eligibility/$t-pending-all.txt"
done
```

Before dispatching a class, build `$R/eligibility/<class>-eligible.txt` from union below, then filter to canonical `scope-hosts.txt` hosts and required sink or state:

```text
endpoint catalog and browser-observed traffic
+ authenticated and unauthenticated mitm traffic
+ API documents and GraphQL operations
+ JS and source-map operation candidates
+ SPA route-manifest candidates
= candidate union before class filtering
```

`$EP pending --test <class>` is only one source. A candidate does not disappear because browser walk failed to trigger it. Examples: SSRF needs attacker-controlled URL-like input; upload needs file input or processing route; IDOR needs object reference; numeric tampering needs numeric state; session and OAuth need auth-flow routes; DOM XSS needs browser-rendered input. Static assets, health checks, unrelated GET routes, and third-party provider traffic are not eligible for every class.

Create an eligibility file and `$R/eligibility/<class>-reason.txt` for every class. For non-empty eligibility, reason file states exact sink/state rule used to include shapes. For empty eligibility, it states why class is mode-inapplicable or why target has no matching sink/state. Add same rationale to relevant angle file when one exists. Do not write `hunt/RECON-NOTES.md`; close-out owns that file. Missing eligibility or reason means class was never routed, not that it had zero work.

Every operation candidate whose `classes` array contains `<class>` must appear in that class's eligibility file. This rule is mechanical. Do not filter it out because request was static-only, browser never fired it, current identity lacks object state, or prerequisite is blocked. Those conditions produce a `partial` event for exact shape. Coverage helper rejects an empty or incomplete eligibility file when matching candidates exist.

Only a schema-valid entry in `raw/orphan-exclusions.jsonl` can exempt a candidate from eligibility. Allowed categories are exact duplicate/alias covered by another eligible or tested shape, explicit program-policy exclusion, third-party/out-of-scope host, or proven parser artifact. Each requires file-backed evidence. Static-only state, no live request, missing identity/context/object, or blocked prerequisite is never an exclusion. Phase 03 documents full schema and closeout helper enforces it.

The remaining work list for a class is intersection of pending and eligible:

```bash
comm -12 "$R/eligibility/$t-pending-all.txt" \
  <(sort -u "$R/eligibility/$t-eligible.txt") \
  > "$R/eligibility/$t-remaining.txt"
```

This keeps coverage measurable without generating `38 × every discovered URL` synthetic skip events. A genuinely eligible shape that cannot be exercised gets `partial` with exact prerequisite or challenge gap. Use `skipped` only for explicit policy exclusion, and name exclusion source in note.

Before working the list by hand, let the CLI close the cheap gaps for you:

```bash
$EP coverage-fill    # single OPTIONS + GET per shape missing methods_allowed or per-identity status
```

**Endpoint-scoped versus target-scoped.** The array above is the endpoint-scoped set: verdicts attach only to eligible shapes. The nine rows in the Infrastructure and exposure matrix are target-scoped sweeps and do not belong to one shape, so `pending` will never list them. Each row requires one `hunt/` angle file with a terminal Status, even when result is `DEAD` or mode-inapplicable `SKIPPED` with reason.

Every exact shape tested gets its own `test` event. Bind verdict to evidence for that shape:

Compute binding with `$EP shape <METHOD> <concrete-url>` before append. Copy output exactly into `evidence_shape`; do not hand-normalize IDs or query strings.

```bash
$EP append --json '{"kind":"test","method":"GET","url":"<concrete url>","test":"ssrf","result":"dead","note":"controlled URL mutation rejected","evidence":"raw/responses/ssrf-dead.json","evidence_shape":"GET https://target/path"}'
$EP append --json '{"kind":"test","method":"POST","url":"<concrete url>","test":"idor","result":"confirmed","severity":"high","report":"reports/REPORT-03.md","evidence":"raw/responses/idor-summary.json","evidence_shape":"POST https://target/path","baseline_evidence":"raw/responses/idor-owner-control.txt","attack_evidence":"raw/responses/idor-cross-account.txt","verification_evidence":"raw/responses/idor-owner-verify.txt"}'
```

`test` must use exact canonical name in `TESTS`. Do not substitute aliases such as `graphql` for `graphql-authz` or `cache-deception` for `cache`. `result` is mandatory and exactly `confirmed` | `dead` | `partial` | `skipped`; do not use `verdict`.

Evidence contract:

- `confirmed` and `dead` require existing target-relative `evidence` and exact `evidence_shape` equal to eligible shape.
- `confirmed` also requires `attack_evidence`. Require `baseline_evidence` for differential, auth, access-control, workflow, and business-logic claims. Require `verification_evidence` when claim changes state or depends on owner-side observation. Use `not-applicable: <reason>` only when that evidence role genuinely does not apply.
- One report may cover several endpoints only when every endpoint has its own event and individual reproduction evidence. Never copy one confirmed verdict across related or unrelated shapes.
- `partial` requires exact gap and existing evidence for work completed so far. Missing account, object, workflow state, browser access, or owner verification is PARTIAL, not DEAD.
- `skipped` is reserved for explicit policy exclusion and requires exclusion source. It does not count as full completion.
- `N/A` exists only at class eligibility level when no relevant sink or state exists. `PENDING` means eligible shape has no controlled terminal verdict. `DEAD` means controlled evidence disproved exact hypothesis.

**A `dead` verdict is worth as much as confirmed one** when it carries controlled evidence. Without it goal cycles re-test what you already disproved.

### Browser-backed retry for challenged API shapes

A direct `curl` response of `403`, `406`, client challenge, captcha HTML, login HTML, or unexpected HTML is not proof endpoint is dead. After network retry policy above, replay once through authenticated CDP or in-page `fetch` using correct live profile and normal application headers. Save direct response and browser-backed response. If browser replay cannot be made, mark exact shape PARTIAL with challenge evidence. Never turn challenge page into DEAD.

After a class has its eligibility file and terminal events, render the human checklist:

```bash
python3 "$COVERAGE_MATRIX" . render
```

## Class matrix

Full territory. Technique detail is in the ref, never here. `A` means controller-only because work uses a browser, an identity, or mutable state. `B` means independent and stateless, so it may be delegated. A target with no accounts still runs all applicable B work and the browser-safe unauthenticated subset of A.

### Infrastructure and exposure - `ref-infrastructure-techniques.md`, `ref-exposed-secrets-techniques.md`

**All target-scoped.** These are sweeps over the target, not verdicts on a shape, so `pending` will never list them. Each gets a `hunt/` angle file with a terminal Status. Anything they discover that *is* an endpoint gets added to the catalog and then tested per-shape by the groups below.

| Record ID | Class | Stage | Modes |
|---|---|---|---|
| `public-admin-debug` | Public admin / debug surfaces | B, target | all |
| `exposed-code-config` | Exposed code and config artefacts: source maps, `.git`, `.env`, backups | B, target | all |
| `information-disclosure` | Information disclosure (stack traces, internal hosts, version leaks) | B, target | all |
| `dns-takeover` | DNS misconfiguration, subdomain takeover | B, target | all |
| `cloud-storage` | Cloud storage misconfiguration (open buckets, writable objects) | B, target | all |
| `open-backends` | Open backend services (metrics, actuator, Hasura, Supabase, Swagger) | B, target | all |
| `bundle-secrets` | Secrets in bundles and source maps | B, target | all |
| `auth-aware-infra` | Auth-aware infrastructure: what authenticated session reveals that unauth recon could not | A, target | any authenticated identity |
| `websockets` | WebSocket discovery: enumerate every WS endpoint | B, target | all |

Each target-scoped row requires its own angle file and machine-readable record. Write record immediately after terminal conclusion, then render matrix. `status` is one of `confirmed`, `dead`, `partial`, `not-applicable`, or `policy-excluded`; `reason` and `angle` are mandatory. `not-applicable` means no relevant target sink or state. Missing mode, identity, object, or prerequisite is `partial`. `policy-excluded` requires exact program exclusion source. Neither counts as full completion.

```bash
ROW_ID="<record ID from table>"
ROW_STATUS="<confirmed|dead|partial|not-applicable|policy-excluded>"
ROW_REASON="<what was tested and terminal result, or exact blocker>"
ROW_ANGLE="hunt/phase-02-<one-row-slug>.md"
jq -n --arg status "$ROW_STATUS" --arg reason "$ROW_REASON" --arg angle "$ROW_ANGLE" \
  '{status:$status,reason:$reason,angle:$angle}' \
  > "$R/coverage-target/$ROW_ID.json"
python3 "$COVERAGE_MATRIX" . render
```

Do not combine several target rows into one angle file. One-to-one records make missing infrastructure coverage visible instead of hiding it under an aggregate `infra` verdict.

### Server-side execution - `ref-ssrf-techniques.md`, `ref-xxe-deserialization-techniques.md`, `ref-file-upload-techniques.md`

| Class | Stage | Modes |
|---|---|---|
| SSRF on any URL / webhook / integration / image-fetch field (`ssrf`) | B | all |
| SSTI (server-side template injection) (`ssti`) | B | all |
| XXE (`xxe`) | B | all |
| Deserialization (`deser`) | B | all |
| JNDI / Log4Shell (`jndi`) | B | all |
| Path traversal / LFI (`lfi`) | B | all |
| Open redirect (server-side, SSRF chain candidate) (`redirect`) | B | all |
| Upload server-side processing: thumbnailer, AV, ffmpeg, pandoc, ImageMagick (`upload`) | B | all |
| Upload surfaces, UI side: oversize, bad MIME, `x.php.png`, traversal filename, SVG/HTML render, EXIF (`upload`) | A | RICH, PARTIAL |

### Access control - `ref-access-control-techniques.md`, `ref-graphql-techniques.md`

| Class | Stage | Modes |
|---|---|---|
| IDOR matrix, cross-account replay (`idor-x`) | A | two authenticated identities, regardless of RICH or PARTIAL label |
| UI-reachable IDOR and access control (`idor`) | A | RICH, PARTIAL |
| Unauthenticated API access (`unauth-access`) | B | all |
| Sequential / predictable ID walk (`id-walk`) | B | all |
| **Path bypass for authorization** (normalisation, traversal, encoded segments) (`path-bypass`) | B | all |
| **HTTP method override** (`X-HTTP-Method-Override`, `_method`) (`method-override`) | B | all |
| **API versioning bypass**: replay each shape on derived `/v1`..`/vN`, unversioned, header, and query variants, including write verbs, even when recon observed only one version (`api-version`) | B | all |
| Function-level access: admin and internal wordlist (`func-level`) | B | all |
| GraphQL access control and introspection (`graphql-authz`) | B | all |

`api-version` eligibility is derived, not discovered. Include every shape carrying a version marker: path `/vN/`, `version=`/`api-version=` query, `api-version`/`LD-API-Version`-style header, `vnd.*.vN` content type. When no marker was observed, still include a bounded set of representative API operations (auth, object read and write, admin, workflow transitions) and probe `/v1`, `/v2`, `/v3`, and unversioned spellings. One test event per eligible shape covers its derived variant set: replay the original request across versions and across verbs the variant accepts (GET plus POST/PUT/PATCH/DELETE). Compare status, schema, and fields between versions; older versions often keep weaker authz or extra fields after the current version adds controls. Uniform 404 across variants is `dead` evidence.

### Authentication and session - `ref-auth-techniques.md`

| Class | Stage | Modes |
|---|---|---|
| JWT analysis: alg confusion, weak secret, claim tampering, audience confusion (`jwt`) | B | RICH, PARTIAL |
| Session management: fixation, revocation, logout, concurrent sessions (`session`) | A | RICH, PARTIAL |
| OAuth / OIDC / SAML: redirect_uri, state, code reuse, binding, postMessage leakage (`oauth`) | A | RICH, PARTIAL |
| **Password reset / email change / magic link** (`pwreset`) | A | RICH, PARTIAL |
| **Token in URL / referer leakage** (`token-leak`) | B | RICH, PARTIAL |
| **SIWE / wallet authentication** (nonce reuse, signature replay, chain-id) (`siwe`) | A | RICH, PARTIAL |
| Auth bypass on pre-auth endpoints (login, activate, reset, verify) (`preauth-bypass`) | B | all |

### Business logic - `ref-business-logic-techniques.md`

| Class | Stage | Modes |
|---|---|---|
| **Numeric tampering**: price, quantity, discount, currency, sign, precision (`numeric`) | A | any authenticated identity |
| **Mass assignment**: `role`, `verified`, `kyc_level`, `is_admin`, tenant id (`massassign`) | A | any authenticated identity |
| Race conditions: TOCTOU, double-spend, lost update (`race`) | A tail | any authenticated identity |
| Idempotency and replay (`idempotency`) | A tail | any authenticated identity |
| **Workflow step-skip / state-machine bypass** (`workflow`) | A | RICH, PARTIAL |
| HTTP method swap (logic side) (`method-swap`) | A | any authenticated identity |
| Entitlement and plan bypass (`entitlement`) | A | any authenticated identity |

### Client-side and protocol - `ref-injection-techniques.md`

| Class | Stage | Modes |
|---|---|---|
| DOM XSS, postMessage (`domxss`) | A | all (CSRF needs a session: RICH, PARTIAL) |
| **CSWSH** (WebSocket origin check) (`cswsh`) | B | RICH, PARTIAL |
| **Cache deception and cache poisoning** (`cache`) | B | all |
| **Header injection** (Host, X-Forwarded-*, CRLF) (`header-inj`) | B | all |
| **Second-order**: plant in one surface, fire in another (`second-order`) | A | RICH, PARTIAL |
| API abuse patterns, subdomain-specific behaviour (`api-abuse`) | B | all |

### Injection, demoted subset - `ref-injection-techniques.md`

| Class | Stage | Modes |
|---|---|---|
| SQLi, NoSQLi, OS command, CSV/formula, GraphQL injection (`injection`) | B | all, **lowest priority** |

**Read the demotion precisely.** What produced zero confirmed origins across the last 100 reports on ~107 hours was *systematic parameter fuzzing for SQLi, NoSQLi, command and CSV injection*. That specific activity is demoted: opportunistic only, on params already surfaced, after everything above it.

It is **not** a demotion of SSTI, path traversal / LFI, or header injection. Those sit at normal Stage B priority in their own groups above. SSTI is server-side RCE, and an ImageMagick-driven arbitrary file read was a confirmed critical.

**Skip outright:** CORS as a standalone (note only a credentialed read of real secrets found incidentally). Registration / login enumeration, account-existence and timing oracles. Rate-limit absence as a standalone. Note one of these only when it directly enables an ATO or mass-data chain, in which case it belongs to Stage C.

## Stage A: controller lanes, serial across browser state

You are the only actor allowed to navigate Phase 1 profiles. In a dual lane, use both `$CDP_A` and `$CDP_B`, but operate them deliberately and serially. Two profiles enable attacker and victim correlation; they do not authorize two agents to mutate the accounts concurrently.

**Dual lane:** A is clean control and attacker. B is victim, verification, and later disruptive-test identity. Finish every cross-account test before any action that could revoke B.

**Single lane:** use whichever auth file is valid. Run authenticated-versus-unauthenticated checks and reversible own-account tests. Cross-account classes are inapplicable, not silently degraded.

**Unauth lane:** acquire one clean profile and keep it leased for Phase 3 Closeout. Run DOM XSS, postMessage, open redirect confirmation, cache deception, and all applicable stateless work. Do not attempt account creation again and do not wait for access.

```bash
if [ "$IDENTITY_LANE" = "unauth" ]; then
  OWNER="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_LEASE_CYCLE}:unauth"
  ACQ=$(python3 "$YOUR_HELPERS_ROOT/browser-lease.py" acquire \
    --owner "$OWNER" --role sweep-unauth --geo "${GEO:-}" --origins-file "$SCOPE_HOSTS_FILE")
  PROFILE_A=$(echo "$ACQ" | grep -oE '^[0-9]+' | head -1)
  [ -n "$PROFILE_A" ] || { echo "ERROR: no browser profile available for unauth Stage A"; exit 1; }
  CDP_A=$((YOUR_CDP_BASE_PORT + PROFILE_A))
  LIVE_A=true
  echo "unauth browser: p$PROFILE_A CDP=$CDP_A  (Closeout releases this lease)"
fi
```

### Bounded browser discovery pre-pass

Spend no more than 15 minutes and visit no more
than 10 routes. Use A only for discovery; leave B untouched for the
bidirectional and disruptive checks below.

Complete the controller-order baseline-health step below before starting this
pass. If A is not healthy, record the browser pass as partial and continue with
saved request evidence rather than trying to repair the account here.

The purpose is to find browser-reachable operations that the endpoint catalog
still lacks. This is not the deep browser hunt owned by P05.

Prioritize:

1. Main application navigation and dashboard.
2. Account, tenant, team, member, invitation, and role management.
3. Settings, security, MFA, SSO, OAuth, and session controls.
4. Billing, plan, entitlement, order, booking, and workflow transitions.
5. Upload, import, attachment, image, document, integration, webhook, export,
   and public-sharing entry points.

For each visited route, correlate the visible action with its underlying
request. Look specifically for client-only paywalls, role gates, MFA gates,
hidden routes, disabled controls, and parameters or object identifiers added by
JavaScript. Append every new in-scope request to the endpoint catalog and add
high-value operations to `raw/operation-candidates.jsonl`. Discovery is not a
verdict. Existing Stage A class checks own the controlled replay.

Before browsing, snapshot the current profile flow. Run the bounded route
driver against the already-authenticated A profile, or the clean unauthenticated
profile acquired above. Then diff the captured request set so P01 traffic is not
mistaken for new P02 discovery.

```bash
BROWSER_COVERAGE="$R/phase02-browser-coverage.jsonl"
BROWSER_ROUTES="$R/phase02-browser-routes.txt"
BROWSER_BEFORE="$R/phase02-browser-before.txt"
BROWSER_AFTER="$R/phase02-browser-after.txt"
BROWSER_NEW="$R/phase02-browser-new.txt"
: > "$BROWSER_COVERAGE"
: > "$BROWSER_ROUTES"
: > "$BROWSER_BEFORE"
: > "$BROWSER_AFTER"
: > "$BROWSER_NEW"

if [ "$LIVE_A" = true ] && [ "${PROFILE_A:-0}" -gt 0 ] 2>/dev/null \
   && curl -fsS "http://$YOUR_CDP_HOST:$CDP_A/json/version" >/dev/null 2>&1; then
  MD=$YOUR_MITMDUMP
  sudo -u "$YOUR_BROWSER_USER" "$MD" -nr "$YOUR_BROWSER_FLOW_DIR/p${PROFILE_A}/current.flow" \
    -s /dev/stdin <<'PY' | sort -u > "$BROWSER_BEFORE"
def response(flow):
    print(f'{flow.response.status_code} {flow.request.method} {flow.request.pretty_url}')
PY

  case "$TARGET" in wild.*) BROWSER_SURFACE=wildcard ;; *) BROWSER_SURFACE=specific ;; esac
  timeout 900 python3 "$YOUR_HELPERS_ROOT/browser-recon.py" drive \
    --cdp-port "$CDP_A" \
    --scope-hosts "$SCOPE_HOSTS_FILE" \
    --live-hosts "$R/live-hosts.txt" \
    --surface "$BROWSER_SURFACE" --apex "$APEX" \
    --routes-output "$BROWSER_ROUTES" \
    --min-routes 6 --max-routes 10 \
    --settle-seconds 3 --post-seconds 2 --captcha-timeout 15 || true

  sudo -u "$YOUR_BROWSER_USER" "$MD" -nr "$YOUR_BROWSER_FLOW_DIR/p${PROFILE_A}/current.flow" \
    -s /dev/stdin <<'PY' | sort -u > "$BROWSER_AFTER"
def response(flow):
    print(f'{flow.response.status_code} {flow.request.method} {flow.request.pretty_url}')
PY
  comm -13 "$BROWSER_BEFORE" "$BROWSER_AFTER" > "$BROWSER_NEW"

  while read -r code method url; do
    [ -z "$url" ] && continue
    "$EP" append --json "$(jq -nc --arg m "$method" --arg u "$url" \
      '{kind:"discover",method:$m,url:$u,source:"phase02-browser"}')"
    if [ "$code" -eq "$code" ] 2>/dev/null; then
      "$EP" append --json "$(jq -nc --arg m "$method" --arg u "$url" --argjson c "$code" \
        '{kind:"probe",method:$m,url:$u,identity:"browser",status:$c}')"
    fi
  done < "$BROWSER_NEW"

  python3 - "$BROWSER_ROUTES" "$BROWSER_COVERAGE" "$IDENTITY_LANE" <<'PY'
import json, sys
from datetime import datetime, timezone
from pathlib import Path
routes, output, lane = Path(sys.argv[1]), Path(sys.argv[2]), sys.argv[3]
now = datetime.now(timezone.utc).isoformat()
rows = []
for url in dict.fromkeys(line.strip() for line in routes.read_text(errors="replace").splitlines() if line.strip()):
    rows.append({"url": url, "surface": "bounded-route-walk", "auth_state": lane, "actions": ["navigate", "capture-requests"], "status": "mapped", "recorded_at": now})
if not rows:
    rows.append({"url": None, "surface": "bounded-route-walk", "auth_state": lane, "actions": [], "status": "partial", "reason": "no route completed or browser unavailable", "recorded_at": now})
output.write_text("".join(json.dumps(row, separators=(",", ":"), sort_keys=True) + "\n" for row in rows))
PY
else
  jq -nc --arg lane "$IDENTITY_LANE" \
    '{url:null,surface:"bounded-route-walk",auth_state:$lane,actions:[],status:"partial",reason:"live browser unavailable; continued with saved request evidence"}' \
    > "$BROWSER_COVERAGE"
fi

"$EP" rebuild
"$EP" stats
```

After the driver, inspect the mapped routes in Chrome. For each relevant menu,
form, or multi-step workflow, append one terminal ledger row with
`status: tested|mapped|partial|blocked`, the action taken, and the request shape
captured. Exercise at most one safe happy-path action per workflow during this
discovery pass. Do not submit irreversible actions. Upload surfaces come first.

Any newly exposed operation must enter the existing P02 eligibility and class
workflow before the phase can finish. The pre-pass does not replace IDOR,
workflow, entitlement, OAuth, DOM XSS, postMessage, CSRF, upload, or other
verdicts below.

Authenticated controller classes need live state, identity correlation, or ordered mutations. A subagent cannot own them. Work the catalog, not memory. Use `$EP pending --test idor`, `--test idor-x`, `--test workflow`, `--test session`, and `--test oauth`, intersected with eligibility. Prefer shapes whose `auth_states` show a successful identity and the top auth-only endpoints from the access contract.

### Controller order

1. **Baseline health.** Confirm each available identity still reaches one known authenticated page or API. Save request, status, identity, profile, and UTC time in `raw/identity-health-before.json`. A live CDP is useful but not required when copied auth still works.
2. **Nondestructive browser coverage.** Run DOM XSS, postMessage, OAuth observation, token leakage, cache behavior, UI-reachable access control, and auth-aware infrastructure without changing credentials or revoking sessions.
3. **Dual-identity access control.** Run the bidirectional matrix below when `IDENTITY_LANE=dual`.
4. **Reversible state changes.** First prove normal happy path for exact operation. Then run workflow, numeric, mass-assignment, method-swap, entitlement, upload UI, and second-order checks against test-owned data. Restore changed values where practical.
5. **Wait on disruptive work.** Logout, revocation, password or email change, reset completion, OTP or MFA attempt counters, resend limits, races, idempotency bursts, and scale checks belong to the tail after stateless agents return.

### Bidirectional cross-account matrix

Run this only when both `auth-a.json` and `auth-b.json` are authenticated. `PARTIAL` with two identities still runs it. `RICH` missing a second working identity does not pretend it ran.

Create `raw/identity-object-map.json` before replay by reconciling `raw/resource-graph.json` with live browser and API evidence. For every relevant object type, record at least one A-owned and one B-owned object ID, exact operation, ownership proof, parent or tenant relationship, current workflow state, and whether operation is read or write. Include tenant/account, primary business object, workflow instance, upload/attachment, invitation/member, subscription/order/booking/entitlement, and status transition whenever target exposes them. Do not stop after first convenient object type.

If required A or B object is missing, attempt smallest reversible happy-path creation first. Save happy-path request and response before any mutation. If prerequisite cannot be created, exact affected operations remain PARTIAL with gap. Missing child state is not DEAD and does not make workflow or business-logic class N/A when operation candidate exists.

For each eligible object operation:

1. Prove A can access A and B can access B as controls.
2. Replay B's object through A's identity.
3. Replay A's object through B's identity. This catches asymmetric tenant and role behavior.
4. Try unauthenticated replay when safe and relevant.
5. For writes, use a reversible marker, verify result from the owner's profile, then restore it. Never use deletion or an irreversible action as the first proof.
6. Record a separate `idor-x` test event per concrete operation and retain both attacker response and owner-side verification.

If one identity has saved auth but its browser is no longer live, use saved auth for request replay and the other available evidence for verification. Do not re-establish, restart, or relabel the held profile in this phase. Do not ask the user or wait. Record `partial` when owner-side verification cannot be completed.

When `IDENTITY_LANE` is `single` or `unauth`, keep object-reference operations in `idor-x-eligible.txt` and record one PARTIAL event per exact shape with `requires two authenticated test identities; available: <count>` plus existing discovery evidence. Only use empty eligibility and N/A when target has no cross-account object operation at all. Do not create cross-account events for unrelated shapes.

Upload surfaces run in reversible step 4 when present: profile picture, avatar, document, attachment, image, or import. Test oversize, MIME mismatch, `x.php.png`, traversal filename, SVG or HTML rendering, and EXIF. Server-side processing may go to Stage B only when the endpoint is isolated from shared account state.

## Stage B: fan-out to stateless subagents

Runs in every mode. Start it as soon as eligibility is built, then keep the controller lane moving. Parallelize by independence, not by matrix row count. Never spawn one subagent per row.

Cap concurrent background subagents at **3-4** (`$YOUR_REFERENCE_ROOT/ref-parallel-subagents.md`). Use four only when all four bundles have real independent work.

Use three or four work bundles, filtered to actual eligible shapes:

| Bundle | Good contents | Exclude |
|---|---|---|
| Exposure | public admin/debug, exposed config, information disclosure, bundle secrets | authenticated navigation or mutable account state |
| Infrastructure and protocol | DNS takeover, cloud storage, open backends, WebSockets, unauthenticated access | shared browser or cross-account work |
| Server-input A | SSRF, LFI, redirect, isolated upload processing | workflow, business logic, race, reset, logout, IDOR-X |
| Server-input B | SSTI, XXE, deserialization, JNDI, later opportunistic injection on distinct endpoints | workflow, business logic, race, reset, logout, IDOR-X |

One bundle may contain several independent leads. Queue every lead separately in `raw/leads.jsonl` before dispatch. A subagent returns one JSONL verdict per lead. Fewer than three bundles is correct when surface is small. Use four only when both server-input bundles contain independent eligible work; otherwise merge them.

### Subagent contract

Hand each one a fully specified bundle of independent leads. A vague brief comes back vague.

- exact URL, field, parameter
- the class to test and the `ref-<class>-techniques.md` to follow
- whether request is unauthenticated or uses an exact copied credential from `raw/auth-a.json` or `auth-b.json`
- the OAST URL if the class needs one

Rules for every subagent:

- **Never touch `$CDP_A`, `$CDP_B`, or any Phase 1 Chrome.** Use curl or scripts. Concurrent navigation corrupts controller state.
- Default to unauthenticated work. Pass copied auth only for an exact read-only request that cannot rotate tokens, revoke sessions, consume a one-time value, change account state, or trigger an availability counter.
- Never give the same authenticated identity to two active subagents. If authenticated work is not independent of controller state, keep it in Stage A.
- Never run login, OTP, MFA, reset, resend, race, idempotency, rate, or lockout checks. Controller tail owns them.
- **Never talk to the user.** No notify, no waiting for a human. Return a verdict and stop.
- **Never write `reports/`.** You assign report numbers and write the files; that keeps numbering sequential and dedup centralised.
- Open bash with `set +u`.
- Append your own `test` events to the catalog as you go. Concurrent appends are safe.
- Before returning verdict, run `export PHASE=phase-02-sweep; . $YOUR_HELPERS_ROOT/bin/provenance.sh; hunt_provenance >/dev/null` and return detected provider, model, effort, and phase. Do not infer them from briefing.

Return one object per lead as JSONL, and nothing else. A one-lead bundle returns one line:

```json
{"lead_id":"...","class":"ssrf","result":"confirmed|dead|partial|skipped","severity":"critical|high|medium|low|info|none",
 "endpoint":"METHOD URL","evidence":"raw/<path>","impact":"one line","note":"...",
 "determined_by":{"phase":"phase-02-sweep","provider":"...","model":"...","effort":"..."}}
```

Keep the subagent's transcript out of your context. Keep the verdict line.

### Lead ledger

Maintain `raw/leads.jsonl`, one line per lead, appended **before** spawning:

```json
{"id":"L07","surface":"webhook url field","url":"POST https://x/api/webhooks","class":"ssrf","status":"queued|dispatched|confirmed|partial|dead|skipped","agent":"","report":""}
```

You are the only one who knows every in-flight lead. Dedupe against the ledger before spawning a duplicate. On compaction, re-read it to recover state.

When a verdict returns, update that lead's existing ledger entry to its terminal status and add evidence/report fields before collecting another completed lead. Do not leave a completed subagent recorded as `dispatched`.

## Stage A tail: disruptive checks after normal coverage

Do not enter this section until all cross-account work is terminal and every stateless subagent has returned. This ordering preserves both identities for higher-value coverage. Closeout does not need a live B session. In dual lane, B becomes sacrificial after normal coverage and must be used for applicable reversible tail checks.

Route by identity lane:

- **Dual:** use B for logout, revocation, reset completion, email or password change, OTP or MFA counter checks, races, and idempotency. Keep A authenticated as clean control. Recheck A after each B-side test. Never skip an applicable B-side check to preserve B for closeout.
- **Single:** preserve the only identity. Run reversible logout and old-token replay last. Do not run generic failed-login, OTP, MFA, resend, reset, or lockout counting. A concrete bypass can be tested only with the bounded proof below.
- **Unauth:** authenticated disruptive work is not applicable. Pre-auth bypass checks remain low-volume and must not target anyone except the owned test identity named by Phase 1, if one exists.

Build applicable checklist before running anything: logout, access-token replay, refresh-token replay, protected-resource verification, revocation, `password-email-change`, `reset-completion`, OTP/MFA counters, resend, race, and idempotency. Each item must end as `completed`, `partial`, `not-applicable`, or `policy-excluded`, with reason and evidence. Dual lane always includes named `password-email-change` and `reset-completion` items. They may be not applicable only when surface is absent, policy excludes it, or concrete safety or prerequisite evidence blocks it. "Preserve account for closeout" is never a valid reason. A class being low priority does not make it not applicable.

For reportable session behavior, minimum sequence is: capture pre-logout protected-resource baseline and access/refresh token snapshot, perform normal logout, replay access token, replay refresh token, then request protected resource with any newly issued or replayed token. Missing refresh token is `not-applicable`; failure to capture or exercise one that exists is `partial`.

Use minimum proof:

- Race or duplicate processing starts with two concurrent requests against a reversible test-owned action. Increase only when first pair is ambiguous, with a hard ceiling of five requests.
- Identity-sensitive failure checks require a concrete bypass or ATO hypothesis. Use B only and stop after three invalid attempts, or one attempt before an advertised lockout threshold, whichever is lower.
- Do not rotate IPs, sessions, endpoint spelling, accounts, or custom headers to evade a counter. Do not use A after B reaches any stop signal.
- Do not repeatedly send email or SMS. One delivery proves trigger behavior. A second is allowed only when replay or counter reset is the exact hypothesis.

At first stop signal, preserve response and headers, mark affected matrix rows `partial` or `skipped`, and continue without that identity. Never wait for a cooldown. If B is lost, A remains available for remaining read-only work. If the single identity is lost, finish unauthenticated and stateless recording rather than blocking the phase.

Write tail record after section. Overall `completed` means every applicable required check has item status `completed`. Overall `partial` means any applicable item was skipped, blocked, unavailable, or ended at safety stop. Overall `not-applicable` means no account or no concrete disruptive hypothesis exists for every item. Overall `policy-excluded` means every otherwise-applicable item is explicitly excluded by program policy. Never write `completed` while reason says an applicable check did not run.

```bash
TAIL_STATUS="<completed|partial|not-applicable|policy-excluded>"
TAIL_REASON="<what ran, or exact reason none was applicable; include any stop signal>"
# Write `TAIL_CHECKS_JSON` as an array. Each item has name, applicable, status,
# reason, and evidence. Evidence is target-relative when traffic was sent.
TAIL_CHECKS_JSON='[{"name":"logout-token-replay","applicable":true,"status":"completed","reason":"access and refresh replay both tested","evidence":"raw/responses/session-logout-sequence.json"}]'
jq -n --arg lane "$IDENTITY_LANE" --arg status "$TAIL_STATUS" \
  --arg reason "$TAIL_REASON" --argjson checks "$TAIL_CHECKS_JSON" \
  --arg completed_at "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  '{identity_lane:$lane,status:$status,reason:$reason,checks:$checks,completed_at:$completed_at}' \
  > "$R/phase02-disruptive-tail.json"
```

## Stage C: chain, after coverage

Run this once Stage A and Stage B have verdicts recorded. It is the last thing you do before close-out, and it is not optional.

Setup already removed any stale completion record. Do not recreate it until every Stage C check below is finished.

Individual verdicts are the deliverable, but the highest-severity findings in this corpus were combinations, not single classes. A stored XSS plus a wildcard CORS read is an account takeover. A price-manipulation medium became a high once it was shown to scale. Nothing downstream does this for you: each goal cycle starts cold and sees one angle at a time, so a chain that spans two of your findings is invisible to all of them.

**Input:** everything you produced. `reports/`, every `hunt/` angle file including the `PARTIAL` ones, `raw/leads.jsonl`, and every `test` event with `result: confirmed` or `partial`.

```bash
grep -l 'CONFIRMED\|PARTIAL' hunt/phase-*.md
jq -r 'select(.status=="confirmed" or .status=="partial") | "\(.class) \(.url)"' raw/leads.jsonl 2>/dev/null
ls reports/
```

**Ask of every pair:** does A give the precondition B needs?

Shapes that have paid here:

| Chain | Becomes |
|---|---|
| Upload / stored content + permissive origin or CORS read | account takeover |
| SSRF + open redirect or DNS rebind | cloud metadata, internal service reach |
| Info disclosure (IDs, emails, tenant list) + IDOR | mass extraction rather than one record |
| Auth bypass + IDOR | unauthenticated mass PII |
| Secret in a bundle + an API that accepts it | privileged access with no account |
| Several individually-minor OAuth defects: wildcard `redirect_uri`, no `state`, code reuse, unbound `redirect_uri` | cross-subdomain code theft, ATO |
| A logic flaw + absence of a rate limit or cap | the same flaw at scale, which is usually the severity jump |
| Second-order: a value planted in one surface that executes in another | stored impact from a reflected-looking bug |

**Also do the scale pass, last.** For each confirmed medium, ask what makes it a high: can it be repeated, does it reach other tenants, does it touch money or PII in bulk, or can victim interaction be removed. This is hypothesis-driven escalation, not a generic rate-limit sweep.

Apply these limits:

- If no confirmed flaw depends on scale, record `not applicable` and send no rate-test traffic.
- Use test-owned objects and minimum proof. Never enumerate real IDs or collect third-party data.
- For identity-sensitive endpoints, use B only in a dual lane and follow the three-attempt ceiling from the disruptive tail. In a single or unauth lane, do not run credential, OTP, MFA, reset, or lockout throughput checks.
- For a non-authenticated or reversible own-resource flaw, use no more than five repeat requests. Stop sooner on any availability signal.
- Do not rotate IPs, accounts, sessions, paths, GraphQL aliases, batches, or custom headers to bypass a counter. A bypass technique is tested only when it is itself the confirmed root cause and the bounded proof stays non-destructive.
- Losing B or seeing a throttle ends the scale pass. Record the limit or partial result and continue to close-out without waiting.

**Recording:**
- Same root cause pushed further → edit the existing report. Bump severity, add an `## Escalation` section, and add `Escalation of: reports/<file>.md` to the angle file.
- Genuinely new root cause created by the combination → new report, and reference both parents.
- Chain tried and it does not hold → write it in the angle file as `DEAD`. A disproved chain is as useful as a disproved class.
- Record the chain in the catalog so it is queryable:

```bash
$EP append --json '{"kind":"chain","method":"POST","url":"<entry url>","chain_id":"C1","note":"upload XSS + wildcard CORS -> ATO, reports/REPORT-04.md"}'
```

After every confirmed report has received a concrete `## Chain value` result, including `standalone` when no useful combination holds, write the completion record. Do not write it if any report is still unchecked.

```bash
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
  STAGE_C_REPORTS=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' | wc -l)
  STAGE_C_REPORT_SHA256=$(find reports -maxdepth 1 -type f -name 'REPORT-*.md' -print0 \
    | sort -z | xargs -0 -r sha256sum | sha256sum | awk '{print $1}')
  STAGE_C_CHAINS=$(jq -r 'select(.kind=="chain") | .chain_id' "$R/endpoints-events.jsonl" 2>/dev/null | sort -u | wc -l)
  STAGE_C_ESCALATIONS=$(grep -rl 'Escalation of:' hunt/*.md 2>/dev/null | wc -l)
  jq -n \
    --arg phase "$PHASE" \
    --arg completed_at "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
    --arg report_sha256 "$STAGE_C_REPORT_SHA256" \
    --argjson report_count "$STAGE_C_REPORTS" \
    --argjson chain_count "$STAGE_C_CHAINS" \
    --argjson escalation_count "$STAGE_C_ESCALATIONS" \
    '{phase:$phase,completed_at:$completed_at,all_confirmed_reviewed:true,report_count:$report_count,report_sha256:$report_sha256,chain_count:$chain_count,escalation_count:$escalation_count}' \
    > "$R/stage-c-complete.json"
  cat "$R/stage-c-complete.json"
else
  echo "Stage C incomplete; completion record not written"
fi
```

## UNAUTH lane

This applies when Phase 1 says UNAUTH, or when a non-UNAUTH contract has zero usable auth identities. Stage A runs only its unauthenticated browser subset: DOM XSS, postMessage, open redirect confirmation, and cache deception. For authenticated session, workflow, business-logic, and cross-account classes, retain static, documented, and observed operation candidates as eligible and mark each PARTIAL with missing-auth prerequisite. Use empty eligibility and N/A only when no relevant operation, sink, or state exists. Stage B still runs, with priorities that do not depend on sessions. Do not retry registration, ask for access, or wait.

**Read `## Surface shape` from `hunt/phase-00-recon.md` first.** It changes how hard this mode is worth pushing:

- `crypto-web3` or `mobile-first`: lean in. Every no-account critical in the corpus came from targets of these shapes, and items 1-3 below are where they came from.
- `saas-b2b` or `consumer-b2c`: work the list, but expect thin. Value sits behind the login on these; record what you covered and say plainly in the contract that the unauthenticated surface was limited rather than padding it.
- `infrastructure` or `mixed`: judge from the endpoint catalog, not the label.

Priority order, taken from what the 31 no-account findings actually were:

1. **Secrets in publicly downloadable artifacts.** JS bundles and source maps in `raw/js/`, pulled by Phase 0 and augmented by Phase 1's authenticated walk. Production API keys, signing secrets, SDK keys, cloud credentials. This produced a hardcoded Zendesk JWT signing secret, a Helius key in a Next.js bundle, an RSA key in a native lib, AES keys in an APK and an iOS app, a DreamFactory key.
2. **Exposed unauthenticated services.** GraphQL introspection, Prometheus `/metrics`, Spring actuator, Hasura, Supabase with weak RLS, admin panels, Swagger. Produced unauth Hasura user data, Prometheus leaking Bitcoin data, misconfigured Supabase RLS.
3. **Unauth API discovery and BOLA** on public endpoints. Every shape in the catalog with no auth state, probed for object access by ID.
4. **Auth bypass on pre-auth endpoints.** Login, activate, reset, verify. Endpoints that return real data before a session exists.
5. **CI and supply chain excluding package-namespace takeover.** Unpinned installers, mutable tags, workflow injection.

One exception to the web-only rule: if the program's scope includes a mobile app and the target is crypto/web3 or mobile-first, an APK or IPA is a publicly downloadable artifact and item 1 applies to it. Extract and scan for hardcoded secrets only. Anything beyond static secret extraction belongs to the mobile pipeline.

## Recording findings

### hunt/ angle files

`hunt/` is your only memory across compaction. An angle not in a file gets retested.

Write the file the moment you pick an angle, **before** testing. Never start a second angle while the first has no file. `ls hunt/` first; `phase-00` and `phase-01` exist, continue at the next free NN as `phase-NN-<slug>.md` where the slug names the angle YOU tested. `CLOSEOUT.md` and `RECON-NOTES.md` are reserved for the close-out phase; never use those names and never write into them.

One angle per file: Angle / Hypothesis / Tried (referencing `raw/`) / Result / Status. Status is the last line and is one of `DEAD`, `PARTIAL`, `SKIPPED`, `CONFIRMED-<SEVERITY>`. `SKIPPED` requires exact mode or prerequisite reason. Extend a `PARTIAL` or `CONFIRMED` in place rather than opening a new file for the same root cause.

A subagent's confirmed lead is your angle to file. Record it like any other.

**Recording DEAD ends matters more than recording wins.** A dead angle you did not write down costs a goal cycle the same hours you just spent.

### reports/

You are the only writer.

Write a report for **every confirmed MEDIUM-or-above**, not just criticals. Highs and mediums are real money. Drop low and info: note in `hunt/` only if it chains.

On confirm, write `reports/REPORT-NN.md` (next free NN) NOW, before the next angle. A bug left only in `hunt/` or a subagent verdict dies on compaction.

`ls reports/` first. New class, root cause, or endpoint means a new file. Same root cause pushed further means edit the existing one: bump severity, add `## Escalation` and `Escalation of: reports/<file>.md` to the angle file.

#### Format

**`$YOUR_HELPERS_ROOT/report-format.md` is the canonical spec.** Read it before writing your first report. Headings are verbatim and downstream triage parses them; the corpus currently carries the same section under five different names because nobody specified it.

Headings, in order: `## Severity`, `## Summary`, `## Description`, `## Impact`, `## Affected`, `## Evidence`, `## Reproduction`, `## Component findings`, `## Chain value`, `## Provenance`.

Two rules decide whether a report survives, so they are repeated here:

- **Severity is one bare uppercase word on its own line.** `CRITICAL | HIGH | MEDIUM | LOW | INFO`. Triage greps it; `Medium (possibly High)` is unparseable.
- **Provenance is mandatory, detected rather than guessed, and written by the shell.** After writing a report, run `stamp_report reports/REPORT-NN.md "<found by>"`. For orchestrator work, found-by is `orchestrator, stage A` or `stage C chain`. For delegated work, include returned identity verbatim, for example `subagent, stage B, lead L07, determined by claude:model-slug, effort high`. `Model` stamped by helper is report-writer model. Keep delegated determining identity in `Found by`; runner records structured run and cycle attribution separately. Helper appends phase, writer provider/model/effort and UTC date, and refuses double-stamp. Attribution reconstructed later is ambiguous; record it at write time.
- **Impact must answer "what does an attacker ACTUALLY do with this?"** Demonstrated outcome, not a missing control. The triage gate applies exactly that test and drops anything that fails it, however true the finding is. If the honest answer needs a precondition you did not prove, leave the angle `PARTIAL` instead of filing.

The same ref carries the `SENSITIVE-FINDINGS.md` entry format. It also shows the platform submission shape, which is different and is **not** written here: that happens after triage, in the escalate phases, built from this report.

Then close the loop in the catalog:

```bash
$EP append --json '{"kind":"test","method":"POST","url":"<url>","test":"idor","result":"confirmed","severity":"high","report":"reports/REPORT-03.md","evidence":"raw/responses/idor-summary.json","evidence_shape":"POST https://target/path","baseline_evidence":"raw/responses/idor-owner-control.txt","attack_evidence":"raw/responses/idor-cross-account.txt","verification_evidence":"raw/responses/idor-owner-verify.txt"}'
```

## Stop condition

Not a bug count. You are done when:

1. `python3 "$COVERAGE_MATRIX" . validate` exits zero and `hunt/phase-02-coverage.md` shows all 47 rows terminal.
2. Every `$R/eligibility/<class>-remaining.txt` is empty. Raw `$EP pending` output may still contain shapes excluded as inapplicable. Partial and policy-excluded shapes remain visible as PARTIAL coverage, never full completion.
3. Every confirmed medium-or-above has a `reports/` file.
4. Every angle attempted has a `hunt/` file with a terminal Status.
5. `raw/leads.jsonl` has no lead left in `queued` or `dispatched`.
6. Stage C has run: every confirmed finding has been checked for chains and for the scale question, with the result recorded either as an escalation or as a `DEAD` chain note.
7. `raw/phase02-identity-lane.json` records zero, one, or two identities, and `raw/phase02-disruptive-tail.json` records valid overall status plus itemized checks. `completed` is rejected if any applicable item did not complete.
8. `python3 "$MOBILE_HANDOFF_HELPER" validate` exits zero. No imported mobile
   candidate remains queued or dispatched, and every terminal result has evidence.

**Budget guidance.** Aim for roughly 90-120 minutes of contiguous work. On a wildcard scope with hundreds of shapes you will not clear the catalog; work in the class-matrix order, and record what you did not reach rather than leaving it silently pending. A truthful partial map beats a padded complete-looking one.

## Depth versus breadth

When something looks bigger than a single verdict mid-sweep, file the report at the severity you can prove, note the escalation path in the angle file, and keep going. Come back to it in Stage C with the rest of the map in hand, which is when you can actually see what it chains with.

Do not spend the sweep's budget on one deep chain while half the catalog is unverified. Equally, do not skip Stage C: an unchained set of mediums undersells what you already found, and no later phase will chain them for you.

---

## What NOT to do
- Do NOT chase depth BEFORE coverage is recorded. Chaining after the sweep is Stage C and is required; abandoning the catalog mid-way to follow one lead is what this ordering prevents.
- Do NOT leave a tested shape without a `test` event. An unrecorded dead end is a re-test later.
- Do NOT run a systematic injection sweep. Opportunistic only, on params already surfaced, after everything above it.
- Do NOT chase CORS or enumeration oracles as standalone findings.
- Do NOT let a subagent touch your Chrome, write `reports/`, or talk to the user.
- Do NOT spawn more than 3-4 concurrent subagents. Use fewer when the surface does not justify another bundle.
- Do NOT let two actors use the same identity, object, browser, or mutable workflow concurrently.
- Do NOT skip bidirectional A-to-B and B-to-A replay when two authenticated identities exist.
- Do NOT run disruptive auth, race, idempotency, or scale checks before cross-account and normal coverage are terminal.
- Do NOT move a lockout or rate test from B to A after a stop signal, rotate around the counter, or wait for cooldown.
- Do NOT skip Stage A wholesale in UNAUTH mode. The auth-gated classes go, the browser-driven ones stay.
- Do NOT treat UNAUTH as a thin run.
- Do NOT default ACCESS_MODE to UNAUTH because the parse failed. Read the contract.
- Do NOT release the Phase 1 browser profiles. Close-out releases them.
- Do NOT use direct `systemctl` or bare `browser-profile-command acquire` to restore an inactive Phase 1 profile.
- Do NOT test hosts outside canonical `raw/scope-hosts.txt`. Evidence-backed first-party functional siblings in that file are target surface and should be tested. Third-party providers and explicitly excluded hosts are not.
- Do NOT add custom or researcher-identifying headers.
- Do NOT connect to VNC.

---

## Phase complete: stamp finish time

```bash
python3 "$MOBILE_HANDOFF_HELPER" validate || {
  echo "ERROR: imported mobile handoff has non-terminal or unsupported results"
  exit 1
}
```

```bash
python3 - "$R/phase02-browser-coverage.jsonl" <<'PY'
import json, sys
from pathlib import Path
path = Path(sys.argv[1])
rows = []
for number, line in enumerate(path.read_text(errors="replace").splitlines(), 1):
    if not line.strip(): continue
    try: row = json.loads(line)
    except json.JSONDecodeError: raise SystemExit(f"invalid browser coverage JSON line {number}")
    if row.get("status") not in {"mapped", "tested", "partial", "blocked"}:
        raise SystemExit(f"nonterminal browser coverage line {number}")
    if not row.get("surface") or not isinstance(row.get("actions"), list):
        raise SystemExit(f"incomplete browser coverage line {number}")
    rows.append(row)
if not rows: raise SystemExit("missing bounded browser coverage")
print(f"browser coverage rows={len(rows)} terminal=yes")
PY

$EP rebuild
$EP stats

# Hard gate. Helper re-renders hunt/phase-02-coverage.md and exits nonzero for
# missing eligibility, malformed test events, remaining shapes, missing target
# records, or target angle files without terminal Status.
python3 "$COVERAGE_MATRIX" . validate || {
  echo "PHASE 02 INCOMPLETE: inspect hunt/phase-02-coverage.md"
  exit 1
}

python3 "$ORPHAN_AUDIT" . validate || {
  echo "PHASE 02 INCOMPLETE: invalid exclusions or unrouted high-value operation candidates"
  exit 1
}

[ -s "$R/stage-c-complete.json" ] || {
  echo "PHASE 02 INCOMPLETE: missing Stage C completion record"
  exit 1
}

jq -e '.identity_lane == "dual" or .identity_lane == "single" or .identity_lane == "unauth"' \
  "$R/phase02-identity-lane.json" >/dev/null 2>&1 || {
  echo "PHASE 02 INCOMPLETE: missing or invalid identity lane record"
  exit 1
}

jq -e '
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
  "$R/phase02-disruptive-tail.json" >/dev/null 2>&1 || {
  echo "PHASE 02 INCOMPLETE: missing or invalid disruptive tail record"
  exit 1
}

if jq -e 'select(.status=="queued" or .status=="dispatched")' "$R/leads.jsonl" >/dev/null 2>&1; then
  echo "PHASE 02 INCOMPLETE: queued or dispatched leads remain"
  exit 1
fi

# Final callback poll is complete. Stop only the <YOUR_OOB_TOOL> process whose command
# matches this target's output path, then preserve proof for Phase 03.
OAST_PID_FILE="$R/phase02-oast.pid"
OAST_CLEANUP="$R/phase02-oast-cleanup.txt"
: > "$OAST_CLEANUP"
if [ -s "$OAST_PID_FILE" ]; then
  OAST_PID=$(head -n 1 "$OAST_PID_FILE")
  case "$OAST_PID" in
    ''|*[!0-9]*)
      echo "FAILED: invalid <YOUR_OOB_TOOL> PID file" | tee -a "$OAST_CLEANUP"
      echo "PHASE 02 INCOMPLETE: OAST cleanup ownership cannot be verified"
      exit 1
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
        echo "PHASE 02 INCOMPLETE: refusing to stop unowned process"
        exit 1
        ;;
    esac
  fi
  if [ -n "$OAST_PID" ] && kill -0 "$OAST_PID" 2>/dev/null; then
    echo "FAILED: target <YOUR_OOB_TOOL> PID $OAST_PID remains active" | tee -a "$OAST_CLEANUP"
    echo "PHASE 02 INCOMPLETE: OAST cleanup failed"
    exit 1
  fi
  echo "VERIFIED: target <YOUR_OOB_TOOL> PID ${OAST_PID:-unknown} is stopped" | tee -a "$OAST_CLEANUP"
else
  echo "VERIFIED: no <YOUR_OOB_TOOL> PID was recorded" | tee -a "$OAST_CLEANUP"
fi

if [ -s "$R/phase02-owned-listeners.md" ] \
   && grep -qE '^ACTIVE[[:space:]]*\|' "$R/phase02-owned-listeners.md"; then
  echo "PHASE 02 INCOMPLETE: owned listener cleanup remains ACTIVE"
  exit 1
fi

ls reports/ 2>/dev/null | wc -l | xargs echo "reports written:"
find hunt -maxdepth 1 -type f -name 'phase-02-*.md' ! -name 'phase-02-coverage.md' \
  | wc -l | xargs echo "Phase 02 angle files:"

$YOUR_HUNT_BIN/hunt-phase-event finish \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-02-sweep true
```
