# OCR Letter Editor — SAFE Revert V6 to V5 + Create Local Checkpoint

## OWNER

OCR LETTER EDITOR RECOVERY OWNER

## MODEL

GPT-5.6 Luna — High

## OBJECTIVE

Return the current OCR Letter working tree to the exact V5 state that existed immediately BEFORE the V6 task started.

Then create a LOCAL Git checkpoint commit for that restored V5 baseline so future tasks can roll back safely.

This is a recovery task.

Accuracy and preservation are more important than speed.

---

# CANONICAL REPO

F:\00-Ticenpi-SaaS\TicenpiLetter

Expected branch:

master

Expected HEAD before all uncommitted V2/V3/V4/V5/V6 work:

4aa0fa90693347065411acf75b9f26524200d65a

IMPORTANT:

V2/V3/V4/V5 changes were intentionally left uncommitted.
V6 was then implemented on top of that same dirty working tree.

Therefore:

DO NOT revert to HEAD.

The goal is:

current V6 working tree
→ remove ONLY V6-owned edits
→ preserve ALL V2/V3/V4/V5 edits
→ validate restored V5 state
→ create a LOCAL checkpoint commit
→ DO NOT PUSH

---

# ABSOLUTE SAFETY RULES

The following commands/actions are FORBIDDEN:

- git reset --hard
- git reset --merge
- git reset --keep
- git restore .
- git restore --source=HEAD .
- git checkout -- .
- git checkout HEAD -- .
- git clean
- git clean -f
- git clean -fd
- git clean -fdx
- deleting untracked files
- mass replacing files from HEAD
- reverting entire files merely because V6 touched them
- force checkout
- force branch switching
- git stash pop/apply unless the stash is positively proven to be the exact V5 baseline
- committing before V5 restoration is proven
- pushing

Do NOT use file-level restore if a file contains both V5 and V6 edits.

Do NOT guess.

Do NOT infer V5 content from HEAD.

---

# WHY THIS MUST BE HUNK-LEVEL

The same files may contain edits from multiple generations:

V2
V3
V4
V5
V6

Example:

frontend/src/views/LetterView.vue

may contain valid V5 work plus bad V6 changes.

Therefore:

restoring the whole file from HEAD
=
DESTROYING valid V2–V5 work.

This task must identify and reverse only V6-owned hunks.

---

# AUTHORITATIVE V5 STATE INFORMATION

Immediately after V5 FINAL, the report stated:

Path:
F:\00-Ticenpi-SaaS\TicenpiLetter

Branch:
master

HEAD:
4aa0fa90693347065411acf75b9f26524200d65a

Tracked dirty before V5:
6

Tracked dirty after V5:
11

Files changed by V5 were reported as including:

- LetterView
- render-front
- render-back
- render-strip
- export-pdf
- styles
- vite.config
- property-objects

V5 functionality reported:

- V4 baseline preserved
- hover tooltips PASS
- envelope-info duplicate Letter Strip controls removed
- Letter Strip source count = generated count
- A4 Letter Strip renderer added
- Letter Strip memory-only lifecycle preserved
- DM-compatible property extraction working
- property preview/frame selection working
- front property insertion working
- property object select/move/resize/z-order/delete/reset working
- property persistence wiring added
- tests 36/36 PASS
- build PASS

V5 was NOT committed.

This report is a validation aid only.
It is NOT sufficient by itself to reconstruct exact file contents.

---

# V6 TASK IDENTITY

The V6 prompt was:

OCR Letter Editor V6 — FrontBelow Root Cause + Grid Property Layout + RemoveBG Wiring Repair

GitHub prompt commit:

b1251750786067ae94f726b578c82c23caf3f451

V6 attempted work around:

- frontBelow root-cause changes
- bodyTextSlot / portraitSlot
- WYSIWYG modal changes
- independent 案件物件 icon
- property grid layout
- RemoveBG wiring changes

The user has rejected the V6 result and wants ALL V6 changes removed.

Do NOT keep a V6 change merely because it appears technically reasonable.

Target is V5-before-V6.

---

# PHASE 0 — FREEZE THE CURRENT STATE

Before ANY revert:

