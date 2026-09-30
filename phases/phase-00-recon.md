> [!IMPORTANT]
> Public reference prompt. Private helpers, infrastructure, RAG, accounts, and orchestration are not included. Replace every `$YOUR_*`, `<YOUR_*>`, and project-specific command before use. See [ADAPTATION.md](../ADAPTATION.md). Never commit secrets.

# Phase 0: Lightweight Surface Map

Cross-phase coverage: read `$YOUR_HELPERS_ROOT/coverage-angles.md`. This is reconnaissance, so do not record a vulnerability test unless one actually occurs. Later phases use the shared angle ledger to distinguish prior tests from untested hypotheses. Treat the local refs and historical examples under `$YOUR_REFERENCE_ROOT` as our bounded RAG corpus: map observed technologies, protocols, and operation classes to likely `ref-<class>-techniques.md` and per-surface example files for the Phase 2 handoff, but do not turn a reference match into a finding or perform extra testing in recon.

## Purpose

Build the minimum unauthenticated map required by P01 and P02. Do not hunt vulnerabilities, validate secrets, scan infrastructure broadly, or write reports. Later phases own vulnerability testing. Prefer a small accurate map to an exhaustive wildcard inventory.

## Target and tracking

- **Domain**: {{target}}
- **Scope**: {{scope}}

Run this before any other work:

```bash
$YOUR_HUNT_BIN/hunt-phase-event start \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-00-recon true
```

Rules: stay on `$YOUR_RESEARCH_HOST`; use only supplied scope and workspace; never add custom or researcher headers; never validate a suspected vulnerability; never write `reports/` or `dig/`; release browser leases; record candidates for P02; do not connect to VNC.

## Deadline contract

Treat `$YOUR_SOFT_DEADLINE_AT` as the end of discovery. At or before that time,
stop starting new work and write the required summary from evidence already
collected. Use `UNKNOWN` for unresolved fields. Reserve the remaining time for
artifact validation, browser lease release, and the completion callback. Never
start a command whose timeout can extend beyond `$YOUR_HARD_DEADLINE_AT`.

## 1. Initialize

```bash
set +u
: "${YOUR_WORKSPACE_ROOT:?runner did not provide isolated workspace}"
: "${YOUR_TARGET_ROOT:?runner did not provide target root}"
: "${YOUR_HELPERS_ROOT:?runner did not provide frozen helpers}"
[ "$YOUR_TARGET_ROOT" = "$YOUR_WORKSPACE_ROOT" ] || exit 1
cd "$YOUR_WORKSPACE_ROOT"
TARGET="{{target_norm}}"; APEX="${TARGET#wild.}"
case "$TARGET" in wild.*) SURFACE=wildcard ;; *) SURFACE=specific ;; esac
mkdir -p "$YOUR_TARGET_ROOT"/{hunt,reports,raw/{js,responses,api-specs},sensitive}
export PHASE=phase-00-recon
EP="$YOUR_HELPERS_ROOT/bin/endpoints"
. "$YOUR_HELPERS_ROOT/bin/provenance.sh" && hunt_provenance
for file in subdomains-all.txt probe-hosts.txt live-hosts-context.txt live-hosts.txt scope-hosts.txt functional-sibling-candidates.txt functional-sibling-hosts.tsv recon-routes.txt recon-flow-summary.txt recon-endpoints-all.txt recon-endpoints-status-all.txt recon-endpoints.txt recon-endpoints-status.txt discovered-urls.txt recon-shapes.txt js-urls.txt js-endpoints.txt js-analysis.txt web3-intel.txt route-manifest-candidates.txt auth-endpoints.txt auth-endpoints-from-flow.txt oauth-providers.txt wallet-auth.txt captcha-providers.txt antibot-vendors.txt role-hints.txt operation-candidates.jsonl endpoints-events.jsonl api-contract-candidates.jsonl; do touch "$YOUR_TARGET_ROOT/raw/$file"; done
```

## 2. Preserve imported mobile handoff

If `raw/mobile-handoff/import-manifest.json` exists, read its bounded seeds and packages. Add concrete hosts, routes, methods, auth hints, object references, and sinks to `raw/operation-candidates.jsonl` with `source: "mobile-static"`. These are P02 hypotheses. Do not validate them here.

## 3. Bounded live-host map

P00 does not run Amass, Naabu, Gau, DNSx, Nuclei, GitHub scanning, default-credential checks, or broad wordlist fuzzing.

