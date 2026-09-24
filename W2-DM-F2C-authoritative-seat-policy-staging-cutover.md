# W2-DM-F2C — DM Authoritative Seat Policy Staging Cutover

ROLE: DM FINAL STAGING CUTOVER OWNER

MODE: CONFIG-AUTHORIZED CUTOVER + LIVE FIXTURE VERIFY/NORMALIZE + SINGLE STAGING DEPLOY + FINAL E2E

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

PLATFORM CONTRACT REFERENCE:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Complete the DM Staging Central Seat authoritative cutover from the already-green final candidate.

This task explicitly authorizes the Staging deploy-config change from unset/shadow Seat policy to:

DM_SEAT_POLICY=require

That policy change is part of the intended DM Central Seat cutover and is NOT an unrelated configuration change.

VERIFIED APP CANDIDATE:
- App/source commit:
  2dd02e2378600f02b9d14590391b9edf84bc14a2
- CI run:
  35968635281
- Backend digest:
  sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
- Frontend digest:
  sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a
- Extraction shadow regression: PASS
- Seat regression: PASS
- Readiness regression: PASS
- Auth/RLS regression: PASS
- API regression: PASS
- Full backend suite: PASS
- CI artifact evidence maps both immutable digests to source commit above
- Production has not been changed

VERIFIED CURRENT STAGING:
- Current release: 20260923-215914
- Current Seat policy is unset/default shadow
- Current Staging is healthy
- Current rollback target is known
- Existing dirty deploy/runtime-staging/compose.yml must NOT be reused wholesale
- Previous F2B stopped correctly because it was not authorized to add DM_SEAT_POLICY=require

RUNTIME TEST IDENTITIES:
DM_TEST_ASSIGNED_USER_ID=<approved ordinary Staging user UUID>
DM_TEST_UNASSIGNED_USER_ID=<approved ordinary Staging user UUID>

Use the already-approved ordinary Staging regression pair.
Do not create new Auth users.

If the exact runtime IDs are not supplied, discover them only from approved existing Staging regression evidence.
Do not infer from email ordering or stale fixture files.
If safe identification is impossible, stop and request the two IDs.

ABSOLUTE RULES:
- Staging only.
- Do NOT touch Production.
- Do NOT rebuild the application.
- Do NOT change app source.
- Do NOT change Seat/readiness/auth/extraction semantics.
- Do NOT modify unrelated dirty WIP.
- Use a clean isolated deploy-config worktree/branch.
- Do NOT copy the dirty compose wholesale.
- Do NOT direct-write product_seat_assignments.
- Use canonical platform_admin RPCs for any fixture normalization.
- Deploy by immutable sha256 digests only.
- Exactly one DM Staging deploy after all preflight gates pass.

AUTHORIZED DEPLOY-CONFIG DELTA:
Only these changes are allowed:

1. Backend image digest:
   sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468

2. Frontend image digest:
   sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a

3. DM_SEAT_POLICY=require

No other config changes are authorized unless required by an already-existing exact Staging contract and proven absent by preflight. Any such discrepancy is a HARD STOP.

1. CI / IMMUTABLE ARTIFACT EVIDENCE

Re-verify CI run 35968635281:
- conclusion success
- source SHA exact match
- backend digest exact match
- frontend digest exact match
- all required build jobs green

Return:
CI_ARTIFACT_EVIDENCE = PASS/FAIL

Do not continue on mismatch.

2. CLEAN DEPLOY-CONFIG BASELINE

Identify the previously reconciled clean DM deploy-config baseline.

Create an isolated clean worktree/branch from that baseline.

Do not reuse current dirty local compose edits.

Apply only the three authorized changes above.

Run:
- config/preflight validation
- diff review
- syntax validation
- any existing deployment identity checks

Return:
ONLY_AUTHORIZED_CONFIG_DELTA = YES/NO

If NO:
HARD STOP.

3. LIVE STAGING FIXTURE VERIFICATION

Use live Staging data, not historical fixture assumptions.

For both approved ordinary users verify:
- Auth user exists
- profile exists
- platform_role=user
- test_access null/none
- non-admin
- both belong to the same approved active business customer

Then verify current DM commercial state:
- DM product exists
- effective DM entitlement exists for the shared customer
- entitlement active
- seat_limit sufficient for this test
- current DM Seat assignment for both users

Return:
DM_TEST_CUSTOMER_ID =
DM_EFFECTIVE_ENTITLEMENT_ID =
DM_SEAT_LIMIT =
ASSIGNED_USER_CURRENT_DM_SEAT =
UNASSIGNED_USER_CURRENT_DM_SEAT =

4. CANONICAL FIXTURE NORMALIZATION

