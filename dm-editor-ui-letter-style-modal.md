# DM EDITOR UI V3 — LETTER-STYLE DIRECT EDIT MODAL + COMPACT CASE DRAWER

## OWNER
DM EDITOR UI OWNER

## WORKING REPO
`F:\00-Ticenpi-SaaS\TicenpiDM`

## REFERENCE REPO
`F:\00-Ticenpi-SaaS\TicenpiLetter`

Use LETTER only as a UX / interaction reference for:
- direct click on A4 text
- centered edit modal
- live preview behavior
- text formatting controls
- object selection behavior

Do NOT copy unrelated LETTER-only features into DM.

---

# GOAL

This task is a Local-only DM editor UI refinement.

The target interaction model is:

1. No Top Bar
2. Left side stays as a compact Icon Rail
3. Clicking the Cases icon opens a compact case-list Drawer/Panel
4. A4 preview remains double-sided
5. Clicking text directly on the A4 preview opens a centered modal for editing
6. Do NOT use a permanent right-side properties inspector
7. Text editing behavior should follow the LETTER project's centered modal model
8. Keep the canvas as visually dominant as possible

This task is NOT a backend rewrite and NOT a redesign of the DM data model.

---

# HARD BOUNDARIES

Only modify:

`F:\00-Ticenpi-SaaS\TicenpiDM`

Do NOT modify:

- `F:\00-Ticenpi-SaaS\TicenpiDM_release_wt`
- Docker image / container
- VPS
- Staging
- Production
- ExtractionHub behavior
- scraper business logic
- RemoveBG backend logic
- Production auth
- deployment scripts

Do NOT commit, push, build Docker, or deploy unless separately approved.

Before any edit, report:

1. repo root
2. branch
3. HEAD
4. git status
5. current uncommitted WIP
6. active worktrees / possible collision

If the writable repo is not exactly `F:\00-Ticenpi-SaaS\TicenpiDM`, HARD STOP.

---

# IMPORTANT CORRECTION TO PREVIOUS UI DIRECTION

Do NOT implement a permanent right-side properties panel.

The previous idea of:

`left rail + center canvas + right inspector`

is NOT the target.

The correct target is:

`left icon rail + optional compact drawer + centered A4 canvas`

When the user wants to edit an A4 object, use a centered modal.

The A4 canvas must stay as wide and unobstructed as practical.

---

# TARGET LAYOUT

Desktop-first.

Conceptually:

```
┌──────┬─────────────────────────────────────────────────────┐
│ Icon │                                                     │
│ Rail │             A4 FRONT      A4 BACK                   │
│      │                                                     │
│      │                                                     │
└──────┴─────────────────────────────────────────────────────┘
                  bottom status / zoom / save
```

When case list is opened:

```
┌──────┬───────────────┬─────────────────────────────────────┐
│ Icon │ Cases Drawer  │       A4 FRONT      A4 BACK         │
│ Rail │ compact       │                                     │
│      │ text-first    │                                     │
└──────┴───────────────┴─────────────────────────────────────┘
```

When text is clicked:

```
A4 remains visible in background
              ↓
      centered Text Edit Modal
```

No permanent right inspector.

---

# A. NO TOP BAR

Remove / avoid fixed Top Bar in the DM editor workspace.

The editor should use the full viewport height.

Preserve required actions through:
- Icon Rail
- bottom status/control area
- contextual modal
- compact drawers

Do not reintroduce a heavy top navigation bar.

---

# B. A4 PREVIEW — KEEP DOUBLE-SIDED

The DM A4 preview is double-sided.

Preserve:
- front page
- back page
- current double-sided layout behavior
- existing page dimensions / print behavior
- existing object positions unless required for the new interaction

Do NOT redesign the actual content of the A4 pages in this task.

Do NOT change the print output just to make the editor UI fit.

The editor UI must adapt around the A4 preview, not the other way around.

---

# C. DIRECT CLICK TEXT EDITING — PRIMARY CHANGE

This is a core requirement.

Any editable text object shown on the A4 preview should be directly clickable.

Expected interaction:

```
User clicks text on A4
↓
select that text object
↓
open centered Text Edit Modal
↓
edit text and formatting
↓
A4 preview updates live
↓
Apply = keep changes
Cancel = rollback to state before modal opened
```

Do NOT require the user to:
- first click a Text tool
- then locate the object in another panel
- then edit in a permanent side inspector

Direct A4 object interaction is the priority.

---

# D. TEXT EDIT MODAL — FOLLOW LETTER UX