Exact targets never enumerate a wildcard. Wildcards consume the runner-prepared inventory, retain its 200-host cap, and use Subfinder only if it is empty. `subdomains-all.txt` retains context while `scope-hosts.txt` is the capped operational set. If its manifest is `partial`, preserve the full Jsmon inventory and retry only `classification_failed` entries; never replace the inventory with a narrower source.

```bash
HTTPX="$YOUR_HTTPX"; [ -x "$HTTPX" ] || HTTPX=$(command -v httpx); PREFLIGHT="$YOUR_TARGET_ROOT/raw/discovery/wildcard-preflight"
if [ "$SURFACE" = wildcard ]; then
  if [ -s "$PREFLIGHT/inventory.jsonl" ]; then
    for name in subdomains-all.txt probe-hosts.txt live-hosts-context.txt live-hosts.txt scope-hosts.txt; do cp "$PREFLIGHT/workflow-$name" "$YOUR_TARGET_ROOT/raw/$name"; done
  else
    timeout 120 subfinder -d "$APEX" -silent > "$YOUR_TARGET_ROOT/raw/subdomains-all.txt" 2>/dev/null || true
    awk 'BEGIN{IGNORECASE=1} $0 ~ /(^|\.)(app|api|auth|account|accounts|id|login|portal|dashboard|console|admin|www)\./ {print "1 " $0; next} {print "2 " $0}' "$YOUR_TARGET_ROOT/raw/subdomains-all.txt" | sort -k1,1 -k2,2 -u | head -200 | awk '{print $2}' > "$YOUR_TARGET_ROOT/raw/probe-hosts.txt"
    timeout 180 "$HTTPX" -l "$YOUR_TARGET_ROOT/raw/probe-hosts.txt" -silent -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36' -status-code -title -tech-detect -content-length -follow-redirects -o "$YOUR_TARGET_ROOT/raw/live-hosts-context.txt" || true
  fi
else
  printf '%s\n' "$TARGET" > "$YOUR_TARGET_ROOT/raw/subdomains-all.txt"; printf '%s\n' "$TARGET" > "$YOUR_TARGET_ROOT/raw/probe-hosts.txt"
  timeout 180 "$HTTPX" -l "$YOUR_TARGET_ROOT/raw/probe-hosts.txt" -silent -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36' -status-code -title -tech-detect -content-length -follow-redirects -o "$YOUR_TARGET_ROOT/raw/live-hosts-context.txt" || true
fi
if [ ! -s "$YOUR_TARGET_ROOT/raw/live-hosts.txt" ]; then
  awk '{u=$1; h=u; sub(/^https?:\/\//,"",h); sub(/\/.*/,"",h); sub(/:[0-9]+$/,"",h); p=(tolower(h) ~ /(^|\.)(app|api|auth|account|accounts|id|login|portal|dashboard|console|admin|www)(\.|$)/)?1:2; print p,u,h}' "$YOUR_TARGET_ROOT/raw/live-hosts-context.txt" | sort -k1,1 -k3,3 -u | head -10 | awk '{print $2}' > "$YOUR_TARGET_ROOT/raw/live-hosts.txt"
  awk '{h=$1; sub(/^https?:\/\//,"",h); sub(/\/.*/,"",h); sub(/:[0-9]+$/,"",h); print tolower(h)}' "$YOUR_TARGET_ROOT/raw/live-hosts.txt" | sort -u > "$YOUR_TARGET_ROOT/raw/scope-hosts.txt"
fi
[ "$SURFACE" = wildcard ] || { printf '%s\n' "$TARGET" | cat - "$YOUR_TARGET_ROOT/raw/scope-hosts.txt" | sort -u > "$YOUR_TARGET_ROOT/raw/scope-hosts.tmp"; mv "$YOUR_TARGET_ROOT/raw/scope-hosts.tmp" "$YOUR_TARGET_ROOT/raw/scope-hosts.txt"; }
```

## 4. Shallow unauthenticated browser map

Acquire one managed flow-capable profile. Visit only 3 to 6 routes, prioritizing the application root, signup, login, reset, account, and API entry points. This is route and sibling discovery only. Do not explore every menu or test any vulnerability.

