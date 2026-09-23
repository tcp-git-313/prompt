# ORC Letter Editor V3 — Stabilization + Object Editing

## OWNER

ORC LETTER EDITOR V3 OWNER

## MODEL

GPT-5.6 Luna — High

## EXECUTION MODE

You are a **constrained frontend executor**.

This is NOT a redesign-from-scratch task.

V2 already exists in the current working tree and must be improved in place.

Your job is to:
1. preserve the current V2 direction,
2. fix the reported bugs,
3. add the missing editor interactions,
4. reuse proven functionality from the DM project only where explicitly requested,
5. avoid unrelated refactors,
6. validate the result in the real local browser.

Do not freely reinterpret the UI.
Do not change unrelated OCR behavior.
Do not modify backend/DB/deploy architecture.
Do not reset or discard the current V2 working tree.

---

# CANONICAL SOURCE

ORC:

F:\00-Ticenpi-SaaS\TicenpiLetter

Reference DM project:

F:\00-Ticenpi-SaaS\TicenpiDM

Local canonical host:

http://localhost:3013/

Letter editor:

http://localhost:3013/letter

---

# IMPORTANT CURRENT BASELINE

Expected git HEAD before this task:

4aa0fa90693347065411acf75b9f26524200d65a

V2 changes may currently exist as UNCOMMITTED modifications on top of that HEAD.

Known V2-modified tracked files from the previous task:

- frontend/src/router/index.js
- frontend/src/views/LetterView.vue
- frontend/src/views/PickerView.vue

This task MUST continue from the current working tree.

DO NOT:

- reset
- checkout old versions
- restore these files
- discard V2 changes
- clean untracked files
- use git add -A
- commit
- push

At start, capture:

- branch
- HEAD
- git status
- tracked dirty files
- untracked files

Then work on top of the current state.

---

# FINAL ROUTING CONTRACT — PRESERVE

Keep:

/        = OCR main screen
/letter  = Letter editor

Remove:

/ocr

Requirements:

- http://localhost:3013/ renders OCR directly
- URL remains /
- /ocr is not a product route
- Letter back icon returns to /
- do not reintroduce / -> /ocr redirect
- do not create /ocr alias

Do not change this unless a hard technical conflict is proven.

---

# V3 GOAL

Turn the current V2 UI into a stable usable editor.

The main work areas are:

1. text live preview + working fonts
2. viewport/scrollbar fix
3. direct object selection on A4
4. image/sticker interaction stabilization
5. working AI background removal
6. consolidated image-material workflow
7. new Envelope Info icon
8. reliable persistence after refresh
9. local project-file save
10. better default typography/layout
11. front/back object selection parity

This is a stabilization/editor-interaction task, not a broad visual redesign.

---

# 1. TEXT EDITING — LIVE PREVIEW + WORKING FONTS

## Current bug

Changing font currently appears to have no effect.

Treat this as a BUG.

## DM reference scope — VERY IMPORTANT

Inspect:

F:\00-Ticenpi-SaaS\TicenpiDM

But for typography, use DM ONLY to find and reuse the **handwriting fonts** that already exist there.

Do NOT copy DM's entire:

- font system
- design system
- text editor architecture
- object-selection architecture
- typography scale

For fonts, the only requested reuse from DM is:

**handwriting font assets / font-face declarations / loading method required to make those handwriting fonts actually work in ORC Letter.**

Keep ORC's existing normal fonts.

Add the DM handwriting fonts as additional working choices.

Verify actual browser rendering, not only CSS declarations.

## Live preview behavior

When the text edit modal is open, any change to:

- text content
- font family
- font size
- bold
- italic
- underline
- text color
- alignment
- line height
- letter spacing

must update immediately in BOTH:

1. the text editing area itself
2. the corresponding selected text object on A4

Do not require clicking Apply just to see the result.

## Apply / Cancel semantics

On modal open:

capture a snapshot of the selected text object's content + style.

During editing:

changes are live-previewed.

Cancel:

restore the exact snapshot from modal-open time.

Apply:

keep the current state.

Do not let Cancel leave partial live changes behind.

---

# TEXT MODAL — PRESERVE V2 SIZE / STRUCTURE

