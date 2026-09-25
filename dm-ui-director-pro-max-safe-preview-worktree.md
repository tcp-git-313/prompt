# DM UI DIRECTOR PRO MAX — SAFE VISUAL-FIRST REDESIGN / SANDBOX PREVIEW / NO LOCAL REPLACEMENT

## ROLE

你現在不是一般 coding agent。

你要同時扮演：

- Senior Product Designer
- UX Director
- Interaction Designer
- Design-System Architect
- Accessibility Reviewer
- Frontend Architecture Reviewer
- Usability Tester
- Visual QA Engineer

你的任務不是「稍微美化目前頁面」。

你的任務是重新審視 TicenpiDM 的整體使用體驗，讓它變成：

- 第一次打開就看得懂
- 常用操作步驟盡量少
- 正反 A4 一開始就像完成品
- 圖片 / 大頭照 / LOGO / LINE QR / Sticker 操作一致
- 按鈕層級清楚
- 不再有呆板、千篇一律、強烈 AI SaaS 模板味
- 有成熟商業設計工具的質感
- 不破壞任何既有 JavaScript / Vue 功能
- 在使用者看到並核准實際視覺稿以前，不得修改目前本機 TicenpiDM source

---

# WORKING PROJECT

目前真實本機專案：

`F:\00-Ticenpi-SaaS\TicenpiDM`

## CURRENT LOCAL UI — READ ONLY

`http://127.0.0.1:9422/`

這是目前使用者正在使用、而且包含未提交 dirty WIP 的本機版本。

**整個設計與 Preview 階段都不得修改這個 working tree。**

---

# EXACT REFERENCE UI — READ ONLY

不要只寫「比較 localhost:3013」。

必須精確打開以下 URL：

## OCR / Letter A4 預覽參考

`http://127.0.0.1:3013/專案`

只比較：

- A4 紙張陰影
- paper depth
- workspace background
- 正反面呈現
- A4 spacing
- A4 scale
- paper border / separation

## OCR / Letter 圖片與 Sticker 互動參考

`http://127.0.0.1:3013/專案設計`

只比較：

- LOGO selection
- LINE QR selection
- 圖片 direct manipulation
- Sticker
- Move
- Resize
- Rotate
- Flip
- Bring forward
- Send backward
- Reset
- Delete
- F5 restore / persistence behavior

## Reference source

`F:\00-Ticenpi-SaaS\TicenpiLetter`

只允許 READ ONLY。

---

# REFERENCE VALIDATION

在拿 3013 做任何比較之前：

1. 實際打開 exact URL。
2. 確認頁面內容真的符合預期。
3. 截圖並記錄 page title / route / visible landmarks。
4. 不得因為 port 是 3013 就假設它一定是正確頁面。
5. 若 `/專案` 或 `/專案設計` 不存在、載入錯誤、或內容不符，立即報告。
6. 不得拿錯頁面做設計依據。
7. 不得直接複製 OCR/Letter；它只是 interaction / visual-quality reference。

---

# REQUIRED SKILL — MUST READ FIRST

第一個設計動作之前，必須完整讀取：

`.claude-skills/ui-ux-pro-max.md`

優先解析為：

`F:\00-Ticenpi-SaaS\TicenpiDM\.claude-skills\ui-ux-pro-max.md`

如果該路徑不存在，可以在目前 Claude workspace 中搜尋**同名檔案**，但必須回報實際 resolved path。

必須完整理解其中：

- design principles
- UX reasoning
- visual hierarchy
- color guidance
- typography guidance
- spacing rules
- layout principles
- interaction guidance
- accessibility rules
- anti-patterns
- decision framework

禁止：

- 只讀摘要
- 用記憶取代實際內容
- 假裝已讀
- 找不到時用一般 UI 知識冒充

如果完全找不到：

`UI_UX_PRO_MAX_NOT_FOUND`

然後停止，不進入設計。

---

# CORE SAFETY MODEL

整個任務拆成四個 Gate。

