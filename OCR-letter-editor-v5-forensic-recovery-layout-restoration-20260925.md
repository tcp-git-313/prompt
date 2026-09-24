# OCR Letter Editor — V5 Forensic Recovery + Layout Restoration

## OWNER

OCR LETTER EDITOR V5 FORENSIC RECOVERY OWNER

## MODEL

GPT-5.6 Luna — High

## OBJECTIVE

Recover the OCR Letter Editor to the TRUE accepted V5 state, prove that the recovered source is really V5, restore the accepted V5 layout/behavior, and only then create a clean LOCAL checkpoint commit.

This task REPLACES both of these older recovery prompts:

- SAFE Revert V6 to V5 + Create Local Checkpoint
- Restore Accepted V5 Layout Safely

Do NOT execute those older prompts separately.

The current layout is reported as broken/misaligned, and the previously created "V5 checkpoint" may itself be wrong.

Therefore:

DO NOT TRUST ANY EXISTING V5 CHECKPOINT UNTIL IT IS FORENSICALLY VERIFIED.

---

# CANONICAL REPO

F:\00-Ticenpi-SaaS\TicenpiLetter

Expected branch:

master

Historical old HEAD before the long uncommitted V2/V3/V4/V5/V6 chain:

4aa0fa90693347065411acf75b9f26524200d65a

CRITICAL:

4aa0fa9 is NOT the accepted V5 baseline.

V2/V3/V4/V5 were developed as dirty working-tree changes on top of that old HEAD.

Never assume:

V5 == 4aa0fa9

That is false.

---

# CORE RECOVERY PRINCIPLE

The recovery target is:

the exact source state that existed immediately after V5 FINAL
and immediately before V6 modifications began.

The recovery source must come from authoritative evidence such as:

1. verified local Git checkpoint created from the real V5 tree
2. pre-V6 IDE/local-history snapshot
3. same-agent/session edit history
4. exact pre-V6 patch/snapshot
5. recovery folder created before V6 revert attempts
6. exact V5 working-tree patch

The following are NOT authoritative enough by themselves:

- prompt prose
- screenshots
- current broken UI
- file names listed in FINAL report
- old HEAD
- a commit merely named "V5 checkpoint"

---

# CURRENT RISK

A previous recovery attempt may have:

- reverted the wrong hunks
- restored whole mixed-history files
- created a checkpoint from an already-broken layout
- preserved V6 contamination
- removed valid V5 layout code
- mixed V5 and V6 state

Therefore existing recovery commits/checkpoints must be treated as CANDIDATES, not truth.

---

# ABSOLUTE SAFETY RULES

Before V5 provenance is proven, FORBIDDEN:

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
- whole-file restore from old HEAD
- force branch switching
- mass search/replace
- manually rebuilding the whole layout from prose
- creating a new checkpoint
- commit
- push

Do not mutate the repo until the forensic inventory is complete.

---

# PHASE 0 — FREEZE CURRENT STATE

Before any mutation:

cd F:\00-Ticenpi-SaaS\TicenpiLetter

Capture:

- git branch --show-current
- git rev-parse HEAD
- git status --short
- git diff --stat HEAD
- git diff --name-status HEAD
- git diff --binary HEAD
- git diff --cached --binary HEAD
- git log --oneline --decorate -n 40
- git reflog --date=iso -n 80

Record:

- tracked modified files
- staged files
- untracked files
- current HEAD
- current branch

Do not edit yet.

---

# PHASE 1 — CREATE COMPLETE NON-DESTRUCTIVE BACKUP

Create a recovery folder OUTSIDE the repo:

F:\00-Ticenpi-SaaS\_recovery\TicenpiLetter-forensic-before-v5-recovery-<timestamp>\

Store:

1. current-working-tree.patch
   - full binary-capable diff against current HEAD

2. staged-working-tree.patch
   - if staged changes exist

3. git-status.txt

4. changed-files.txt

5. git-log.txt

6. git-reflog.txt

7. untracked-files.txt

8. untracked-backup\
   - copy every current untracked repo file
   - preserve relative paths

9. current-file-hashes.txt
   - SHA256 for every tracked dirty file and every untracked file

Verify backups exist and are readable.

Do not continue until backup verification succeeds.

---

# PHASE 2 — FORENSICALLY INVENTORY ALL CANDIDATE V5 SOURCES

Search and classify every possible V5 recovery source.

