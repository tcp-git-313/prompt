# P4A-S2 — Post Live Staging Shadow Auth E2E

ROLE: POST LIVE STAGING SHADOW VERIFICATION OWNER

MODE: LIVE STAGING E2E / READ-MOSTLY / NO CUTOVER / NO DEPLOY

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Verify that the already-deployed Post Central Seat shadow actually executes on real authenticated Staging requests, compares legacy authorization with Central product_seat_status("post"), logs the expected observation, and never changes user-visible authorization.

KNOWN DEPLOYED STAGING RELEASE:

Release:
20260923-183949

Source commit:
b07e3d71b32a4aad2d3339418ffb707418f581c2

Image digest:
sha256:42974599edc1c40bcd298206ec112cfb5f469a56155c0a3e04410f8e952d5cd6

Prior deployment result:
P4A_READY_BY_EVIDENCE = YES
POST_STAGING_SHADOW_DEPLOYED = YES
runtime health = PASS
Production = unchanged

IMPORTANT:
The prior task did NOT execute user login/auth flow, so live shadow RPC execution is not yet proven.

ABSOLUTE RULES:
- Do NOT deploy.
- Do NOT rebuild.
- Do NOT change code.
- Do NOT change runtime flags.
- Do NOT change SEAT_POLICY.
- Do NOT change AUTO_BOOTSTRAP.
- Do NOT change ALLOW_TENANT_KEY.
- Do NOT make Central Seat authoritative.
- Do NOT touch Production.
- Do NOT modify Central Seat SQL/RPC.
- Do NOT create permanent Staging users.
- Do NOT print passwords, bearer tokens, refresh tokens, JWTs, Supabase keys, or cookies.
- Do NOT use service-role in the browser/user flow.

## 1. VERIFY EXACT ACTIVE RELEASE

Read live Post Staging identity and prove:

ACTIVE_RELEASE = 20260923-183949

ACTIVE_COMMIT = b07e3d71b32a4aad2d3339418ffb707418f581c2

ACTIVE_IMAGE_DIGEST = sha256:42974599edc1c40bcd298206ec112cfb5f469a56155c0a3e04410f8e952d5cd6

If any mismatch:
STOP.

## 2. VERIFY EFFECTIVE RUNTIME FLAGS

Read actual running container values.

Required:

TICENPI_COMMERCIAL_GATE = 1
TICENPI_AUTO_TENANT_BOOTSTRAP = 1
TICENPI_SEAT_POLICY = skip
TICENPI_ALLOW_TENANT_KEY = 0
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED = 1

If shadow enabled is not 1:
STOP and report exact runtime evidence.

Do not change it here.

## 3. DISCOVER APPROVED STAGING TEST IDENTITIES

Use existing fixed Staging test identities and existing secure credential sources only.

Do NOT create or reset accounts unless an existing approved Staging test workflow explicitly requires temporary fixture creation and guaranteed cleanup.

Identify available roles/cases such as:

- platform_admin
- ordinary user with valid Post legacy access
- ordinary user with no Central Post Seat
- test_access allow / deny if existing
- assigned vs unassigned business user if existing

Never print credentials.

Report only:
TEST_IDENTITY_LABEL
ROLE/CASE
EXPECTED_LEGACY_RESULT
EXPECTED_CENTRAL_RESULT if known

If only one safe authenticated identity is available, still verify actual shadow invocation with that identity and mark unavailable mismatch cases NOT_REACHED.

## 4. ESTABLISH LOG BASELINE

Before sending auth requests:

- identify the exact Post Staging API container/log stream
- record a timestamp/marker window
- capture baseline count of Central Seat shadow events if practical

Do not clear logs.

## 5. AUTHENTICATED LIVE REQUEST

Using the approved Staging login path:

- authenticate as the selected test identity
- perform a normal Post request that definitely traverses:

resolve_principal()
→ authorize_commercial_and_seat()
→ Post tenant_for()
→ protected/session/bootstrap path

Preferred:
existing session/bootstrap or protected Web API smoke path already used by Post.

Do not use X-Tenant-Key.

The request must use a real user bearer/session from Staging auth.

Record:
HTTP result
legacy expected result
whether Post tenant mapping was exercised

Do not expose token content.

## 6. PROVE SHADOW RPC EXECUTION

Using logs/observability from the same request window, prove that:

product = post

and one of the expected shadow events occurred:

MATCH_ALLOW
MATCH_DENY
MISMATCH_LEGACY_ALLOW_CENTRAL_DENY
MISMATCH_LEGACY_DENY_CENTRAL_ALLOW
CENTRAL_ERROR
CENTRAL_UNKNOWN

The evidence must correspond to the authenticated request.

If the code logs only mismatches/errors and the request is a match with no log by design:
use another safe verification method already present in the code/test instrumentation, or select an existing identity expected to produce a mismatch.

Do not modify code merely to create a log.

If actual shadow invocation cannot be proven:
RESULT must not be PASS.

## 7. PREFERRED LIVE MATRIX