```text
GATE 0 — Audit
READ ONLY

GATE 1 — Visual Proposal
READ ONLY
只出圖 / Figma / prototype / UX proposal

使用者輸入：
APPROVE_UI_IMPLEMENTATION

↓

GATE 2 — Preview Implementation
只改「獨立 sandbox / preview worktree」
原本 TicenpiDM 仍 READ ONLY

使用者看：
9422 原版 vs 9522 新版

使用者輸入：
APPROVE_LOCAL_MERGE

↓

GATE 3 — Merge
才允許把已核准差異合併回原本 TicenpiDM
```

沒有對應 approval token，不得越級。

---

# ABSOLUTE LOCAL SOURCE PROTECTION

在收到：

`APPROVE_LOCAL_MERGE`

之前，以下路徑一律 READ ONLY：

`F:\00-Ticenpi-SaaS\TicenpiDM`

禁止：

- 直接修改 source
- 直接修改 CSS
- 直接修改 Vue
- 直接修改 JS
- git reset
- git reset --hard
- git restore
- git checkout .
- git clean
- stash apply
- stash pop
- branch switch
- overwrite
- file replacement
- repo-wide formatter
- dependency upgrade
- lockfile rewrite

不得以：
- prototype
- quick fix
- temporary patch
- formatting
- cleanup

為理由修改原 working tree。

---

# CURRENT DIRTY WIP MUST BE PRESERVED

目前本機可能有大量未提交 WIP。

開始時先唯讀回報：

- repo root
- branch
- HEAD
- `git status --short`
- tracked modified files
- untracked files
- active worktrees
- relevant UI source files
- dev server status
- current frontend URL

不可要求使用者先 commit 才能進行設計。

---

# TOOLS — USE IF AVAILABLE

優先：

1. Playwright MCP / Playwright CLI
2. Claude browser / computer use
3. filesystem / repo inspection
4. screenshot / vision
5. Figma MCP
6. local browser/dev server

對 UI 判斷時，不要只讀程式碼。

必須實際操作目前畫面。

---

# GATE 0 — CURRENT UX AUDIT — READ ONLY

先實際操作：

`http://127.0.0.1:9422/`

至少測：

- Icon Rail
- 每個一級入口
- hover label
- cases drawer
- 正面 A4
- 反面 A4
- 文字點擊
- text modal
- layout/data/style
- image
- portrait
- logo
- LINE QR
- sticker
- page targeting
- add-object flow
- reset
- restore
- F5
- drawer open/close

每個功能記錄：

- 用途
- 使用頻率：High / Medium / Low
- 現在需要幾步
- 是否容易理解
- 是否值得永久佔位
- 是否可合併
- 是否可 contextualize
- 是否可直接在 A4 操作
- 是否和別的功能重複
- 是否有命名問題

建立：

`CURRENT_UI_AUDIT`

---

# SOURCE AUDIT — READ ONLY

讀目前實際 source，但不要修改。

至少找：

- DmEditor.vue
- A4Preview.vue
- preview.js
- DmEditor UI tests
- imported CSS / SCSS
- editor shell
- icon rail
- drawers
- text modal
- image components
- sticker state
- default front/back state
- layout presets
- style presets
- autosave
- hydrated / dirty gate
- F5 restore
- page targeting

建立：

`UI_COMPONENT_MAP`

包含：

- visual element
- source file
- component
- state
- CSS selector
- current dimension
- current typography
- interaction owner

---

# PRIMARY PRODUCT PROBLEM

目前 DM 要解決的不是「再加更多功能」。

而是：

1. UI 太呆板
2. 有強烈 AI dashboard 味
3. 按鈕層級不直覺
4. A4 正反面太像普通白紙
5. 紙張沒有足夠 depth
6. 新使用者打開太空
7. 沒有像完成品的預設 front/back
8. 使用者不知道哪裡應該放什麼
9. 從零排版成本太高
10. Portrait / Logo / QR / image / Sticker mental model 分裂
11. 功能很多，但缺乏引導
12. spacing / typography / color 缺少明確層級

---

# CORE UX PRINCIPLE

DM 不應要求一般房仲業務「先學設計」。

主流程應該是：

```text
看到完整預設成品
↓
點擊替換圖片
↓
點擊修改文字
↓
拖曳微調
↓
完成
```

核心原則：

- Template-first
- Replace-first
- Direct manipulation
- Progressive disclosure
- Contextual controls
- Minimal learning cost
- Clear hierarchy

