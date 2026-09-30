> [!IMPORTANT]
> Public reference prompt. Private helpers, infrastructure, RAG, accounts, and orchestration are not included. Replace every `$YOUR_*`, `<YOUR_*>`, and project-specific command before use. See [ADAPTATION.md](../ADAPTATION.md). Never commit secrets.

# Phase 1: Access (account setup + authenticated walk)

Cross-phase coverage: read `$YOUR_HELPERS_ROOT/coverage-angles.md`. Record any actual vulnerability hypothesis tested during signup or the authenticated walk in the shared angle ledger, including the local ref or historical-example citation used. Account setup and ordinary navigation are not vulnerability verdicts. Prior results are advisory, never a ban on a different angle.

You register account A, record exactly how, then fork: account B's registration and account A's authenticated walk run concurrently.

## Resources
Read `$YOUR_RESOURCE_GUIDE` first. Lists OOB, VPS, email, SMS, browser profiles, mobile devices, wallets, tools, and compute. Use already provisioned paid resources only within documented controls and caps. Do not request payment approval for an unprovisioned resource. Record the blocker and continue with free or unauthenticated testing.

You drive registration through real Chrome profiles managed by your browser system via CDP: navigate, find inputs, fill them, submit, wait for the verification email, click the link, confirm authenticated state. <YOUR_CAPTCHA_SOLVER> attempts reCAPTCHA, hCaptcha, Turnstile, Cloudflare, and GeeTest in-page. Supported token captchas have one bounded API fallback after the extension pass.

## Hunt examples (past findings)
If a registration or walk anomaly worth turning into a finding shows up (registration upsert as ATO, mass-assignment of `role=admin` at signup, admin endpoint reachable from a regular-user UI), read the matching local technique ref and use `$YOUR_REFERENCE_ROOT/hunt-examples-INDEX.md` to find the relevant historical section. Check Status and triage activity, and carry the citation plus untested hypothesis into the Phase 2 handoff. This is a setup phase. Log anomalies, don't pursue. Phase 2 owns the hunting.

## Target
- **Domain**: {{target}}
- **Scope**: {{scope}}

## Your Role: orchestrator

**Establish what access we have, prove it is real, and route Phase 2.**

You register account A yourself, record exactly how you did it, then dispatch two subagents in parallel and handle whatever comes back. You are the only one who talks to the user and the only one who writes shared files.

Three outcomes, all of which continue to Phase 2. None stop the target.

| ACCESS MODE | Meaning | Phase 2 behaviour |
|---|---|---|
| **RICH** | Both accounts authenticated AND the walk found a real authenticated surface | Full sweep, all classes |
| **PARTIAL** | One account only, or the walk came back thin | Sweep runs; cross-account classes degraded or skipped |
| **UNAUTH** | No account | Sweep runs unauth-mode: bundle/APK secrets, exposed APIs, admin/metrics/GraphQL, and applicable supply-chain checks |

**UNAUTH is not failure.** Multiple submitted findings including criticals came from programs where no account was ever created. Never exit early on account failure alone.

**Hard limits:**
- **Desktop Chrome only. Never register or log in on a phone.** Phones are SMS inboxes only in this phase.
- **Five distinct flow-capable desktop profiles before identity fallback**, per required account. One controlled native-signup pass each. Do not ask the user after the first four recoverable signup failures. After those passes, exhaust applicable social identities, then allow one final matching platform-alias native retry at the most promising proven native condition. The platform-alias retry changes identity, not the five-pass egress diagnosis, and does not restart that ladder.
- **Email-domain validation is not an egress retry.** If the form deterministically rejects an address before creating an account, replace only the address in the same controlled pass and continue the documented email ladder. Do not acquire a new profile merely to change an invalid email domain.
- **The five passes must vary useful conditions.** Cover fresh static <YOUR_PROXY_PROVIDER> IPs in the expected geo, an alternate <YOUR_PROXY_PROVIDER> geo when the target is not country-locked, and a fresh same-country sticky <YOUR_PROXY_PROVIDER> provider path when available. Account B starts with Account A's proven working geo/provider, not a guessed default.
- **One pass per profile.** Never repeat the same submission on the same profile with unchanged conditions.
- **Release confirmed-failed leases promptly.** Save diagnostics, check whether signup partially succeeded, then release a profile as soon as it is confirmed unauthenticated. Never release a profile holding authenticated or uncertain state.
- **Identity fallback order is fixed.** Start with the configured catch-all domains on native signup. After native signup is exhausted, use distinct researcher-owned social identities only when the target visibly offers that sign-in method. After applicable social identities fail, try one unique matching platform alias through native signup: `<PLATFORM_ALIAS_BASE>+<unique>@<HACKERONE_ALIAS_DOMAIN>` for HackerOne or `<PLATFORM_ALIAS_BASE>+<unique>@<BUGCROWD_ALIAS_DOMAIN>` for Bugcrowd. Other platforms have no configured platform-alias fallback. Never use one platform's alias on another platform. This Phase 1 final fallback is an explicit exception to the general reservation of platform aliases for post-finding validation. Social profiles may be CDP-only and may not provide traffic-capture files.
- **Never connect to VNC. Ever.** It is a user-only escape hatch. Surface it only for an unresolved interactive CAPTCHA or anti-bot challenge that remains actionable in a held profile.
- **Automatic access verdict is the default.** Missing signup, unavailable or pending platform credentials, expired mail, deterministic identity rejection, invite-only access, KYC, payment, national ID, unavailable SMS/MFA, SSO-only access, thin coverage, and exhausted signup ladders never ask whether to continue unauthenticated. Select RICH, PARTIAL, or UNAUTH from evidence and continue.

## Phase tracking

Run this first, before anything else.

```bash
$YOUR_HUNT_BIN/hunt-phase-event start \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-01-access true
```

## User intervention notify: CAPTCHA only

Fire this only when an unresolved interactive CAPTCHA or anti-bot challenge is still visible and actionable in a held browser profile after the bounded automated solver, token fallback, profile, provider, geo, Google, and platform-alias paths that apply. A user clicking **Show** and solving that exact challenge must plausibly change the outcome. Generic 403s, blank pages, fingerprint rejection, missing fields, unavailable credentials, MFA, SMS, KYC, payment, invite, SSO, provisioning, and exhausted access are not intervention reasons. **Only the orchestrator fires it. Subagents have no user channel.** In an autonomous public workflow run this pauses only this Phase 1 cycle's timeout while preserving wall time and intervention wait time.

```bash
$YOUR_HUNT_BIN/hunt-intervention wait \
  --platform '{{platform}}' --handle '{{handle}}' \
  --target '{{target_norm}}' --phase phase-01-access \
  --title 'Access blocked | <short cause and profiles, max 120 chars>' \
  --action '<specific user action or choice, max 120 chars>'
```

Keep fields compact. `title` states outcome and evidence, e.g. `Access blocked | challenge failed on multiple fresh profiles`. `action` states one concrete next step. No greeting, no repeated target, no orchestration URL.

One user reply does not mean account issue is resolved. Discuss while wait remains paused. Only when you have an actionable next attempt, run this immediately before resuming tools:

```bash
$YOUR_HUNT_BIN/hunt-intervention continue
```

That starts a fresh Phase 1 hard-timeout window. If the CAPTCHA attempt still fails, select PARTIAL or UNAUTH from actual access and continue. Do not open another intervention round for the same challenge. Phase 1 completes only after you select RICH, PARTIAL, or UNAUTH.

## Setup

```bash
set +u
: "${ACCESS_MODE:=UNAUTH}"
: "${PROFILE_A:=}" "${PROFILE_B:=}" "${CDP_A:=}" "${CDP_B:=}"
: "${MFA_METHOD_A:=none}" "${MFA_METHOD_B:=none}"
: "${MFA_DESTINATION_A:=not-applicable}" "${MFA_DESTINATION_B:=not-applicable}"
: "${TOTP_SECRET_A:=not-applicable}" "${TOTP_SECRET_B:=not-applicable}"
: "${RECOVERY_CODES_A:=none-issued}" "${RECOVERY_CODES_B:=none-issued}"

: "${YOUR_WORKSPACE_ROOT:?runner did not provide isolated workspace}"
: "${YOUR_TARGET_ROOT:?runner did not provide target root}"
: "${YOUR_ACCOUNTS_FILE:?runner did not provide run account ledger}"
: "${YOUR_HELPERS_ROOT:?runner did not provide frozen helpers}"
[ "$YOUR_TARGET_ROOT" = "$YOUR_WORKSPACE_ROOT" ] || { echo "ERROR: isolated target root mismatch"; exit 1; }
BASE="$YOUR_WORKSPACE_ROOT"
cd "$YOUR_WORKSPACE_ROOT"
echo "Run workspace: $YOUR_WORKSPACE_ROOT"

TARGET="{{target_norm}}"
APEX="$TARGET"; APEX="${APEX#wild.}"
# $TARGET is the folder identifier; for wildcard scopes (wild.X) it is NOT DNS-resolvable.
# Use $APEX for tooling that needs the bare apex domain.

mkdir -p "$YOUR_TARGET_ROOT"/{hunt,raw/{responses,prewalk-logs},sensitive}

export PHASE=phase-01-access
EP=$YOUR_HELPERS_ROOT/bin/endpoints
cd "$YOUR_TARGET_ROOT"
SCOPE_HOSTS_FILE="$YOUR_TARGET_ROOT/raw/scope-hosts.txt"
HELP=$YOUR_HELPERS_ROOT/phase-01-access
OP_CANDIDATE_HELPER=$YOUR_HELPERS_ROOT/operation-candidates.py
THREAT_MODEL_HELPER=$YOUR_HELPERS_ROOT/threat-model.py
RESOURCE_GRAPH_VALIDATOR="$HELP/validate-resource-graph.py"
```

### Shared helpers

Phase 01 uses canonical public workflow helpers. Both subagents need readable script files because they do not inherit orchestrator shell functions. Source stays outside generated prompt and is referenced through `$HELP`.

```bash
for helper in dump-auth.py prewalk.py walk.sh validate-resource-graph.py captcha-fallback.py your-sms-helper.py; do
  [ -r "$HELP/$helper" ] || { echo "ERROR: missing public workflow Phase 01 helper: $HELP/$helper"; exit 1; }
done
[ -r "$OP_CANDIDATE_HELPER" ] || { echo "ERROR: missing public workflow operation candidate helper"; exit 1; }
[ -r "$THREAT_MODEL_HELPER" ] || { echo "ERROR: missing public workflow threat model helper"; exit 1; }

# Record exact helper revision used by this target without copying source into prompt.
sha256sum "$HELP/dump-auth.py" "$HELP/prewalk.py" "$HELP/walk.sh" \
  "$RESOURCE_GRAPH_VALIDATOR" "$HELP/captcha-fallback.py" "$HELP/your-sms-helper.py" \
  "$OP_CANDIDATE_HELPER" "$THREAT_MODEL_HELPER" \
  > "$YOUR_TARGET_ROOT/raw/phase01-helper-sha256.txt"
cat "$YOUR_TARGET_ROOT/raw/phase01-helper-sha256.txt"
```
## Testing Policy (read before any work)

**No custom headers. No researcher-identifying headers. No program-mandated rate limits.** Non-negotiable; overrides any program scope instruction telling you otherwise.

A real attacker doesn't send `X-Bug-Bounty: <handle>`, doesn't add `X-Forwarded-For: 127.0.0.1` "just in case", and doesn't throttle to 1 RPS because the program asked. Findings that only fire WITH researcher headers reflect known issues, not real attack surface.

Auth headers (`Cookie`, `Authorization: Bearer ...`) and explicit test-payload headers are the payload itself, allowed.

If an unannotated request returns 403 / WAF-block / Cloudflare challenge, document it and move on. The block is data.

## Inputs

