# OCR Letter Editor V5 — A4 Letter Strip + DM Property Object Integration

## OWNER

OCR LETTER EDITOR V5 OWNER

## MODEL

GPT-5.6 Luna — High

## EXECUTION MODE

You are a constrained implementation + browser-verification agent.

This task continues directly from the current OCR Letter Editor working tree after V4 Acceptance.

Do NOT restart V4.
Do NOT redo already-passed editor mechanics.
Do NOT redesign the editor from scratch.
Do NOT reset or discard the current working tree.
Do NOT commit or push.
Do NOT touch backend / DB / Docker / deploy / Staging / Production unless a HARD STOP condition is reached.

Your job is to implement the remaining V5 product changes and verify them in the real browser.

---

# CANONICAL PROJECTS

OCR Letter project:

F:\00-Ticenpi-SaaS\TicenpiLetter

DM reference project:

F:\00-Ticenpi-SaaS\TicenpiDM

Canonical local host:

http://localhost:3013/

OCR home:

http://localhost:3013/

Letter editor:

http://localhost:3013/letter

---

# CURRENT AUTHORITATIVE BASELINE — V4 ACCEPTANCE

Use this as the starting state.

Source:

Path:
F:\00-Ticenpi-SaaS\TicenpiLetter

Branch:
master

HEAD:
4aa0fa90693347065411acf75b9f26524200d65a

V4 Acceptance reported:

LOGO:
- direct select PASS
- resize smaller PASS
- resize larger PASS
- move PASS
- F5 restore PASS
- LOGO_RESIZE PASS

LINE QR:
- direct select PASS
- resize smaller PASS
- resize larger PASS
- move PASS
- F5 restore PASS
- QR_RESIZE PASS

Sticker:
- repeated rotate PASS
- resize PASS
- drag PASS
- flip PASS
- z-order PASS
- double click PASS
- remains visible PASS
- reset transform PASS
- F5 restore PASS
- STICKER_TRANSFORM_STRESS PASS

Normal Letter persistence:
- IndexedDB/F5 restore PASS

開發信條 lifecycle:
- memory-only PASS
- close/reopen preserved PASS
- F5 clears PASS

Routing:
- / = OCR
- /ocr absent
- /letter = Letter editor
- back = /

Validation:
- git diff --check PASS
- frontend tests 36/36 PASS
- build PASS

Do NOT spend this task rebuilding these already-passed mechanics.

Only run a light regression smoke after V5 changes.

---

# V4 ITEMS THAT REMAIN UNVERIFIED

## RemoveBG

Previous fixture:

photo.png

was only 30 bytes and was NOT a real portrait.

Therefore previous status was:

REMOVE_BG_INPUT_REQUIRED

This is NOT proof that the V4 transform/editor mechanics are broken.

V5 must verify RemoveBG only with a real portrait image.

## Old fixed-200 開發信條 PDF test

Previous V4 acceptance attempted a fixed 200-entry PDF and timed out.

That acceptance scenario is now obsolete.

V5 product behavior is:

actual current data count N
→ generate exactly N 開發信條

Do NOT preserve a hard-coded 200-entry product assumption.

Do NOT spend time fixing the old fixed-200 implementation as a separate feature.

V5 replaces it with the A4 WYSIWYG Letter Strip implementation below.

---

# GIT SAFETY

At task start capture:

- branch
- HEAD
- git status
- tracked dirty files
- untracked files

Current V2/V3/V4 work is expected to be uncommitted.

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

# PRODUCT CONTRACT — PRESERVE

Routing:

/        = OCR main screen
/letter  = Letter editor
/ocr     = absent

Do NOT reintroduce /ocr.
Do NOT redirect / to /ocr.

Product naming:

OCR

User-visible ORC naming must remain removed.

Backend/internal compatibility identifiers containing "orc" may remain if they are actual contracts.

---

# V5 PRIMARY WORKSTREAMS

There are FOUR required workstreams:

A. Fix real Icon Rail hover tooltips + remove duplicate 開發信條 controls from 信封資訊

B. Rebuild 開發信條 into true A4 WYSIWYG batch preview:
   - current data count = generated strip count
   - screen / print / PDF share the same renderer
   - session-memory only
   - no fixed 200 assumption

