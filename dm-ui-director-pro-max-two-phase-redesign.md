# DM UI DIRECTOR PRO MAX — CLAUDE TWO-PHASE UX REDESIGN

## ROLE

你現在不是一般 coding agent。

你要同時扮演：

- Senior Product Designer
- UX Director
- Interaction Designer
- Frontend Architecture Reviewer
- Design-System Auditor
- Accessibility Reviewer
- Usability Tester

你的任務不是「稍微美化目前頁面」。

你的任務是根據專案既有的 UI/UX 規範，全面重新審視 DM 編輯器，讓它變成：

- 第一次打開就看得懂
- 操作步驟盡量少
- A4 正反面一開始就像完成品
- 圖片 / 貼紙 / LOGO / QR / 大頭照操作一致
- 按鈕層級清楚
- 視覺不再呆板、不再有強烈 AI 模板味
- 有成熟 SaaS / 設計工具的質感
- 不破壞原本功能

---

# WORKING REPO

`F:\00-Ticenpi-SaaS\TicenpiDM`

## REQUIRED DESIGN SKILL

第一件事必須完整讀取：

`.claude-skills/ui-ux-pro-max.md`

如果檔案存在於 repo 或目前 Claude skill workspace，必須完整閱讀其：

- 設計規範
- UX 原則
- 推理流程
- 視覺規則
- 色彩規則
- typography 原則
- spacing 原則
- component / interaction guidance
- accessibility guidance
- anti-pattern
- decision framework

不得只讀摘要。
不得憑印象聲稱已遵守。
後續設計提案必須明確對應該 skill 的原則。

如果該檔案不存在或無法讀取，立即停止在 Phase 1 並回報：
`UI_UX_PRO_MAX_NOT_FOUND`

不要自行用一般知識取代。

---

# LOCAL REFERENCE

實際比較：

- DM:
  `http://localhost:9422`

- OCR / Letter:
  `http://localhost:3013/專案`
  `http://localhost:3013/專案設計`

- Letter source:
  `F:\00-Ticenpi-SaaS\TicenpiLetter`

Reference project 只能唯讀。

---

# TOOLS

如果可用，優先使用：

1. Playwright MCP / Browser automation
2. Claude browser / computer use
3. filesystem / repo inspection
4. screenshot / vision
5. Figma MCP
6. local dev server

不要只看 source 就判斷 UX。

必須實際操作 localhost：

- hover
- click
- drawer
- modal
- image manipulation
- text editing
- front/back switch
- layout controls
- restore behavior

---

# NON-NEGOTIABLE TWO-PHASE WORKFLOW

## PHASE 1
只做：

- UX audit
- visual direction
- information architecture
- design system proposal
- layout proposal
- template proposal
- button hierarchy proposal
- interaction proposal

**禁止修改任何程式碼。**

Phase 1 完成後必須停止，等待使用者確認。

只有使用者明確說：

- 可以
- 照這版做
- 開始實作
- implement
- 進 Phase 2

才可以進入 Phase 2。

## PHASE 2
使用者確認後才：

- 修改 code
- refactor CSS
- 調整 component
- 補 interaction
- 補 animation
- 調整 default template
- 跑 tests / build / Playwright

---

# SAFETY / REPO RULES

目前 repo 有 dirty WIP。

嚴禁：

- git reset
- git reset --hard
- git restore .
- git checkout .
- git clean
- branch switch
- stash pop/apply
- 覆蓋既有 WIP
- 修改 release worktree
- 修改 Docker / VPS / Staging / Production

開始前先回報：

- Repo
- Branch
- HEAD
- git status
- dirty files
- active worktrees
- current frontend entry
- current UI files

本任務只允許修改：

`F:\00-Ticenpi-SaaS\TicenpiDM`

Reference repo 只能讀。

---

# PRIMARY PROBLEM TO SOLVE

目前 DM 的問題不是「缺功能」而已。

核心問題：

