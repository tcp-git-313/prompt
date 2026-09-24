# OCR Letter Editor — Restore Accepted V5 Layout Safely

## OWNER

OCR LETTER EDITOR V5 LAYOUT RECOVERY OWNER

## MODEL

GPT-5.6 Luna — High

## OBJECTIVE

Restore the OCR Letter Editor to the accepted V5 visual/layout baseline without guessing CSS values and without destroying valid V2–V5 work.

The current layout is reported as broken/misaligned.

This is a RECOVERY task, not a redesign task.

Your priority is:

1. find the real V5 checkpoint/snapshot if it exists,
2. restore from that exact source,
3. only use the documented V5 layout spec as a browser acceptance checklist,
4. never rebuild V5 merely from prose if exact source history exists.

---

# CANONICAL REPO

F:\00-Ticenpi-SaaS\TicenpiLetter

Expected branch:

master

Historical pre-checkpoint HEAD may have been:

4aa0fa90693347065411acf75b9f26524200d65a

IMPORTANT:

That old HEAD is NOT the V5 baseline.

V2/V3/V4/V5 had been developed as uncommitted changes on top of that HEAD.

Therefore:

DO NOT reset to 4aa0fa9.

That would erase accepted V2–V5 work.

---

# FIRST PRINCIPLE

The correct recovery source is:

an exact V5 checkpoint / pre-V6 snapshot / authoritative local history

NOT:

the prose layout summary alone.

The layout summary below is only for acceptance validation after source recovery.

---

# ABSOLUTE SAFETY RULES

Before proving an exact V5 checkpoint/snapshot exists, DO NOT use:

- git reset --hard
- git reset --merge
- git reset --keep
- git restore .
- git checkout -- .
- git checkout HEAD -- .
- git clean
- git clean -f
- git clean -fd
- deleting untracked files
- restoring whole mixed-history files from old HEAD
- manually reconstructing all CSS from memory/spec
- mass search/replace
- commit
- push

Do not guess.

Do not silently drop current files.

---

# PHASE 0 — FREEZE AND BACK UP CURRENT BROKEN STATE

Before any recovery mutation:

cd F:\00-Ticenpi-SaaS\TicenpiLetter

Capture:

- git branch --show-current
- git rev-parse HEAD
- git status --short
- git diff --stat HEAD
- git diff --name-status HEAD
- git diff --binary HEAD

Create a recovery folder OUTSIDE the repo, for example:

F:\00-Ticenpi-SaaS\_recovery\TicenpiLetter-before-v5-layout-restore-<timestamp>\

Save:

- current-working-tree.patch
- staged-working-tree.patch if staged changes exist
- git-status.txt
- changed-files.txt
- untracked-files.txt
- copies of all current untracked files, preserving relative paths

Verify backup artifacts exist.

Do not continue until the backup is complete.

---

# PHASE 1 — FIND THE REAL V5 CHECKPOINT

Search local Git history first.

Run read-only checks such as:

git log --oneline --decorate -n 30
git log --all --oneline --decorate --grep="V5"
git log --all --oneline --decorate --grep="checkpoint"
git reflog --date=iso -n 50

Look specifically for a local checkpoint created after the safe V6→V5 recovery task.

Expected commit message may resemble:

checkpoint: restore OCR Letter V5 baseline before V6

Do not assume the exact SHA.

If such a checkpoint exists, inspect it BEFORE using it:

git show --stat <candidate>
git show --name-status <candidate>

Confirm it contains the complete accepted V5 Letter Editor baseline rather than only a prompt/document change.

---

# PHASE 2A — IF A VERIFIED V5 CHECKPOINT EXISTS

Only after proving the commit is the accepted V5 baseline:

1. preserve the current broken-state backup from Phase 0
2. confirm no user-created file will be lost
3. restore the worktree to that exact V5 checkpoint

A hard reset is allowed ONLY in this branch of the procedure, because the target has now been positively identified as the full V5 checkpoint.

Use:

git reset --hard <VERIFIED_V5_CHECKPOINT_SHA>

DO NOT run git clean.

