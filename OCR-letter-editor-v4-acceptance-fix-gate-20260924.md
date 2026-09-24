# OCR Letter Editor V4 — Acceptance / Minimal Fix Gate

## OWNER

OCR LETTER EDITOR V4 ACCEPTANCE OWNER

## MODEL

GPT-5.6 Luna — High

## MODE

This is a **narrow acceptance + minimal-fix task**.

Do NOT redesign the editor.
Do NOT add new product features.
Do NOT refactor architecture unless required to fix one of the explicitly listed acceptance failures.
Do NOT touch unrelated UI.

The purpose is to finish the V4 browser acceptance that was left unverified.

---

# CANONICAL SOURCE

F:\00-Ticenpi-SaaS\TicenpiLetter

Local canonical host:

http://localhost:3013/

Letter editor:

http://localhost:3013/letter

Current expected Git HEAD:

4aa0fa9

IMPORTANT:

V2 / V3 / V4 work may still exist as uncommitted changes on top of this HEAD.

Preserve the current working tree exactly.

At start record:

- branch
- HEAD
- git status
- tracked dirty files
- untracked files

Never:

- reset
- restore
- checkout old versions
- clean
- delete unrelated untracked files
- use git add -A
- commit
- push

---

# CURRENT V4 STATUS

Already reported PASS / implemented:

- visible OCR naming
- / root OCR route
- /ocr removed
- /letter editor
- text modal behavior
- rich text implementation
- live preview
- handwriting fonts
- object add
- front/back selectedObject model
- envelope info
- 開發信條 independent icon
- 200+ 開發信條 pagination
- 開發信條 memory-only lifecycle
- normal Letter IndexedDB persistence
- .ocrletter save/import
- JPG removed
- tests 36/36 PASS
- Vite build PASS

Do NOT redo these areas unless one of the acceptance checks below proves a regression.

---

# ONLY FOUR PRIMARY ACCEPTANCE TARGETS

You are allowed to modify code ONLY when necessary to make one of these pass:

1. LOGO / LINE QR actual resize + persistence
2. Sticker transform stress / disappearance bug
3. Real portrait RemoveBG browser flow
4. 200-entry 開發信條 PDF export completion

Also capture browser console evidence during the checks.

---

# 1. LOGO / LINE QR — ACTUAL RESIZE ACCEPTANCE

## Goal

Prove that both LOGO and LINE QR are truly editable canvas objects, not just selectable.

Test each separately.

### LOGO

- click LOGO directly on A4
- confirm selected state
- resize visibly smaller
- resize visibly larger
- move it
- confirm it remains visible
- F5
- confirm position + size restore correctly from normal Letter persistence

### LINE QR

Repeat:

- direct click
- select
- resize smaller
- resize larger
- move
- remain visible
- F5
- position + size restore

## PASS criteria

LOGO_RESIZE = PASS only if actual browser dimensions visibly change and survive F5.

QR_RESIZE = PASS only if actual browser dimensions visibly change and survive F5.

Do not mark PASS based only on code inspection.

## Minimal fix rule

If resize fails:

- inspect shared object/transform handling
- make the smallest fix
- do not create separate transform architecture for LOGO and QR
- reuse shared selectedObject / transform state

---

# 2. STICKER TRANSFORM STRESS TEST

## Goal

Prove that image/sticker objects cannot disappear during repeated transform operations.

Use an existing normal image or portrait sticker.

Perform this sequence in the real browser:

1. select sticker
2. rotate
3. rotate again
4. resize smaller
5. resize larger
6. drag to another position
7. rotate again
8. horizontal flip
9. bring forward
10. send backward
11. drag again
12. double click if current UI responds to it
13. resize again
14. rotate again

Then confirm:

- object is still visible
- object remains selectable
- object remains inside or recoverable relative to the A4 canvas
- no NaN/invalid transform
- no transform explosion
- no sudden zero width/height

## Reset test

Use:

重設位置 / 變形

Confirm it returns the object to a safe visible state.

## F5 persistence

After a non-default transform:

- F5
- confirm normal Letter persistence restores the transformed object correctly

## PASS criteria

STICKER_TRANSFORM_STRESS = PASS only after actual browser manipulation.

If the object disappears:

- reproduce once more
- inspect x/y/width/height/rotation/flip/zIndex
- fix only the shared transform bug
- rerun the full stress sequence

Do not redesign the image UI.

---

# 3. REAL PORTRAIT REMOVE BG ACCEPTANCE

## Goal

Prove AI 去背 really produces a transparent portrait that is usable on A4.

A 200 response is NOT sufficient.

## Reference

If frontend integration is wrong, inspect the already-working DM implementation:

F:\00-Ticenpi-SaaS\TicenpiDM

Compare ONLY RemoveBG flow:

- endpoint
- method
- FormData field
- MIME
- auth/header handling
- response format
- blob/base64 conversion
- transparent PNG handling
- error handling

Do not copy unrelated DM editor code.

## Test input

Use an existing suitable local/test portrait image if one is already available in the repo or current test assets.

Do NOT invent a fake transparent result.

If no real portrait image is available locally, report:

REMOVE_BG_INPUT_REQUIRED

with the exact UI/file input needed.

Do not broaden scope.

## Browser acceptance

Perform:

1. upload/select real portrait
2. show original with background
3. click AI 去背
4. wait for actual completion
5. confirm background is visually transparent
6. confirm result is placed on A4
7. move it
8. resize it
9. rotate it
10. confirm transparency remains

Inspect browser console/network only as evidence, not as the final success criterion.

## PASS criteria

REMOVE_BG = PASS only if transparent visual output is actually observed in browser.

If frontend integration differs from DM and can be fixed without backend changes:

apply minimal frontend fix and retest.

## HARD STOP

If success requires:

- backend endpoint contract change
- new backend code
- new secret
- new external service

STOP and report:

REMOVE_BG_BACKEND_BLOCKED

Do not modify backend.

---

# 4. 開發信條 — 200 ENTRY PDF EXPORT ACCEPTANCE

## Goal

Prove that a real 200-entry 開發信條 PDF can finish exporting.

Do NOT treat browser automation timeout by itself as proof of failure.

## Setup

Create/load approximately 200 temporary 開發信條 entries in the existing memory-only session.

Verify:

- modal remains usable
- pagination/windowed rendering remains responsive
- temporary data remains memory-only

## PDF test

Record:

- entry count
- export start timestamp
- export completion timestamp
- elapsed time

Click:

匯出 PDF

Then wait for completion.

If the browser automation step times out:

DO NOT immediately mark FAIL.

Instead verify through local filesystem/browser download evidence whether the PDF finished after the automation timeout.

If appropriate, inspect the user's normal browser Downloads destination for the most recent matching PDF.

Do not delete unrelated files.

## Verify actual output

Confirm:

- a real PDF file was produced
- file size > 0
- PDF opens
- expected 200-entry content is represented
- no obvious blank/corrupt output
- no permanent temp PDF stored on VPS
- temporary Blob/Object URLs are released where applicable

If exact page count differs because multiple letter strips are designed per page, verify the expected **entry count/content**, not blindly 200 pages.

## Performance diagnosis

If export is very slow, determine whether the issue is:

- expected client-side generation time
- rendering all 200 entries at once
- image/font embedding overhead
- repeated synchronous work
- actual hang/infinite wait

Only optimize if needed to make the existing feature complete reliably.

Do not redesign the PDF pipeline if it already completes acceptably.

## PASS criteria

LETTER_STRIP_200_PDF = PASS only when the actual PDF is confirmed.

---

# 5. BROWSER CONSOLE / NETWORK EVIDENCE

During all four checks capture:

- relevant console errors
- relevant warnings
- failed network requests
- RemoveBG request result
- PDF export exceptions if any

Final requirement:

NEW_RELEVANT_CONSOLE_ERRORS = NONE

Pre-existing unrelated warnings may be reported separately.

Do not fix unrelated warnings.

---

# 6. DO NOT MODIFY THESE AREAS UNLESS REGRESSION IS PROVEN

Do not touch:

- OCR home UI
- routing contract
- recipient-management UX
- text modal layout
- rich-text design
- handwriting font list
- envelope info UX
- 開發信條 memory-only lifecycle
- save/import UI
- normal autosave architecture
- fullscreen design
- general icon rail design

No new features.

No visual redesign.

---

# 7. PERSISTENCE SAFETY

Normal Letter Editor:

- IndexedDB persistence remains active
- F5 restores normal editor state