1. stop dev editing
2. capture:
   - git branch --show-current
   - git rev-parse HEAD
   - git status --short
   - git diff --stat HEAD
   - git diff --name-status HEAD
   - git diff --binary HEAD

3. record:
   - tracked modified files
   - staged files
   - untracked files

Do not modify anything yet.

---

# PHASE 1 — CREATE A NON-DESTRUCTIVE SAFETY BACKUP

Create a recovery folder OUTSIDE the repo, for example:

F:\00-Ticenpi-SaaS\_recovery\TicenpiLetter-v6-before-revert-<timestamp>\

Store:

1. full tracked diff from HEAD:

current-working-tree.patch

using a binary-capable diff so image/binary changes are recoverable where possible.

2. git status:

git-status.txt

3. changed-file list:

changed-files.txt

4. untracked-file list:

untracked-files.txt

5. copy every currently untracked file that belongs to this repo into:

untracked-backup\

preserving relative paths.

6. if staged changes exist, also record:

git diff --cached --binary HEAD

separately.

Verify the backup files are non-empty where expected.

This backup is recovery insurance only.

Do NOT restore from it during the normal revert unless needed.

---

# PHASE 2 — FIND AN AUTHORITATIVE V6 EDIT HISTORY

Use the strongest available source in this order:

## Preferred A — Same Agent / IDE session edit history

If this is the same agent/session that performed V6:

use its exact edit/apply-patch history.

Identify every edit performed AFTER the V6 task began.

This is the preferred source.

## Preferred B — IDE/local-history snapshots

If the IDE stores local history or pre-edit snapshots:

locate snapshots created immediately before V6 edits.

Use timestamps and file identity carefully.

## Preferred C — Explicit V6 patch/diff recorded by the prior agent

If the V6 agent saved or exposed exact patch hunks:

use those exact hunks.

---

# HARD STOP IF V6 OWNERSHIP CANNOT BE PROVEN

If there is NO authoritative way to distinguish:

V5-owned hunks
from
V6-owned hunks

DO NOT attempt the revert by intuition.

DO NOT use current-vs-HEAD diff to guess.

DO NOT reconstruct V5 from prompt requirements.

STOP and return:

SAFE_REVERT_BLOCKED

with:

- exact files where V5/V6 ownership is ambiguous
- evidence sources checked
- whether IDE/session history exists
- whether any pre-V6 snapshot exists
- safest next recovery options

Preserving V5 is more important than forcing a revert.

---

# PHASE 3 — BUILD A V6-ONLY REVERT PLAN

Before editing, produce an internal table:

FILE
V6-OWNED HUNKS
V5-OWNED HUNKS PRESENT?
REVERT METHOD
RISK

For every file V6 touched.

Rules:

- if file contains V5 + V6:
  revert ONLY V6 hunks
- if file was created only by V6:
  remove it ONLY after proving it did not exist in V5
- if file existed before V6:
  never delete it
- if file was untracked before V6:
  preserve it
- if ownership is uncertain:
  HARD STOP for that file

Do not proceed until every targeted V6 change has a provenance.

---

# PHASE 4 — REVERSE ONLY V6 HUNKS

Use one of:

- IDE/session undo of exact V6 edits
- inverse patch of exact V6 patch
- manually restore exact pre-V6 hunks from authoritative snapshot

Do NOT restore entire mixed-history files.

After EACH file:

- inspect diff
- confirm valid V5 hunks remain
- confirm only identified V6 hunks disappeared

Do not batch blindly.

---

# PHASE 5 — VERIFY THE RESULT LOOKS LIKE V5, NOT HEAD

After V6 revert:

confirm:

HEAD remains:

4aa0fa90693347065411acf75b9f26524200d65a

The worktree should still be DIRTY because V2–V5 are intentionally preserved.

Expected:

V5 functionality remains.

At minimum browser-check:

- OCR home /
- /letter
- /ocr absent
- hover tooltips
- LOGO resize
- LINE QR resize
- sticker transform
- rich text modal
- IndexedDB F5 restore
- Envelope Info without duplicate Letter Strip controls
- independent 開發信條 icon
- Letter Strip current-data count behavior
- DM-compatible property extraction
- V5 property frame/object behavior

Do NOT expect V6-only behavior to remain.

