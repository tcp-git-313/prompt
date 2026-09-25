# W1-FS-A1 — Ticenpi Workspace / Worktree / Evidence Cleanup Audit

## ROLE

You are the **TICENPI WORKSPACE INVENTORY & CLEANUP AUDIT OWNER**.

Your task is to audit the filesystem layout under:

`F:\00-Ticenpi-SaaS\`

and determine exactly which folders are:

- canonical repositories
- active Git worktrees
- release worktrees
- temporary deployment source checkouts
- release evidence
- generated artifacts
- incident/recovery material
- historical/manual scratch folders
- safe archive candidates
- possible delete candidates
- unknown and therefore **DO NOT TOUCH**

This is **A1: READ-ONLY AUDIT ONLY**.

Do **not** move, rename, delete, prune, clean, reset, rebase, archive, compress, or otherwise mutate any folder or repository in this task.

A later task, **W1-FS-A2**, will perform approved moves/cleanup using the A1 report.

---

# 0. PRIMARY GOAL

The current root directory has become crowded with product repos, worktrees, temporary folders, evidence, and historical repair directories mixed together.

The goal of A1 is to produce a trustworthy map of what every top-level folder actually is, so a later cleanup can safely converge the workspace into a structure such as:

```
F:\00-Ticenpi-SaaS\
│
├─ products\
│  ├─ TicenpiPost\
│  ├─ TicenpiDM\
│  ├─ TicenpiORC\
│  ├─ TicenpiSign\
│  ├─ Ticenpi591\
│  └─ TicenpiLetter\
│
├─ platform\
│  ├─ ticenpi-platform\
│  └─ Ticenpi-Launcher\
│
├─ shared\
│  └─ ExtractionHub\
│
├─ ops\
│  └─ deploy\
│
├─ .worktrees\
├─ .release-evidence\
├─ .artifacts\
└─ archive\
```

This is only a **target concept**, not permission to move anything.

Do not force the existing workspace into this exact shape if the evidence shows another structure is safer or more appropriate.

---

# 1. ABSOLUTE SAFETY RULES

## 1.1 Read-only means read-only

Forbidden in A1:

- `Move-Item`
- `Rename-Item`
- `Remove-Item`
- `rmdir`
- `del`
- `git clean`
- `git reset`
- `git checkout -- .`
- `git restore`
- `git worktree remove`
- `git worktree prune`
- `git branch -D`
- `git gc`
- `git rebase`
- deleting temp folders
- changing `.git` files
- modifying worktree registrations
- changing junctions/symlinks
- changing deployment evidence
- changing source code
- changing documentation
- committing anything
- deploying anything

Do not "clean up while investigating."

## 1.2 Never infer identity from folder name alone

Examples:

```
.dm-deploy-src-3fab2f2-pin
TicenpiDM_release_wt
Ticenpi-Launcher-w2-f1
deploy-w2-post
.recovery
.studio-build
```

may be:

- Git worktrees
- detached checkouts
- copied repos
- release staging areas
- temporary build directories
- historical evidence
- active dependency of another script

You must prove what they are before classifying them.

## 1.3 Unknown = do not touch

If a folder cannot be confidently classified:

```
CLASSIFICATION = UNKNOWN_DO_NOT_TOUCH
```

Do not guess.

---

# 2. KNOWN ROOT EXAMPLES — VERIFY THEM

The root has recently contained folders similar to:

```
.dm-deploy-src-3fab2f2-pin
.dm-deploy-src-4bda9be
.dm-deploy-src-d9a227e
.dm-deploy-src-e026756
.incident-cache
.incident-worktrees
.release-evidence
.release-worktrees
.staging-orchestration
.worktrees
.wrangler
_frontend-style-edits
_recovery
_studio-build
artifacts
deploy
deploy-w2-post
dm-commercial-audit-prompts
dm-ui-history-preview
ExtractionHub
hub-workflow-verification
post-local-extension-repair
Ticenpi591
TicenpiDM
TicenpiDM_release_wt
Ticenpi-Geo
Ticenpi-Launcher
Ticenpi-Launcher-w2-f1
Ticenpi-Launcher-w2-launcher-f1
TicenpiLetter
```

There may be more folders than this list.

Enumerate the actual current top level. Do not limit the audit to these examples.

---

# 3. PHASE A — TOP-LEVEL INVENTORY

Enumerate every direct child of:

`F:\00-Ticenpi-SaaS\`

For each item capture:

- exact full path
- name
- type: directory/file/junction/symlink
- last modified time
- approximate size if practical without excessive cost
- hidden/system status when relevant
- whether it contains:
  - `.git\`
  - a `.git` file
  - `package.json`
  - `pyproject.toml`
  - `requirements*.txt`
  - `docker-compose*.yml`
  - `Dockerfile`
  - deployment scripts
  - CI workflows
  - release evidence
  - obvious generated output
- whether it appears nested under another registered Git repository/worktree

Do not recursively hash the entire drive.

Use targeted inspection.

---

# 4. PHASE B — GIT REPOSITORY / WORKTREE FORENSICS

For every folder that may be a Git repository or worktree, determine:

```
PATH =
GIT_TOPLEVEL =
GIT_DIR =
COMMON_GIT_DIR =
REMOTE =
BRANCH =
HEAD =
DETACHED =
DIRTY =
UNTRACKED_COUNT =
WORKTREE_REGISTERED =
WORKTREE_MAIN_REPO =
WORKTREE_LOCKED =
WORKTREE_PRUNABLE =
```

Use safe commands such as:

```powershell
git -C "<path>" rev-parse --show-toplevel
git -C "<path>" rev-parse --git-dir
git -C "<path>" rev-parse --git-common-dir
git -C "<path>" remote -v
git -C "<path>" branch --show-current
git -C "<path>" rev-parse HEAD
git -C "<path>" status --short
git -C "<path>" worktree list --porcelain
```

For a `.git` **file**, inspect its text to determine the referenced Git dir.

Do not edit it.

For each canonical/main repo, run:

```
git worktree list --porcelain
```

and map every registered worktree back to its actual path.

You must detect:

- active registered worktrees
- detached worktrees
- stale-looking folders that are still registered
- folders that look like worktrees but are not registered
- duplicate clones with the same remote
- source snapshots that are not Git repos
- nested repositories

Do not call a worktree stale merely because its branch name is old.

---

# 5. PHASE C — CANONICAL REPO DETECTION

Determine which path is the likely canonical working repository for each product/component.

Minimum components to detect if present:

- Post
- DM
- ORC
- Sign
- 591
- Letter
- Launcher
- Platform / Commercial Core
- ExtractionHub
- deploy / release tooling
- Geo or other active products discovered

For each component report:

```
COMPONENT =
CANONICAL_REPO_PATH =
REMOTE =
DEFAULT_BRANCH =
CURRENT_BRANCH =
HEAD =
WHY_CANONICAL =
OTHER_RELATED_PATHS =
CONFIDENCE = HIGH/MEDIUM/LOW
```

Do not use folder naming as the only evidence.

Prefer evidence such as:

- expected git remote
- default/main branch relationship
- commit history
- active documentation references
- deployment script references
- current worktree relationships
- release evidence references
- environment manifests

If `TICENPI_SYSTEM_CURRENT_STATE.md`, `SYSTEM_OWNERSHIP.md`, or similar canonical architecture docs exist from W1-ARCH-A1, read them as supporting evidence, but verify filesystem facts independently.

---

# 6. PHASE D — CLASSIFICATION

Assign every top-level folder exactly one primary classification:

```
CANONICAL_REPO
ACTIVE_WORKTREE
RELEASE_WORKTREE
TEMP_DEPLOY_SOURCE
RUNTIME_EVIDENCE
GENERATED_ARTIFACT
BUILD_CACHE
INCIDENT_RECOVERY
HISTORICAL_REFERENCE
ARCHIVE_CANDIDATE
DELETE_CANDIDATE
KEEP_ROOT_INFRA
UNKNOWN_DO_NOT_TOUCH
```

Also assign:

```
ACTIVE_DEPENDENCY = YES/NO/UNKNOWN
SAFE_TO_MOVE_LATER = YES/NO/CONDITIONAL/UNKNOWN
SAFE_TO_ARCHIVE_LATER = YES/NO/CONDITIONAL/UNKNOWN
SAFE_TO_DELETE_LATER = YES/NO/CONDITIONAL/UNKNOWN
```

Important:

`DELETE_CANDIDATE` does **not** mean delete now.

A folder can only be classified as `DELETE_CANDIDATE` if there is strong evidence that:

- it is not a registered worktree
- it is not the only copy of uncommitted work
- it is not referenced by deployment scripts
- it is not required release evidence
- it is not a source of unreleased work
- it is not the only rollback/evidence copy
- it is not referenced by current docs/scripts
- it has an identifiable replacement/canonical copy

If any of these are uncertain, use `ARCHIVE_CANDIDATE` or `UNKNOWN_DO_NOT_TOUCH` instead.

---

# 7. PHASE E — DIRTY / UNIQUE WORK PROTECTION

This is critical.

For every repo/worktree that is dirty or has untracked files, determine:

- changed file count
- untracked file count
- whether those changes exist elsewhere
- whether commits containing the same work exist in another branch/repo
- whether this path may contain unique unrecoverable work

Classify:

```
UNIQUE_WORK_RISK =
NONE
LOW
MEDIUM
HIGH
UNKNOWN
```

Any `HIGH` or `UNKNOWN` unique-work risk must automatically block delete recommendations.

Do not diff huge binary/generated folders unless needed.

Do not modify anything.

---

# 8. PHASE F — REFERENCE SEARCH

Before recommending that a folder can later be moved or archived, search the workspace for references to its exact path or folder name.

Check likely reference sources:

- PowerShell
- shell scripts
- YAML
- JSON
- TOML
- Markdown
- system/deploy manifests
- compose files
- CI workflows
- release scripts
- local orchestration scripts

Examples:

```
F:\00-Ticenpi-SaaS\TicenpiDM_release_wt
F:\00-Ticenpi-SaaS\.release-worktrees
deploy-w2-post
.dm-deploy-src-
```

Classify path coupling:

```
PATH_COUPLING = NONE / LOW / MEDIUM / HIGH / UNKNOWN
```

If scripts hardcode an exact path, later relocation must include an explicit reference update plan.

Do not update those references in A1.

---

# 9. PHASE G — EVIDENCE / ARTIFACT RETENTION

Audit these areas carefully if present:

- `.release-evidence`
- `artifacts`
- `.release-worktrees`
- deployment source snapshots
- rollback backups
- incident evidence
- staging orchestration output

For evidence/artifacts determine:

```
PURPOSE =
OWNER =
DATE_RANGE =
REFERENCED_BY_RELEASE =
UNIQUE =
RETENTION_VALUE =
MOVE_RISK =
```

Do not treat old evidence as useless merely because it is old.

Release/rollback evidence may need to remain accessible even after repository cleanup.

Recommend a retention model such as:

```
.release-evidence\
  <product>\
    <release-id>\

