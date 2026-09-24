# OCR Letter Editor V4 — Rich Text + Object Add + Session Letter Strip

## OWNER

OCR LETTER EDITOR V4 OWNER

## MODEL

GPT-5.6 Luna — High

## EXECUTION MODE

You are a constrained frontend executor.

This task continues from the current OCR Letter Editor V2/V3 working tree.

Do not redesign from scratch.
Do not reset the working tree.
Do not discard existing V2/V3 changes.
Do not touch backend/DB/Docker/deploy/Staging/Production unless a hard stop is triggered.

Your job is to fix the current editor precisely according to this specification.

---

# CANONICAL SOURCE

OCR project:

F:\00-Ticenpi-SaaS\TicenpiLetter

DM reference project:

F:\00-Ticenpi-SaaS\TicenpiDM

Canonical local host:

http://localhost:3013/

Letter editor:

http://localhost:3013/letter

Expected base HEAD may still be:

4aa0fa90693347065411acf75b9f26524200d65a

with uncommitted V2/V3 changes on top.

At start record:

- branch
- HEAD
- git status
- tracked dirty files
- untracked files

Never:

- reset
- restore unrelated files
- clean
- delete unrelated untracked files
- use git add -A
- commit
- push

---

# 0. PRODUCT NAMING CORRECTION — ORC -> OCR

The product name was previously typed incorrectly.

The correct product name is:

OCR

All user-visible ORC naming in this Letter Editor task must become OCR.

Search for:

- ORC
- Orc
- orc

Then classify each occurrence before changing it.

## Change safely

Update user-visible/product-facing naming such as:

- UI text
- labels
- tooltips
- editor titles
- comments/docs specific to this frontend feature
- local design-file extension
- frontend-only editor storage keys where safe

Use:

OCR
OCR Letter Editor
.ocrletter

## Do NOT blindly rename contracts

Do not break:

- existing backend API routes
- deployment service names
- environment variables
- database/schema names
- external service identifiers

If an internal/backend contract still contains "orc" and changing it would cross frontend scope, preserve the contract and only correct the user-facing label.

Report any preserved compatibility naming in the final report.

---

# 1. ROUTING — PRESERVE FINAL CONTRACT

Keep:

/        = OCR main screen
/letter  = Letter editor

Remove:

/ocr

Requirements:

- http://localhost:3013/ renders OCR directly
- URL remains /
- /ocr is not a product route
- Letter back action returns to /
- do not reintroduce / -> /ocr redirect
- do not create /ocr alias

---

# 2. TEXT EDIT MODAL — CANNOT CLOSE BY CLICKING OUTSIDE

Current requirement is strict.

Once a text-edit modal is open, it may close ONLY by:

- 套用
- 取消

Do NOT close it when:

- user clicks modal backdrop
- user clicks blank area
- user clicks A4 canvas
- user clicks another object
- user presses ESC

Backdrop click must be ignored while the text modal is active.

ESC must NOT close the text-edit modal.

## Apply behavior

套用:

- accept current live-preview state
- close modal

## Cancel behavior

取消:

- restore exact snapshot from before modal opened
- restore text content
- restore all text formatting/ranges
- close modal

No unsaved live change may survive Cancel.

---

# 3. RICH TEXT — SELECTION-BASED FORMATTING

The text editor must support partial-format selection.

Behavior:

## If text is highlighted/selected

Formatting changes apply ONLY to the selected text range.

Examples:

- font family
- font size
- bold
- italic
- underline
- color
- line-height where technically valid
- letter spacing
- alignment where technically valid for the block/paragraph

## If no text is highlighted

Formatting change applies to the entire current text object.

Do not implement this with deprecated browser execCommand shortcuts.

Use a stable rich-text representation.

Preferred conceptual model:

text object
→ content/runs or spans/ranges

Example:

[
  { text: "您好，我是", style: {...} },
  { text: "王大明", style: { fontFamily: "handwriting", fontSize: ... } },
  { text: "，很高興...", style: {...} }
]

The exact implementation may differ, but it must reliably preserve mixed formatting.

---

# 4. TEXT MODAL MUST MATCH A4 TEXT RENDERING

The text editing area must visually match the selected A4 text object as closely as practical.

The user wants the editor to feel WYSIWYG.

The modal editing surface should reuse the SAME rendering/formatting logic as the A4 text object wherever possible.

For the selected text object, match:

- width
- text wrapping
- font family
- font size
- font weight
- italic
- underline
- color
- alignment
- line height
- letter spacing
- padding
- white-space behavior
- paragraph breaks