C. Fix and verify OCR RemoveBG using DM's proven working implementation as reference

D. Upgrade 新增物件 to support DM-style property object integration:
   - paste property URL
   - extract property data
   - choose a frame/template
   - add property object to front/back A4
   - use existing selectedObject mechanics

Do not broaden beyond these four workstreams except for minimum regression fixes.

---

# A1. ICON HOVER TOOLTIP — REAL BROWSER BUG

Current report:

Hovering the left Icon Rail does not visibly show text.

Treat this as a real UI bug.

Native browser title text is NOT sufficient.

Implement a real visible tooltip.

Required tooltips:

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

Requirements:

- appears after a short hover delay around 300–500ms
- renders to the RIGHT of the icon rail
- not clipped by overflow:hidden
- correct z-index above canvas/panels
- disappears when pointer leaves
- does not resize/reflow icon rail
- readable in current light theme
- no permanent text labels

PASS requires actual visible text in the real browser.

---

# A2. REMOVE DUPLICATE 開發信條 FROM 信封資訊

Because 開發信條 is now an independent main icon, 信封資訊 must not contain duplicate Letter Strip controls.

Remove from 信封資訊:

- 開發信條
- 開發信條字體
- 開發信條列印
- 開發信條 PDF
- any other 開發信條-only control

Preserve true envelope/postal settings:

- 寄件人姓名
- 寄件地址
- 郵遞區號
- 印刷品 ON/OFF
- envelope recipient display 姓氏 / 貴住戶
- other legitimate envelope-only settings

Do not remove valid envelope functionality.

---

# B1. 開發信條 — INDEPENDENT MAIN ICON

Keep 開發信條 as its own main Icon Rail item.

Do NOT move it under Print/Export.

Clicking it opens a large centered modal immediately.

Recommended modal size:

- width: 90–94vw
- height: 90–94vh

---

# B2. 開發信條 COUNT = ACTUAL CURRENT DATA COUNT

This is a HARD REQUIREMENT.

Use the current authoritative recipient/source data already present in Letter Editor.

Rule:

valid current source record count N
=
generated 開發信條 count N

Examples:

3 records
→ 3 strips

47 records
→ 47 strips

203 records
→ 203 strips

Do NOT:

- hard-code 200
- generate fake filler records
- pad to a page
- create blank extra strips
- drop valid records

Modal header must show the real count.

Example:

開發信條 · 47 筆

If there are zero current valid records:

show an empty-state message and do not fabricate strips.

---

# B3. A4 WYSIWYG IS THE SOURCE OF TRUTH

The user requires the on-screen A4 preview to match print/PDF as closely as practical.

Do NOT maintain:

screen-list renderer
+
separate print layout
+
separate PDF layout

Use ONE canonical Letter Strip A4 renderer/layout model.

Concept:

current recipient data
→ LetterStrip layout model
→ shared A4 page renderer
→ screen preview
→ print
→ PDF

The same renderer/layout rules must determine:

- page size
- strip size
- typography
- line wrapping
- padding
- borders
- spacing
- recipient display
- page breaks

The browser A4 preview is the visual source of truth.

---

# B4. PHYSICAL A4 DIMENSIONS

Use print-safe physical dimensions where appropriate.

A4:

210mm × 297mm

Use proper print CSS / @page handling.

Do not blindly force zero margins if the existing print architecture requires safe printer margins.

The key requirement is:

screen preview
≈
print preview
≈
PDF

No material differences in:

- line wrapping
- strip placement
- clipping
- font sizing
- page break locations

---

# B5. PAGINATION BY REAL STRIP SIZE

Do NOT hard-code a fixed strips-per-page count unless the actual layout mathematically results in that count.

Derive pagination from the real strip height/layout.

Requirements:

- no strip split across two pages
- no clipped strip
- no phantom blank strip
- final page contains only remaining real records

If a page fits 8 strips and source has 19:

page 1 = 8
page 2 = 8
page 3 = 3

But "8" is only an example.

---

# B6. 開發信條 MODAL LAYOUT

Target structure:

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

- header stays visible
- middle A4 preview area scrolls
- bottom formatting toolbar stays fixed
- preview shows actual A4 pages, not a generic list/table

---