```bash
if [ -s "$YOUR_TARGET_ROOT/raw/live-hosts.txt" ]; then
  RECON_OWNER="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_LEASE_CYCLE}:temp"
  ACQ=$(python3 "$YOUR_HELPERS_ROOT/browser-lease.py" acquire --owner "$RECON_OWNER" --role recon --origins-file "$YOUR_TARGET_ROOT/raw/scope-hosts.txt")
  RECON_PROFILE=$(printf '%s\n' "$ACQ" | grep -oE '^[0-9]+' | head -1)
  if [ -n "$RECON_PROFILE" ]; then
    RECON_CDP=$((YOUR_CDP_BASE_PORT + RECON_PROFILE))
    timeout 420 python3 "$YOUR_HELPERS_ROOT/browser-recon.py" drive --cdp-port "$RECON_CDP" --scope-hosts "$YOUR_TARGET_ROOT/raw/scope-hosts.txt" --live-hosts "$YOUR_TARGET_ROOT/raw/live-hosts.txt" --surface "$SURFACE" --apex "$APEX" --routes-output "$YOUR_TARGET_ROOT/raw/recon-routes.txt" --min-routes 3 --max-routes 6 --settle-seconds 2 --post-seconds 1 --captcha-timeout 10 || true
    MD=$YOUR_MITMDUMP
    sudo -u "$YOUR_BROWSER_USER" "$MD" -nr "$YOUR_BROWSER_FLOW_DIR/p${RECON_PROFILE}/current.flow" > "$YOUR_TARGET_ROOT/raw/recon-flow-summary.txt" 2>/dev/null || true
    sudo -u "$YOUR_BROWSER_USER" "$MD" -nr "$YOUR_BROWSER_FLOW_DIR/p${RECON_PROFILE}/current.flow" -s /dev/stdin <<'PY' | sort -u > "$YOUR_TARGET_ROOT/raw/recon-endpoints-all.txt"
def request(flow):
    print(f'{flow.request.method} {flow.request.pretty_url}')
PY
    sudo -u "$YOUR_BROWSER_USER" "$MD" -nr "$YOUR_BROWSER_FLOW_DIR/p${RECON_PROFILE}/current.flow" -s /dev/stdin <<'PY' | sort -u > "$YOUR_TARGET_ROOT/raw/recon-endpoints-status-all.txt"
def response(flow):
    print(f'{flow.response.status_code} {flow.request.method} {flow.request.pretty_url}')
PY
    python3 "$YOUR_HELPERS_ROOT/browser-recon.py" filter --scope-hosts "$YOUR_TARGET_ROOT/raw/scope-hosts.txt" --surface "$SURFACE" --apex "$APEX" --format request --input "$YOUR_TARGET_ROOT/raw/recon-endpoints-all.txt" --output "$YOUR_TARGET_ROOT/raw/recon-endpoints.txt"
    python3 "$YOUR_HELPERS_ROOT/browser-recon.py" filter --scope-hosts "$YOUR_TARGET_ROOT/raw/scope-hosts.txt" --surface "$SURFACE" --apex "$APEX" --format status --input "$YOUR_TARGET_ROOT/raw/recon-endpoints-status-all.txt" --output "$YOUR_TARGET_ROOT/raw/recon-endpoints-status.txt"
    python3 "$YOUR_HELPERS_ROOT/browser-lease.py" release --profile "$RECON_PROFILE" --owner "$RECON_OWNER" --role recon >/dev/null 2>&1 || true
  fi
fi
```

Build `functional-sibling-candidates.txt` from hosts in `recon-endpoints-all.txt` and `recon-routes.txt`. Review redirects, form actions, XHR, fetch, WebSocket, and visible links. Add only evidence-backed first-party siblings to `functional-sibling-hosts.tsv` as `host<TAB>role<TAB>flow evidence`. Ignore analytics, captcha, CDN, payment, support, and identity providers.

After review, merge the first column of `functional-sibling-hosts.tsv` into `scope-hosts.txt`, then rerun both `browser-recon.py filter` commands against the preserved `recon-endpoints-all.txt` and `recon-endpoints-status-all.txt`. This second filter is authoritative because it includes reviewed siblings.

## 5. Minimal JS and operation inventory

Download at most 20 in-scope JavaScript files observed by the shallow flow. Do not run analysis subagents, source-map expansion, secret validation, Web3 analysis, or vulnerability checks. Extract only auth, API, GraphQL, WebSocket, upload, billing, role, and route hints needed by P01 and P02.

