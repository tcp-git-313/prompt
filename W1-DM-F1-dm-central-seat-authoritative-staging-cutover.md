# W1-DM-F1 — DM Central Seat Authoritative Staging Cutover

ROLE: DM CENTRAL SEAT STAGING CUTOVER OWNER

MODE: IMPLEMENTATION + RELEASE CANDIDATE + STAGING DEPLOY + FINAL E2E

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

CENTRAL PLATFORM REPO (REFERENCE / CANONICAL RPC USE ONLY):
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Take DM from the current locally-implemented Central Seat shadow state to a fully authoritative Central Seat Staging state, with fixed assigned/unassigned regression users and final E2E.

Do not touch Production.

## VERIFIED PRIOR STATE

Prior DM shadow work:
- P4B-R2 = PASS
- DM_SHADOW_IMPLEMENTED = YES
- LEGACY_AUTHORITY_PRESERVED = YES
- DIRTY_YUCT_WIP_PRESERVED = YES
- READY_FOR_DM_STAGING_SHADOW = YES

Known baseline branch at that time:
dm/integration-20260920

Known baseline HEAD at that time:
561bd8c76fec5c0fb86cb9cf4f56734d28367b8a

Known dirty work in the main checkout included YUCT extraction-shadow and OAuth/UAT work.

The Central Seat core is already live in Staging and verified.

Post already proved the canonical business pattern:
- platform_admin exempt
- assigned ordinary business user allowed
- unassigned ordinary business user denied
- invalid JWT denied

DM must now implement the same commercial rule for product_code="dm", while preserving DM-specific RLS and local-dev behavior.

## ABSOLUTE RULES

- Staging only.
- No Production mutation.
- Do not overwrite YUCT/OAuth dirty WIP.
- Prefer an isolated release worktree/branch.
- Do not reset/clean/stash/rebase the user's main DM checkout.
- Do not weaken DM RLS.
- Do not remove get_current_user(), require_dm_access(), EntitledUserDep, or commercial context.
- Do not use service-role in browser/user flow.
- Do not use test_access allow/deny to fake final Seat E2E.
- Do not make Central Seat checks route-by-route.
- Do not run another product deployment in parallel.
- Roll back DM Staging only on mandatory failure.

## 1. PREFLIGHT / CURRENT STATE RECONSTRUCTION

Record:
- main checkout branch
- main checkout HEAD
- git status --short
- git diff --stat
- current Staging release
- current Staging image digest
- current Staging auth/commercial config

Re-read the current working-tree DM shadow implementation:
- src/backend/app/auth/seat_shadow.py
- src/backend/app/auth/entitlement.py
- tests/auth/test_seat_shadow.py
- tests/auth/test_entitlement.py

Confirm the local bypass/YUCT changes remain distinct.

If the shadow implementation is not present anymore:
STOP and report exact drift.

## 2. ISOLATE THE DM CENTRAL SEAT CANDIDATE

Create a dedicated release worktree/branch from the appropriate validated DM integration baseline.

Bring in ONLY the DM Central Seat shadow changes and any minimal authoritative-policy additions required by this task.

Do not include:
- unrelated YUCT UI work
- OAuth/UAT work
- unrelated extraction changes
- unrelated dirty files

Required:
DM_CANDIDATE_DIFF_ISOLATED = YES

If exact isolation cannot be proven:
STOP.

## 3. DEFINE ONE SHARED DM ACCESS DECISION

The final authenticated DM access path must remain centralized:

get_current_user
→ require_dm_access()
→ commercial context
→ canonical Central Seat decision for product_code="dm"
→ protected DM routes
→ DM RLS

No per-router Seat duplication.

Local/dev bypass:
- remains available only under existing safe local conditions
- skips real Central Seat because there is no real caller bearer
- must not become active in Staging/Production

## 4. ADD AUTHORITATIVE POLICY WITHOUT REWRITING SHADOW ARCHITECTURE

Reuse the existing isolated seat_shadow client/helper where practical.

Introduce one explicit DM Central Seat policy with clear modes:

shadow
require

Default formal Staging target:
require

In shadow:
- legacy result remains authoritative

In require:
- canonical Central Seat becomes authoritative for ordinary business users

Do not create a third Seat authority.

## 5. FINAL DM COMMERCIAL RULE

For real authenticated users:

platform_admin:
ALLOW / exempt according to canonical Central Seat contract

individual valid entitlement:
ALLOW without manual Seat if canonical Central Seat returns not_required/exempt/allowed

business active member + effective DM entitlement + assigned DM Seat:
ALLOW

business valid entitlement but no DM Seat:
DENY 403

invalid/suspended/removed commercial state:
DENY

missing/invalid JWT:
401

Central dependency failure in require mode:
fail closed with 503 or the repository's canonical service-unavailable mapping
Do NOT convert Central infrastructure failure into 401.

Preserve existing RLS.

## 6. FIXED STAGING DM TEST FIXTURE

Reuse existing ordinary Staging Auth regression users if they can be safely discovered from the existing test-only Commercial Core fixture.

Preferred:
- same two ordinary test Auth users already used for Post Central Seat regression
- same test-only customer if appropriate

Do not require new Auth accounts.

If identities are ambiguous, accept runtime operator inputs:
DM_TEST_ASSIGNED_USER_ID
DM_TEST_UNASSIGNED_USER_ID

Do not publish credentials.

