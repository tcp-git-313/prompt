# OCR Letter Editor V6 — FrontBelow Root Cause + Grid Property Layout + RemoveBG Wiring Repair

## OWNER

OCR LETTER EDITOR V6 OWNER

## MODEL

GPT-5.6 Luna — High

## EXECUTION MODE

You are a constrained root-cause implementation + browser-verification agent.

This task continues directly from the current OCR Letter Editor V5 working tree.

Do NOT restart V2/V3/V4/V5.
Do NOT redo already-passed editor mechanics.
Do NOT redesign the entire product.
Do NOT reset or discard the current dirty working tree.
Do NOT commit or push.

The priority is now the A4 editor itself.

Do NOT spend time on PDF optimization before the screen A4 model is correct.

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

# CURRENT WORKING BASELINE

Branch:

master

Expected HEAD:

4aa0fa90693347065411acf75b9f26524200d65a

V2/V3/V4/V5 work is expected to remain uncommitted on top of this HEAD.

Latest V5 reported:

- tracked dirty before: 6
- tracked dirty after: 11
- V5 touched LetterView / front-back-strip renderers / PDF export / styles / vite.config / property objects
- LOGO resize PASS
- QR resize PASS
- sticker transform PASS
- normal Letter F5 restore PASS
- hover tooltip PASS
- envelope duplicate Letter Strip controls removed
- property extraction from DM-compatible URL PASS
- property data extraction and frames exist
- RemoveBG still not browser-proven

Treat the current dirty tree as the authoritative source.

At task start capture:

- branch
- HEAD
- git status
- tracked dirty files
- untracked files
- diff summary

Never:

- git reset
- git restore
- checkout old versions
- clean
- delete unrelated untracked files
- use git add -A
- commit
- push

---

# USER-OBSERVED CURRENT FAILURES

## Failure 1 — RemoveBG still fails

OCR background removal is still failing in real use.

DM has a working RemoveBG flow.

This task may repair the OCR RemoveBG wiring more deeply than the prior frontend-only restriction.

Allowed OCR-side minimal wiring changes:

- frontend request
- Vite/dev proxy
- OCR backend route
- OCR backend proxy/config glue directly related to existing RemoveBG service

Still NOT allowed:

- new RemoveBG provider
- new external service
- new API key
- new ML dependency
- DM source modification
- Production deployment
- unrelated backend refactor

DM remains READ ONLY.

---

## Failure 2 — Property object is implemented like a Sticker, but that is the wrong product model

Current property extraction works, but property cards must NOT behave primarily as free-floating stickers.

The user needs controlled print layout on front/back A4.

Property objects must become a dedicated main tool with grid-based page placement, similar to the DM property workflow but adapted for OCR print layout.

Required new independent main icon:

案件物件

Hover:

案件物件

This is separate from generic:

新增物件

Generic 新增物件 remains for normal text/images/LOGO/QR/decorative objects.

案件物件 is specifically:

URL extraction
→ property data
→ frame
→ front/back property grid

---

## Failure 3 — frontBelow.text data/rendering appears overlapped or duplicated

Observed DOM #1:

<div class="letter-body text-object-selected"
     data-editable-text="true"
     data-edit-key="frontBelow.text"
     data-edit-side="front">
  ...
  敬啟者 您好：
  ...
  冒昧來信...
  ...
</div>

Observed DOM #2:

<div class="letter-body text-object-selected"
     data-editable-text="true"
     data-edit-key="frontBelow.text"
     data-edit-side="front">
  1
  2
  3
  ...
  12
</div>

Both use the SAME:

data-edit-key="frontBelow.text"
data-edit-side="front"

but display different content.

Symptoms:

- the text edited in the dialog does not reliably appear on the front page
- some content appears missing
- some content appears duplicated/overlapped
- frontBelow width/layout appears wrong
- visible text region does not use the intended full available width

Treat this as a DATA / RENDERER ROOT-CAUSE BUG first, not a CSS-only problem.

---

## Failure 4 — Text editor dialog is not WYSIWYG with the A4 selected text area

The user explicitly requires:

the text editor's editing surface
=
the selected A4 text area's visual layout

Current modal does not match.

The dialog must use the same:

- content model
- rich-text runs
- logical width
- logical height
- font family
- font size
- font weight
- italic
- underline
- color
- alignment
- line-height
- letter-spacing
- padding
- paragraph breaks
- wrapping behavior

Do NOT merely make the modal "look similar".

Use the same renderer/model.

---

## Failure 5 — Portrait needs a defined position in the right-side blank area

Current front body area visually has text on the left and unused white space on the right.

The portrait should be structurally positioned in that right-side area.

Do not leave the portrait as an unrestricted free-floating object by default.

---

# PRIMARY GOALS

A. Find and fix the frontBelow.text renderer/state duplication root cause

B. Rebuild the front-body layout into explicit bodyTextSlot + portraitSlot

C. Make the text modal truly WYSIWYG with the A4 text object

D. Replace property Sticker placement with a dedicated 案件物件 grid system on front/back

E. Repair OCR RemoveBG by tracing the full working DM chain and fixing only the minimum OCR wiring needed

F. Preserve all already-passed V4/V5 mechanics unless directly affected

---

# A1. FRONTBELOW ROOT-CAUSE INVESTIGATION FIRST

Before changing CSS, locate ALL renderers/state paths for:

frontBelow.text

Search:

- data-edit-key="frontBelow.text"
- frontBelow.text
- frontBelow
- renderFront
- shared text renderer
- modal text draft/editor state
- designDoc mapping
- restore/hydrate logic
- template defaults
- live preview overlays

In the running browser, inspect:

document.querySelectorAll(
  '[data-edit-key="frontBelow.text"][data-edit-side="front"]'
)

In the normal front A4 preview state, there must be exactly ONE active/visible source renderer for that text object.

If more than one exists:

identify:

- component/file for each renderer
- state source for each renderer
- why both are visible
- which one the modal edits
- which one A4 reads
- whether hydration/default logic is overwriting one state with another

Do NOT hide the duplicate with CSS.

Fix the source/render ownership.

---

# A2. SINGLE SOURCE OF TRUTH

Target model:

designDoc
→ frontBelow.text
→ one RichText model
→ one shared Text Renderer
   ├─ A4
   └─ Text Edit Modal

No parallel copies such as:

- A4 text state
- modal textarea state
- preview overlay state
- default/template state

unless temporary modal draft state is explicitly transactional.

If temporary modal draft exists:

open modal
→ clone snapshot/draft
→ live preview uses draft
→ Apply commits to designDoc
→ Cancel discards draft and restores snapshot

Do not maintain two long-lived truth sources.

---

# A3. DEFAULT/HYDRATION COLLISION CHECK

Investigate whether the observed "1–12" data and normal letter body data are coming from:

- defaults
- test/demo data
- recipient preview data
- stale persisted draft
- current template
- duplicated renderer
- hydration ordering

Explicitly prove the root cause before fixing.

Do not delete real saved user data.

Do not clear IndexedDB just to make the symptom disappear.

Do not reset user state.

---

# B1. FRONT BODY LAYOUT — EXPLICIT SLOTS

Refactor the front-body region conceptually into:

frontBelow
├─ bodyTextSlot
└─ portraitSlot

The exact internal names may differ, but the layout contract must be explicit.

The text and portrait must not unintentionally overlap.

---

# B2. BODY TEXT SLOT

bodyTextSlot must consume 100% of its allocated text-column width.

Do not use a stale magic width.

Current observed behavior suggests an unintended narrow region.

Rules:

if portrait slot is enabled:

bodyTextSlot width
=
frontBelow total usable width
-
portraitSlot width
-
column gap

if portrait slot is disabled/hidden:

bodyTextSlot expands to full usable width

Text must wrap based on this real slot width.

Do not force the text to the full physical A4 width when the portrait slot legitimately occupies the right column.

---

# B3. PORTRAIT SLOT

The right-side blank region becomes the default portrait area.

Default portrait placement:

- right-side portrait slot
- centered horizontally in the slot
- vertically centered or optically balanced in the slot
- preserve image aspect ratio

Portrait actions remain:

- 更換
- AI 去背
- 縮放
- 水平翻轉
- reset

Rotation may remain if already supported, but default placement must be slot-aware.

Do not let the portrait initially float unpredictably over body text.