開發信條:

- remains memory-only
- F5 clears it
- no localStorage
- no sessionStorage
- no IndexedDB
- no backend/VPS persistence
- not included in .ocrletter

Do not accidentally persist 開發信條 while fixing PDF export.

---

# 8. FILE / SCOPE SAFETY

Allowed:

- frontend-only minimal fixes related directly to the four targets
- existing test updates/additions directly covering the fixes

Not allowed:

- backend
- DB
- Docker
- CI
- deploy
- Staging
- Production
- auth/entitlement
- unrelated refactors
- new external service
- new secret
- major dependency

If more than 6 tracked files need changes:

HARD STOP before modifying the 7th file.

Explain why.

---

# 9. VALIDATION ORDER

Follow this order:

1. preflight/git state
2. reproduce LOGO/QR resize
3. reproduce sticker transform stress
4. reproduce RemoveBG
5. reproduce 200-entry PDF export
6. make only necessary fixes
7. rerun all four browser checks
8. capture console/network evidence
9. git diff --check
10. existing frontend tests
11. production build

Do not stop after source-level inspection.

---

# 10. REQUIRED TESTS

At minimum:

git diff --check

Existing frontend tests:

expect current baseline of 36/36 or better.

Vite production build:

PASS required.

If a previous test fails due to your change:

FAIL until corrected.

If a known unrelated pre-existing failure appears, report separately and prove it is unrelated.

---

# 11. HARD STOP CONDITIONS

HARD STOP if:

1. RemoveBG requires backend modification
2. PDF export requires backend/VPS persistence
3. current working tree cannot be preserved safely
4. a new external service/secret is required
5. more than 6 tracked files are required
6. a major dependency is required
7. resolving one acceptance target requires broad editor redesign

Output only:

CURRENT
EXPECTED
CONFLICT
EVIDENCE
MINIMUM OPTIONS

---

# 12. PASS RULE

Overall PASS requires:

LOGO_RESIZE = PASS
QR_RESIZE = PASS
STICKER_TRANSFORM_STRESS = PASS
RESET_TRANSFORM = PASS
REMOVE_BG = PASS
LETTER_STRIP_200_PDF = PASS
NEW_RELEVANT_CONSOLE_ERRORS = NONE
git diff --check = PASS
frontend tests = PASS
build = PASS

If any critical browser acceptance remains unverified:

do NOT report PASS.

Use:

BLOCKED

only for a real external/backend/input blocker.

Use:

FAIL

when the feature is demonstrably broken and not fixed.

---

# FINAL REPORT

Return exactly:

OCR V4 ACCEPTANCE / FIX FINAL

1. SOURCE
Path:
Branch:
HEAD:
Tracked dirty before:
Tracked dirty after:
Files changed by this task:

2. LOGO
Direct select:
Resize smaller:
Resize larger:
Move:
F5 restore:
LOGO_RESIZE:

3. LINE QR
Direct select:
Resize smaller:
Resize larger:
Move:
F5 restore:
QR_RESIZE:

4. STICKER STRESS
Repeated rotate:
Resize:
Drag:
Flip:
Z-order:
Double click:
Still visible:
Reset transform:
F5 restore:
STICKER_TRANSFORM_STRESS:

5. REMOVE BG
Test image:
DM flow compared:
Request:
Transparent visual result:
Placed on A4:
Move/resize/rotate:
REMOVE_BG:

6. 開發信條 200 PDF
Entry count:
Start time:
Completion time:
Elapsed:
Automation timeout:
Actual PDF produced:
File size:
PDF opens:
Expected content verified:
VPS storage used = NO:
LETTER_STRIP_200_PDF:

7. CONSOLE / NETWORK
New relevant console errors:
Failed relevant requests:
Notes:

8. REGRESSION SAFETY
Normal Letter F5 restore:
開發信條 still memory-only:
開發信條 F5 clears:
OCR home:
Fullscreen:
Other V4 regression:

9. VALIDATION
git diff --check:
Frontend tests:
Build:

10. OUT OF SCOPE
Backend changed = NO
DB changed = NO
Docker changed = NO
CI/deploy changed = NO
Staging changed = NO
Production changed = NO
Commit/push performed = NO

11. FINAL STATUS
PASS / BLOCKED / FAIL
