# W1-DM-R3 — DM Staging Release Identity Reconciliation (Read-Only)

ROLE: DM STAGING RELEASE IDENTITY AUDIT OWNER

MODE: READ-ONLY / NO SOURCE CHANGES / NO DEPLOY / NO DB MUTATION

WORKSPACE:
F:\00-Ticenpi-SaaS

DM REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

DEPLOY REPO:
F:\00-Ticenpi-SaaS\deploy

PLATFORM REPO:
F:\00-Ticenpi-SaaS\ticenpi-platform

GOAL:
Reconcile the current DM Staging release identity mismatch and produce one exact, deploy-safe identity model for the next authoritative Central Seat candidate.

Do NOT modify files.
Do NOT build images.
Do NOT commit/push.
Do NOT deploy.
Do NOT restart containers.
Do NOT mutate Staging/Production.
Do NOT change Central Seat fixtures.

## VERIFIED CURRENT EVIDENCE

Current DM Staging release:
20260923-215914

Current public/runtime health commit:
25db5f9b8b65a231dafb7cc8f1f05262fb7519aa

Running backend digest:
sha256:51efc74c75e35d0d857434e3f113bb0546c9072b2b12398f4c33b1555ffb0e83

Running frontend digest:
sha256:0842f90d476ab7fa4b74d1c70781cc5bd1b1a7513848e926e5f5c9c148230093

Deployment identity source.gitSha:
34a59a7fb50c

Current repo deploy/runtime-staging/compose.yml pins older digests:
backend:
sha256:c4f4bf32a9faf277f06f90d1c104479d08d7293cc77d44369e433e18f0c7ea9e

frontend:
sha256:c3d9473e8f4142b4acbd03211ec842a239a4f97125ce0428bd52c9314dc0d8b1

Those pinned digests match prior known-good release:
20260922-131327

Current runtime:
- environment=staging
- Supabase project=jlsqjvehwblkeuycjoyj
- DM_REQUIRE_AUTH=1
- TICENPI_COMMERCIAL_GATE_ENABLED is currently unset/default disabled
- /api/ready can return 200 but has known cold-cache Gunicorn timeout risk

## 1. PREFLIGHT

Record:
- DM main checkout branch/HEAD/status
- relevant release worktrees if any
- deploy repo branch/HEAD/status
- active Staging release
- current compose source
- current deployment identity files

Preserve all dirty WIP.

## 2. PROVE ACTUAL IMAGE SOURCE

For each running digest:
- locate trusted CI / registry / release evidence
- identify exact source commit used to build it
- identify workflow run ID if available
- identify image labels
- identify whether backend/frontend came from same source commit

Return:

RUNNING_BACKEND_SOURCE_COMMIT =
RUNNING_FRONTEND_SOURCE_COMMIT =
RUNNING_BACKEND_BUILD_RUN =
RUNNING_FRONTEND_BUILD_RUN =
RUNNING_ARTIFACT_PROVEN = YES/NO

Do not infer from runtime env alone.

## 3. EXPLAIN COMMIT IDENTITY SPLIT

Trace why these differ:

- runtime health commit = 25db5f9...
- deployment identity source.gitSha = 34a59a7...
- DM repo current HEAD = 561bd8c...
- running image source commit = <proven result>

Classify each field:
- APP_SOURCE_COMMIT
- DEPLOY_SOURCE_COMMIT
- RUNTIME_ENV_COMMIT
- REPO_CURRENT_HEAD

Determine whether current health is mislabelled, deploy metadata is stale, or the deployment intentionally combines different identities.

Return:

COMMIT_IDENTITY_ROOT_CAUSE =
HEALTH_COMMIT_TRUSTWORTHY = YES/NO/PARTIAL
DEPLOY_IDENTITY_TRUSTWORTHY = YES/NO/PARTIAL

## 4. EXPLAIN COMPOSE PIN DRIFT

Trace exactly when and why deploy/runtime-staging/compose.yml moved back to the prior release digests while the live runtime remained on newer images.

Use git history / release records / deployment manifests.

Return:

COMPOSE_PIN_DRIFT_ROOT_CAUSE =
CURRENT_COMPOSE_MATCHES_RUNNING = NO
CURRENT_COMPOSE_MATCHES_ROLLBACK_TARGET = YES/NO

Do not edit compose.

## 5. DEFINE THE NEXT DEPLOY-SAFE IDENTITY CONTRACT

Without implementation, define the exact fields that the next DM authoritative candidate must freeze:

APP_SOURCE_COMMIT
BACKEND_IMAGE_DIGEST
FRONTEND_IMAGE_DIGEST
DEPLOY_CONFIG_COMMIT
RELEASE_ID
RUNTIME_HEALTH_COMMIT

State which must be equal and which may intentionally differ.

If DM should adopt the split app-artifact/deploy-config identity pattern already used successfully by Post, say so and identify the minimum DM deployment tooling changes required.

Do not implement.

## 6. COMMERCIAL GATE DELTA

Read-only identify the exact code/config path controlling:
TICENPI_COMMERCIAL_GATE_ENABLED

Determine:
- default
- Staging desired value for DM Central Seat authoritative cutover
- whether DM require_dm_access currently depends on it
- whether enabling it needs image rebuild or only config change

Return:

COMMERCIAL_GATE_CURRENT =
COMMERCIAL_GATE_REQUIRED_FOR_DM_F1 =
COMMERCIAL_GATE_CHANGE_TYPE = config-only / image-required / unknown

## 7. FINAL NEXT-CANDIDATE SPEC

Produce a deterministic candidate specification for W1-DM-F1R:

BASE_SOURCE_COMMIT =
REQUIRED_SEAT_SHADOW_SOURCE =
REQUIRED_AUTH_SOURCE =
BACKEND_DIGEST_STRATEGY =
FRONTEND_DIGEST_STRATEGY =
DEPLOY_CONFIG_STRATEGY =
COMMERCIAL_GATE_TARGET =
ROLLBACK_RELEASE = 20260922-131327

List exact files/symbols that the implementation task is allowed to touch.

List DO_NOT_TOUCH.

## 8. FINAL REPORT

A. PREFLIGHT
B. RUNNING ARTIFACT PROVENANCE
C. COMMIT IDENTITY SPLIT
D. COMPOSE PIN DRIFT
E. DEPLOY-SAFE IDENTITY CONTRACT
F. COMMERCIAL GATE DELTA
G. NEXT CANDIDATE SPEC
H. SAFETY

SOURCE_FILES_MODIFIED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
DEPLOYED = NO
COMMIT_CREATED = NO
PUSHED = NO

I. FINAL RESULT

RESULT = PASS/BLOCKED

If proven:
DM_RELEASE_IDENTITY_RECONCILED = YES
NEXT_DM_CANDIDATE_SPEC_COMPLETE = YES
READY_FOR_DM_F1R = YES

If not:
DM_RELEASE_IDENTITY_RECONCILED = NO
READY_FOR_DM_F1R = NO
BLOCKER = exact missing evidence

Finish and stop.
