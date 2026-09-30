> [!IMPORTANT]
> Public reference prompt. Private helpers, infrastructure, RAG, accounts, and orchestration are not included. Replace every `$YOUR_*`, `<YOUR_*>`, and project-specific command before use. See [ADAPTATION.md](../ADAPTATION.md). Never commit secrets.

/goal Find 10 NEW not-yet-reported CRITICAL bugs with real impact on {{target_display}}. In-scope assets only.

Cross-phase coverage: read `$YOUR_HELPERS_ROOT/coverage-angles.md` and inspect the shared ledger before each concrete angle. Load the matching local ref and relevant status-checked historical examples when they improve the hypothesis; for browser-state leads, also load `$YOUR_REFERENCE_ROOT/ref-browser-state-techniques.md` and record the context graph, controls, and final parsed URL or origin. Cite what was actually used in the angle record. Reuse strong exact-angle evidence, but test any materially different mechanism, context, identity, route, method, payload family, or stronger independent proof. Record planned and final outcomes alongside own angle files; prior `DEAD` is not a veto on a new hypothesis.

Browser lease lifecycle, run before browsing:

```bash
BROWSER_PREFIX="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_LEASE_CYCLE}:"
BROWSER_OWNER="${BROWSER_PREFIX}browser"
# Clear only a stale lease from an interrupted prior run of this exact target/cycle.
sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND release-prefix "$BROWSER_PREFIX"
ACQ=$(python3 "$YOUR_HELPERS_ROOT/browser-lease.py" acquire \
  --owner "$BROWSER_OWNER" --role browser-walk --origins-file "$YOUR_TARGET_ROOT/raw/scope-hosts.txt")
BROWSER_PROFILE=$(echo "$ACQ" | grep -oE '^[0-9]+' | head -1)
[ -n "$BROWSER_PROFILE" ] || { echo "ERROR: no browser profile available"; exit 1; }
BROWSER_CDP=$((YOUR_CDP_BASE_PORT + BROWSER_PROFILE))
BROWSER_MITM=$((YOUR_MITM_BASE_PORT + BROWSER_PROFILE))
echo "browser-walk browser: p$BROWSER_PROFILE CDP=$BROWSER_CDP mitm=$BROWSER_MITM owner=$BROWSER_OWNER"
```

Use `$BROWSER_CDP` as this cycle's single Chrome. Log in with account credentials from `sensitive/`. Before stopping normally, run:

```bash
python3 "$YOUR_HELPERS_ROOT/browser-lease.py" release \
  --profile "$BROWSER_PROFILE" --owner "$BROWSER_OWNER" --role browser-walk
```


Provenance. Source once, before you write any report; every `stamp_report` call afterwards
records the phase and the model that actually produced the finding.

```bash
export PHASE=phase-04b-browser-walk
. $YOUR_HELPERS_ROOT/bin/provenance.sh && hunt_provenance
```

Runner also releases this exact target/cycle prefix when timer ends. Never acquire without cycle owner tag.

Docker resources. If this phase needs a local container or network, read and follow
`$YOUR_HELPERS_ROOT/runtime-resources.md`. Label and register every exact resource.

Scope loaded previously; stay strictly within it. `*.` = wildcard (any subdomain of that apex) minus Out of Scope.

Use `$YOUR_TARGET_ROOT/raw/scope-hosts.txt` as operational host list. Evidence-backed first-party functional siblings in it are part of target surface, including auth, account, application UI, API, and billing hosts. Do not discard them merely because display scope names `www.X`. Explicit named exclusions and third-party providers remain outside.

Web/API assets only. Android apps, even if in scope, are a separate pipeline: do NOT test here.