Inspect the actual LETTER implementation before coding.

Reference repo:

`F:\00-Ticenpi-SaaS\TicenpiLetter`

Find the real source files that implement:
- A4 text click
- text modal
- formatting toolbar
- live preview
- apply/cancel rollback
- modal sizing
- text selection behavior if already implemented

Reuse proven interaction patterns where appropriate, but adapt to DM's architecture instead of copy-pasting blindly.

## Modal behavior

Target:
- centered
- large enough for comfortable editing, roughly LETTER-style
- around 50vw width and high usable vertical space where appropriate
- should not cover the entire screen unnecessarily
- A4 remains visible behind it
- visually clear selected text editing context

Prefer:
- clicking backdrop does NOT close
- ESC does NOT silently discard work
- explicit Apply
- explicit Cancel

Cancel must restore the pre-modal state.

---

# E. LIVE PREVIEW

While the Text Edit Modal is open:

- typing should immediately reflect on the A4 preview
- font changes should immediately reflect
- font size changes should immediately reflect
- bold / italic / underline should immediately reflect
- color should immediately reflect
- alignment should immediately reflect
- line-height should immediately reflect
- letter-spacing should immediately reflect

Do not require pressing Apply just to see what the final A4 looks like.

Apply confirms.

Cancel rolls back.

---

# F. WYSIWYG

The editing experience should match the A4 result as closely as practical.

Investigate and align:
- font-family
- font-size
- font-weight
- line-height
- letter-spacing
- width
- wrapping
- alignment

Avoid a situation where:
- modal looks one way
- A4 wraps differently
- print output becomes visibly different

The A4 preview remains the source of truth.

---

# G. TEXT FORMATTING IN MODAL

Do NOT spread font controls across the main workspace.

All relevant text controls should be inside the centered text-edit modal.

At minimum, if already supported by DM architecture:

- font family
- font size
- font weight / bold
- italic
- underline
- text color
- alignment
- line-height
- letter-spacing

If some controls do not yet exist, first report the current support before expanding scope.

If LETTER already has a reusable implementation pattern, adapt it.

---

# H. SELECTED TEXT / WHOLE OBJECT BEHAVIOR

If the current editor architecture supports rich-text selection:

- selected text range → formatting affects selected range only
- no selected range → formatting affects the whole text object

If DM currently stores text as plain strings and does NOT support inline rich spans, do NOT silently redesign the data model.

Instead:
1. report the limitation
2. keep whole-object formatting for this task
3. propose inline range formatting as a separate follow-up

Do not create a risky rich-text migration just to satisfy this task.

---

# I. CASE LIST — COMPACT ICON-TRIGGERED DRAWER

The case list is NOT the A4 preview.

This change only affects how the Icon-triggered case list is displayed.

Current problem:
- opening the case list blocks too much of the A4 preview
- property titles are hard to read
- fully showing every title would make rows too tall
- adding large thumbnails would consume even more space

Target:

- click Cases icon
- open compact drawer/panel
- drawer should be narrow and information-dense
- no large thumbnail cards
- A4 preview should remain as visible as practical

## Each case row

Use a compact text-first structure:

- title: maximum 1–2 lines
- overflow: ellipsis
- compact secondary metadata line
- price aligned to the right if available
- selected state
- small overflow/more control only if needed

Example:

```
春季居家系列－美好生活從家…
3房2廳｜68坪｜大安區          2,380萬
```

Do NOT make every row tall just to show full title.

## Full title access

Provide one low-space-cost way to see full title:
- hover tooltip
- selected-case summary area
- or compact selected detail strip at bottom of drawer

Prefer the cleanest implementation consistent with the current UI.

---

# J. CASE DRAWER SEARCH / FILTER

Keep the drawer practical.

Support existing capabilities where already available:
- search
- recent / all cases
- selected state
- scroll

Do not add complex new case-management functionality unless already supported.

This task is mainly presentation and interaction optimization.

---

# K. LEFT ICON RAIL

Keep a slim Icon Rail.

Potential entries based on current DM:
- Cases
- Text
- Images
- Style
- Add Object
- Settings

Rules:
- compact width
- consistent icon size
- clear active state
- hover text / tooltip
- no giant text buttons
- do not duplicate controls that are available by directly clicking A4 objects

Important:
Because text is now directly editable from A4, the Text icon should not force a permanent side panel.
It may be used for:
- adding new text
- opening text-related creation action
- or remain as existing creation entry

Existing text objects should be edited by clicking them directly.

---

# L. STYLE CONTROLS

Do not leave a large group of style buttons permanently exposed.