1. `hunt/phase-00-recon.md`, especially **Auth Surface**: registration URL, login URL, captcha provider, verification method, and Phase 1 research account policy. Note any **auth-sibling host** the target redirects to during signup (`www.X.com` → `id.X.com` / `auth.X.com` / `account.X.com`). Phase 1 policy is always CATCHALL. Preserve any Final submission account policy for P09, but never apply it here. If the lightweight registration or login map is missing or vague, perform one targeted unauthenticated discovery pass against the listed host and evidence-backed first-party siblings, update the handoff files, and continue. Do not request a full Phase 0 rerun solely to fill an auth-map field.
2. `$YOUR_ACCOUNTS_FILE`, the empty run-local ledger. Never read or reuse global or prior-run accounts.
3. `raw/scope-hosts.txt` (Phase 0), for the auth-state dump across in-scope siblings.
4. `raw/recon-endpoints.txt`, `raw/operation-candidates.jsonl`, and any SPA route manifests from Phase 0. These seed authenticated route coverage instead of relying on guessed paths alone.
5. `sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND list`. If the browser manager is unreachable, surface and stop.
6. When present, `raw/mobile-handoff/import-manifest.json`, imported package
   reports, seeds, and `python3 $YOUR_HELPERS_ROOT/mobile-handoff-results.py list`.
   They describe exact P02 hypotheses whose account, subscription, object, role,
   collaboration, shared-link, or workflow prerequisites must be considered now.
7. When present, `sensitive/platform-credential-status.json` and
   `sensitive/platform-supplied-credentials.json`. The status file is
   non-secret inventory context. The supplied file is the first access source.
   Validate supplied accounts before attempting native or social signup. Never
   echo, paste into chat, or copy secret values outside the three approved
   sensitive account locations. Both files are re-read from the platform at
   every cycle spawn, so trust the on-disk copy over any earlier Phase 0 note
   about credential availability. Read the status file's `instruction` and
   per-asset `inquiry_description` in full: a program that grants access by
   researcher email domain rather than by a claimed value states it only
   there, and that text alone satisfies the "target explicitly requires the
   matching platform researcher domain" condition of the platform-alias
   fallback below.

## Goal

**2 accounts, both driven to a usable state**, then authenticated surface walked and mined. Phase 2 needs object depth as well as identities. Build a resource graph for every relevant reversible object type the product exposes, not only one convenient primary resource.

Imported mobile hypotheses do not change the safety or spending rules, but they
do change preparation. When a hypothesis names a concrete safe prerequisite,
create the smallest researcher-owned state P02 needs and record it in the account
recipe and resource graph. If payment, KYC, SMS, special enterprise entitlement,
or another unavailable prerequisite prevents that state, record the exact gap so
P02 can return `partial` or `blocked` rather than rebuilding accounts later.

### Scope nuance: first-party functional siblings are target surface

Many programs list `www.X` while signup, login, account management, or the application API lives on `app.X.example`, `id.X.com`, `auth.X.com`, `account.X.com`, `login.X.com`, or `api.X.com`. **A first-party sibling directly used by the listed application is part of the same target surface.** Following that flow is normal user behavior.

- Phase 1 may walk signup, verification, login, onboarding, account, and application routes on that sibling to establish access.
- Record each sibling in `raw/functional-sibling-hosts.tsv` with role and flow evidence, and ensure its hostname exists in `raw/scope-hosts.txt`. Phase 2 and later phases may test it as part of the target.
- Passive DNS discovery or a suggestive hostname is not enough. Third-party identity providers, captcha, analytics, CDN, support SaaS, and payment processors remain external.
- A program rule explicitly excluding the sibling wins. Generic scope wording that names `www.X` does not exclude a directly linked first-party `account.X`, `auth.X`, or `api.X`.

### Hard blockers

These stop a registration ladder early. They do **not** stop the phase; you continue with whatever access exists and finish PARTIAL or UNAUTH.

An obvious deterministic wall does not justify repeating five unchanged native submissions. Skip redundant native passes, then try Google only if changing the authentication route could plausibly bypass the wall. A target-side post-auth KYC, payment, invite, residency, or entitlement wall proven identical after social login is exhausted after that proof; do not burn the remaining Gmail identities or a platform alias against the same post-auth requirement.

**This shortcut is scoped to post-auth walls and never skips the platform-alias fallback.** A pre-auth identity rejection, meaning an email-domain constraint, tenant or org allowlist, or a signup route that rejects the identity before any account exists, is the exact condition the alias fallback exists to defeat. Advance to it rather than terminating, as Step 4's five-pass classification already requires. Never derive UNAUTH or PARTIAL from an inferred wall: an access mode is valid only when the ladder it claims to have exhausted produced at least one recorded live attempt per rung, or the rung is genuinely unavailable for a stated reason. A Phase 0 observation is a hypothesis to test in Step 4, never a substitute for an attempt. When the credential status file names a required researcher email domain, go straight to the matching platform-alias attempt without first spending the catch-all and Google rungs against a domain that is already known to fail.

- KYC document upload required (passport / ID / bank statement)
- Mandatory paid plan with no free tier
- Hard invite-only or paid waitlist with no public signup
- Federated SSO ONLY with no supported social option usable by the configured research identities
- National-ID or strict-checksum field (Finnish HETU, Swedish personnummer)
- SMS verification no available number can receive after the owned US, UK, <YOUR_SMS_PROVIDER> AU, AU device SIM, and bounded <YOUR_SMS_PROVIDER> fallback
- Program rules EXPLICITLY forbid interacting with the auth sibling. Generic "don't test out-of-scope hosts" does NOT count.

## Things to know before you start

- Cookie / consent / region banners are everywhere. Dismiss first.
- If the site offers English, switch to it.
- Match inputs generically (type, name, id, autocomplete, placeholder, label). Don't hardcode per-site selectors.
- React inputs need the prototype `value` setter + dispatched `input`/`change` events. `el.value = x` alone does not update React state.
- <YOUR_CAPTCHA_SOLVER> attempts reCAPTCHA, hCaptcha, Turnstile, Cloudflare, and GeeTest in-page. Wait up to 30s and never click the widget during the extension pass. A timeout means the extension pass failed, not that every <YOUR_CAPTCHA_SOLVER> API path is unavailable.
- Record the captcha iframe host and version. Custom hCaptcha asset hosts may not match the extension's `*.hcaptcha.com/captcha/*` content-script rule. Legacy GeeTest v3 (`.geetest_panel`) differs from current GeeTest v4 (`.geetest_captcha`). Do not group them under a generic captcha failure.
- SMS OTP: prefer email. Use provider order, current number inventory, and inbox commands in `$YOUR_RESOURCE_GUIDE` section "SMS OTP (last resort)." Poll only messages newer than send time and use physical SIMs only after virtual options fail. After all fixed US, UK, and AU numbers fail, use the bounded <YOUR_SMS_PROVIDER> ladder: cheapest available different-country `opt19` number first, then the target's exact service code. The helper enforces USD 2.00 per number and at most three <YOUR_SMS_PROVIDER> allocations are allowed per target account. Cancel a number immediately when the site rejects it. Block it after 580 seconds with no SMS.
- Multi-step signup: after submit, re-read the page. New form? Fill it. Repeat.
- Email verification for catch-all accounts: poll `https://<EMAIL_API_HOST>/emails?to=<exact-address>`. For Bugcrowd or HackerOne aliases, poll `$YOUR_FORWARDED_INBOX` and isolate mail by expected subject and request timestamp. Pull the link or OTP only from that matching message.
- "Ready, you can now log in" does NOT count as authenticated. Drive to a real authed view and confirm the session cookie.
- **Social sign-in precedes the platform-alias fallback.** Start with fresh catch-all native signup. After the native profile ladder fails, use distinct configured social identities only if the target visibly offers that sign-in method. Use a different identity per target account. Clear only target-origin state before each social attempt; preserve social-provider cookies. Do not test the social provider itself. Social profiles may provide CDP evidence but no traffic-capture file, so access can remain PARTIAL even when authentication succeeds. If social sign-in is unavailable or exhausted and the failure supports an email-domain or identity restriction, make the final matching platform-alias native retry described below.

### MFA and TOTP persistence

Do not enable optional MFA merely for setup. If signup, onboarding, login, or usable product access requires MFA, complete it with a researcher-owned factor and make the account reusable before calling it confirmed.

- Set `MFA_METHOD_A` and `MFA_METHOD_B` to one of `none`, `email-otp`, `sms-otp`, `totp`, `push`, `passkey`, `webauthn`, or a precise vendor method.
- For TOTP, capture the full `otpauth://` URI or visible manual Base32 secret before confirming enrollment. If only a QR is shown, decode it locally with `zbarimg`; never use an online QR or TOTP service. Do not rely on a retained QR screenshot as the only copy.
- Store a distinct `TOTP_SECRET_A` or `TOTP_SECRET_B`. Generate the confirmation code locally with `oathtool --totp -b "$TOTP_SECRET_A"` or the B equivalent. Verify one generated code succeeds. Never save an expired six-digit TOTP code as a credential.
- Capture every recovery or backup code when issued. Store each account's codes separately in `RECOVERY_CODES_A` or `RECOVERY_CODES_B`. If a code is later used, mark that exact code consumed in all saved credential records.
- For email or SMS MFA, save the method, full researcher-owned destination, and inbox retrieval route. Do not save transient email or SMS OTP values.
- For push, passkey, WebAuthn, or another device-bound factor, record the enrolled device/profile and recovery path. If no reusable factor or recovery path can be preserved, label the account `MFA session-only`; keep the authenticated profile, but do not count it as durable reusable access or `usable=true` critical capacity.
- Save reusable MFA material only in the three existing credential records: `$YOUR_ACCOUNTS_FILE`, `sensitive/SENSITIVE-FINDINGS.md`, and `hunt/phase-01-access.md`. Do not place seeds, `otpauth://` URIs, recovery codes, or live OTPs in `raw/`, normal logs, screenshots, or reports.
- A and B must have separate factors and seeds. Never enroll both accounts with one TOTP secret or treat A's recovery codes as B's.

## Step 1: Pre-flight

```bash
sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND list
cat "$YOUR_ACCOUNTS_FILE"

READ_KEY="$($YOUR_EMAIL_API_KEY_COMMAND)"
[ -n "$READ_KEY" ] || { echo "ERROR: email API key unavailable"; exit 1; }
curl -s -H "X-Api-Key: $READ_KEY" \
  "https://<EMAIL_API_HOST>/emails?to=health-check@<CATCH_ALL_DOMAIN_1>&limit=1" \
  -o /dev/null -w "%{http_code}\n"
# Expect 200. If not, email is down; flag and proceed without verification polling.

SUPPLIED_CREDENTIALS="$YOUR_TARGET_ROOT/sensitive/platform-supplied-credentials.json"
CREDENTIAL_STATUS="$YOUR_TARGET_ROOT/sensitive/platform-credential-status.json"
if [ -s "$CREDENTIAL_STATUS" ]; then
  # instruction and inquiry_description are non-secret program text and often
  # carry the real grant, including a required researcher email domain. Always
  # read them before classifying a signup rejection as a deterministic wall.
  jq '{source,platform,program_handle,state,claimed_value_count,instruction,
       assets:[.assets[]|{scope_id,identifier,available,claimed_by_you,
         inquiry_description,request_pending,requested_aliases,
         request_submitted_at}]}' \
    "$CREDENTIAL_STATUS"
fi
if [ -s "$SUPPLIED_CREDENTIALS" ]; then
  jq -e '
    .credentials.claimed | type == "array" and length > 0 and
    all(.[]; (.value | type == "string" and length > 0))
  ' "$SUPPLIED_CREDENTIALS" >/dev/null \
    || { echo "ERROR: malformed platform-supplied credential inventory"; exit 1; }
  jq '{source,platform,program_handle,
       claimed_count:(.credentials.claimed|length),
       credential_sets:[.credentials.claimed[]|(.bucket_name // .bucketName // "unlabelled")]}' \
    "$SUPPLIED_CREDENTIALS"
fi
```

