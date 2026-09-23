# ORC Letter UI Redesign — Luna High Constrained Executor

## OWNER

ORC LETTER UI OWNER

## MODEL

GPT-5.6 Luna — High

## EXECUTION MODE

You are a **constrained frontend executor**, not an architecture planner.

The UI direction and scope are already decided.

Your job is to:
1. inspect the existing implementation,
2. make the smallest safe frontend changes,
3. preserve all existing product behavior,
4. validate locally,
5. stop if the task would require broader architectural work.

Do **not** redesign unrelated areas.
Do **not** opportunistically refactor.
Do **not** "clean up" files outside the required scope.
Do **not** change behavior just because you think another design is better.

---

# CANONICAL SOURCE

F:\00-Ticenpi-SaaS\TicenpiLetter

Local DEV frontend:

http://127.0.0.1:3013

Primary OCR page:

http://127.0.0.1:3013/ocr

Letter page:

http://127.0.0.1:3013/letter

This task is **LOCAL FRONTEND UI/UX ONLY**.

---

# FROZEN USER REQUIREMENTS

These decisions are final. Do not reinterpret them.

## 1. Default route

Opening:

http://127.0.0.1:3013

must land on:

http://127.0.0.1:3013/ocr

Use the existing router correctly.
Do not duplicate the OCR page.

## 2. /ocr is the main product screen

Do not redesign /ocr.

Only verify it still works after the routing change.

## 3. /letter is a secondary editor workspace

The target structure is:

TOP HEADER
+
LEFT EDITOR RAIL
+
LARGE A4 PREVIEW WORKSPACE

No old application sidebar on /letter.

## 4. Remove old /letter shell

On /letter remove the visual structure represented by:

- .sidebar
- .sidebar-brand
- .sidebar-app-bar
- launcher sidebar button
- offline trial footer
- API status footer
- version footer
- old app sidebar shell

Do not keep both:
- old app sidebar
and
- new letter editor rail

There must be only one left-side editor area on /letter.

## 5. Replace old letter toolbar

The old bulky toolbar represented by:

- .letter-toolbar
- old "📮 開發信" title block
- old "回 AI辨識" toolbar button

should be replaced by a slim top header.

Required header content:

- 返回 OCR
- ORC mark/text if already available without new asset work
- 開發信
- small subtitle indicating this page is generated from OCR results

"返回 OCR" must navigate to:

/ocr

Do not add a second redundant "前往 AI OCR" button.

## 6. Left editor rail

Move the existing letter controls to the **far left**.

Preserve these functions:

### 收件人
- 從 AI辨識歷史勾選
- 匯入 Excel
- 手動新增
- recipient list
- delete recipient

### 版型與主題

### 開發信範本

### 正面內文

### 信封樣式

### 業務資訊

### 圖片
- LOGO
- 大頭照
- LINE QR

### 字體

### 匯出
- PDF
- JPG

You may improve only:
- grouping
- spacing
- section presentation
- accordion/collapse presentation
- width
- scrolling

Do not change:
- data model
- event behavior
- form bindings
- business logic
- export behavior
- recipient logic

## 7. A4 preview is highest priority

The user explicitly wants:

**A4 預覽放到最大**

Therefore:

- preview must take most of the remaining viewport
- reduce dead space
- reduce unnecessary chrome
- keep A4 centered and readable
- use viewport height efficiently
- left rail must not become unnecessarily wide
- envelope preview is secondary
- envelope preview may remain if it fits without shrinking the A4 too much

Do not shrink controls to unusable sizes merely to enlarge A4.

## 8. Remove "適應視窗"

Remove the user-facing:

"適應視窗"

control.

Simple zoom controls may remain:

- zoom out
- zoom in
- page indicator

If an internal fit calculation is required for initial sizing, it may remain internally.
Only the visible control must be removed.

---

# VISUAL TARGET

Desktop-first, modern B2B SaaS editor.

Use:

- slim top header
- clean far-left configuration rail
- large neutral preview workspace
- dominant A4 letter preview
- restrained blue accents
- clear typography hierarchy
- compact cards/sections
- subtle borders/shadows
- clean spacing

Avoid:

- dashboard clutter
- duplicate navigation
- oversized headers
- floating panels
- transform-based positioning hacks
- giant empty areas
- decorative redesign unrelated to the existing product
- introducing new design systems or dependencies

---

# STRICT CHANGE BOUNDARY

Before editing, inspect the actual implementation and identify the minimum files.

Preferred change surface:

1. router file controlling /
2. /letter view/component
3. letter-specific CSS/style block
4. at most small letter-only child component(s) if absolutely necessary

### File budget

Target: **2–4 modified frontend files**.

If more than **5 tracked files** appear necessary:

HARD STOP before editing the sixth file.

Report why broader change is required.

### Shared files

If a shared layout/router file must be changed:

- make the smallest route-specific change
- preserve /ocr behavior
- do not refactor shared architecture

### Never modify

- backend/**
- db/**
- deploy/**
- docker-compose*
- Dockerfile*
- Supabase config/schema/migrations
- auth contract
- entitlement contract
- OCR API logic
- Cloudflare/DNS
- Staging
- Production
- CI workflow
- package dependencies unless absolutely required

No new npm package is expected.
If you believe a package is required: HARD STOP.

---

# GIT SAFETY

At start record:

- branch
- HEAD
- git status
- tracked dirty files
- untracked files

Rules:

- preserve all pre-existing unrelated changes
- never reset
- never restore unrelated files
- never clean
- never delete unrelated untracked files
- never use git add -A
- do not commit
- do not push

This task ends with local verified changes only.

---

# BEHAVIOR PRESERVATION RULE

This is critical.

For /letter existing functionality:

**preserve methods, events, state, API calls, watchers, computed values, form bindings, export handlers and data flow unless a UI requirement absolutely requires a tiny change.**

Prefer:

- DOM structure adjustment
- container/layout adjustment
- scoped CSS adjustment
- route redirect

over:

- business logic rewrite
- state refactor
- component architecture rewrite

If an existing function looks ugly but works, leave it alone.

---

# PHASE 0 — PREFLIGHT / BASELINE

Before editing:

1. inspect actual route definitions
2. locate /ocr view
3. locate /letter view
4. locate shared app shell/sidebar logic
5. locate letter panel
6. locate A4 preview
7. locate envelope preview
8. locate zoom / fit-to-window controls

Then establish baseline:

- open /ocr
- open /letter
- note existing console errors
- run the smallest relevant existing frontend tests/build if practical

Do not fix unrelated baseline errors.

Record them separately.

---

# PHASE 1 — ROUTE ONLY

Implement:

/ -> /ocr

Use the existing router mechanism.

Validate:

http://127.0.0.1:3013
→ /ocr

Check for redirect loops.

Do not change /ocr implementation.

---

# PHASE 2 — /letter SHELL ISOLATION

Make /letter visually independent from the old app sidebar shell.

Important:

- if /ocr needs the old shell, leave it unchanged
- isolate the change to /letter
- do not remove shared shell functionality globally

If safe route-specific isolation is not possible without broad shared-layout redesign:

HARD STOP.

Do not improvise a global rewrite.

---

# PHASE 3 — TOP HEADER

Create a slim top header for /letter.

Required:

- 返回 OCR
- 開發信
- concise OCR-related subtitle

Behavior:

返回 OCR → /ocr

Keep the header compact.
Do not use unnecessary hero-sized spacing.

---

# PHASE 4 — LEFT EDITOR RAIL

Move the existing letter settings into a proper far-left rail.

Required layout characteristics:

- stable desktop width
- top-aligned under header
- vertically scrollable
- no floating transforms
- no overlap
- compact section spacing
- all existing controls remain usable

Do not rewrite the underlying controls unless necessary for layout.

---

# PHASE 5 — PREVIEW WORKSPACE

Use remaining width/height for preview.

Priority order:

1. A4 letter preview
2. editing rail
3. envelope preview
4. minor preview controls

A4 must be materially larger than baseline.

Prefer CSS/layout changes over changing preview rendering logic.

Do not change document content generation.

---

# PHASE 6 — REMOVE FIT CONTROL

Remove visible:

適應視窗

Keep only useful minimal preview controls.

Do not break automatic initial preview sizing.

---

# PHASE 7 — SELF-REVIEW BEFORE TEST

Before running final tests, inspect your own diff.

Reject your own changes and correct them if any of these are true:

- unrelated files changed
- /ocr layout changed
- backend/db/deploy files changed
- business logic was rewritten without necessity
- existing letter feature disappeared
- left rail duplicates old sidebar
- A4 did not materially grow
- large hard-coded transforms were introduced
- unnecessary dependency added
- console errors were introduced by your change

Run:

git diff --check

It must pass.

---

# PHASE 8 — LOCAL BROWSER VALIDATION

Use the existing local DEV environment.

Validate all:

## Route

http://127.0.0.1:3013
→ /ocr

PASS required.

## /ocr

- loads normally
- main OCR UI remains unchanged
- no new relevant console error

PASS required.

## /letter

- no old app sidebar shell
- slim header present
- 返回 OCR works
- settings rail is far left
- all required sections still exist
- controls are usable
- A4 preview is clearly larger than baseline
- envelope preview behaves normally if retained
- no 適應視窗 button
- no overlap
- no broken scroll
- no severe clipping
- no new relevant console errors

PASS required.

## Existing behavior smoke

Without changing data/business state unnecessarily, verify representative existing interactions still respond:

- recipient selection UI opens/works
- theme/template interaction responds
- text area remains editable
- image upload controls still exist
- export buttons still exist

Do not perform destructive actions.

---

# PHASE 9 — TEST / BUILD

Run only existing relevant frontend tests/build.

Do not repair unrelated pre-existing failures.

Report separately:

- baseline/pre-existing failure
- failure caused by this change

PASS requires no new failure caused by this task.

---

# HARD STOP CONDITIONS

Immediately stop and do not broaden scope if any occurs:

1. more than 5 tracked files appear necessary
2. requires backend change
3. requires DB/Supabase change
4. requires Docker/deploy change
5. requires new dependency
6. requires global shared-shell rewrite that risks /ocr
7. existing letter business logic must be redesigned
8. unrelated dirty work blocks safe editing
9. route structure differs so much that / -> /ocr cannot be safely done surgically

Output only:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS

---

# PASS CRITERIA

PASS only if ALL are true:

- / -> /ocr works
- /ocr remains functionally and visually intact
- /letter old sidebar shell is gone
- /letter has one clear top header
- 返回 OCR works
- one far-left letter editor rail exists
- required letter controls are preserved
- A4 preview is materially larger and dominant
- 適應視窗 is removed
- no new dependency
- no backend/DB/Docker/deploy changes
- git diff --check passes
- relevant frontend tests/build have no new failures
- actual browser visual validation passes

---

# FINAL REPORT — KEEP SHORT

Return exactly:

ORC LETTER UI REDESIGN FINAL

1. SOURCE
Branch:
HEAD:
Tracked dirty before:
Tracked dirty after:

2. FILES CHANGED
- file
- file

3. ROUTING
/ -> /ocr:
返回 OCR -> /ocr:

4. UI
Old sidebar removed:
Left rail:
A4 preview enlarged:
Fit-to-window removed:
Existing controls preserved:

5. VALIDATION
/ocr regression:
 /letter visual:
Console:
git diff --check:
Tests/build:

6. OUT OF SCOPE
Backend changed = NO
DB changed = NO
Docker changed = NO
Staging changed = NO
Production changed = NO
Commit/push performed = NO

7. FINAL STATUS
PASS / BLOCKED / FAIL
