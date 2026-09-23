# P4A-D1 — Reconcile Post Staging Image Digest & Complete Final Central Seat Cutover

ROLE: POST STAGING ARTIFACT RECONCILIATION + FINAL CUTOVER OWNER

MODE: STAGING ONLY / EXACT ARTIFACT RECONCILIATION / FINAL E2E

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Resolve the final Post Staging blocker by making the Staging compose/release identity reference the exact CI-verified image digest for the already-green authoritative candidate, then deploy that exact artifact, switch Staging to Central Seat authoritative mode, and run final E2E.

DO NOT touch Production.
DO NOT rebuild application code unless absolutely required by the repository's release process.
DO NOT alter Central Seat fixture state except for verification.
DO NOT recreate test users.
DO NOT change Central Seat SQL/RPC semantics.

## VERIFIED CURRENT STATE

Central Seat fixture is READY:

FIXED_STAGING_TEST_FIXTURE_READY = YES
POST_TEST_ASSIGNED_SEAT = YES
POST_TEST_UNASSIGNED_SEAT = NO
POST_TEST_ASSIGNED_TENANT_MAPPING = YES
POST_TEST_UNASSIGNED_TENANT_MAPPING = YES

Test customer:
5b3b204b-25a5-411b-aacb-ca8b4c617482

Authoritative candidate commit:
27e4efe

CI:
GREEN

CI-verified image digest:
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Current active Staging runtime image:
sha256:42974599edc1c40bcd298206ec112cfb5f469a56155c0a3e04410f8e952d5cd6

Current candidate Staging compose incorrectly pins:
sha256:6af118f63a3da6336a016010817aa3980382ceb8c0fc2b28f400a717f8b2abdb

Current Staging runtime policy:
TICENPI_COMMERCIAL_GATE=1
TICENPI_AUTO_TENANT_BOOTSTRAP=1
TICENPI_SEAT_POLICY=skip
TICENPI_ALLOW_TENANT_KEY=0

Production:
UNCHANGED

## 1. PREFLIGHT

Record:
- branch
- HEAD
- git status --short
- git diff --stat
- exact Staging compose file/path
- current compose image reference
- current active Staging release/image/commit

Do not reset/clean/stash/rebase.

## 2. VERIFY THE CI ARTIFACT IDENTITY

Using existing GitHub/CI evidence and repository release metadata, prove:

CANDIDATE_COMMIT = 27e4efe

CI_IMAGE_DIGEST =
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Verify the digest belongs to the Post image produced from that exact candidate commit.

Do not trust a copied value without checking the existing CI/release evidence.

Required:
CI_ARTIFACT_IDENTITY_PROVEN = YES

If NO:
STOP.

## 3. INSPECT WHY COMPOSE PINS THE WRONG DIGEST

Determine exactly why Staging compose currently references:

sha256:6af118f63a3da6336a016010817aa3980382ceb8c0fc2b28f400a717f8b2abdb

Classify:

- stale prior release pin
- generated file not refreshed
- candidate branch missing release identity update
- deployment manifest drift
- other exact cause

Do not guess.

Report:
DIGEST_MISMATCH_ROOT_CAUSE =

## 4. CREATE THE MINIMAL ARTIFACT-IDENTITY FIX

Change ONLY the Staging deployment identity/config required to make the authoritative candidate reference the exact CI-verified digest.

Preferred scope:
deploy/runtime-staging/compose.yml

Expected new image digest:
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Do NOT modify:
- application auth logic
- Central Seat logic
- Post tenant logic
- Production compose/config
- unrelated runtime services

If another generated release-identity file must change by repository convention, include only that required file and explain why.

## 5. VERIFY CONFIG SEMANTICS

Before commit prove:

- Post Staging service only is affected
- port remains 19419
- Production image reference unchanged
- no other service digest changed
- no unrelated env drift
- current desired formal flags are represented correctly for final cutover

Required final Staging effective intent:

TICENPI_COMMERCIAL_GATE=1
TICENPI_SEAT_POLICY=require
TICENPI_ALLOW_TENANT_KEY=0

TICENPI_AUTO_TENANT_BOOTSTRAP may remain 1.

If SEAT_POLICY=require is not yet represented in the Staging runtime configuration and the repository requires compose/env versioning, include that exact Staging-only config change in this candidate.

Do not touch Production.

## 6. LOCAL / STATIC VALIDATION

Run:
- compose syntax/config validation
- repository deployment preflight
- git diff --check
- targeted Post auth/Seat tests if any config-coupled tests exist
- runtime-staging contract tests, including test_tenant_bootstrap_flag_contract.py if applicable

Confirm:
- candidate app commit remains 27e4efe or clearly document if a config-only child commit is required
- application code is unchanged
- exact image digest remains d6a0c592...

## 7. CREATE CONFIG-ONLY RELEASE CANDIDATE

If repository release process requires GitHub commit/CI for the compose correction:

Create a dedicated Staging release candidate branch from the already-approved authoritative candidate ancestry.

Do not rewrite history.

Commit only the minimal config/release identity files.

Suggested commit message:
chore(post): pin staging to verified central seat candidate image

Push dedicated branch.
Run required CI/checks.

Required:
- green CI
- immutable config candidate SHA
- app image digest still d6a0c592...
- no application rebuild to a different digest unless the release system necessarily rebuilds and produces a new verified digest from identical app source

If CI rebuilds and emits a NEW digest:
that new digest becomes authoritative only if:
- exact source identity is proven
- CI is green
- compose is updated to that exact new digest
- there is no recursive mismatch

