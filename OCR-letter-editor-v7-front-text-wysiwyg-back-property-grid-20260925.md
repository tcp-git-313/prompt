# OCR Letter Editor V7 — Front Text WYSIWYG + Back 2×3 Property Grid

## OWNER

OCR LETTER EDITOR V7 OWNER

## MODEL

GPT-5.6 Luna — High

## OBJECTIVE

Implement ONLY the following two product changes on top of the VERIFIED accepted V5 baseline:

1. Front-page `frontBelow.text` must use a stable 706px logical text box, and its text-edit modal must render with the SAME logical box/rendering so screen editing and A4 preview do not drift.

2. Back page must use a DM-referenced property workflow:
   URL extraction → property preview → frame selection → insert into a structured property grid.
   Default back-page layout = 2 columns × 3 rows.

This is NOT a redesign task.

Do not touch unrelated Letter functionality.

---

# CANONICAL PROJECTS

OCR Letter:

F:\00-Ticenpi-SaaS\TicenpiLetter

DM reference:

F:\00-Ticenpi-SaaS\TicenpiDM

Canonical local host:

http://localhost:3013/

Letter:

http://localhost:3013/letter

DM is READ ONLY.

---

# PRECONDITION — VERIFIED V5 BASELINE REQUIRED

Before editing, prove the current source is the trusted accepted V5 baseline or a clean descendant of it.

Check:

- branch
- HEAD
- git status
- latest local checkpoint
- whether rejected V6 layout code is present

If there is no verified V5 checkpoint/baseline:

HARD STOP.

Return:

V5_BASELINE_REQUIRED

Do NOT implement V7 on an ambiguous/broken tree.

Do NOT reset to historical old HEAD `4aa0fa9`.

---

# GIT POLICY

At task start:

- record branch
- record HEAD
- record git status
- record dirty/untracked files

Do not:

- git reset --hard to an unverified SHA
- git restore .
- git checkout -- .
- git clean
- delete untracked files
- overwrite unrelated work
- push

If the V5 baseline is accepted and current worktree is clean, proceed.

At the END:

If FINAL STATUS = PASS:

- run tests/build
- create a LOCAL checkpoint commit automatically
- report commit SHA
- DO NOT PUSH

Recommended commit message:

feat: OCR Letter front WYSIWYG and back property grid

If FAIL/BLOCKED:

- DO NOT commit over the accepted baseline

---

# SCOPE

Allowed:

- LetterView / Letter-specific frontend files
- shared front text renderer
- text editor modal
- front/back renderers
- property workflow UI
- property-grid state
- property persistence / .ocrletter serializer if needed
- styles directly related to these changes
- read-only inspection of DM source

Not allowed:

- OCR home redesign
- unrelated image/sticker redesign
- recipient redesign
- Letter Strip redesign
- Save/Import redesign
- backend scraper rewrite
- new scraping service
- new external provider
- DB/schema changes
- Docker/deploy changes
- Staging/Production changes
- DM source modification

---

# PART A — FRONT `frontBelow.text`

## A1. AUTHORITATIVE TARGET

Observed current A4 DOM:

```html
<div
  class="letter-body text-object-selected"
  data-editable-text="true"
  data-edit-key="frontBelow.text"
  data-edit-side="front"
>
  <p
    data-selected="true"
    data-label-id="0"
    style="height: 706px; transform: translate(0px, -19px);"
  >
    ...
  </p>
</div>
```

Required logical text height:

706px

The width must NOT be guessed from prompt text.

Determine the real current width from the actual front A4 `frontBelow.text` layout box.

Use that measured/source-defined width as the canonical logical width.

---

# A2. DO NOT BLINDLY COPY `translateY(-19px)`

The observed:

```css
transform: translate(0px, -19px)
```

must be investigated before preservation.

Determine whether `-19px` is:

A. intentional template/layout offset

or

B. stale object transform / drag residue / accidental state

If A:
- preserve it as part of the canonical layout metrics

If B:
- normalize the object/layout
- do NOT propagate the stale transform into the modal

Report the evidence.

Do not guess.

---

# A3. SINGLE CANONICAL TEXT BOX

Target model:

```
frontBelow.text
        ↓
RichText model
        ↓
Canonical Text Box Metrics
        ↓
Shared Text Renderer
   ├─ Front A4 preview
   └─ Text Edit Modal
```

