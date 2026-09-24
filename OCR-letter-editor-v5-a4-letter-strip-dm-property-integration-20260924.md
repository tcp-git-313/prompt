# OCR Letter Editor V5 — A4 Letter Strip + DM Property Object Integration

## OWNER

OCR LETTER EDITOR V5 OWNER

## MODEL

GPT-5.6 Luna — High

## EXECUTION MODE

You are a constrained implementation + verification agent.

This task continues from the current OCR Letter Editor V2/V3/V4 working tree.

Do NOT redesign from scratch.
Do NOT reset the working tree.
Do NOT discard current uncommitted Letter Editor work.
Do NOT refactor unrelated areas.
Do NOT change backend/DB/Docker/deploy/Staging/Production unless a HARD STOP condition is reached.

Your job is to implement the exact V5 changes below, reuse proven DM behavior where explicitly requested, and validate in the real browser.

---

# CANONICAL PROJECTS

OCR Letter project:

F:\00-Ticenpi-SaaS\TicenpiLetter

DM reference project:

F:\00-Ticenpi-SaaS\TicenpiDM

Local canonical host:

http://localhost:3013/

OCR home:

http://localhost:3013/

Letter editor:

http://localhost:3013/letter

---

# GIT SAFETY — CRITICAL

Current HEAD may still be:

4aa0fa90693347065411acf75b9f26524200d65a

with V2/V3/V4 changes uncommitted on top.

At task start capture:

- branch
- HEAD
- git status
- tracked dirty files
- untracked files

Never:

- reset
- restore current Letter changes
- checkout old versions
- clean
- delete unrelated untracked files
- use git add -A
- commit
- push

Work on top of the current working tree.

---

# EXISTING PRODUCT CONTRACT — PRESERVE

Routing:

/        = OCR main screen
/letter  = Letter editor
/ocr     = absent

Do NOT reintroduce /ocr.
Do NOT redirect / to /ocr.

Product naming:

OCR

User-visible ORC naming must remain removed.

Internal/backend compatibility identifiers containing "orc" may remain if they are actual contracts.

---

# V5 PRIMARY GOALS

This task has FOUR main workstreams:

A. Fix real hover tooltips + remove duplicate Letter Strip settings from Envelope Info

B. Rebuild 開發信條 as true A4 WYSIWYG batch preview:
   - number of strips = actual current data count
   - screen/print/PDF share one renderer
   - session-memory only

C. Fix OCR RemoveBG by comparing the proven working DM implementation

D. Replace the simplistic Add Object flow with DM-style Property Object integration:
   - paste property URL
   - extract property data
   - choose a property frame/template
   - add property object to front/back A4
   - edit/select/resize/move with shared object model

Do not broaden beyond these areas except for minimal regression fixes.

---

# A. ICON HOVER TOOLTIP — CURRENT BUG

Current report:

Hovering the left Icon Rail does NOT visibly show text.

Treat this as a BUG.

Do not count native browser title text as completion.

Implement a real visible tooltip component/style.

All main icons must show Traditional Chinese text on hover.

Expected icon tooltips include:

- 返回 OCR
- 收件人
- 新增物件
- 版面與樣式
- 文字與內容
- 圖片素材
- 信封資訊
- 開發信條
- 儲存 / 匯入
- 列印 / 匯出
- 全螢幕

Tooltip requirements:

- appears after a short hover delay around 300–500ms
- renders to the RIGHT of the icon rail
- is not clipped by overflow:hidden
- is above the A4 canvas / panels via correct z-index
- disappears when pointer leaves
- does not resize/reflow the icon rail
- remains legible in current light theme
- no permanent icon labels

Browser visual validation is required.

PASS requires actual tooltip text visible in the browser.

---

# B. 開發信條 — INDEPENDENT MAIN ICON

開發信條 remains an independent main Icon Rail entry.

Do NOT move it under Print/Export.

Hover:

開發信條

Click:

open a large centered modal immediately.

Recommended modal size:

- width: 90–94vw
- height: 90–94vh

Do not use a narrow list modal.

---

# B1. 開發信條 COUNT = REAL CURRENT DATA COUNT

Current behavior must NOT assume or manufacture 200 entries.

Rule:

current usable recipient/source data count
=
開發信條 count

Examples:

3 current records
→ 3 strips

47 current records
→ 47 strips

203 current records
→ 203 strips

Requirements:

- no fake padding rows
- no hard-coded 200
- no blank filler records
- no missing valid records
- modal header displays real count