1. 整體介面呆板
2. 按鈕層級不夠直覺
3. 有明顯 AI-generated dashboard 味
4. A4 正反面太像普通白紙
5. 缺少紙張層次 / shadow / workspace depth
6. 新使用者看到太多空白
7. 沒有完整預設版型
8. 不知道每一區應該放什麼
9. 要自己從零設計成本太高
10. 圖片 / Logo / QR / Portrait / Sticker 操作模型不一致
11. 功能存在，但不是「引導式產品體驗」
12. 畫面資訊層級與 spacing 缺乏節奏

---

# DESIGN GOAL

目標不是「炫」。

目標是：

```text
成熟
高質感
清楚
有層次
不費腦
低學習成本
高可預測性
```

使用者第一次打開，應該能理解：

- 哪裡換圖片
- 哪裡改文字
- 哪裡放業務照
- 哪裡放 QR
- 哪裡調版型
- 正面與反面各自負責什麼

---

# VISUAL DIRECTION EXPLORATION

不要只給一種。

根據 UI/UX Pro Max 規則，至少提出 3 個方向。

可以參考但不限於：

## Direction A — Premium Japanese Minimal
特色：
- 大留白
- 明確 grid
- typography hierarchy 強
- shadow 很克制
- 中性色為主
- 微量 accent
- 高級而安靜

## Direction B — Refined Bento Workspace
特色：
- Bento Grid
- 功能入口分層清楚
- panel 卡片化但不過度
- contextual tools
- 高資訊密度但不擁擠

## Direction C — Glass / Layered Creative Tool
特色：
- subtle glassmorphism
- translucent floating controls
- layered workspace
- A4 紙張浮層
- micro interaction 明顯但克制

禁止：

- 大量漸層
- 過度 neon
- 一堆圓角大卡片
- 紫色 AI SaaS 模板
- blur 過重
- shadow 過重
- 每個東西都 glass
- 只為了漂亮犧牲辨識度

---

# COLOR SYSTEM

依 `.claude-skills/ui-ux-pro-max.md` 的規則重新設計 Color Tokens。

至少產出：

- canvas background
- panel background
- elevated background
- paper background
- border subtle
- border strong
- text primary
- text secondary
- text muted
- accent primary
- accent hover
- accent active
- success
- warning
- danger
- focus ring
- overlay

要求：

- 嚴格檢查對比度
- 主要文字 / 控制文字盡可能符合 WCAG AAA
- 如果 AAA 與品牌 / UI 情境衝突，必須在 Phase 1 明確說明哪一項無法達到以及原因
- 不可只說「AAA compliant」而沒有實際 contrast ratio / token pairing 檢查

---

# TYPOGRAPHY SYSTEM

重新盤點：

- page / product title
- section title
- panel title
- body
- label
- helper
- metadata
- button
- tooltip
- A4 object defaults

建立明確 typography scale。

例如需要產出：

- font family
- font size
- weight
- line-height
- letter-spacing
- usage

不要讓：

- 13px / 14px / 15px / 16px 到處亂用
- title hierarchy 不明
- button / label 混在一起

---

# SPACING SYSTEM

建立統一 spacing scale。

例如可以用：
- 4
- 8
- 12
- 16
- 24
- 32
- 48

但實際值要依 UI/UX Pro Max 和目前介面判斷。

所有：
- padding
- gap
- panel spacing
- icon spacing
- drawer spacing
- modal spacing
- A4 gap

都應該收斂成一致節奏。

---

# BUTTON / CONTROL SYSTEM

重新檢查所有 controls。

每個 control 分類：

- Primary
- Secondary
- Tertiary
- Icon
- Destructive
- Contextual

要求：

- 按鈕高度一致
- icon size 一致
- hover state 清楚
- active state 清楚
- focus state 清楚
- disabled state 清楚
- hit target 足夠
- icon-only 必須 tooltip

不要保留歷史上隨機產生的尺寸。

---

# MICRO-INTERACTION / MOTION

加入高質感但克制的微互動。

優先使用：

- CSS transition
- CSS transform
- Vue transition

只有現有架構已經有適合的 motion dependency 時才考慮額外 framework。

不要為了動畫硬加 Framer Motion，因為目前是 Vue 專案。

Vue 專案應優先：

- CSS transitions
- Vue Transition
- existing motion library if already installed

Motion 使用場景：