Specifically verify V6 rejected changes are gone where applicable:

- no V6 property-grid replacement if that did not exist in V5
- no V6 bodyTextSlot/portraitSlot restructuring if introduced only by V6
- no V6 RemoveBG wiring changes if introduced only by V6
- no V6 frontBelow structural rewrites if introduced only by V6

---

# PHASE 6 — VALIDATION BEFORE COMMIT

Run:

git diff --check

Run the existing frontend tests.

Expected baseline:

36/36 PASS or better

Run frontend production build.

PASS required.

If tests/build fail because V5 itself already had a known issue:

report exact evidence.

Do NOT repair unrelated functionality during this recovery task.

---

# PHASE 7 — CREATE A LOCAL V5 CHECKPOINT COMMIT

ONLY after:

V5_RESTORED = YES

and validation is complete.

Create a LOCAL Git commit.

Recommended message:

checkpoint: restore OCR Letter V5 baseline before V6

This commit is intentionally a checkpoint, not a claim that every V5 feature had final production acceptance.

The commit should include the complete restored V2–V5 working tree that is now accepted as the rollback baseline.

Before commit:

- inspect git status
- inspect diff --stat
- ensure no V6-only changes remain
- ensure no unrelated files were accidentally added
- do NOT use git add -A blindly

Stage only the verified V5 baseline files.

Then commit locally.

DO NOT PUSH.

After commit:

- capture new commit SHA
- git status --short
- verify working tree is clean, except any explicitly preserved unrelated/untracked files that were intentionally not part of V5

---

# NEW GIT POLICY — APPLY FROM NOW ON

This recovery happened because multiple accepted milestones were left on one uncommitted working tree.

From now on use this policy:

## Before a new major task

If the current state is an accepted baseline:

create a LOCAL checkpoint commit before starting the next risky task.

Do not push automatically.

## At the end of a task

If FINAL STATUS = PASS:

- run tests/build
- create a LOCAL commit automatically
- report commit SHA
- do NOT push unless explicitly approved

If FINAL STATUS = BLOCKED or FAIL:

- do NOT commit over the accepted baseline
- either revert the task changes
- or preserve them on a separate WIP branch only with explicit approval

## Push policy

LOCAL COMMIT:
automatic after accepted PASS/checkpoint

PUSH:
requires explicit user approval unless the task explicitly says otherwise

This prevents another V5/V6 rollback ambiguity.

---

# FORBIDDEN SHORTCUTS

Never use:

"V5 = HEAD"

False.

Never use:

"V5 touched these files, so restore those files"

Unsafe.

Never use:

"git diff HEAD shows V6"

False — it shows V2+V3+V4+V5+V6.

Never use:

"rebuild V5 from the V5 prompt"

Unsafe and non-identical.

Never use:

"the file seems like V6"

Insufficient provenance.

Exact edit ownership is required.

---

# FINAL REPORT

Return exactly:

OCR V6 → V5 SAFE REVERT FINAL

1. SOURCE
Repo:
Branch:
HEAD before:
HEAD after revert:
Current timestamp:

2. SAFETY BACKUP
Recovery folder:
Tracked patch:
Staged patch:
Untracked backup:
Backup verified:

3. V6 PROVENANCE
Authoritative source used:
Same-session edit history:
IDE local history:
Saved V6 patch:
Ambiguous files:

4. V6 REVERT
V6 files identified:
V6 hunks reverted:
V6-only files removed:
Mixed V5/V6 files handled hunk-by-hunk:
Whole-file restore used = NO
git reset --hard used = NO
git clean used = NO

5. V5 PRESERVATION
V2–V5 edits preserved:
V5 hover tooltip:
V5 Letter Strip:
V5 property extraction/object:
V5 persistence:
Other V5 behavior:

6. VALIDATION
git diff --check:
Frontend tests:
Build:
Browser smoke:

7. LOCAL CHECKPOINT
V5_RESTORED:
Checkpoint commit created:
Checkpoint SHA:
Commit message:
Pushed = NO

8. FINAL WORKTREE
git status:
Untracked files intentionally preserved:

9. FINAL STATUS
PASS / SAFE_REVERT_BLOCKED / FAIL