### Blind-run identity rule

Create new unique identities for native signup. Never log in with an account found outside `$YOUR_ACCOUNTS_FILE`, except frozen run-local `sensitive/platform-supplied-credentials.json` when present, the configured social fallback after native failures, and the final matching platform-alias native fallback. The social exception uses dedicated researcher-owned identities and may create a new target account or reopen a prior target-side account linked to that identity. The platform-alias exception always creates a fresh unique plus alias for this run and account. Record `target_account_state=new|existing|unknown`; never claim an existing social account was freshly registered. Validate every usable identity live, then copy confirmed target credentials into the run-local ledger. Never read canonical or prior-run target credentials. If registration fails for a recoverable profile, IP, geo, or bot-wall reason, apply the five-profile ladder from Step 4.

When the supplied file exists, privately read each `credentials.claimed[]` value
and map its labelled account or role to the target login. Test supplied accounts
before creating Account A or B. A supplied account counts only after a live
authenticated view proves it works. Copy each validated account into
`$YOUR_ACCOUNTS_FILE` and `sensitive/SENSITIVE-FINDINGS.md`, then use it as A, B,
C, or D as appropriate.

If a supplied value is an email address without a password, do not try the shared
research password. Use the target's normal **Forgot password** flow with that
email, once. Record the request timestamp before submitting it. For a supplied
Bugcrowd or HackerOne researcher-domain address, query the forwarded destination
`$YOUR_FORWARDED_INBOX`; isolate only the matching reset message by subject and timestamp.
Set a fresh run-specific password, validate the real authenticated view, and save
the resulting account in the three approved sensitive locations. If reset is not
offered, no matching message arrives, or the reset remains blocked after the
bounded flow, mark that supplied identity unusable and continue native signup.

If the platform inventory says `request_pending`, `request_required`,
`claimed_no_values`, or `unavailable`, do not ask the user whether to continue
because of that state. Continue native signup and finish PARTIAL or UNAUTH if no
usable access is found.
Create fresh accounts only for missing independent identities or when every
supplied credential is proven unusable. Record failed supplied logins and exact
blockers without exposing secret values.

For every `request_pending` asset with `requested_aliases`, check forwarded mail
**before native signup**. HackerOne and Bugcrowd researcher-domain aliases
forward to `$YOUR_FORWARDED_INBOX`. Query only that destination, then filter in-process for
an exact requested alias, program handle, asset identifier, expected subject, and
mail newer than `request_submitted_at`. Never print or retain unrelated messages.
Some programs ignore the aliases typed in the inquiry and provision a different
HackerOne plus alias. If no exact requested-alias match exists, accept a message
only when its authenticated sender or forwarding headers identify the program,
its timestamp is newer than `request_submitted_at`, and its content is an account
activation, password setup, or reset for the expected asset. Record the actual
recipient alias as the account identity. A timestamp alone is never enough.
Search the active rolling store first at `/emails`. If it has no match, search the
historical store at `/archive/emails` with the same authenticated `to`, subject,
and limit filters. The active store holds only the newest 500 messages globally;
an active miss is not evidence that the program never sent mail. Archive results
are newest-last, like the active endpoint. Do not broaden either query beyond the
exact forwarded destination and run-owned identity markers.

An activation link, temporary password, password-setup link, or reset message is
supplied access: drive it in the target browser, validate the authenticated view,
and save the account in the three approved sensitive locations.

A supplied identity often carries no password at all because the target is
passwordless. When the credential value is an email or username and the target
logs in by magic link, email code, or social sign-in, that login route **is** the
credential: request the link or code for that exact supplied identity on the
target's normal login page, read it at the forwarded destination, and complete
sign-in. Do not treat a missing password as an unusable credential, and do not
look for a Forgot Password route on a target that has no password. When the
program issued one identity per tenant, drive each through the same route to get
A and B rather than registering anything natively.

If an archived
activation link or temporary password has expired, or neither store has matching
provisioning mail, try the target's Forgot Password flow once for each exact
requested alias when the target exposes a reset route. Record the reset request
time, then check the active store for only the matching newer reset message. A
successful reset proves that the program provisioned the account even when the
original message aged out. An unknown-account response, no matching reset mail,
or an absent reset route exhausts that alias without intervention. Check this
sequence once at Phase 1 start. Do not wait or ask the user when it yields no
access. Continue the normal native signup ladder, then PARTIAL or UNAUTH as
appropriate. A later platform sync may still turn the pending request into
claimed credential values for a future run.

## Step 2: Acquire Account A's first browser profile

Never hardcode a native-signup profile number. Pass a geo supported by your browser manager when the target is region-specific, or `acquire` with no geo when region does not matter. Keep any reserved social identities in the browser manager's own configuration, not in these prompts.

Acquire only A now. Do not reserve B before A proves a working configuration. B is acquired after A succeeds and starts on A's exact working geo/provider.

```bash
# EXPECTED_CC="us"  # target's expected 2-letter country from recon/locale
# GEO="us"          # set to EXPECTED_CC for us/gb/au/de/sg; otherwise leave empty and use <YOUR_PROXY_PROVIDER> below
OWNER_A="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_LEASE_CYCLE}:profile-a"
ACQ_A=$(python3 "$YOUR_HELPERS_ROOT/browser-lease.py" acquire \
  --owner "$OWNER_A" --role access-a --geo "${GEO:-}" --origins-file "$SCOPE_HOSTS_FILE")
PROFILE_A=$(echo "$ACQ_A" | grep -oE '^[0-9]+' | head -1)
[ -z "$PROFILE_A" ] && { echo "ERROR: failed to acquire profile A. Output: $ACQ_A"; exit 1; }
CDP_A=$((YOUR_CDP_BASE_PORT + PROFILE_A)); MITM_A=$((YOUR_MITM_BASE_PORT + PROFILE_A))

if [ -n "${EXPECTED_CC:-}" ] && [ -z "${GEO:-}" ]; then
  sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND country "$PROFILE_A" "$EXPECTED_CC"
fi

sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND list
echo "A=p$PROFILE_A CDP=$CDP_A mitm=$MITM_A"
sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND country "$PROFILE_A"
```

If `acquire` returns "no free profile", STOP. Do not release another client's profile. Surface contention.

A and B are not each other's fallbacks. If A fails, acquire a new profile for A. No B lease exists yet.

## Step 3: Pick identities

- **Names**: realistic first + last matching the target's locale. TLD is the primary signal (`.fi` Finnish, `.de` German, `.uk` British; `.com`/`.io`/`.net` default to US/UK English unless recon says otherwise). A and B differ. Never "Test", "BugBounty", "Admin", "asdf".
- **Email policy**: use configured catch-all domains in order: `<CATCH_ALL_DOMAIN_1>`, `<CATCH_ALL_DOMAIN_2>`, `<CATCH_ALL_DOMAIN_3>`, then `<CATCH_ALL_DOMAIN_4>`. Move to the next catch-all only after a deterministic domain rejection. If all configured domains fail, exhaust applicable social sign-in identities, then make one final native-signup attempt per required account with the matching platform alias.
- **Platform-alias final fallback**: HackerOne uses a fresh `<PLATFORM_ALIAS_BASE>+<unique>@<HACKERONE_ALIAS_DOMAIN>`; Bugcrowd uses a fresh `<PLATFORM_ALIAS_BASE>+<unique>@<BUGCROWD_ALIAS_DOMAIN>`. Use only the alias matching `{{platform}}`. Both forward to `$YOUR_FORWARDED_INBOX`; query that destination and filter by expected subject and request timestamp. Never use the bare alias, reuse an alias, or use a platform alias before the configured catch-alls and applicable social identities are exhausted. On platforms without a configured alias, record `platform alias unavailable` and continue PARTIAL or UNAUTH.
- **Supplied credentials are exempt from the alias-shape rules above.** Those rules govern identities *you* generate for native signup. An identity the program provisioned in `sensitive/platform-supplied-credentials.json` is used exactly as issued, including a bare `<PLATFORM_ALIAS_BASE>@<HACKERONE_ALIAS_DOMAIN>` with no plus suffix, and including reuse across A and B when the program issued one identity per tenant. Never regenerate, substitute, or add a plus suffix to a supplied identity, and never reject one for failing the freshness or plus-alias rule.
- **Phone**: use a researcher-owned test number permitted by the target and your program rules. A and B must use distinct numbers when the workflow requires it. Never use a real person's number.
- **DOB**: pre-1990. A and B can share.
- **Password**: start with a researcher-chosen test password stored outside the prompt. If the target explicitly rejects it under a concrete password policy, make the smallest compliant change and record the actual account password. A password-policy rejection is not a domain, captcha, profile, or egress failure.

```bash
TS=$(date +%s)
PASSWORD="<ACCOUNT_PASSWORD>"

A_FIRST="..."; A_LAST="..."; PHONE_A="..."
B_FIRST="..."; B_LAST="..."; PHONE_B="..."
DOB="..."   # pre-1990, e.g. 1984-03-19

slug() { echo "$1.$2" | iconv -t ASCII//TRANSLIT 2>/dev/null | tr '[:upper:]' '[:lower:]' | tr -cd 'a-z.' ; }
ACCOUNT_EMAIL_POLICY="catchall"  # final fallback may become platform-alias
CATCHALL_DOMAIN="<CATCH_ALL_DOMAIN_1>"  # retry order: <CATCH_ALL_DOMAIN_1>, <CATCH_ALL_DOMAIN_2>, <CATCH_ALL_DOMAIN_3>, <CATCH_ALL_DOMAIN_4>

case "$CATCHALL_DOMAIN" in
  <CATCH_ALL_DOMAIN_1>|<CATCH_ALL_DOMAIN_2>|<CATCH_ALL_DOMAIN_3>|<CATCH_ALL_DOMAIN_4>) ;;
  *) echo "ERROR: invalid catch-all domain"; exit 1 ;;
esac
ACCT_A_EMAIL="$(slug "$A_FIRST" "$A_LAST").${TS}@${CATCHALL_DOMAIN}"
ACCT_B_EMAIL="$(slug "$B_FIRST" "$B_LAST").${TS}@${CATCHALL_DOMAIN}"
MAILBOX_QUERY_A="$ACCT_A_EMAIL"; MAILBOX_QUERY_B="$ACCT_B_EMAIL"

echo "A: $A_FIRST $A_LAST  $ACCT_A_EMAIL  $PHONE_A  p$PROFILE_A"
echo "B: $B_FIRST $B_LAST  $ACCT_B_EMAIL  $PHONE_B  profile=pending-A-result"
echo "Email policy: $ACCOUNT_EMAIL_POLICY  mailbox A=$MAILBOX_QUERY_A mailbox B=$MAILBOX_QUERY_B"
```

Record `ACCOUNT_EMAIL_POLICY`, `CATCHALL_DOMAIN`, any platform alias used, and every domain rejection in `hunt/account-creation-process.md`. Before requesting each verification message, record UTC send time. Query only the exact catch-all account address for catch-all identities. For a platform alias, query destination `$YOUR_FORWARDED_INBOX` and isolate the message by expected subject and request timestamp.

The email ladder is bounded and separate from the five-profile egress ladder. On an exact `email domain not allowed`, `invalid email`, or equivalent rejection before account creation, try `<CATCH_ALL_DOMAIN_1>`, then `<CATCH_ALL_DOMAIN_2>`, then `<CATCH_ALL_DOMAIN_3>`, then `<CATCH_ALL_DOMAIN_4>` in the same profile. After all configured domains fail, continue through applicable social identities first. If social sign-in is unavailable or exhausted, use one final matching platform-alias native retry at the most promising proven native condition. Do not treat captcha, generic 403, timeout, fraud rejection, or `security_not_allowed` as proof an email domain failed. Those remain profile or egress diagnostics.