archive\
  incidents\
  historical-worktrees\
  obsolete-snapshots\
```

but do not create or move anything in A1.

---

# 10. PHASE H — TARGET WORKSPACE DESIGN

After classification, propose a final workspace design.

Prefer a small number of durable top-level categories.

The design should clearly separate:

1. canonical source repositories
2. active worktrees
3. shared/platform components
4. deployment/operations
5. release evidence
6. generated artifacts
7. archives

For example:

```
F:\00-Ticenpi-SaaS\
├─ products\
├─ platform\
├─ shared\
├─ ops\
├─ .worktrees\
├─ .release-evidence\
├─ .artifacts\
└─ archive\
```

But adapt to actual evidence.

Avoid unnecessarily deep nesting.

Do not relocate canonical repos just for visual cleanliness if the relocation would break tooling and provides little value.

---

# 11. PHASE I — A2 MOVE PLAN

Produce a future move/cleanup plan, but do not execute it.

Every proposed operation must be one of:

```
KEEP
MOVE_LATER
ARCHIVE_LATER
DELETE_LATER_CANDIDATE
REVIEW_REQUIRED
```

For each proposed move report:

```
SOURCE =
TARGET =
TYPE =
WHY =
GIT_WORKTREE = YES/NO
PATH_REFERENCES =
PRECONDITIONS =
SAFE_METHOD =
ROLLBACK_METHOD =
```

If it is a registered Git worktree:

- do not recommend Explorer cut/paste as the primary method
- specify a Git-aware relocation strategy
- verify the installed Git version supports the intended method before A2
- if `git worktree move` is unsuitable for the case, specify the safe alternative
- ensure main repo worktree metadata remains consistent

If it is a normal canonical repository clone:

- path references must be updated in the A2 plan
- remote/branch/dirty state must be rechecked immediately before move

If it is evidence:

- preserve identity and provenance
- do not rename release IDs casually

---

# 12. NO USER QUESTIONS FOR DISCOVERABLE FACTS

Do not ask the user:

- "Which folder is the real DM repo?"
- "Is this a worktree?"
- "Can I delete this?"
- "What is deploy-w2-post?"

until you have exhausted discoverable local evidence.

Investigate first.

Only ask the user if:

- two candidates remain genuinely indistinguishable
- the difference depends on undocumented human intent
- proceeding with classification would otherwise risk data loss

Even then, leave the item as `UNKNOWN_DO_NOT_TOUCH` and continue auditing the rest.

---

# 13. REQUIRED FINAL REPORT

Return exactly these sections.

## A. RESULT

```
RESULT = PASS / PARTIAL_PASS / HARD_STOP
ROOT = F:\00-Ticenpi-SaaS
MUTATIONS = NONE
```

## B. ROOT SUMMARY

```
TOP_LEVEL_ITEMS =
GIT_REPOS =
REGISTERED_WORKTREES =
DIRTY_REPOS_OR_WORKTREES =
UNKNOWN_ITEMS =
ARCHIVE_CANDIDATES =
DELETE_CANDIDATES =
```

## C. CANONICAL REPOSITORIES

Table:

| Component | Canonical Path | Remote | Branch | HEAD | Dirty | Confidence |
|---|---|---|---|---|---|---|

## D. WORKTREE MAP

Table:

| Path | Main Repo | Branch/HEAD | Registered | Dirty | Locked | Purpose | Risk |
|---|---|---|---|---|---|---|---|

## E. COMPLETE TOP-LEVEL CLASSIFICATION

Every top-level folder must appear exactly once.

Table:

| Folder | Classification | Active Dependency | Move Later | Archive Later | Delete Candidate | Unique Work Risk | Evidence |
|---|---|---:|---:|---:|---:|---|---|

## F. PATH COUPLING

List every folder whose current absolute path is referenced by source/docs/scripts.

## G. HIGH-RISK ITEMS

Only items where moving/deleting could break:

- Git
- deploy
- release
- rollback
- evidence
- uncommitted work
- unique source

## H. PROPOSED FINAL STRUCTURE

Show the recommended directory tree.

## I. A2 OPERATION PLAN

Ordered future operations.

Separate into:

```
WAVE 1 — unquestionably safe moves
WAVE 2 — Git-aware worktree relocation
WAVE 3 — archive candidates
WAVE 4 — delete candidates after explicit approval
```

Do not execute any wave.

## J. DO-NOT-TOUCH

Explicitly list every folder that A2 must not touch until another prerequisite is satisfied.

## K. A2 READINESS

```
A2_READY = YES / PARTIAL / NO
BLOCKERS =
```

## L. NEXT TASK

If A2 is ready or partially ready, provide the exact recommended next task name:

```
W1-FS-A2 — Ticenpi Workspace Safe Relocation & Cleanup
```

Do not execute A2.

---

# 14. SUCCESS CONDITION

A1 is successful when:

> Every top-level item under `F:\00-Ticenpi-SaaS` has been identified using filesystem/Git/reference evidence rather than folder-name guesses, all active worktrees and unique dirty work are protected, canonical repositories are clearly identified, path coupling is documented, and a later cleanup can be executed without risking source code, release evidence, Git metadata, or deployment tooling.

Do not stop at a visual folder list.

Perform the repository/worktree/reference investigation necessary to justify the classification.

Again: **READ-ONLY. DO NOT MOVE OR DELETE ANYTHING IN A1.**