- icon hover
- drawer open/close
- tooltip
- modal entrance
- selected object
- card hover
- A4 focus
- contextual action appearance
- sticker selection handles
- button press

要求：

- easing 一致
- duration 一致
- 不延遲操作
- 不做浮誇 entrance
- 支援 prefers-reduced-motion

Phase 1 必須提出：
- duration tokens
- easing tokens
- 哪些 element 有 motion
- 哪些 element 不應該動

---

# INFORMATION ARCHITECTURE

重新審視左側工具。

不要因為現在有很多功能，就保留很多一級 icon。

應該依使用者 mental model 分類。

可能方向：

```text
Cases
Assets
Add
Layout
Style
Data
Settings
```

但這只是參考。

你要根據實際功能重新決定。

所有一級入口必須回答：

- 為什麼它值得常駐？
- 使用頻率？
- 可不可以 merge？
- 可不可以改 contextual？
- 是否跟 A4 直接操作重複？

---

# A4 WORKSPACE

A4 是主角。

要求：

- 無 Top Bar
- workspace 100dvh
- 不要外層 vertical scroll
- 左 rail compact
- 正反 A4 最大化
- 紙張有層次
- workspace 有深度
- front/back 容易辨識

參考 OCR / Letter 的 paper treatment：

- subtle shadow
- paper border
- layered depth
- proper spacing

但不要直接複製。

---

# TEMPLATE-FIRST EXPERIENCE

這是最重要的 UX 改動。

新使用者 / 新 draft：

**不能再看到空白 A4。**

必須直接載入一套完整 front/back template。

這個 template 應該像已完成 70% 的作品。

包含：

- sample hero image
- editable title
- editable subtitle
- sample property / card
- logo placeholder
- business portrait placeholder
- QR placeholder
- CTA
- supporting content

目的：

使用者不是「從零設計」。

而是：

```text
看懂
↓
替換
↓
微調
↓
完成
```

---

# PLACEHOLDER UX

Placeholder 不只是灰框。

應該看起來像真實成品，但可明確替換。

例如：

- 點擊替換主圖
- 點擊修改標題
- 上傳業務照片
- 替換 LOGO
- 設定 LINE QR

hover 才顯示 replacement hint。

正常狀態不要滿畫面都是說明文字。

---

# TEMPLATE DIRECTIONS

Phase 1 至少設計三套 front/back。

## A. Editorial / Magazine
- hero image
- strong type
- asymmetric rhythm
- premium visual

## B. Real Estate Commercial
- listing-oriented
- clear property hierarchy
- business contact
- CTA / QR
- practical

## C. Personal Brand / Social
- portrait
- sticker-friendly
- more expressive
- personal sales identity

不要固定 2×3 六格。

可使用：
- 1 hero + supporting cards
- 2×2
- mixed size
- 4-column grid
- asymmetrical composition

現有 layout system 若限制 max 4 columns / 12 rows，必須尊重。

---

# IMAGE / STICKER MENTAL MODEL

從使用者角度：

- Portrait
- Logo
- LINE QR
- Uploaded image
- Sticker

都屬於：

`Asset / Image Object`

優先統一互動。

使用者只需要學一套：

- select
- drag
- resize
- rotate
- flip
- front/back z-order
- reset
- delete

不要大頭照一套、Logo 一套、QR 一套。

如果技術 state 不同，可以在底層不同，但 UX 要一致。

---

# TEXT EDITING

保留：

- A4 文字直接點擊
- centered modal
- live preview
- Apply
- Cancel rollback

Phase 1 要重新評估 modal toolbar：

- 是否資訊過多
- 是否單排更好
- 低頻設定是否收起
- formatting 是否 contextual

不要回到大型永久右側 inspector。

---

# DIRECT MANIPULATION FIRST

任何基本物件操作優先：

`directly on A4`

例如：

## Image
點 → handles / quick actions

## Text
點 → edit modal

## Placeholder
點 → replace

## Layout
只有 page-level / grid-level 才進 layout panel

不要讓使用者為了移一張圖先找 sidebar。

---

# UX AUDIT METRICS

Phase 1 要記錄目前常見任務的操作步數。

至少：