## Step 4: Register Account A

**You do this one yourself.** A is the reference run; everything you learn here becomes the recipe B replays.

```bash
REG_URL=""  # paste exact Registration page URL from phase-00-recon.md Auth Surface
if [ ! -s "$SCOPE_HOSTS_FILE" ]; then
  case "$TARGET" in
    wild.*) echo "ERROR: wildcard target has no in-scope scope-hosts.txt"; exit 1 ;;
    *)      echo "$APEX" > "$SCOPE_HOSTS_FILE" ;;
  esac
fi
if [ -z "$REG_URL" ]; then
  case "$TARGET" in
    wild.*) DEFAULT_REG_HOST=$(head -1 "$SCOPE_HOSTS_FILE" 2>/dev/null) ;;
    *)      DEFAULT_REG_HOST="$APEX" ;;
  esac
  [ -n "$DEFAULT_REG_HOST" ] || { echo "ERROR: no registration seed host"; exit 1; }
  REG_URL="https://$DEFAULT_REG_HOST/"
fi
echo "REG_URL=$REG_URL"
```

Reconcile registration host before navigation. If `REG_URL` is a first-party functional sibling documented by Phase 0, or directly reached from the listed target's signup flow now, add it to both files before authenticated walk. Record concrete redirect, link, form action, or API evidence. Never add a third-party IdP or payment host.

```bash
REG_HOST=$(printf '%s\n' "$REG_URL" | sed -E 's|^[A-Za-z][A-Za-z0-9+.-]*://||; s|/.*||; s|:[0-9]+$||' | tr '[:upper:]' '[:lower:]')
FUNCTIONAL_SIBLING_HOST=""  # set to REG_HOST only after the evidence review above
FUNCTIONAL_ROLE="auth/account"
FUNCTIONAL_EVIDENCE=""      # exact listed-target URL and redirect/link/form evidence
if [ -n "$FUNCTIONAL_SIBLING_HOST" ]; then
  [ "$FUNCTIONAL_SIBLING_HOST" = "$REG_HOST" ] || { echo "ERROR: sibling does not match registration host"; exit 1; }
  [ -n "$FUNCTIONAL_EVIDENCE" ] || { echo "ERROR: functional sibling evidence is required"; exit 1; }
  printf '%s\t%s\t%s\n' "$FUNCTIONAL_SIBLING_HOST" "$FUNCTIONAL_ROLE" "$FUNCTIONAL_EVIDENCE" \
    >> raw/functional-sibling-hosts.tsv
  { cat "$SCOPE_HOSTS_FILE"; printf '%s\n' "$FUNCTIONAL_SIBLING_HOST"; } \
    | sort -u > "$SCOPE_HOSTS_FILE.tmp"
  mv "$SCOPE_HOSTS_FILE.tmp" "$SCOPE_HOSTS_FILE"
fi
```

Open Chrome via CDP on `http://$YOUR_CDP_HOST:$CDP_A`, navigate the registration URL, read the page, do the next thing. Repeat until authed.

**Tools:** CDP (`/json/list`, `Page.navigate`, `Runtime.evaluate`, `Network.getCookies`, `Input.dispatchKeyEvent`); <YOUR_CAPTCHA_SOLVER> in-page; catch-all email at `GET https://<EMAIL_API_HOST>/emails?to=<exact-account-address>` with `X-Api-Key` loaded from your protected secret store; traffic capture at `$YOUR_BROWSER_FLOW_DIR/p$PROFILE_A/current.flow`.

**Done means:** form submitted, verification completed, page on a real authed view, a session credential present in cookies / localStorage / sessionStorage, and any mandatory MFA enrolled with reusable access material saved. A live session whose mandatory device-bound factor cannot be recovered is `MFA session-only`, not durable reusable access.

That is registration done, not *access* done. Step 6 is where you establish whether there is anything behind the login. Do not declare A finished at an onboarding wall.

### Five-profile native ladder, Google social fallback, then platform alias

Record a diagnostics row before each pass:

| Account | Pass | Profile | Egress mode and country | Outcome | Released | Failure signature |
|---------|------|---------|-------------------------|---------|----------|-------------------|
| A | 1/2/3/4/5 or social | pN | `us-ws`, `gb-sticky`, `google-native-tls` | success/fail | yes/no + reason | exact page error, HTTP status, captcha vendor, or timeout |

Run `browser-profile-command country N` before navigation to capture live egress. Never use rotating mode for signup; IP rotation breaks sessions.

**Pass 1:** the Step 2 profile, expected geo. One complete pass. Multi-step pages and email verification are part of this single pass.

Track the winning lease explicitly. Initialize before the first pass:

```bash
WINNING_PROFILE_A="$PROFILE_A"
WINNING_OWNER_A="$OWNER_A"
WINNING_GEO_A="${GEO:-${EXPECTED_CC:-}}"
WINNING_PROVIDER_A=<YOUR_PROXY_PROVIDER>
WINNING_CAPTURE_MODE_A=mitm
AUTH_METHOD_A=native
```

**Before every replacement:** poll email and try login to determine whether the previous pass actually created the account. Save diagnostics. If it is confirmed unauthenticated, release it immediately with `browser-lease.py release --profile N --owner OWNER --role access-a-failed-pass-N`. If state is authenticated or uncertain, keep it held and reconcile before another submission.

**Pass 2, fresh profile, same geo and <YOUR_PROXY_PROVIDER> provider:** required if Pass 1 fails recoverably. This separates one exit-IP failure from a geo/provider failure.

```bash
ACCOUNT_LABEL=a
PASS_N=2
RETRY_OWNER="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_LEASE_CYCLE}:profile-${ACCOUNT_LABEL}-retry-${PASS_N}"
ACQ_RETRY=$(python3 "$YOUR_HELPERS_ROOT/browser-lease.py" acquire \
  --owner "$RETRY_OWNER" --role "access-${ACCOUNT_LABEL}-retry" --geo "${GEO:-}" \
  --origins-file "$SCOPE_HOSTS_FILE")
PROFILE_RETRY=$(echo "$ACQ_RETRY" | grep -oE '^[0-9]+' | head -1)
# Do not exit on one unavailable retry. Record it and continue through the
# remaining planned conditions. After the ladder, derive access automatically unless an actionable CAPTCHA remains.
if [ -z "$PROFILE_RETRY" ]; then
  echo "WARN: pass $PASS_N unavailable; continue remaining conditions"
else
  CDP_RETRY=$((YOUR_CDP_BASE_PORT + PROFILE_RETRY)); MITM_RETRY=$((YOUR_MITM_BASE_PORT + PROFILE_RETRY))
  if [ -n "${EXPECTED_CC:-}" ] && [ -z "${GEO:-}" ]; then
    sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND country "$PROFILE_RETRY" "$EXPECTED_CC"
  fi
  sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND country "$PROFILE_RETRY"
fi
```

**Passes 3 through 5 use distinct useful conditions:**

- **Pass 3:** fresh <YOUR_PROXY_PROVIDER> profile in a different supported geo. Prefer `us` when Passes 1 and 2 were not US; prefer `gb` when they were US. Warm-browse the target origin first.
- **Pass 4:** change provider. If country-locked, use a fresh profile with a validated sticky <YOUR_PROXY_PROVIDER> exit in the expected country. Otherwise use another supported <YOUR_PROXY_PROVIDER> geo not yet tried. Never use rotating mode.
- **Pass 5:** use the most promising condition reached so far with a fresh exit IP. If no pass progressed further than the others, use an untried <YOUR_PROXY_PROVIDER> geo or same-country <YOUR_PROXY_PROVIDER>. Do not duplicate Pass 4's geo, provider, and exit condition without recording that all alternatives failed validation or were unavailable.
- Verify live country and IP before navigation. Retry <YOUR_PROXY_PROVIDER> validation once. A dead proxy validation does not consume a signup pass; acquire another condition.

Classify conservatively:
- One profile fails, later same-geo profiles succeed → profile / exit-IP block.
- Both <YOUR_PROXY_PROVIDER> fail, same-geo <YOUR_PROXY_PROVIDER> succeeds → <YOUR_PROXY_PROVIDER> provider block.
- Two same-geo <YOUR_PROXY_PROVIDER> profiles fail, alternate-geo succeeds → original geo/provider reputation condition; do not call it an application blocker.
- All 5 fail with the same deterministic validation response → **application or identity requirement, not profile-specific.** Continue to Google fallback if offered, then the matching platform-alias fallback when relevant, before using this classification at Step 9.

Whenever Pass 2 through Pass 5 succeeds, immediately set `WINNING_PROFILE_A`, `WINNING_OWNER_A`, `WINNING_GEO_A`, `WINNING_PROVIDER_A`, and `WINNING_CAPTURE_MODE_A=mitm`. Then rebind the canonical A variables before writing the recipe, provisioning, or dispatching the walker:

```bash
if [ "$WINNING_PROFILE_A" != "$PROFILE_A" ]; then
  PROFILE_A="$WINNING_PROFILE_A"
  OWNER_A="$WINNING_OWNER_A"
  CDP_A=$((YOUR_CDP_BASE_PORT + PROFILE_A)); MITM_A=$((YOUR_MITM_BASE_PORT + PROFILE_A))
fi
echo "winning A profile: p$PROFILE_A CDP=$CDP_A mitm=$MITM_A owner=$OWNER_A"
```

Do not dispatch the walker with the original failed profile. Save its diagnostics, then release only that failed lease as shown.

**Social fallback after five native passes:**

Use it only when a visible supported social sign-in control exists on the target. Try available configured social identities in order, skipping identities already attempted for this target. Each social identity is used for at most one target account and one pass. Never reuse A's social identity for B.

Identity map: use only researcher-owned social identities configured for this run. Keep their addresses and recovery material outside the prompt. If the provider requests recovery, use a fresh challenge timestamp and the configured recovery destination or SMS inbox. Query only the matching newer challenge. After applicable social identities, automated recovery, and the platform-alias fallback are exhausted, derive PARTIAL or UNAUTH without asking unless an actionable CAPTCHA remains.

```bash
SOCIAL_OWNER="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_LEASE_CYCLE}:profile-a-social-${SOCIAL_N}"
SOCIAL_ACQ=$(python3 "$YOUR_HELPERS_ROOT/browser-lease.py" prepare-social \
  --profile "$SOCIAL_N" --owner "$SOCIAL_OWNER" --role access-a-google \
  --origins-file "$SCOPE_HOSTS_FILE")
```

`prepare-social` clears only target-origin cookies, storage, cache, and tabs. It preserves Google cookies and verifies target cookies are empty. Drive the target's real Google button via CDP. Do not navigate Google account settings, change recovery data, or test Google. A successful social pass must end on a real target-authenticated view with a target session credential.

After OAuth, classify whether the target created a new account, reopened an existing Google-linked account, or leaves that unknown. Existing researcher-owned target state is usable fallback access but must be recorded as such. Distinct A/B Google identities remain mandatory for cross-account work.

On social failure, record the target error and release that exact owner through `browser-lease.py release` before trying the next unused identity. On success set `WINNING_PROFILE_A=$SOCIAL_N`, `WINNING_OWNER_A=$SOCIAL_OWNER`, `WINNING_GEO_A=<observed-geo>`, `WINNING_PROVIDER_A=<YOUR_PROXY_PROVIDER>-native-tls`, `WINNING_CAPTURE_MODE_A=cdp-only-native-tls`, and `AUTH_METHOD_A=social`; then recompute `PROFILE_A`, `OWNER_A`, `CDP_A`, and `MITM_A`. Keep the winner leased. The authenticated walk uses CDP route evidence, ignores any flow file, and cannot be RICH solely from this capture.

**Platform-alias native fallback after Google:**

Use this once per required account only after all four catch-all domains were deterministically rejected or the target explicitly requires the matching platform researcher domain, and applicable Google identities are unavailable or exhausted. Do not use it to relabel a captcha, bot, IP, geo, KYC, payment, invite, or generic fraud rejection as an email-domain failure.

