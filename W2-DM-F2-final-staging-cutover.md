# W2-DM-F2 — Final DM Staging Cutover

ROLE: DM FINAL STAGING RELEASE OWNER

WORKSPACE: F:\00-Ticenpi-SaaS
DM REPO: F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Finish the already-green DM final candidate through artifact proof, clean deploy-config pin, one Staging deploy, and final Central Seat E2E.

VERIFIED INPUTS:
- App/fix source commit: 2dd02e2378600f02b9d14590391b9edf84bc14a2
- CI run: 35968635281
- Backend digest: sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
- Frontend digest: sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a
- Extraction shadow PASS
- Seat regression PASS
- Readiness regression PASS
- Auth/RLS regression PASS
- API regression PASS
- Staging not yet changed by this final candidate
- Production untouched
- Existing deploy/runtime-staging/compose.yml has unrelated dirty changes and must not be reused blindly

RUNTIME INPUTS:
DM_TEST_ASSIGNED_USER_ID=<approved ordinary Staging user>
DM_TEST_UNASSIGNED_USER_ID=<approved ordinary Staging user>

RULES:
- Staging only.
- Do not touch Production.
- Do not rebuild app unless published artifact proof fails.
- Do not change Seat/readiness/auth/extraction semantics.
- Preserve unrelated dirty WIP.
- Use a clean isolated deploy-config worktree.
- Do not direct-write Central Seat tables.

1. ARTIFACT PROOF
Directly verify both exact published image digests from the registry/package metadata using the authenticated environment available to the operator.

Cross-check:
- digest exists
- repository identity is correct
- revision/source metadata points to 2dd02e2378600f02b9d14590391b9edf84bc14a2
- CI run 35968635281 reports the same digest pair

If package-read authorization is missing, pause and ask the operator to grant only the minimum package-read permission through the supported GitHub authentication flow, then continue in the same session.

Required:
REGISTRY_PROVENANCE = PASS

Do not deploy if this is not PASS.

2. CLEAN DEPLOY-CONFIG PIN
Start from the proven DM deploy-config baseline in an isolated clean worktree.

Change only the exact backend/frontend image pins to:
backend = sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
frontend = sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a

Keep:
- environment=staging
- Staging Supabase target
- auth required
- authoritative DM Central Seat policy
- readiness-hardening behavior

Run preflight/config validation and prove no unrelated change is included.

Commit only this deploy-config change.

Return:
DM_DEPLOY_CONFIG_COMMIT =

3. FIXTURE GATE
Verify the approved assigned/unassigned ordinary Staging users:
- both exist
- both non-admin
- test_access none/null
- same active business customer
- same effective DM entitlement
- sufficient seat_limit
- assigned user has DM Seat
- unassigned user has no DM Seat

If normalization is required, use only canonical platform_admin RPCs.

Required:
DM_FIXTURE_READY = YES

4. DEPLOY ONCE
Deploy only DM Staging from the clean deploy-config candidate.

Verify:
- exact new release ID
- running backend digest matches expected
- running frontend digest matches expected
- app source commit matches 2dd02e2378600f02b9d14590391b9edf84bc14a2
- deploy-config commit matches
- health 200
- /api/ready 200 and shallow
- deep health still available
- no crash loop
- Central Seat is authoritative
- environment remains Staging

5. SECURITY SMOKE
Verify:
- no token -> 401
- invalid token -> 401
- auth/RLS verifier PASS
- fail-closed behavior preserved

6. FINAL BROWSER E2E
Assigned ordinary user:
- login succeeds
- DM opens
- protected API succeeds
- no Seat denial

Unassigned ordinary user:
- login succeeds
- DM commercial access denied
- canonical 403 / Seat-not-assigned behavior
- protected DM data does not load

If Google login requires operator action, pause only for login and continue afterward.

7. CAUSAL PROOF
Prove the two ordinary users share the same active customer and DM entitlement and differ only in DM Seat assignment.

Required:
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

8. ROLLBACK
On any mandatory deploy/health/security/E2E failure, roll back only DM Staging to the previous known-good release and prove health 200.

9. FINAL REPORT
Return:
RESULT =
REGISTRY_PROVENANCE =
DM_APP_SOURCE_COMMIT =
BACKEND_IMAGE_DIGEST =
FRONTEND_IMAGE_DIGEST =
BUILD_RUN_ID = 35968635281
DM_DEPLOY_CONFIG_COMMIT =
DM_STAGING_RELEASE =
DM_FIXTURE_READY =
CENTRAL_AUTHORITATIVE =
DM_ASSIGNED_ALLOW =
DM_UNASSIGNED_DENY =
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN =
READY_SHALLOW =
DEEP_HEALTH_PRESERVED =
AUTH_RLS_REGRESSION =
PRODUCTION_MUTATED = NO

If all pass:
RESULT = PASS
DM_CENTRAL_SEAT_STAGING_CUTOVER = PASS
READY_FOR_DM_PRODUCTION_CUTOVER = YES

If only interactive login remains:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_REQUIRED = YES
READY_FOR_DM_PRODUCTION_CUTOVER = NO

Do not deploy Production.