Centered modal.

Desktop target:

- width: approximately 50vw
- height: approximately 90–94vh

Order is FROZEN:

1. modal title
2. 文字內容 label
3. large text editing area
4. formatting toolbar
5. cancel/apply actions

The text editing area must consume most of the modal height.

---

# FORMATTING TOOLBAR — HARD REQUIREMENT

The formatting toolbar must remain ONE SINGLE ROW.

Required controls:

字型 | 字級 | B | I | U | 文字顏色 | 左對齊 | 置中 | 右對齊 | 行距 | 字距

Requirements:

- no wrapping to a second row on normal desktop width
- compact dropdowns
- compact icon buttons
- compact color control
- reduce padding before considering horizontal overflow
- normal desktop use should not show a horizontal scrollbar

Do not vertically stack controls.

---

# 2. VIEWPORT HEIGHT / SCROLLBAR BUG

Current screenshot shows unwanted vertical scrollbars / duplicated scroll tracks.

The main Letter editor page must fit the viewport height automatically.

Goal:

maximize A4 height WITHOUT exceeding the viewport.

The page shell should not create body/page scrolling.

Required layout behavior:

- Letter editor root: height = 100dvh
- main page shell: overflow hidden
- icon rail: fit 100dvh and do not create its own visible scrollbar
- canvas workspace: fit 100dvh and do not create page scrollbar
- A4 scale is computed/fitted so front/back preview does not exceed usable viewport height

The A4 should be as tall as possible but never force the whole page to scroll.

Allowed internal scrolling:

- recipient modal contents
- text modal content area where needed
- temporary settings panels

Not allowed:

- body/page vertical scrollbar
- icon rail vertical scrollbar
- duplicate left-side scrollbars
- canvas workspace page scrollbar caused by oversized preview

Validate at the user's current desktop viewport and at one smaller common desktop viewport.

---

# 3. DIRECT OBJECT SELECTION ON A4

The A4 canvas must support direct selection.

The user should NOT need to go to the icon rail first just to edit an object that is already visible.

Required:

## Text

click visible text
→ select text object
→ show selection outline
→ open text edit modal

## Image

click visible image
→ select image object
→ show image selection controls / image settings

## Sticker / portrait

click sticker
→ select
→ show resize/rotate interaction handles

## Empty canvas

click empty area
→ clear selectedObject

Create or preserve ONE shared selection concept:

selectedObject

Front and back pages must use the same object-selection behavior.

Do not implement separate unrelated selection systems for front and back.

---

# 4. IMAGE MATERIALS — CONSOLIDATE

There must NOT be a separate portrait icon anymore.

Keep one:

圖片素材

Inside it:

- LOGO
- LINE QR
- 其他圖片
- 大頭照

## Other images

Other images must support normal image placement.

They must also have an action:

製作貼紙

If converted to a sticker, support:

- move
- resize
- rotate
- horizontal flip
- bring forward
- send backward
- delete
- reset position / transform

## Portrait

Portrait is managed inside 圖片素材.

When portrait is selected, available actions:

- 更換
- AI 去背
- 縮放
- 旋轉
- 水平翻轉
- 置前
- 置後
- 刪除
- 重設位置 / 變形

Do not restore a separate portrait icon.

---

# 5. STICKER TRANSFORM BUG — IMAGE DISAPPEARS AFTER ROTATION

Current user-reported bug:

after clicking/rotating multiple times, the image can disappear and there is no obvious way to recover it.

Treat this as a BUG.

Do not solve it with ad-hoc transform string concatenation.

Each transformable image/sticker should have explicit state such as:

- x
- y
- width/height or scale
- rotation
- flipX
- zIndex

Render transform from that single state.

Required interaction combinations to test:

- rotate repeatedly
- resize repeatedly
- rotate then drag
- drag then rotate
- resize then rotate
- rotate then resize
- double click
- flip then rotate
- z-order then move

The object must not disappear.

## Safety/recovery

Add:

重設位置 / 變形

This must restore the selected sticker/image to a safe visible position and sane transform.

Prevent an object from becoming permanently unrecoverable outside the A4 canvas.