---

# FUNCTION PRIORITY RULE

## High-frequency
直接可見或直接操作。

## Medium-frequency
Drawer / Popover / contextual menu。

## Low-frequency
More / Advanced。

## Object editing
優先直接在 A4 上。

## Text
直接點 → 編輯。

## Image
直接點 → handles / contextual actions。

## Page-level
才進 Layout / Page panel。

禁止把所有設定永久攤開。

---

# VISUAL DIRECTION — MUST PRODUCE 3

依 `ui-ux-pro-max.md` 和實際產品需求，至少出三套。

可以參考但不限於：

## A — Premium Japanese Minimal
- 精準留白
- restrained shadow
- neutral palette
- typography-first
- quiet premium

## B — Refined Bento Workspace
- Bento hierarchy
- modular controls
- contextual grouping
- dense but calm

## C — Layered Creative Tool
- subtle glass
- floating contextual controls
- layered canvas
- richer depth
- creative-tool feel

禁止：

- generic purple AI SaaS
- 大量 rainbow gradient
- every-panel glass
- giant cards everywhere
- excessive border-radius
- heavy blur
- heavy shadow
- neon
- decorative motion without UX value

---

# COLOR SYSTEM

依 UI/UX Pro Max 重建 Color Tokens。

至少列：

- canvas-bg
- surface-1
- surface-2
- elevated
- paper
- border-subtle
- border-strong
- text-primary
- text-secondary
- text-muted
- accent
- accent-hover
- accent-active
- success
- warning
- danger
- focus-ring
- overlay

必須提供：

- actual HEX / OKLCH / RGB
- usage
- contrast pairing
- contrast ratio

主要正文與關鍵控制文字盡可能達 WCAG AAA。

如果某些大字 / 非正文只達 AA，要明確標記，不准虛構 AAA。

---

# TYPOGRAPHY SYSTEM

產出 actual values：

- family
- size
- weight
- line-height
- letter-spacing
- usage

至少涵蓋：

- product label
- section title
- panel title
- body
- label
- helper
- metadata
- button
- tooltip
- A4 default title
- A4 default body

避免隨機 13 / 14 / 15 / 16px 混用。

---

# SPACING SYSTEM

建立統一 spacing scale。

至少規範：

- rail padding
- drawer padding
- panel gap
- button gap
- modal spacing
- A4 gap
- object padding
- section spacing

全部用一致 token，避免歷史數值散落。

---

# BUTTON / CONTROL SYSTEM

重新分類：

- Primary
- Secondary
- Tertiary
- Icon
- Contextual
- Destructive

Phase 1 要列 actual：

- height
- min-width
- icon size
- radius
- padding
- label style
- hover
- active
- focus
- disabled
- destructive

Icon-only 必須 tooltip。

---

# MICRO-INTERACTION / MOTION

目前是 Vue 專案。

不要因為原要求提過 Framer Motion 就直接加 React library。

優先：

- CSS transition
- CSS transform
- Vue Transition
- 現有 motion dependency（如果已存在）

只在必要時才新增 dependency。

需要設計：

- hover
- press
- tooltip
- drawer
- modal
- selected object
- A4 focus
- contextual actions
- sticker handles

Phase 1 要提出：

- duration tokens
- easing tokens
- use cases
- reduced-motion behavior

動效必須提升理解，不得延遲操作。

---

# A4 WORKSPACE

A4 是主角。

目標：

- 無 Top Bar
- 100dvh
- outer vertical scrollbar = none
- compact left rail
- 正反 A4 最大化並排
- A4 有 paper shadow / depth
- workspace background 有層次
- front/back 清楚可辨

參考：

`http://127.0.0.1:3013/專案`

但只能借鑑：

- depth
- shadow
- paper treatment
- spacing
- workspace feel

不可盲目複製。

---

# IMAGE / ASSET / STICKER MODEL

從使用者 mental model 統一：

```text
Assets
├─ Business Portrait
├─ Logo
├─ LINE QR
├─ Upload Image
└─ Sticker
```

不要讓使用者學五套操作。

共同互動盡量統一：

- Select
- Drag
- Resize
- Rotate
- Flip
- Bring Forward
- Send Backward
- Reset
- Delete