Preserve untracked files unless there is separate proof they belong only to rejected later work.

After reset:

- HEAD must equal the verified V5 checkpoint
- inspect git status
- do not push

Then proceed to browser acceptance.

---

# PHASE 2B — IF NO VERIFIED V5 CHECKPOINT EXISTS

DO NOT reset to 4aa0fa9.

Instead search authoritative recovery sources:

1. IDE local history
2. prior same-agent edit history
3. pre-V6 snapshots
4. exact saved V6 patch
5. exact V5 working-tree patch if one exists
6. recovery backup created during earlier V6→V5 safe revert

The goal is to recover the exact V5 source tree.

If exact V5 provenance cannot be established:

STOP.

Return:

V5_LAYOUT_RECOVERY_BLOCKED

with:

- Git history checked
- reflog checked
- local-history source checked
- recovery folders checked
- files whose V5 content cannot be proven
- safest next recovery option

DO NOT rebuild from prose.

---

# ACCEPTED V5 LAYOUT — VALIDATION SPEC ONLY

Use the following AFTER restoring exact V5 source.

Do not use this section as a substitute for source recovery.

---

# 1. PAGE SHELL

Letter Editor desktop layout:

- no permanent Top Bar
- root height: 100dvh
- outer page/body does not show a vertical scrollbar
- Icon Rail on the left
- A4 canvas workspace fills remaining width
- front/back A4 previews displayed side-by-side when viewport allows
- A4 previews scale to available viewport instead of forcing outer-page scroll

Concept:

┌──────┬─────────────────────────────────────────────┐
│      │                                             │
│ 60px │             A4 Canvas Workspace            │
│ Icon │                                             │
│ Rail │        [ Front A4 ]   [ Back A4 ]          │
│      │                                             │
└──────┴─────────────────────────────────────────────┘

---

# 2. ICON RAIL

Accepted V5 baseline:

approximately 60px wide
height: 100dvh
overflow must allow real tooltips to render

Main order:

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

V5 does NOT include a separate portrait icon.

V5 does NOT include the later rejected V6 independent property-grid icon.

---

# 3. ICON TOOLTIPS

Real tooltip, not only native HTML title.

Accepted V5 behavior:

- hover delay around 400ms
- tooltip appears to the right of the Icon Rail
- z-index around 1000
- rail overflow allows tooltip to escape
- tooltip disappears when pointer leaves
- no permanent text label inside rail

Browser-verify every main icon.

---

# 4. A4

Physical output target:

210mm × 297mm

Screen display scales responsively.

Do not force a fixed browser-pixel A4 dimension if current V5 source already implements responsive scaling.

Front/back are the visual source used for later print/PDF flow.

---

# 5. FRONT BODY TEXT BASELINE

Observed accepted V5 text style for frontBelow body text:

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

IMPORTANT:

Do NOT invent a permanent fixed width from screenshots.

Do NOT treat previously observed height:404px as a design token.

V5 had later-known frontBelow issues; recovery goal is V5 source fidelity, not speculative V6 layout fixes.

---

# 6. TEXT EDIT MODAL

Accepted V5 baseline:

- centered modal
- approximately 50vw width
- approximately 90–92vh height
- main editing area occupies most of modal
- formatting toolbar at bottom
- toolbar stays one row on normal desktop width

Unnecessary labels should remain absent:

- 直接編輯目前選取的 A4 文字物件
- 文字內容

Close rules:

- 套用 closes
- 取消 closes
- backdrop click does NOT close
- clicking blank A4 does NOT close
- ESC does NOT close text-edit modal

Apply:
commit current edit

Cancel:
rollback modal-open snapshot

---

# 7. TEXT TOOLBAR

Accepted controls:

- 字型
- 字級
- B
- I
- U
- 文字顏色
- 左對齊
- 置中
- 右對齊
- 行距
- 字距

Behavior:

selected text range
→ formatting applies to selected range

no text selection
→ formatting applies to whole text object

Preserve handwriting-font options already integrated from DM.

Do not invent exact button pixel sizes unless current restored V5 CSS specifies them.

---

# 8. RECIPIENTS