A partially out-of-bounds object may be allowed if intentional, but there must always be a recoverable selected state / reset action.

---

# 6. AI BACKGROUND REMOVAL — MUST ACTUALLY WORK

Current V2 background removal is reported as not working.

Do not mark PASS because an endpoint exists.

## DM reference

Inspect the working DM project:

F:\00-Ticenpi-SaaS\TicenpiDM

For RemoveBG ONLY, compare the working DM implementation with ORC.

Inspect:

- request URL
- HTTP method
- request/form-data shape
- image field name
- MIME handling
- auth/header behavior
- timeout
- response schema
- blob/base64 conversion
- transparent PNG handling
- error handling

ORC may already expose:

/api/letter/removebg

Reuse existing backend capability if it is valid.

Do NOT:

- create a new backend endpoint
- add a new external paid service
- add a new API secret
- add a new large ML dependency
- redesign backend architecture

Fix frontend request/response integration as needed.

## Acceptance test

Use a real local portrait image and verify:

source image
→ AI remove background
→ output visibly has transparent background
→ transparent image appears on A4
→ sticker remains movable/resizable/rotatable

Visual/browser evidence is required.

Endpoint 200 alone is NOT sufficient.

---

# 7. NEW ICON — 信封資訊

Add a new icon to the left icon rail:

信封資訊

Hover tooltip:

信封資訊

Do NOT put the following settings under 版面與樣式.

All envelope / sender / postal presentation settings belong to 信封資訊.

Include existing envelope-related settings where already supported, such as:

- 寄件人姓名
- 寄件地址
- 郵遞區號
- existing envelope/postal fields

And restore these two missing features:

---

## 印刷品 toggle

UI:

印刷品
[ON / OFF]

Behavior:

ON
→ show the red 印刷品 mark/stamp on the envelope side

OFF
→ hide it

This must update preview immediately.

---

## 收件人顯示 mode

This is mutually exclusive.

Use segmented control or radio behavior, not two independent checkboxes.

UI concept:

收件人顯示

● 顯示姓氏
○ 顯示「貴住戶」

Rules:

### 顯示姓氏

preserve the existing recipient name/surname presentation logic.

### 顯示「貴住戶」

all recipients display:

貴住戶

Do not show the actual recipient surname/name in that recipient-label position.

Exactly one option must always be selected.

Do not allow both.
Do not allow neither unless the existing business logic explicitly requires a third state, which is not expected.

---

# 8. ICON RAIL — FINAL CONTENT

Keep a compact left rail.

Expected icons:

1. 返回 OCR
2. 收件人
3. 版面與樣式
4. 文字與內容
5. 圖片素材
6. 信封資訊
7. 儲存設計
8. 列印 / 匯出
9. 全螢幕

No separate portrait icon.

Every icon must have a Traditional Chinese hover tooltip.

---

# 9. PERSISTENCE BUG — REFRESH MUST NOT CLEAR THE EDITOR

Current user-reported behavior:

- refresh clears data
- 儲存設計 does not truly preserve the design

Treat this as a critical BUG.

There are TWO persistence layers.

---

## A. AUTO-SAVE WORKING DRAFT

Use IndexedDB for the working editor draft.

Do NOT rely only on volatile Vue state.

Do NOT clear existing localStorage/session state.

Persist the complete editor state required to restore the document:

- text content
- text styles
- text object settings
- selected template/theme where relevant
- recipient data/selection
- image slot references/data
- sticker/image transforms
- front/back object state
- layout choices
- envelope info
- 印刷品 toggle
- recipient display mode
- other current letter-specific settings required for exact restoration

## User isolation

Draft must be isolated per signed-in user identity where available.

Do not allow tcp.a2026i data to appear under another signed-in account.

If running offline/local mode, use a deterministic local/offline draft namespace.

## Restore behavior

modify editor
→ autosave
→ F5
→ exact state restored

close browser / reopen
→ state restored

Do not let default sample content overwrite a real saved draft.

Priority:

1. restored real draft/user state
2. existing real loaded data
3. defaults only if truly empty

---

# 10. SAVE DESIGN = REAL LOCAL FILE

