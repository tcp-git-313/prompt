# P4A-D2 — Break Post Staging Image-Digest Recursion with Separate Artifact and Deploy Config Identity

ROLE: POST RELEASE PIPELINE / ARTIFACT IDENTITY OWNER

MODE: CI + DEPLOY CONTRACT REPAIR ONLY

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Fix the release architecture that currently makes a config-only commit rebuild the Docker image and therefore changes the digest every time the Staging compose pin is updated.

The desired model is:

APP ARTIFACT IDENTITY
= immutable application source commit that produced the image

DEPLOY CONFIG IDENTITY
= separate commit that only pins the already-verified image digest and Staging runtime policy

A config-only deployment commit MUST NOT rebuild the application image.

This task must repair that contract, validate it, and produce a deployable Staging candidate.
Do NOT deploy Staging in this task.
Do NOT touch Production.

## VERIFIED CURRENT STATE

Authoritative application candidate:
APP_SOURCE_COMMIT = 27e4efe

Its verified CI image digest:
APP_IMAGE_DIGEST =
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Config candidate previously created:
CONFIG_COMMIT =
319c60adc1ebc33df4a94225b1ec37f468b30089

That config candidate pinned d6a0..., but CI rebuilt an image because every push rebuilds and labels image with commit/run metadata.

CI then emitted:
sha256:c03f6f9c...

This creates recursive mismatch:
config commit pins prior digest
→ push triggers rebuild
→ new digest
→ updating pin triggers another rebuild
→ repeat forever

Current Staging remains:
release 20260923-183949
SEAT_POLICY=skip
ALLOW_TENANT_KEY=0
healthy

Central Seat fixture is already ready:
FIXED_STAGING_TEST_FIXTURE_READY = YES

Production:
UNCHANGED

## ABSOLUTE RULES

- No Staging deploy in this task.
- No Production changes.
- Do not rebuild application code unnecessarily.
- Do not change Central Seat logic.
- Do not change Post auth/Seat semantics.
- Do not rewrite git history.
- Do not force push.
- Do not weaken provenance validation.
- Do not accept an arbitrary external digest.
- Do not make deploy trust mutable tags.

## 1. PREFLIGHT

Record:

- branch
- HEAD
- tracking branch
- git status --short
- relevant CI workflow files
- Dockerfile
- deploy/runtime-staging/compose.yml
- deploy scripts that verify release identity
- release evidence / artifact metadata scripts

Preserve unrelated dirty work.

## 2. DOCUMENT THE CURRENT IDENTITY COUPLING

Trace exactly how these are currently coupled:

A. Git commit SHA
B. GitHub Actions run ID
C. Docker image labels
D. Docker image digest
E. Staging compose image pin
F. deploy preflight expected commit/digest
G. runtime health/release identity

Show the current flow:

commit
→ CI build
→ image labels include commit/run
→ digest
→ compose pin
→ new commit
→ CI rebuild
→ new digest

Set:

RECURSIVE_DIGEST_CAUSE_PROVEN = YES/NO

Do not proceed unless YES.

## 3. DEFINE THE NEW TWO-IDENTITY CONTRACT

Implement this model:

### Application Artifact Identity

APP_SOURCE_COMMIT
APP_IMAGE_DIGEST
APP_BUILD_RUN_ID

The image digest is immutable and tied to the exact application source commit that built it.

### Deploy Config Identity

DEPLOY_CONFIG_COMMIT

This commit may change only deployment/configuration metadata such as:

- Staging compose digest pin
- Staging-only runtime policy
- release manifest metadata

It MUST NOT cause a new application image build when no image-affecting source changed.

### Runtime must expose both

At minimum the deployment/audit path must be able to report:

APP_SOURCE_COMMIT
APP_IMAGE_DIGEST
DEPLOY_CONFIG_COMMIT

If current /api/health cannot expose both without application rebuild, keep runtime app health identity as APP_SOURCE_COMMIT and record DEPLOY_CONFIG_COMMIT in release/deploy metadata instead.

Do not require application code changes merely to expose config commit.