# B7. 開發信條 TOOLBAR — FIXED AT BOTTOM

Formatting toolbar applies to the current Letter Strip batch by default.

Controls:

- font family
- font size
- bold
- italic
- underline
- text color where applicable
- alignment
- line height
- letter spacing
- recipient display mode

Keep it compact.

Do not create N independently styled documents unless existing business logic already requires that.

---

# B8. 開發信條 姓氏 / 貴住戶

Inside 開發信條 modal:

收件人顯示

● 顯示姓氏
○ 顯示「貴住戶」

Exactly one active.

This state is:

- specific to 開發信條
- session-memory only
- independent from 信封資訊 recipient-display setting

Do not couple the two.

---

# B9. PRINT + PDF

Top actions:

- 列印
- 匯出 PDF

No JPG.

Do NOT generate PDF on modal open.

Generate only after explicit user action.

Print/PDF must use the same A4 layout rules as the screen preview.

Do not reintroduce the obsolete V4 requirement of "always generate exactly 200".

Acceptance uses the actual current data count.

---

# B10. SESSION MEMORY ONLY

開發信條 is temporary working-session data.

Do NOT write it to:

- localStorage
- sessionStorage
- IndexedDB
- Supabase
- DB
- VPS filesystem
- .ocrletter

Lifecycle:

open /letter
→ use current working session

open 開發信條
→ build temporary view/session state

close modal
→ reopen
→ temporary session remains

switch tools
→ return
→ session remains

F5 / Ctrl+R
→ temporary 開發信條 session clears

close tab/browser
→ clears

No cleanup API call is required.

Normal Letter Editor IndexedDB persistence remains separate and must continue to survive F5.

---

# B11. PERFORMANCE

Store lightweight data models.

Avoid:

- Base64 screenshots per strip
- canvas snapshots per strip
- pre-generated PDF blobs
- hundreds of permanent Blob URLs

If many A4 pages exist, use efficient page rendering such as:

- page virtualization
- lazy page rendering
- windowed page rendering

Prefer existing dependencies.
Do not add a new virtualization library unless unavoidable.

---

# C1. REMOVE BG — USE DM AS PROVEN REFERENCE

Current OCR RemoveBG remains unverified because the previous test asset was invalid.

DM currently has a working RemoveBG flow.

Reference:

F:\00-Ticenpi-SaaS\TicenpiDM

Trace the REAL working DM code path:

UI action
→ frontend function/composable
→ request URL
→ HTTP method
→ FormData shape
→ image field name
→ MIME
→ auth/header behavior
→ timeout
→ response schema
→ blob/base64 handling
→ transparent image rendering

Do not infer from comments.

---

# C2. COMPARE DM VS OCR

Before changing code, map:

DM working flow:
- endpoint
- request
- payload
- response
- display

OCR current flow:
- endpoint
- request
- payload
- response
- display

Identify the exact mismatch.

---

# C3. FRONTEND-ONLY FIX PREFERRED

If OCR can use the already-proven DM service/contract:

reuse the proven frontend integration pattern.

Do NOT:

- build a new RemoveBG server
- create a new backend endpoint
- add a new paid service
- add a new API secret
- add ML dependencies
- redesign backend

---

# C4. REAL PORTRAIT INPUT REQUIRED

Do not reuse the invalid 30-byte photo.png fixture.

Use a real portrait image available in current test assets or explicitly provided for browser testing.

If no valid real portrait is available:

do NOT fake success.

Report:

REMOVE_BG_INPUT_REQUIRED

with the exact required input.

Continue other V5 work.

Final overall status may be BLOCKED only on this acceptance item if everything else passes.

---

# C5. REMOVE BG BROWSER ACCEPTANCE

With a real portrait:

1. show original image with background
2. click AI 去背
3. request completes
4. background is visibly transparent
5. result appears on A4
6. direct select works
7. move works
8. resize works
9. rotate works

200 response alone is NOT PASS.

---

# C6. REMOVE BG HARD STOP

If success requires changing backend/service contract:

HARD STOP on this workstream.

Report:

DM_WORKING_FLOW
OCR_CURRENT_FLOW
EXACT_CONTRACT_DIFFERENCE
WHY_FRONTEND_ONLY_FIX_IS_INSUFFICIENT
MINIMUM_SAFE_OPTIONS

