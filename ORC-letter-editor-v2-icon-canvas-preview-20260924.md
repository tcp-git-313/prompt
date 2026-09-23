# ORC Letter Editor V2 — Icon Rail + Maximum A4 Canvas + Direct Object Editing

## OWNER

ORC LETTER EDITOR V2 OWNER

## MODEL

GPT-5.6 Luna — High

## EXECUTION MODE

You are a constrained frontend executor.

The product direction is already decided. Do not redesign the product beyond this specification.

This iteration is for **LOCAL DEV visual/interaction validation only** so the user can inspect the result at:

http://localhost:3013/

Do not deploy.
Do not touch Docker.
Do not touch Staging.
Do not touch Production.
Do not touch Supabase schema.
Do not touch backend OCR logic.
Do not change auth/entitlement contracts.
Do not commit or push unless the user explicitly asks later.

Canonical repo:

F:\00-Ticenpi-SaaS\TicenpiLetter

Expected current baseline:

4aa0fa9

If HEAD differs, inspect git status and report it. Never reset or discard user work.

---

# PRODUCT ROUTING — FINAL RULE

The OCR product has only one canonical home.

## Keep

/        = OCR main screen
/letter  = Letter editor

## Remove

/ocr

Do not keep /ocr as an alias.
Do not redirect / to /ocr.
Do not maintain two OCR routes.

Required result:

http://localhost:3013
http://localhost:3013/

Both render the OCR main screen and the URL stays at root.

http://localhost:3013/ocr

must no longer be a valid product route.

The Letter editor return action must navigate to:

/

Before removing /ocr, search the frontend for hard-coded "/ocr" references and replace only those that belong to the ORC product navigation. Do not modify unrelated external URLs.

---

# PRIMARY DESIGN GOAL

Rebuild /letter into a lightweight Canva/Word-style letter editor.

The priority order is:

1. A4 preview/canvas
2. direct editing on the canvas
3. compact icon tools
4. recipient management
5. secondary configuration

The A4 canvas must occupy as much useful viewport area as possible.

The current left configuration rail and top bar must not remain.

---

# FINAL PAGE STRUCTURE

The /letter page must be:

ICON RAIL + MAXIMUM A4 CANVAS

No permanent top bar.

Concept:

┌──┬───────────────────────────────────────────────────────────────┐
│← │                                                               │
│👥│                                                               │
│▦ │                                                               │
│T │                        A4 CANVAS                              │
│🖼│                                                               │
│👤│                                                               │
│💾│                                                               │
│🖨│                                                               │
│⛶ │                                                               │
└──┴───────────────────────────────────────────────────────────────┘

Use real SVG/icon components already available in the project if possible.
Do not use emoji in the final UI.

---

# ICON RAIL

Create a narrow left-side icon rail.

Target width:

52–60px

It must remain compact and vertically aligned.

Each icon MUST show a clear hover tooltip because the icon alone is not sufficient.

Required icons/actions:

1. Back
   Tooltip: 返回 OCR
   Action: navigate to /

2. Recipients
   Tooltip: 收件人
   Show recipient count badge when count > 0

3. Layout
   Tooltip: 版面與樣式

4. Text
   Tooltip: 文字與內容

5. Images
   Tooltip: 圖片素材

6. Portrait
   Tooltip: 大頭照貼紙

7. Save
   Tooltip: 儲存設計

8. Print/Export
   Tooltip: 列印 / 匯出

9. Fullscreen
   Tooltip: 全螢幕預覽

Tooltip requirements:

- appears on hover
- concise Traditional Chinese
- visually consistent
- does not cover the icon
- does not resize the rail

Do not add text labels permanently beside the icons.

---

# REMOVE PERMANENT TOP UI

Remove the permanent /letter top bar.

Do not reserve empty vertical space for:

- title bar
- subtitle bar
- page toolbar
- preview toolbar
- zoom toolbar
- fit-to-window toolbar

The A4 canvas should begin near the top of the usable viewport.

If a tiny product title is still required for context, place it discreetly inside the far-left area or as non-blocking overlay text. Do not create a horizontal top band.

---

# REMOVE PREVIEW ADJUSTMENT BAR

The user does not want the current preview adjustment bar occupying the top of the canvas.

