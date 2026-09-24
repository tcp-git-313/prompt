# W3 — STAGING COMMERCIAL FINALIZATION MASTER

ROLE: STAGING COMMERCIAL FINALIZATION OWNER

RECOMMENDED MODEL: GPT-5.6 Sol — High

MODE:
DIAGNOSE ONLY WHAT IS STILL UNKNOWN
→ FIX ONLY PROVEN ROOT CAUSE
→ COMPLETE DM CUTOVER
→ COMPLETE ORC BROWSER E2E
→ COMPLETE LAUNCHER CONTRACT E2E
→ DECLARE STAGING COMMERCIAL FREEZE

WORKSPACE:
F:\00-Ticenpi-SaaS

REPOS:
DM:
F:\00-Ticenpi-SaaS\TicenpiDM

ORC + Letter:
F:\00-Ticenpi-SaaS\TicenpiLetter

Launcher:
F:\00-Ticenpi-SaaS\Ticenpi-Launcher

Platform:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Finish the remaining Staging commercial validation work and reach one unambiguous final state:

POST = PASS
PLATFORM = PASS
DM = PASS
ORC = PASS
LAUNCHER WORKBENCH CONTRACT E2E = PASS
STAGING COMMERCIAL FREEZE = YES

Do not stop merely because one old prompt assumed a condition that is no longer true.
Follow the current verified architecture and evidence below.

======================================================================
A. VERIFIED CURRENT STATE
======================================================================

POST
- Central Seat authoritative Staging E2E already PASS.
- No further Post work in this task.

PLATFORM
- Commercial Admin contracts applied to Staging.
- Central Seat regression PASS.
- member/customer lifecycle PASS.
- rollback rehearsal PASS.
- Production untouched.
- No further Platform schema work unless a new live contract defect is directly proven.

ORC
- Staging Central Seat authoritative implementation is deployed.
- product code = ocr.
- ORC_CENTRAL_SEAT_ENABLED=true.
- entitlement active.
- seat_limit=2.
- assigned and unassigned ordinary users exist.
- health 200.
- invalid/no token 401.
- RLS preserved.
- Letter remains internal component.
- Only remaining ORC gate is real browser assigned/unassigned E2E.
- No ORC code/deploy changes are expected.

LAUNCHER
- There is NO separate Launcher Staging deployment.
- Stable Launcher remains Production-only.
- Canonical local candidate has already passed local Hub/Desktop validation.
- Platform Staging admin contracts are live.
- Remaining Launcher work is local/dev Workbench E2E against Central Supabase Staging.
- Do not create a Staging Launcher environment.

DM
Current healthy Staging remains rolled back to:
release = 20260923-215914
app commit = 25db5f9b8b65a231dafb7cc8f1f05262fb7519aa

Final DM candidate history:
- readiness hardening completed
- Central Seat implementation completed
- extraction-shadow CI blocker fixed
- tests/CI passed
- final app source used in F3 path:
  2dd02e2378600f02b9d14590391b9edf84bc14a2
- prior CI run:
  35968635281
- prior backend digest:
  sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468
- prior frontend digest:
  sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a

F3 attempted authoritative cutover and rolled back safely.
The failure was reported as:
product_seat_status("dm") response rejected as invalid/unexpected format.

Static C1 inspection then proved:
- Platform canonical product_seat_status(text) returns a jsonb object.
- Post successful consumer expects that canonical JSON object.
- DM consumer also appears to expect the same canonical JSON object.
- No static parser mismatch has been proven.
- Therefore DO NOT modify the DM parser until the live/raw response and the F3 acceptance path are compared.

======================================================================
B. GLOBAL RULES
======================================================================

1. STAGING ONLY.
2. DO NOT touch Production.
3. Preserve all unrelated dirty WIP.
4. Use isolated worktrees/branches for source changes.
5. Do not repeat already-passed broad audits.
6. Do not rebuild if no source fix is required.
7. Do not create new Auth test users.
8. Do not direct-write Central Seat tables.
9. Use canonical platform_admin RPCs for any required fixture normalization.
10. Do not use service-role credentials in end-user runtime auth.
11. Do not invent extra DM-only release gates.
12. Immutable digest deployment remains required.
13. Do not require direct GHCR metadata access if the standard CI → exact digest → post-deploy running-digest/source chain passes.
14. A missing UI logout button is NOT a blocker.
15. For assigned/unassigned browser E2E use separate isolated browser contexts/profiles, or clear only the relevant Staging site's auth/session storage between accounts.
16. Do not clear unrelated browser data.
17. Pause for human action only when Google OAuth truly requires the operator.
18. After the operator completes login, continue from the same preserved task/session automatically.