Where current DM has style presets:
- prefer a compact dropdown
- compact popover
- or collapsible preset menu

Keep this separate from text-object formatting.

Do not mix:
- whole-document style preset
with
- current text object's font formatting

---

# M. IMAGES / OBJECT INTERACTION

Inspect the existing DM object interaction model.

If existing image objects can safely be clicked directly:
- preserve / improve direct selection
- do not introduce a right inspector just for consistency

For this task, prioritize text editing first.

Do not broaden scope into a full transform-system rewrite unless required.

If image editing already has an appropriate modal/popover pattern, preserve it.

---

# N. BOTTOM STATUS / CONTROLS

Since Top Bar is removed, keep lightweight persistent status/actions at the bottom where appropriate.

Examples:
- autosave state
- front/back state
- zoom
- fullscreen
- save status

Do not overload the bottom bar.

The A4 canvas must remain visually dominant.

---

# O. AUTOSAVE / DATA SAFETY

This UI task must not break the current draft lifecycle.

Before touching modal live-preview behavior, identify:
- current autosave trigger
- dirty state
- hydrated state
- draft persistence behavior

Important:
Live preview changes inside an open modal must not accidentally become permanently saved if the user presses Cancel.

Cancel must rollback both:
- visible Vue state
- any draft/autosave state that would otherwise persist the temporary edit

If current autosave architecture makes this unsafe, HARD STOP and report before implementing a workaround.

---

# P. IMPLEMENTATION PHASES

## Phase 1 — Investigation

Before modifying code, identify:

1. current main editor component
2. A4 front/back preview components
3. current text object rendering component
4. current text editing logic
5. current object-selection state
6. current case-list component
7. current Icon Rail component
8. current style controls
9. current autosave/draft flow
10. corresponding LETTER components implementing direct text click + modal

Report:
- exact files
- current interaction path
- minimal proposed changes

Then implement.

## Phase 2 — Implementation

Implement minimal coherent changes:
- no Top Bar
- compact case drawer
- preserve double-sided A4
- direct click text
- centered text modal
- modal formatting controls
- live preview
- Apply / Cancel rollback
- no permanent right inspector

Do not rewrite unrelated editor architecture.

## Phase 3 — Local Validation

Run the current project's existing relevant tests and build.

At minimum validate manually/local browser:

### Case drawer
- Cases icon opens compact drawer
- drawer does not excessively block A4
- long title truncates cleanly
- full title is still accessible
- selecting a case still works

### A4 text edit
- click existing front-page text
- modal opens centered
- edit content
- A4 updates live
- Apply persists
- Cancel restores old value

Repeat on back-page text.

### Formatting
- font family
- font size
- color
- alignment
- supported style controls

### Layout
- no Top Bar
- double-sided preview remains
- no permanent right inspector
- canvas remains usable at normal desktop width

### Persistence
- F5 after Apply retains expected saved state according to current design
- Cancel does not become an autosaved permanent change

---

# ACCEPTANCE CRITERIA

PASS only if all are true:

1. No Top Bar in the DM editor
2. A4 remains double-sided
3. No permanent right-side properties panel
4. Existing A4 text can be clicked directly
5. Clicking text opens centered modal
6. Modal supports practical text editing / existing formatting
7. A4 updates live while editing
8. Apply keeps edits
9. Cancel correctly rolls back
10. Case list is a compact Icon-triggered drawer
11. Case list uses compact rows, not large thumbnail cards
12. Long case titles do not make rows excessively tall
13. Full case title remains accessible
14. Autosave does not make Cancel unsafe
15. Local frontend build/tests pass
16. No Docker / Staging / Production changes

---

# HARD STOP CONDITIONS

Stop and report instead of improvising if:

- Cancel cannot safely rollback because autosave persists temporary modal edits
- implementing selected-range rich text requires a data model migration
- double-sided A4 structure would need to be rewritten
- current WIP conflicts with another active editor refactor
- implementation would require backend schema changes
- working tree/repo differs from the required path
- changes would affect Staging or Production auth/deployment

---

# FINAL REPORT

Return:

1. repo / branch / HEAD
2. files changed
3. how direct text click works
4. how Text Modal works
5. how live preview + Apply/Cancel works
6. how autosave safety was handled
7. how case drawer was made more compact
8. how long case titles are handled
9. confirmation that no permanent right inspector remains
10. confirmation that double-sided A4 remains
11. tests/build/browser checks
12. remaining limitations
13. Docker changed = NO
14. Staging changed = NO
15. Production changed = NO