Remove visible controls such as:

- fit-to-window
- zoom slider
- large preview toolbar
- page-control bar if it consumes permanent vertical space

The default A4 display should already be large enough.

Do not remove internal scaling calculations if they are required to render the A4 correctly.

Fullscreen is handled by the left-side icon instead.

---

# FULLSCREEN PREVIEW

The Fullscreen icon lives in the left rail.

Clicking it should enter a clean preview mode:

- hide icon rail
- hide editor panels/modals
- maximize the A4 preview
- neutral clean background
- preserve front/back preview behavior if currently supported
- ESC exits fullscreen

Do not create another top fullscreen toolbar.

---

# RECIPIENT MANAGEMENT

Do NOT show 200 recipients in the left rail.
Do NOT use a dropdown for the full recipient list.

Clicking the Recipients icon must open a large centered recipient-management modal.

Use a layout suitable for 200+ entries.

Required capabilities:

- title with recipient count
- search by name/address
- add from OCR history
- import Excel
- manual add
- list/table of recipients
- single delete
- multi-select
- select all
- clear selection
- close/apply

The left icon should show a count badge, for example:

200

The main canvas does not need to display the full list.

For large lists, use either:

- pagination
or
- virtualized/efficient scrolling

Prefer existing project dependencies. Do not add a new dependency only for virtualization.

Recipient management is data management, so a centered modal is preferred over a side panel.

---

# LAYOUT / STYLE TOOL

Clicking the Layout icon may open a compact panel from the left rail.

Include current existing functions such as:

- layout/template
- theme
- envelope style
- front/back arrangement if currently supported

This panel should be temporary and closable.
It must not permanently reduce the A4 canvas width.

Preserve current bindings and behavior.

---

# TEXT / CONTENT TOOL

Clicking the Text icon may open a compact panel for high-level content actions such as:

- letter template
- apply preset text
- reset template
- general text/content options

But detailed text editing should primarily happen by clicking the text directly on the A4 preview.

---

# ALL VISIBLE TEXT ON A4 SHOULD BE EDITABLE

The user wants all visible text to be freely editable.

Examples include:

- title
- body
- recipient name
- recipient address
- agent name
- phone
- company
- address
- LINE
- email
- footer text
- template text
- other visible text blocks

Do not restrict editing only to a hard-coded small set of fields.

However, do NOT turn the entire A4 DOM into one giant contenteditable surface.

Treat visible text blocks as editable text objects/regions so the layout remains controlled.

Prefer preserving the current document structure and adding an editing interaction layer rather than rewriting the entire rendering system.

---

# CLICK A4 TEXT -> CENTERED TEXT EDIT MODAL

When the user clicks editable text on the A4 preview:

1. visually indicate the selected text region
2. open a centered text-edit modal
3. preload the clicked text content and current formatting
4. apply changes back to the preview only when the user confirms

Do not use a tiny floating toolbar beside the text.

The user explicitly wants a centered modal because it is cleaner.

---

# TEXT EDIT MODAL SIZE

Desktop-first.

Width:

approximately 50vw

Height:

approximately 90–94vh

The modal should be vertically large and centered.

Do not make it a small dialog.

Suggested structure:

┌──────────────────────────────────────────────────────┐
│ 編輯文字                                             │
├──────────────────────────────────────────────────────┤
│ 文字內容                                             │
│ ┌──────────────────────────────────────────────────┐ │
│ │                                                  │ │
│ │             LARGE TEXT EDIT AREA                 │ │
│ │                                                  │ │
│ └──────────────────────────────────────────────────┘ │
│                                                      │
│ [字型][字級][B][I][U][色][左][中][右][行距][字距]    │
│                                                      │
│                                  [取消] [套用]       │
└──────────────────────────────────────────────────────┘

---

# TEXT CONTENT MUST BE ABOVE THE FORMATTING TOOLBAR

The order is frozen:

1. Modal title
2. "文字內容" label
3. large text editing area
4. formatting toolbar
5. cancel/apply actions

The text content area is above the formatting controls.

The text area should consume most of the modal height.

If the content is long, the text area can scroll vertically.

The formatting toolbar remains visible below it.

---

# FORMATTING TOOLBAR MUST BE ONE SINGLE ROW