Allow limited fine adjustment if already supported, but portrait reset must return it to the defined portrait slot.

---

# C1. TEXT EDIT MODAL MUST SHARE THE A4 RENDERER

The selected frontBelow text editor must use the SAME rendering logic as the A4 text object wherever practical.

Do not build a second typography implementation.

Create/reuse one canonical text renderer capable of:

- read mode on A4
- editable mode in modal

Both consume the same rich-text model and same logical box metrics.

---

# C2. LOGICAL BOX PARITY

When editing frontBelow.text, the modal editing canvas must use the same logical dimensions as the A4 bodyTextSlot.

For example conceptually:

A4 logical text box:
width = W
height = H

Modal editor:
logical width = W
logical height = H
then visually scale to fit available modal space if needed

Do NOT change line wrapping just because the modal window is physically wider.

Target:

a line that wraps at the same location in A4
should wrap at essentially the same location in modal

Minor browser antialiasing difference is acceptable.
Different paragraph wrapping is NOT.

---

# C3. REMOVE UNNECESSARY MODAL LABELS

Do not restore explanatory labels such as:

直接編輯目前選取的 A4 文字物件

文字內容

The editing canvas itself is the main surface.

Formatting toolbar remains below it.

---

# C4. MODAL CLOSE RULES — PRESERVE

Text modal closes ONLY by:

- 套用
- 取消

Backdrop click:
NO CLOSE

Blank A4 click:
NO CLOSE

ESC:
NO CLOSE

Apply:
commit

Cancel:
rollback exact modal-open snapshot

---

# C5. RICH TEXT — PRESERVE

Keep:

- partial selection formatting
- no selection = whole text object formatting
- live preview
- handwriting fonts
- single-row formatting toolbar

Do not regress V4/V5 rich text behavior.

---

# D1. PROPERTY OBJECT — NEW INDEPENDENT MAIN ICON

Add:

案件物件

as its own main Icon Rail entry.

Expected rail concept:

返回 OCR
收件人
新增物件
案件物件
版面與樣式
文字與內容
圖片素材
信封資訊
開發信條
儲存 / 匯入
列印 / 匯出
全螢幕

All icons must keep visible hover tooltips.

Do NOT hide 案件物件 inside generic 新增物件.

---

# D2. DM IS THE PROPERTY WORKFLOW REFERENCE

Read-only reference:

F:\00-Ticenpi-SaaS\TicenpiDM

Trace the actual DM flow:

URL
→ POST /api/scrape or current real extraction path
→ normalized property payload
→ images
→ property fields
→ frame/template
→ design insertion

Use actual DM code as source of truth.

Do not invent another extraction contract.

Do not modify DM.

---

# D3. PROPERTY WORKFLOW

案件物件 icon opens a centered property manager/modal.

Required flow:

1. choose target side:
   - 正面
   - 反面

2. paste property URL

3. 擷取案件

4. show extracted preview

5. choose a property frame/template

6. choose page-grid layout

7. add property into a grid cell

---

# D4. PROPERTY OBJECT IS NOT A STICKER

Do NOT use the normal sticker transform model as the primary property layout.

Property cards are controlled by a page grid.

Concept:

front property grid:

columns = 1 / 2 / 3
rows    = 1 / 2 / 3 / 4

Example:

2 columns × 3 rows

┌────────────┬────────────┐
│ property A │ property B │
├────────────┼────────────┤
│ property C │ property D │
├────────────┼────────────┤
│ property E │ property F │
└────────────┴────────────┘

Back page has its own independent grid settings.

---

# D5. GRID SETTINGS

User must be able to control:

橫向幾格:
1 / 2 / 3

直向幾格:
1 / 2 / 3 / 4

Store as explicit page-layout state.

Concept:

propertyGrid.front = {
  columns,
  rows,
  cells
}

propertyGrid.back = {
  columns,
  rows,
  cells
}

Do not derive layout from free x/y coordinates.

---

# D6. PROPERTY CELL BEHAVIOR

Each extracted property card occupies a grid cell.

Required operations:

- add into empty cell
- select cell/property
- move to another cell
- swap two property cards
- delete property
- replace property
- change frame/template

If safe within current architecture, support:

- colSpan
- rowSpan