If the live fixture is not ready, normalize only through canonical platform_admin RPCs.

Required final state:
- shared active business customer
- effective active DM entitlement
- seat_limit >= 2
- assigned test user has DM Seat
- unassigned test user has no DM Seat

Do not direct-write Central Seat tables.

Re-read after mutation and prove exact final state.

Return:
DM_FIXTURE_READY = YES/NO

Do not continue if NO.

5. DEPLOY-CONFIG COMMIT

Commit only the authorized deploy-config delta:
- backend digest
- frontend digest
- DM_SEAT_POLICY=require

Return:
DM_DEPLOY_CONFIG_COMMIT =

No app-source commit in this step.

6. SINGLE DM STAGING DEPLOY

Deploy only dmruntimestaging.

Do not deploy any other service.

Immediately record:
DM_STAGING_RELEASE =
RUNNING_BACKEND_DIGEST =
RUNNING_FRONTEND_DIGEST =
RUNNING_APP_SOURCE_COMMIT =
RUNNING_DEPLOY_CONFIG_COMMIT =
RUNNING_DM_SEAT_POLICY =

7. POST-DEPLOY IMMUTABLE DIGEST CHAIN

Prove:
- running backend digest == expected backend digest
- running frontend digest == expected frontend digest
- runtime app source commit == 2dd02e2378600f02b9d14590391b9edf84bc14a2
- running deploy-config identity == DM_DEPLOY_CONFIG_COMMIT
- DM_SEAT_POLICY == require
- no mutable tag substitution

If all match:
REGISTRY_PROVENANCE = PASS_BY_IMMUTABLE_DIGEST_CHAIN

If any mismatch:
rollback DM Staging immediately.

8. HEALTH / READINESS / SECURITY

Verify:
- /api/health = 200
- /api/ready = 200 and shallow/fast
- /api/health/detail remains available
- no crash/restart loop
- environment = staging
- Supabase project = jlsqjvehwblkeuycjoyj
- no token -> 401
- invalid token -> 401
- auth/RLS verifier PASS
- fail-closed behavior preserved
- Central Seat policy is authoritative

Return:
CENTRAL_AUTHORITATIVE = YES/NO

9. FINAL ORDINARY-USER BROWSER E2E

Assigned ordinary user:
- Google login succeeds
- DM opens
- protected API succeeds
- no Seat denial

Return:
DM_ASSIGNED_ALLOW = PASS/FAIL

Then fully logout/clear session.

Unassigned ordinary user:
- Google login succeeds
- authentication itself succeeds
- DM commercial access is denied
- canonical 403 / Seat-not-assigned behavior
- protected DM data/API does not load

Return:
DM_UNASSIGNED_DENY = PASS/FAIL

If Google OAuth requires operator interaction:
pause only for the relevant login and continue in the same task/session afterward.

10. CAUSAL CENTRAL SEAT PROOF

Prove both ordinary users share:
- same active business customer
- same effective DM entitlement
- same ordinary/non-admin class
- same test_access semantics

and differ in:
- DM Seat assignment

Required:
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

11. ROLLBACK

On any mandatory deploy/health/security/E2E failure:
rollback only DM Staging to the previous known-good release.
Prove health 200.
Do not touch Production.

12. FINAL REPORT

Return exactly:

RESULT =
CI_ARTIFACT_EVIDENCE =
ONLY_AUTHORIZED_CONFIG_DELTA =
DM_DEPLOY_CONFIG_COMMIT =
DM_STAGING_RELEASE =
RUNNING_BACKEND_DIGEST =
RUNNING_FRONTEND_DIGEST =
RUNNING_APP_SOURCE_COMMIT =
RUNNING_DEPLOY_CONFIG_COMMIT =
RUNNING_DM_SEAT_POLICY =
REGISTRY_PROVENANCE =
DM_TEST_CUSTOMER_ID =
DM_EFFECTIVE_ENTITLEMENT_ID =
DM_SEAT_LIMIT =
DM_FIXTURE_READY =
CENTRAL_AUTHORITATIVE =
DM_ASSIGNED_ALLOW =
DM_UNASSIGNED_DENY =
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN =
READY_SHALLOW =
DEEP_HEALTH_PRESERVED =
AUTH_RLS_REGRESSION =
PRODUCTION_MUTATED = NO

If everything passes:
RESULT = PASS
REGISTRY_PROVENANCE = PASS_BY_IMMUTABLE_DIGEST_CHAIN
DM_CENTRAL_SEAT_STAGING_CUTOVER = PASS
READY_FOR_DM_PRODUCTION_CUTOVER = YES

If only interactive login remains:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_REQUIRED = YES
READY_FOR_DM_PRODUCTION_CUTOVER = NO

Do not deploy Production.