```bash
case "{{platform}}" in
  hackerone) PLATFORM_ALIAS_DOMAIN="<HACKERONE_ALIAS_DOMAIN>" ;;
  bugcrowd)  PLATFORM_ALIAS_DOMAIN="<BUGCROWD_ALIAS_DOMAIN>" ;;
  *)         PLATFORM_ALIAS_DOMAIN="" ;;
esac

if [ -n "$PLATFORM_ALIAS_DOMAIN" ]; then
  ACCOUNT_EMAIL_POLICY="platform-alias"
  ACCT_A_EMAIL="<PLATFORM_ALIAS_BASE>+workflow-a-${TS}@${PLATFORM_ALIAS_DOMAIN}"
  MAILBOX_QUERY_A="$YOUR_FORWARDED_INBOX"
else
  echo "platform alias unavailable for {{platform}}"
fi
```

Use a fresh flow-capable profile configured to the most promising proven native geo/provider condition. This is one identity fallback attempt, not a new five-profile egress ladder. Drive native signup with the new alias, record UTC send time, then query `$YOUR_FORWARDED_INBOX` for only the matching newer subject or request. On failure, save diagnostics and release that exact lease. On success set the literal winning profile and owner, `WINNING_CAPTURE_MODE_A=mitm`, `AUTH_METHOD_A=platform-alias`, and `target_account_state=new`; keep the winner leased. Account B uses `<PLATFORM_ALIAS_BASE>+workflow-b-${TS}@${PLATFORM_ALIAS_DOMAIN}` and its own fresh profile. Add a further unique suffix if any generated alias was already attempted.

**Bot-wall handling.** Identify the vendor and apply the fix to the *next fresh profile*:
- **Kasada** (`KPSDK` / `ips.js` + `KP_UIDz` cookie; blank or stuck page): caused by stealth injector JS patches. Profiles default to webrtc-only, which passes. If the full set is on, `echo off > $YOUR_STEALTH_MODE` or remove `$YOUR_STEALTH_FEATURES` before the next acquire. MITM is irrelevant to Kasada.
- **CDN bot protection** (`_abck=~-1~` + `403 Access Denied` / `cdn.example` on POST): MITM TLS fails the edge sensor; the flow needs MITM off. Stealth is irrelevant.
- **Captcha unresolved after ~30s:** classify before rotating. Record provider, version, iframe host, visible challenge text, response-token length, and whether the flow contains a <YOUR_CAPTCHA_SOLVER> recognition request. A recognition request plus an empty target token means the solver acted but the challenge was rejected. No recognition request with a custom iframe host usually means an injection or unsupported-widget gap.
- **Supported token captcha fallback:** before rotating, run at most one <YOUR_CAPTCHA_SOLVER> token job for that distinct sitekey/profile condition. Currently the helper supports hCaptcha:

  ```bash
  python3 "$HELP/captcha-fallback.py" \
    --profile "$PROFILE_A" --provider hcaptcha --url-prefix "$REG_URL"
  ```

  It reads the key from the managed extension, sends the exact profile <YOUR_PROXY_PROVIDER> proxy, cookies, page URL, sitekey, and real browser User-Agent through <YOUR_CAPTCHA_SOLVER>'s documented Token API, then injects through the widget's page callback. It never prints the key, proxy credentials, or token. It rejects an active <YOUR_PROXY_PROVIDER> override because that would make the API solve from a different IP than Chrome. It does **not** submit the target form. Submit normally and verify the target response. `token-injected-target-unverified` is not success. A provider timeout remains unresolved and proceeds to the next useful profile condition. A target rejection, such as HTTP 422, remains a captcha failure and must not be relabeled as an email-domain rejection. Custom vendor endpoints can reject an otherwise valid standard hCaptcha token.
- **Legacy GeeTest v3 (`.geetest_panel`):** <YOUR_CAPTCHA_SOLVER> v0.6.1 may detect the panel without moving the slider. <YOUR_CAPTCHA_SOLVER> has a GeeTest recognition API but no GeeTest Token API. Do not claim API fallback success unless a version-matched drag executor reaches the target's verified state. Otherwise continue the bounded profile/geo ladder, then Google fallback when offered.
- **Fingerprint reject, form error, blank page, or two fill attempts with no progress:** next fresh profile.

**If all five native profiles, all applicable social identities, and the matching platform-alias fallback fail or are unavailable for A:** do not dispatch subagents. Record the exact failure and derive UNAUTH or PARTIAL without asking. Only go to the CAPTCHA intervention block if the final failure is an unresolved actionable CAPTCHA or anti-bot challenge. If no supported social button exists, record `social fallback unavailable` rather than attempting unrelated IdPs. If `{{platform}}` is neither HackerOne nor Bugcrowd, record `platform alias unavailable` rather than borrowing another platform's domain.

## Step 5: Write the recipe

`hunt/account-creation-process.md`. This is what the registrar subagent replays. Write it immediately after A succeeds, while the detail is fresh.

It must be operational, not prose. B should be able to follow it without re-deriving anything.

```markdown
# Account creation process, $TARGET
Date: {date}
Derived from: Account A, profile p{PROFILE_A}, pass {N}

## Working configuration
- Registration URL: {REG_URL}
- Account email policy: {catchall / google / platform-alias}
- Catch-all domain: {<CATCH_ALL_DOMAIN_1> / <CATCH_ALL_DOMAIN_2> / <CATCH_ALL_DOMAIN_3> / <CATCH_ALL_DOMAIN_4> / not-applicable}
- Platform alias: {exact unique alias / not-used}
- Email policy reason: identity ladder outcome; include rejected catch-all, Google, and platform-alias diagnostics
- Inbox query: {exact catch-all account address / $YOUR_FORWARDED_INBOX for platform alias / social recovery destination}
- Functional auth/account sibling (included in `raw/scope-hosts.txt`): {host, role, evidence; or none}
- Geo: {us/gb/...}  Egress mode: {us-ws / gb-sticky / ...}
- Winning profile: p{PROFILE_A} (native pass {N} of 5, social fallback, or platform-alias native fallback)
- Stealth setting required: {default webrtc-only / off}
- Capture: {traffic capture for catch-all or platform-alias native signup / CDP-only native TLS for social fallback}

## Page sequence
1. {URL} - {what is on it}
   - fields: {name/id/selector} ← {value shape, e.g. "first name"}
   - React prototype-setter needed: {yes/no}
   - submit: {selector or action}
   - what indicates success: {text, redirect, status}
2. {URL} - ...
3. ...

## Captcha
- Provider: {recaptcha / hCaptcha / Turnstile / Cloudflare / GeeTest / none}
- Version and frame host: {e.g. hCaptcha custom `newassets-captcha.example`; GeeTest v3/v4}
- Appeared at: {which step}
- <YOUR_CAPTCHA_SOLVER> cleared it: {yes / no, took ~Ns}
- Extension evidence: {recognition request seen / no request; final token length; exact challenge text}
- Token API fallback: {not applicable / not attempted / injected then target accepted / injected then target rejected with status}

## Verification
- Type: {email / SMS / none}
- Address or number used: {value}
- Poll: {exact endpoint and how the link or OTP was extracted}
- Typical delay: {seconds}

## MFA
- Required for: {signup / onboarding / login / product access / optional / none}
- Method: {none / email-otp / sms-otp / totp / push / passkey / webauthn / vendor method}
- Enrollment steps: {ordered steps sufficient for B to enroll its own distinct factor}
- Destination or enrolled device/profile: {researcher-owned destination / profile / not-applicable}
- TOTP secret saved: {yes / no / not-applicable; never put the secret itself in this recipe}
- Recovery codes saved: {count / none issued / not-applicable; never put code values in this recipe}
- Re-login verified with saved factor: {yes / no and reason}

## Success signal
- Final authed URL: {url}
- Session credential: {cookie name / localStorage key / header}

## Provisioning
Left blank at Step 5; filled in at Step 6 once you have actually provisioned A's
resource. B cannot provision its own without this section.
- Primary resource required: {yes/no}
- Steps to create it: {ordered, or "none"}

## Resource graph
Left blank at Step 5; filled in at Step 6. For every relevant type, state creation steps, owner field, ID location, reversible cleanup, and whether B can create an equivalent.
- Tenant/account: {...}
- Primary business object: {...}
- Workflow instance and status transition: {...}
- Upload or attachment: {...}
- Invitation or member: {...}
- Subscription, order, booking, or entitlement: {...}

## What did NOT work
| Pass | Profile | Egress | Failure | Released | Interpretation |
|------|---------|--------|---------|----------|----------------|
| 1 | pN | us-ws | {exact error} | {yes/no + reason} | {profile-specific / geo / deterministic} |

Do not repeat these. If a listed approach is the only one left, return blocked instead.
```

## Step 6: Provision A's resource graph

An account is not the goal. A *usable product surface* is. Registration can succeed while leaving the account at an empty onboarding shell, which gives later phases little meaningful product state to test.

**Signals you are looking at an empty shell:** an incomplete onboarding wizard; a dashboard with one "create your first X" CTA and no other nav; nav items that all route back to setup; empty collections everywhere in the API.

**If so, provision it.** Create the primary service, project, workspace, organisation, tenant, store, instance, cluster, or equivalent resource the product is built around. Use a free tier or trial credits already granted to the account. Choose the smallest or lowest-cost option available. Never add a payment card, spend real money, complete KYC, or accept a sales-assisted or paid contract. This is normal onboarding, not testing.

Preserve resource state for later phases. Record type, tier, region, resource ID, owner identity, parent ID, console URL, creation endpoint, current state, and any displayed trial-credit limit or expiry.

```bash
PRIMARY_RESOURCE_A="<resource type, tier/region, resource id, console URL, trial-credit limit/expiry if shown; or 'none required'>"
echo "PRIMARY_RESOURCE_A=$PRIMARY_RESOURCE_A"
```

Then build `raw/resource-graph.json`. Start from menu routes, `raw/operation-candidates.jsonl`, browser traffic, and objects created during onboarding. For each relevant type, create the smallest safe A-owned object and record whether a matching B-owned object is required. Assess every canonical category, even when product has no matching surface:

- `tenant-account`;
- `primary-business-object`;
- `workflow-instance`;
- `upload-attachment`;
- `invitation-member`;
- `subscription-order-entitlement`;
- `status-transition`.

Each category status is `usable`, `gap`, or `not-exposed`. `usable` requires a real A-owned object with file-backed ownership evidence. `gap` requires a matching gap record that preserves relevant Phase 2 work as PARTIAL. `not-exposed` requires evidence showing the authenticated navigation, APIs, and operation ledger were checked. Placeholder, demo, sample, unknown, or empty-state IDs never count as usable objects.

Use this shape:

```json
{
  "schema_version":1,
  "access_mode":"PARTIAL",
  "generated_at":"2026-01-01T00:00:00Z",
  "assessed_categories":[
    {"category":"primary-business-object","status":"usable","reason":"owned patient created","evidence":["raw/responses/patient-a.json"]},
    {"category":"workflow-instance","status":"gap","reason":"creation prerequisite unavailable","evidence":["raw/responses/journey-list.json"]}
  ],
  "objects":[
    {"category":"primary-business-object","type":"patient","owner":"A","id":"patient-7341","parent_id":"tenant-82","endpoint":"POST https://target/api/patients","ownership_evidence":["raw/responses/patient-a.json"],"state":"created","reversible":true,"b_equivalent_required":true}
  ],
  "gaps":[
    {"category":"workflow-instance","type":"journey","reason":"creation requires unavailable prerequisite","evidence":["raw/responses/journey-list.json"],"phase02_effect":"PARTIAL for workflow and object-access checks"}
  ]
}
```

