# W2-DM-F1R — Final DM Central Seat Authoritative Staging Candidate

ROLE: DM FINAL COMMERCIAL STAGING OWNER

MODE: IMPLEMENTATION + CI + STAGING DEPLOY + FINAL E2E

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

PLATFORM CONTRACT REFERENCE:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Create one final DM Staging commercial candidate that combines the verified readiness hardening with authoritative Central Seat enforcement, then deploy once and complete final Staging E2E.

VERIFIED BASELINE:
- Current DM Staging release: 20260923-215914
- App source baseline: 25db5f9b8b65a231dafb7cc8f1f05262fb7519aa
- Deploy-config baseline: 34a59a7fb50ce5f8b6071e90ab5ee12ca763f3fb
- Running backend/frontend digests are reconciled to exact CI evidence
- Readiness hardening commit: 67508d1eb290556d1bb33ea9f6218dd12974438f
- Readiness hardening tests: PASS
- Current DM commercial access uses my_commercial_context()/has_dm_access()
- Current deployed source does NOT enforce product_seat_status("dm")
- Rollback release: 20260922-131327

RUNTIME INPUTS:
DM_TEST_ASSIGNED_USER_ID=<existing ordinary Staging Auth UUID>
DM_TEST_UNASSIGNED_USER_ID=<existing ordinary Staging Auth UUID>

Use the same two ordinary Staging users already proven by Post.
Do not create new Auth accounts.

ABSOLUTE RULES:
- Staging only.
- Do NOT touch Production.
- Use a clean isolated worktree/branch.
- Do NOT include YUCT, OAuth, extraction, UI, local-admin-bypass, or unrelated dirty WIP.
- Do NOT use service-role for runtime user access.
- Do NOT direct-write product_seat_assignments.
- Do NOT weaken DM RLS.
- Do NOT duplicate Seat checks route-by-route.
- Do NOT deploy readiness hardening separately.
- Build/deploy exactly one final DM candidate.

1. PREFLIGHT
Record:
- current main checkout branch/HEAD/status
- exact baseline commits above
- current Staging release/health
- current runtime auth/commercial config
- current rollback target

2. CREATE ISOLATED FINAL CANDIDATE
Start from the reconciled DM deployment baseline.
Integrate only:
A. readiness hardening commit 67508d1...
B. previously validated Central Seat shadow/require implementation and focused tests
C. minimal final config needed for authoritative mode

Required:
FINAL_CANDIDATE_ISOLATED = YES

3. SHARED DM ACCESS CHAIN
Final chain must be centralized:

Bearer JWT
→ get_current_user
→ require_dm_access
→ my_commercial_context / existing DM commercial eligibility
→ product_seat_status("dm")
→ DM protected APIs
→ existing DM RLS

No per-router duplication.

4. AUTHORITATIVE POLICY
Provide explicit policy modes if existing design already supports them:
shadow
require

Final Staging target:
require

Required behavior:
- platform_admin → ALLOW/exempt
- individual valid entitlement → ALLOW when canonical Seat RPC says not_required/exempt/allowed
- business active member + valid DM entitlement + assigned DM Seat → ALLOW
- business valid entitlement + no DM Seat → 403
- invalid/missing JWT → 401
- Central dependency failure → fail closed / 503
- local safe bypass remains local-only and inactive in Staging

5. COMMERCIAL GATE
Inspect actual code before setting config.
Do not invent a separate gate if require_dm_access already owns the path.

If TICENPI_COMMERCIAL_GATE_ENABLED is required by the actual DM path:
set the Staging candidate explicitly to the required value.

Record:
COMMERCIAL_GATE_REQUIRED =
COMMERCIAL_GATE_EFFECTIVE =

6. FIXED STAGING FIXTURE
Using canonical platform_admin RPCs only:
- verify both supplied users are ordinary, non-admin, test_access=null
- verify active membership in the approved shared test customer
- ensure effective DM entitlement exists for that customer
- set seat_limit >= 2
- assign DM Seat to DM_TEST_ASSIGNED_USER_ID
- ensure DM_TEST_UNASSIGNED_USER_ID has no DM Seat

Do not direct-write Seat tables.

7. TESTS
Mandatory:
- readiness tests
- assigned business allow
- unassigned business 403
- platform_admin allow/exempt
- individual behavior
- invalid JWT 401
- Central failure fail-closed
- local bypass unchanged
- test_access semantics unchanged
- DM RLS unchanged
- no secret logging

Run:
- focused auth/Seat tests
- readiness tests
- verify_auth_rls.py
- relevant API tests
- git diff --check

The previously observed unrelated removebg/images auth assertions must NOT be silently fixed in this task.
If they still fail unchanged, report them as baseline-known failures with evidence.

8. CI / IMMUTABLE ARTIFACT
Build exact candidate through normal CI.
Record:
DM_APP_SOURCE_COMMIT
BACKEND_IMAGE_DIGEST
FRONTEND_IMAGE_DIGEST
BUILD_RUN_ID

9. DEPLOY-CONFIG PIN
Create the smallest config-only commit that pins those exact digests.
Use the split artifact/deploy-config identity model already proven by current DM release.

Record:
DM_DEPLOY_CONFIG_COMMIT

10. STAGING DEPLOY
Deploy DM Staging only once.

Verify:
- release identity
- exact digests
- config identity
- health 200
- /api/ready fast and shallow
- deep health still available
- no crash loop

11. FINAL E2E
A. platform_admin → ALLOW
B. assigned ordinary user → ALLOW
C. unassigned ordinary user → 403
D. invalid JWT → 401

Prove assigned/unassigned users share:
- same active customer
- same DM entitlement eligibility
- same DM data-boundary eligibility

and differ in:
Central DM Seat assignment

Required:
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

12. ROLLBACK
If mandatory deploy/E2E/security fails:
rollback DM Staging only to 20260922-131327 and prove health 200.

13. FINAL REPORT
Return:
FINAL_CANDIDATE_ISOLATED =
DM_APP_SOURCE_COMMIT =
BACKEND_IMAGE_DIGEST =
FRONTEND_IMAGE_DIGEST =
DM_DEPLOY_CONFIG_COMMIT =
COMMERCIAL_GATE_REQUIRED =
DM_SEAT_POLICY =
CENTRAL_AUTHORITATIVE =
DM_ASSIGNED_ALLOW =
DM_UNASSIGNED_DENY =
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN =
READY_SHALLOW =
DEEP_HEALTH_PRESERVED =
RLS_PRESERVED =
PRODUCTION_MUTATED = NO

If complete:
RESULT = PASS
DM_CENTRAL_SEAT_STAGING_CUTOVER = PASS
READY_FOR_DM_PRODUCTION_CUTOVER = YES

If interactive login alone remains:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_E2E_REQUIRED = YES
READY_FOR_DM_PRODUCTION_CUTOVER = NO

Do not deploy Production.