- 換主圖
- 改標題
- 換業務照片
- 替換 Logo
- 設定 LINE QR
- 移動圖片
- resize
- rotate
- 調整正面 layout
- 調整反面 layout

建立：

`BEFORE_CLICK_COUNT`

然後提出：

`TARGET_CLICK_COUNT`

常用操作目標：
盡量 <= 2 interactions。

---

# PHASE 1 DELIVERABLE — STOP AFTER THIS

Phase 1 只輸出方案，不改 code。

必須包含：

## 1. CURRENT UX PROBLEMS
逐項說明目前問題。

## 2. UI/UX PRO MAX PRINCIPLES APPLIED
列出這次真正採用了 `.claude-skills/ui-ux-pro-max.md` 的哪些原則。

不要空泛。

## 3. THREE VISUAL DIRECTIONS
每個包含：

- mood
- color
- typography
- spacing
- panel style
- A4 treatment
- interaction feel
- advantages
- disadvantages

## 4. RECOMMENDED DIRECTION
選一個最適合 DM 的主方向，說明原因。

## 5. COLOR TOKENS
列 actual color values。

## 6. TYPOGRAPHY SCALE
列 actual sizes / weights / line-height。

## 7. SPACING SCALE
列 actual spacing tokens。

## 8. BUTTON / CONTROL SPEC
列尺寸、radius、icon、state。

## 9. MOTION TOKENS
列 duration / easing / use case。

## 10. NEW INFORMATION ARCHITECTURE
左 rail / drawer / modal / contextual control。

## 11. FRONT TEMPLATE
完整結構。

## 12. BACK TEMPLATE
完整結構。

## 13. OTHER TEMPLATE OPTIONS
另外兩套。

## 14. IMAGE / STICKER INTERACTION
完整 mental model。

## 15. BEFORE / AFTER FLOW
包含 click count。

## 16. IMPLEMENTATION IMPACT
列出預計需要改哪些 source files，但不要修改。

## 17. SCREEN / MOCKUP
如果可用：
- Figma
- screenshot mockup
- HTML prototype

至少提供一種可視化 proposal。

完成後明確輸出：

`PHASE_1_COMPLETE_WAITING_FOR_USER_APPROVAL`

然後停止。

---

# PHASE 2 — ONLY AFTER APPROVAL

只有使用者確認後才執行。

實作要求：

- 優先 reuse 現有 component / state
- 不做 backend migration
- 不做 schema rewrite
- existing drafts 不得被 default template 覆蓋
- new defaults 只套 new/empty draft
- autosave 不得退化
- F5 restore 不得退化
- text modal rollback 不得退化
- image transforms 不得退化

---

# PHASE 2 VALIDATION

實作後：

## Unit / component
跑現有 tests。

## Build
Vite build PASS。

## Playwright
重新驗證 localhost。

## UX
確認：

- first-time user understands layout
- front/back look complete
- main image replace
- title edit
- portrait replace
- Logo
- QR
- sticker manipulation

## Accessibility
檢查：

- contrast
- focus
- hit targets
- tooltip
- keyboard
- reduced motion

---

# FINAL PRODUCT TEST

只有以下問題回答 YES 才能宣告 PASS：

`一個完全沒看過 DM 的房仲業務，第一次打開後，能不能在 3 分鐘內理解怎麼換圖、改字、換業務照片、調整基本版面，並做出一份看起來已經能交付的正反面？`

如果不是 YES，繼續改善。

---

# PHASE 2 FINAL REPORT

```text
DM UI PRO MAX IMPLEMENTATION

Repo:
Branch:
HEAD:

Approved visual direction:

Files changed:
- ...

Design system:
- Color:
- Typography:
- Spacing:
- Buttons:
- Motion:

Workspace:
- ...

Icon rail:
- ...

A4:
- ...

Front default:
- ...

Back default:
- ...

Image / Sticker:
- ...

Text editing:
- ...

Before / after click count:
- ...

Accessibility:
- ...

Tests:
Build:
Playwright:

Existing dirty WIP preserved: YES
Docker changed: NO
Staging changed: NO
Production changed: NO
Committed: NO
Pushed: NO
```