```bash
awk '$2 ~ /^https?:\/\// && $2 ~ /\.m?js([?#]|$)/ {print $2}' "$YOUR_TARGET_ROOT/raw/recon-endpoints.txt" | sort -u | head -20 > "$YOUR_TARGET_ROOT/raw/js-urls.txt"
while read -r url; do
  [ -z "$url" ] && continue
  name=$(printf '%s' "$url" | sha256sum | cut -c1-16)
  curl -fsSL --max-time 20 -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36' "$url" -o "$YOUR_TARGET_ROOT/raw/js/$name.js" || true
done < "$YOUR_TARGET_ROOT/raw/js-urls.txt"
strings -n 8 "$YOUR_TARGET_ROOT"/raw/js/*.js 2>/dev/null | grep -Ei '(https?://|wss?://|/api/|/graphql|login|sign.?up|register|forgot|reset|oauth|saml|sso|upload|billing|subscription|tenant|role)' | head -1000 > "$YOUR_TARGET_ROOT/raw/js-analysis.txt" || true
grep -Ei '(login|sign.?in|sign.?up|register|forgot|reset|oauth|saml|sso)' "$YOUR_TARGET_ROOT/raw/recon-endpoints.txt" "$YOUR_TARGET_ROOT/raw/js-analysis.txt" 2>/dev/null | sort -u > "$YOUR_TARGET_ROOT/raw/auth-endpoints.txt" || true
grep -Ei '(google|apple|github|facebook|microsoft|auth0|okta|cognito)' "$YOUR_TARGET_ROOT/raw/js-analysis.txt" 2>/dev/null | sort -u | head -50 > "$YOUR_TARGET_ROOT/raw/oauth-providers.txt" || true
```

## 5a. Optional API contract import

When a supplied or observed OpenAPI, Swagger, or Postman contract is available,
save it under `raw/api-specs/` and import it before seeding the operation ledger.
Inspect only documentation URLs already seen in browser traffic, page links, or
the bounded JS pass. Fetch at most three exact, in-scope contract URLs with a
10 MiB response cap and a 20-second timeout. Do not guess or enumerate API-doc
paths in this lightweight phase.
Only hosts already in `raw/scope-hosts.txt` can produce candidates. A contract
never expands program scope. If it lacks a base URL, pass the observed in-scope
API origin with `--base-url`.

```bash
API_CONTRACT_HELPER="$YOUR_HELPERS_ROOT/api-contract.py"
for spec in "$YOUR_TARGET_ROOT"/raw/api-specs/*.json "$YOUR_TARGET_ROOT"/raw/api-specs/*.yaml "$YOUR_TARGET_ROOT"/raw/api-specs/*.yml; do
  [ -f "$spec" ] || continue
  python3 "$API_CONTRACT_HELPER" "$YOUR_TARGET_ROOT" --spec "$spec" \
    || printf '%s\n' "$spec" >> "$YOUR_TARGET_ROOT/raw/api-contract-import-errors.txt"
done
```

`raw/api-contract-candidates.jsonl` is an auditable discovery source. Imported
operations remain hypotheses for normal P02 testing. Review
`raw/api-contract-import-errors.txt`; retry a contract that lacks a usable base
with `--base-url <observed-in-scope-API-origin>`. Never infer a base from an
out-of-scope server declaration.

## 6. Endpoint and operation handoff

Seed browser-observed requests into the endpoint catalog. Preserve mobile-static records. Write `raw/operation-candidate-counts.json`, run the frozen `operation-candidates.py` validator, rebuild the catalog, and collapse observed paths into `raw/recon-shapes.txt`. Candidates are P02 inputs, not findings.