======================================================================
C. PHASE 1 — DM LIVE RESPONSE ROOT CAUSE
======================================================================

Use the currently available DM Staging login/session if authenticated.

Do not force logout/re-login if a valid ordinary Staging session already exists.

First obtain the exact live product_seat_status("dm") response using the caller JWT.

Capture without mutation:
- HTTP status
- Content-Type
- raw response body
- JSON-decoded top-level type:
  object / array / scalar / null
- exact keys
- state
- reason
- product_code
- any wrapper shape introduced by the HTTP/Supabase client

Also inspect the exact F3 acceptance/gate implementation that classified the response as invalid.

Compare four things:
1. live raw RPC response
2. Platform canonical SQL contract
3. Post successful consumer
4. DM runtime consumer + F3 acceptance/gate

Classify exactly one primary root cause:

A. F3 acceptance/gate script incorrectly rejected a valid canonical response
B. DM runtime consumer incorrectly rejected a valid canonical response
C. live Staging Platform RPC shape differs from canonical contract
D. client wrapper produced a shape not handled by the runtime/gate
E. another directly proven cause

Return:
DM_LIVE_RPC_HTTP_STATUS =
DM_LIVE_RPC_RAW_BODY =
DM_LIVE_RPC_TOP_LEVEL_TYPE =
DM_LIVE_RPC_STATE =
DM_LIVE_RPC_REASON =
DM_LIVE_RPC_PRODUCT_CODE =
DM_ROOT_CAUSE_CLASS =
DM_ROOT_CAUSE =

IMPORTANT:
Do not modify code before this evidence exists.

======================================================================
D. PHASE 2 — DM MINIMAL REMEDIATION
======================================================================

CASE A — acceptance/gate bug only:
- Fix only the F3 acceptance/gate logic.
- Do NOT rebuild DM application images.
- Reuse the already-verified application artifacts if still valid.

CASE B or D — DM runtime/client compatibility bug:
- Make the smallest fix in the DM Central Seat consumer/client.
- Preserve strict canonical validation and fail-closed behavior.
- Add focused regression for the exact live response shape.
- Run Seat/auth/readiness/extraction/RLS/API regression.
- Run normal CI.
- Capture new source SHA + backend/frontend immutable digests.
- Do not deploy until CI is green.

CASE C — live Platform drift:
- Prove the exact Staging drift.
- Repair only the canonical Platform contract using the existing Platform migration/security conventions.
- Re-run Platform Central Seat regression.
- Then re-test the DM live RPC.
- Do not change DM parser to accommodate an incorrect Platform contract.

CASE E:
- Make only the directly evidenced minimal correction.

Required after remediation:
DM_CONTRACT_ROOT_CAUSE_RESOLVED = YES

======================================================================
E. PHASE 3 — DM STANDARD COMMERCIAL CUTOVER
======================================================================

Before deploy, verify live DM fixture using approved ordinary test identities.

Resolve the already-approved Staging ordinary regression pair from existing evidence.
Do not infer from email ordering.

Verify both:
- exist
- platform_role=user
- test_access null/none
- non-admin
- same active business customer

Verify live DM commercial state:
- DM product exists
- active effective DM entitlement exists
- seat_limit >= 2
- assigned test user HAS DM Seat
- unassigned test user DOES NOT have DM Seat

If not ready:
normalize only through canonical platform_admin RPCs.

If admin Google OAuth is required:
pause only for that login, then continue.

Required:
DM_FIXTURE_READY = YES

Create a clean deploy-config candidate.

Authorized DM cutover config:
- backend immutable digest = current final verified candidate digest
- frontend immutable digest = current final verified candidate digest
- DM_SEAT_POLICY=require

No unrelated config changes.

Commit only this deploy-config delta.

Deploy ONLY DM Staging once.

Post-deploy verify:
- running backend digest exact match
- running frontend digest exact match
- runtime app source SHA exact match
- deploy-config identity exact match
- DM_SEAT_POLICY=require
- environment=staging
- Supabase project=jlsqjvehwblkeuycjoyj
- /api/health=200
- /api/ready=200 and shallow/fast
- deep health preserved
- no restart/crash loop
- no token=401
- invalid token=401
- auth/RLS verifier PASS
- fail-closed behavior preserved

Artifact proof standard:
CI source SHA
→ CI immutable digests
→ deploy-config exact digest pins
→ post-deploy running digests
→ runtime source SHA

All must agree.

Return:
DM_ARTIFACT_PROVENANCE = PASS_BY_IMMUTABLE_DIGEST_CHAIN

Any mismatch:
rollback DM Staging to previous known-good release.

