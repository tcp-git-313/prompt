# P4B-R1 — DM Auth Collision Reconciliation

ROLE: DM AUTH COLLISION RECONCILIATION OWNER

MODE: READ-ONLY / NO IMPLEMENTATION

REPO:
F:\00-Ticenpi-SaaS\TicenpiDM

GOAL:
Resolve the current dirty-file collision around:

src/backend/app/auth/entitlement.py::require_dm_access

without modifying or discarding existing user work.

The output must identify exactly what the pre-existing local bypass changes do, whether they are intentional and compatible with the planned Central Seat shadow, and the safest insertion point for P4B.

DO NOT edit files.
DO NOT reset/restore/stash/clean/rebase.
DO NOT commit/push/deploy.
DO NOT touch Staging/Production.
DO NOT modify Post, Launcher, or ticenpi-platform.

## 1. Preflight

Record:
- branch
- HEAD
- git status --short
- git diff --stat

Known baseline:
branch = dm/integration-20260920
HEAD = 561bd8c76fec5c0fb86cb9cf4f56734d28367b8a

Do not assume the worktree is unchanged since the previous P4B attempt.

## 2. Inspect the collision precisely

For:

src/backend/app/auth/entitlement.py

compare:
- HEAD version
- current working-tree version

Focus on:
- require_dm_access()
- has_dm_access()
- my_commercial_context()
- any local/dev bypass conditions
- platform_admin logic
- test_access logic
- error mapping
- imports/helpers newly introduced

Report the exact semantic differences.

Do not paste large diffs.
Summarize each changed hunk by behavior.

## 3. Determine ownership of the dirty changes

Trace references from the changed code to determine whether the local bypass changes belong to:

- local/dev auth work
- OAuth work
- YUCT extraction-shadow work
- another currently active DM task
- unknown/unattributed work

Use evidence from:
- nearby modified files
- tests
- comments
- call sites
- git status/diff
- current branch context

Do not guess.

Classify:

DIRTY_CHANGE_OWNER =
LOCAL_DEV_AUTH
OAUTH
YUCT_EXTRACTION_SHADOW
OTHER
UNKNOWN

If UNKNOWN, say so.

## 4. Reconstruct the current effective auth flow

Show the current working-tree flow, not only HEAD:

get_current_user
→ require_dm_access()
→ ...
→ protected router

Include all branches for:

- Production auth required
- non-Production auth required
- local/dev bypass
- platform_admin
- Staging test_access allow
- Staging test_access deny
- entitlement allow
- entitlement deny
- central/RPC error

State current HTTP behavior where visible:
401 / 403 / 503 / allow

## 5. Find safe Shadow insertion options

Evaluate at least these candidate strategies:

A. Inside require_dm_access()
B. Immediately after legacy has_dm_access() decision inside a helper extracted later
C. A new shadow helper called from require_dm_access()
D. A wrapper dependency around EntitledUserDep
E. Another existing shared auth layer

For each report:

- files/symbols touched
- overlaps current dirty hunks YES/NO
- preserves local bypass YES/NO
- preserves current user-visible auth behavior YES/NO
- route duplication risk
- test impact
- recommended / not recommended

Do NOT implement any option.

## 6. Required compatibility rule

The future P4B shadow implementation must preserve all current intentional dirty behavior exactly.

It must NOT:
- remove or reorder the current local/dev bypass
- change platform_admin behavior
- change Staging test_access behavior
- change 401/403 behavior
- change DM RLS
- make Central Seat authoritative
- edit unrelated YUCT/OAuth work

Shadow must be observational only.

## 7. Collision classification

Return exactly one:

COLLISION_CLASS = SAFE_TO_PATCH_IN_PLACE
if the exact dirty logic is understood and a minimal non-destructive patch can be made in the same file while preserving all current behavior.

COLLISION_CLASS = SAFE_WITH_NEW_HELPER
if a new isolated helper/module is safer and require_dm_access() only needs a minimal call-site addition.

COLLISION_CLASS = WAIT_FOR_OTHER_WIP
if current overlapping work is still actively changing or its intent cannot be proven.

COLLISION_CLASS = HARD_STOP
if safe integration cannot be designed without rewriting/overwriting unknown dirty work.

## 8. Minimal future patch contract

If SAFE_TO_PATCH_IN_PLACE or SAFE_WITH_NEW_HELPER, specify the exact future allowed edit scope:

ALLOWED_FILES =
ALLOWED_SYMBOLS =
DO_NOT_TOUCH =

Also specify the exact order:

legacy current behavior
→ shadow Central Seat lookup
→ mismatch logging
→ return original legacy result

If the local/dev bypass should skip the shadow call entirely, state that explicitly and why.

If it should still shadow in certain non-Production modes, state that explicitly and why.

## 9. Tests required for the resumed P4B

List the exact tests that must prove:

- pre-existing local bypass unchanged
- platform_admin unchanged
- test_access allow unchanged
- test_access deny unchanged
- legacy allow + central deny leaves allow unchanged
- legacy deny + central allow leaves deny unchanged
- central error leaves legacy result unchanged
- no secrets logged
- protected routers still use EntitledUserDep
- DM RLS unaffected

Identify existing test files to extend if present.

Do not edit them.

## 10. Final report

### A. PREFLIGHT

### B. DIRTY DIFF SUMMARY

Table:
File | Symbol | HEAD behavior | Working-tree behavior | Likely owner

### C. CURRENT WORKTREE AUTH FLOW

### D. BYPASS SEMANTICS

LOCAL_DEV_BYPASS =
PLATFORM_ADMIN =
TEST_ACCESS_ALLOW =
TEST_ACCESS_DENY =
ENTITLEMENT_DENY =
RPC_ERROR =

### E. SHADOW INSERTION OPTIONS

Table:
Option | Touches dirty hunk | Behavior risk | Duplication risk | Recommendation

### F. COLLISION RESULT

COLLISION_CLASS =

### G. RESUMED P4B PATCH CONTRACT

ALLOWED_FILES =
ALLOWED_SYMBOLS =
DO_NOT_TOUCH =
SHADOW_ORDER =

### H. REQUIRED TESTS

### I. SAFETY

SOURCE_FILES_MODIFIED = NO
WORKTREE_CLEANED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
COMMIT_CREATED = NO
PUSHED = NO
DEPLOYED = NO

### J. FINAL RESULT

If safe:

RESULT = PASS
DM_AUTH_COLLISION_RECONCILED = YES
READY_TO_RESUME_P4B = YES
RECOMMENDED_STRATEGY = SAFE_TO_PATCH_IN_PLACE or SAFE_WITH_NEW_HELPER

If not safe:

RESULT = BLOCKED
DM_AUTH_COLLISION_RECONCILED = NO
READY_TO_RESUME_P4B = NO
BLOCKER = exact factual blocker

Finish reconciliation and stop.
