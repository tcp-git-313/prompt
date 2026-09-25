# W1-FS-A2-PREFLIGHT — Dirty Worktree & Path Coupling Resolution

## ROLE

You are the **TICENPI WORKSPACE CLEANUP PREFLIGHT OWNER**.

Your job is to prepare the workspace for a later safe relocation/cleanup task without moving or deleting anything yet.

Workspace root:

`F:\00-Ticenpi-SaaS\`

Primary input report:

`F:\00-Ticenpi-SaaS\W1-FS-A1-AUDIT-REPORT.md`

Read that report first, then independently revalidate all facts that affect safety.

This task exists specifically to resolve the blockers reported by W1-FS-A1 before W1-FS-A2 is allowed to perform any actual relocation.

---

# 0. TASK INTENT

W1-FS-A1 reported, at the time of that audit:

- 63 top-level items
- 11 canonical repositories
- 74 registered Git worktrees
- 15 dirty repos/worktrees
- HIGH unique-work risk in at least:
  - Post / Wave2 integration work
  - TicenpiDM
  - TicenpiLetter
  - TicenpiSign
  - Ticenpi591
  - Ticenpi-Launcher
- `.release-evidence` is path-coupled to deployment tooling, including:
  - `deploy/tools/operator_commands.ps1`
- 0 safe DELETE_CANDIDATES
- A2 readiness was only PARTIAL

Treat these as the **A1 snapshot**, not current truth.

Revalidate before drawing conclusions.

The purpose of this preflight is to resolve exactly three classes of blocker:

1. **dirty / unique work risk**
2. **Git worktree relocation capability and constraints**
3. **absolute-path / folder-name coupling in tooling and documentation**

Do not perform the cleanup itself.

---

# 1. ABSOLUTE SAFETY BOUNDARY

This is still a **NON-RELOCATION / NON-DELETION task**.

Allowed:

- read files
- inspect Git metadata
- inspect worktree registrations
- inspect status/diffs
- inspect branch/ref history
- inspect exact path references
- inspect deployment scripts
- inspect release evidence metadata
- calculate hashes without writing Git objects
- run non-mutating Git capability/help/version commands
- write exactly one task report:
  - `F:\00-Ticenpi-SaaS\W1-FS-A2-PREFLIGHT-REPORT.md`

Forbidden:

- move folders
- rename folders
- delete files/folders
- archive folders
- compress folders
- `git worktree move`
- `git worktree remove`
- `git worktree prune`
- `git clean`
- `git reset`
- `git restore`
- branch deletion
- branch creation
- commit
- push
- rebase
- stash
- checkout/switch that changes working-tree state
- changing `.git` files
- modifying deployment scripts
- modifying documentation other than the one preflight report
- changing environment files
- changing symlinks/junctions
- running deployment
- restarting services
- changing Staging or Production
- database mutation

Do not "fix" anything while investigating it.

If a command might mutate Git/worktree/filesystem state, do not run it.

---

# 2. PHASE A — REVALIDATE A1 SAFETY BASELINE

Read:

`F:\00-Ticenpi-SaaS\W1-FS-A1-AUDIT-REPORT.md`

Extract:

- canonical repository map
- registered worktree map
- dirty repo/worktree list
- high-risk items
- proposed A2 Wave 1/2/3/4 plan
- path-coupled folders
- do-not-touch list
- A2 blockers

Then revalidate only the facts necessary for this preflight.

At minimum rerun safe checks for:

- every canonical repo
- every dirty repo/worktree
- every candidate to be moved in Wave 1 or Wave 2
- `.release-evidence`
- `.release-worktrees`
- `.worktrees`
- deploy tooling
- any folder named as HIGH risk by A1

Do not repeat an expensive full-drive forensic audit unless A1 is clearly stale or incomplete.

Produce a delta section:

```
A1_FACT =
CURRENT_FACT =
CHANGED = YES/NO
IMPACT =
```

for any material difference.

---

# 3. PHASE B — DIRTY / UNIQUE WORK RESOLUTION

This is the most important part.

For every currently dirty repository/worktree, collect:

```
PATH =
MAIN_REPO =
BRANCH =
HEAD =
DETACHED =
STATUS =
TRACKED_MODIFIED =
TRACKED_DELETED =
UNTRACKED =
CONFLICTS =
UPSTREAM =
AHEAD =
BEHIND =
REGISTERED_WORKTREE =
```

Use safe commands only, such as:

```powershell
git -C "<path>" status --short
git -C "<path>" status --branch --short
git -C "<path>" branch --show-current
git -C "<path>" rev-parse HEAD
git -C "<path>" rev-parse --git-common-dir
git -C "<path>" ls-files --others --exclude-standard
git -C "<path>" diff --name-status
git -C "<path>" diff --stat
git -C "<path>" log --oneline --decorate -n 20
git -C "<path>" branch -a --contains <sha>
```

Do not alter working-tree state.

## 3.1 Unique-work determination

For each changed/untracked file, determine whether the content appears to be:

- already committed in the same repository under another branch/ref
- duplicated exactly in another active worktree
- generated/reproducible output
- local configuration
- task evidence
- genuinely unique source/work
- unknown

Use targeted comparison only.

Safe techniques may include:

- file hash comparison
- `git hash-object <file>` **without `-w`**
- `git cat-file -e <object>` where appropriate
- `git log --all -- <path>`
- `git rev-list --all --objects`
- exact diff comparison between worktrees
- comparing same-path content across related branches/worktrees

Do not claim "duplicated" based only on filename.

## 3.2 Risk classification

Assign every dirty repo/worktree:

```
UNIQUE_WORK_RISK =
NONE
LOW
MEDIUM
HIGH
UNKNOWN
```

And separately:

```
A2_MOVE_STATUS =
SAFE_NOW
SAFE_AFTER_COMMIT_OR_BACKUP
SAFE_AFTER_REFERENCE_UPDATE
BLOCKED_UNIQUE_WORK
BLOCKED_UNKNOWN
KEEP_IN_PLACE
```

Important:

- Do not commit the work.
- Do not create a backup copy in this task.
- Do not stash.
- Do not silently downgrade HIGH risk just because a similar branch exists.
- If untracked source exists and cannot be proven duplicated/reproducible, risk must remain HIGH or UNKNOWN.

## 3.3 Recovery provenance

For HIGH/UNKNOWN items, report the minimum action required before relocation:

Examples:

```
REQUIRED_ACTION = commit task-owned work to its intended branch
REQUIRED_ACTION = create explicit backup/patch in separate approved task
REQUIRED_ACTION = identify canonical replacement
REQUIRED_ACTION = user/owner intent required
```

Do not execute the action.

---

# 4. PHASE C — GIT WORKTREE MOVE CAPABILITY

Determine the actual installed Git capability.

Run only non-mutating checks such as:

```powershell
git --version
git worktree -h
git worktree move -h
```

Record:

```
GIT_VERSION =
WORKTREE_MOVE_AVAILABLE = YES/NO
```

Then inspect the registered worktrees and determine which are theoretically relocatable.

For each linked worktree candidate:

```
PATH =
MAIN_REPO =
BRANCH_OR_DETACHED_HEAD =
LOCKED =
PRUNABLE =
HAS_SUBMODULES =
DIRTY =
CURRENT_VOLUME =
TARGET_VOLUME =
WORKTREE_MOVE_COMPATIBLE =
CONSTRAINT =
```

Do not execute `git worktree move`.

## 4.1 Main worktree vs linked worktree

Explicitly distinguish:

- primary/main working tree
- linked worktree
- detached linked worktree
- folder that merely looks like a worktree
- duplicate clone

Do not propose `git worktree move` for something that is not a registered linked worktree.

For main repos, relocation must use a different future strategy.

## 4.2 Windows path considerations

Check whether proposed A2 moves remain on the same drive/volume.

Document risks involving:

- NTFS
- long paths
- junction/symlink use
- file locks
- IDE processes
- Docker bind mounts
- hardcoded Windows paths
- scripts relying on exact root paths

Do not change any Windows setting.

## 4.3 Recommended relocation method

For each candidate, state one future method:

```
RELOCATION_METHOD =
GIT_WORKTREE_MOVE
NORMAL_DIRECTORY_MOVE_AFTER_REFERENCE_UPDATE
KEEP_IN_PLACE
RECREATE_FROM_COMMIT_THEN_RETIRE_OLD_PATH
MANUAL_REVIEW
```

This is a recommendation only.

---

# 5. PHASE D — PATH COUPLING FORENSICS

Search the relevant workspace for references to top-level folder names and absolute paths that A2 might relocate.

Priority paths:

- `F:\00-Ticenpi-SaaS\.release-evidence`
- `F:\00-Ticenpi-SaaS\.release-worktrees`
- `F:\00-Ticenpi-SaaS\.worktrees`
- `F:\00-Ticenpi-SaaS\deploy`
- each canonical repo proposed for future relocation
- each active worktree proposed for future relocation
- each Wave 1 folder proposed by A1

Search likely text sources:

- `*.ps1`
- `*.psm1`
- `*.sh`
- `*.bat`
- `*.cmd`
- `*.yml`
- `*.yaml`
- `*.json`
- `*.toml`
- `*.ini`
- `*.env.example`
- `*.md`
- CI workflows
- compose files
- deployment manifests
- orchestration scripts

Exclude obvious high-noise/generated locations where appropriate:

- `.git`
- `node_modules`
- build/dist caches
- large binary artifacts
- package caches

Do not edit references.

## 5.1 Reference classification

Every meaningful reference must be classified:

```
REFERENCE_PATH =
REFERENCING_FILE =
REFERENCE_TYPE =
  RUNTIME_HARDCODE
  DEPLOY_HARDCODE
  CI_HARDCODE
  DOC_ONLY
  HISTORICAL
  TEST_FIXTURE
  EXAMPLE
  GENERATED
  UNKNOWN