Example:

開發信條 · 203 筆

Determine the current authoritative source of recipient data from the existing Letter Editor state.

Do not create a second independent duplicate dataset unnecessarily.

---

# B2. 開發信條 MUST LOOK LIKE A4 PREVIEW

The user wants screen preview and printed A4 to match as closely as practical.

Do NOT implement:

screen list renderer
+
separate print renderer
+
separate PDF renderer

Use ONE canonical A4 renderer.

Concept:

Recipient data
→ LetterStrip layout model
→ shared A4 page renderer
→ screen preview
→ print
→ PDF

Create/reuse a single renderer/component for the actual A4 page content.

For example conceptually:

LetterStripA4Page

The exact file/component name may differ.

Shared renderer must control:

- A4 dimensions
- strip dimensions
- font family
- font size
- line-height
- letter spacing
- alignment
- borders
- spacing
- padding
- recipient presentation
- page breaks

---

# B3. PHYSICAL A4 DIMENSIONS

Use physical print dimensions where appropriate.

A4:

210mm × 297mm

For print CSS use proper @page handling.

Example concept:

@page {
  size: A4;
  margin: 0;
}

Do not blindly copy this if existing printer margins/business requirements demand a safer explicit print margin.

The key requirement is:

screen A4 renderer and print/PDF renderer must share the SAME content/layout source.

Avoid pixel-only layout that produces materially different output on print.

---

# B4. AUTO PAGINATION BY ACTUAL STRIP SIZE

Do NOT hard-code "8 strips per page" unless the current real strip height mathematically requires exactly 8.

Calculate or deterministically derive how many strips fit on each A4 page based on the actual strip layout.

Example only:

203 entries
at 8 strips/page
→ 26 A4 pages
→ final page contains 3 strips

But actual strips-per-page must come from the real layout.

No content may be clipped across page boundaries.

No strip may straddle two printed pages.

---

# B5. 開發信條 MODAL LAYOUT

Recommended structure:

┌─────────────────────────────────────────────────────────┐
│ 開發信條 · N 筆                    [列印] [匯出 PDF]   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│                  A4 page preview                        │
│               ┌───────────────────┐                     │
│               │ strip             │                     │
│               │ strip             │                     │
│               │ strip             │                     │
│               └───────────────────┘                     │
│                                                         │
│                  A4 next page                           │
│                        ...                              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ [字型][字級][B][I][U][色][對齊][行距][字距][姓氏/貴住戶] │
└─────────────────────────────────────────────────────────┘

Requirements:

- header fixed
- A4 preview section scrolls
- bottom formatting toolbar fixed
- toolbar does not scroll away
- preview shows actual A4 pages, not a generic data table/list

---

# B6. 開發信條 FONT TOOLBAR — FIXED AT BOTTOM

Formatting toolbar stays at the bottom.

It should control the whole current Letter Strip batch by default.

Required/expected controls:

- font family
- font size
- bold
- italic
- underline
- text color where useful
- alignment
- line height
- letter spacing
- recipient display mode

Keep it compact.

Do not make 203 individually styled strip documents unless there is a pre-existing requirement.

---

# B7. 開發信條 姓氏 / 貴住戶

Inside the 開發信條 modal:

收件人顯示

● 顯示姓氏
○ 顯示「貴住戶」

Exactly one active.

This is TEMPORARY session-only state for 開發信條.

It is independent from Envelope Info's persistent/current design setting.

Do not couple them.

---

# B8. 開發信條 — PRINT + PDF

Top actions:

- 列印
- 匯出 PDF

No JPG.

Do NOT generate PDF on modal open.

Only generate when user clicks PDF.

Print/PDF must use the same A4 renderer/layout used by the on-screen preview.

Acceptance standard:

what appears on screen
≈
what is printed/exported

Minor browser/printer anti-aliasing differences are acceptable.

Layout shifts, different line wrapping, different strip placement, or missing rows are NOT acceptable.

---

# B9. 開發信條 — SESSION MEMORY ONLY

This product behavior remains mandatory.

開發信條 temporary working data must be MEMORY ONLY.

Do NOT write it to:

- localStorage
- sessionStorage
- IndexedDB
- Supabase
- database
- VPS filesystem
- .ocrletter

Lifecycle:

open /letter
→ build temporary Letter Strip session as needed

close modal
→ reopen
→ current Letter Strip session remains

switch to another Letter tool
→ return
→ session remains

