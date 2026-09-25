# W1-FS-DIRTY-SAFEGUARD — Preserve Unique Work & Resolve Post Conflict

## ROLE

You are the **TICENPI DIRTY-WORK SAFEGUARD OWNER**.

Your job is to remove the **data-loss blocker** before workspace relocation by:

1. preserving all currently unique/uncommitted work in the known HIGH-risk repositories/worktrees in a recoverable local safeguard package;
2. determining whether any of those repositories can be safely committed locally without mixing unrelated work;
3. resolving the `TicenpiPost-wave2-integration` unmerged conflict only after evidence proves the intended final content;
4. producing a precise relocation-safety result for the later `W1-FS-A2` task.

Workspace root:

`F:\00-Ticenpi-SaaS\`

Primary prerequisite reports:

- `F:\00-Ticenpi-SaaS\W1-FS-A1-AUDIT-REPORT.md`
- `F:\00-Ticenpi-SaaS\W1-FS-A2-PREFLIGHT-REPORT.md`

Read both reports first.

This task is **not a workspace relocation task**.

Do not move, rename, delete, unregister, prune, or archive repositories/worktrees.

---

# 0. CURRENT SAFETY CONTEXT — VERIFY BEFORE USING

The latest preflight reported:

- Git version: `2.45.2`
- `git worktree move` is available
- 74 local linked worktrees are theoretically relocatable
- six canonical repositories currently contain HIGH-risk unique dirty work:
  - `TicenpiDM`
  - `TicenpiLetter`
  - `TicenpiSign`
  - `Ticenpi591`
  - `Ticenpi-Launcher`
  - `ExtractionHub`
- `TicenpiPost-wave2-integration` has an unmerged conflict (`UU`)
- preflight suggested that commit `f8c0535` may contain the incoming content needed to resolve the Post conflict
- `.release-evidence` must remain in place for now
- no delete candidates are approved

These are snapshot findings.

Revalidate current Git state before taking any mutation.

Do not assume a repo is still dirty, a conflict still exists, or `f8c0535` is still the correct resolution merely because the report said so.

---

# 1. ABSOLUTE SCOPE

## 1.1 Allowed

You may:

- inspect Git state, refs, history, worktree metadata, diffs, conflict stages
- inspect source/history/docs needed to understand ownership of dirty changes
- create a local safeguard directory outside all product Git repositories
- create:
  - text manifests
  - binary-capable Git patches
  - hashes
  - copies of unique untracked files
  - copies of conflicted files/stages needed for recovery
- make a **local Git commit** in one of the six HIGH-risk repositories only when all commit-safety conditions in this prompt are satisfied
- resolve the Post `wave2-integration` conflict only when all conflict-resolution gates are satisfied
- complete the already-in-progress Post Git operation only when its type and intended result are proven
- run non-destructive tests
- write one task report:
  - `F:\00-Ticenpi-SaaS\W1-FS-DIRTY-SAFEGUARD-REPORT.md`

## 1.2 Forbidden

Do not:

- move or rename any repository/worktree/folder
- delete any file/folder
- archive workspace folders
- run `git worktree move`
- run `git worktree remove`
- run `git worktree prune`
- run `git clean`
- run `git reset --hard`
- run destructive restore/checkout commands
- abort an in-progress merge/rebase/cherry-pick
- drop or clear a stash
- force checkout
- delete branches
- rebase unrelated history
- amend an existing commit unless current operation explicitly requires it and evidence proves it is the correct continuation
- push
- force-push
- merge PRs
- deploy
- modify Staging
- modify Production
- mutate databases
- change `.release-evidence`
- change deploy tooling/path coupling
- unregister incident/staging worktrees
- touch unrelated repositories

No remote push is permitted in this task.

---

# 2. SAFEGUARD DESTINATION

The safeguard must not live inside any product repository or linked worktree.

Preferred root:

`F:\00-Ticenpi-SaaS\.workspace-safeguard\W1-FS-DIRTY-SAFEGUARD\<run-id>\`

Before using it, verify that:

`F:\00-Ticenpi-SaaS\`

is not itself inside a Git worktree whose untracked state would be polluted by this directory.

If the workspace root is inside a Git repository, use instead:

`F:\00-Ticenpi-SaaS-SAFEGUARD\W1-FS-DIRTY-SAFEGUARD\<run-id>\`

Record the actual path as:

`SAFEGUARD_ROOT`

Use a deterministic timestamp/run id.

Do not place safeguard data inside `.release-evidence`.

---

# 3. TARGETS

At minimum inspect and safeguard these six repositories if still dirty:

1. `TicenpiDM`
2. `TicenpiLetter`
3. `TicenpiSign`
4. `Ticenpi591`
5. `Ticenpi-Launcher`
6. `ExtractionHub`

Also handle:

7. `TicenpiPost-wave2-integration`

Use the paths discovered by A1/A2-PREFLIGHT rather than guessing names.

If current evidence shows one of these paths changed or no longer exists, resolve the actual registered/canonical path from Git metadata.

Do not broaden into unrelated dirty repos unless the preflight report omitted a blocker that directly prevents A2.

---

# 4. PHASE A — REVALIDATE EACH TARGET

For every target collect:

```
COMPONENT =
PATH =
MAIN_REPO =
REMOTE =
BRANCH =
HEAD =
UPSTREAM =
AHEAD =
BEHIND =
DETACHED =
REGISTERED_WORKTREE =
GIT_OPERATION =
DIRTY =
TRACKED_MODIFIED =
TRACKED_DELETED =
UNTRACKED =
UNMERGED =
```

Safe commands may include:

```powershell
git -C "<path>" rev-parse --show-toplevel
git -C "<path>" rev-parse --git-common-dir
git -C "<path>" remote -v
git -C "<path>" branch --show-current
git -C "<path>" rev-parse HEAD
git -C "<path>" status --branch --short
git -C "<path>" diff --name-status
git -C "<path>" diff --cached --name-status
git -C "<path>" ls-files --others --exclude-standard
git -C "<path>" ls-files -u
git -C "<path>" log --oneline --decorate -n 30
```

Detect in-progress Git operations safely from Git metadata:

- merge
- rebase
- cherry-pick
- revert
- bisect

Do not continue or abort anything until its state is understood.

---

# 5. PHASE B — CREATE RECOVERABLE SAFEGUARD BEFORE ANY COMMIT/RESOLUTION

For every target that is dirty, create a dedicated safeguard package:

`<SAFEGUARD_ROOT>\<component>\`

Minimum contents:

```
manifest.md
status.txt
branch-head.txt
remote.txt
worktree-metadata.txt
tracked-working.patch
tracked-index.patch
untracked-files.txt
unmerged-index.txt
hashes.sha256
```

Where applicable also create:

```
untracked\...
conflicts\...
operation-state\...
```

## 5.1 Tracked changes

Capture binary-capable patches:

```powershell
git -C "<path>" diff --binary
git -C "<path>" diff --cached --binary
```

Write them to separate files.

Do not rely on a normal text diff for binary changes.

## 5.2 Untracked files

For each untracked file:

- record relative path
- record size
- record SHA-256
- classify likely type:
  - SOURCE
  - DOC
  - CONFIG
  - GENERATED
  - BINARY
  - SECRET_LIKELY
  - UNKNOWN

For files containing likely credentials/secrets:

- do not print contents into the report
- do not include secret contents in Git commits
- copy only to the local safeguard destination if preservation is necessary
- label as `SENSITIVE_LOCAL_BACKUP`
- retain exact relative path under the safeguard package
- never expose values in console summary

For other unique untracked files, copy them preserving relative paths.

Do not assume generated output is disposable until proven reproducible.

## 5.3 Conflict state

For any unmerged path:

record:

```
git ls-files -u
```

and preserve:

- stage 1 blob if present
- stage 2 blob if present
- stage 3 blob if present
- current working-tree conflicted file
- current operation metadata relevant to recovery

Use read-only blob extraction.

Do not resolve anything until this safeguard completes.

## 5.4 Integrity

Generate SHA-256 hashes for safeguard artifacts and copied files.

The manifest must state:

```
SOURCE_PATH =
SOURCE_HEAD =
SOURCE_BRANCH =
CREATED_AT =
TRACKED_PATCH_PRESENT =
INDEX_PATCH_PRESENT =
UNTRACKED_BACKUP_COUNT =
CONFLICT_BACKUP_PRESENT =
SAFEGUARD_COMPLETE =
```

A target is not considered safeguarded until its manifest and hashes are complete.

---

# 6. PHASE C — DETERMINE WHETHER LOCAL COMMIT IS SAFE

The purpose of this task is **not** to force every dirty repo clean.

Default action is:

`SAFEGUARD_ONLY`

A local commit is allowed only when **all** conditions below are true:

1. the current branch is clearly the intended branch for the dirty work;
2. the changes form one coherent known task;
3. source/history/docs make the task intent identifiable;
4. no unrelated edits are mixed in;
5. no likely secret file would be committed;
6. generated files are committed only if the repository intentionally tracks them;
7. the changed files can be staged using explicit paths;
8. appropriate tests can be run;
9. the commit does not include pre-existing unrelated work;
10. the safeguard package already exists.

If any condition is unclear:

`COMMIT_DECISION = SAFEGUARD_ONLY`

Do not ask the user merely because commit intent is unclear; safeguard the work and continue.

## 6.1 Commit rules

If a local commit is clearly safe:

- inspect the full diff first
- stage explicit paths only
- never use `git add .`
- never use `git add -A`
- do not stage unrelated untracked files
- run appropriate targeted tests
- use a descriptive commit message based on the actual existing task
- verify the resulting commit contains only intended files
- do not push

Record:

```
LOCAL_COMMIT_CREATED = YES
COMMIT =
FILES =
TESTS =
```

If the repo remains dirty due to unrelated/ambiguous work, keep the remainder safeguarded and report it.

A local commit is not required merely to make the report look clean.

---

# 7. PHASE D — POST WAVE2 CONFLICT FORENSICS

Target:

`TicenpiPost-wave2-integration`

Do not use `git checkout --ours` or `git checkout --theirs` blindly.

Their meaning can differ depending on merge/rebase context.

First determine:

```
PATH =
BRANCH =
HEAD =
GIT_OPERATION =
MERGE_HEAD =
REBASE_STATE =
CHERRY_PICK_HEAD =
UNMERGED_PATHS =
```

For each unmerged path inspect:

- base/stage 1
- stage 2
- stage 3
- current working file
- the version of that path at commit `f8c0535`, if that commit exists and contains the path
- relevant surrounding history

Use explicit blob/content comparisons.

For example, when valid:

```powershell
git -C "<path>" show :1:<file>
git -C "<path>" show :2:<file>
git -C "<path>" show :3:<file>
git -C "<path>" show f8c0535:<file>
```

Do not assume `f8c0535` is "incoming" until proven by the actual operation/history.

---

# 8. POST CONFLICT RESOLUTION GATE

You may resolve the conflict only if all of the following are true:

1. the conflict is still present;
2. the in-progress Git operation is clearly identified;
3. safeguard of the conflict stages/current file is complete;
4. commit `f8c0535` exists in the correct Post repository/history;
5. the intended final content can be proven from surrounding commits/task history/source;
6. choosing content equivalent to `f8c0535` does not discard unique local changes that are not preserved elsewhere;
7. all unmerged paths are understood;
8. the exact resolution can be expressed without ambiguous ours/theirs semantics.

If any condition fails:

```
POST_CONFLICT = BLOCKED
```

Do not abort the Git operation.

Do not invent a resolution.

---

# 9. POST CONFLICT EXECUTION

If the gate passes, resolve using explicit final content.

Prefer writing the proven final blob/content rather than relying on semantic shorthand such as `--theirs`.

For each resolved path:

1. write exact intended content;
2. inspect diff;
3. `git add <explicit-path>`;
4. verify `git ls-files -u` no longer contains that path.

After all conflicts are resolved:

- run the relevant Post targeted tests/contract tests that cover the affected files;
- inspect the staged diff;
- verify no unrelated files were staged.

Then complete the **existing** Git operation using the correct continuation only:

- merge → commit the merge using the existing intended message/context
- cherry-pick → `git cherry-pick --continue`
- rebase → `git rebase --continue`
- other operation → continue only if its proper semantics are verified

Do not convert one operation type into another merely to finish faster.

Do not push.

If tests fail before continuation and the failure may be caused by the resolution:

- stop;
- leave the safeguarded state intact;
- report `POST_CONFLICT = RESOLVED_NOT_COMMITTED_TEST_BLOCKED` or equivalent;
- do not hide the failure.

After successful continuation, verify:

```
git status --branch --short
git ls-files -u
git rev-parse HEAD
```

and run the relevant tests again if appropriate.

---

# 10. PHASE E — RELOCATION SAFETY DECISION

For each of the six HIGH-risk repos and the Post conflict worktree, return exactly one:

```
A2_RELOCATION_GUARD =
PASS_CLEAN
PASS_WITH_LOCAL_COMMIT
PASS_WITH_VERIFIED_SAFEGUARD
BLOCKED_CONFLICT
BLOCKED_UNSAFE_UNIQUE_WORK
BLOCKED_INCOMPLETE_BACKUP
BLOCKED_UNKNOWN
```

Interpretation:

### PASS_CLEAN
No dirty/unique work remains.

### PASS_WITH_LOCAL_COMMIT
Unique work is now captured in a verified local commit and repository state is suitable for later relocation.

### PASS_WITH_VERIFIED_SAFEGUARD
Dirty work may remain, but a complete verified local safeguard exists and A2 may later relocate the directory **without deleting or cleaning that work**, subject to A2's path/reference checks.

### BLOCKED_*
A2 must not relocate that target.

A safeguard is not permission to delete the source.

---

# 11. IMPORTANT DISTINCTION: BACKUP VS CLEANUP

This task must not conflate:

```
protected
```

with:

```
committed
```

or:

```
safe to delete
```

A dirty repo can become safe for relocation after a verified safeguard package without becoming clean.

The later A2 task must preserve its exact dirty working-tree state while moving it.

No target from this task becomes a delete candidate.

---

# 12. .release-evidence AND TOOLING PATHS

Do not touch:

`F:\00-Ticenpi-SaaS\.release-evidence`

Current policy for this task:

```
.release-evidence = KEEP_IN_PLACE
W1-FS-TOOLING-PATHS = DEFERRED
```

Do not modify `TICENPI_RELEASE_EVIDENCE_ROOT`.

Do not update the 10 path-coupled files.

This is not a blocker for safeguarding dirty repos.

---

# 13. WORKTREE REGISTRATION

Do not unregister:

- `.staging-orchestration` worktrees
- `.incident-worktrees` worktrees
- any of the 74 registered linked worktrees

Do not run worktree relocation yet.

Those belong to later A2 execution.

---

# 14. TESTING PRINCIPLE

For a repo where you create a local commit or resolve Post conflict:

- run the smallest relevant reliable test set first;
- if a repository has a documented standard gate that is practical, run it;
- do not launch an unrelated full-production deployment test suite solely for filesystem safeguarding.

Report tests exactly.

Do not convert historical PASS results into current results.

If tests are unavailable:

```
TEST_STATUS = NOT_RUN
REASON =
```

---

# 15. REPORT FILE

Write:

`F:\00-Ticenpi-SaaS\W1-FS-DIRTY-SAFEGUARD-REPORT.md`

This report may contain:

- paths
- branch names
- SHAs
- file lists
- hash values
- test outcomes
- safeguard locations

It must not contain secret values.

---

# 16. REQUIRED FINAL RESPONSE

Return a concise result with the following exact sections.

## A. RESULT

```
RESULT = PASS / PARTIAL_PASS / HARD_STOP
REPORT = F:\00-Ticenpi-SaaS\W1-FS-DIRTY-SAFEGUARD-REPORT.md
SAFEGUARD_ROOT =
MOVED = 0
DELETED = 0
WORKTREES_UNREGISTERED = 0
PUSHED = NO
STAGING_MUTATED = NO
PRODUCTION_MUTATED = NO
```

## B. SAFEGUARD SUMMARY

```
TARGETS_EXPECTED = 7
TARGETS_CURRENTLY_DIRTY =
TARGETS_SAFEGUARDED =
TARGETS_WITH_LOCAL_COMMITS =
TARGETS_BLOCKED =
```

## C. SIX HIGH-RISK REPOS

For each:

```
COMPONENT =
PATH =
BRANCH =
HEAD_BEFORE =
DIRTY_BEFORE =
SAFEGUARD =
SAFEGUARD_HASH_VERIFIED =
COMMIT_DECISION = SAFEGUARD_ONLY / LOCAL_COMMIT / NOT_NEEDED / BLOCKED
LOCAL_COMMIT =
DIRTY_AFTER =
A2_RELOCATION_GUARD =
BLOCKER =
```

Include:

- TicenpiDM
- TicenpiLetter
- TicenpiSign
- Ticenpi591
- Ticenpi-Launcher
- ExtractionHub

## D. POST WAVE2 CONFLICT

```
PATH =
GIT_OPERATION =
HEAD_BEFORE =
UNMERGED_PATHS_BEFORE =
F8C0535_EXISTS =
F8C0535_ROLE_PROVEN =
RESOLUTION_DECISION =
RESOLUTION_CONTENT_SOURCE =
TESTS =
OPERATION_COMPLETED =
HEAD_AFTER =
UNMERGED_PATHS_AFTER =
DIRTY_AFTER =
POST_CONFLICT_STATUS =
A2_RELOCATION_GUARD =
```

Do not report `f8c0535` as the chosen resolution unless actually proven.

## E. LOCAL COMMITS

List every local commit created:

```
REPO =
BRANCH =
COMMIT =
FILES =
PUSHED = NO
```

If none:

`LOCAL_COMMITS = NONE`

## F. REMAINING HIGH-RISK WORK

List only unresolved unique-work/conflict blockers.

For each:

```
PATH =
RISK =
WHY =
MINIMUM_NEXT_ACTION =
```

## G. A2 READINESS DELTA

```
A2_DIRTY_WORK_BLOCKER = CLEARED / PARTIAL / BLOCKED
A2_POST_CONFLICT_BLOCKER = CLEARED / PARTIAL / BLOCKED
A2_TOOLING_PATH_BLOCKER = DEFERRED_KEEP_RELEASE_EVIDENCE_IN_PLACE
A2_WORKTREE_MOVE_CAPABILITY = PREVIOUSLY_VERIFIED
```

Then list:

```
SAFE_FOR_A2_NOW =
NOT_SAFE_FOR_A2 =
```

## H. NEXT TASK

If dirty-work and Post blockers are cleared or safely safeguarded:

```
NEXT_TASK = W1-FS-A2 — Ticenpi Workspace Safe Relocation & Cleanup
```

If a target remains blocked, name the smallest prerequisite task first.

Do not execute A2.

---

# 17. SUCCESS CONDITION

This task succeeds when:

> Every HIGH-risk dirty repository has either a verified recoverable safeguard or a clearly justified local commit, the Post `wave2-integration` conflict has either been safely resolved from proven evidence or explicitly blocked without data loss, no unique work has been discarded, nothing has been pushed/deployed/moved/deleted, and W1-FS-A2 has an explicit allow/block decision for every target.

The priority is **preserving work**, not making `git status` look clean.

Again:

**DO NOT MOVE, DELETE, PRUNE, UNREGISTER WORKTREES, PUSH, OR DEPLOY IN THIS TASK.**