======================================================================
F. PHASE 4 — DM REAL USER E2E
======================================================================

Do not require a product logout button.

Use two isolated browser contexts/profiles.

CONTEXT A — assigned ordinary user
- Google login succeeds
- DM opens
- protected API succeeds
- no Seat denial

Return:
DM_ASSIGNED_ALLOW = PASS/FAIL

CONTEXT B — unassigned ordinary user
- Google login succeeds
- authentication succeeds
- DM commercial access denied
- canonical 403 / Seat-not-assigned behavior
- protected DM data/API does not load

Return:
DM_UNASSIGNED_DENY = PASS/FAIL

Prove both users share:
- same active business customer
- same effective DM entitlement
- same ordinary/non-admin class
- same test_access semantics

and differ only in:
- DM Seat assignment

Required:
DM_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES

When complete:
DM_CENTRAL_SEAT_STAGING_CUTOVER = PASS

======================================================================
G. PHASE 5 — ORC FINAL BROWSER E2E
======================================================================

Do not modify ORC code, DB fixture, or deployment.

Use isolated browser contexts/profiles.

Assigned ORC account:
- login succeeds
- ORC opens
- protected OCR API succeeds
- Letter internal component remains available

Unassigned ORC account:
- login succeeds
- ORC commercial access denied by Seat
- protected API/data not loaded
- Letter is not an authorization bypass

Do not rely on a logout button.
Use a separate browser context/profile or clear only ORC Staging site auth/session storage.

Required:
ORC_ASSIGNED_ALLOW = PASS
ORC_UNASSIGNED_DENY = PASS
ORC_CENTRAL_SEAT_CAUSAL_E2E_PROVEN = YES
ORC_CENTRAL_SEAT_STAGING_CUTOVER = PASS

======================================================================
H. PHASE 6 — LAUNCHER WORKBENCH CONTRACT E2E
======================================================================

IMPORTANT:
DO NOT deploy Launcher Staging.
DO NOT create a Staging Launcher build.
DO NOT switch stable Production Launcher to Staging.

Use the canonical local/dev Launcher candidate against Central Supabase Staging only.

Use existing platform_admin Staging session if already authenticated.
Do not force logout/re-login unnecessarily.

Before mutations:
snapshot the test Customer B state:
- customer status
- members/status
- entitlements
- seat_limit
- product Seat assignments

Then run through the Workbench UI:

1. list/search Customer B
2. verify member list uses canonical stable user_id
3. assign an eligible Seat
4. verify count/list
5. release that Seat
6. assign a Seat to the lifecycle-test member
7. suspend member
8. verify Seat auto-release
9. reactivate member
10. verify Seat does NOT auto-restore
11. suspend Customer B
12. verify commercial access denies
13. reactivate Customer B
14. verify status restores without inventing Seat assignments
15. verify product isolation across post/dm/ocr
16. restore Customer B to the pre-test snapshot through canonical RPC/UI flows

Required:
LAUNCHER_WORKBENCH_STAGING_CONTRACT_VALIDATED = YES
LAUNCHER_STAGING_DEPLOYED = NO
STABLE_PRODUCTION_LAUNCHER_CHANGED = NO

======================================================================
I. FINAL STAGING COMMERCIAL FREEZE
======================================================================

Final freeze is allowed only when:

POST = PASS
PLATFORM = PASS
DM = PASS
ORC = PASS
LAUNCHER_WORKBENCH_STAGING_CONTRACT_VALIDATED = YES

Then output:

RESULT = PASS
POST_STAGING_COMMERCIAL = PASS
PLATFORM_STAGING_COMMERCIAL = PASS
DM_CENTRAL_SEAT_STAGING_CUTOVER = PASS
ORC_CENTRAL_SEAT_STAGING_CUTOVER = PASS
LAUNCHER_WORKBENCH_STAGING_CONTRACT_VALIDATED = YES
STAGING_COMMERCIAL_FREEZE = YES
PRODUCTION_MUTATED = NO

Also freeze and report exact identities needed for the later Production phase:
- Platform contract commit/migration identity
- DM app source commit
- DM backend/frontend digests
- DM deploy-config commit
- ORC app/deploy identity
- Launcher canonical candidate commit

Do not start Production cutover in this task.

If a human Google OAuth action is the ONLY remaining item:
RESULT = PARTIAL_PASS
MANUAL_LOGIN_REQUIRED = YES
STATE_TO_RESUME_FROM = exact phase/role
STAGING_COMMERCIAL_FREEZE = NO

If a technical blocker remains:
RESULT = BLOCKED
BLOCKER = exact proven blocker
NEXT_ACTION = smallest direct action

Do not create another broad audit.