The 儲存設計 icon must save a real local project file.

This is distinct from IndexedDB autosave.

Recommended extension:

.orcletter

Example:

我的開發信.orcletter

The file should contain enough structured data to reconstruct the editor state.

Prefer:

File System Access API

e.g. showSaveFilePicker()

when supported.

Fallback:

download the project file.

Do NOT silently treat localStorage/IndexedDB as the user's explicit local-file save.

The explicit Save Design action must produce/save a file.

If practical within existing frontend scope, also provide:

開啟設計

to load a .orcletter file back into the editor.

If adding Open Design safely would broaden scope too much, HARD STOP only on that sub-feature and report it separately; do not block the required Save Design file behavior.

---

# 11. DEFAULT DATA / DEFAULT IMAGES SAFETY

Defaults should make the editor look complete only when there is no real state.

Never overwrite restored user state.

Never reset editor state on mount when data exists.

Default/sample data should include sensible values for:

- name
- title
- phone
- company
- address
- LINE
- email
- body text

Default images can exist for:

- logo
- LINE QR
- portrait

When user uploads a replacement, the user image replaces the default.

Do not duplicate the placeholder beside the replacement.

---

# 12. DEFAULT TYPOGRAPHY / LAYOUT CLEANUP

The current default preview has inconsistent sizing and weak hierarchy.

This task should improve the built-in default layout without redesigning the whole product.

Do NOT copy DM visual design.

DM is only a technical reference for:

1. handwriting fonts
2. RemoveBG implementation

For ORC Letter itself, establish a small consistent typography/spacing system.

Normalize visual hierarchy for:

- brand/company name
- main headline
- subheadline
- agent name
- phone
- body
- address
- helper/meta text
- envelope recipient name
- envelope address
- footer

Normalize:

- font family
- size hierarchy
- weight
- line height
- letter spacing
- spacing
- alignment
- image size/proportion

Use existing frontend/design skill or project conventions if available.

Do not add a new design-system dependency.

The goal is:

clean, intentional, professional, consistent.

Not:

a brand-new visual identity.

---

# 13. FRONT / BACK OBJECT SELECTION PARITY

The front and back A4 pages must both support selecting visible objects.

Selectable object categories include as applicable:

- text
- logo
- LINE QR
- normal image
- portrait
- sticker
- decorative object

Use one shared selection concept.

Required:

click object
→ selected outline/state

click empty canvas
→ deselect

text
→ opens text modal

image
→ image editing controls

sticker
→ resize/rotate controls

The previous ability to select objects on front/back must not be lost.

---

# 14. FULLSCREEN

Preserve V2 fullscreen behavior.

Fullscreen:

- hides icon rail
- hides temporary panels
- maximizes A4 preview
- ESC exits

Do not introduce a permanent fullscreen toolbar.

---

# 15. RECIPIENT MANAGEMENT — PRESERVE V2

Preserve the V2 recipient-management modal.

Required:

- 200+ recipient usability
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

Do not redesign this unless required to fix a regression.

---

# 16. PRINT / EXPORT — PRESERVE

Preserve existing:

- PDF export
- JPG export
- print flow

Do not rewrite PDF generation.

Do not break current export behavior while introducing object editing/persistence.

---

# STRICT SCOPE BOUNDARY

Allowed:

- frontend router cleanup if needed
- LetterView/editor components
- letter-specific CSS
- letter-specific state/persistence helpers
- IndexedDB helper
- local project-file save/load helper
- reuse existing frontend RemoveBG integration
- read/reference DM source

Not allowed:

- backend code changes
- DB/schema changes
- Docker changes
- CI changes
- deploy changes
- Staging changes
- Production changes
- auth/entitlement redesign
- new external service
- new secrets
- broad OCR page redesign

If backend change appears necessary:

HARD STOP.

Do not silently cross the boundary.

---

# FILE BUDGET / ANTI-OVERREFACTOR

Target:

4–10 frontend files.

If more than 12 tracked files appear necessary:

HARD STOP before modifying the 13th file.

Do not:

- reformat unrelated files
- rename unrelated modules
- restructure the whole frontend
- replace existing state management globally
- introduce a new framework
- introduce unnecessary dependencies