for a larger feature card.

But span is optional in this task unless easy to implement safely.

Do NOT block core grid layout on span support.

---

# D7. GRID FIT / PRINT SAFETY

Property frame must fit inside its cell.

The cell controls:

- available width
- available height
- property card scale/layout

Property card must not overflow into neighboring cells.

No arbitrary free-floating sticker movement for the card itself.

If a property frame supports internal image crop/fit, preserve that behavior.

---

# D8. FRONT / BACK INDEPENDENCE

Front grid and back grid are separate.

Changing:

front columns/rows

must not alter:

back columns/rows

A property can be assigned to either page.

The user must clearly see which page is being edited.

---

# D9. PERSISTENCE

Property grid state and property records are part of normal Letter design.

They MUST persist in:

- IndexedDB normal Letter autosave
- F5 restore
- .ocrletter save
- .ocrletter import

They must NOT be part of temporary 開發信條 session state.

---

# D10. GENERIC 新增物件 REMAINS

Generic 新增物件 remains for:

- text
- normal image
- LOGO
- LINE QR
- portrait
- decorative image/object

Remove 案件物件 from generic Sticker-like add flow if currently duplicated there.

There should be one canonical property workflow:

案件物件 icon.

---

# E1. REMOVE BG — TRACE FULL DM WORKING CHAIN

Read-only DM project:

F:\00-Ticenpi-SaaS\TicenpiDM

Trace the real working RemoveBG chain:

DM UI
→ frontend request
→ dev/proxy wiring
→ backend route
→ existing RemoveBG service
→ response
→ transparent result

Record:

- frontend URL
- HTTP method
- request shape
- JSON/FormData
- field name
- auth/header
- size limit
- timeout
- proxy rewrite
- backend route
- response content type/schema
- transparent image conversion

Do not stop at frontend source.

---

# E2. TRACE OCR REMOVE BG FULL CHAIN

Do the same for OCR:

OCR UI
→ frontend request
→ Vite proxy
→ OCR backend route
→ existing RemoveBG service
→ response

Compare DM and OCR step by step.

Find the first real divergence/failure.

---

# E3. ALLOWED OCR MINIMAL WIRING FIXES

Unlike V5, this task may change minimal OCR-side RemoveBG wiring if necessary.

Allowed examples:

- wrong frontend URL
- wrong Vite proxy rewrite
- missing dev proxy path
- incorrect OCR backend forwarding route
- request shape mismatch
- response handling mismatch
- timeout mismatch
- auth header forwarding mismatch

Only changes directly required to make the existing service work.

Do not touch unrelated backend logic.

---

# E4. REMOVE BG FORBIDDEN CHANGES

Do NOT:

- create a new RemoveBG provider
- add new API key
- change DM
- add ML packages
- change DB
- change deployment architecture
- deploy to Staging/Production
- replace the shared service

If the existing service itself is down/unavailable after wiring is proven correct:

BLOCKED — REMOVE_BG_SERVICE_UNAVAILABLE

Provide evidence.

---

# E5. REAL IMAGE ACCEPTANCE

Use a real portrait image.

The user has now reported actual failure, so source-level comparison is not sufficient.

Required browser flow:

real portrait
→ upload
→ click AI 去背
→ real network request
→ successful transparent result
→ transparent portrait appears in portraitSlot
→ move/resize if allowed
→ reset returns to portraitSlot

If no suitable real portrait test image is available in local/test assets:

report:

REMOVE_BG_REAL_INPUT_REQUIRED

Do not fake pass.

---

# F. DO NOT PRIORITIZE PDF YET

Do NOT spend this task optimizing PDF generation before the A4 source layout is corrected.

You may run a smoke test after the A4 layout is correct.

Priority order:

1. frontBelow state/render correctness
2. bodyTextSlot/portraitSlot layout
3. WYSIWYG text modal
4. property grid
5. RemoveBG
6. regression
7. PDF/print smoke only

---

# G. ALREADY-PASSED MECHANICS — PRESERVE

Do not rebuild:

- LOGO direct select/resize/move
- QR direct select/resize/move
- normal sticker transforms
- sticker reset
- rich text partial formatting
- handwriting fonts
- hover tooltip system
- normal Letter IndexedDB persistence
- 開發信條 memory-only lifecycle
- OCR routing

