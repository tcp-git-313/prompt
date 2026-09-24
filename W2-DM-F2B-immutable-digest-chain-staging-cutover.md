# W2-DM-F2B — DM Immutable Digest Chain Staging Cutover

ROLE: DM FINAL STAGING RELEASE OWNER

MODE: CI EVIDENCE + IMMUTABLE DIGEST PIN + STAGING DEPLOY + FINAL E2E

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Finish the DM Staging commercial cutover without requiring direct GHCR package-metadata access.

Use the already-green CI release evidence, exact immutable image digests, exact source SHA, clean deploy-config pin, and post-deploy runtime digest/source verification as the artifact provenance chain.

VERIFIED INPUTS:
- Final source commit:
  2dd02e2378600f02b9d14590391b9edf84bc14a2
- CI run:
  35968635281
- Backend digest:
  sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
- Frontend digest:
  sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a
- Extraction shadow: PASS
- Seat regression: PASS
- Readiness regression: PASS
- Auth/RLS regression: PASS
- API regression: PASS
- Full backend suite: PASS
- CI backend/frontend/ARM64 jobs: PASS
- CI release evidence maps both exact digests to the exact source SHA above
- Dockerfile/workflow revision-label configuration points to the same source SHA
- Direct GHCR metadata read is unavailable only because the local GitHub CLI token lacks read:packages
- Existing dirty deploy/runtime-staging/compose.yml changes are unrelated and must not be reused blindly
- Production has not been changed

PROVENANCE POLICY FOR THIS TASK:
Direct GHCR label access is NOT required.

Artifact provenance passes only if all of the following are true:

1. CI run 35968635281 is PASS.
2. CI release evidence reports the exact backend/frontend digests listed above.
3. CI release evidence identifies source commit 2dd02e2378600f02b9d14590391b9edf84bc14a2.
4. Deployment pins the exact sha256 digests, never a mutable tag.
5. After deployment, docker/runtime inspection shows the exact same backend/frontend digests.
6. Runtime health/config identity reports the same app source commit.
7. Deploy-config commit is recorded separately and matches the release.

If all seven conditions pass:
REGISTRY_PROVENANCE = PASS_BY_IMMUTABLE_DIGEST_CHAIN

If any condition disagrees:
STOP and rollback/abort. Do not reinterpret or waive the mismatch.

ABSOLUTE RULES:
- Staging only.
- Do NOT touch Production.
- Do NOT request or require read:packages.
- Do NOT rebuild the application.
- Do NOT change Seat/readiness/auth/extraction semantics.
- Do NOT modify unrelated dirty WIP.
- Use a clean isolated deploy-config worktree.
- Do NOT copy the current dirty compose wholesale.
- Do NOT direct-write Central Seat tables.
- Do NOT deploy by mutable image tags.

RUNTIME TEST IDENTITIES:
Reuse the already-approved ordinary Staging regression pair previously proven in Post/ORC.
Do not create new Auth users.
Do not publish their UUIDs into source files or public prompts.

If the exact pair cannot be resolved safely from existing approved test evidence, stop and request the operator to supply the two IDs.

1. CI ARTIFACT EVIDENCE

Re-read CI run 35968635281 and its release evidence.

Verify:
- run conclusion = success
- source SHA = 2dd02e2378600f02b9d14590391b9edf84bc14a2
- backend digest exact match
- frontend digest exact match
- backend/frontend/ARM64 jobs all successful

Return:
CI_SOURCE_SHA =
CI_BACKEND_DIGEST =
CI_FRONTEND_DIGEST =
CI_ARTIFACT_EVIDENCE = PASS/FAIL

Do not continue on mismatch.

2. CLEAN DEPLOY-CONFIG CANDIDATE

Create a clean isolated worktree/branch from the previously reconciled DM deploy-config baseline.

Change only the minimum required image pins:

backend:
sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468

frontend:
sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a

Preserve:
- environment=staging
- Staging Supabase project
- DM_REQUIRE_AUTH
- authoritative DM Central Seat require policy
- readiness hardening
- current service/port/domain identity