Goal:

what the user sees in the modal text editing area should correspond closely to what will appear in the A4 preview.

Avoid maintaining separate incompatible text-style implementations.

## Remove unnecessary UI labels

Remove unnecessary explanatory elements such as:

- "直接編輯目前選取的 A4 文字物件"
- "文字內容"

The editor itself should be visually self-explanatory.

---

# 5. TEXT LIVE PREVIEW — PRESERVE

Text changes must update immediately in:

1. the modal editing surface
2. the selected A4 text object

No Apply click should be required just to preview changes.

Apply confirms.
Cancel rolls back.

The formatting toolbar remains below the text editing surface.

---

# 6. FORMATTING TOOLBAR — ONE SINGLE ROW

Keep the toolbar on ONE horizontal row.

Required controls:

字型 | 字級 | B | I | U | 文字顏色 | 左對齊 | 置中 | 右對齊 | 行距 | 字距

Requirements:

- no wrapping on normal desktop width
- compact controls
- compact dropdowns
- compact icon buttons
- horizontal overflow only as last-resort fallback
- normal desktop should not show a horizontal toolbar scrollbar

---

# 7. DM FONT REFERENCE — HANDWRITING FONTS ONLY

Inspect:

F:\00-Ticenpi-SaaS\TicenpiDM

For fonts, reuse ONLY the handwriting fonts already used by DM.

Do NOT copy:

- DM's whole font system
- DM typography scale
- DM text-editor architecture
- DM design system

Find:

- handwriting font assets
- font-face declarations
- loading method
- required frontend wiring

Then add those handwriting fonts as functioning options in OCR Letter.

Keep OCR's current normal fonts.

Verify in the real browser that choosing the handwriting font visibly changes the text.

---

# 8. OBJECT MODEL — RESTORE / ADD "新增物件"

The current template/object selection exists, but the ability to add objects is missing.

Restore/add a clear:

新增物件

entry.

Use the DM project only as an interaction/reference source for how object addition/selection is handled.

Do NOT copy DM wholesale.

## Add-object UI

Use a dedicated icon in the left rail:

新增物件

Hover tooltip:

新增物件

Click opens a centered modal.

The modal must allow selecting:

新增到：
● 正面
○ 反面

Object types:

- 文字
- 圖片
- LOGO
- LINE QR
- 大頭照
- 裝飾圖片
- basic decorative object only if already supported by current template system

When an object is added:

- place it at a safe visible position on the selected page
- immediately select it
- make it editable using the shared selectedObject model

---

# 9. SHARED selectedObject MODEL — FRONT + BACK

Front and back must share one object-selection system.

Concept:

selectedObject
activePage = front | back

Each editable object should have stable state such as:

- id
- type
- page
- x
- y
- width
- height
- rotation
- zIndex
- visible
- locked where supported
- type-specific content/style

Selectable categories may include:

- text
- logo
- LINE QR
- image
- portrait
- sticker
- decorative object

Behavior:

click object
→ select object

click blank canvas
→ deselect

EXCEPTION:

if text-edit modal is open, blank canvas click must NOT close the modal.

---

# 10. IMAGE MATERIALS — FINAL CONTENT

There is ONE image-material icon.

Do NOT use a separate portrait icon.

圖片素材 contains:

- LOGO
- LINE QR
- 其他圖片
- 大頭照

## LOGO

Must support direct A4 selection and resize.

Required:

- drag/move
- resize
- bring forward
- send backward
- reset transform

## LINE QR

Must support direct A4 selection and resize.

Required:

- drag/move
- resize
- bring forward
- send backward
- reset transform

## 其他圖片

Support:

- place image
- direct selection
- resize
- optional 製作貼紙 action

When converted to sticker:

- move
- resize
- rotate
- horizontal flip
- bring forward
- send backward
- delete
- reset position/transform

## 大頭照

Managed inside 圖片素材.

Actions when selected:

- 更換
- AI 去背
- 縮放
- 旋轉
- 水平翻轉
- 置前
- 置後
- 刪除
- 重設位置 / 變形

---

# 11. STICKER TRANSFORM BUG — MUST FIX

Current reported bug:

rotating/clicking the image multiple times can make it disappear.

Treat as critical.

Do not build transforms by repeatedly appending CSS transform strings.

Maintain explicit transform state:

- x
- y
- scale or width/height
- rotation
- flipX
- zIndex

Render transform from that state.