NO REFRESH
→ session remains

F5 / Ctrl+R
→ Letter Strip temporary session is cleared

close tab/browser
→ session cleared

This is intentional.

Do not add delete/cleanup API calls.
Browser memory disappearing naturally is the cleanup.

Normal Letter Editor IndexedDB autosave remains separate and must continue to survive F5.

---

# B10. PERFORMANCE — DO NOT STORE 200 HEAVY PREVIEWS

For large batches, store lightweight data models only.

Avoid keeping:

- Base64 screenshots for every strip
- 200 canvas snapshots
- generated PDF binary before requested
- permanent Blob URLs

For preview, use efficient rendering.

Because the user now requires A4 page preview, use a sensible strategy such as:

- page virtualization
- windowed A4 page rendering
- lazy page rendering

Do not render hundreds of expensive offscreen pages simultaneously if it causes performance issues.

Prefer existing dependencies; do not add a new virtualization package unless absolutely necessary.

---

# C. REMOVE DUPLICATE 開發信條 FROM 信封資訊

Because 開發信條 is now its own independent main function, remove all duplicate Letter Strip controls from 信封資訊.

信封資訊 must NOT contain:

- 開發信條
- 開發信條字體
- 開發信條列印
- 開發信條 PDF
- 開發信條 settings

信封資訊 remains only for true envelope/postal settings such as:

- 寄件人姓名
- 寄件地址
- 郵遞區號
- 印刷品 ON/OFF
- envelope recipient display 姓氏 / 貴住戶
- other actual envelope-only settings

Do not remove legitimate envelope functionality.

---

# D. REMOVE BG — OCR CURRENT SERVER FLOW FAILS

Current report:

OCR RemoveBG server/integration fails.

DM currently has a working RemoveBG flow.

Use DM as the reference implementation.

Reference project:

F:\00-Ticenpi-SaaS\TicenpiDM

---

# D1. INSPECT DM WORKING REMOVEBG END TO END

Locate the actual currently working DM RemoveBG implementation.

Trace:

UI action
→ frontend function/composable
→ request URL
→ method
→ FormData shape
→ image field name
→ MIME handling
→ auth/header
→ timeout
→ server response
→ blob/base64 conversion
→ transparent result
→ UI display

Do not infer from comments.
Trace the real code path.

---

# D2. COMPARE DM VS OCR

Inspect current OCR RemoveBG flow and produce a concise internal comparison:

DM working:
- endpoint
- request
- payload
- response
- rendering

OCR failing:
- endpoint
- request
- payload
- response
- rendering

Find the actual mismatch.

---

# D3. REUSE PROVEN FLOW, DO NOT INVENT ANOTHER SERVICE

Preferred fix:

make OCR frontend integration follow the already-proven DM approach where compatible.

Do NOT:

- create a new RemoveBG server
- create a new backend endpoint
- add a new paid service
- add a new secret
- add ML dependencies
- redesign backend architecture

If DM and OCR already share a compatible existing service, use the correct frontend integration.

---

# D4. REMOVEBG HARD STOP

If OCR cannot use the proven DM flow without changing backend/service contract:

HARD STOP.

Report:

DM_WORKING_FLOW
OCR_CURRENT_FLOW
EXACT_CONTRACT_DIFFERENCE
WHY_FRONTEND_ONLY_FIX_IS_INSUFFICIENT
MINIMUM_SAFE_OPTIONS

Do not modify backend.

---

# D5. REMOVEBG BROWSER ACCEPTANCE

Use a real portrait image.

Acceptance:

original image with background
→ click AI 去背
→ request succeeds
→ resulting background is visibly transparent
→ transparent portrait appears on A4
→ portrait remains selectable
→ move
→ resize
→ rotate

200 response alone is NOT PASS.

---

# E. 新增物件 — DM-STYLE PROPERTY OBJECT INTEGRATION

The current generic Add Object model is incomplete.

The user specifically wants DM-style property-object creation.

Reference:

F:\00-Ticenpi-SaaS\TicenpiDM

Study the CURRENT DM workflow for:

property URL
→ extract property
→ normalize data
→ choose a property frame/template
→ create property card/object
→ place into design canvas

Do not invent a second property extractor if DM already has one.

---

# E1. FIRST MAP THE REAL DM FLOW

Find the actual DM code handling:

- URL input
- property extraction request/function
- supported source/site behavior
- normalized property data
- property images
- property title/name
- price
- area/ping
- layout/rooms
- address/district
- other fields DM actually supports
- property-card templates/frames
- property object creation
- design-canvas insertion

