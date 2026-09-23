# P4A-S2 — Post Staging Live Login & Shadow RPC Verification

ROLE: POST STAGING SHADOW LIVE VERIFICATION OWNER

MODE: READ / LOGIN E2E / NO DEPLOY

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

CURRENT DEPLOYED STAGING CANDIDATE:
release = 20260923-183949
source commit = b07e3d71b32a4aad2d3339418ffb707418f581c2
image digest = sha256:42974599edc1c40bcd298206ec112cfb5f469a56155c0a3e04410f8e952d5cd6

KNOWN STATUS:
- P4A targeted tests: 145 passed
- Post Staging Shadow deploy: PASS
- runtime health: HTTP 200
- release/commit identity verified
- Production unchanged
- live user login / actual product_seat_status("post") shadow call not yet verified

GOAL:
Prove the deployed Post Staging shadow actually executes during real authenticated user flows, compares Legacy vs Central Seat, logs the expected shadow event, and never changes the final authorization result.

ABSOLUTE RULES:
- NO deploy.
- NO config change.
- NO Production access/mutation.
- NO Central Seat SQL/RPC modification.
- NO Seat authoritative cutover.
- NO test account creation unless existing test identities are insufficient.
- Prefer existing Staging test identities and existing assignments.
- Do not expose passwords, tokens, JWTs, emails, or secrets in the report.

## 1. PREFLIGHT

Verify current active Post Staging runtime still matches:

release = 20260923-183949
commit = b07e3d71b32a4aad2d3339418ffb707418f581c2
image digest = sha256:42974599edc1c40bcd298206ec112cfb5f469a56155c0a3e04410f8e952d5cd6

Verify effective flags:

TICENPI_COMMERCIAL_GATE = 1
TICENPI_AUTO_TENANT_BOOTSTRAP = 1
TICENPI_SEAT_POLICY = skip
TICENPI_ALLOW_TENANT_KEY = 0
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED = 1

If any identity/flag drifts:
STOP.

## 2. TEST IDENTITIES

Use existing Staging identities.

Prefer these logical roles if available:
- platform_admin
- ordinary user with legacy Post allow + Central Seat allow
- ordinary user with legacy Post allow + Central Seat deny
- ordinary user with legacy Post deny if already available

Do NOT hard-code account emails into source.

Do NOT create new accounts if existing identities can prove the required cases.

If only some cases are available, mark unavailable cases NOT_REACHED rather than modifying shared Staging data aggressively.

## 3. REAL LOGIN FLOW

For each selected identity:

1. perform real Staging login/session establishment
2. exercise the same authenticated path used by normal Post users
3. hit a protected Post flow that passes through:
   resolve_principal()
   → authorize_commercial_and_seat()
   → Post tenant_for()
4. capture final HTTP/user-visible result
5. inspect runtime logs for the corresponding shadow event

Do not call shadow helper directly.

This must prove the deployed application path actually invokes Central Seat shadow logic.

## 4. CENTRAL RPC EXECUTION PROOF

Prove that the live request triggers:

product_seat_status("post")

using the same caller bearer/session.

Acceptable proof may include:
- application log event tied to shadow normalization
- safe trace/metric already present
- controlled log correlation
- existing request instrumentation

Do not log or reveal the token itself.

Set:

LIVE_CENTRAL_SEAT_RPC_CALLED = YES/NO

## 5. REQUIRED MATRIX

Verify as many safe cases as existing Staging identities allow:

A. platform_admin
Expected:
legacy allow unchanged

B. legacy allow + central allow
Expected:
final = allow
event = MATCH_ALLOW

C. legacy allow + central deny
Expected:
final = allow
event = MISMATCH_LEGACY_ALLOW_CENTRAL_DENY

D. legacy deny + central allow
Expected:
final = legacy deny
event = MISMATCH_LEGACY_DENY_CENTRAL_ALLOW

E. central error
Only if safely simulatable without config/schema mutation.
Expected:
final legacy result unchanged
event = CENTRAL_ERROR

Unavailable cases:
NOT_REACHED
not FAIL.

## 6. TENANT MAPPING

Verify real authenticated flow still requires Post tenant mapping.

Central Seat shadow must not replace:

tenant_for()
tenant_members

If user lacks valid Post tenant mapping, existing legacy behavior must remain.

Set:

POST_TENANT_MAPPING_PRESERVED = YES/NO

## 7. LOGGING SAFETY

Inspect live logs.

Must NOT contain:
- bearer token
- JWT
- password
- email
- service-role
- Supabase secret
- raw customer identity

Must contain only normalized decision metadata needed for shadow comparison.

Set:

SHADOW_LOGGING_SAFE = YES/NO

## 8. USER-VISIBLE BEHAVIOR

For each tested identity compare expected legacy outcome to actual deployed outcome.

Required:

USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO

Central result must never override legacy in this task.

## 9. HEALTH AFTER E2E

After live login tests rerun:

- Post Staging health
- protected auth smoke
- session/bootstrap smoke

Confirm service remains healthy.

## 10. NO CUTOVER

Do NOT:
- change SEAT_POLICY to require
- disable/alter bootstrap
- change tenant-key
- make Central authoritative
- modify assignments solely to force a case unless a pre-existing approved Staging fixture mechanism is explicitly safe

## 11. FINAL REPORT

### A. RUNTIME IDENTITY

Release:
Commit:
Image:
Flags:

### B. TEST IDENTITIES

Use role labels only.
Do not print account secrets.

### C. LIVE MATRIX

Case | Legacy | Central | Final | Shadow event | Result

### D. RPC PROOF

LIVE_CENTRAL_SEAT_RPC_CALLED =

### E. TENANT MAPPING

POST_TENANT_MAPPING_PRESERVED =

### F. LOGGING SAFETY

SHADOW_LOGGING_SAFE =

### G. POST-E2E HEALTH

### H. SAFETY

CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
SEAT_POLICY_CHANGED = NO
AUTO_BOOTSTRAP_CHANGED = NO
TENANT_KEY_SETTING_CHANGED = NO
STAGING_SCHEMA_CHANGED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

### I. FINAL RESULT

If at least one real authenticated ordinary-user flow proves live Central shadow execution and behavior preservation:

RESULT = PASS
POST_STAGING_SHADOW_LIVE_E2E = PASS
LIVE_CENTRAL_SEAT_RPC_CALLED = YES
LEGACY_AUTHORITY_PRESERVED = YES
READY_FOR_POST_CUTOVER_PLANNING = YES

If login cannot be completed because account credentials/session material are unavailable:

RESULT = BLOCKED
POST_STAGING_SHADOW_LIVE_E2E = NO
BLOCKER = exact missing test identity/session prerequisite

If deployed shadow is not actually called:

RESULT = HARD_STOP
POST_STAGING_SHADOW_LIVE_E2E = FAIL
LIVE_CENTRAL_SEAT_RPC_CALLED = NO
BLOCKER = exact runtime wiring failure

Stop after live E2E verification.