Independent main icon:

收件人

Large centered modal intended for 200+ records.

Features:

- search
- OCR history add
- Excel import
- manual add
- single delete
- multi-select
- select all
- clear
- apply/close
- recipient count badge

Do not redesign this during recovery.

---

# 9. GENERIC ADD OBJECT

Independent main icon:

新增物件

V5 supported adding existing object types through a centered modal.

Preserve V5 exact implementation.

Do NOT import rejected V6 property-grid behavior.

---

# 10. IMAGE MATERIALS

Single main icon:

圖片素材

Contains:

- LOGO
- LINE QR
- 其他圖片
- 大頭照

No separate portrait icon.

---

# 11. LOGO

Accepted mechanics already browser-verified before later layout breakage:

- direct select
- resize smaller
- resize larger
- move
- z-order
- reset
- F5 restore

Do not rebuild this logic.

---

# 12. LINE QR

Accepted mechanics:

- direct select
- resize smaller
- resize larger
- move
- z-order
- reset
- F5 restore

Do not rebuild this logic.

---

# 13. OTHER IMAGE / STICKER

Other images can become stickers.

Sticker state uses explicit transform data such as:

- x
- y
- width/height or scale
- rotation
- flipX
- zIndex

Accepted interactions:

- drag
- resize
- rotate
- horizontal flip
- bring forward
- send backward
- delete
- reset position/transform

Previously verified:

- repeated rotate PASS
- resize PASS
- drag PASS
- flip PASS
- z-order PASS
- double click PASS
- reset PASS
- F5 restore PASS

Do not regress.

---

# 14. PORTRAIT

Portrait is inside:

圖片素材 → 大頭照

Accepted V5 operations include:

- 更換
- AI 去背 entry/action
- resize
- rotate
- horizontal flip
- bring forward
- send backward
- delete
- reset transform

IMPORTANT:

The later rejected V6 portraitSlot/bodyTextSlot architecture is NOT part of V5 baseline.

Do not recreate V6 slot layout during V5 recovery.

---

# 15. LAYOUT / STYLE

Main icon:

版面與樣式

Preserve current V5 theme/template implementation.

Do not reconstruct cards or dimensions manually.

Use restored source.

---

# 16. ENVELOPE INFO

Independent main icon:

信封資訊

Contains true envelope settings such as:

- 寄件人姓名
- 寄件地址
- 郵遞區號
- 印刷品 ON/OFF
- 收件人顯示:
  - 顯示姓氏
  - 顯示「貴住戶」

The two recipient-display options are mutually exclusive.

V5 had already removed duplicate 開發信條 controls from this panel.

Verify they remain absent.

---

# 17. 開發信條

Independent main icon:

開發信條

Not nested under Envelope Info.

Accepted V5 modal concept:

- centered
- approximately 90–94vw wide
- approximately 90–94vh high
- header contains current record count and output actions
- middle preview area scrolls
- bottom formatting toolbar stays fixed

Data count rule:

current valid source count N
=
generated Letter Strip count N

No fixed 200 assumption.

V5 example validation had:
source count 2
generated count 2

Do not turn that example into a fixed value.

---

# 18. 開發信條 LIFECYCLE

Memory-only temporary state.

Must NOT persist to:

- localStorage
- sessionStorage
- IndexedDB
- Supabase
- DB
- VPS
- .ocrletter

Behavior:

close modal
→ remains in current page session

switch tools
→ remains

F5
→ clears temporary Letter Strip session

close tab/browser
→ clears

Normal Letter autosave remains persistent separately.

---

# 19. SAVE / IMPORT

Independent main icon:

儲存 / 匯入

Centered modal with:

- 儲存設計
- 匯入設計

Project file extension:

.ocrletter

Normal working draft persists separately through IndexedDB.

---

# 20. NORMAL LETTER AUTOSAVE

IndexedDB.

F5 must restore normal Letter design.

Includes normal editor state such as:

- text
- rich text formatting
- images
- transforms
- recipients/settings as implemented
- envelope settings
- property object state that belonged to accepted V5

Do not include temporary 開發信條 session state.

---

