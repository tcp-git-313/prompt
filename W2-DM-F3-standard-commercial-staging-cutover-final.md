# W2-DM-F3 — DM Standard Commercial Staging Cutover Final

ROLE: DM STANDARD COMMERCIAL STAGING CUTOVER OWNER

WORKSPACE: F:\00-Ticenpi-SaaS
DM REPO: F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Finish DM exactly like the other products: complete the missing Staging commercial settings and fixture, deploy the already-tested immutable candidate once, then run final assigned/unassigned E2E.

VERIFIED CANDIDATE
- source commit: 2dd02e2378600f02b9d14590391b9edf84bc14a2
- CI run: 35968635281
- backend digest: sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
- frontend digest: sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a
- backend/extraction/Seat/readiness/auth-RLS/API tests: PASS
- Production unchanged

CURRENT MISSING PIECES
1. clean deploy config must add DM_SEAT_POLICY=require
2. live DM entitlement/Seat fixture must be verified and normalized
3. exact immutable digests must be pinned and deployed to DM Staging
4. assigned/unassigned browser E2E must be completed

RUNTIME INPUTS
DM_TEST_ASSIGNED_USER_ID=<approved ordinary Staging UUID>
DM_TEST_UNASSIGNED_USER_ID=<approved ordinary Staging UUID>

Use the already-approved ordinary Staging regression pair. Do not create new Auth users.

RULES
- Staging only; never touch Production.
- Do not rebuild or change app source.
- Do not change Seat/readiness/auth/extraction implementation.
- Preserve unrelated dirty WIP.
- Use a clean isolated deploy-config worktree.
- Do not copy dirty compose wholesale.
- Use canonical platform_admin RPCs for fixture normalization.
- Never direct-write Seat tables.
- Deploy exact sha256 digests, never mutable tags.
- Do not add extra DM-only release gates beyond the standard immutable digest chain.

STANDARD ARTIFACT PROOF
Artifact identity is PASS when:
1. CI run is successful.
2. CI source SHA matches the candidate commit.
3. CI backend/frontend digests match the expected digests.
4. deploy config pins those exact digests.
5. post-deploy running digests match exactly.
6. runtime source identity matches the same source SHA.
7. deploy-config commit is recorded.

No direct GHCR metadata read is required if this chain passes.

1. PREFLIGHT
Record current DM branch/HEAD/status, current Staging release/health, rollback target, clean deploy-config baseline, and unrelated dirty compose state.

2. CI EVIDENCE
Re-verify run 35968635281, source SHA, both digests, and required green jobs.
Return CI_ARTIFACT_EVIDENCE=PASS/FAIL.
Stop on mismatch.

3. CLEAN DEPLOY-CONFIG
From the reconciled clean deploy-config baseline, authorize exactly these changes:
- backend digest = sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
- frontend digest = sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a
- DM_SEAT_POLICY=require

Preserve all other verified Staging settings.
Run config/preflight validation.
Require ONLY_AUTHORIZED_CONFIG_DELTA=YES.

4. LIVE DM FIXTURE
Using live Staging data, verify both approved users:
- exist
- platform_role=user
- test_access null/none
- non-admin
- same active business customer

Verify current DM state:
- active effective DM entitlement
- seat_limit >= 2
- assigned user has DM Seat
- unassigned user has no DM Seat

If not ready, normalize with canonical platform_admin RPCs only.
If an admin Google login is required, pause only for that login, then continue.

Require DM_FIXTURE_READY=YES.

5. COMMIT DEPLOY CONFIG
Commit only:
- backend digest
- frontend digest
- DM_SEAT_POLICY=require

Return DM_DEPLOY_CONFIG_COMMIT.

6. SINGLE DM STAGING DEPLOY
Deploy only dmruntimestaging.

7. POST-DEPLOY IDENTITY
Verify:
- running backend digest exact match
- running frontend digest exact match
- runtime app source commit = 2dd02e2378600f02b9d14590391b9edf84bc14a2
- running deploy-config identity = DM_DEPLOY_CONFIG_COMMIT
- DM_SEAT_POLICY=require
- environment=staging
- Staging Supabase project

If all match:
ARTIFACT_PROVENANCE=PASS_BY_IMMUTABLE_DIGEST_CHAIN

Any mismatch: rollback DM Staging.

8. HEALTH / SECURITY
Verify:
- health 200
- /api/ready 200 and shallow/fast
- deep health preserved
- no crash loop
- no token 401
- invalid token 401
- auth/RLS verifier PASS
- fail-closed behavior preserved
- Central Seat authoritative

9. FINAL USER E2E
Assigned ordinary user:
- Google login succeeds
- DM opens
- protected API succeeds
- no Seat denial

Then fully logout.

Unassigned ordinary user:
- Google login succeeds
- authentication succeeds
- DM access denied
- canonical 403 / Seat-not-assigned behavior
- protected DM data does not load

If login requires operator interaction, pause only for that login and continue.

10. CAUSAL PROOF
Prove both users share the same active customer and effective DM entitlement and differ in DM Seat assignment.
Require DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN=YES.

11. FINAL REPORT
Return:
RESULT
CI_ARTIFACT_EVIDENCE
ONLY_AUTHORIZED_CONFIG_DELTA
DM_DEPLOY_CONFIG_COMMIT
DM_STAGING_RELEASE
ARTIFACT_PROVENANCE
RUNNING_BACKEND_DIGEST
RUNNING_FRONTEND_DIGEST
RUNNING_APP_SOURCE_COMMIT
RUNNING_DEPLOY_CONFIG_COMMIT
RUNNING_DM_SEAT_POLICY
DM_TEST_CUSTOMER_ID
DM_EFFECTIVE_ENTITLEMENT_ID
DM_SEAT_LIMIT
DM_FIXTURE_READY
CENTRAL_AUTHORITATIVE
DM_ASSIGNED_ALLOW
DM_UNASSIGNED_DENY
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN
READY_SHALLOW
DEEP_HEALTH_PRESERVED
AUTH_RLS_REGRESSION
PRODUCTION_MUTATED=NO

If all pass:
RESULT=PASS
DM_CENTRAL_SEAT_STAGING_CUTOVER=PASS
READY_FOR_DM_PRODUCTION_CUTOVER=YES

If only interactive login remains:
RESULT=PARTIAL_PASS
MANUAL_LOGIN_REQUIRED=YES
READY_FOR_DM_PRODUCTION_CUTOVER=NO

Do not deploy Production.