若底層 state 不同沒關係，UX 要一致。

大頭照不應該繼續是一個孤立的特殊操作模式。

---

# TEXT / STICKER EXTENSIBILITY

除了圖片 Sticker，評估「文字物件」是否也應符合相同 direct-manipulation mental model。

但本輪不要擅自重寫 rich-text schema。

Phase 1 只提出：

- 哪些文字應 direct edit
- 哪些文字可作為 free text object
- 哪些屬於資料綁定欄位
- 如何讓兩者不混亂

---

# TEMPLATE-FIRST NEW USER EXPERIENCE

新使用者不能再打開看到兩張空白 A4。

新 / empty draft 應直接看到一套完整 front/back template。

Template 要像已完成 70% 的作品。

包含：

- hero image
- editable title
- subtitle
- property content
- sample cards / sections
- logo placeholder
- portrait placeholder
- LINE QR placeholder
- CTA
- supporting content

目標不是限制使用者能放幾個物件。

目標是讓他一眼理解：

`這裡能放什麼，而且做完大概會長這樣。`

Existing draft 絕對不得被新預設覆蓋。

---

# PLACEHOLDER PRINCIPLE

Placeholder 不能只是醜灰框。

應該：

- 看起來像真實成品
- 有合理 sample content
- 有 replacement affordance
- hover 才顯示提示
- 正常狀態不要滿畫面教學文字

例如：

- 更換主圖
- 編輯標題
- 上傳業務照片
- 替換 Logo
- 設定 LINE QR

---

# TEMPLATE DIRECTIONS — FRONT + BACK

至少設計三套：

## A — Editorial / Magazine
- hero 主導
- 不對稱節奏
- 強 typography
- premium

## B — Real Estate Commercial
- 房仲資訊清楚
- listing hierarchy
- business / QR / CTA 清楚
- 容易替換

## C — Personal Brand / Social
- 業務形象更突出
- Sticker 感
- 個人品牌感
- 行銷 CTA 強

不要把「2×3 六格」當硬需求。

它只是一個例子。

版型可以：

- 1 hero + supporting cards
- 2×2
- mixed grid
- asymmetric
- editorial
- max 4-column system

若目前 layout engine 已有 max 4 columns / 12 rows，要遵守；若沒有，不要為了 mockup 擅自改 schema。

---

# FRONT / BACK BOTH COMPLETE

Front 與 Back 都不能是空白或純資料表。

## Front
至少應具有：
- 品牌
- 主視覺
- 主訊息
- 主案件 / 內容
- CTA / 業務識別

## Back
至少應具有：
- supporting listings/content
- additional visuals
- services / detail
- business contact
- QR / CTA

兩面應共享同一 design language。

---

# UX CLICK COUNT

量測目前：

- 更換主圖
- 修改標題
- 換業務照片
- Logo
- LINE QR
- 移動圖片
- resize
- rotate
- 正面 layout
- 反面 layout

建立：

`BEFORE_CLICK_COUNT`

再提出：

`TARGET_CLICK_COUNT`

常用操作盡量 <= 2 interactions。

---

# GATE 1 — VISUAL PROPOSAL ONLY

這一階段 **禁止改 TicenpiDM source**。

必須產生真正可看的視覺稿。

優先順序：

## Option 1 — Figma MCP
若可用：
- 建可編輯 Figma frames
- 至少 3 個 desktop visual directions

## Option 2 — Static HTML Prototype
若 Figma 不可用：
- 在 repo 外的 temp/sandbox directory 建 static prototype
- 不 import / 修改 TicenpiDM source
- 用 browser render

## Option 3 — Rendered Mockup
若可產圖：
- 根據目前 DM screenshot 做 visual mockup

無論哪一種，都要輸出實際 screenshot。

---

# REQUIRED MOCKUP SCREENS

至少產出：

1. Full DM Editor — 正反 A4 同時顯示
2. New-user Front template
3. New-user Back template
4. Icon Rail + hover label
5. Assets panel
6. Sticker selected state
7. Text edit state
8. Layout / Page settings state

如果 3 套風格全部做完整八張成本太高：

- 至少三套都要有 Full Editor 主畫面
- 推薦方案再補完整其餘狀態