Attempt only cases safely available from existing Staging test identities/data.

Case A:
platform_admin
Expected:
legacy ALLOW
user-visible result unchanged

Case B:
ordinary valid Post user with Central ALLOW
Expected:
legacy ALLOW
central ALLOW
final ALLOW

Case C:
ordinary legacy-allowed Post user with no assigned Central Post Seat
Expected:
legacy ALLOW
central DENY
final ALLOW
event:
MISMATCH_LEGACY_ALLOW_CENTRAL_DENY

This is the most useful Shadow proof if safely available.

Case D:
legacy DENY + Central ALLOW
Only if an existing safe fixture exists.
Do not create risky state solely to force it.

Unavailable cases:
NOT_REACHED

NOT_REACHED is acceptable.
It is not FAIL.

## 8. USER-VISIBLE AUTHORITY PROOF

For every reached mismatch case:

prove final HTTP behavior still follows legacy.

Required invariant:

legacy ALLOW + central DENY
→ request remains allowed

legacy DENY + central ALLOW
→ request remains denied by existing legacy behavior

Central ERROR/UNKNOWN
→ request outcome unchanged

Set:

LEGACY_AUTHORITY_LIVE_PROVEN = YES/NO

## 9. TENANT MAPPING PROOF

Confirm live authenticated path still requires Post tenant mapping.

Central Seat shadow must not replace:

Post tenant_for()/tenant_members

Do not modify mappings.

Set:

POST_TENANT_MAPPING_PRESERVED = YES/NO

## 10. LOGGING SAFETY

Inspect the relevant live shadow events.

Confirm no log contains:

- Authorization header
- bearer token
- JWT
- refresh token
- email
- password
- service-role
- Supabase key
- cookie
- raw customer identity

Set:

SHADOW_LOG_SECRET_LEAK = NO/YES

If YES:
HARD STOP.

## 11. HEALTH AFTER TEST

After E2E:

- /api/health = 200
- release identity unchanged
- commit identity unchanged
- no restart/crash loop
- external Staging endpoint healthy

No rollback is needed if verification fails but service remains healthy, because this task makes no change.

## 12. NO DATA MUTATION

Prefer existing accounts/data.

If an approved temporary Seat assignment/release is absolutely necessary to produce a mismatch case:
- use Staging only
- use synthetic/test identity only
- record before state
- restore exact prior state in finally
- prove cleanup

Do NOT mutate real customer data.

If safe existing data cannot prove mismatch:
mark NOT_REACHED instead.

## 13. FINAL REPORT

### A. ACTIVE RELEASE

ACTIVE_RELEASE =
ACTIVE_COMMIT =
ACTIVE_IMAGE_DIGEST =
IDENTITY_MATCH = YES/NO

### B. RUNTIME FLAGS

COMMERCIAL_GATE =
AUTO_BOOTSTRAP =
SEAT_POLICY =
ALLOW_TENANT_KEY =
CENTRAL_SEAT_SHADOW_ENABLED =

### C. TEST IDENTITIES

Table:
Label | Case | Legacy expected | Central expected | Used

No credentials.

### D. LIVE AUTH MATRIX

Table:
Case | Request path | HTTP result | Legacy | Central | Shadow event | Final authority | PASS/NOT_REACHED

### E. SHADOW EXECUTION PROOF

PRODUCT_SEAT_STATUS_POST_LIVE_CALLED = YES/NO
LIVE_SHADOW_EVENT_OBSERVED = YES/NO
LEGACY_AUTHORITY_LIVE_PROVEN = YES/NO

### F. TENANT MAPPING

POST_TENANT_MAPPING_PRESERVED = YES/NO

### G. LOGGING SAFETY

SHADOW_LOG_SECRET_LEAK = NO/YES

### H. POST-TEST HEALTH

HEALTH_200 =
RELEASE_UNCHANGED =
COMMIT_UNCHANGED =
CRASH_LOOP = NO/YES

### I. SAFETY

CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
SEAT_POLICY_CHANGED = NO
AUTO_BOOTSTRAP_CHANGED = NO
TENANT_KEY_SETTING_CHANGED = NO
STAGING_CODE_CHANGED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

### J. FINAL RESULT

If actual authenticated live shadow execution is proven:

RESULT = PASS
POST_STAGING_SHADOW_DEPLOYED = YES
POST_STAGING_SHADOW_VERIFIED = YES
PRODUCT_SEAT_STATUS_POST_LIVE_CALLED = YES
LEGACY_AUTHORITY_LIVE_PROVEN = YES
READY_FOR_POST_CUTOVER_PLANNING = YES

If service is deployed but shadow invocation cannot be proven:

RESULT = PARTIAL_PASS
POST_STAGING_SHADOW_DEPLOYED = YES
POST_STAGING_SHADOW_VERIFIED = NO
BLOCKER = exact missing live evidence

If secret leakage or user-visible auth changed:

RESULT = HARD_STOP
POST_STAGING_SHADOW_VERIFIED = NO
BLOCKER = exact safety violation

Stop after live auth E2E verification.