Regression-smoke only.

---

# H. FILE / SCOPE BOUNDARY

Allowed:

OCR frontend:
- LetterView
- front/back renderers
- shared text renderer
- styles
- property-grid state/components
- persistence serializer
- .ocrletter serializer

OCR RemoveBG wiring:
- frontend request
- Vite proxy
- OCR backend route/proxy glue directly tied to existing RemoveBG service

DM:
- READ ONLY

Not allowed:

- DM source modification
- DB/schema changes
- Docker changes
- deploy changes
- Staging/Production changes
- new external service
- new secrets
- new scraping backend
- broad backend refactor
- unrelated OCR home redesign

If more than 16 tracked files are required:

HARD STOP before file 17.

---

# PHASE 0 — PREFLIGHT

Before editing:

1. capture git state
2. inspect current browser state
3. reproduce frontBelow duplicate/missing-text issue
4. count visible frontBelow renderers
5. trace frontBelow state ownership
6. trace modal state ownership
7. trace hydration/default sources
8. inspect current front body layout
9. inspect current portrait placement
10. inspect current property Sticker implementation
11. inspect DM property workflow
12. trace full DM RemoveBG chain
13. trace full OCR RemoveBG chain

Do not edit until the root-cause map is written internally.

---

# PHASE 1 — FRONTBELOW ROOT CAUSE

Fix duplicate renderer/state ownership.

Acceptance:

- one visible frontBelow renderer
- one source of truth
- no missing modal edits
- no stale duplicate/default overlay

---

# PHASE 2 — BODY TEXT + PORTRAIT SLOTS

Implement:

frontBelow
├─ bodyTextSlot
└─ portraitSlot

Acceptance:

- body text uses full allocated text-column width
- portrait sits in right-side slot
- no overlap
- portrait reset returns to slot
- no magic stale width

---

# PHASE 3 — TRUE WYSIWYG TEXT MODAL

Use shared text renderer/model.

Acceptance:

- same logical width/height
- same wrapping
- same typography
- modal edit appears immediately on A4
- Apply commits
- Cancel restores
- no backdrop/ESC close

---

# PHASE 4 — PROPERTY GRID ICON + LAYOUT

Create independent 案件物件 icon.

Move property workflow out of generic Sticker model.

Implement:

URL
→ scrape
→ preview
→ frame
→ choose front/back
→ choose columns/rows
→ add into grid cell

---

# PHASE 5 — PROPERTY GRID INTERACTION

Implement:

- select
- move cell
- swap
- delete
- replace
- frame change
- front/back independent layouts
- persistence
- .ocrletter round-trip

---

# PHASE 6 — REMOVE BG ROOT CAUSE / REPAIR

Compare DM and OCR full chains.

Apply minimum OCR wiring fix.

Use real portrait.

Do not mark PASS without visible transparency.

---

# PHASE 7 — REGRESSION

Run light regression on already-passed mechanics.

Then run tests/build.

---

# REQUIRED BROWSER ACCEPTANCE

Use:

http://localhost:3013/

Do not substitute 127.0.0.1 in acceptance notes.

## frontBelow

Run in console:

document.querySelectorAll(
  '[data-edit-key="frontBelow.text"][data-edit-side="front"]'
)

Report:

- total matched
- visible matched
- owning components

Normal preview target:

visible matched = 1

Edit text in modal:

- type obvious unique text
- verify it appears immediately on front A4
- Apply
- verify remains
- reopen
- Cancel a change
- verify rollback

F5:
- committed text remains
- no duplicate renderer appears

## WYSIWYG

Use multi-line test content.

Compare modal vs A4:

- line breaks
- wrapping
- font size
- line-height
- width
- paragraph spacing

They must closely match.

## Portrait slot

Verify portrait default position is in the right-side blank slot.

Verify:

- no text overlap
- resize stays usable
- reset returns to slot

## Property grid

Use known working DM-compatible property URL if still available.

Verify:

- 案件物件 icon exists
- URL scrape succeeds
- property preview appears
- frame can be selected
- front grid columns/rows configurable
- add property to a chosen front cell
- add/test back page grid
- cards fit inside cells
- move/swap cells
- no Sticker-style arbitrary floating placement
- F5 restore
- .ocrletter save/import restore