Do not modify backend.

---

# D1. 新增物件 — DM-STYLE PROPERTY OBJECT

The user does NOT want only a generic "add image/text" flow.

新增物件 must support a DM-style property object workflow.

Reference:

F:\00-Ticenpi-SaaS\TicenpiDM

Study the current DM flow:

property URL
→ extract property
→ normalize data
→ select property frame/template
→ create property object
→ insert into canvas

Do not invent a second property extractor if DM already has one.

---

# D2. MAP THE REAL DM PROPERTY FLOW FIRST

Locate the actual DM code handling:

- URL input
- extraction request/function
- supported sites
- normalized property fields
- images
- title
- price
- area/ping
- layout/rooms
- address/district
- any other real supported fields
- property frames/templates
- property object creation
- canvas insertion

Use DM implementation as source of truth.

Do not assume field names.

---

# D3. REUSE STRATEGY

Preferred order:

1. reuse existing shared helper/module if already shareable
2. extract a very small neutral frontend helper only if safe
3. minimally port proven DM frontend logic into OCR when cross-repo sharing is impractical

Do NOT:

- modify DM behavior
- rewrite DM
- create a new scraper backend
- duplicate scraper logic unnecessarily
- introduce a second normalization contract

If safe reuse requires a broad cross-repo architecture change:

HARD STOP and report options.

---

# D4. 新增物件 MODAL

Keep the independent main icon:

新增物件

Hover tooltip:

新增物件

Click opens a centered modal.

Page target:

新增到：
● 正面
○ 反面

Main options:

- 案件物件
- 文字
- 圖片
- LOGO
- LINE QR
- 大頭照
- 裝飾圖片 / currently-supported decorative object

---

# D5. PROPERTY OBJECT FLOW

When 案件物件 is selected:

案件網址
[ https://... ]

[擷取案件]

After successful extraction:

show an actual extracted-data preview.

Display only fields actually produced by the DM normalized contract.

Then:

選擇案件框架

Show available suitable property frames/templates.

Then:

加入正面
or
加入反面

based on selected target.

Do not fabricate success if the URL extraction fails.

---

# D6. PROPERTY FRAME

Do not add raw extracted text loosely to A4.

Create one Property Object using a property frame/template.

Frame controls:

- main image region
- title
- price
- area
- layout
- secondary metadata
- typography hierarchy
- spacing
- background/border
- crop behavior

Reference DM's proven card/frame design.

Use print-safe frame behavior.

If multiple suitable existing frames are available, expose a small selection.

---

# D7. PROPERTY OBJECT USES EXISTING OCR selectedObject

Do NOT create a new canvas engine.

After insertion, property object participates in the existing shared object model.

Required:

- direct click
- selected outline
- move
- resize
- bring forward
- send backward
- delete
- reset transform
- front page
- back page

Optional:

- change frame/template after insertion, only if safe

---

# D8. PROPERTY OBJECT WYSIWYG

Screen A4 rendering and normal Letter PDF/print must use the same Property Object renderer/layout where practical.

Do not create:

editor card
+
different PDF card

A4 preview remains source of truth.

---

# D9. PROPERTY TEST INPUT

Use a real currently supported property URL if available from DM dev/test context.

Do not fabricate extraction results.

If no usable URL is available:

report:

PROPERTY_TEST_INPUT_REQUIRED

with the exact type of supported URL needed.

Continue other workstreams.

---

# E. ALREADY-PASSED V4 MECHANICS — REGRESSION SMOKE ONLY

Do NOT rebuild these:

- LOGO resize
- LINE QR resize
- sticker rotate
- sticker resize
- sticker drag
- sticker flip
- sticker z-order
- reset transform
- normal Letter F5 restore
- 開發信條 memory-only lifecycle

After V5 changes, perform a short regression smoke only.

If a regression is discovered, fix the minimum regression.

---

# F. NORMAL LETTER PERSISTENCE

Normal Letter Editor remains persistent using IndexedDB.

Normal Letter data should survive F5.

If Property Objects are added, they must serialize into normal Letter persistence.

開發信條 temporary session must NOT serialize into IndexedDB.

---

# G. .OCRLETTER SAVE / IMPORT

Normal Letter design continues to support:

儲存 / 匯入

using:

.ocrletter

Property Objects must serialize/restore safely.

開發信條 session state must remain excluded.

---

# H. JPG REMAINS REMOVED

Normal Letter:

- PDF
- print

開發信條:

- own PDF
- own print

No JPG export.

---

# I. ICON RAIL — EXPECTED ORDER

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

Every item must have a real visible hover tooltip.

No separate portrait icon.

---

# J. VIEWPORT — PRESERVE

No permanent Top Bar.

Main editor:

- 100dvh
- no body vertical scrollbar
- no icon rail scrollbar
- A4 maximized within viewport
- temporary modals/panels may scroll internally

Do not regress current editor layout.

---

# SCOPE BOUNDARY

Allowed:

- OCR frontend Letter editor
- Letter-specific components/styles
- real tooltip implementation
- Letter Strip in-memory session state
- shared A4 Letter Strip renderer
- print/PDF frontend flow
- Property Object frontend integration
- reading/reference of DM implementation
- minimal shared frontend helper only if safe and clearly justified

Not allowed:

- backend source change
- DB/schema change
- Docker change
- CI/deploy change
- Staging/Production change
- auth/entitlement redesign
- new external service
- new secret
- new scraper backend
- new RemoveBG backend
- DM source modification without prior HARD STOP

---

# ANTI-OVERREFACTOR

Target:

6–14 OCR frontend files.

If more than 16 tracked files need modification:

HARD STOP before changing file 17.

If modifying DM source becomes necessary:

HARD STOP first.

Do not silently alter DM.

---

# PHASE 0 — PREFLIGHT

Before editing:

1. capture OCR git state
2. inspect current V4 working tree
3. reproduce current tooltip failure
4. inspect current 開發信條 implementation
5. identify exact authoritative source-data array/list
6. inspect duplicate controls in 信封資訊
7. inspect current OCR RemoveBG path
8. trace working DM RemoveBG path
9. trace DM property URL extraction
10. trace DM normalized property data
11. trace DM property frames/cards
12. inspect current OCR selectedObject + persistence

Do not edit until this map is understood.

---

# PHASE 1 — TOOLTIP + DUPLICATE CLEANUP

Implement real hover tooltips.

Remove duplicate 開發信條 controls from 信封資訊.

Browser verify both.

---

# PHASE 2 — LETTER STRIP A4 WYSIWYG

Implement:

actual source count
→ exact strip count
→ A4 pages

Create/reuse one canonical A4 renderer/layout model.

No fixed 200 behavior.

---

# PHASE 3 — LETTER STRIP PRINT / PDF

Wire the same A4 layout to:

- on-screen preview
- print
- PDF

Validate visual parity.

Keep temporary state memory-only.

---

# PHASE 4 — REMOVE BG

Trace DM.
Compare OCR.
Apply frontend-only fix if compatible.
Use a real portrait for final acceptance.

If no real portrait exists:
REMOVE_BG_INPUT_REQUIRED

---

# PHASE 5 — DM PROPERTY OBJECT INTEGRATION

Trace DM property flow.

Implement:

URL
→ extract
→ preview
→ frame
→ add front/back
→ selectedObject

---

# PHASE 6 — PROPERTY OBJECT PERSISTENCE

Validate:

- front insertion
- back insertion
- move
- resize
- z-order
- delete
- reset
- F5 restore
- .ocrletter save/import

---

# PHASE 7 — REGRESSION SMOKE

Only regression-smoke already-passed V4 mechanics.

Do not rerun a broad V4 implementation task.

---

# REQUIRED BROWSER VALIDATION

Canonical host:

http://localhost:3013/

Do not change canonical documentation to 127.0.0.1.

## Tooltips

Hover every main icon.

Visible tooltip required.

## Envelope Info

No 開發信條 duplicate controls.

True envelope controls remain.

## Letter Strip count

Verify:

source count N
=
generated count N

No fabricated records.

## Letter Strip A4

Verify:

- actual A4 ratio/dimensions
- correct page grouping
- no strip splits
- no clipping
- final page contains only real remaining records
- bottom toolbar fixed

## Print preview

Compare against on-screen A4.

No material layout drift.

## PDF

Generate PDF from the current real dataset.

Open it.

Verify:

- expected records represented
- page layout matches preview closely
- no blank/fake records
- no corrupt output

Do not require exactly 200 unless current real dataset actually has 200 records.

## Letter Strip lifecycle

Without refresh:
- close/reopen modal → remains
- switch tools → remains

F5:
- temporary 開發信條 state clears

Normal Letter state:
- survives F5

## RemoveBG

Use real portrait.

If available:

- original background visible
- AI remove
- actual transparent result
- A4 placement
- move/resize/rotate

If no real portrait:
- report REMOVE_BG_INPUT_REQUIRED
- do not fake PASS

## Property Object

Use a real supported property URL if available.

- paste URL
- extract actual property
- show normalized preview
- choose frame
- add to front
- select/move/resize
- add or test on back
- F5 restore
- save/import .ocrletter

If no usable URL:
- report PROPERTY_TEST_INPUT_REQUIRED
- do not fabricate success

---

# VALIDATION

Run:

git diff --check

Expected frontend test baseline:

36/36 PASS or better.

Run production frontend build.

PASS required.

Capture relevant browser console/network errors for V5 flows.

Do not fix unrelated pre-existing warnings.

---

# HARD STOP CONDITIONS

Stop before broadening scope if:

1. RemoveBG requires backend contract change
2. Property extraction requires new scraper/backend
3. DM source must be modified
4. DM property flow cannot be reused without broad architecture changes
5. DB change required
6. Docker/deploy change required
7. new secret required
8. new paid/external service required
9. more than 16 tracked files required
10. current V4 working tree cannot be preserved

Output:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS

---

# FINAL STATUS RULES

PASS only if all V5 requirements are verified.

If everything implementable passes but a real portrait is unavailable:

BLOCKED — REMOVE_BG_INPUT_REQUIRED

If everything implementable passes but a real supported property URL is unavailable:

BLOCKED — PROPERTY_TEST_INPUT_REQUIRED

If both inputs are unavailable:

BLOCKED — TEST_INPUT_REQUIRED

Do not call these FAIL unless the feature is actually demonstrated broken.

FAIL when implementation is demonstrably broken after available test input is used.

---

# FINAL REPORT

Return exactly:

OCR LETTER EDITOR V5 FINAL

1. SOURCE
Path:
Branch:
HEAD:
Tracked dirty before:
Tracked dirty after:
Files changed by V5:

2. V4 BASELINE PRESERVED
LOGO resize regression:
QR resize regression:
Sticker transform regression:
Normal Letter F5 regression:
開發信條 memory-only regression:

3. HOVER TOOLTIPS
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

4. ENVELOPE INFO
Letter-strip duplicate removed:
Sender fields:
Printed-material toggle:
Envelope surname/貴住戶:

5. LETTER STRIP DATA
Authoritative source:
Source count:
Generated count:
Exact match:

6. LETTER STRIP A4
Shared renderer:
A4 dimensions:
Actual strips/page:
Total pages:
No split strips:
No fake/blank records:
Bottom toolbar fixed:

7. LETTER STRIP OUTPUT
Screen preview:
Print preview:
Print parity:
PDF generated:
PDF opens:
PDF parity:
Expected source records represented:

8. LETTER STRIP LIFECYCLE
Memory-only:
Close/reopen preserved:
Tool switch preserved:
F5 clears:
localStorage = NO
sessionStorage = NO
IndexedDB = NO
Backend/VPS = NO
Included in .ocrletter = NO

9. REMOVE BG
DM working path:
OCR mismatch:
Frontend-only fix:
Real portrait available:
Transparent visual result:
A4 move/resize/rotate:
Status:

10. DM PROPERTY REFERENCE
DM extraction path:
Supported URL/source:
Normalized fields:
Frames/templates:
Reuse strategy:

11. PROPERTY OBJECT
Real URL available:
URL extraction:
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

12. PROPERTY PERSISTENCE
IndexedDB:
F5 restore:
.ocrletter save:
.ocrletter import:
Letter Strip excluded:

13. REGRESSION
OCR home:
Rich text:
Images:
Fullscreen:
Normal PDF:
Normal print:
JPG = REMOVED:

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