```bash
while read -r code method url; do
  [ -z "$url" ] && continue
  "$EP" append --json "$(jq -nc --arg m "$method" --arg u "$url" '{kind:"discover",method:$m,url:$u,source:"mitm-recon"}')"
  if [ "$code" -eq "$code" ] 2>/dev/null; then
    "$EP" append --json "$(jq -nc --arg m "$method" --arg u "$url" --argjson c "$code" '{kind:"probe",method:$m,url:$u,identity:"unauth",status:$c}')"
  fi
done < "$YOUR_TARGET_ROOT/raw/recon-endpoints-status.txt"

python3 - "$YOUR_TARGET_ROOT" <<'PY'
import json, sys
from pathlib import Path
root = Path(sys.argv[1]); target = root / "raw/operation-candidates.jsonl"
rows, seen = [], set()
for line in target.read_text(errors="replace").splitlines():
    try: row = json.loads(line)
    except json.JSONDecodeError: continue
    key = (str(row.get("method", "")).upper(), str(row.get("url", "")), str(row.get("operation", "")))
    if key not in seen: seen.add(key); rows.append(row)
for line in (root / "raw/recon-endpoints.txt").read_text(errors="replace").splitlines():
    parts = line.split(None, 1)
    if len(parts) != 2: continue
    method, url = parts[0].upper(), parts[1]
    key = (method, url, "")
    if key in seen: continue
    seen.add(key)
    rows.append({"method": method, "url": url, "source": "browser", "operation": "", "auth_hint": "unauth", "object_refs": [], "sinks": [], "stateful": method not in {"GET", "HEAD", "OPTIONS"}, "priority": "medium" if method not in {"GET", "HEAD", "OPTIONS"} else "low", "classes": ["workflow"] if method not in {"GET", "HEAD", "OPTIONS"} else [], "reason": "Observed during bounded unauthenticated browser mapping"})
target.write_text("".join(json.dumps(row, separators=(",", ":"), sort_keys=True) + "\\n" for row in rows))
counts = {"total": len(rows), "sources": {}}
for row in rows:
    source = str(row.get("source") or "unknown")
    counts["sources"][source] = counts["sources"].get(source, 0) + 1
(root / "raw/operation-candidate-counts.json").write_text(json.dumps(counts, indent=2) + "\\n")
PY

python3 "$YOUR_HELPERS_ROOT/threat-model.py" "$YOUR_TARGET_ROOT" derive \
  --target-name '{{target_norm}}' --scope '{{scope}}' --phase phase-00-recon
python3 "$YOUR_HELPERS_ROOT/threat-model.py" "$YOUR_TARGET_ROOT" validate

sha256sum "$YOUR_HELPERS_ROOT/operation-candidates.py" > "$YOUR_TARGET_ROOT/raw/operation-candidates-helper-sha256.txt"
python3 "$YOUR_HELPERS_ROOT/operation-candidates.py" "$YOUR_TARGET_ROOT" validate
python3 "$YOUR_HELPERS_ROOT/operation-candidates.py" "$YOUR_TARGET_ROOT" seed
"$EP" rebuild
awk '{print $NF}' "$YOUR_TARGET_ROOT/raw/recon-endpoints.txt" | sed -E \
  -e 's#/[0-9]+(/|$|\\?)#/{id}\\1#g' \
  -e 's#/[0-9a-fA-F]{8}-[0-9a-fA-F-]{27,}(/|$|\\?)#/{uuid}\\1#g' \
  -e 's#/[0-9a-fA-F]{32,}(/|$|\\?)#/{hash}\\1#g' \
  | sort -u > "$YOUR_TARGET_ROOT/raw/recon-shapes.txt"
```

## 7. P01/P02 contract

Write `hunt/phase-00-recon.md` with these exact sections and use `UNKNOWN` for unresolved fields:

```markdown
# Phase 0: Lightweight Recon Summary
## Scope shape
## Surface shape
## Operational hosts
## Host Priority Tiers
## Technology Stack
## Auth Surface
## Candidate inventory
## Imported mobile handoff
## P00 limitations
## Files generated
```

Include registration URL and method, auth sibling, login URL and method, verification, captcha, anti-bot, roles or tiers, recommended P01 account count, top operational hosts, candidate counts, and paths of all generated artifacts. A report count of zero is normal and required for P00.
Include `raw/threat-model.json` and any imported `raw/api-contract-candidates.jsonl`
in the file list. The model records inferred versus observed boundaries and does
not replace the program scope file.
Where a high-value candidate has a clear bug class, note a local RAG route such as `ref-access-control-techniques.md` plus the matching per-surface file from `hunt-examples-INDEX.md`. These are hypotheses for P02, not findings or proof.

## Phase complete

```bash
test -s "$YOUR_TARGET_ROOT/hunt/phase-00-recon.md"
test -f "$YOUR_TARGET_ROOT/raw/scope-hosts.txt"
test -f "$YOUR_TARGET_ROOT/raw/functional-sibling-hosts.tsv"
test -f "$YOUR_TARGET_ROOT/raw/recon-endpoints.txt"
test -f "$YOUR_TARGET_ROOT/raw/recon-shapes.txt"
test -f "$YOUR_TARGET_ROOT/raw/operation-candidates.jsonl"
test -s "$YOUR_TARGET_ROOT/raw/operation-candidate-counts.json"
test -s "$YOUR_TARGET_ROOT/raw/threat-model.json"
test -f "$YOUR_TARGET_ROOT/raw/endpoints.json"
$YOUR_HUNT_BIN/hunt-phase-event finish \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-00-recon true
```
