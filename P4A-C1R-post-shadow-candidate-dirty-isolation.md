# P4A-C1R — Resume Post Shadow Candidate Commit & CI with Dirty-File Isolation

ROLE: POST SHADOW RELEASE CANDIDATE OWNER

MODE: GIT / CI ONLY — NO DEPLOY

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Resume P4A candidate creation without falsely blocking on unrelated unstaged local files.

CURRENT KNOWN STATE:
- branch: feat/post-tenant-provisioning
- HEAD: 3681a1c23dbaaa22d0e630b93a887f15c540354d
- tracking: origin/feat/post-tenant-provisioning
- ahead commits: 4
- P4A core diff exists
- targeted P4A tests previously passed
- worktree currently has 6 modified files

Known modified files include:

P4A core:
- backend/app/core/identity.py
- backend/app/api/auth.py
- backend/app/api/studio.py
- backend/tests/test_tenant_bootstrap.py

Additional dirty files:
- backend/tests/test_tenant_bootstrap_flag_contract.py
- deploy/runtime-staging/compose.yml

IMPORTANT:
Do NOT treat unrelated unstaged dirty files as an automatic blocker.
Git can safely create a commit from an explicit staged file list.

The safety gate is:
- exact staged-file control
- exact commit diff control
- ahead-commit audit
- no history rewrite
- green CI

ABSOLUTE RULES:
- Do NOT deploy Staging.
- Do NOT touch Production.
- Do NOT change runtime flags.
- Do NOT reset/restore/stash/clean/rebase.
- Do NOT discard the two additional dirty files.
- Do NOT amend existing commits.
- Do NOT force push.
- Do NOT blindly push the current feature branch.
- Do NOT stage files outside the explicitly approved candidate scope.

## 1. PREFLIGHT

Record:
- branch
- HEAD
- tracking branch
- git status --short
- git diff --stat
- git diff --check
- git log --oneline --decorate --graph origin/<tracking>..HEAD

## 2. CLASSIFY THE TWO ADDITIONAL DIRTY FILES

Inspect only:

backend/tests/test_tenant_bootstrap_flag_contract.py
deploy/runtime-staging/compose.yml

For each report:
- exact semantic change
- likely task/owner
- whether required for P4A local shadow code
- whether required later for Staging shadow enablement
- whether it must be in THIS CI candidate commit
- whether leaving it unstaged changes the remote candidate behavior

Classify each as exactly one:

P4A_CORE_REQUIRED
P4A_STAGING_CONFIG_FOLLOWUP
RELATED_TEST_FOLLOWUP
UNRELATED_LOCAL_WIP
UNKNOWN

Rules:

A.
If UNKNOWN:
STOP.

B.
If P4A_STAGING_CONFIG_FOLLOWUP or RELATED_TEST_FOLLOWUP:
do NOT stage it in the core P4A code commit unless the test is required for CI to prove the code contract.

C.
deploy/runtime-staging/compose.yml must NOT be included in the core code commit merely to enable the flag.
Runtime flag enablement belongs to the later Staging deploy/config step unless the repository's release contract requires this exact config file to travel with the candidate. If it does, explain that exact requirement.

D.
Do not modify either file in this task.

## 3. AUDIT THE FOUR AHEAD COMMITS

Inspect every commit in:

origin/<tracking>..HEAD

For each report:
- SHA
- subject
- changed files
- purpose
- whether required as candidate ancestry
- classification:
  REQUIRED
  SAFE_RELATED
  UNRELATED
  UNKNOWN

If any commit is UNRELATED or UNKNOWN:
STOP.

Do not rewrite history.

## 4. DETERMINE CANDIDATE FILE SCOPE

Default core candidate scope:

backend/app/core/identity.py
backend/app/api/auth.py
backend/app/api/studio.py
backend/tests/test_tenant_bootstrap.py

Potential fifth file:

backend/tests/test_tenant_bootstrap_flag_contract.py

Include that test file ONLY if Section 2 proves:
- it directly tests the new P4A shadow flag contract, AND
- CI needs it to validate the candidate, AND
- it contains no unrelated behavior.

Never include:
deploy/runtime-staging/compose.yml
in the core candidate commit unless the approved CI/release contract requires runtime-staging compose to be versioned with the code candidate. If required, STOP and report rather than automatically mixing runtime config into the code commit.