The A4 and modal must not maintain independent width/height/wrapping implementations.

---

# A4. LOGICAL METRICS

Canonical front text box:

- height = 706 logical px
- width = actual current A4 frontBelow available text width
- font/line-height/letter-spacing from current text style
- same padding
- same paragraph spacing
- same wrapping behavior
- same Rich Text runs

Current observed base style remains:

```
font-family: "Noto Sans TC", sans-serif
font-size: 20px
font-weight: 400
font-style: normal
text-decoration: none
color: #172033
text-align: left
line-height: 1.5
letter-spacing: 0px
```

Do not overwrite user-selected Rich Text ranges with this base style.

---

# A5. MODAL MUST USE THE SAME LOGICAL BOX

The text editor modal must render the selected text inside the SAME logical box:

```
width  = same as A4 frontBelow text box
height = 706 logical px
```

If the physical modal cannot display a 706px canvas at 1:1 scale:

DO NOT change the logical dimensions.

Instead:

```
canonical logical box
→ scale visually to fit modal viewport
```

The scale must affect the entire editing canvas consistently.

Goal:

- same line wrapping
- same paragraph breaks
- same visible line positions
- same font metrics
- same spacing

Minor anti-aliasing differences are acceptable.

Different wrapping is NOT.

---

# A6. EDITING SURFACE

Do not use a generic textarea whose layout differs from A4.

Use the shared Rich Text renderer/editor in editable mode.

The edit modal must remain the actual editing canvas.

Do not re-add labels such as:

- 直接編輯目前選取的 A4 文字物件
- 文字內容

---

# A7. RICH TEXT BEHAVIOR — PRESERVE

Keep V5 behavior:

Selected range exists:
→ formatting applies only to selected range

No selection:
→ formatting applies to entire text object

Keep:

- font
- font size
- B
- I
- U
- color
- alignment
- line-height
- letter-spacing
- handwriting fonts

Toolbar stays one row on normal desktop.

---

# A8. MODAL TRANSACTION RULES — PRESERVE

Text modal closes ONLY by:

- 套用
- 取消

Backdrop:
NO CLOSE

Click blank A4:
NO CLOSE

ESC:
NO CLOSE

Apply:
commit current draft to designDoc

Cancel:
restore exact modal-open snapshot

Live preview:
YES

A4 must update while editing.

---

# A9. FRONT TEXT ACCEPTANCE TEST