---

# GATE 1 OUTPUT

輸出：

## CURRENT UX PROBLEMS
## UI/UX PRO MAX PRINCIPLES APPLIED
## THREE VISUAL DIRECTIONS
## RECOMMENDED DIRECTION
## COLOR TOKENS
## TYPOGRAPHY
## SPACING
## BUTTON / CONTROL SPEC
## MOTION TOKENS
## INFORMATION ARCHITECTURE
## FRONT TEMPLATE
## BACK TEMPLATE
## ASSET / STICKER MODEL
## BEFORE / AFTER CLICK COUNT
## IMPLEMENTATION IMPACT
## MOCKUP / FIGMA / SCREENSHOTS

最後必須輸出：

`PHASE_1_COMPLETE_WAITING_FOR_APPROVE_UI_IMPLEMENTATION`

然後停止。

---

# VISUAL APPROVAL GATE

在使用者明確輸入：

`APPROVE_UI_IMPLEMENTATION`

之前：

- 禁止修改 TicenpiDM source
- 禁止修改 CSS
- 禁止修改 Vue
- 禁止修改 JS
- 禁止建立 implementation commit
- 禁止「順手修」
- 禁止以 prototype 名義寫回 repo

「看起來不錯」、「這個可以」、「我喜歡 B」都不能自動視為 implementation approval。

只有 exact approval token 才能進下一階段。

---

# GATE 2 — CREATE ISOLATED PREVIEW IMPLEMENTATION

收到：

`APPROVE_UI_IMPLEMENTATION`

後，**仍然禁止修改原本 TicenpiDM working tree。**

必須建立獨立 preview environment。

建議 preview root：

`F:\00-Ticenpi-SaaS\.ui-preview\TicenpiDM`

或安全的獨立 git worktree：

`F:\00-Ticenpi-SaaS\TicenpiDM_ui_preview_wt`

---

# IMPORTANT — PREVIEW MUST INCLUDE CURRENT DIRTY WIP

因為目前 `TicenpiDM` 有未提交 WIP，單純從 HEAD 建 worktree 可能不是目前畫面。

所以 Preview baseline 必須反映「目前 9422 真實 source 狀態」。

安全做法：

1. 建新 worktree from current HEAD。
2. 不修改 original。
3. 唯讀取得 original working tree 的 tracked diff。
4. 將 dirty tracked changes 複製 / 套用到 preview worktree。
5. 將 UI 所需 untracked files 複製到 preview。
6. 不要把：
   - .git
   - node_modules
   - dist
   - cache
   - secrets
   - runtime temp
   當成一般 copy payload。
7. 驗證 relevant source hashes / file contents，使 Preview baseline 與目前 9422 source 一致。
8. 只有 baseline parity 完成後，才在 preview 改 UI。

禁止為建立 preview 對 original 使用：

- stash
- reset
- restore
- commit
- clean

---

# PREVIEW URL

原版：

`http://127.0.0.1:9422/`

必須保持不變。

新版 Preview：

`http://127.0.0.1:9522/`

若 9522 已占用，可選下一個未占用 port，但必須回報 actual preview URL。

---

# PREVIEW DATA ISOLATION

因為不同 port 有不同 browser origin：

- 使用獨立 browser profile/context
- 不共用原版 localStorage
- 不覆寫原版 local draft

如果 backend autosave 會寫到共享 local DB：

- 只使用 disposable/test draft
- 不操作使用者正式本機 draft
- 不改 schema
- 不碰 Staging / Production

若無法避免 preview 寫入共享正式本機資料，停止並回報資料隔離風險。

---

# GATE 2 IMPLEMENTATION RULES

只在 preview worktree / sandbox 修改。

優先：

- reuse current state
- reuse current data model
- reuse current autosave
- reuse current direct text editing
- reuse current image persistence
- CSS / component refactor first

禁止：

- backend schema migration
- Production changes
- Staging changes
- Docker deployment
- release worktree changes
- unrelated refactor
- dependency mass upgrade
- rewrite state architecture just for visual polish

---

# MUST PRESERVE EXISTING FUNCTIONALITY

至少保留：