BREAKS_IF_MOVED = YES/NO/UNKNOWN
REQUIRED_UPDATE_OWNER =
```

## 5.2 Critical `.release-evidence` check

Specifically verify the A1 claim that:

`deploy/tools/operator_commands.ps1`

depends on the current `.release-evidence` location.

Determine:

- exact reference
- whether it is absolute or root-relative
- whether other deploy tools use the same convention
- whether release promotion/acceptance gates depend on it
- whether an environment/config override exists
- whether moving it requires a tooling change first

Unless proven otherwise:

```
.release-evidence = KEEP_IN_PLACE
```

for A2.

Do not modify the script.

---

# 6. PHASE E — WAVE 1 SAFETY CHECK

Take the A1 proposed:

`WAVE 1 — unquestionably safe non-Git moves`

and re-evaluate every proposed Wave 1 item.

For each item:

```
PATH =
A1_ACTION =
CURRENT_CLASSIFICATION =
GIT_REPO =
REGISTERED_WORKTREE =
DIRTY =
UNIQUE_WORK_RISK =
PATH_COUPLING =
ACTIVE_PROCESS_OR_TOOL_DEPENDENCY =
SAFE_FOR_A2_WAVE1 =
BLOCKER =
```

A folder can remain in Wave 1 only if all of the following are true:

- not a canonical repo
- not a registered Git worktree
- no unique work
- no active runtime/deploy dependency
- no unresolved hardcoded path dependency
- no release/rollback evidence semantics that would be broken
- target location is defined
- rollback method is trivial and clear

If not, demote it to later wave or REVIEW_REQUIRED.

---

# 7. PHASE F — WAVE 2 WORKTREE CONSOLIDATION CHECK

For every A1 Wave 2 worktree candidate, determine:

```
SOURCE_PATH =
MAIN_REPO =
REGISTERED =
DIRTY =
UNIQUE_WORK_RISK =
BRANCH =
HEAD =
TARGET_ROOT =
TARGET_PATH =
MOVE_CAPABILITY =
PATH_COUPLING =
IDE_OR_SCRIPT_COUPLING =
SAFE_FOR_A2_WAVE2 =
PRECONDITIONS =
```

Preferred future target may be:

`F:\00-Ticenpi-SaaS\.worktrees\<repo-or-product>\<worktree-name>\`

but do not force this if A1 or current evidence supports a different target.

Do not move anything.

---

# 8. PHASE G — WAVE 3 ARCHIVE CHECK

Re-evaluate A1 archive candidates.

Every archive candidate must answer:

```
PATH =
WHY_ARCHIVE =
NOT_GIT_ACTIVE =
NOT_UNIQUE_SOURCE =
NOT_RUNTIME_DEPENDENCY =
NOT_REQUIRED_ACTIVE_RELEASE_EVIDENCE =
RETENTION_REASON =
TARGET_ARCHIVE_PATH =
SAFE_FOR_A2_WAVE3 =
```

Do not archive in this task.

---

# 9. PHASE H — DELETE POLICY

A1 reported zero DELETE_CANDIDATES.

Do not try to manufacture delete candidates merely to reduce clutter.

Default:

```
A2_DELETE_WAVE = DISABLED
```

Only report a future delete candidate if new evidence now satisfies every A1 deletion safety criterion.

Even then:

- do not delete it
- require explicit approval in a separate cleanup wave

---

# 10. PHASE I — DETERMINE WHAT A2 MAY ACTUALLY DO

Build a final action gate with these states:

```
SAFE_A2_WAVE1
SAFE_A2_WAVE2
SAFE_A2_WAVE3
KEEP_IN_PLACE
BLOCKED_DIRTY
BLOCKED_UNIQUE_WORK
BLOCKED_PATH_COUPLING
BLOCKED_GIT_CAPABILITY
BLOCKED_UNKNOWN
```

Every proposed folder operation must have exactly one state.

No ambiguous "probably safe".

---

# 11. A2 EXECUTION DESIGN

If enough blockers are resolved, produce the exact safe execution order for W1-FS-A2.

The order should normally be:

## Wave 0 — pre-move revalidation

Immediately before future A2 mutation:

- re-run status
- re-run worktree list
- re-run path-reference checks for touched paths
- stop if drift is detected

## Wave 1 — safe non-Git moves

Only `SAFE_A2_WAVE1`.

## Wave 2 — Git-aware linked worktree relocation

Only `SAFE_A2_WAVE2`.

Move one worktree at a time.

After each future move, A2 must validate:

- `git worktree list --porcelain`
- repo status
- HEAD
- branch
- common Git dir
- path references
- expected target directory

## Wave 3 — archive consolidation

Only `SAFE_A2_WAVE3`.

## Wave 4 — delete

Disabled unless separately approved.

Do not execute any wave in this task.

---

# 12. AUTONOMOUS INVESTIGATION RULE

Do not ask the user for facts that can be discovered locally.

Examples:

Do not ask:

- "Which dirty worktree matters?"
- "Is this branch still active?"
- "Does operator_commands.ps1 use .release-evidence?"
- "What Git version is installed?"
- "Can worktree move be used?"

Investigate them.

Ask only if the answer depends on undocumented human intent after all objective evidence is exhausted.

If human intent is required:

```
STATUS = BLOCKED_UNKNOWN
DO_NOT_TOUCH = YES
```

and continue with the rest.

---

# 13. REQUIRED REPORT FILE

Write:

`F:\00-Ticenpi-SaaS\W1-FS-A2-PREFLIGHT-REPORT.md`

This report file is the **only filesystem modification allowed in this task**.

Do not place it inside a product repository unless that root itself is a repository and doing so would create untracked source risk. If the root is a Git repo, note that fact and still do not commit the report.

---

# 14. REQUIRED FINAL RESPONSE

Return a concise summary with these exact sections.

## A. RESULT

```
RESULT = PASS / PARTIAL_PASS / HARD_STOP
REPORT = F:\00-Ticenpi-SaaS\W1-FS-A2-PREFLIGHT-REPORT.md
MOVED = 0
DELETED = 0
ARCHIVED = 0
GIT_MUTATIONS = 0
```

## B. DIRTY WORK

```
DIRTY_REPOS_OR_WORKTREES =
HIGH_RISK =
UNKNOWN_RISK =
SAFE_TO_MOVE_AFTER_REVALIDATION =
BLOCKED_UNIQUE_WORK =
```

List every HIGH/UNKNOWN path.

## C. GIT WORKTREE CAPABILITY

```
GIT_VERSION =
WORKTREE_MOVE_AVAILABLE =
REGISTERED_WORKTREES =
MOVE_COMPATIBLE =
MOVE_BLOCKED =
```

## D. PATH COUPLING

```
CRITICAL_PATH_REFERENCES =
.release-evidence = KEEP_IN_PLACE / MOVE_AFTER_TOOLING_CHANGE / UNKNOWN
TOOLING_CHANGE_REQUIRED_BEFORE_MOVE =
```

List exact referencing files.

## E. WAVE READINESS

```
WAVE1_READY = YES/PARTIAL/NO
WAVE1_ITEM_COUNT =