Test:

- rotate repeatedly
- resize repeatedly
- rotate then drag
- drag then rotate
- resize then rotate
- rotate then resize
- flip then rotate
- z-order then move
- double click

The image/sticker must remain visible/recoverable.

Add:

重設位置 / 變形

This restores a safe visible state.

---

# 12. AI REMOVE BG — USE DM WORKING FLOW AS REFERENCE

Inspect:

F:\00-Ticenpi-SaaS\TicenpiDM

For RemoveBG ONLY, compare DM working implementation with OCR.

Check:

- endpoint
- method
- FormData structure
- field name
- MIME
- headers/auth
- timeout
- response schema
- blob/base64 handling
- transparent PNG handling
- error handling

Reuse OCR's existing backend capability where valid.

Do NOT:

- create new backend endpoint
- add external paid service
- add new secret
- add ML dependency
- modify backend architecture

Acceptance requires a real portrait:

source image
→ AI remove background
→ transparent result visibly appears
→ result usable as A4 sticker

200 response alone is not PASS.

---

# 13. ENVELOPE INFO ICON — PRESERVE

Keep the independent icon:

信封資訊

Hover:

信封資訊

Envelope/postal settings stay here.

Include:

- 寄件人姓名
- 寄件地址
- 郵遞區號
- existing envelope-related settings
- 印刷品 ON/OFF
- 收件人顯示 姓氏 / 貴住戶

## 印刷品

ON
→ show 印刷品 mark

OFF
→ hide

## 收件人顯示

Mutually exclusive:

● 顯示姓氏
○ 顯示「貴住戶」

Exactly one active.

---

# 14. 開發信條 — INDEPENDENT MAIN ICON

This must be its own main icon.

Do NOT hide it under Print/Export.

Add independent left-rail icon:

開發信條

Hover tooltip:

開發信條

Click opens a LARGE centered modal immediately.

---

# 15. 開發信條 MODAL — SESSION-ONLY BULK TOOL

This is designed for approximately 200+ letter-strip entries.

It is a temporary batch-working tool.

Recommended modal size:

- width: approximately 88–92vw
- height: approximately 88–92vh

## Top area

Keep it simple:

- 開發信條
- count
- 列印
- 匯出 PDF

Do NOT overload the header.

## Main area

Show the 200+ letter-strip items efficiently.

Use:

- virtualization
or
- pagination
or
- efficient windowed rendering

Prefer existing project dependencies.
Do not add a new virtualization library just for this if avoidable.

Do NOT generate 200 images/PDF pages merely by opening the modal.

Render only what is needed for preview.

## Bottom area — FONT TOOLBAR

The font/format tool library must be fixed at the BOTTOM of the modal.

It must not scroll away with the 200 items.

Provide only the tools needed for the letter-strip batch.

At minimum:

- font family
- font size
- bold
- italic/underline if currently useful
- text color if useful
- alignment if useful
- line height
- letter spacing

Keep the toolbar compact.

The batch formatting should apply to the whole letter-strip set by default.

Do not create 200 independently styled letter-strip documents unless current requirements explicitly demand it.

---

# 16. 開發信條 — 姓氏 / 貴住戶

Inside the 開發信條 modal include:

收件人顯示

● 顯示姓氏
○ 顯示「貴住戶」

This setting is SESSION-ONLY for 開發信條.

It is independent from the persistent envelope-info setting.

Meaning:

- Envelope Info can have its own persistent/current design setting
- 開發信條 can choose its own temporary surname/貴住戶 mode

Do not couple these two states.

---

# 17. 開發信條 — PRINT / PDF

The modal must support:

- 列印
- 匯出 PDF

Do NOT support JPG.

Do not pre-generate PDF on modal open.

Only generate print/PDF output when the user explicitly clicks:

- 列印
or
- 匯出 PDF

After PDF generation/download, release temporary Blob/Object URLs where relevant.

Do not write generated PDF to VPS storage.

---

# 18. 開發信條 DATA LIFECYCLE — MEMORY ONLY

This is a HARD REQUIREMENT.

開發信條 is temporary session data only.

It must NOT persist to:

- localStorage
- sessionStorage
- IndexedDB
- Supabase
- database
- VPS filesystem
- .ocrletter project file

Use only frontend in-memory state/store.

Desired lifecycle:

open /letter
→ create current in-memory session

load/generate 200 letter-strip entries
→ stored in memory only

close 開發信條 modal
→ reopen modal
→ data still exists