Do not deploy until:
COMPOSE_DIGEST == VERIFIED_CI_DIGEST

## 8. FINAL DEPLOY GATE

Required before deployment:

FIXED_STAGING_TEST_FIXTURE_READY = YES
CI_ARTIFACT_IDENTITY_PROVEN = YES
COMPOSE_DIGEST_MATCHES_CI = YES
STAGING_ONLY_CONFIG = YES
PRODUCTION_UNCHANGED = YES

If any is NO:
STOP.

## 9. DEPLOY POST STAGING ONLY

Use the approved single-service Post Staging deploy chain.

Record:
PREVIOUS_RELEASE =
NEW_RELEASE =
DEPLOYED_COMMIT =
DEPLOYED_IMAGE_DIGEST =
DEPLOY_AUDIT =

Required:
DEPLOYED_IMAGE_DIGEST == VERIFIED_CI_DIGEST

No other service may change.

## 10. VERIFY LIVE RUNTIME FLAGS

Read actual running container environment.

Required:

TICENPI_COMMERCIAL_GATE = 1
TICENPI_SEAT_POLICY = require
TICENPI_ALLOW_TENANT_KEY = 0

Also record:
TICENPI_AUTO_TENANT_BOOTSTRAP =
TICENPI_CENTRAL_SEAT_SHADOW_ENABLED =

Prove:
SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP = YES

## 11. FINAL LIVE E2E

Use the already-configured fixed Staging regression identities.

### Admin
Expected:
ALLOW

### Assigned ordinary user
Expected:
- active membership
- effective Post entitlement
- Central Post Seat assigned
- valid Post tenant mapping
- final ALLOW

### Unassigned ordinary user
Expected:
- active membership
- effective Post entitlement
- no Central Post Seat
- valid Post tenant mapping
- final 403

### Invalid JWT
Expected:
401

Do not use X-Tenant-Key.

If browser credentials/session are unavailable for ordinary users:
do not alter fixture.
Return PARTIAL_PASS with exact manual login steps only.

## 12. CENTRAL SEAT CAUSAL PROOF

Prove the assigned and unassigned ordinary users have equivalent:

- customer membership
- Post entitlement eligibility
- Post tenant mapping validity

and differ in:
Central Post Seat assignment

Required:
CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

## 13. HEALTH / SECURITY

Verify:
- internal /api/health 200
- external Staging 200
- correct release/commit/digest
- no crash loop
- no auth secret leakage

AUTH_LOG_SECRET_LEAK = NO

## 14. ROLLBACK

If mandatory deploy/E2E fails:
- rollback Post Staging only
- restore prior Staging config
- keep Central Seat fixture intact
- verify previous release and health 200

## 15. FINAL REPORT

A. PREFLIGHT

B. CI ARTIFACT IDENTITY
CANDIDATE_COMMIT =
VERIFIED_CI_DIGEST =
CI_ARTIFACT_IDENTITY_PROVEN =

C. DIGEST MISMATCH ROOT CAUSE
DIGEST_MISMATCH_ROOT_CAUSE =

D. CONFIG FIX
FILES_CHANGED =
COMPOSE_DIGEST =
SEAT_POLICY_INTENT =

E. RELEASE CANDIDATE
CONFIG_CANDIDATE_BRANCH =
CONFIG_CANDIDATE_COMMIT =
CI_STATUS =

F. DEPLOY
PREVIOUS_RELEASE =
NEW_RELEASE =
DEPLOYED_COMMIT =
DEPLOYED_IMAGE_DIGEST =
DEPLOY_AUDIT =

G. RUNTIME FLAGS
COMMERCIAL_GATE =
SEAT_POLICY =
ALLOW_TENANT_KEY =
AUTO_TENANT_BOOTSTRAP =
CENTRAL_SEAT_SHADOW_ENABLED =
SEAT_ENFORCEMENT_DECOUPLED_FROM_AUTO_BOOTSTRAP =

H. FINAL E2E
Case | Seat | Tenant mapping | Final result | PASS/NOT_REACHED

I. CAUSAL PROOF
CENTRAL_SEAT_CAUSAL_E2E_PROVEN =

J. HEALTH / SECURITY
HEALTH_200 =
AUTH_LOG_SECRET_LEAK =
CRASH_LOOP =

K. PRODUCTION
PRODUCTION_MUTATED = NO

L. FINAL RESULT

If complete:

RESULT = PASS
COMPOSE_DIGEST_MATCHES_CI = YES
POST_CENTRAL_SEAT_STAGING_CUTOVER = PASS
CENTRAL_AUTHORITATIVE = YES
BUSINESS_ASSIGNED_ALLOW = PASS
BUSINESS_UNASSIGNED_DENY = PASS
CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES
READY_FOR_POST_PRODUCTION_CUTOVER = YES

If deploy succeeds but manual B/C login is still required:

RESULT = PARTIAL_PASS
COMPOSE_DIGEST_MATCHES_CI = YES
POST_CENTRAL_SEAT_STAGING_CUTOVER = PENDING_MANUAL_LOGIN_E2E
MANUAL_LOGIN_E2E_REQUIRED = YES
READY_FOR_POST_PRODUCTION_CUTOVER = NO

If exact artifact identity cannot be reconciled:

RESULT = HARD_STOP
COMPOSE_DIGEST_MATCHES_CI = NO
BLOCKER = exact factual mismatch

Do not deploy Production.
