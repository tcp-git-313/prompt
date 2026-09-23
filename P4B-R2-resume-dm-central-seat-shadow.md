# P4B-R2 — Resume DM Central Seat Shadow Integration

ROLE: DM CENTRAL SEAT SHADOW IMPLEMENTATION OWNER

MODE: IMPLEMENTATION / LOCAL ONLY / NON-AUTHORITATIVE

REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Resume P4B using the collision-safe strategy proven by P4B-R1:

COLLISION_CLASS = SAFE_WITH_NEW_HELPER

Implement Central Seat shadow comparison for DM without changing current user-visible authorization behavior and without modifying unrelated YUCT/OAuth dirty work.

## VERIFIED RECONCILIATION

Current branch baseline:
dm/integration-20260920

Baseline HEAD:
561bd8c76fec5c0fb86cb9cf4f56734d28367b8a

Current auth path:
get_current_user
→ require_dm_access()
→ my_commercial_context()
→ has_dm_access()
→ EntitledUserDep-protected routes

Pre-existing dirty behavior:
- local/dev bypass exists
- local/dev bypass may add platform_admin only under explicit local flag + loopback conditions
- this dirty auth work belongs to YUCT extraction-shadow support
- local/dev bypass must remain exactly unchanged
- local/dev bypass must skip Central Seat shadow entirely because it has no real caller bearer/session

Recommended strategy:
SAFE_WITH_NEW_HELPER

## ABSOLUTE BOUNDARIES

ALLOWED FILES:

1.
src/backend/app/auth/seat_shadow.py
NEW FILE

2.
src/backend/app/auth/entitlement.py
ONLY the minimal call-site integration inside require_dm_access()

3.
tests/auth/test_seat_shadow.py
NEW FILE

4.
tests/auth/test_entitlement.py
ONLY if minimal extension is required

DO NOT TOUCH:

src/backend/app/auth/settings.py
src/backend/app/auth/platform_admin.py
tests/test_extraction_shadow.py
YUCT extraction-shadow files
OAuth/UAT files
routers
DM RLS migrations
Staging config
Production config
ticenpi-platform
Post
Launcher

Do NOT reset, restore, stash, clean, rebase, or overwrite dirty work.

Do NOT commit, push, or deploy unless explicitly instructed.

## 1. PREFLIGHT

Record:

branch
HEAD
git status --short
git diff --stat

Re-read current working-tree versions of:

src/backend/app/auth/entitlement.py
src/backend/app/auth/settings.py

Confirm P4B-R1 assumptions are still true.

If require_dm_access() changed materially since P4B-R1 and the approved insertion point is no longer valid:
STOP.

Do not guess.

## 2. SHADOW HELPER

Create:

src/backend/app/auth/seat_shadow.py

The helper must be isolated from legacy decision logic.

Its job:

- call Central product_seat_status("dm")
- use the same authenticated caller bearer/session context already available to require_dm_access()
- normalize Central result
- compare against the already-computed legacy decision
- log mismatch/error safely
- return observation only

It must NOT:

- decide final HTTP allow/deny
- raise access-denied exceptions merely because Central says deny
- alter existing legacy response behavior
- modify RLS
- modify router dependencies
- use service-role
- accept caller-supplied user_id as authority

## 3. CENTRAL CALL CONTRACT

Use:

product_seat_status("dm")

Identity must derive from the same authenticated bearer/session.

Do not pass a separate user id to Central Seat.

Reuse existing safe Supabase HTTP/RPC conventions from DM where practical.

If an existing shared RPC client/helper can be reused without modifying unrelated files, reuse it.

Do not duplicate secrets/config unnecessarily.

## 4. SHADOW NORMALIZATION

Normalize Central response into exactly:

ALLOW
DENY
ERROR
UNKNOWN

Examples:

Central assigned / not_required / exempt / equivalent canonical allowed state
→ ALLOW

Central not_assigned / inactive / denied equivalent
→ DENY

HTTP failure / timeout / malformed response / unavailable Central service
→ ERROR

Unrecognized valid response shape/status
→ UNKNOWN

Do not redesign Central Seat semantics.

If actual canonical response names differ, map them based on existing Central Seat contract and tests.

## 5. LEGACY DECISION MUST REMAIN AUTHORITATIVE

Order inside require_dm_access():

A. Existing local/dev bypass
→ preserve exactly
→ return before any shadow call

B. Existing authenticated commercial context retrieval
→ unchanged

C. Existing has_dm_access() legacy decision
→ compute/store legacy result

D. Invoke new Seat shadow helper using same caller auth context

E. Safely log comparison

F. Return/raise according to the ORIGINAL legacy result only

Required invariant:

CENTRAL_AUTHORITATIVE = NO

Examples:

legacy=ALLOW, central=DENY
→ current request still ALLOW
→ mismatch logged

legacy=DENY, central=ALLOW
→ current request still DENY with existing 403 behavior
→ mismatch logged

central=ERROR
→ legacy result unchanged

central=UNKNOWN
→ legacy result unchanged

## 6. PRESERVE CURRENT BYPASSES

Must preserve:

LOCAL DEV BYPASS
- no real bearer
- skips Central Seat shadow entirely
- existing dev-user behavior unchanged
- existing local platform_admin loopback behavior unchanged

PLATFORM ADMIN
- real authenticated platform_admin keeps current legacy allow behavior
- Central shadow may run and compare
- mismatch is observational only

STAGING TEST ACCESS ALLOW
- legacy allow unchanged
- shadow may run

STAGING TEST ACCESS DENY
- legacy deny + existing 403 unchanged
- shadow may run