Set:

CANDIDATE_FILES = exact list

## 5. LOCAL GATE

Run:
- targeted P4A shadow/auth tests
- CommercialGate / SeatChecker / identity tests
- include test_tenant_bootstrap_flag_contract.py if classified as direct P4A test
- git diff --check

Required:
all candidate-relevant tests PASS.

Known unrelated full-suite failures do not block if proven pre-existing and outside candidate files.

Set:

P4A_READY_BY_EVIDENCE = YES/NO

## 6. CREATE DEDICATED CANDIDATE BRANCH

Only if Sections 2–5 pass.

Create from current HEAD:

release/post-central-seat-shadow-staging-20260923

If branch exists, use a collision-safe suffix.

Do not reset or alter history.

Record:

CANDIDATE_BRANCH =

## 7. STAGE ONLY CANDIDATE FILES

Use explicit paths.

Before commit run:

git diff --cached --name-only

Required:
exactly CANDIDATE_FILES
and nothing else.

The two non-candidate dirty files may remain unstaged in the worktree.

This is NOT a blocker.

## 8. COMMIT

Commit message:

feat(post): add central seat shadow observation

Record:

P4A_CANDIDATE_COMMIT =

After commit verify:

git show --name-only --format=fuller HEAD

Required:
exact candidate files only.

Also verify local unstaged dirty files remain preserved.

## 9. PUSH DEDICATED BRANCH ONLY

Push only:

CANDIDATE_BRANCH

Never force push.

Do not modify the remote feature branch.

Verify:
REMOTE_CANDIDATE_COMMIT == P4A_CANDIDATE_COMMIT

## 10. CI

Wait for all required CI checks on the exact candidate SHA.

Record:
- workflow/check names
- run IDs
- result

If CI fails:
- inspect cause
- do not deploy
- do not broaden file scope automatically

## 11. RUNTIME CONFIG FOLLOWUP

If deploy/runtime-staging/compose.yml was classified as P4A_STAGING_CONFIG_FOLLOWUP:

Report its exact intended role, e.g. enabling:

TICENPI_CENTRAL_SEAT_SHADOW_ENABLED=1

But do not commit or deploy it here.

State whether the later deploy task must:
- apply it as a separate config change,
- use an existing runtime override mechanism,
- or create a separate reviewed config commit.

Do not guess; derive from actual deploy architecture.

## 12. FINAL REPORT

A. PREFLIGHT

B. EXTRA DIRTY FILE CLASSIFICATION

Table:
File | Classification | Required in core candidate | Required later | Preserved unstaged

C. AHEAD COMMIT AUDIT

Table:
SHA | Subject | Classification | Candidate ancestry safe

D. CANDIDATE FILES

CANDIDATE_FILES =

E. LOCAL TESTS

F. CANDIDATE IDENTITY

CANDIDATE_BRANCH =
P4A_CANDIDATE_COMMIT =
REMOTE_CANDIDATE_COMMIT =

G. CI

P4A_CANDIDATE_CI_GREEN = YES/NO

H. FOLLOWUP CONFIG

RUNTIME_STAGING_COMPOSE_REQUIRED_LATER = YES/NO
RECOMMENDED_HANDLING =

I. SAFETY

STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
HISTORY_REWRITTEN = NO
FORCE_PUSHED = NO
UNRELATED_FILES_COMMITTED = NO
EXTRA_DIRTY_FILES_PRESERVED = YES
DEPLOYED = NO

J. FINAL RESULT

If successful:

RESULT = PASS
P4A_CANDIDATE_COMMITTED = YES
P4A_CANDIDATE_PUSHED = YES
P4A_CANDIDATE_CI_GREEN = YES
READY_FOR_P4A_STAGING_DEPLOY = YES

If unsafe ahead commits or UNKNOWN dirty-file ownership:

RESULT = HARD_STOP
READY_FOR_P4A_STAGING_DEPLOY = NO
BLOCKER = exact factual blocker

If CI fails:

RESULT = BLOCKED
P4A_CANDIDATE_COMMITTED = YES
P4A_CANDIDATE_PUSHED = YES
P4A_CANDIDATE_CI_GREEN = NO
READY_FOR_P4A_STAGING_DEPLOY = NO
BLOCKER = exact failing check

Stop after CI. Do not deploy.