Use obvious multi-line test content:

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
```

Verify:

- A4 text box logical height = 706px
- modal logical height = 706px
- modal width = same logical A4 width
- line positions/wrapping closely match
- typing in modal immediately appears in A4
- Apply persists
- Cancel rolls back
- F5 restores committed content
- no duplicate `frontBelow.text` visible renderer

In browser console verify:

```js
document.querySelectorAll(
  '[data-edit-key="frontBelow.text"][data-edit-side="front"]'
)
```

Normal visible renderer count must be 1.

---

# PART B — BACK PAGE PROPERTY SYSTEM

## B1. DM IS THE WORKFLOW REFERENCE

Read:

F:\00-Ticenpi-SaaS\TicenpiDM

Trace the CURRENT actual DM property workflow.

At minimum identify:

- URL input UI
- scrape/extraction call
- request contract
- normalized property payload
- image list
- title
- price
- area/ping
- layout/rooms
- location/address
- any other supported fields
- property frames/templates
- frame selection UI
- how DM inserts a property into its design

Do not infer field names.

Do not copy stale/dead code.

Use the active working DM code path.

---

# B2. NO NEW SCRAPER

If OCR already has the V5 DM-compatible extraction contract working:

reuse it.

Do NOT:

- create a second scraper
- create a new backend route
- duplicate normalization logic
- change DM
- invent fake property data

Known previously working test URL may be:

```
https://x.ychouse.tw/8Q4Mdj
```

Use it only if it still works and is appropriate.

If no real working URL is available:

PROPERTY_TEST_INPUT_REQUIRED

Do not fake PASS.

---

# B3. INDEPENDENT PROPERTY TOOL

Property management must have an independent main Icon Rail entry:

案件物件

Hover tooltip:

案件物件

Keep generic:

新增物件

for:

- text
- normal images
- LOGO
- LINE QR
- portrait
- decoration

Remove property cards from generic Sticker-style placement if currently duplicated.

---

# B4. PROPERTY FLOW

Click:

案件物件

Open a centered modal/panel.

Required flow:

1. target side:
   - 正面
   - 反面

2. property URL

3. 擷取案件

4. extracted property preview

5. select property frame

6. choose grid layout / destination cell

7. add to A4

---

# B5. BACK PAGE DEFAULT GRID

Back page is designed primarily for property cards.

Default:

```
columns = 2
rows = 3
capacity = 6
```

Visual model:

```
┌──────────────┬──────────────┐
│   案件 1     │   案件 2     │
├──────────────┼──────────────┤
│   案件 3     │   案件 4     │
├──────────────┼──────────────┤
│   案件 5     │   案件 6     │
└──────────────┴──────────────┘
```

The default is 2 × 3.

If V5/current UI already supports user-adjustable grid size, preserve safe controls such as:

columns:
1 / 2 / 3

rows:
1 / 2 / 3 / 4

but INITIAL DEFAULT for back page must be:

2 × 3

Do not broaden grid controls if they require a large rewrite.

---

# B6. PROPERTY CARDS ARE GRID OBJECTS, NOT STICKERS

Property cards must NOT primarily use free x/y Sticker placement.

Do not allow normal property layout to become:

- arbitrary free drag anywhere
- arbitrary rotation
- floating overlap with other property cards

Each property object belongs to a grid cell.

State concept:

```js
backPropertyGrid = {
  columns: 2,
  rows: 3,
  cells: [
    propertyOrNull,
    propertyOrNull,
    propertyOrNull,
    propertyOrNull,
    propertyOrNull,
    propertyOrNull
  ]
}
```

Exact implementation may differ.

---

# B7. GRID CELL OPERATIONS

Required:

- add property into empty cell
- select property/cell
- move property to another empty cell
- swap two occupied cells
- delete
- replace property
- change frame/template

Optional:

- rowSpan
- colSpan

Do NOT block core 2×3 implementation on span support.

---

# B8. PROPERTY FRAME FIT

Each selected property frame must render INSIDE its cell.

The cell controls available:

- width
- height

Property card renderer must adapt to those dimensions.

No overflow into neighboring cells.

No arbitrary rotation.

Internal image crop/fit may follow DM's existing frame behavior.

---

# B9. FRONT PAGE PROPERTY SUPPORT

The property workflow must still support adding to front page if the user selects 正面.

However:

do NOT restructure the front page around a 2×3 grid unless current design explicitly requires it.

The explicit 2×3 default is for the BACK page.

Front page may use its existing property placement/layout behavior if already accepted.

Do not break front text layout.

---

# B10. BACK PAGE PRINT-SAFE LAYOUT

Back A4 physical target:

210mm × 297mm

Grid must be computed inside the back-page printable content area.

Account for:

- page padding
- grid gap
- footer/required fixed areas if they exist

Do not hard-code cell sizes from screenshots.

Derive them from actual back A4 content box.

Property cards must not split or overlap.

---

# B11. PROPERTY PERSISTENCE

Property grid state is part of NORMAL Letter design.

Persist:

- extracted property payload needed for rendering
- chosen frame
- page
- cell index
- grid columns/rows
- card-specific safe settings

Must survive:

- IndexedDB autosave
- F5
- .ocrletter save
- .ocrletter import

Do NOT put property grid data into temporary 開發信條 memory state.

---

# B12. PROPERTY ACCEPTANCE

Use a real supported property URL.

Verify:

- URL extraction succeeds
- actual property data preview appears
- frame can be selected
- target = back
- default grid = 2×3
- property inserts into chosen cell
- second property can insert into another cell
- moving between cells works
- swap works
- delete works
- frame change works
- no free-floating Sticker behavior
- F5 restores
- .ocrletter save/import restores grid

Also smoke:

- target = front
- property can still be added without breaking front text

---

# PART C — DO NOT TOUCH UNRELATED V5 FEATURES

Regression-smoke only:

- OCR home /
- /letter
- /ocr absent
- Icon Rail
- tooltips
- recipients
- image materials
- LOGO resize
- LINE QR resize
- normal Sticker behavior
- envelope info
- 開發信條
- save/import
- print/PDF
- fullscreen
- normal Letter autosave

Do not redesign these.

---

# FILE / CHANGE BUDGET

Target:

4–10 frontend files.

If more than 12 tracked files are required:

HARD STOP before modifying file 13.

Explain why.

Do not modify DM source.

Do not modify backend unless property extraction is proven broken due to an existing OCR integration regression.

If backend change appears necessary:

HARD STOP first.

---

# PHASE ORDER

## Phase 0 — Preflight

- prove V5 baseline
- capture Git state
- locate frontBelow renderer
- locate modal renderer
- locate actual front text width source
- classify `translateY(-19px)`
- trace DM active property workflow
- inspect current OCR property workflow

## Phase 1 — Shared front text metrics

- canonical width
- height 706
- shared renderer
- duplicate renderer check

## Phase 2 — Modal WYSIWYG

- same logical width/height
- same typography/wrap
- live preview
- apply/cancel

## Phase 3 — Property tool

- independent icon
- DM extraction workflow
- property preview
- frame selection

## Phase 4 — Back 2×3 grid

- default 2 columns × 3 rows
- cell placement
- move/swap/delete/frame change

## Phase 5 — Persistence

- IndexedDB
- F5
- .ocrletter round-trip

## Phase 6 — Regression

- tests
- build
- browser smoke

---

# VALIDATION

Run:

git diff --check

Frontend tests:

expected baseline 36/36 PASS or better

Production build:

PASS

Browser acceptance required.

Do not report PASS from source inspection only.

---

# HARD STOP CONDITIONS

Stop if:

1. trusted V5 baseline cannot be proven
2. frontBelow canonical width cannot be identified
3. the `-19px` transform cannot be classified safely
4. DM property workflow cannot be identified
5. property extraction requires a new backend/service
6. DM source must be modified
7. DB/schema change required
8. Docker/deploy/Staging/Production change required
9. more than 12 tracked files required
10. unrelated accepted V5 behavior would need broad rewrite

Return:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM_SAFE_OPTIONS

---

# LOCAL COMMIT RULE

If FINAL STATUS = PASS:

Create a LOCAL commit automatically.

Recommended message:

feat: OCR Letter front WYSIWYG and back 2x3 property grid

Do NOT push.

If FINAL STATUS = FAIL/BLOCKED:

do NOT commit over the accepted V5 checkpoint.

---

# FINAL REPORT

Return exactly:

OCR LETTER EDITOR V7 FINAL

1. SOURCE
Repo:
Branch:
Baseline checkpoint:
HEAD before:
Trusted V5 proven:
Dirty before:

2. FRONT TEXT BOX
frontBelow renderer file:
Canonical width source:
Canonical logical width:
Logical height = 706:
Observed -19px classification:
Visible renderer count:
Single source of truth:

3. TEXT MODAL WYSIWYG
Shared renderer:
Modal logical width:
Modal logical height:
Visual scale:
Wrapping parity:
Typography parity:
Live preview:
Apply:
Cancel:
Backdrop close = NO:
ESC close = NO:

4. RICH TEXT
Range formatting:
Whole-object formatting:
Handwriting fonts:
Single-row toolbar:

5. DM PROPERTY REFERENCE
Active DM flow:
Extraction contract:
Normalized fields:
Frames/templates:
DM source changed = NO:

6. PROPERTY TOOL
Independent 案件物件 icon:
Tooltip:
Property removed from generic Sticker flow:
URL extraction:
Extracted preview:
Frame selection:

7. BACK PROPERTY GRID
Default columns = 2:
Default rows = 3:
Capacity = 6:
Printable content box:
Cell dimensions:
Add cell:
Move cell:
Swap:
Delete:
Replace:
Frame change:
Free-floating Sticker = NO:

8. FRONT PROPERTY SMOKE
Add to front:
Front text unaffected:

9. PERSISTENCE
IndexedDB:
F5 restore:
.ocrletter save:
.ocrletter import:

10. REGRESSION
OCR home:
Icon Rail:
Tooltips:
Recipients:
LOGO:
QR:
Normal Sticker:
Envelope info:
Letter Strip:
Save/import:
Print/PDF:
Fullscreen:

11. VALIDATION
Console:
Network:
git diff --check:
Tests:
Build:

12. FILES CHANGED
- ...

13. LOCAL CHECKPOINT
Created:
SHA:
Message:
Pushed = NO

14. FINAL STATUS
PASS / BLOCKED / FAIL