Append exact creation steps to recipe Provisioning and Resource graph sections so B can create equivalents. **B needs its own equivalents for every cross-tenant-relevant type**, not a share of A's. A missing child object does not prove a class dead. Record gap and route relevant Phase 2 work as PARTIAL.

If provisioning requires payment, a payment card, KYC, a sales call, or a paid contract, that is a hard blocker. Set `PROVISION_BLOCKED_A` with the exact reason and still fork because there is a surface to walk and B to establish. These blockers are recorded and resolve automatically to PARTIAL or UNAUTH in Step 9. Do not stop the target or let the wall pass silently.

```bash
PROVISION_BLOCKED_A=""   # e.g. "service creation requires a paid plan"; empty if fine
```

## Step 7: Fork

Acquire B only now, using A's proven route:

- If A won through <YOUR_PROXY_PROVIDER>, acquire B in `WINNING_GEO_A`.
- If A won through same-country <YOUR_PROXY_PROVIDER>, acquire a fresh flow-capable profile in the expected <YOUR_PROXY_PROVIDER> geo, then apply a fresh validated sticky <YOUR_PROXY_PROVIDER> exit in A's winning country.
- If A won through social sign-in, skip native B retries that A already proved ineffective. Prepare the next unused social identity and replay the same target flow.
- If A won through a platform alias, acquire a fresh flow-capable profile in A's winning native geo/provider and use B's distinct matching platform alias. Never reuse A's plus alias.

```bash
OWNER_B="${YOUR_LEASE_OWNER_PREFIX}:${YOUR_LEASE_CYCLE}:profile-b"
if [ "${WINNING_CAPTURE_MODE_A:-mitm}" = "mitm" ]; then
  if [ "${WINNING_PROVIDER_A:-<YOUR_PROXY_PROVIDER>}" = "<YOUR_ALTERNATE_PROXY_PROVIDER>" ]; then
    B_START_GEO="${GEO:-}"
  else
    B_START_GEO="${WINNING_GEO_A:-${GEO:-}}"
  fi
  ACQ_B=$(python3 "$YOUR_HELPERS_ROOT/browser-lease.py" acquire \
    --owner "$OWNER_B" --role access-b --geo "$B_START_GEO" \
    --origins-file "$SCOPE_HOSTS_FILE")
  PROFILE_B=$(echo "$ACQ_B" | grep -oE '^[0-9]+' | head -1)
  [ -n "$PROFILE_B" ] || { echo "WARN: B pass 1 unavailable; registrar continues remaining conditions"; }
  if [ -n "$PROFILE_B" ]; then
    CDP_B=$((YOUR_CDP_BASE_PORT + PROFILE_B)); MITM_B=$((YOUR_MITM_BASE_PORT + PROFILE_B))
    if [ "${WINNING_PROVIDER_A:-<YOUR_PROXY_PROVIDER>}" = "<YOUR_ALTERNATE_PROXY_PROVIDER>" ]; then
      sudo -u "$YOUR_BROWSER_USER" $YOUR_BROWSER_PROFILE_COMMAND country "$PROFILE_B" "$WINNING_GEO_A"
    fi
  fi
else
  PROFILE_B=""; CDP_B=""; MITM_B=""
fi
```

Dispatch both subagents in one message after this preparation. A social-login A is CDP-only, so its walker produces route coverage without mitm evidence. The registrar owns B acquisition retries and can start from no pre-acquired B profile when A's proven route is social.

### Rules that apply to both

- **Neither subagent talks to the user.** No notify, no waiting for a human. If blocked, return a verdict and stop.
- **Neither writes a shared file.** Run-local `$YOUR_ACCOUNTS_FILE` and `SENSITIVE-FINDINGS.md` are orchestrator-only. Subagents own only their own artifacts.
- **Distinct lease owners.** The registrar uses label-specific `...:profile-b*` owners. The walker never acquires a profile; it inherits p{PROFILE_A}.
- **Open with the defensive prelude.** Subagents do not inherit the orchestrator's shell. First two lines of any bash: `set +u` and the `: "${VAR:=}"` defaults they need. A subagent that inherits `set -u` aborts on the first unset reference otherwise.
- Return a JSON verdict as the final message, nothing else.

Substitute the literal values for every `$VAR` below before dispatching. The subagents get text, not your shell.

### Subagent 1: walker

> You are walking an already-authenticated Chrome profile for target `$TARGET`. Do not register anything, do not log out, do not submit forms, do not click destructive buttons. Do not acquire a browser profile; you inherit one. Start any bash with `set +u`.
>
> Set these first, with the literal values substituted:
> `export BASE="$BASE" TARGET="$TARGET" APEX="$APEX"`
> `export PROFILE_A=$PROFILE_A CDP_A=$CDP_A`
> `export EP=$YOUR_HELPERS_ROOT/bin/endpoints HELP="$HELP" SCOPE_HOSTS_FILE="$SCOPE_HOSTS_FILE"`
>
> 1. Dump auth state:
>    `python3 $HELP/dump-auth.py $CDP_A a $YOUR_TARGET_ROOT/raw/auth-a.json $SCOPE_HOSTS_FILE`
>    If it reports `authenticated=false` but cookies exist, inspect the JSON; if you can identify a credential cookie or storage key, treat A as authenticated and say so in your note.
> 2. Build `$YOUR_TARGET_ROOT/raw/route-manifest-candidates.txt`, one absolute in-scope browser route per line. Extract SPA route tables, menu/navigation links, authenticated JS route strings, source maps, and routes exposed by `raw/resource-graph.json`. Add locale variants or any manual extras to `raw/prewalk-extra.txt`. Do not put API POST/PUT/PATCH/DELETE URLs in browser route files.
> 3. Run the walk: `bash $HELP/walk.sh`
>    It builds URL union, adaptively follows visible in-scope menu links, and writes actual navigation evidence to `raw/auth-route-coverage.jsonl`. On flow-capable profiles it also mines traffic capture, extracts auth-only JS chunks and source maps, collapses shapes, and runs method probes. On CDP-only social profiles, `capture_mode=cdp-only-native-tls`; traffic-capture counts are zero and must never be presented as full authenticated coverage.
> 4. If `new_js_n` > 10, spawn a nested JS-analysis subagent over `raw/js/authwalk-*.js` (secrets, endpoints, internal hosts, admin URLs, role constants) writing `raw/js-analysis-authwalk.txt`.
> 5. Read `raw/recon-endpoints-auth-only.txt`, `raw/auth-route-coverage.jsonl`, and `raw/operation-candidates.jsonl`. Append newly observed authenticated operations to operation ledger with source `auth-js`, `browser`, or `mitm`; never replace Phase 0 records. Every appended candidate must include canonical Phase 02 `classes`. High-value candidates require at least one class. Pick 5-10 strongest entries for `top_auth_only`.
>
> Return exactly, with the counts copied from `raw/walk-counts.json`:
> `{"role":"walker","status":"ok|thin|failed","auth_a_ok":bool,"prewalk_n":int,"route_pending_n":int,"unauth_n":int,"authwalk_n":int,"merged_n":int,"auth_only_n":int,"new_js_n":int,"new_maps_n":int,"top_auth_only":["METHOD URL",...],"anomalies":["..."],"note":"..."}`
>
> Set `status` to `thin` if `auth_only_n` is under 20, and say in `note` which cause it looks like: **dead auth** (the prewalk log shows everything redirecting to `/login`) or **un-provisioned surface** (few routes existed to walk in the first place). The orchestrator acts differently on each.

### Subagent 2: registrar

> You are establishing account B for target `$TARGET` by replaying a known-good recipe. Read `$YOUR_TARGET_ROOT/hunt/account-creation-process.md` first and follow it exactly. It records what worked for account A and what did not; do not repeat the failed approaches. Start any bash with `set +u`.
>
> **Mode: REGISTER_OR_IDENTITY_FALLBACK.** Drive native signup from the recipe using a new run-unique identity. Prior target-account reuse is forbidden. Dedicated social identities are allowed only after the native ladder, or immediately when A already proved native signup exhausted and social sign-in worked. A fresh matching platform alias is allowed only after the configured catch-alls and applicable social identities are exhausted, or immediately when A proved that exact platform-alias path works.
>
> Inputs: `BASE=$BASE`, `TARGET=$TARGET`, initial profile `p$PROFILE_B` when present, CDP `$CDP_B`, mitm `$MITM_B`, helpers in `$HELP`, scope hosts `$SCOPE_HOSTS_FILE`. Identity: `$B_FIRST $B_LAST`, `$ACCT_B_EMAIL`, `$PHONE_B`, password `$PASSWORD`, DOB `$DOB`. A's proven configuration is `WINNING_GEO_A`, `WINNING_PROVIDER_A`, and `WINNING_CAPTURE_MODE_A`; start there. Read the recipe and its attempted social-profile list.
>
> 1. Establish B via CDP on `$CDP_B`, following the recipe's page sequence.
> 2. Up to 5 distinct native-signup profiles total. Pass 1 uses A's exact proven geo/provider. Pass 2 uses a fresh IP under that same condition. Pass 3 uses an alternate <YOUR_PROXY_PROVIDER> geo when not country-locked. Pass 4 changes provider with same-country sticky <YOUR_PROXY_PROVIDER> when country-locked, otherwise another untried <YOUR_PROXY_PROVIDER> geo. Pass 5 uses the most promising condition with a fresh exit or an untried validated condition. Warm-browse before changed-egress passes. Record diagnostics before each pass. After each confirmed unauthenticated failure, save evidence and immediately release that exact lease through `browser-lease.py release --profile N --owner OWNER --role access-b-failed-pass-N`. Never release authenticated or uncertain state.
> 2a. If SMS is mandatory, exhaust the fixed US, UK, <YOUR_SMS_PROVIDER> AU, and AU device inventory in documented order. Then use `$HELP/your-sms-helper.py`: cheapest different-country `opt19` first, exact target service second, hard maximum USD 2.00 per number, and no more than three <YOUR_SMS_PROVIDER> allocations for B. Cancel a site-rejected number immediately. Block an order after 580 seconds without SMS. Record quote, order, and cleanup evidence.
> 3. If all five native passes fail and the target visibly offers a supported social sign-in, try unused social identities through `browser-lease.py prepare-social`. Preserve provider cookies, clear only target-origin state, and never reuse A's social identity or any social identity already attempted for this target. Release each failed social lease after diagnostics. Exhaust available distinct social identities before the platform-alias fallback. If A's proven route was social, start B with the next unused social identity rather than repeating native conditions A already exhausted.
> 3a. After applicable social identities fail or are unavailable, try one final native signup with B's fresh matching platform alias: `<PLATFORM_ALIAS_BASE>+workflow-b-<unique>@<HACKERONE_ALIAS_DOMAIN>` for HackerOne or `<PLATFORM_ALIAS_BASE>+workflow-b-<unique>@<BUGCROWD_ALIAS_DOMAIN>` for Bugcrowd. Use only the matching platform domain. Query forwarded verification mail at `$YOUR_FORWARDED_INBOX`, filtered by expected subject and request timestamp. Use a fresh flow-capable profile at the most promising native geo/provider. Do not restart the five-pass ladder. If A already proved the platform-alias route, use it directly for B after acquiring a distinct profile.
> 4. If mandatory MFA appears, follow the MFA persistence contract in this prompt. Enroll B's own factor, capture B's reusable TOTP seed or recovery material before confirmation, and verify one locally generated TOTP when applicable. Never reuse A's factor. Return reusable material only in the structured verdict for immediate central-writer persistence; do not echo it in prose or write it under `raw/`.
> 5. Provision B's own equivalents for every `b_equivalent_required` object in `raw/resource-graph.json`, following recipe. Append B object IDs and ownership evidence to your own `raw/resource-graph-b.json`. Never share A's object as B's substitute. If a child object cannot be created, record exact gap and continue.
> 6. Dump auth state: `python3 $HELP/dump-auth.py $CDP_B b $YOUR_TARGET_ROOT/raw/auth-b.json $SCOPE_HOSTS_FILE`
>
> **You have no user channel.** Never wait for a human, poll a captcha indefinitely, or fire the notify endpoint. Return blocked only after five useful native conditions, all applicable unused social identities, and the matching platform-alias fallback, or immediately for a truly non-bypassable KYC, mandatory payment, invite-only, explicit scope, or unavailable required-value wall. The orchestrator handles the user.
>
> Return exactly: `{"role":"registrar","status":"ok|blocked","auth_b_ok":bool,"profile_b":int,"profile_b_owner":"exact lease owner","auth_method":"native|google|platform-alias","target_account_state":"new|existing|unknown","capture_mode":"mitm|cdp-only-native-tls","mfa_method":"none|email-otp|sms-otp|totp|push|passkey|webauthn|vendor","mfa_destination":"literal researcher-owned destination, enrolled profile, or not-applicable","totp_secret":"Base32 secret or not-applicable","recovery_codes":["literal unused code",...],"mfa_relogin_verified":bool,"mfa_reusable":bool,"primary_resource_b":"...","blocker":"...","failure_class":"profile-ip|proxy-provider|geo|deterministic|captcha|anti-bot|none","attempts":[{"pass":"1|2|3|4|5|social-pN|platform-alias","profile":N,"owner":"exact lease owner","egress":"us-ws","outcome":"fail","signature":"...","released":true}],"anomalies":["..."],"note":"..."}`

