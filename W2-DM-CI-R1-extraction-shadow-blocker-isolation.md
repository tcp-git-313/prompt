# W2-DM-CI-R1 — DM Extraction Shadow CI Blocker Isolation

ROLE: DM FINAL-CANDIDATE CI BLOCKER OWNER

MODE: DIAGNOSE + MINIMAL FIX + CI ONLY
NO DEPLOY

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

FINAL-CANDIDATE SOURCE COMMIT:
94c5c9c6e7b6fb63f96389ac9f4180db78d4a45b

READINESS HARDENING COMMIT:
67508d1eb290556d1bb33ea9f6218dd12974438f

FAILED CI RUN:
35964746084

GOAL:
Isolate the existing extraction-shadow CI failure that prevented immutable DM artifacts from being produced, fix only the proven blocker if necessary, rerun CI, and return exact verified image digests for resuming W2-DM-F1R.

DO NOT redo DM Seat/readiness/auth work.

VERIFIED CURRENT STATE:
- DM Central Seat require implementation completed locally
- focused Seat tests PASS
- auth/health/API/RLS verification PASS
- verify_auth_rls.py PASS
- Production untouched
- Staging not deployed
- no deploy-config pin exists because CI did not produce verified artifacts
- failure is in extraction shadow test area
- current Staging remains old release and healthy

ABSOLUTE RULES:
- Do NOT deploy.
- Do NOT mutate Staging or Production.
- Do NOT change Seat semantics.
- Do NOT change readiness semantics.
- Do NOT touch OAuth, YUCT, Launcher, ORC, Platform, or unrelated DM WIP.
- Do NOT disable or skip the failing test merely to make CI green.
- Do NOT weaken extraction safety contracts.
- Preserve exact final-candidate scope.

1. REPRODUCE EXACT FAILURE

Check out the exact candidate commit/worktree corresponding to:
94c5c9c6e7b6fb63f96389ac9f4180db78d4a45b

Reproduce the exact failing extraction-shadow test from CI run 35964746084 locally if possible.

Record:
FAILED_TEST =
FAILURE_MESSAGE =
FAILURE_PHASE =
REPRODUCIBLE_LOCALLY = YES/NO

2. BASELINE COMPARISON

Run the same failing test against:
A. the pre-final-candidate DM baseline
B. the final-candidate commit

Determine whether the failure is:
- pre-existing baseline
- caused by Seat integration
- caused by readiness integration
- caused by stale fixture/expectation
- caused by extraction-shadow contract drift
- caused by CI/environment only

Return:
FAILURE_CLASS =
CANDIDATE_INDUCED = YES/NO
ROOT_CAUSE =

Do not assume "pre-existing" without reproducing evidence.

3. MINIMAL FIX

If candidate-induced:
fix the smallest exact cause.

If baseline/pre-existing but CI now blocks artifact production:
determine the correct minimal contract-preserving repair needed for CI to represent the intended extraction-shadow behavior.

Allowed change scope:
- extraction shadow implementation
- extraction shadow tests/fixtures
- minimal shared helper directly required

Do not modify Seat/readiness/auth behavior unless root-cause evidence proves that is necessary.

Do not skip, xfail, delete, or loosen assertions without a documented contract reason.

4. REGRESSION

Run:
- failing extraction-shadow test
- full extraction-shadow suite
- DM focused Seat suite
- readiness suite
- auth/RLS verifier
- relevant API suite
- git diff --check

Required:
SEAT_REGRESSION = PASS
READINESS_REGRESSION = PASS
AUTH_RLS_REGRESSION = PASS
EXTRACTION_SHADOW = PASS

5. COMMIT

Create one minimal isolated repair commit if code/test change is required.

Return:
BLOCKER_FIX_COMMIT =

If no source change is required and rerun succeeds due confirmed transient CI condition:
BLOCKER_FIX_COMMIT = NONE
Explain exact evidence.

6. CI RERUN

Run normal DM CI for the exact repaired final candidate.

Do not deploy.

Required:
CI = PASS

7. IMMUTABLE ARTIFACT EVIDENCE

Only after CI is fully green, capture:
DM_APP_SOURCE_COMMIT
BACKEND_IMAGE_DIGEST
FRONTEND_IMAGE_DIGEST
BUILD_RUN_ID

Cross-check digests against CI artifact/image labels.

Do not create deploy-config pin in this task unless the existing workflow automatically does so.
Do not deploy.

8. FINAL REPORT

Return:
RESULT =
FAILED_TEST =
FAILURE_CLASS =
CANDIDATE_INDUCED =
ROOT_CAUSE =
BLOCKER_FIX_COMMIT =
EXTRACTION_SHADOW =
SEAT_REGRESSION =
READINESS_REGRESSION =
AUTH_RLS_REGRESSION =
CI =
DM_APP_SOURCE_COMMIT =
BACKEND_IMAGE_DIGEST =
FRONTEND_IMAGE_DIGEST =
BUILD_RUN_ID =
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO

If CI/artifacts succeed:
READY_TO_RESUME_DM_F1R_FROM_DEPLOY_CONFIG_PIN = YES

If still blocked:
READY_TO_RESUME_DM_F1R_FROM_DEPLOY_CONFIG_PIN = NO
BLOCKER = exact remaining issue