## RemoveBG

With real portrait:

- request succeeds
- transparent result visible
- portrait lands in portraitSlot
- reset works

## Regression smoke

- LOGO resize
- QR resize
- normal sticker rotate
- rich text formatting
- hover tooltips
- OCR home
- fullscreen
- normal Letter F5 restore

---

# VALIDATION

Run:

git diff --check

Frontend tests:

current baseline 36/36 PASS or better.

Production frontend build:

PASS.

If OCR backend/proxy wiring was changed for RemoveBG:

run the narrowest existing backend/unit/smoke validation relevant to that route.

Do not run unrelated deployment tasks.

---

# HARD STOP CONDITIONS

Stop before broadening scope if:

1. DM source modification becomes necessary
2. a new RemoveBG provider/service is required
3. a new scraper backend is required
4. DB/schema change required
5. Docker/deploy/Staging/Production change required
6. new secret/API key required
7. more than 16 tracked files required
8. current dirty working tree cannot be safely preserved
9. fixing frontBelow requires rewriting the entire editor outside Letter scope

Output:

CURRENT
EXPECTED
ROOT_CAUSE
CONFLICT
EVIDENCE
MINIMUM_SAFE_OPTIONS

---

# FINAL PASS RULE

PASS requires:

- frontBelow duplicate/state collision root cause fixed
- one visible frontBelow renderer
- modal edits render on A4 correctly
- modal/A4 wrapping and typography closely match
- bodyTextSlot uses full allocated width
- portraitSlot exists and portrait is correctly positioned
- independent 案件物件 icon exists
- property cards use grid placement, not Sticker placement
- front grid rows/columns configurable
- back grid rows/columns configurable
- property cards fit cells
- property grid persists through F5
- property grid survives .ocrletter round-trip
- RemoveBG works with real portrait OR a clearly evidenced external service blocker is reported
- already-passed mechanics do not regress
- git diff --check PASS
- tests PASS
- build PASS

Do not report PASS based only on code inspection.

---

# FINAL REPORT

Return exactly:

OCR LETTER EDITOR V6 FINAL

1. SOURCE
Path:
Branch:
HEAD:
Tracked dirty before:
Tracked dirty after:
Files changed by V6:

2. FRONTBELOW ROOT CAUSE
Matched renderers:
Visible renderers before:
Visible renderers after:
State sources found:
Root cause:
Single source of truth:
Duplicate/default overlay removed:

3. FRONT BODY LAYOUT
bodyTextSlot:
portraitSlot:
Text allocated width:
Portrait default position:
Text/portrait overlap:
Portrait reset:

4. TEXT WYSIWYG
Shared renderer:
Logical width parity:
Logical height parity:
Wrapping parity:
Typography parity:
Live preview:
Apply:
Cancel rollback:
Backdrop close = NO:
ESC close = NO:

5. PROPERTY ICON
Independent icon:
Hover tooltip:
Removed property from generic Sticker flow:

6. PROPERTY GRID
DM reference path:
URL extraction:
Frame selection:
Front columns:
Front rows:
Back columns:
Back rows:
Add front cell:
Add back cell:
Move cell:
Swap:
Delete:
Replace:
Frame change:
Sticker free-float = NO:

7. PROPERTY PERSISTENCE
IndexedDB:
F5 restore:
.ocrletter save:
.ocrletter import:

8. REMOVE BG
DM full path:
OCR full path:
First divergence:
OCR wiring changed:
Real portrait:
Network result:
Transparent visual result:
Portrait slot result:
Status:

9. REGRESSION
LOGO:
QR:
Sticker transforms:
Rich text:
Hover tooltip:
OCR home:
Fullscreen:
Normal Letter F5:

10. VALIDATION
Console:
Network:
git diff --check:
Frontend tests:
Backend/route smoke if applicable:
Build:

11. OUT OF SCOPE
DM source changed = NO
DB changed = NO
Docker changed = NO
Deploy changed = NO
Staging changed = NO
Production changed = NO
New service = NO
New secret = NO
Commit/push performed = NO

12. FINAL STATUS
PASS / BLOCKED / FAIL
