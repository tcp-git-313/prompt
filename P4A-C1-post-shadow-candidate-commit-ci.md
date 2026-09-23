# P4A-C1 — Post Shadow Candidate Commit & CI Gate

ROLE: POST SHADOW RELEASE CANDIDATE OWNER

MODE: GIT / CI ONLY — NO DEPLOY

REPO:
F:\00-Ticenpi-SaaS\TicenpiPost

GOAL:
Turn the already-verified P4A Central Seat shadow implementation into an exact GitHub-hosted release candidate commit and obtain green CI, without deploying Staging yet.

CURRENT VERIFIED STATE:
- branch: feat/post-tenant-provisioning
- HEAD: 3681a1c23dbaaa22d0e630b93a887f15c540354d
- P4A diff isolated to exactly 4 intended files
- P4A targeted tests: 136 passed
- P4A_READY_BY_EVIDENCE = YES
- current branch is reported as 4 commits ahead of origin
- deploy pipeline requires exact candidate commit on GitHub with green CI

ABSOLUTE RULES:
- Do NOT deploy Staging.
- Do NOT touch Production.
- Do NOT change Central Seat SQL/RPC.
- Do NOT change runtime flags.
- Do NOT include unrelated files in the P4A commit.
- Do NOT rewrite, rebase, reset, squash, or force-push history.
- Do NOT discard existing commits/work.
- Do NOT amend existing commits.
- Do NOT push the current branch blindly before auditing the 4 ahead commits.

## 1. PREFLIGHT

Record:
- current branch
- HEAD
- origin tracking branch
- git status --short
- git diff --stat
- git log --oneline --decorate --graph for:
  origin/<tracking>..HEAD

Confirm the working-tree P4A diff is still exactly limited to:

backend/app/core/identity.py
backend/app/api/auth.py
backend/app/api/studio.py
backend/tests/test_tenant_bootstrap.py

If any additional source file is modified:
STOP.

## 2. AUDIT THE FOUR AHEAD COMMITS

Inspect each commit that exists in local HEAD history but not in origin tracking branch.

For each commit report:
- SHA
- subject
- changed files
- purpose
- whether it is required for the current Post branch/candidate
- whether it contains unrelated work
- whether it has already been validated elsewhere if discoverable

Classify each:
REQUIRED
SAFE_RELATED
UNRELATED
UNKNOWN

Do not modify history.

## 3. RELEASE-CANDIDATE BASE DECISION

Preferred safe outcome:

If ALL 4 ahead commits are REQUIRED or SAFE_RELATED:
- they may remain as ancestors of the P4A candidate
- do not push current branch directly
- create a dedicated release-candidate branch from current HEAD before committing P4A

Recommended branch name:
release/post-central-seat-shadow-staging-20260923

If that exact branch already exists locally or remotely:
use a collision-safe suffix; do not overwrite.

If ANY ahead commit is UNRELATED or UNKNOWN:
STOP.

Do not cherry-pick/rebase/reset automatically.
Report the exact offending commit(s).

## 4. RE-RUN P4A LOCAL GATE

Before committing:

- run targeted P4A auth/shadow tests
- relevant CommercialGate / SeatChecker / identity tests
- git diff --check

Required:
all targeted tests PASS.

Expected prior evidence:
136 passed

Do not let unrelated existing full-suite invariant failures block this candidate if they are proven pre-existing and outside the four P4A files.

## 5. CREATE DEDICATED CANDIDATE BRANCH

Only if Sections 2–4 pass:

Create a new branch from CURRENT HEAD, preserving history exactly.

Do not move/reset current branch history.

Branch:
release/post-central-seat-shadow-staging-20260923
or safe suffixed variant.

Record:
CANDIDATE_BRANCH =

## 6. COMMIT ONLY THE FOUR P4A FILES

Stage exactly:

backend/app/core/identity.py
backend/app/api/auth.py
backend/app/api/studio.py
backend/tests/test_tenant_bootstrap.py

Before commit:
- show staged file list
- prove no other file staged

Commit message:
feat(post): add central seat shadow observation

Do not include unrelated work.

Record:
P4A_CANDIDATE_COMMIT =

After commit:
- git status --short
- git show --stat --oneline HEAD
- git diff HEAD^..HEAD --name-only

Required:
the candidate commit contains exactly the four intended files.

## 7. PUSH ONLY THE DEDICATED CANDIDATE BRANCH

Push the dedicated candidate branch normally.

Never force push.

Do not push or modify the original feat/post-tenant-provisioning remote branch in this task.

Verify:
- remote branch exists
- candidate commit SHA on GitHub exactly matches local candidate HEAD

Record:
REMOTE_CANDIDATE_COMMIT =

Required:
REMOTE_CANDIDATE_COMMIT == P4A_CANDIDATE_COMMIT

## 8. CI

Observe the CI checks triggered for the exact candidate commit.

Required relevant workflows/checks must finish green.

Do not deploy.

If CI fails:
- inspect failure
- if failure is directly caused by P4A code and fix is small/within the same 4-file scope, report it but DO NOT silently broaden scope
- if failure is pre-existing/unrelated, report evidence
- do not deploy on non-green CI

Record:
CI_RUN_IDS =
CI_STATUS =
CI_CHECKS =

## 9. CANDIDATE IDENTITY

Produce the exact immutable candidate identity:

Candidate branch:
Candidate commit:
GitHub remote SHA:
CI status:
Changed files:

This exact SHA will be used by the next Staging deploy task.

## 10. SAFETY

Must state:

STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
RUNTIME_FLAGS_CHANGED = NO
CURRENT_ORIGINAL_BRANCH_FORCE_PUSHED = NO
HISTORY_REWRITTEN = NO
UNRELATED_FILES_COMMITTED = NO
DEPLOYED = NO

## 11. FINAL RESULT

If successful:

RESULT = PASS
P4A_CANDIDATE_COMMITTED = YES
P4A_CANDIDATE_PUSHED = YES
P4A_CANDIDATE_CI_GREEN = YES
P4A_CANDIDATE_COMMIT = <sha>
CANDIDATE_BRANCH = <branch>
READY_FOR_P4A_STAGING_DEPLOY = YES

If ahead commits are unsafe:

RESULT = HARD_STOP
P4A_CANDIDATE_COMMITTED = NO
P4A_CANDIDATE_PUSHED = NO
READY_FOR_P4A_STAGING_DEPLOY = NO
BLOCKER = exact ahead commit classification

If CI fails:

RESULT = BLOCKED
P4A_CANDIDATE_COMMITTED = YES
P4A_CANDIDATE_PUSHED = YES
P4A_CANDIDATE_CI_GREEN = NO
READY_FOR_P4A_STAGING_DEPLOY = NO
BLOCKER = exact failing CI/check

Stop after CI. Do not deploy.