Use actual DM implementation as source of truth.

Do not assume field names.

---

# E2. REUSE / MINIMAL EXTRACT FROM DM

Goal:

avoid creating divergent property logic between DM and OCR Letter.

Prefer one of:

1. reuse existing shared function/module if already sharable
2. extract a small neutral shared frontend helper if safe
3. minimally port the proven DM frontend logic if repository architecture prevents direct sharing

Do NOT:

- rewrite the whole DM project
- modify DM behavior
- redesign scraper backend
- create a new scraper service
- duplicate extraction logic unnecessarily

If sharing requires a larger cross-repo architecture change:

HARD STOP and report minimum options.

---

# E3. ADD OBJECT MODAL — UPDATED

Left Icon Rail keeps:

新增物件

Hover:

新增物件

Click opens a centered modal.

Main options:

- 案件物件
- 文字
- 圖片
- LOGO
- LINE QR
- 大頭照
- 裝飾圖片 / existing supported decorative type

Page target:

新增到：

● 正面
○ 反面

---

# E4. PROPERTY OBJECT FLOW

When user selects:

案件物件

Show a focused property workflow.

Required concept:

案件網址
[ https://... ]

[擷取案件]

After successful extraction show a compact preview of actual extracted data.

Display fields based on real DM normalized data.

Likely examples may include:

- main image
- title
- price
- area
- room/layout
- location

But use the actual DM-supported fields.

Then:

選擇案件框架

Show existing/suitable property frames.

User picks a frame.

Then:

[加入正面]
or
[加入反面]

depending on active target.

---

# E5. PROPERTY FRAME / TEMPLATE

Do not add raw property data directly onto A4 as loose text.

Wrap property data into a designed Property Object frame.

Reference DM's existing property/card design behavior.

Property frame should control:

- image region
- title
- price
- area
- layout
- secondary metadata
- typography hierarchy
- spacing
- border/background
- crop behavior

Do not copy a frame that is inappropriate for print if DM has multiple variants; choose or adapt the most relevant print-safe frame(s).

If multiple proven frames already exist, expose a small selection.

---

# E6. PROPERTY OBJECT ON A4

After insertion, property object becomes part of the shared selectedObject system.

Required interactions:

- direct click
- selected outline
- drag/move
- resize
- bring forward
- send backward
- delete
- reset transform
- optionally change frame/template if safely supported

It must work on both:

- front
- back

Do not create a separate canvas engine for property cards.

Use the existing OCR Letter object model.

---

# E7. PROPERTY OBJECT WYSIWYG

Property object screen rendering and PDF/print rendering must use the same object renderer where possible.

Do not create:

editor property card
+
different PDF property card

The A4 preview is the source of truth.

---

# F. LOGO / QR / TRANSFORM — PRESERVE PRIOR V4 REQUIREMENTS

Do not regress these.

LOGO:

- direct select
- resize
- move
- z-order
- reset

LINE QR:

- direct select
- resize
- move
- z-order
- reset

Other image/sticker:

- resize
- move
- rotate
- flip
- z-order
- reset

Do not reintroduce the rotation-disappears bug.

If V4 acceptance for these is still incomplete, validate them during this task.

---

# G. NORMAL LETTER PERSISTENCE — PRESERVE

Normal Letter Editor state remains persistent with IndexedDB.

F5 must restore normal Letter design.

This includes current normal editor objects such as:

- text
- rich text
- images
- property objects if added
- transforms
- template/theme
- envelope info
- normal recipient settings

IMPORTANT:

開發信條 temporary session must NOT be included.

---

# H. .OCRLETTER SAVE / IMPORT — PRESERVE

Normal Letter design:

儲存 / 匯入

uses:

.ocrletter

If property objects are added, they must serialize safely in .ocrletter.

Do NOT include temporary 開發信條 session state.

---

# I. JPG REMAINS REMOVED

No JPG export.

Normal Letter:

- PDF
- print

開發信條:

- own PDF
- own print

---

# J. ICON RAIL — EXPECTED MAIN ITEMS

Expected compact main rail:

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

Every item must have real visible hover tooltip.

Do not add duplicate property/add icons unless truly required.

---

# K. UI / VIEWPORT — PRESERVE

No permanent Top Bar.

Main editor:

- 100dvh
- no body vertical scrollbar
- no icon rail scrollbar
- A4 maximized
- temporary panels/modals may scroll internally

Do not regress current editor layout while implementing V5.

---

# L. SCOPE BOUNDARY

Allowed:

- OCR frontend Letter editor
- letter-specific components/styles
- in-memory session store for 開發信條
- shared A4 Letter Strip renderer
- print/PDF frontend flow
- property-object frontend integration
- reference/read DM implementation
- minimal shared frontend helper if safe and clearly justified

Not allowed:

- backend source change
- DB/schema change
- Docker change
- CI/deploy change
- Staging/Production change
- auth/entitlement redesign
- new service
- new secret
- new scraping backend
- new RemoveBG backend
- broad DM rewrite

If a backend/service change is required:

HARD STOP.

---

# M. ANTI-OVERREFACTOR

Do not turn this task into a frontend rewrite.

Target:

6–14 OCR frontend files.

If more than 16 tracked files need modifications:

HARD STOP before changing the 17th file.

If modifying DM source becomes necessary:

HARD STOP first.

This task should primarily READ DM and implement/reuse proven patterns in OCR.

Do not silently change DM.

---

# PHASE 0 — PREFLIGHT

Before editing:

1. capture OCR git state
2. inspect current V4 working tree
3. browser-check current hover failure
4. inspect current 開發信條 implementation
5. inspect current Envelope Info duplicates
6. inspect current OCR RemoveBG failure
7. trace working DM RemoveBG
8. trace DM property URL extraction
9. trace DM property normalization
10. trace DM property frame/card rendering
11. inspect current OCR selectedObject/object persistence

Do not edit until these are mapped.

---

# PHASE 1 — TOOLTIP + DUPLICATE CLEANUP

Fix real hover tooltip.

Remove 開發信條 duplication from 信封資訊.

Browser verify both.

---

# PHASE 2 — 開發信條 DATA + A4 RENDERER

Change batch count to exact current data count.

Build/reuse a canonical A4 Letter Strip renderer.

Render actual A4 pages in modal.

Implement deterministic pagination from actual strip size.

---

# PHASE 3 — 開發信條 PRINT/PDF

Wire screen A4 renderer to print/PDF.

Ensure layout parity.

Keep session-memory-only data.

Test small batch first, then large batch.

---

# PHASE 4 — DM REMOVEBG COMPARISON / FIX

Trace DM.

Compare OCR.

Apply frontend-only minimal fix if possible.

Real browser portrait acceptance required.

---

# PHASE 5 — DM PROPERTY OBJECT INTEGRATION

Trace DM property extraction.

Implement Add Object → Property Object flow.

URL
→ extract
→ preview data
→ choose frame
→ add front/back
→ selectedObject

---

# PHASE 6 — PROPERTY OBJECT CANVAS BEHAVIOR

Validate:

- front insertion
- back insertion
- drag
- resize
- z-order
- delete
- reset
- persistence
- .ocrletter serialization

---

# PHASE 7 — REGRESSION

Validate previous OCR Letter functionality.

---

# REQUIRED BROWSER VALIDATION

Use canonical host:

http://localhost:3013/

Do not switch test documentation to 127.0.0.1.

## Hover

For EVERY icon:

hover
→ visible tooltip text

No clipping.

## Envelope Info

Confirm no 開發信條 duplicate controls remain.

Confirm legitimate envelope controls remain.

## 開發信條 counts

Test at least:

- a small actual dataset
- a larger actual dataset if available

Verify count exactly equals source data.

## 開發信條 A4 preview

Verify:

- real A4 page dimensions/ratio
- strips fit page
- no split strip
- page transitions correct
- bottom toolbar fixed

## Print

Use print preview.

Compare to on-screen A4.

No material wrapping/layout drift.

## PDF

Generate PDF.

Open output.

Compare against on-screen A4.

Verify expected entry count represented.

## Session lifecycle

Without refresh:

- close modal
- reopen
- data remains

Switch tools:
- remains

F5:
- 開發信條 temporary state clears

Normal Letter state:
- survives F5

## RemoveBG

Real portrait:

- original background
- AI remove
- actual transparency
- place on A4
- move/resize/rotate

## Property Object

Use a real supported property URL if one is available from the current DM dev/test data.

Flow:

- paste URL
- extract
- show actual property data
- select frame
- add to front
- select
- drag/resize
- add another to back or move flow to back
- validate back behavior

Do not fabricate a successful extraction.

If a real supported URL/input is required and unavailable:

PROPERTY_TEST_INPUT_REQUIRED

Report exact input needed.

## Persistence

Normal property object:
- F5
- remains

.ocrletter:
- save
- import
- property object restored

Temporary 開發信條:
- excluded

---

# TEST / BUILD

Run:

git diff --check

Existing frontend tests:

current expected baseline 36/36 PASS or better.

Production build:

PASS required.

Add focused tests where practical for:

- exact Letter Strip count
- session-only store not serialized
- property object serialization
- tooltip rendering state if existing test setup supports it

Do not add brittle screenshot tests unless existing project already uses them.

---

# HARD STOP CONDITIONS

Stop if:

1. OCR RemoveBG requires backend contract change
2. Property URL extraction requires a new scraper/backend
3. DM property extraction cannot be reused without modifying DM architecture
4. DM source must be modified
5. DB change required
6. Docker/deploy change required
7. new secret required
8. new external paid service required
9. more than 16 tracked files needed
10. existing V4 working tree cannot be preserved

Output:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS

---

# FINAL PASS RULE

PASS requires:

- all main icon hover tooltips visibly work
- duplicate 開發信條 removed from 信封資訊
- 開發信條 count = actual data count
- 開發信條 shown as A4 page preview
- screen/print/PDF share equivalent layout
- print preview matches screen closely
- PDF matches screen closely
- 開發信條 remains memory-only
- F5 clears 開發信條
- normal Letter state survives F5
- RemoveBG visually works with real portrait
- Add Object supports DM-style property URL extraction
- property frame selection works
- property object can be added to front/back
- property object is selectable/movable/resizable
- property object persists in normal draft
- property object serializes in .ocrletter
- no JPG export
- no backend/DB/Docker/deploy changes
- git diff --check PASS
- tests PASS
- build PASS

If RemoveBG or property extraction is blocked by an external/backend contract:

BLOCKED

Do not fake PASS.

---

# FINAL REPORT

Return exactly:

OCR LETTER EDITOR V5 FINAL

1. SOURCE
Path:
Branch:
HEAD:
Pre-existing tracked dirty:
Pre-existing untracked:

2. HOVER TOOLTIPS
Return OCR:
Recipients:
Add object:
Layout:
Text:
Images:
Envelope info:
Letter strip:
Save/import:
Print/export:
Fullscreen:
Clipping/z-index:

3. ENVELOPE INFO
Letter-strip duplicate removed:
Sender fields:
Printed-material toggle:
Envelope surname/貴住戶:

4. LETTER STRIP DATA
Source data:
Source count:
Generated count:
Count exact match:

5. LETTER STRIP A4
Shared renderer:
A4 dimensions:
Strips per page:
Total A4 pages:
No split strips:
Bottom toolbar fixed:

6. LETTER STRIP OUTPUT
Screen preview:
Print preview:
Print parity:
PDF generated:
PDF parity:
Expected data count represented:

7. LETTER STRIP LIFECYCLE
Memory-only:
Close/reopen preserved:
Tool switch preserved:
F5 clears:
localStorage = NO
sessionStorage = NO
IndexedDB = NO
Backend/VPS = NO
Included in .ocrletter = NO

8. REMOVE BG
DM working flow located:
OCR mismatch:
Frontend fix:
Real portrait used:
Transparent result:
A4 move/resize/rotate:
Status:

9. DM PROPERTY REFERENCE
DM extraction code path:
DM normalized fields:
DM frames/templates:
Reuse strategy:

10. ADD PROPERTY OBJECT
URL input:
Extraction:
Extracted preview:
Frame selection:
Add front:
Add back:
Direct selection:
Move:
Resize:
Z-order:
Delete:
Reset:

11. PROPERTY PERSISTENCE
IndexedDB restore:
F5 restore:
.ocrletter save:
.ocrletter import:
Letter-strip session excluded:

12. REGRESSION
OCR home:
Rich text:
Images:
LOGO resize:
QR resize:
Sticker transforms:
Fullscreen:
Normal PDF:
Normal print:
JPG = REMOVED:

13. FILES CHANGED
- ...

14. VALIDATION
Console:
Network:
git diff --check:
Tests:
Build:

15. OUT OF SCOPE
Backend changed = NO
DB changed = NO
Docker changed = NO
CI/deploy changed = NO
Staging changed = NO
Production changed = NO
DM source changed = NO
Commit/push performed = NO

16. FINAL STATUS
PASS / BLOCKED / FAIL