## A. Git commits

Search:

git log --all --oneline --decorate --grep="V5"
git log --all --oneline --decorate --grep="checkpoint"
git log --all --oneline --decorate --grep="OCR Letter"
git log --all --oneline --decorate --grep="restore"

For each candidate:

- SHA
- timestamp
- parent
- commit message
- files changed
- diffstat
- whether it was created before or after the failed V6/recovery attempts
- whether it contains V5 UI/layout code or merely recovery/prompt metadata

Do NOT trust commit name alone.

## B. Reflog

Identify:

- HEAD movement around V5 completion
- checkpoint creation
- V6 start
- prior recovery attempts
- resets/checkouts if any

## C. IDE local history / local snapshots

Inspect the IDE/local-history source used during V5/V6 work.

Look for snapshots immediately:

- after V5 FINAL
- before V6 edits
- before the first failed V6→V5 recovery attempt

## D. Agent/session edit history

If the same Agent/IDE session retains edit history:

identify exact V5-end and V6-start boundaries.

## E. Recovery directories

Search under:

F:\00-Ticenpi-SaaS\_recovery\

for previous recovery artifacts including:

- pre-V6 patch
- pre-revert patch
- untracked backups
- status snapshots
- file copies

## F. Saved patches

Search repo parent/workspaces for:

- *.patch
- *.diff
- backup directories
- task snapshots

Do not modify anything while gathering this evidence.

---

# PHASE 3 — BUILD V5 PROVENANCE MATRIX

Create an internal matrix:

SOURCE
TIMESTAMP
BEFORE_V6?
COMPLETE_WORKTREE?
INCLUDES_UNTRACKED?
FILES COVERED
LAYOUT FILES COVERED
TRUST LEVEL
NOTES

Candidate trust levels:

A = exact pre-V6 full working-tree source
B = exact pre-V6 partial source with strong complementary evidence
C = likely V5 but incomplete
D = ambiguous / unsafe

Only A, or a provably complete combination of A/B sources, may be used to restore automatically.

---

# PHASE 4 — IDENTIFY HIGH-RISK LAYOUT FILES

Pay special attention to files reported or likely touched across V5/V6:

- frontend/src/views/LetterView.vue
- front renderer file(s)
- back renderer file(s)
- strip renderer file(s)
- export-pdf file(s)
- style/CSS file(s)
- frontend/vite.config.js
- property-object file(s)
- letter persistence file(s)
- project-file serializer file(s)
- image asset helper(s)
- router/index.js
- PickerView.vue

Do not assume exact paths from names.

Locate actual files in the current repo.

For each high-risk file determine:

- did it exist in V5?
- did V5 modify it?
- did V6 modify it?
- did failed recovery attempts modify it?
- can exact V5 content be proven?

---

# PHASE 5 — VERIFY ANY EXISTING "V5 CHECKPOINT" BEFORE USING IT

If a checkpoint candidate exists, DO NOT RESET TO IT YET.

First:

1. inspect commit diff
2. inspect parent diff
3. compare relevant files to pre-V6/local-history evidence
4. compare key V5 behaviors against V5 FINAL report
5. compare layout source with V5-era screenshots/evidence where available

A candidate V5 checkpoint is VERIFIED only if:

- timestamp/provenance is compatible with V5 end / V6 start
- it contains the V2–V5 accepted working tree
- it does not contain rejected V6 structural changes
- it has not lost known V5 functions
- high-risk files match authoritative pre-V6 evidence

If any of these fail:

mark checkpoint:

UNTRUSTED

Do not use it as restore target.

---

# AUTHORITATIVE V5 FINAL FUNCTIONAL EVIDENCE

Use this as behavior validation, not exact source reconstruction.

V5 FINAL reported:

## V4 baseline preserved

- LOGO resize regression PASS
- QR resize regression PASS
- Sticker transform regression PASS
- Normal Letter F5 regression PASS
- 開發信條 memory-only regression PASS

## Hover

All tooltips PASS.

Reported implementation details:

- rail overflow: visible
- hover delay: 400ms
- tooltip z-index: 1000

## Envelope info

- duplicate Letter Strip controls removed
- sender fields PASS
- printed-material toggle PASS
- surname / 貴住戶 PASS
- Envelope setting independent from Letter Strip setting

## Letter Strip

Authoritative source:

letterBatch.recipients valid address data

V5 validation example:

source count = 2
generated count = 2

Important rule:

actual valid source N
=
generated N

A4:

210mm × 297mm

Screen/PDF used shared renderStrip path.

Reported actual strips/page in that specific test:

18

but this was dynamically estimated and must NOT be treated as a permanent fixed design token.

Lifecycle:

- memory-only
- close/reopen preserved
- tool switch preserved
- F5 clears temporary Letter Strip state
- no localStorage
- no sessionStorage
- no IndexedDB
- no backend/VPS
- excluded from .ocrletter

## Property flow

DM reference flow:

DmEditor.vue
→ POST /api/scrape
→ applyScrapedPayload

A real URL was reported working:

https://x.ychouse.tw/8Q4Mdj

V5 reported:

- real extraction PASS
- title PASS
- 9 normalized fields
- 8 images
- extracted preview PASS
- frame selection PASS
- add front PASS
- direct select PASS
- move PASS
- resize PASS
- z-order PASS
- delete control present
- reset PASS

Back insertion was implemented but not fully browser re-tested.

## Persistence

- IndexedDB PASS
- F5 restore PASS
- .ocrletter save wiring
- .ocrletter import wiring
- Letter Strip excluded

## Validation

- console latest reload: no error
- git diff --check PASS
- 7 files / 36 tests PASS
- build PASS

These facts help verify a candidate source is plausibly V5.

They do NOT substitute for source provenance.

---

# ACCEPTED V5 LAYOUT VALIDATION SPEC

Use only AFTER exact V5 source is restored.

---

# 1. PAGE SHELL

Expected:

- no permanent Top Bar
- root height approximately 100dvh
- no outer-page vertical scrollbar
- left Icon Rail
- A4 workspace fills remaining area
- front/back A4 side-by-side where viewport allows
- A4 scales to available viewport rather than forcing page scroll

Concept:

┌──────┬─────────────────────────────────────────────┐
│      │                                             │
│ 60px │             A4 Canvas Workspace            │
│ Icon │                                             │
│ Rail │        [ Front A4 ]   [ Back A4 ]          │
│      │                                             │
└──────┴─────────────────────────────────────────────┘

Do not force dimensions from prose if restored source specifies exact values.

---

# 2. ICON RAIL

Approximate accepted width:

60px

Expected order:

1. 返回 OCR
2. 收件人
3. 新增物件
4. 版面與樣式
5. 文字與內容
6. 圖片素材
7. 信封資訊
8. 開發信條
9. 儲存 / 匯入
10. 列印 / 匯出
11. 全螢幕

Important:

- no separate portrait icon
- no rejected V6 property-grid main icon

---

# 3. TOOLTIPS

Expected:

- real visible tooltip
- not only native title
- around 400ms hover delay
- appears right of rail
- z-index around 1000
- not clipped
- disappears on pointer leave

---

# 4. A4

Physical output:

210mm × 297mm

Screen preview scales responsively.

Do not hard-code screenshot-derived pixel size unless restored source already does so.

---

# 5. FRONT BODY TEXT BASELINE

Observed V5 frontBelow style:

font-family:
"Noto Sans TC", sans-serif

font-size:
20px

font-weight:
400

font-style:
normal

text-decoration:
none

color:
#172033

text-align:
left

line-height:
1.5

letter-spacing:
0px

Do NOT treat previously observed height:404px as a permanent token.

Do NOT invent a fixed width.

The later frontBelow duplicate/data bug was known after V5.

Recovery target is source fidelity, not speculative V6 fixes.

---

# 6. TEXT EDIT MODAL

Expected:

- centered
- approximately 50vw wide
- approximately 90–92vh high
- large text editing surface
- formatting toolbar below
- toolbar one row on normal desktop

Unnecessary labels absent:

- 直接編輯目前選取的 A4 文字物件
- 文字內容

Close behavior:

- 套用 closes
- 取消 closes
- backdrop does NOT close
- A4 blank click does NOT close
- ESC does NOT close text modal

---

# 7. RICH TEXT TOOLBAR

Expected controls:

- 字型
- 字級
- B
- I
- U
- 文字顏色
- 左
- 中
- 右
- 行距
- 字距

Behavior:

selected range
→ format selected range

no selection
→ format whole text object

Handwriting fonts remain available.

Do not invent button pixel sizes if exact V5 source is available.

---

# 8. RECIPIENTS

Independent icon:

收件人

Large centered modal.

Expected functions:

- search
- OCR history add
- Excel import
- manual add
- delete
- multi-select
- select all
- clear
- apply/close
- recipient count

---

# 9. ADD OBJECT

Independent icon:

新增物件

Preserve exact V5 implementation.

Do not import rejected V6 grid architecture during recovery.

---

# 10. IMAGE MATERIALS

Single icon:

圖片素材

Contains:

- LOGO
- LINE QR
- 其他圖片
- 大頭照

No separate portrait icon.

---

# 11. LOGO

Previously verified:

- direct select
- resize smaller
- resize larger
- move
- z-order
- reset
- F5 restore

---

# 12. LINE QR

Previously verified:

- direct select
- resize smaller
- resize larger
- move
- z-order
- reset
- F5 restore

---

# 13. STICKER

Expected explicit state-driven transform behavior:

- move
- resize
- rotate
- flip
- z-order
- delete
- reset

Previously verified:

- repeated rotate PASS
- resize PASS
- drag PASS
- flip PASS
- z-order PASS
- double click PASS
- reset PASS
- F5 restore PASS

---

# 14. PORTRAIT

Inside:

圖片素材 → 大頭照

V5 operations:

- 更換
- AI 去背 entry
- resize
- rotate
- horizontal flip
- bring forward
- send backward
- delete
- reset

Do NOT restore rejected V6 portraitSlot/bodyTextSlot architecture as part of V5 recovery.

---

# 15. ENVELOPE INFO

Independent icon:

信封資訊

Expected:

- sender name
- sender address
- postal code
- 印刷品 ON/OFF
- recipient display:
  - 姓氏
  - 貴住戶

No duplicated 開發信條 controls.

---

# 16. 開發信條

Independent main icon.

Modal:

approximately 90–94vw × 90–94vh

Expected structure:

- header with current count and output actions
- A4 preview area scrolls
- bottom formatting toolbar fixed

Rule:

valid source count N
=
generated N

No fixed 200 assumption.

---

# 17. 開發信條 TEMP DATA

Memory-only.

Must not persist into:

- localStorage
- sessionStorage
- IndexedDB
- Supabase
- backend/VPS
- .ocrletter

F5 clears Letter Strip temp state.

Normal Letter state persists separately.

---

# 18. SAVE / IMPORT

Independent icon:

儲存 / 匯入

Centered modal:

- 儲存設計
- 匯入設計

File extension:

.ocrletter

Normal working draft persists via IndexedDB.

---

# 19. PRINT / EXPORT

Normal Letter user-facing outputs:

- PDF
- print

JPG absent.

---

# 20. FULLSCREEN

Independent icon:

全螢幕

Expected:

- hides rail/panels as implemented
- maximizes A4
- ESC exits fullscreen
- active text modal is not closed by ESC

---

# PHASE 6 — RESTORE TRUE V5 SOURCE

Only after V5 provenance is proven.

Preferred recovery methods in order:

1. verified exact V5 checkpoint
2. exact pre-V6 full snapshot
3. authoritative local-history restore
4. exact V5 patch
5. hunk-level reconstruction from multiple authoritative sources

If using a verified exact V5 checkpoint:

git reset --hard <VERIFIED_V5_SHA>

is allowed ONLY at this stage.

Do NOT run git clean.

If using local history / hunk restore:

restore only proven V5 content.

Do not preserve rejected V6 changes.

---

# PHASE 7 — FIRST BROWSER CHECK BEFORE ANY FIX

Start local dev using the normal project flow.

Use canonical:

http://localhost:3013/letter

Do NOT modify layout immediately.

First inspect exact restored V5 source in browser.

Check:

- page shell
- rail
- tooltip
- A4 placement
- text modal
- recipients
- image materials
- envelope info
- Letter Strip
- save/import
- print/export
- fullscreen

If exact restored V5 still contains a known V5 bug:

REPORT IT.

Do not silently "improve" it during recovery.

The goal is to establish a trustworthy baseline.

---

# PHASE 8 — SOURCE-TO-UI LAYOUT INVENTORY

Once true V5 is restored, generate an ACTUAL layout inventory from source.

For every major UI component record:

- source file
- CSS selector
- width
- height
- min/max width/height
- padding
- margin
- gap
- font-family
- font-size
- font-weight
- line-height
- border-radius
- icon size
- button size
- modal size
- z-index
- responsive/clamp/calc rules

