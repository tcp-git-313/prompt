# ORC Letter UI Redesign — /ocr Default Landing + /letter Layout Overhaul

## OWNER

ORC LETTER UI OWNER

## MODEL

GPT-5.6 Sol — High

## CANONICAL SOURCE

F:\00-Ticenpi-SaaS\TicenpiLetter

## LOCAL DEV

Frontend:
http://127.0.0.1:3013

Primary OCR page:
http://127.0.0.1:3013/ocr

Letter page:
http://127.0.0.1:3013/letter

This task is LOCAL FRONTEND UI/UX ONLY.

Do not modify Staging, Production, Supabase schema, Docker, CI, Cloudflare, DNS, backend OCR logic, auth contract, or DB contract.

---

# USER-DECIDED REQUIREMENTS — FROZEN

## 1. /ocr becomes the default landing page

Opening:

http://127.0.0.1:3013

must immediately land on:

http://127.0.0.1:3013/ocr

Use the proper router redirect / app routing mechanism. Do not duplicate the OCR page.

---

## 2. /letter needs a full layout redesign

Current page:

http://127.0.0.1:3013/letter

The current layout is visually poor and wastes space.

The goal is a clean desktop-first letter editor where the A4 preview is the main focus.

---

# REMOVE OLD LETTER PAGE SHELL

On /letter remove the old sidebar-shell presentation, including the visual structure represented by the current:

- .sidebar
- .sidebar-brand
- .sidebar-app-bar
- launcher sidebar button
- offline trial footer
- API status footer
- version footer
- old left application shell

Do not keep the old app sidebar and then add another editor sidebar beside it.

The /letter page should have ONE clear editor layout.

Also remove the old bulky letter toolbar presentation represented by:

- .letter-toolbar
- old "📮 開發信" toolbar block
- old "回 AI辨識" toolbar action if replaced by the new top header

Do not remove product functionality; replace the presentation with the new structure below.

---

# NEW PAGE STRUCTURE

Use 3 zones.

## A. Top header

Create a slim clean header across the top.

Left side:

- clear back button: "返回 OCR"
- ORC product mark / text if appropriate
- page title: "開發信"
- small subtitle, e.g. "從 OCR 辨識結果，快速生成專業開發信"

Back behavior:

"返回 OCR" -> /ocr

Since / is also redirected to /ocr, direct /ocr navigation is preferred.

Do not keep a duplicate "前往 AI OCR" action if the back button already serves the same purpose.

---

## B. Left configuration rail

Move the existing letter editor controls to the FAR LEFT.

This rail replaces the current awkward/floating letter panel.

It should be:

- fixed/stable width
- aligned from the top under the header
- scrollable vertically
- compact but readable
- desktop-first
- visually clean
- card/accordion sections are acceptable
- no floating translate hacks
- no large unused gaps

Preserve these editing functions:

1. 收件人
   - 從 AI 辨識歷史勾選
   - 匯入 Excel
   - 手動新增
   - recipient list / delete

2. 版型與主題

3. 開發信範本

4. 正面內文

5. 信封樣式

6. 業務資訊

7. 圖片
   - LOGO
   - 大頭照
   - LINE QR

8. 字體

9. 匯出
   - PDF
   - JPG

You may improve grouping, collapse behavior, spacing and labels, but do not silently remove these functions.

---

## C. Main preview workspace

The A4 preview must become the dominant visual area.

Requirements:

- A4 preview as large as reasonably possible
- maximize available viewport height
- minimize dead space
- do not let the left rail crowd the preview
- keep preview centered / visually balanced
- allow the document to remain readable at a glance
- use clean neutral workspace background
- envelope preview may remain as a secondary preview if useful
- envelope must not steal focus from the A4 letter

The user specifically wants:

A4 預覽放到最大

Treat this as the highest visual priority.

---

# REMOVE "適應視窗"

Remove the "適應視窗" / fit-to-window control from /letter.

If zoom controls exist, keep only simple useful controls.

Preferred preview toolbar:

- page indicator
- simple zoom out
- simple zoom in

No fit-to-window button.

---

# DESIGN DIRECTION

Create a polished modern B2B SaaS editor.

Desired character:

- clean
- professional
- modern
- desktop-first
- preview-centric
- compact controls
- strong information hierarchy
- less dashboard-like
- less clutter
- more like an editor/design workspace

The visual reference direction approved by the user is:

- slim top header
- far-left configuration rail
- large central A4 preview
- optional smaller envelope preview on the right
- light neutral workspace
- restrained blue accents
- no old app sidebar shell

Do not copy decorative content literally if it conflicts with the current product data; use the reference only for layout hierarchy and visual direction.

---

# ROUTING REQUIREMENT

Inspect the actual Vue/router implementation first.

Implement:

/ -> /ocr