switch to another editor tool
→ return
→ data still exists

NO PAGE REFRESH
→ data remains

F5 / Ctrl+R
→ data clears naturally

close tab
→ data clears

close browser
→ data clears

No clear API call is needed.
Do not send a delete request to VPS.
Let browser memory disappear naturally.

This is deliberate product behavior.

---

# 19. 開發信條 MUST NOT CONSUME VPS STORAGE

Do not persist the 200+ temporary entries server-side.

Do not upload temporary preview images/PDFs to the VPS.

Keep raw temporary working data as lightweight JS objects.

Avoid storing:

- Base64 preview images for every row
- 200 generated canvases permanently
- 200 PDF pages in memory before requested

Use lazy/on-demand rendering.

---

# 20. SAVE / IMPORT — MERGE INTO ONE MAIN ACTION

Replace separate:

- 儲存設計
- 開啟設計檔

with one main icon/action:

儲存 / 匯入

Hover:

儲存 / 匯入

Click opens a centered modal with two clear options:

- 儲存設計
- 匯入設計

Use .ocrletter.

## Save

Save a real local .ocrletter project file.

Prefer File System Access API.

Fallback to browser download.

## Import

Import .ocrletter and restore editor state.

Before import:

autosave current normal Letter Editor working draft first.

Do not include 開發信條 session-only data in .ocrletter.

---

# 21. NORMAL LETTER EDITOR AUTOSAVE — PRESERVE

This is separate from 開發信條.

The normal Letter Editor design SHOULD remain persistent across refresh using IndexedDB.

Persist:

- normal letter text
- text styles/rich text
- object positions
- object transforms
- normal images
- template/theme
- recipients as appropriate to current design
- envelope-info settings
- print-mark setting
- other normal editor state

User isolation required.

F5 must restore normal Letter Editor state.

IMPORTANT:

開發信條 data must still clear on F5.

So keep these two layers separate:

A. Normal Letter Editor
→ IndexedDB persistent working draft

B. 開發信條
→ memory-only session

Do not mix them.

---

# 22. DEFAULTS MUST NOT OVERWRITE REAL STATE

Priority:

1. restored IndexedDB normal Letter draft
2. existing real loaded user data
3. defaults only if empty

Never reset real editor state on mount.

Defaults should not affect 開發信條 session behavior.

---

# 23. REMOVE JPG EXPORT

JPG export is no longer required.

Remove user-facing JPG export from the Letter Editor.

Keep:

- PDF
- print

Do not remove internal image-generation logic if it is still required by another working feature, but do not expose JPG as an export choice.

---

# 24. LEFT ICON RAIL — FINAL ORDER

Use compact icon rail with hover tooltip.

Recommended order:

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

Every icon:

- consistent SVG/icon style
- Traditional Chinese hover tooltip
- no permanent text label
- no separate portrait icon

---

# 25. NORMAL LETTER PRINT/EXPORT

Normal Letter Editor keeps:

- PDF
- direct print

Remove:

- JPG

Do not merge 開發信條 into normal Print/Export.
開發信條 remains its own main icon.

---

# 26. UNDO / REDO — RECOMMENDED BUT SCOPED

If the current editor architecture already has a simple safe history mechanism or can support it without broad refactor, add:

- Ctrl+Z
- Ctrl+Y / Ctrl+Shift+Z

for normal Letter Editor object/text operations.

Do NOT make this a blocker if it requires broad architecture work.

If not implemented, report:

UNDO_REDO = DEFERRED

Do not let this expand scope.

---

# 27. AUTOSAVE STATUS — SMALL NON-BLOCKING FEEDBACK

For normal Letter Editor only, provide subtle feedback such as:

儲存中…
已自動儲存

This must NOT recreate a top bar.

Use a small non-blocking toast/status near a corner and fade it away.

Do not show this for 開發信條 memory-only session.

---

# 28. VIEWPORT / SCROLLBAR — PRESERVE V3 REQUIREMENT

Normal Letter Editor page:

- root height: 100dvh
- body/page no vertical scrollbar
- icon rail no visible scrollbar
- A4 canvas maximized but does not exceed viewport

Internal scrolling allowed only in:

- modals
- temporary panels
- long lists

Validate duplicate left scrollbars are gone.

---

# 29. FULLSCREEN — PRESERVE

Fullscreen:

- hides icon rail
- hides temporary panels
- maximizes A4
- ESC exits fullscreen

EXCEPTION:

if text-edit modal is active, ESC must NOT close the text modal.