This is a hard requirement.

The entire formatting tool set must stay in one horizontal row:

字型 | 字級 | B | I | U | 文字顏色 | 左對齊 | 置中 | 右對齊 | 行距 | 字距

Requirements:

- exactly one row on normal desktop width
- no wrapping to a second row
- use compact controls
- use icon buttons for B / I / U / alignment
- use compact dropdowns for font / size / line height / letter spacing
- use a compact color control
- reduce padding before allowing overflow
- if absolutely necessary as a last-resort fallback, horizontal overflow is acceptable
- normal desktop size should not show a horizontal scrollbar

Do not stack formatting controls vertically.

---

# TEXT FORMATTING FEATURES

Required:

- text content
- font family
- font size
- bold
- italic
- underline
- text color
- left align
- center align
- right align
- line height
- letter spacing

Do not add more text-formatting features in this iteration unless they already exist and can be preserved with zero complexity.

---

# IMAGE MATERIAL TOOL

Clicking the Images icon opens a temporary image-material panel.

Support current/expected image roles:

- LOGO
- LINE QR
- other template images where applicable

Each image slot should have a default placeholder image.

When the user uploads their own image:

default image
→ replaced by user image

Do not append a duplicate image beside the default.

Each image slot should support:

- replace
- remove
- restore default

Use existing project assets where appropriate.
Do not download arbitrary external assets.

---

# DEFAULT DATA / TEMPLATE CONTENT

The first-open letter editor should not look empty.

Provide sensible editable sample defaults for fields such as:

- 姓名
- 職稱
- 電話
- 公司
- 地址
- LINE
- Email
- body text

Example placeholders may look like:

姓名：王大明
職稱：不動產顧問
電話：0912-345-678
公司：XX不動產
地址：台北市○○區○○路100號
LINE：@example
Email：service@example.com

These are template/sample values only.

User edits must replace the sample values.

Do not persist fake/sample values into shared backend customer data.

Prefer local/default editor state/template defaults.

---

# DEFAULT IMAGES

Provide default/placeholder images for:

- logo
- portrait
- LINE QR

These defaults are visual placeholders.

When a user supplies their own image, it replaces the default for that slot.

Do not create duplicated stacked placeholders.

---

# PORTRAIT AS A STICKER

The current fixed portrait frame in the A4 template must be removed.

The portrait becomes a free sticker object on the A4 canvas.

Workflow:

upload/select portrait
→ remove background
→ create transparent portrait sticker
→ place on A4 canvas

The sticker must support:

- drag to move
- resize from corners
- preserve aspect ratio by default
- rotate
- horizontal flip
- bring forward
- send backward
- delete

Rotation is REQUIRED because the user may need to correct a tilted photo.

Provide a visible rotation handle or equivalent intuitive control when the sticker is selected.

Do not permanently show sticker controls when it is not selected.

---

# BACKGROUND REMOVAL SAFETY

First inspect whether the repository already has a supported remove-background capability.

If a safe existing remove-background implementation/API already exists, reuse it.

Do NOT:

- introduce a new paid external service
- add a new secret
- modify backend architecture
- invent an API endpoint
- install a large new ML dependency

If no existing safe capability exists:

do not broaden scope.

For this local visual iteration:
- implement the sticker workflow and image replacement using the uploaded image
- clearly report "background removal integration pending"
- do not fake successful background removal

This absence alone does not block the rest of the UI preview.

---

# SAVE ICON

The Save icon should preserve the current editor/design state using the existing safe persistence mechanism if one already exists.

Do not invent a new backend persistence model.

If the current app already autosaves, the icon may trigger the existing save path / explicit save confirmation.

Tooltip:

儲存設計

Do not change database contracts.

---

# PRINT / EXPORT ICON

Clicking the Print/Export icon should expose the existing export actions cleanly.

Expected actions:

- export double-sided PDF
- export JPG
- print if existing functionality supports it

Preserve existing export logic.

Do not rewrite PDF generation in this iteration.

---

# INTERACTION PANELS

General behavior:

- icon rail always compact
- only one temporary side panel should be open at a time
- centered modals are used for:
  - recipients
  - text editing
  - print/export if a modal is useful
- temporary side panels are used for:
  - layout/style
  - high-level text/template
  - image materials