New dependency is NOT expected.

If you think one is necessary:

HARD STOP and explain why.

---

# PHASE 0 — PREFLIGHT

Before editing:

1. capture git state
2. inspect current V2 implementation
3. inspect current browser behavior
4. reproduce each reported bug
5. locate relevant DM handwriting-font implementation
6. locate relevant DM RemoveBG implementation
7. locate ORC persistence code
8. locate current image transform code
9. locate front/back selection behavior

Do not edit before you can map these areas.

---

# PHASE 1 — FIX VIEWPORT / SCROLLBARS

Fix main editor sizing first.

Acceptance:

- no body scrollbar
- no icon rail scrollbar
- no duplicate vertical scrollbar
- A4 maximized but contained within viewport

---

# PHASE 2 — UNIFY selectedObject

Establish/preserve one shared selectedObject model for front/back.

Do not yet redesign every object.

Make selection reliable first.

---

# PHASE 3 — TEXT LIVE PREVIEW + HANDWRITING FONTS

Fix working font application.

Import only the requested DM handwriting fonts.

Implement live preview in:

- text edit area
- A4 selected text

Implement proper snapshot Cancel behavior.

Keep toolbar one row.

---

# PHASE 4 — IMAGE DIRECT SELECTION + IMAGE MATERIAL CONSOLIDATION

Remove separate portrait icon if still present.

Ensure 圖片素材 contains:

- LOGO
- LINE QR
- 其他圖片
- 大頭照

Enable direct A4 image selection.

Add 製作貼紙 for 其他圖片.

---

# PHASE 5 — STICKER TRANSFORM STABILIZATION

Fix disappearing-image/rotation bug.

Use explicit transform state.

Add reset position/transform.

Test repeated mixed transform operations.

---

# PHASE 6 — FIX REMOVE BG

Compare DM working flow.

Fix ORC frontend integration only.

Verify actual transparent result visually.

---

# PHASE 7 — ADD 信封資訊 ICON

Move envelope/postal-specific settings there.

Restore:

- 印刷品 ON/OFF
- 收件人顯示: 姓氏 / 貴住戶

Verify live preview.

---

# PHASE 8 — PERSISTENCE

Implement IndexedDB autosave/restore with user isolation.

Verify:

- F5
- browser reopen
- switching/reloading route

does not lose the editor state.

Do not overwrite real state with defaults.

---

# PHASE 9 — SAVE DESIGN FILE

Implement .orcletter local file save.

Prefer File System Access API with fallback download.

Verify a real local file can be produced.

---

# PHASE 10 — TYPOGRAPHY / DEFAULT LAYOUT CLEANUP

Improve only the default template hierarchy and spacing.

Do not disturb user-loaded/saved content.

---

# PHASE 11 — REGRESSION

Validate all existing V2 behavior plus V3 fixes.

---

# HARD STOP CONDITIONS

Stop and report before broadening scope if any occurs:

1. backend change is required
2. DB/Supabase change is required
3. Docker/deploy change is required
4. new external service is required
5. new secret is required
6. more than 12 tracked files are required
7. a new major dependency is required
8. current V2 dirty working tree cannot be safely preserved
9. fixing persistence would require redesigning global app state beyond Letter scope
10. fixing RemoveBG requires changing backend endpoint contract

HARD STOP output:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS

---

# SELF-REVIEW BEFORE FINAL VALIDATION

Inspect your own diff.

Correct the implementation if:

- /ocr was reintroduced
- V2 changes were lost
- body scrollbar remains
- icon rail scrollbar remains
- text font still does not visibly change
- handwriting font option does not render
- live preview requires Apply
- Cancel does not restore snapshot
- text toolbar wraps to two rows
- portrait icon still exists separately
- 圖片素材 does not include portrait
- other image cannot become sticker
- image can disappear after rotate/resize
- no reset transform exists
- RemoveBG only returns 200 but not visible transparency
- 信封資訊 icon is missing
- 印刷品 toggle is missing
- 姓氏/貴住戶 is implemented as independent checkboxes
- F5 loses state
- Save Design only saves to localStorage/IndexedDB
- defaults overwrite real state
- front/back selection behavior differs
- backend/db/docker/deploy files changed