WAVE2_READY = YES/PARTIAL/NO
WAVE2_ITEM_COUNT =

WAVE3_READY = YES/PARTIAL/NO
WAVE3_ITEM_COUNT =

WAVE4_DELETE = DISABLED
```

## F. KEEP IN PLACE

List paths A2 must not relocate.

## G. BLOCKERS

Concrete blockers only.

Each blocker must state:

```
PATH =
BLOCKER =
OWNER =
MINIMUM_RESOLUTION =
```

## H. A2 READINESS

```
A2_READY = YES / PARTIAL / NO
SAFE_OPERATIONS_COUNT =
BLOCKED_OPERATIONS_COUNT =
```

## I. NEXT TASK

If A2 is at least PARTIAL:

```
NEXT_TASK = W1-FS-A2 — Ticenpi Workspace Safe Relocation & Cleanup
```

If unique dirty work requires a separate task before A2, name the smallest prerequisite task(s) first.

Do not execute them.

---

# 15. SUCCESS CONDITION

This preflight is successful when:

> Every dirty repo/worktree has an evidence-based unique-work risk classification, the installed Git/worktree relocation capability is known, every critical path coupling is mapped, `.release-evidence` has an explicit keep/move decision, and the future A2 task has a precise allowlist of operations it may perform without risking source code, Git metadata, release evidence, deployment tooling, or unrecoverable local work.

Do not optimize for making the workspace look clean.

Optimize for making the later cleanup **provably safe**.

Again:

**DO NOT MOVE, DELETE, ARCHIVE, PRUNE, STASH, COMMIT, OR MODIFY SOURCE IN THIS PREFLIGHT.**