Drive Chrome via CDP. Log in with `sensitive/` accounts and walk every menu / form / multi-step flow end to end; log to `raw/` (mitm, HAR, screenshots). Focus: browser-driven bugs; UI-reachable IDOR / access-control, OAuth / SAML / SSO flow, DOM XSS, postMessage, CSRF. Uploads first; every upload surface (profile pic, avatar, doc, attachment, image, import): oversize, bad MIME / extension, `x.php.png` trick, path-traversal filename, SVG / HTML content render, EXIF injection. Server-side, infrastructure, injection, SSRF, and business-logic fan-out belongs to P05.

Run single-threaded: this phase is authenticated browser walking on a shared session. Do not fan out subagents; parallel investigation belongs to P05. See `$YOUR_REFERENCE_ROOT/ref-parallel-subagents.md`.
Skip CORS: low-yield, never pays; only note a credentialed CORS read of real secrets if found incidentally.
Skip low-impact oracles (username / email enumeration, account-existence, timing / response side-channels): don't chase as standalone bugs; only note one if it directly enables an ATO or mass data exposure chain.

Refs in `$YOUR_REFERENCE_ROOT/`: `ref-<class>-techniques.md` = how to test each bug class; `ref-browser-state-techniques.md` = browser context, cross-origin, worker, cache, and URL-normalization checks; `hunt-examples-INDEX.md` = historical reports by surface, including rejected claims. Check Status and triage activity before reusing a shape. Skim the relevant class refs + the INDEX when stuck.

Memory & paths
Target dir (all paths relative): $YOUR_TARGET_ROOT/ - `sensitive/` creds+tokens+sessions · `raw/` artifacts (dumps/payloads/scan output) · `raw/endpoints.json` endpoint catalog + auth state + per-class verdicts. Query/append the catalog via `$YOUR_HELPERS_ROOT/bin/endpoints` (`stats`, `list --untested`, `pending --test idor`, `get '<shape>'`, `append --json '...'`); read before picking targets. Tools/accounts/resources: $YOUR_RESOURCE_GUIDE.
`hunt/` is your only memory across compaction: an angle not in a file gets retested, so recording DEAD ends matters MORE than wins. Write the phase file the moment you pick an angle, BEFORE testing; never start a 2nd angle while the 1st has no file. `ls hunt/` first and continue at the next free NN: `phase-NN-<slug>.md`, slug = angle YOU tested. The earlier phases leave `phase-00-recon.md`, `phase-01-access.md`, `account-creation-process.md`, their own numbered angle files, plus `RECON-NOTES.md` and `CLOSEOUT.md`. Never write into those two. One angle per file: Angle / Hypothesis / Tried (ref `raw/`) / Result / Status (`DEAD`/`PARTIAL`/`CONFIRMED-CRITICAL`, last line). Extend `PARTIAL`/`CONFIRMED` in place rather than reopening.

Report rules
Write a report for EVERY confirmed MEDIUM-or-above finding, not just criticals: a high/medium still gets filed (only CRITICALs count toward the goal of 10, but mediums/highs are real money, never discard them). Drop low/info: note in `hunt/` only if it chains. On confirm, write `reports/REPORT-NN.md` (next free NN) NOW, before the next angle: a bug left only in `hunt/` dies on compaction. Format is specified in `$YOUR_HELPERS_ROOT/report-format.md`; use those headings verbatim. Severity is one bare uppercase word on its own line, and Impact must say what an attacker ACTUALLY does, not that a control is missing, or triage drops it. Stamp provenance with `stamp_report reports/REPORT-NN.md "<found by>"` (helper in the same ref) so the phase and model that produced the finding are recorded rather than reconstructed later. `ls reports/` first: new class/root cause/endpoint = new file; same root cause pushed = edit existing (bump severity, add `## Escalation` + `Escalation of: reports/<file>.md` to the phase file).

Stop condition
Ten CONFIRMED-CRITICAL distinct findings (escalations of one bug = one), each with a working PoC, clear impact (what attacker gains, who's affected, why critical not high), and dedup-checked against `reports/` and `hunt/`. Stop when all ten exist.