# 21. PRINT / EXPORT

Accepted user-facing normal Letter outputs:

- PDF
- direct print

JPG is removed.

Do not restore JPG.

---

# 22. FULLSCREEN

Independent main icon:

全螢幕

Accepted behavior:

- hides Icon Rail / temporary panels as implemented
- maximizes A4 preview
- ESC exits fullscreen

Exception:

ESC must not close an active text-edit modal.

---

# PHASE 3 — BROWSER ACCEPTANCE AFTER SOURCE RESTORE

Use:

http://localhost:3013/letter

Do not modify source yet.

Validate exact restored source first.

Check:

1. no Top Bar
2. no outer-page scrollbar
3. ~60px Icon Rail
4. correct V5 icon order
5. every tooltip visibly works
6. A4 front/back layout looks like accepted V5
7. text modal structure is correct
8. text toolbar single row
9. LOGO resize works
10. QR resize works
11. sticker transforms work
12. image materials contains LOGO/QR/other/portrait
13. envelope info contains no duplicate Letter Strip settings
14. independent 開發信條 icon exists
15. current N records generate N strips
16. 儲存 / 匯入 exists
17. JPG absent
18. fullscreen works
19. normal Letter F5 restore works

If source is exact V5 but one item still looks wrong:

REPORT IT.

Do NOT immediately redesign.

Recovery fidelity comes before new fixes.

---

# PHASE 4 — VALIDATION

Run:

git diff --check

Run existing frontend tests.

Expected prior baseline:

36/36 PASS or better

Run frontend production build.

PASS required.

Do not fix unrelated warnings during recovery.

---

# PHASE 5 — CREATE/CONFIRM LOCAL CHECKPOINT

If recovery used an existing V5 checkpoint:

do NOT create another duplicate commit unless source changed during safe restoration.

If recovery came from local history/patch and no checkpoint existed:

after V5 restoration is browser-verified and tests/build pass, create a LOCAL checkpoint commit:

checkpoint: restore OCR Letter V5 layout baseline

Do NOT push.

Do not use git add -A blindly.

Stage only verified V5 baseline files.

---

# FUTURE GIT RULE

From now on:

- accepted PASS milestone → local checkpoint commit automatically
- next risky task starts from that checkpoint
- push still requires explicit approval

Never stack another major redesign on top of an uncommitted accepted baseline.

---

# HARD STOP CONDITIONS

Stop immediately if:

1. no exact V5 checkpoint/snapshot/history can be found
2. candidate checkpoint provenance is uncertain
3. recovery would require guessing entire mixed-history files
4. untracked user files risk deletion
5. restoring V5 would require resetting to old 4aa0fa9
6. more than a minimal recovery operation is needed before source provenance is established

Return:

V5_LAYOUT_RECOVERY_BLOCKED

with exact evidence and safest options.

---

# FINAL REPORT

Return exactly:

OCR LETTER V5 LAYOUT RECOVERY FINAL

1. SOURCE
Repo:
Branch:
HEAD before:
HEAD after:
Recovery source:

2. SAFETY BACKUP
Recovery folder:
Tracked patch:
Untracked backup:
Verified:

3. V5 CHECKPOINT
Candidate SHA:
Commit message:
Proven as V5 baseline:
Reset-to-old-4aa0fa9 used = NO
git clean used = NO

4. LAYOUT RECOVERY
No Top Bar:
Icon Rail:
Icon order:
Tooltips:
A4 front/back:
Outer scrollbar:
Text modal:
Text toolbar:
Recipients:
Image materials:
Envelope info:
Letter Strip:
Save/import:
Print/PDF:
JPG removed:
Fullscreen:

5. EDITOR MECHANICS
LOGO:
QR:
Sticker:
Rich text:
Normal Letter F5 restore:

6. VALIDATION
git diff --check:
Tests:
Build:
Browser smoke:

7. CHECKPOINT
Existing V5 checkpoint reused:
New checkpoint created:
Checkpoint SHA:
Pushed = NO

8. FINAL STATUS
PASS / V5_LAYOUT_RECOVERY_BLOCKED / FAIL