Run:

git diff --check

It must pass.

---

# REQUIRED LOCAL BROWSER VALIDATION

Canonical host:

http://localhost:3013/

Do not switch documentation/testing to 127.0.0.1.

## Routing

- / = OCR
- URL stays /
- /letter works
- /ocr absent
- back icon returns /

## Scrollbar

- no page vertical scrollbar
- no icon-rail vertical scrollbar
- A4 fits maximum height

## Text

- click front-page text
- edit content
- change normal font
- change DM handwriting font
- change size
- bold/italic/underline
- color
- alignment
- line height
- letter spacing

Confirm editor text and A4 both update live.

Cancel restores original.
Apply retains.

Repeat on back-page text.

## Images

- click existing LOGO directly
- click LINE QR directly
- click portrait directly
- add other image
- convert other image to sticker

## Sticker torture test

Perform repeatedly:

- rotate several times
- resize
- drag
- rotate again
- resize again
- flip
- z-order
- double click
- reset transform

Object must remain visible/recoverable.

## RemoveBG

Use a real portrait.

Verify:

- before
- run AI remove background
- actual transparent result
- result visible on A4

## Envelope info

Verify:

- 印刷品 ON/OFF changes preview
- 姓氏 mode
- 貴住戶 mode
- only one recipient-display mode active

## Persistence

Make obvious edits to:

- text
- font
- sticker transform
- recipient display setting
- 印刷品 toggle

Then:

F5

All must restore.

Close/reopen local app if practical.

All must restore.

## Save Design

Save .orcletter to local file.

Confirm actual file exists / browser reports successful save or fallback download.

## Existing regression

Verify:

- recipient management
- theme/template
- front/back preview
- image upload
- PDF
- JPG
- print
- fullscreen
- OCR home

No new relevant console errors.

Run existing frontend tests/build.

---

# FINAL STATUS RULES

PASS only if all critical items work.

If RemoveBG is the only unresolved item because the existing backend itself is objectively broken and frontend integration matches DM:

BLOCKED — REMOVE BG BACKEND CONTRACT

Do not fake PASS.

If refresh still loses state:

FAIL

If image can still disappear after transform:

FAIL

If font selection still has no visible effect:

FAIL

---

# FINAL REPORT

Return exactly:

ORC LETTER EDITOR V3 FINAL

1. SOURCE
Path:
Branch:
HEAD:
Pre-existing tracked dirty:
Pre-existing untracked:

2. ROUTING
/:
/ocr:
/letter:
Back action:

3. VIEWPORT
Body scrollbar:
Icon rail scrollbar:
Canvas fit:

4. TEXT
Normal font:
DM handwriting fonts:
Live preview:
Cancel restore:
Single-row toolbar:
Front text:
Back text:

5. OBJECT SELECTION
selectedObject:
Front:
Back:
Deselect:

6. IMAGES
Image-material contents:
Portrait separate icon:
Direct image click:
Other image -> sticker:
Transform state:
Rotation stress:
Reset transform:

7. REMOVE BG
DM reference checked:
ORC request:
Transparent result:
Browser visual result:

8. ENVELOPE INFO
Icon:
印刷品:
姓氏:
貴住戶:
Mutual exclusivity:

9. PERSISTENCE
IndexedDB:
User isolation:
F5 restore:
Browser reopen restore:
Defaults overwrite real state = NO

10. LOCAL PROJECT FILE
Extension:
Save method:
Fallback:
Actual file save verified:

11. DEFAULT DESIGN
Typography hierarchy:
Spacing:
User data preserved:

12. EXISTING FEATURES
Recipients:
Templates/themes:
PDF:
JPG:
Print:
Fullscreen:
OCR regression:

13. FILES CHANGED
- ...

14. VALIDATION
Console:
git diff --check:
Tests:
Build:

15. OUT OF SCOPE
Backend changed = NO
DB changed = NO
Docker changed = NO
Staging changed = NO
Production changed = NO
Commit/push performed = NO

16. FINAL STATUS
PASS / BLOCKED / FAIL