## 4. DETERMINE IMAGE-AFFECTING PATHS

Inspect actual build inputs.

Classify repository paths into:

IMAGE_AFFECTING
DEPLOY_CONFIG_ONLY
TEST_ONLY
DOC_ONLY

IMAGE_AFFECTING should include actual application source/build inputs, for example where appropriate:
- backend source
- frontend source
- dependency manifests/locks
- Dockerfile
- build scripts copied into image

DEPLOY_CONFIG_ONLY should include, where architecture confirms:
- deploy/runtime-staging/compose.yml
- Staging-only release manifest/config
- deployment documentation not copied into image

Do not invent path filters without verifying Docker build context/COPY behavior.

## 5. REPAIR CI TRIGGERING

Modify CI so a config-only commit that changes only DEPLOY_CONFIG_ONLY files:

- does NOT rebuild/publish the application image
- DOES run appropriate validation:
  - compose syntax/config
  - deploy preflight
  - config contract tests
  - provenance check
  - relevant security checks

Application source changes must continue to run full build/image CI.

Preferred design:
separate jobs/workflows:

A. app-build
runs only when IMAGE_AFFECTING inputs change or when explicitly dispatched for a chosen app source commit

B. deploy-config-validation
runs for deploy config changes
does not build/push image

Do not disable required security checks merely for speed.

## 6. REPAIR DEPLOY PROVENANCE CONTRACT

Current deploy gate apparently assumes:
DEPLOY_CONFIG_COMMIT == IMAGE_SOURCE_COMMIT

That assumption must be replaced with a stronger but correct rule:

The deploy config commit may pin an image built from a DIFFERENT verified app commit, provided all are proven:

1. digest is immutable
2. digest exists in trusted GHCR/release evidence
3. release evidence proves the digest was built from APP_SOURCE_COMMIT
4. APP_SOURCE_COMMIT CI is green
5. deploy config pins that exact digest
6. deploy config CI/validation is green
7. Staging-only scope is proven
8. Production config is unchanged

Required relationship:

COMPOSE_DIGEST == VERIFIED_APP_IMAGE_DIGEST

NOT:

DEPLOY_CONFIG_COMMIT == APP_SOURCE_COMMIT

Record both identities.

## 7. VERIFY EXISTING APP ARTIFACT

Use existing CI/release evidence to prove:

APP_SOURCE_COMMIT = 27e4efe
APP_IMAGE_DIGEST =
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Do not rebuild if existing trusted evidence is sufficient.

Required:

APP_ARTIFACT_PROVEN = YES

## 8. CREATE CLEAN STAGING CONFIG CANDIDATE

Use the verified app digest above.

Staging compose must pin exactly:

sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

Staging policy intent:

TICENPI_COMMERCIAL_GATE=1
TICENPI_SEAT_POLICY=require
TICENPI_ALLOW_TENANT_KEY=0

AUTO_TENANT_BOOTSTRAP may remain 1.

Do not touch Production compose/config.

Create or update a dedicated config candidate branch.

Commit only:
- CI/deploy contract changes necessary for the two-identity model
- Staging compose/config pin
- required tests/docs for that contract

Do not include unrelated application changes.

## 9. CI EXPECTATION FOR CONFIG CANDIDATE

Push the config candidate.

Required behavior after the fix:

CONFIG_VALIDATION_CI = GREEN

APPLICATION_IMAGE_REBUILT = NO

NEW_APP_IMAGE_DIGEST_CREATED = NO

COMPOSE_DIGEST remains:
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

If config-only commit still rebuilds/publishes a new app image:
FAIL.

## 10. DEPLOY DRY-RUN / PREFLIGHT

Do NOT deploy.

Run deployment preflight/dry-run against the config candidate.

It must accept:

APP_SOURCE_COMMIT = 27e4efe

APP_IMAGE_DIGEST =
sha256:d6a0c592fa860cfb4427ba6c16883c7d510bb432235b19c606d694029b6394b0

DEPLOY_CONFIG_COMMIT = <new config candidate SHA>

Required:

PROVENANCE_CONTRACT_ACCEPTS_SPLIT_IDENTITY = YES

COMPOSE_DIGEST_MATCHES_VERIFIED_APP_ARTIFACT = YES

STAGING_SCOPE_ONLY = YES

PRODUCTION_UNCHANGED = YES

## 11. REGRESSION TESTS

Add/adjust tests proving:

1. app source change triggers app image build
2. Dockerfile/build-input change triggers image build
3. Staging compose-only change does NOT trigger app image build
4. config-only candidate still runs deploy/config validation
5. deploy gate accepts distinct APP_SOURCE_COMMIT and DEPLOY_CONFIG_COMMIT
6. deploy gate rejects unknown/unverified digest
7. deploy gate rejects digest whose provenance does not match APP_SOURCE_COMMIT
8. deploy gate rejects red app CI
9. deploy gate rejects red config validation CI
10. Production config drift blocks
11. mutable tag-only deployment is rejected
12. exact digest pin is required

Run relevant release/deploy/security tests.

## 12. DOCUMENT RELEASE IDENTITY

Update the minimal deployment documentation to state explicitly:

Runtime/release identity consists of TWO independent commits:

APP_SOURCE_COMMIT
→ source that produced application image

DEPLOY_CONFIG_COMMIT
→ source that selected/pinned the image and runtime config

And one immutable artifact:

APP_IMAGE_DIGEST

This is intentional and not identity drift.

## 13. DO NOT DEPLOY

End this task after:
- CI architecture fixed
- clean config candidate created
- config validation green
- no app rebuild for config-only commit
- deploy preflight accepts split identity

Staging remains on its existing release until the next final deploy task.

## 14. FINAL REPORT

### A. PREFLIGHT

### B. ROOT CAUSE

RECURSIVE_DIGEST_CAUSE_PROVEN =
ROOT_CAUSE =

### C. TWO-IDENTITY CONTRACT

APP_SOURCE_COMMIT =
APP_IMAGE_DIGEST =
DEPLOY_CONFIG_COMMIT =

### D. CI CHANGES

IMAGE_AFFECTING_PATHS =
DEPLOY_CONFIG_ONLY_PATHS =
APP_BUILD_TRIGGER_RULE =
CONFIG_VALIDATION_TRIGGER_RULE =

### E. FILES CHANGED

### F. TESTS

### G. CONFIG CANDIDATE

CONFIG_BRANCH =
CONFIG_COMMIT =
CONFIG_VALIDATION_CI =
APPLICATION_IMAGE_REBUILT =
NEW_APP_IMAGE_DIGEST_CREATED =

### H. ARTIFACT PROVENANCE

APP_ARTIFACT_PROVEN =
COMPOSE_DIGEST =
COMPOSE_DIGEST_MATCHES_VERIFIED_APP_ARTIFACT =

### I. DEPLOY PREFLIGHT

PROVENANCE_CONTRACT_ACCEPTS_SPLIT_IDENTITY =
STAGING_SCOPE_ONLY =
PRODUCTION_UNCHANGED =

### J. SAFETY

STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
CENTRAL_SEAT_FIXTURE_CHANGED = NO
APPLICATION_AUTH_LOGIC_CHANGED = NO
HISTORY_REWRITTEN = NO
FORCE_PUSHED = NO
DEPLOYED = NO

### K. FINAL RESULT

If complete:

RESULT = PASS
RECURSIVE_DIGEST_BLOCKER_CLEARED = YES
CONFIG_ONLY_COMMIT_REBUILDS_IMAGE = NO
APP_ARTIFACT_IDENTITY = VERIFIED
DEPLOY_CONFIG_IDENTITY = VERIFIED
COMPOSE_DIGEST_MATCHES_VERIFIED_APP_ARTIFACT = YES
READY_FOR_P4A_FINAL_STAGING_DEPLOY = YES

If config-only commits still rebuild app image:

RESULT = HARD_STOP
RECURSIVE_DIGEST_BLOCKER_CLEARED = NO
BLOCKER = exact remaining coupling

Do not deploy Staging.
Do not deploy Production.