---

# STRICT SCOPE

Allowed:

- frontend router
- Letter editor components
- letter-specific CSS
- rich-text state/rendering
- selectedObject/object helpers
- IndexedDB helper for normal editor
- local .ocrletter save/import
- in-memory store for 開發信條
- frontend RemoveBG integration fixes
- read/reference DM source

Not allowed:

- backend source changes
- DB/schema changes
- Docker changes
- CI changes
- deploy changes
- Staging changes
- Production changes
- auth/entitlement redesign
- new external service
- new secret
- new heavy dependency

If backend contract must change:

HARD STOP.

---

# ANTI-OVERREFACTOR

Target:

6–12 frontend files.

If more than 15 tracked files appear necessary:

HARD STOP before the 16th file.

Do not:

- rewrite the entire frontend
- rename unrelated modules
- reformat unrelated files
- copy DM wholesale
- introduce new framework
- replace global state management
- redesign OCR home

---

# PHASE 0 — PREFLIGHT

Before editing:

1. capture git state
2. inspect current V2/V3 working tree
3. inspect current browser behavior
4. reproduce current bugs
5. inspect DM handwriting font source
6. inspect DM object-add interaction
7. inspect DM RemoveBG integration
8. inspect current Letter object model
9. inspect normal editor persistence
10. inspect current print/PDF flow

Do not edit until these are understood.

---

# PHASE 1 — NAMING + ROUTING

Correct user-facing ORC -> OCR safely.

Preserve backend/internal compatibility names where required.

Confirm:

/ = OCR
/letter = editor
/ocr absent

---

# PHASE 2 — TEXT MODAL STABILITY

Implement:

- no backdrop close
- no ESC close
- Apply only
- Cancel rollback

Remove unnecessary labels.

---

# PHASE 3 — RICH TEXT + WYSIWYG

Implement:

- selection-based formatting
- no-selection = whole object formatting
- live preview
- same A4/text-editor renderer where practical
- DM handwriting fonts only

---

# PHASE 4 — OBJECT ADD + SHARED SELECTION

Restore/add:

新增物件

Use shared selectedObject for front/back.

---

# PHASE 5 — IMAGES / TRANSFORM

Consolidate image materials.

Ensure:

- LOGO resize
- QR resize
- other image -> sticker
- portrait inside image materials
- transform bug fixed
- reset transform

---

# PHASE 6 — REMOVE BG

Compare DM working frontend flow.

Fix OCR frontend integration only.

Validate real transparent result.

---

# PHASE 7 — 開發信條 MAIN ICON

Add independent main icon and large centered modal.

Implement 200+ efficient preview.

Bottom fixed font toolbar.

Add session-only surname/貴住戶 option.

Add print/PDF.

No JPG.

---

# PHASE 8 — 開發信條 MEMORY-ONLY STORE

Keep modal/session data in frontend memory only.

Validate:

close/reopen modal
→ data remains

switch tools
→ remains

F5
→ clears

Do not touch VPS/DB/IndexedDB/localStorage/sessionStorage/.ocrletter.

---

# PHASE 9 — SAVE / IMPORT

Merge into one main action:

儲存 / 匯入

Implement:

- save .ocrletter
- import .ocrletter

Do not include 開發信條 session data.

---

# PHASE 10 — NORMAL EDITOR AUTOSAVE

Preserve/repair IndexedDB autosave for normal Letter Editor.

Validate F5 restores normal editor.

Validate 開發信條 still clears.

---

# PHASE 11 — PRINT / EXPORT CLEANUP

Normal editor:

- PDF
- print
- no JPG

開發信條:

- own print
- own PDF

Do not mix them.

---

# PHASE 12 — REGRESSION

Validate all normal Letter Editor behavior plus 開發信條 behavior.

---

# HARD STOP CONDITIONS

Stop if any occurs:

1. backend change required
2. DB change required
3. Docker/deploy change required
4. new external service required
5. new secret required
6. heavy/new framework dependency required
7. more than 15 tracked files needed
8. current V2/V3 dirty state cannot be safely preserved
9. rich text requires rewriting the entire Letter renderer
10. RemoveBG requires backend contract change

Output:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS

---

# REQUIRED BROWSER VALIDATION

Use:

http://localhost:3013/

Do not switch canonical testing to 127.0.0.1.

## Naming

- visible ORC naming removed/replaced by OCR
- backend compatibility names preserved if required

## Text modal

- backdrop click does not close
- blank A4 click does not close
- ESC does not close
- Cancel restores snapshot
- Apply keeps state