## Step 8: What the walk does (reference)

`$HELP/walk.sh` is the canonical public workflow executable run by walker. This section describes expected outputs; do not copy or rewrite helper source.

| Stage | Does | Writes |
|---|---|---|
| 1 | Builds pre-walk URL union: standard SPA paths, caller extras, SPA/JS route-manifest candidates, browser-safe GET operation candidates, resource graph routes, UI-shaped Phase 0 GETs, and sibling roots. Dedupes while preserving priority. | `raw/prewalk-urls.txt`, `raw/route-manifest-candidates.txt` |
| 2 | Drives list through CDP via `prewalk.py`, 4s per URL, follows newly visible in-scope menu links within hard cap, and records requested URL, final URL, outcome, title, profile, and evidence. mitm captures request cascade. | `raw/prewalk-logs/prewalk-a.log`, `raw/auth-route-coverage.jsonl` |
| 3 | Mines the mitm flow for endpoints and status codes, merges with Phase 0's unauth list, diffs out the auth-only surface, appends `discover` + `probe` events to the catalog, rebuilds the tier-1 host list. | `recon-endpoints-authwalk-a.txt`, `-status.txt`, `recon-endpoints-merged.txt`, `recon-endpoints-auth-only.txt`, `tier1-hosts.txt` |
| 4 | Pulls auth-only JS chunks (diffed against Phase 0's), downloads each plus any source map. | `js-urls-authwalk.txt`, `raw/js/authwalk-*.js[.map]`, `source-maps-authwalk.txt` |
| 5 | Collapses the merged endpoint list to shapes (ids, uuids, hashes, JWTs, tx/eth addresses). | `recon-shapes-merged.txt` |
| 6 | OPTIONS probe over up to 300 unique shape-roots, **10-way parallel**, recording `Allow:` / `Access-Control-Allow-Methods` as `method-probe` events. Serial this would be up to 25 min and the long pole in the fork; parallel it is ~2-3 min. | `options-probe-urls.txt` |
| 7 | Emits every count as JSON so walker verdict is mechanical, not hand-tallied. | `raw/walk-counts.json` |

`recon-endpoints-merged.txt` is the canonical endpoint list Phase 2 reads.

## Step 9: Join, and handle blockers

Collect both verdicts. You cannot observe a subagent mid-flight; you act on what they return.

Merge `raw/resource-graph-b.json` into canonical `raw/resource-graph.json` after registrar returns. Keep one object per `(type, owner, id)`, preserve both ownership evidence paths, and retain every gap. Do not convert a registrar gap into absence of relevance.

Validate and seed the expanded operation ledger after walker output is merged. This fails if authenticated additions omit canonical classes or duplicate an existing `(method, url, operation)` record.

```bash
python3 "$OP_CANDIDATE_HELPER" "$YOUR_TARGET_ROOT" seed || exit 1
$EP rebuild
```

First verify every failed B attempt in the registrar verdict has `released=true`, unless its note explicitly says authenticated or uncertain state required it to remain held. Then rebind B to returned `profile_b`, `profile_b_owner`, `auth_method`, `target_account_state`, `capture_mode`, `mfa_method`, `mfa_destination`, `totp_secret`, `recovery_codes`, `mfa_relogin_verified`, and `mfa_reusable`. A retry may have succeeded on a different slot, or B may have started without a pre-acquired profile when A used Google or a platform alias. Set `PROFILE_B`, `OWNER_B`, `MFA_METHOD_B`, `MFA_DESTINATION_B`, `TOTP_SECRET_B`, and `RECOVERY_CODES_B` to the returned literal values, then recompute `CDP_B` and `MITM_B`. Do not release an already-released initial lease again. All contracts and sensitive entries must name the winning slot, auth method, target-account state, capture mode, and reusable MFA state.

**Re-derive state from disk first.** The subagents ran in their own shells; your variables are stale or unset. Never compute the mode from shell state you did not set yourself.

```bash
R=$YOUR_TARGET_ROOT/raw
AUTH_A_OK=$(jq -r '.authenticated // false' $R/auth-a.json 2>/dev/null || echo false)
AUTH_B_OK=$(jq -r '.authenticated // false' $R/auth-b.json 2>/dev/null || echo false)
AUTH_ONLY_N=$(wc -l < $R/recon-endpoints-auth-only.txt 2>/dev/null || echo 0)
[ -s $R/walk-counts.json ] && cat $R/walk-counts.json
echo "A=$AUTH_A_OK B=$AUTH_B_OK auth_only=$AUTH_ONLY_N"
```

`dump-auth.py` uses credential-name heuristics. If a subagent returned `auth_a_ok=true` or `auth_b_ok=true` and named a concrete working session cookie or storage credential, but the corresponding JSON says `authenticated=false`, correct that JSON to `authenticated=true`, add an `authentication_basis` field naming the credential, then rerun the block above. Do not override based only on a successful-looking page or a CSRF cookie.

If A failed its native and social ladders at Step 4 and no fork happened, these all come back false/0, which is correct: the mode is UNAUTH.

**If the walker returned `thin`:** distinguish the two causes before writing PARTIAL.
- Dead auth (every captured request redirected to `/login`) → check `auth-a.json`, re-login via the recipe, re-dispatch the walker once. If still dead, derive PARTIAL or UNAUTH automatically.
- Un-provisioned surface (few routes existed to walk) → Step 6 did not actually take. Make one bounded provisioning repair and re-dispatch once. If still thin, select PARTIAL automatically.
- Native-TLS social capture (`capture_mode=cdp-only-native-tls`) → keep the authenticated account, do not re-dispatch expecting a `.flow`, and select at most PARTIAL unless independent evidence proves the small app is fully covered.

**Automatic blocker policy:** paid plans, purchases, bookings, deposits, card funding,
subscriptions, KYC, invite-only access, missing signup, pending or unavailable
platform credentials, expired provisioning mail, deterministic identity rejection,
unavailable SMS/MFA, SSO-only access, thin coverage, and exhausted native/social/
platform-alias ladders are not intervention reasons. Record the exact blocker,
preserve any access already obtained, choose PARTIAL or UNAUTH from actual state,
and continue. Registration succeeding while provisioning silently fails is still
recorded as PARTIAL, not RICH.

**Only CAPTCHA intervention:** fire the notify and ask if a registrar or the
orchestrator records `failure_class=captcha` or `failure_class=anti-bot`, the
challenge remains visible in a held profile, and a human solving it is the only
remaining action that could plausibly change the result. Do not fire for a generic
`blocked` verdict without that evidence.

Then send ONE message and wait only for that actionable CAPTCHA.

```
ACCESS CAPTCHA BLOCKED. Human challenge remains actionable.

Account: {A or B} on pN
Challenge: {captcha or anti-bot vendor, version, iframe host, visible state}
Automated attempts: {<YOUR_CAPTCHA_SOLVER> extension, supported token fallback, and useful profile conditions exhausted}
Failure class: {captcha | anti-bot}

Open your browser handoff interface for profile pN and solve the visible
challenge. Reply `done` so I can capture and re-derive access. Reply `skip` if the
challenge cannot be solved; I will automatically finish PARTIAL or UNAUTH.

Do not expose a shared remote-desktop port. Do not retry or poll while waiting.
```

On reply: `done` → re-run `dump-auth.py` on the affected profile, then rerun the Step 9 state-derivation block so `AUTH_A_OK`, `AUTH_B_OK`, and `AUTH_ONLY_N` are current. `skip` → automatically choose PARTIAL or UNAUTH from actual state and continue to Step 10. Do not accept unrelated missing values through this intervention path.

**Mode is computed from actual outcomes, not from the reply text.**

## Step 10: ACCESS MODE, shared files, contract

```bash
ACCESS_MODE=UNAUTH
if [ "$AUTH_A_OK" = "true" ] || [ "$AUTH_B_OK" = "true" ]; then
  if [ "${AUTH_ONLY_N:-0}" -ge 20 ] && [ "$AUTH_A_OK" = "true" ] && [ "$AUTH_B_OK" = "true" ] \
     && [ "${WINNING_CAPTURE_MODE_A:-mitm}" = "mitm" ]; then
    ACCESS_MODE=RICH
  else
    ACCESS_MODE=PARTIAL
  fi
fi
echo "ACCESS_MODE=$ACCESS_MODE  (A=$AUTH_A_OK B=$AUTH_B_OK auth_only=${AUTH_ONLY_N:-0})"
```

Override only with a stated reason. A genuinely small app can be RICH under 20 auth-only endpoints; a large one at 15 is a thin capture.

**You write all shared files. The subagents wrote none of these.**

### 10a. SENSITIVE-FINDINGS.md

Entry format is specified in `$YOUR_HELPERS_ROOT/report-format.md` § 2; the test-account shape below matches it. Append, never overwrite. Substitute literal values: a saved file still containing `{PROFILE_A}` is a bug, because the next phase reads it expecting a number.

```markdown
### [PHASE-01] Test Account A
- **Type**: Test Account + Auth Token
- **Name**: {A_FIRST} {A_LAST}
- **Email**: {ACCT_A_EMAIL}
- **Password**: {PASSWORD}
- **MFA method**: {MFA_METHOD_A}
- **MFA destination / enrolled factor**: {MFA_DESTINATION_A}
- **TOTP secret**: {TOTP_SECRET_A or not-applicable}
- **Recovery codes**: {all literal unused codes, comma-separated / none-issued / not-applicable}
- **MFA re-login verified**: {yes / no; if no, exact limitation}
- **Login URL**: {from auth-a.json post_login_url}
- **Registration URL**: {REG_URL}
- **Auth token / cookie**: {e.g. "session_id cookie + JWT in localStorage 'auth_token'"}
- **Token location**: cookie | Authorization header | localStorage key | etc.
- **Browser profile**: p{PROFILE_A} (CDP $YOUR_CDP_HOST:{CDP_A}, capture proxy $YOUR_MITM_HOST:{MITM_A})
- **Primary resource**: {PRIMARY_RESOURCE_A}
- **Service**: $TARGET
- **Status**: CONFIRMED VALID
- **Access**: Standard user
- **Notes**: Profile held; do not release until the close-out phase.

### [PHASE-01] Test Account B
{same shape, B values, PRIMARY_RESOURCE_B from the registrar verdict}
```

### 10b. accounts.md

Append confirmed rows only to `$YOUR_ACCOUNTS_FILE`:

```
| {target} | {REG_URL} | {login URL} | {ACCT_A_EMAIL} | {PASSWORD} | {ids + name; MFA method; TOTP secret and unused recovery codes when applicable; owned email/SMS destination or enrolled factor} | p{PROFILE_A} | {standard role; MFA reusable / MFA session-only} |
| {target} | {REG_URL} | {login URL} | {ACCT_B_EMAIL} | {PASSWORD} | {ids + name; MFA method; TOTP secret and unused recovery codes when applicable; owned email/SMS destination or enrolled factor} | p{PROFILE_B} | {standard role; MFA reusable / MFA session-only} |
```

`Browser Profile` is `-` if no profile is held.

### 10c. hunt/phase-01-access.md

Lowercase `{placeholders}` below are fields from the **walker's verdict JSON** (mirrored in `raw/walk-counts.json`). Uppercase `{PROFILE_A}` style are shell values you set yourself. Substitute literals for both; leave no placeholder text in the saved file.

```markdown
# Phase 1: Access, $TARGET
Date: {date}

## ACCESS MODE
{RICH | PARTIAL | UNAUTH}

Phase 2 routes on this value. None of the three stop the target.

## Accounts

| Label | Name | Email | Password | Profile | CDP | mitm | Auth Token Location | Primary resource |
|-------|------|-------|----------|---------|-----|------|---------------------|------------------|
| A | {A_FIRST} {A_LAST} | {ACCT_A_EMAIL} | {PASSWORD} | p{PROFILE_A} | $YOUR_CDP_HOST:{CDP_A} | $YOUR_MITM_HOST:{MITM_A} | {cookie/Bearer/localStorage} | {PRIMARY_RESOURCE_A} |
| B | {B_FIRST} {B_LAST} | {ACCT_B_EMAIL} | {PASSWORD} | p{PROFILE_B} | $YOUR_CDP_HOST:{CDP_B} | $YOUR_MITM_HOST:{MITM_B} | {same} | {PRIMARY_RESOURCE_B} |

Both accounts should have their own equivalents for each cross-tenant-relevant object type in resource graph. If one is missing, name exact object gap. Relevant Phase 2 work is PARTIAL until prerequisite exists, not DEAD or N/A.

## MFA access

| Label | Method | Destination or factor | TOTP secret | Recovery codes | Re-login verified | Reusable |
|-------|--------|-----------------------|-------------|----------------|-------------------|----------|
| A | {MFA_METHOD_A} | {MFA_DESTINATION_A} | {TOTP_SECRET_A / not-applicable} | {all unused codes / none-issued / not-applicable} | {yes/no} | {yes/no; session-only reason} |
| B | {MFA_METHOD_B} | {MFA_DESTINATION_B} | {TOTP_SECRET_B / not-applicable} | {all unused codes / none-issued / not-applicable} | {yes/no} | {yes/no; session-only reason} |

Never copy A's factor into B's row. Never write a transient one-time code here.

## Recipe
`hunt/account-creation-process.md` holds the working page sequence, captcha and verification path, winning geo/profile, and the approaches that failed.

## Auth flow observed
- Registration URL: {REG_URL}
- Login URL / API endpoint: {observed from the registration/login mitm flow}
- Auth mechanism: form / OAuth / wallet / hybrid
- Token type: JWT (HS256) / opaque session cookie
- Token rotation: every request / on expiry / fixed
- Refresh endpoint: POST {url}
- Cross-subdomain cookies: {from auth-a.json}

## Walk coverage
- Pre-walk URLs visited: {prewalk_n}
- Profile walked: A (p{PROFILE_A}). B untouched; its auth state is saved for Phase 2.
- Authenticated route ledger: `raw/auth-route-coverage.jsonl` ({n navigated, n redirected, n error, n not visited at cap})
- Route sources used: standard SPA, visible menu, route manifest/static JS, operation candidates, resource graph, mitm

| Source | Count |
|---|---|
| Phase 0 unauth shape | {unauth_n} |
| Authwalk | {authwalk_n} |
| Merged (canonical) | {merged_n} |
| Auth-only (diff) | {auth_only_n} |
| Auth-only JS chunks | {new_js_n} |
| Auth-only source maps | {new_maps_n} |

## Top auth-only endpoints (Tier 1 for Phase 2)
- {METHOD URL}; likely {GET-list / POST-create / PATCH-update}

## Resource graph
- A-owned object types: {list with IDs and ownership evidence paths}
- B-owned equivalents: {list with IDs and ownership evidence paths}
- Missing child state: {list with exact prerequisite; relevant Phase 2 rows must be PARTIAL}
- Canonical files: `raw/resource-graph.json`, `raw/resource-graph-b.json`

## Registration diagnostics

| Account | Pass | Profile | Egress | Outcome | Released | Failure signature |
|---------|------|---------|--------|---------|----------|-------------------|
| A | 1 | pN | us-ws | {result} | {yes/no + reason} | {exact} |
| B | 1 | pN | us-ws | {result} | {yes/no + reason from registrar verdict} | {from registrar verdict} |

- Likely failure class: profile-ip | proxy-provider | geo | deterministic | none
- Working configuration: {geo, egress, anti-bot setting}

## Blockers
- Captcha: {provider}, solved via <YOUR_CAPTCHA_SOLVER> / user / not solved
- Phone verification: {not required / number used / all rejected + reason}
- KYC: {not encountered / REQUIRED}
- Paid plan: {not required / REQUIRED for provisioning}

## Anomalies observed (flagged, not pursued)
{merged from both subagent verdicts; for each possible vulnerability hypothesis, give a local technique-ref or historical-example citation, retrieval status, and first untested Phase 2 angle. Never include credentials, tokens, private object IDs, or response bodies.}

## Files generated
- `raw/auth-a.json`, `raw/auth-b.json`
- `raw/prewalk-urls.txt`, `raw/prewalk-logs/prewalk-a.log`
- `raw/route-manifest-candidates.txt`, `raw/auth-route-coverage.jsonl`
- `raw/recon-endpoints-authwalk-a.txt`, `raw/recon-endpoints-authwalk-status.txt`
- `raw/recon-endpoints-merged.txt` (canonical input for Phase 2)
- `raw/recon-endpoints-auth-only.txt`, `raw/recon-shapes-merged.txt`, `raw/tier1-hosts.txt`
- `raw/js-urls-authwalk.txt`, `raw/js/authwalk-{hash}.js[.map]`, `raw/source-maps-authwalk.txt`
- `raw/options-probe-urls.txt`, `raw/walk-counts.json`
- `raw/operation-candidates.jsonl` (Phase 0 union plus authenticated candidates)
- `raw/resource-graph.json`, `raw/resource-graph-b.json`
- `raw/phase01-helper-sha256.txt` (exact canonical helper revisions used)
- `hunt/account-creation-process.md`
```

**If ACCESS MODE is UNAUTH**, still write the contract, and still produce the canonical inputs Phase 2 expects:

```bash
if [ "$ACCESS_MODE" = "UNAUTH" ]; then
  cp $YOUR_TARGET_ROOT/raw/recon-endpoints.txt \
     $YOUR_TARGET_ROOT/raw/recon-endpoints-merged.txt 2>/dev/null
  : > $YOUR_TARGET_ROOT/raw/recon-endpoints-auth-only.txt
  awk '$NF ~ /^https?:\/\// {print $NF}' $YOUR_TARGET_ROOT/raw/recon-endpoints-merged.txt \
    | awk -F/ '{print $1"//"$3}' | sort -u | head -20 \
    > $YOUR_TARGET_ROOT/raw/tier1-hosts.txt 2>/dev/null
fi

# Stamp final access mode into canonical resource graph, then reject placeholder
# objects, missing category assessments, unproved usable state, and silent B gaps.
[ -s "$YOUR_TARGET_ROOT/raw/resource-graph.json" ] \
  || { echo "ERROR: missing canonical resource graph"; exit 1; }
RESOURCE_GRAPH_TMP=$(mktemp "$YOUR_TARGET_ROOT/raw/resource-graph.json.XXXXXX")
jq --arg mode "$ACCESS_MODE" '.schema_version=1 | .access_mode=$mode' \
  "$YOUR_TARGET_ROOT/raw/resource-graph.json" > "$RESOURCE_GRAPH_TMP" \
  && mv "$RESOURCE_GRAPH_TMP" "$YOUR_TARGET_ROOT/raw/resource-graph.json"
python3 "$RESOURCE_GRAPH_VALIDATOR" "$YOUR_TARGET_ROOT" || exit 1

# Refresh the shared attacker model from actual access, then append any
# trust-boundary correction with `threat-model.py amend --change <JSON>`.
python3 "$THREAT_MODEL_HELPER" "$YOUR_TARGET_ROOT" derive \
  --target-name '{{target_norm}}' --scope '{{scope}}' \
  --access-mode "$ACCESS_MODE" --phase phase-01-access
python3 "$THREAT_MODEL_HELPER" "$YOUR_TARGET_ROOT" validate || exit 1
```

Before Phase 2 handoff, read `raw/threat-model.json`. Derive again with
`--identity '<A or B: actual role and tenant relation>'` for each usable
account, or omit identities when none exists. Append an amendment with
`threat-model.py amend --change '{"text":"<concrete boundary correction>","evidence":["raw/<evidence>"]}'`
for each role or ownership boundary established by the access walk. Correct
Phase 0 inferences that changed. Keep credentials, tokens, and cookies in
`sensitive/`.

## Don't release successful profiles

Phase 2 inherits logged-in state. Winning authenticated profiles stay running and leased until the close-out phase. Tmux session end does not release browser services. A failed replacement profile may be released once its diagnostics are saved and you confirm it holds no authenticated session. Release only profiles leased by this phase, always with `--if-owner`.

## Anomalies to flag, not pursue

- Registration response includes an auth token directly (no email gate)
- API silently accepts extra fields (`role: "admin"`, `verified: true`, `kyc_level: 3`)
- Verification token guessable (sequential, short, MD5-of-email)
- Login response returns internal fields (password hash, MFA secret)
- Welcome email leaks internal info
- Admin-only or `/internal/` endpoint reachable from a regular-user UI
- A response showing PII for users other than A/B
- "Generate API key" exposing a full key in the response body

Log to `SENSITIVE-FINDINGS.md` using the entry format in `$YOUR_HELPERS_ROOT/report-format.md` § 2, and note it in the contract. Do NOT pursue. Phase 2 owns all of it.

---

## What NOT to do
- Do NOT test for vulnerabilities. Log signals; Phase 2 pursues them.
- Do NOT stop the target because account setup failed. UNAUTH is a productive mode.
- Do NOT dispatch the subagents if A exhausted five native profiles, all applicable social identities, and the matching platform-alias fallback. Notify first.
- Do NOT let a subagent talk to the user, wait on a human, or write a shared file.
- Do NOT report a usable state you did not reach. An account at an onboarding wall is PARTIAL, not RICH.
- Do NOT give B a share of A's resource. It needs its own or cross-tenant testing is impossible.
- Do NOT use real personal email or phone numbers.
- Do NOT use a Bugcrowd or HackerOne alias before the configured catch-all domains and applicable social identities are exhausted. The final alias must match the current platform, be a fresh plus alias, and be used for only one required account attempt.
- Do NOT register on hosts outside canonical `raw/scope-hosts.txt`, or through phone Chrome. Evidence-backed first-party functional siblings belong in that file before use.
- Do NOT KYC test accounts with real documents.
- Do NOT log credentials anywhere except the three saved locations.
- Do NOT submit forms, click destructive buttons, or log out during the walk.
- Do NOT connect to VNC. Ever.
- Do NOT silently finalise a thin contract.
- Do NOT let a provisioning wall pass without recording it and reducing the access mode. Registration success does not mean the product is usable.

---

## Phase complete: stamp finish time

```bash
$EP rebuild
$EP stats

$YOUR_HUNT_BIN/hunt-phase-event finish \
  '{{platform}}' '{{handle}}' '{{target_norm}}' phase-01-access true
```