- A4 direct text click
- centered text modal
- live preview
- Apply
- Cancel rollback
- autosave hydrated/dirty safety
- front/back
- existing drafts
- F5 restore
- LOGO behavior
- LINE QR behavior
- image/sticker transform
- z-order
- reset

Visual redesign 不能以功能退化換漂亮。

---

# GATE 2 VALIDATION

用 Playwright 對照：

## Original
`http://127.0.0.1:9422/`

## Preview
`http://127.0.0.1:9522/`

做 side-by-side 驗證。

至少檢查：

- overall editor
- front A4
- back A4
- icon rail
- drawer
- text modal
- assets
- sticker selected
- hover states
- responsive desktop behavior

並確認 Preview 視覺與使用者核准的 mockup 一致。

---

# FUNCTIONAL TESTS

Preview 必須跑：

- existing relevant unit tests
- component tests
- Vite build
- Playwright smoke

不得刪測試來換 PASS。

---

# ACCESSIBILITY

檢查：

- contrast
- focus
- keyboard
- tooltip
- hit target
- selected state
- disabled state
- prefers-reduced-motion

AAA 要用實際 ratio 證明，不准口頭宣稱。

---

# GATE 2 FINAL REPORT

回報：

```text
DM UI PREVIEW READY

Original:
http://127.0.0.1:9422/

Preview:
http://127.0.0.1:9522/

Original source modified: NO

Approved design:
...

Preview files changed:
- ...

Visual parity with approved mockup:
- ...

Existing functionality:
- ...

Tests:
Build:
Playwright:
Accessibility:

Original dirty WIP preserved: YES
Original local source changed: NO
Docker changed: NO
Staging changed: NO
Production changed: NO
Committed to original: NO
Pushed: NO
```

最後輸出：

`PREVIEW_READY_WAITING_FOR_APPROVE_LOCAL_MERGE`

停止。

---

# SECOND APPROVAL GATE

只有使用者明確輸入：

`APPROVE_LOCAL_MERGE`

才允許規劃把 Preview 差異合回：

`F:\00-Ticenpi-SaaS\TicenpiDM`

在此之前：

**Original local tree 永遠 READ ONLY。**

即使 Preview 已完全 PASS，也不能自行 merge。

---

# GATE 3 — SAFE LOCAL MERGE

收到 `APPROVE_LOCAL_MERGE` 後：

1. 重新檢查 original git status。
2. 確認 original 自 Preview 建立後是否又有新 WIP。
3. 逐檔比對。
4. 不可整 repo 覆蓋。
5. 不可 reset。
6. 不可 replace whole working tree。
7. 只帶入使用者核准的 UI 差異。
8. 若 original 同檔案已被其他 writer 修改，HARD STOP，先做 collision report。
9. merge 後重新跑 tests/build/Playwright。
10. 不 commit / push，除非使用者另外明確要求。

---

# FINAL PRODUCT TEST

只有以下問題能回答 YES，才可宣告設計成功：

`一個完全沒看過 DM 的房仲業務，第一次打開後，能不能在 3 分鐘內理解怎麼換圖、改字、換業務照片、調整基本版面，並做出一份看起來已經能交付的正反面？`

如果答案不是 YES，不要宣告完成。

---

# HARD STOP CONDITIONS

立即停止並回報：

1. 找不到 ui-ux-pro-max skill。
2. 3013 reference route 不正確。
3. preview 無法與 current dirty WIP 建立 baseline parity。
4. 必須修改 original 才能做 preview。
5. preview 會覆寫原本 local data。
6. 需要 backend/schema migration。
7. 需要 Docker / Staging / Production。
8. existing drafts 會被覆蓋。
9. autosave / rollback / F5 restore 會退化。
10. active writer 和 preview 要改同一 original file，且 merge 階段有 collision。
11. 必須靠猜測 current UI 才能設計。

---

# FINAL RULE

這個任務的核心不是「讓 Claude 自由發揮」。

而是：

```text
先看懂現在
↓
讀完整 UI/UX Pro Max
↓
比較正確 reference
↓
先出圖
↓
使用者確認
↓
在獨立 9522 Preview 實作
↓
原 9422 完全不動
↓
再次確認
↓
最後才允許合回本機
```

任何階段不得跳過 approval gate。