At minimum inventory:

- page shell
- Icon Rail
- icon buttons
- tooltips
- A4 workspace
- A4 page
- text modal
- text toolbar
- recipient modal
- add-object modal
- image-material panel/modal
- envelope-info panel/modal
- Letter Strip modal
- Letter Strip toolbar
- save/import modal
- print/export UI
- fullscreen behavior

If a value comes from CSS variables/calc/clamp:

record the real expression.

Do not invent missing values.

This inventory becomes the new authoritative V5 design spec.

---

# PHASE 9 — VALIDATION

Run:

git diff --check

Frontend tests:

expected baseline 36/36 PASS or better

Production build:

PASS

Browser smoke:

- /
- /letter
- /ocr absent
- tooltips
- LOGO resize
- QR resize
- sticker transform
- text modal
- F5 restore
- Envelope Info
- Letter Strip
- save/import
- fullscreen

---

# PHASE 10 — CREATE A NEW TRUSTED V5 CHECKPOINT

Only if:

TRUE_V5_RECOVERED = YES
and
browser validation is acceptable
and
tests/build pass

Create a NEW local checkpoint commit.

Recommended message:

checkpoint: verified OCR Letter V5 baseline

Before commit:

- inspect status
- inspect diff stat
- ensure rejected V6 code is absent
- ensure no unrelated files are staged
- do NOT use git add -A blindly

Stage only verified V5 source files.

Commit locally.

DO NOT PUSH.

Capture checkpoint SHA.

This new checkpoint supersedes prior untrusted recovery checkpoints.

---

# FUTURE GIT POLICY

From now on:

## Before major task

Accepted current baseline must have a local checkpoint.

## After task PASS

Automatically:

- tests
- build
- local commit
- report SHA

Do not ask the user again for permission to make a LOCAL checkpoint after PASS.

## After task FAIL/BLOCKED

Do NOT commit over accepted baseline.

Preserve task diff separately or revert it.

## Push

Still requires explicit approval unless the task explicitly authorizes push.

---

# HARD STOP CONDITIONS

Stop and report FORENSIC_RECOVERY_BLOCKED if:

1. no authoritative V5 source can be proven
2. all checkpoint candidates are contaminated/ambiguous
3. pre-V6 source is incomplete for high-risk files
4. recovery would require guessing entire layout files
5. user/untracked files risk deletion
6. only old 4aa0fa9 is available
7. exact V5 source cannot be distinguished from V6/recovery edits

Do not create another guessed checkpoint.

---

# FINAL REPORT

Return exactly:

OCR LETTER V5 FORENSIC RECOVERY FINAL

1. CURRENT SOURCE
Repo:
Branch:
HEAD at start:
Dirty tracked:
Untracked:

2. SAFETY BACKUP
Recovery folder:
Working-tree patch:
Staged patch:
Untracked backup:
Hashes:
Verified:

3. FORENSIC SOURCES
Git checkpoint candidates:
Reflog evidence:
IDE/local-history evidence:
Agent/session history:
Recovery folders:
Saved patches:

4. CHECKPOINT AUDIT
Existing V5 checkpoint SHA(s):
Trusted:
Untrusted:
Reason:

5. TRUE V5 SOURCE
Authoritative source used:
Timestamp:
Why it is pre-V6:
High-risk files proven:
TRUE_V5_PROVEN:

6. RESTORE
Method:
Old 4aa0fa9 reset used = NO
git clean used = NO
Untracked files lost = NO
Rejected V6 code remaining:

7. V5 LAYOUT
No Top Bar:
Icon Rail:
Icon order:
Tooltip:
A4 front/back:
Text modal:
Text toolbar:
Recipients:
Add object:
Image materials:
Envelope info:
Letter Strip:
Save/import:
Print/PDF:
JPG absent:
Fullscreen:

8. V5 MECHANICS
LOGO:
QR:
Sticker:
Rich text:
Normal Letter F5:

9. ACTUAL LAYOUT INVENTORY
Inventory generated:
Files/selectors covered:
Unknown/unproven values:

10. VALIDATION
git diff --check:
Tests:
Build:
Browser smoke:

11. NEW TRUSTED CHECKPOINT
Created:
Commit SHA:
Commit message:
Pushed = NO

12. FINAL STATUS
PASS / FORENSIC_RECOVERY_BLOCKED / FAIL