Do not introduce redirect loops.

Confirm:

http://127.0.0.1:3013
-> http://127.0.0.1:3013/ocr

Also verify:

/ocr remains functional
/letter remains functional

---

# IMPLEMENTATION RULES

## Allowed

- frontend router change
- /letter component/layout changes
- scoped CSS changes
- small component extraction/refactor if it reduces complexity
- responsive desktop sizing
- remove obsolete /letter-only UI controls
- adjust preview scaling/layout

## Not allowed

- backend changes
- OCR API changes
- DB changes
- Supabase schema changes
- Docker changes
- Staging changes
- Production changes
- deployment
- Cloudflare/DNS
- entitlement/auth redesign
- unrelated UI redesign of /ocr

Do not use git add -A.
Do not reset / restore unrelated user work.
Do not remove unrelated untracked files.

---

# PHASE 0 — INSPECT CURRENT IMPLEMENTATION

From:

F:\00-Ticenpi-SaaS\TicenpiLetter

find the real files controlling:

- Vue router
- /
- /ocr
- /letter
- shared shell
- sidebar
- letter toolbar
- letter panel
- A4 preview
- envelope preview
- zoom / fit controls

Do not assume filenames.

Map current structure before editing.

---

# PHASE 1 — ROOT ROUTE

Implement / -> /ocr.

Validate directly in browser.

---

# PHASE 2 — ISOLATE /letter FROM OLD SIDEBAR SHELL

Make /letter use the new focused editor shell.

If /ocr still depends on the existing application shell, keep /ocr behavior unchanged.

Do not break OCR page just to remove the shell from /letter.

If the shared shell architecture makes safe route-specific removal impossible without broad redesign:

HARD STOP.

Report:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS

Do not proceed with a risky shared-layout rewrite.

---

# PHASE 3 — BUILD NEW /letter HEADER

Implement a slim top header.

Required:

- 返回 OCR
- 開發信
- concise OCR-related subtitle

Remove redundant old toolbar UI.

---

# PHASE 4 — MOVE LETTER CONTROLS TO FAR-LEFT RAIL

Rebuild the current letter-panel layout as a proper left editor rail.

Important:

- do not change data behavior unnecessarily
- preserve current form bindings
- preserve current recipient/template/theme/export behavior
- remove brittle transform positioning
- no overlapping/floating panel

Make sections easy to scan.

---

# PHASE 5 — MAXIMIZE A4 PREVIEW

Use the remaining viewport for preview.

Evaluate current preview CSS and scale behavior.

Target:

- significantly larger than current preview
- stable on common desktop resolutions
- no giant empty region
- no clipping of important content
- page remains centered and readable

Envelope preview is secondary.

---

# PHASE 6 — REMOVE FIT-TO-WINDOW

Remove UI and dead code only if safe.

If internal fit calculation is still needed for initial rendering, it may remain internally, but the user-facing "適應視窗" control must be gone.

Do not break preview initialization.

---

# PHASE 7 — LOCAL VALIDATION

Run existing frontend tests/build as appropriate.

Then browser-check:

1. http://127.0.0.1:3013
   -> lands on /ocr

2. /ocr
   - still works
   - no unintended layout regression

3. /letter
   - old app sidebar shell absent
   - new top header present
   - 返回 OCR works
   - editor controls on far left
   - controls remain usable
   - A4 preview clearly larger
   - envelope preview works if retained
   - no 適應視窗 control
   - no overlap / broken scrolling
   - no new relevant console errors

Prefer actual browser visual verification, not source-only claims.

---

# PASS CRITERIA

PASS only if all are true:

- / opens /ocr
- /ocr still works
- /letter old sidebar shell removed
- /letter top header redesigned
- 返回 OCR works
- editor settings moved to far left
- A4 preview is materially larger and dominant
- 適應視窗 removed
- existing letter functions remain available
- frontend build/tests relevant to scope pass
- browser visual check passes
- no backend/DB/Docker/Staging/Production changes

---

# FINAL REPORT

Return exactly:

ORC LETTER UI REDESIGN FINAL

1. SOURCE
Path:
Branch:
HEAD before:
HEAD after:

2. ROUTING
/ behavior:
/ocr behavior:
/letter behavior:

3. UI CHANGES
Header:
Left rail:
A4 preview:
Envelope preview:
Old sidebar shell:
Fit-to-window:

4. FILES CHANGED
- ...

5. LOCAL VALIDATION
/ -> /ocr:
/ocr:
/letter:
Back button:
Letter controls:
A4 size:
Console:
Tests/build:

6. OUT-OF-SCOPE
Backend changed = NO
DB changed = NO
Docker changed = NO
Staging changed = NO
Production changed = NO

7. FINAL STATUS
PASS / BLOCKED / FAIL