Run config/preflight validation.
Prove no unrelated files/fields changed.

Commit only the clean deploy-config pin.

Return:
DM_DEPLOY_CONFIG_COMMIT =
ONLY_EXPECTED_DEPLOY_CONFIG_CHANGED = YES/NO

3. FIXTURE GATE

Using the approved ordinary Staging test pair, verify:
- both are ordinary/non-admin
- test_access is null/none
- both belong to the same approved active business customer
- both share the same effective DM entitlement
- seat_limit is sufficient
- assigned user has DM Seat
- unassigned user has no DM Seat

If normalization is required, use only canonical platform_admin RPCs.

Do not direct-write Seat tables.

Return:
DM_FIXTURE_READY = YES/NO

4. DEPLOY DM STAGING ONCE

Deploy only dmruntimestaging using the clean deploy-config candidate.

Do not deploy any other service.

5. POST-DEPLOY IMMUTABLE DIGEST PROOF

Immediately verify from the running deployment:
- running backend digest equals the exact expected backend digest
- running frontend digest equals the exact expected frontend digest
- runtime app source commit equals 2dd02e2378600f02b9d14590391b9edf84bc14a2
- running deploy-config identity equals DM_DEPLOY_CONFIG_COMMIT
- release ID is recorded
- no mutable tag substitution occurred

Required:
POST_DEPLOY_BACKEND_DIGEST_MATCH = YES
POST_DEPLOY_FRONTEND_DIGEST_MATCH = YES
POST_DEPLOY_SOURCE_SHA_MATCH = YES
POST_DEPLOY_CONFIG_MATCH = YES

If all CI and post-deploy checks agree:
REGISTRY_PROVENANCE = PASS_BY_IMMUTABLE_DIGEST_CHAIN

If any mismatch occurs:
rollback DM Staging immediately to the previous known-good release.

6. HEALTH / READINESS / SECURITY

Verify:
- /api/health = 200
- /api/ready = 200 and fast/shallow
- /api/health/detail remains available
- no crash/restart loop
- environment=staging
- Supabase project is Staging
- Central Seat policy is authoritative/require
- no token -> 401
- invalid token -> 401
- auth/RLS verifier PASS
- fail-closed behavior preserved

7. FINAL ORDINARY-USER E2E

Assigned ordinary user:
- Google login succeeds
- DM opens
- protected API succeeds
- no Seat denial

Then fully logout/clear session.

Unassigned ordinary user:
- Google login succeeds
- authentication itself succeeds
- DM commercial access is denied
- backend returns canonical 403 / Seat-not-assigned behavior
- protected DM data/API does not load

If Google OAuth requires operator interaction:
pause only for the login action and continue afterward in the same session.

8. CAUSAL SEAT PROOF

Prove both users share:
- same active business customer
- same effective DM entitlement
- same ordinary/non-admin class
- same test_access semantics

and differ only in:
- DM Central Seat assignment

Required:
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

9. ROLLBACK

On any mandatory artifact, deployment, health, security, or E2E failure:
rollback only DM Staging to the previous known-good release and prove health 200.

10. FINAL REPORT

Return exactly:

RESULT =
PROVENANCE_METHOD = IMMUTABLE_DIGEST_CHAIN
REGISTRY_PROVENANCE =
CI_SOURCE_SHA =
CI_BACKEND_DIGEST =
CI_FRONTEND_DIGEST =
DM_APP_SOURCE_COMMIT =
DM_DEPLOY_CONFIG_COMMIT =
DM_STAGING_RELEASE =
RUNNING_BACKEND_DIGEST =
RUNNING_FRONTEND_DIGEST =
POST_DEPLOY_BACKEND_DIGEST_MATCH =
POST_DEPLOY_FRONTEND_DIGEST_MATCH =
POST_DEPLOY_SOURCE_SHA_MATCH =
POST_DEPLOY_CONFIG_MATCH =
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