## Rich text

- select substring
- change font
- only selected substring changes
- no selection
- whole text object changes
- modal editor matches A4 wrapping/formatting closely
- handwriting font visibly renders
- toolbar one row

## Add object

- add to front
- add to back
- immediate selection
- direct manipulation

## Images

- LOGO resize
- QR resize
- other image -> sticker
- portrait inside image materials
- repeated rotate/resize does not disappear
- reset transform works

## RemoveBG

- real portrait
- transparent result visible

## 開發信條

- independent icon exists
- large centered modal
- handles 200+ entries without rendering everything at once
- bottom font toolbar stays fixed
- surname mode works
- 貴住戶 mode works
- print works
- PDF works
- no JPG

## 開發信條 lifecycle

Create visible temporary edits/data.

Then:

close modal
→ reopen
→ data remains

switch editor tools
→ return
→ data remains

F5
→ temporary 開發信條 data clears

Verify it was not written to:

- localStorage
- sessionStorage
- IndexedDB
- .ocrletter
- backend/VPS

## Normal Letter persistence

Make normal Letter edits.

F5.

Normal Letter state must restore.

This verifies the two persistence layers are correctly separated.

## Save / Import

- 儲存 / 匯入 main icon
- centered modal
- save .ocrletter
- import .ocrletter
- 開發信條 session data excluded

## Export

Normal editor:
- PDF
- print
- JPG absent

開發信條:
- PDF
- print

## Regression

- OCR home
- recipients
- templates/themes
- fullscreen
- front/back preview
- no page scrollbar regression
- no new console errors

Run:

git diff --check

Run existing frontend tests/build.

---

# FINAL STATUS RULES

PASS only if all critical items work.

FAIL if:

- text modal still closes from backdrop/ESC
- partial rich-text selection formatting does not work
- normal editor F5 loses data
- 開發信條 survives F5
- 開發信條 writes to persistent storage
- image transform can still disappear
- LOGO/QR cannot resize
- font still has no visible effect

BLOCKED if:

- RemoveBG requires backend contract changes
- current architecture makes rich text impossible without broad renderer rewrite

Do not fake PASS.

---

# FINAL REPORT

Return exactly:

OCR LETTER EDITOR V4 FINAL

1. SOURCE
Path:
Branch:
HEAD:
Pre-existing tracked dirty:
Pre-existing untracked:

2. NAMING
Visible ORC -> OCR:
Preserved compatibility names:

3. ROUTING
/:
/ocr:
/letter:
Back action:

4. TEXT MODAL
Backdrop close:
ESC close:
Apply:
Cancel rollback:
Removed unnecessary labels:

5. RICH TEXT
Selection formatting:
Whole-object formatting:
Live preview:
A4/modal renderer parity:
Handwriting fonts:
Single-row toolbar:

6. OBJECTS
Add object:
Front add:
Back add:
selectedObject:
Direct selection:

7. IMAGES
Image materials:
Portrait separate icon:
LOGO resize:
QR resize:
Other image -> sticker:
Rotation stress:
Reset transform:
RemoveBG:

8. ENVELOPE INFO
印刷品:
姓氏:
貴住戶:

9. 開發信條
Independent icon:
Modal size:
200+ rendering:
Bottom font toolbar:
Surname/貴住戶:
Print:
PDF:
JPG = REMOVED:

10. 開發信條 DATA LIFECYCLE
Memory-only:
Close/reopen preserved:
Tool switch preserved:
F5 clears:
localStorage write = NO
sessionStorage write = NO
IndexedDB write = NO
Backend/VPS write = NO
Included in .ocrletter = NO

11. NORMAL LETTER PERSISTENCE
IndexedDB:
User isolation:
F5 restore:
Defaults overwrite real state = NO

12. SAVE / IMPORT
Combined action:
Save .ocrletter:
Import .ocrletter:
開發信條 excluded:

13. EXPORT
Normal PDF:
Normal print:
Normal JPG = REMOVED:

14. VIEWPORT
Body scrollbar:
Icon rail scrollbar:
Canvas fit:
Fullscreen:

15. FILES CHANGED
- ...

16. VALIDATION
Console:
git diff --check:
Tests:
Build:

17. OUT OF SCOPE
Backend changed = NO
DB changed = NO
Docker changed = NO
Staging changed = NO
Production changed = NO
Commit/push performed = NO

18. FINAL STATUS
PASS / BLOCKED / FAIL