Ensure both users:
- are ordinary non-admin
- are active members of the same test-only customer
- share the same effective DM entitlement eligibility
- are not using test_access bypass

Ensure the test customer has:
product_code = dm
effective status active/trial
seat_limit >= 2

Using canonical RPCs:
- assign DM Seat to assigned user
- ensure unassigned user has no DM Seat

Do not directly write product_seat_assignments.

## 7. DM TENANCY / DATA BOUNDARY

Inspect the actual DM tenancy model.

Do NOT invent a Post-style tenant_members table if DM does not use one.

Prove what actually scopes DM data:
- customer_id
- organization_id
- RLS claims/context
- other exact canonical mechanism

Final E2E must ensure assigned/unassigned difference is attributable to Central Seat, while DM RLS remains unchanged.

Set:
DM_DATA_BOUNDARY_PRESERVED = YES

## 8. LOCAL TESTS

Required tests:
1. shadow mode preserves legacy behavior
2. require mode enforces Central Seat
3. assigned business user → allow
4. unassigned business user → 403
5. platform_admin → allow/exempt
6. individual valid entitlement → allowed according to canonical Central result
7. invalid JWT → 401
8. Central error in require mode → fail closed / 503
9. local bypass unchanged
10. test_access behavior unchanged where still relevant
11. no secret logging
12. EntitledUserDep remains the shared protected-route dependency
13. DM RLS tests unchanged and passing

Run:
- new Seat policy tests
- tests/auth/test_entitlement.py
- tests/auth/test_jwt_dependencies.py
- tests/api/test_entitlement_contract.py
- scripts/verify_auth_rls.py
- relevant RLS tests
- git diff --check

## 9. RELEASE CANDIDATE / CI

Create a dedicated DM Staging release candidate.

Commit only isolated Central Seat changes.

Push normally.
No force push.

Run required CI.

Do not deploy if required CI is not green.

Record immutable:
DM_APP_SOURCE_COMMIT
DM_IMAGE_DIGEST
DM_DEPLOY_CONFIG_COMMIT if the deployment system separates them

## 10. STAGING DEPLOY — SHADOW GATE FIRST

Deploy DM Staging only.

Start in shadow mode if the current architecture supports a config-only switch without rebuilding.

Use one real ordinary Staging user request to prove:
product_seat_status("dm")
is actually called.

Required:
DM_NON_ADMIN_LIVE_CENTRAL_CALL_PROVEN = YES

If shadow mode cannot be separated from require without another safe config step, use the smallest safe equivalent proof before enforcing.

## 11. SWITCH DM STAGING TO REQUIRE

After live Central call proof:

set formal Staging DM policy to require.

Do not alter Production.

Read actual runtime config after deploy.

Required:
DM_CENTRAL_SEAT_POLICY = require
CENTRAL_AUTHORITATIVE = YES

## 12. FINAL LIVE E2E

Mandatory:

A. platform_admin
→ ALLOW

B. assigned ordinary DM user
→ Central DM Seat ALLOW
→ DM data boundary valid
→ final ALLOW

C. unassigned ordinary DM user
→ Central DM Seat DENY/not_assigned
→ final 403

D. invalid JWT
→ 401

Optional if safely testable:
Central service failure
→ fail closed / 503

## 13. CAUSAL PROOF

Assigned and unassigned ordinary users must share:
- same test customer
- same membership eligibility
- same effective DM entitlement class
- same DM data-boundary eligibility

and differ only in:
Central DM Seat assignment

Required:
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

## 14. HEALTH / SECURITY / ROLLBACK

Verify:
- DM internal/external health 200
- no crash loop
- auth logs contain no tokens/JWT/password/service-role/secrets
- DM RLS still passes

Before deploy record previous DM Staging release.

If mandatory deploy/E2E/security fails:
rollback DM Staging only
restore prior config
keep the fixed test fixture for retry
prove health 200

## 15. FINAL REPORT

A. PREFLIGHT
B. ISOLATED CANDIDATE
C. FINAL AUTH CHAIN
D. TEST FIXTURE
E. DM DATA BOUNDARY
F. LOCAL TESTS
G. CI / ARTIFACT IDENTITY
H. STAGING DEPLOY
I. LIVE CENTRAL CALL
J. REQUIRE POLICY
K. FINAL E2E
L. CAUSAL PROOF
M. HEALTH / SECURITY / ROLLBACK
N. PRODUCTION SAFETY

Required final fields:

DM_CANDIDATE_DIFF_ISOLATED =
DM_NON_ADMIN_LIVE_CENTRAL_CALL_PROVEN =
DM_CENTRAL_SEAT_POLICY =
CENTRAL_AUTHORITATIVE =
DM_BUSINESS_ASSIGNED_ALLOW =
DM_BUSINESS_UNASSIGNED_DENY =
DM_DATA_BOUNDARY_PRESERVED =
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN =
PRODUCTION_MUTATED = NO

If complete:

RESULT = PASS
DM_CENTRAL_SEAT_STAGING_CUTOVER = PASS
READY_FOR_DM_PRODUCTION_CUTOVER = YES

If live browser login is unavailable after fixture/deploy:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_E2E_REQUIRED = YES
READY_FOR_DM_PRODUCTION_CUTOVER = NO

If isolation/runtime safety fails:
RESULT = BLOCKED or ROLLED_BACK
BLOCKER = exact factual blocker

Do not deploy Production.