Do NOT reorder legacy precedence.

## 7. ERROR SEMANTICS

Do not "fix" the current legacy error semantics in this task.

Existing legacy behavior remains whatever current code does.

Central Seat shadow RPC failure:

- must NOT become 401
- must NOT become 403
- must NOT become 503
- must NOT alter the request outcome

It is only:

shadow_state = ERROR

and an observability event.

HTTP semantic cleanup belongs to the later authoritative-cutover task.

## 8. SAFE LOGGING

Log only what is required to compare decisions.

Allowed fields:

product = dm
legacy_decision
central_decision
normalized_reason
environment
release/commit if already available
optional non-reversible actor hash if an existing safe helper exists

Never log:

Authorization header
bearer token
JWT
email
raw service-role
secret
password
full customer identity
Supabase secret
raw access token

Avoid raw user/customer IDs unless the repo already has a safe irreversible hash helper.

Do not introduce new cryptographic infrastructure just for logging.

## 9. OBSERVABILITY EVENTS

At minimum distinguish:

MATCH_ALLOW
MATCH_DENY
MISMATCH_LEGACY_ALLOW_CENTRAL_DENY
MISMATCH_LEGACY_DENY_CENTRAL_ALLOW
CENTRAL_ERROR
CENTRAL_UNKNOWN

Logging must not throw and must never break auth flow.

## 10. TESTS

Create:

tests/auth/test_seat_shadow.py

Add/extend only minimal entitlement tests if required.

Required tests:

1.
local bypass returns exactly as before
and Central Seat client/helper is NOT called

2.
local bypass platform_admin loopback behavior remains unchanged

3.
legacy ALLOW + central ALLOW
→ request ALLOW unchanged
→ match event

4.
legacy ALLOW + central DENY
→ request ALLOW unchanged
→ mismatch event

5.
legacy DENY + central DENY
→ original 403 unchanged
→ match deny event

6.
legacy DENY + central ALLOW
→ original 403 unchanged
→ mismatch event

7.
central ERROR
→ legacy ALLOW remains ALLOW

8.
central ERROR
→ legacy DENY remains original DENY

9.
central UNKNOWN
→ legacy result unchanged

10.
platform_admin legacy behavior unchanged

11.
test_access=allow unchanged

12.
test_access=deny unchanged

13.
no secret/token contents in captured logs

14.
protected routers still depend on shared EntitledUserDep
Do not rewire routers.

15.
existing DM RLS tests remain unaffected

Run relevant existing test suites:

tests/auth/test_entitlement.py
tests/auth/test_jwt_dependencies.py
tests/api/test_entitlement_contract.py
tests/rls/test_designs_rls.py

and the new Seat shadow tests.

Do not edit tests/test_extraction_shadow.py.

## 11. NO LIVE STAGING REQUIREMENT

This task is LOCAL IMPLEMENTATION ONLY.

Do not:
- deploy to Staging
- mutate Staging
- call Production
- make Central authoritative
- change Staging env
- change Production env

If a live Central call is required merely to run tests, use mocks/fakes consistent with existing test conventions instead.

## 12. NO CUTOVER

Do not remove:

has_dm_access()
my_commercial_context()
EntitledUserDep
legacy 403 behavior
local bypass
platform_admin behavior
test_access behavior

Do not make product_seat_status("dm") authoritative.

That belongs to a later cutover task after Staging shadow evidence.

## 13. FILE COLLISION RULE

Before editing entitlement.py:

capture its current working-tree diff.

Modify ONLY the minimal call-site region after the local bypass path.

If the required edit would overwrite/restructure the existing dirty YUCT auth hunk:
STOP.

Do not refactor require_dm_access() broadly.

## 14. FINAL REPORT

### A. PREFLIGHT

Branch:
HEAD:
Dirty files relevant to auth:

### B. COLLISION SAFETY

P4B-R1 assumptions still valid:
YES/NO

Existing YUCT auth hunk preserved:
YES/NO

### C. FILES CHANGED

List only actual files changed.

### D. SHADOW ARCHITECTURE

Legacy authority:
Shadow helper:
Central RPC:
Insertion point:
Local bypass shadow behavior:

### E. DECISION MATRIX

Legacy | Central | Final request result | Event

Include:
ALLOW/ALLOW
ALLOW/DENY
DENY/ALLOW
DENY/DENY
ALLOW/ERROR
DENY/ERROR
UNKNOWN cases

### F. TEST RESULTS

New Seat shadow tests:
Existing entitlement tests:
JWT dependency tests:
API entitlement contract tests:
RLS tests:

### G. SAFETY

Must state:

CENTRAL_AUTHORITATIVE = NO
USER_VISIBLE_AUTH_BEHAVIOR_CHANGED = NO
LOCAL_DEV_BYPASS_CHANGED = NO
PLATFORM_ADMIN_BEHAVIOR_CHANGED = NO
TEST_ACCESS_BEHAVIOR_CHANGED = NO
DM_RLS_CHANGED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

### H. FINAL RESULT

If successful:

RESULT = PASS
DM_SHADOW_IMPLEMENTED = YES
LEGACY_AUTHORITY_PRESERVED = YES
DIRTY_YUCT_WIP_PRESERVED = YES
READY_FOR_DM_STAGING_SHADOW = YES

If blocked:

RESULT = BLOCKED
DM_SHADOW_IMPLEMENTED = NO
READY_FOR_DM_STAGING_SHADOW = NO
BLOCKER = exact file/symbol collision or failing invariant

Stop after local implementation and tests.