Clicking the active icon again may close its panel.

ESC should close the topmost temporary panel/modal where safe.

---

# A4 CANVAS PRIORITY

The canvas must be materially larger than the current 4aa0fa9 layout.

Do not reserve permanent space for controls that can be opened on demand.

Use viewport height efficiently.

The user should immediately see a large A4 document when entering /letter.

Avoid:

- giant margins
- empty top bands
- preview toolbars
- permanent 278px settings rail
- floating translated panels

---

# CHANGE BOUNDARY

This is still a frontend-focused local preview task.

Preferred scope:

- router
- App/shared shell only if required
- LetterView
- letter-specific styles/components
- small letter-specific helper/component files if needed

Do not modify:

- backend/**
- db/**
- deploy/**
- Docker*
- CI
- Supabase migrations
- auth/entitlement
- Cloudflare
- Staging/Production

No new dependency unless absolutely unavoidable.
If you believe a dependency is necessary, HARD STOP and explain why.

---

# FILE BUDGET / ANTI-OVERREFACTOR RULE

Target tracked file changes:

4–8 frontend files

If more than 10 tracked files appear necessary:

HARD STOP before editing the 11th file.

Do not use the task as an excuse to refactor the whole frontend.

Do not rename unrelated components.
Do not reformat unrelated files.
Do not replace the existing design system.
Do not change unrelated OCR UI.

---

# GIT SAFETY

At the beginning capture:

- branch
- HEAD
- git status
- pre-existing dirty tracked files
- pre-existing untracked files

Never:

- reset
- restore unrelated files
- clean
- delete unrelated untracked files
- use git add -A

Do not commit.
Do not push.

This task is to preview and validate the UI first.

---

# PHASE 0 — INSPECT BEFORE EDITING

Inspect the actual implementation.

Identify:

- router
- OCR home route
- LetterView
- current left rail
- current top bar
- preview/A4 rendering
- front/back page rendering
- recipient logic
- theme/template logic
- text rendering
- image handling
- current portrait handling
- current export handlers
- existing save/autosave
- any existing remove-background capability

Do not assume filenames.

Record a small map of the current structure internally before making changes.

---

# PHASE 1 — ROUTE CLEANUP

Implement the final route contract:

/ = OCR home
/letter = letter editor
/ocr = removed

Update only relevant internal navigation references.

Validate URL stays at root when viewing OCR.

---

# PHASE 2 — REMOVE TOP BAR + BUILD ICON RAIL

Remove permanent /letter top bar and old permanent left configuration rail.

Build the 52–60px icon rail with hover tooltips.

Do not yet rewrite business logic.

Reuse existing handlers where possible.

---

# PHASE 3 — MAXIMIZE A4 CANVAS

Remove permanent preview toolbar/adjustment bar.

Make A4 use maximum practical workspace.

Confirm front/back preview remains functional.

Add fullscreen icon behavior.

---

# PHASE 4 — RECIPIENT MODAL

Move recipient management into a centered modal suitable for 200+ entries.

Preserve existing recipient data logic.

Add search/select/delete UI around existing data without changing data contract.

---

# PHASE 5 — TEMPORARY TOOL PANELS

Create on-demand panels for:

- layout/style
- high-level text/template
- images

Only one panel open at a time.

Keep canvas large when panels are closed.

---

# PHASE 6 — TEXT OBJECT CLICK + CENTER MODAL

Make visible A4 text regions selectable/editable.

Clicking a text region opens the centered 50vw x 90–94vh modal.

Text content appears above the formatting toolbar.

Formatting toolbar remains one single row.

Apply updates the preview.
Cancel discards unsaved modal changes.

Do not implement a giant page-wide contenteditable region.

---

# PHASE 7 — DEFAULT DATA / DEFAULT IMAGES

Add local/template defaults so the editor looks complete on first open.

Do not overwrite real loaded user data.

Rule:

real existing user/editor data wins
otherwise use safe template defaults

This rule is important to avoid another "my data disappeared" regression.

Do not clear current localStorage/session data.

Do not reset editor state on mount if data already exists.

---

# PHASE 8 — PORTRAIT STICKER

Remove the fixed portrait-frame presentation.

Convert portrait display into a sticker interaction.

Implement:

- move
- resize
- rotate
- flip
- z-order
- delete

Reuse background removal only if a safe existing capability exists.

Do not introduce backend scope.

---

# PHASE 9 — SAVE / PRINT

Wire Save and Print/Export icons to existing safe logic.

Do not redesign backend persistence or export generation.

---

# PHASE 10 — SELF REVIEW

Before final validation inspect the diff.

Reject/correct your own implementation if any of these happened:

- /ocr still exists
- / redirects to /ocr
- top bar still consumes height
- permanent wide left settings rail still exists
- icon tooltips missing
- recipient list permanently occupies the canvas
- A4 is not clearly larger
- preview toolbar still wastes vertical space
- text modal is small
- text content is below the formatting toolbar
- formatting toolbar wraps into two lines
- portrait remains trapped in fixed frame
- rotation is missing
- real existing user state gets overwritten by defaults
- backend/db/docker/deploy files changed
- unrelated OCR UI changed

Run:

git diff --check

It must pass.

---

# LOCAL VALIDATION

Use:

http://localhost:3013/

Do not switch the documented canonical host to 127.0.0.1.

This avoids localStorage/session origin confusion.

Validate:

## Routing

- / renders OCR
- URL remains /
- /letter works
- /ocr no longer acts as a product route
- Letter back icon returns to /

## OCR regression

- existing OCR home still works
- existing logged-in/local state is not cleared by this UI task
- no new relevant console errors

## Letter editor

- no permanent top bar
- 52–60px icon rail
- every icon has hover tooltip
- recipient badge works
- A4 preview is maximized
- no permanent preview adjustment bar
- fullscreen works
- recipient modal works
- 200+ layout is sensible
- layout/text/image panels open on demand
- click visible A4 text opens centered text modal
- modal width approximately 50vw
- modal height approximately 90–94vh
- text content is above toolbar
- formatting toolbar stays one row
- all required formatting controls exist
- apply/cancel behavior works
- default values do not overwrite real data
- default images are replaced by user images
- portrait sticker can move/resize/rotate
- portrait fixed frame is removed
- save/export actions remain available

## Existing behavior smoke

Verify current functions were not lost:

- recipient import/history/manual add
- template/theme
- front/back preview
- body editing
- image upload
- PDF/JPG export

Run existing relevant frontend tests/build.

Do not fix unrelated pre-existing failures.

---

# PASS / PARTIAL RULE

PASS only if all required UI behavior works.

If everything works except background removal because no safe existing capability exists:

use:

PARTIAL PASS — UI COMPLETE, BACKGROUND REMOVAL INTEGRATION PENDING

Do not mark FAIL for that one scoped integration absence.

If existing user data/session is cleared or overwritten:

FAIL

That is a regression and must be corrected before completion.

---

# FINAL REPORT

Return:

ORC LETTER EDITOR V2 FINAL

1. SOURCE
Path:
Branch:
HEAD:
Pre-existing dirty state:

2. ROUTING
/:
/ocr:
/letter:
Back action:

3. ICON RAIL
Width:
Icons:
Hover tooltips:
Recipient badge:

4. CANVAS
Top bar:
Preview toolbar:
A4 size:
Fullscreen:

5. RECIPIENTS
Modal:
Search:
200+ handling:
Delete/select/clear:

6. TEXT EDITING
Clickable text:
Modal size:
Text content position:
Single-row toolbar:
Controls:
Apply/cancel:

7. IMAGES
Defaults:
Replace behavior:
Portrait frame removed:
Sticker move:
Sticker resize:
Sticker rotate:
Sticker flip/z-order:
Background removal:

8. DATA SAFETY
Existing data preserved:
Defaults only when empty:
LocalStorage/session cleared = NO

9. SAVE / EXPORT
Save:
PDF:
JPG:
Print:

10. FILES CHANGED
- ...

11. VALIDATION
Routing:
OCR regression:
Letter UI:
Console:
git diff --check:
Tests/build:

12. OUT OF SCOPE
Backend changed = NO
DB changed = NO
Docker changed = NO
Staging changed = NO
Production changed = NO
Commit/push performed = NO

13. FINAL STATUS
PASS / PARTIAL PASS / BLOCKED / FAIL
