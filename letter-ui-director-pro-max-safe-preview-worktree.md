# LETTER UI DIRECTOR PRO MAX — SAFE VISUAL-FIRST REDESIGN / THREE-FOLD LETTER / DM REFERENCE READ ONLY

## ROLE

你現在不是一般 coding agent。

你要同時扮演：

- Senior Product Designer
- UX Director
- Interaction Designer
- Design-System Architect
- Print Layout Designer
- Accessibility Reviewer
- Frontend Architecture Reviewer
- Usability Tester
- Visual QA Engineer

你的任務不是「稍微美化 Letter」。

你的任務是重新整理 TicenpiLetter 的 **開發信 / 三折信 Editor**，讓它：

- 一打開就像完成度很高的房仲開發信
- 正面仍維持目前成熟的正文設計方向
- 反面有清楚、固定、可切換的案件展示版型
- 反面下方信封資訊區的位置與占版固定，不因案件內容而漂移
- 三折後的資訊位置合理、可印刷、可郵寄
- 案件資料、品牌資料、業務資料有合理預設內容
- 大頭貼、LOGO、LINE QR、圖片與案件操作方式參考 DM 的成熟互動
- 但整體功能比 DM 更聚焦、更簡單
- 不做成一個過度自由、過度複雜的通用設計器
- 不破壞目前 Letter 已有功能
- 使用者看到並核准實際視覺方案以前，不得修改原本 TicenpiLetter working tree

---

# TARGET PROJECT — THE ONLY WRITABLE PRODUCT

目前真實 Letter 專案：

`F:\00-Ticenpi-SaaS\TicenpiLetter`

## CURRENT LETTER LOCAL UI — READ ONLY UNTIL APPROVAL

主入口：

`http://localhost:3013/`

Letter Editor：

`http://localhost:3013/letter`

IMPORTANT:

- 本任務的產品目標只有 **TicenpiLetter**
- `F:\00-Ticenpi-SaaS\TicenpiLetter` 是唯一最終允許被修改的產品 repo
- 在 Preview approval 以前，原始 working tree 一律 READ ONLY
- 不要改 TicenpiDM
- 不要把 DM 當成 target project
- 不要把 `127.0.0.1:3013` 當成 Letter canonical origin
- 本任務 canonical Letter origin 是 `localhost:3013`

---

# REFERENCE PROJECT — READ ONLY

DM 只作為 **UI / interaction / property workflow 參考**：

`F:\00-Ticenpi-SaaS\TicenpiDM`

DM 必須全程 READ ONLY。

你可以閱讀 DM source、操作 DM 本機畫面、研究它的交互與資料流程。

但絕對禁止：

- 修改 TicenpiDM
- commit TicenpiDM
- 建檔到 TicenpiDM
- reset / restore / clean TicenpiDM
- 把 DM 整包複製進 Letter
- 把 DM 當成 implementation target
- 把 DM 的完整自由設計器複雜度搬到 Letter

如果任何 DM tracked / untracked source 被本任務修改：

`DM_REFERENCE_MUTATED = FAIL`

並立即停止。

---

# DM REFERENCE SCOPE

這次方向是：

```text
Letter
  ↓ 參考
DM
```

不是：

```text
DM
  ↓ 參考
Letter
```

因此不要再建立任何「OCR / Letter A4 預覽參考」或「OCR / Letter Sticker 互動參考」章節。

Letter 本身是 target，不是 reference。

DM 主要參考：

## 1. 案件擷取工作流

研究 DM 實際 active flow：

```text
貼案件 URL
↓
擷取
↓
資料正規化
↓
案件預覽
↓
選版型 / frame
↓
放入設計
```

必須找到 CURRENT active source path，不得憑記憶猜。

## 2. 案件卡 / frame

參考：

- 物件資料層級
- 圖片比例
- 案名
- 價格
- 坪數
- 格局
- 地址 / 區域
- supporting metadata
- card hierarchy
- frame selection

但 Letter 的卡片尺寸要依自己的 A4 / 三折版面重新設計，不得直接把 DM card 硬縮進去。

## 3. 大頭貼 / 圖片操作

參考 DM：

- direct select
- replace
- move
- resize
- rotate（只在合理情況）
- flip
- reset
- delete
- z-order（若 Letter 物件需要）
- contextual controls

重點：

- 不要醜框死
- 不要過度拘束
- 不要用很多固定外框造成呆板感
- 但仍必須 print-safe
- 預設擺位要合理
- 不要讓使用者一開始就需要自己排版

## 4. Text / Modal / Property control

參考 DM：

- 點擊 A4 直接選物件
- contextual edit
- centered modal
- toolbar grouping
- replace-first
- direct manipulation
- progressive disclosure

## 5. Placeholder / Default Content

參考 DM「打開就像完成品」的概念。

不要複製 DM 文案。

Letter 要做自己的房仲開發信預設內容。

---

# REQUIRED SKILL — MUST READ FIRST

第一個設計動作之前，必須完整讀取：

`.claude-skills/ui-ux-pro-max.md`

優先解析：

`F:\00-Ticenpi-SaaS\TicenpiLetter\.claude-skills\ui-ux-pro-max.md`

如果不存在：

1. 在目前 workspace 搜尋同名檔案
2. 可以讀取 DM repo 的同名 skill 作為設計 skill reference
3. 必須回報實際 resolved path
4. skill 仍然只是設計規範，不代表 DM 可寫

必須完整理解：

- design principles
- UX reasoning
- visual hierarchy
- color guidance
- typography guidance
- spacing
- layout
- interaction
- accessibility
- anti-patterns
- decision framework

禁止：

- 只讀摘要
- 用記憶代替檔案
- 假裝已讀
- 找不到時用一般 UI 常識冒充

完全找不到則：

`UI_UX_PRO_MAX_NOT_FOUND`

停止。

---

# PRODUCT DEFINITION — LETTER IS NOT DM

Letter 是：

**房仲開發信用的三折 A4 設計器**

不是：

- 通用海報設計器
- Canva clone
- DM 全功能複製版
- 任意自由畫布
- 大量 object composition 工具

主要使用者應該：

```text
看到完整開發信
↓
改正文
↓
換業務 / 品牌資料
↓
貼案件 URL
↓
挑一個反面案件版型
↓
挑一個信封樣式
↓
微調圖片 / 大頭貼
↓
列印 / PDF
```

目標是「完成得快」。

不是「自由度最大」。

---

# THREE-FOLD PRINT STRUCTURE — HARD PRODUCT CONSTRAINT

Letter 是三折開發信。

因此正反面不能當成一般自由 A4。

必須先讀目前 Letter 的：

- A4 front renderer
- A4 back renderer
- three-fold/fold assumptions
- print/PDF renderer
- envelope information area
- recipient/sender area
- printed-material marker
- existing layout presets
- current fixed regions
- autosave / persistence
- .ocrletter serializer

建立：

`LETTER_PRINT_STRUCTURE_MAP`

至少標明：

- 正面固定區
- 正面可編輯區
- 反面案件展示區
- 反面信封固定區
- 三折折線 / fold zones（若目前 source 有）
- printable safe area
- 不應被案件覆蓋的區域
- PDF / print 對應 source

---

# FIXED OCCUPANCY RULE — CRITICAL

以下區域的「位置與占版」不能因為 UI redesign 隨意變動。

## Front

正面主要正文區：

- 維持目前文字設計方向
- 占版範圍需穩定
- 不能因為新增案件樣式而任意縮成小框
- 不能改成 DM-style property canvas
- 保留目前 Letter 的文字主導特性

如果目前已經有經驗證的 `frontBelow.text` logical box / WYSIWYG contract：

- 必須先讀現行 source
- 保留已接受的文字 metrics / logical-box 行為
- 不可在視覺 redesign 中偷偷重寫 rich-text schema

## Back

反面下方信封資訊區：

- 位置固定
- 高度 / 占版固定
- 不因案件數量改變
- 不因模板切換改變三折位置
- 不允許案件侵入
- 不允許自由物件覆蓋
- 必須 print-safe / postal-safe

案件展示區只能使用「剩餘可用區域」。

---

# CORE BACK LAYOUT MODEL

反面主要案件區要像 DM 一樣「有設計好的版型可以選」。

但不要做成自由 Sticker canvas。

至少提出 4 種適合 Letter 的固定版型。

其中必須包含：

## Preset A — 2 × 3 Grid

```text
┌──────────────┬──────────────┐
│   案件 1     │   案件 2     │
├──────────────┼──────────────┤
│   案件 3     │   案件 4     │
├──────────────┼──────────────┤
│   案件 5     │   案件 6     │
└──────────────┴──────────────┘
[ 固定信封資訊區 ]
```

這是主要候選 / 預設方案。

## Preset B — 1 Hero + 2 Support

一個主案件 + 兩個次案件。

## Preset C — 2 × 2 Balanced

4 案件，卡片更大，資訊更完整。

## Preset D — Editorial Mixed

不對稱但仍固定 cell system。

例如：

- 左大右兩小
- 上大下兩小
- 一主兩輔

IMPORTANT:

即使是 mixed layout，也必須是 preset-based。

不是讓案件自由 x/y 飄。

---

# PROPERTY OBJECT RULE

案件卡不是一般 Sticker。

案件卡應該屬於：

`Property Layout System`

而不是：

`Free Sticker System`

案件操作優先：

- 加入指定 slot
- replace
- change frame
- move to another slot
- swap slots
- delete
- select property
- select frame

不要預設允許：

- 任意旋轉
- 任意飄出 cell
- 疊在其他案件上
- 覆蓋信封區

---

# PROPERTY WORKFLOW — DM REFERENCE, LETTER IMPLEMENTATION

Letter 案件流程目標：

```text
案件物件
↓
貼 URL
↓
擷取
↓
案件預覽
↓
選案件 card/frame
↓
選反面 layout preset
↓
選 slot
↓
加入
```

如果 Letter 目前已有 V5/V7 兼容 DM 的 extraction：

- 優先沿用
- 不重造 scraper
- 不重造 normalization
- 不新增 backend provider

只有在證明 Letter 現有 extraction path 壞掉，而且修正屬於必要 integration regression 時，才提出最小修正。

未核准前不要改 backend。

---

# FRONT DESIGN — PRESERVE, POLISH, DO NOT REINVENT

正面仍然是「文字開發信」主導。

需要做：

- 整理 spacing
- typography hierarchy
- alignment
- section rhythm
- image/portrait integration
- logo/QR/brand polish
- paper feel
- direct editing clarity

不要做：

- 把正文切成很多 dashboard cards
- 讓案件 Grid 吃掉正文
- 把 Letter 變成 DM
- 大改目前已經接受的正文 layout contract

正面應該看起來：

- 像專業開發信
- 有人情味
- 有品牌感
- 可列印
- 不是 SaaS dashboard
- 不是海報

---

# BACK ENVELOPE AREA — FIXED BUT VISUALLY UPGRADED

目前反面信封區白底、單調。

此次要設計多個可切換的 **Envelope Visual Presets**。

位置 / 占版不變。

只改視覺與內部 hierarchy。

至少提出 4 套：

## Envelope A — Clean Professional
- 高可讀性
- restrained lines
- 低裝飾
- 正式

## Envelope B — Warm Editorial
- 柔和品牌色
- 輕量區塊
- 更有人味

## Envelope C — Premium Minimal
- 極簡
- 精緻 typography
- 細線 / subtle accent

## Envelope D — Brand Accent
- 品牌色帶 / small motif
- 保持郵寄資訊可讀
- 不影響地址辨識

禁止：

- 大面積深色造成列印浪費
- 太花導致地址難讀
- heavy gradient
- neon
- postal information contrast 不足

---

# PRINTED-MATERIAL MARK — MUST REDESIGN

目前「印刷品」標記視覺不接受。

不要沿用現在的圖案。

需要重新設計至少 4 個可切換樣式：

## Style A — Modern Outline
## Style B — Postal Stamp
## Style C — Minimal Capsule
## Style D — Editorial Label

要求：

- 印刷清楚
- 黑白也可辨識
- 不俗氣
- 不過度卡通
- 尺寸可控
- 和 envelope preset 協調
- 可由使用者切換

不要只是換 icon。

要把：

- outline
- typography
- spacing
- shape
- weight

一起設計。

---

# DEFAULT CONTENT — TEMPLATE-FIRST

新 / empty draft 不應該看起來像空白工程畫面。

但 existing draft 不能被覆蓋。

只有真正 empty / new draft 可以顯示預設內容。

預設內容要和 DM 的「完成品預覽」概念一致，但文案要屬於 Letter。

至少包含：

## Brand

- 品牌名稱
- 公司名稱
- 品牌短句 / slogan

## Agent

- 業務姓名
- 職稱
- 手機
- LINE / QR 提示
- 服務區域

## Front Letter

- 稱謂
- 開場
- 主要開發信正文
- 專業但不生硬的說明
- CTA
- 祝福語
- 簽名 / 業務資訊

## Envelope

- 寄件人姓名
- 寄件地址
- 郵遞區號
- 收件人 placeholder
- 貴住戶 / 姓氏模式預覽

## Property

- 案件標題
- 價格
- 坪數
- 格局
- 區域 / 地址
- 簡短特色
- 圖片 placeholder

預設內容目的：

- 看版型
- 看 typography
- 看 spacing
- 看完成品

不是把 sample data 存成使用者真實資料。

---

# PLACEHOLDER PRINCIPLE

Placeholder 不能只是醜灰框。

應該：

- 看起來接近真實完成品
- 有合理 sample content
- hover / selected 才出現替換提示
- 平常不滿版教學字
- replacement affordance 明確

例如：

- 更換大頭照
- 替換 Logo
- 設定 LINE QR
- 擷取案件
- 更換主圖

---

# PORTRAIT / IMAGE UX — REFERENCE DM

研究 DM 的 Business Portrait / image interaction。

Letter 要吸收「好用的部分」，但要適合開發信。

目標：

- 預設位置合理
- 比目前自然
- 不用醜邊框硬限制
- 選取後才顯示控制
- replace-first
- resize / move 容易
- reset 回合理預設位置
- 保持比例
- 不會跑出 print-safe area

如果 Letter 已有：

- move
- resize
- rotate
- flip
- z-order
- reset
- delete
- F5 persistence

優先沿用現有 state / persistence。

本任務不要為了視覺重寫 transform engine。

---

# INFORMATION ARCHITECTURE

Letter 工具應比 DM 少。

先 Audit 目前 Icon Rail / Modal / Panel。

再依實際功能重新整理層級。

原則：

## High-frequency

- 正文
- 案件
- 圖片 / 大頭貼
- 收件人

直接可見 / 直接點擊。

## Medium-frequency

- 版型
- 信封樣式
- 印刷品樣式
- 品牌資料

Drawer / Modal / contextual panel。

## Low-frequency

- advanced
- unusual print options

放 More / Advanced。

不要把所有設定永久攤滿畫面。

---

# VISUAL DIRECTION — MUST PRODUCE 3

依 `ui-ux-pro-max.md` 和 Letter 三折產品定位，至少提出三套方向。

## A — Premium Japanese Letter

- 精準留白
- 細線
- restrained shadow
- soft neutral palette
- typography-first
- 溫度 + 專業

## B — Refined Real Estate Editorial

- 房仲案件 hierarchy 清楚
- 反面像小型 property editorial
- 正面仍是正式書信
- brand accent
- premium commercial

## C — Modern Personal Brand

- 業務個人品牌更突出
- 大頭貼 / QR / CTA 融入自然
- 信封區有品牌感
- 仍保持印刷 / 郵寄可讀性

禁止：

- generic purple AI SaaS
- giant dashboard cards
- every-panel glass
- rainbow gradient
- neon
- heavy blur
- excessive rounded cards
- heavy shadows
- overly decorative motion

---

# COLOR SYSTEM

重建 Letter-specific tokens。

至少列：

- workspace-bg
- paper
- paper-edge
- surface-1
- surface-2
- elevated
- border-subtle
- border-strong
- text-primary
- text-secondary
- text-muted
- accent
- accent-hover
- accent-active
- postal-safe-text
- success
- warning
- danger
- focus-ring
- overlay

提供 actual：

- HEX / OKLCH / RGB
- usage
- contrast pairing
- contrast ratio where relevant

正文与郵寄資訊要優先高可讀性。

---

# TYPOGRAPHY SYSTEM

必須產出 actual values：

- family
- size
- weight
- line-height
- letter-spacing
- usage

至少涵蓋：

- product label
- panel title
- body
- label
- helper
- button
- tooltip
- A4 front body
- A4 property title
- A4 property metadata
- envelope sender
- envelope recipient
- printed-material label

不要隨機混用一堆 13 / 14 / 15 / 16px。

---

# SPACING SYSTEM

建立一致 spacing tokens。

至少：

- rail padding
- modal padding
- panel padding
- button gap
- A4 front/back gap
- property-cell gap
- envelope internal spacing
- section spacing
- text modal spacing
- object control spacing

---

# BUTTON / CONTROL SYSTEM

重新整理：

- Primary
- Secondary
- Tertiary
- Icon
- Contextual
- Destructive
- Toggle / segmented
- Preset card

Phase 1 列 actual：

- height
- min-width
- icon size
- radius
- padding
- label style
- hover
- active
- selected
- focus
- disabled
- destructive

Icon-only 必須 tooltip。

---

# MICRO-INTERACTION

Vue 專案。

優先：

- CSS transition
- CSS transform
- Vue Transition
- 現有 dependency

不要亂加 React/Framer Motion。

設計：

- hover
- press
- tooltip
- modal
- selected object
- preset selection
- property slot selection
- envelope preset change
- printed-material style change

動效必須提升理解，不得拖慢工作。

---

# ORIGINAL WORKING TREE PROTECTION

在收到：

`APPROVE_LETTER_LOCAL_MERGE`

之前：

`F:\00-Ticenpi-SaaS\TicenpiLetter`

原始 working tree 一律 READ ONLY。

禁止：

- 直接改 source
- 直接改 CSS
- 直接改 Vue / JS / TS
- git reset
- git restore
- git checkout .
- git clean
- stash apply
- stash pop
- branch switch
- overwrite
- whole-file replacement
- repo-wide formatter
- dependency upgrade
- lockfile rewrite
- commit original
- push

不得以：

- quick fix
- prototype
- temporary patch
- cleanup
- formatter

為理由越過 Gate。

---

# CURRENT DIRTY WIP MUST BE PRESERVED

開始時唯讀回報 Letter：

- repo root
- branch
- HEAD
- `git status --short`
- tracked modified files
- untracked files
- active worktrees
- relevant Letter UI source files
- dev server status
- current URL
- current visible UI screenshot

不要求使用者先 commit 才能做 Audit / proposal。

---

# CORE SAFETY GATES

整個任務分四個 Gate。

```text
GATE 0 — Audit
READ ONLY
Letter + DM reference inspection

GATE 1 — Visual Proposal
READ ONLY
只出設計方案 / prototype / screenshot

使用者輸入：
APPROVE_LETTER_UI_IMPLEMENTATION

↓

GATE 2 — Preview Implementation
只改獨立 Letter preview worktree
原始 Letter 3013 source 仍 READ ONLY

Original:
http://localhost:3013/letter

Preview:
http://localhost:3113/letter

使用者輸入：
APPROVE_LETTER_LOCAL_MERGE

↓

GATE 3 — Safe Merge
才允許把核准差異合回原始 TicenpiLetter
```

沒有 exact approval token，不得越級。

---

# GATE 0 — LETTER UX AUDIT — READ ONLY

實際操作：

`http://localhost:3013/letter`

至少測：

- overall shell
- Icon Rail
- 正面 A4
- 反面 A4
- front text
- text modal
- recipient
- image materials
- portrait
- LOGO
- LINE QR
- property flow
- property frames
- envelope info
- printed-material mark
- layout presets
- 開發信條
- save/import
- print/PDF
- fullscreen
- F5 restore

對每個功能記錄：

- 用途
- 使用頻率
- 現在幾步
- 是否容易理解
- 是否值得永久佔位
- 是否可 contextualize
- 是否可直接在 A4 操作
- 是否和其他功能重複
- 是否適合三折信
- 是否會破壞固定 print region

建立：

`CURRENT_LETTER_UI_AUDIT`

---

# SOURCE AUDIT — LETTER READ ONLY

至少找：

- LetterView
- front renderer
- back renderer
- print/PDF renderer
- property renderer
- property extraction/client
- text modal
- image components
- portrait state
- LOGO / LINE QR
- selectedObject
- sticker/transform state
- property layouts
- envelope info
- printed-material mark
- theme presets
- default content
- IndexedDB autosave
- hydrated / dirty gate
- F5 restore
- .ocrletter serializer/import
- router
- relevant CSS

建立：

`LETTER_COMPONENT_MAP`

包含：

- visual element
- source file
- component
- state
- CSS selector
- current dimension
- current typography
- interaction owner
- persistence owner
- print impact

---

# DM SOURCE AUDIT — READ ONLY

不要做 DM 全面 audit。

只追本任務需要的 active path：

- property URL input
- scrape call
- normalized payload
- property preview
- property frame / templates
- image / portrait interaction
- direct object editing
- contextual controls
- default content / placeholder design

建立：

`DM_REFERENCE_MAP`

每項標記：

- DM source file
- active/not active
- 可借鑑什麼
- Letter 不應照搬什麼

---

# GATE 1 — VISUAL PROPOSAL ONLY

這一階段禁止修改原始 Letter source。

必須產生真正可看的視覺稿。

優先：

## Option 1 — Figma / design tool

如果可用：
- 建 desktop frames

## Option 2 — Static HTML Prototype

如果沒有 Figma：
- 在 repo 外 temp/sandbox 建 prototype
- 不 import/修改原始 Letter source
- browser render

## Option 3 — Rendered Mockup

根據 current Letter screenshot 做 visual mockup。

無論哪種都要輸出實際 screenshot。

---

# REQUIRED VISUAL PROPOSALS

至少產出：

1. Full Letter Editor — 正反 A4 同時顯示
2. 正面完整開發信
3. 反面 2×3 案件版型 + 固定信封區
4. 反面 1 Hero + 2 Support
5. 反面 2×2 Balanced
6. 反面 Editorial Mixed
7. Envelope Style A/B/C/D
8. Printed-material Style A/B/C/D
9. 案件 URL 擷取 → preview → frame → slot flow
10. 大頭貼 selected / replace / resize state
11. Empty/new draft with default content

三套整體視覺方向至少都要有：

- Full Editor
- Front
- Back

推薦方向再補完整其他狀態。

---

# GATE 1 OUTPUT

輸出：

## CURRENT LETTER UX PROBLEMS
## LETTER PRINT STRUCTURE MAP
## DM REFERENCE MAP
## THREE VISUAL DIRECTIONS
## RECOMMENDED DIRECTION
## FRONT DESIGN
## BACK PROPERTY PRESETS
## FIXED ENVELOPE REGION
## ENVELOPE VISUAL PRESETS
## PRINTED-MATERIAL PRESETS
## DEFAULT CONTENT
## PORTRAIT / IMAGE INTERACTION
## COLOR TOKENS
## TYPOGRAPHY
## SPACING
## BUTTON / CONTROL SPEC
## MOTION TOKENS
## INFORMATION ARCHITECTURE
## IMPLEMENTATION IMPACT
## MOCKUPS / SCREENSHOTS

最後輸出：

`PHASE_1_COMPLETE_WAITING_FOR_APPROVE_LETTER_UI_IMPLEMENTATION`

停止。

---

# VISUAL APPROVAL GATE

只有使用者明確輸入：

`APPROVE_LETTER_UI_IMPLEMENTATION`

才進 Gate 2。

「這個不錯」
「用 B」
「可以」
「繼續」

都不視為 implementation approval。

---

# GATE 2 — CREATE ISOLATED LETTER PREVIEW

收到：

`APPROVE_LETTER_UI_IMPLEMENTATION`

後，仍然禁止修改原始 Letter working tree。

建立獨立 preview worktree。

建議：

`F:\00-Ticenpi-SaaS\TicenpiLetter_ui_preview_wt`

或：

`F:\00-Ticenpi-SaaS\.ui-preview\TicenpiLetter`

---

# PREVIEW MUST MATCH CURRENT LETTER BASELINE

如果原始 Letter 有 dirty WIP：

單純從 HEAD 建 worktree 可能不是目前 3013 畫面。

因此：

1. worktree from current HEAD
2. original 不修改
3. 唯讀取得 original tracked diff
4. 將必要 dirty tracked changes套到 preview
5. 將 Letter UI 必須的 untracked source 複製到 preview
6. 不複製：
   - .git
   - node_modules
   - dist
   - cache
   - secrets
   - runtime temp
7. 驗證 relevant source parity
8. baseline parity 完成後才開始 UI implementation

禁止為了建立 preview 對 original 使用：

- stash
- reset
- restore
- commit
- clean

---

# PREVIEW URL

Original：

`http://localhost:3013/letter`

Preview：

`http://localhost:3113/letter`

如果 3113 被占用：

選下一個安全未占用 port。

但必須：

- 回報 actual URL
- 不占用 3013
- 不停止原始 3013
- 不把 preview 啟動成 127.0.0.1 canonical URL

---

# PREVIEW DATA SAFETY

Preview 是不同 origin。

不要假設會共享 original browser storage。

使用獨立 browser context/profile。

如果 Preview 會寫同一份 backend/local DB：

- 只用 disposable/test draft
- 不操作使用者正式草稿
- 不清原本 IndexedDB
- 不清 localStorage/session
- 不改 schema

如果無法避免污染原本資料：

停止並回報：

`PREVIEW_DATA_ISOLATION_BLOCKED`

---

# GATE 2 IMPLEMENTATION PRIORITY

先做：

1. Editor shell / workspace polish
2. Front polish without rewriting front text contract
3. Back fixed property region
4. Property layout presets
5. Fixed envelope region visual presets
6. Printed-material presets
7. Default content
8. DM-inspired portrait/image controls
9. Property URL → preview → frame → slot interaction
10. accessibility / motion polish

---

# GATE 2 IMPLEMENTATION RULES

優先：

- reuse Letter current state
- reuse current text data model
- reuse existing autosave
- reuse existing property extraction
- reuse existing image persistence
- reuse existing object transform
- CSS/component refactor before state rewrite

禁止：

- backend schema migration
- DB migration
- new scraping provider
- Production changes
- Staging changes
- Docker deployment
- release changes
- unrelated refactor
- dependency mass upgrade
- state architecture rewrite just for visual polish

---

# MUST PRESERVE LETTER FUNCTIONALITY

至少保留：

- A4 direct text click
- text modal
- live preview
- Apply
- Cancel rollback
- rich-text behavior
- autosave hydrated/dirty safety
- front/back
- existing drafts
- F5 restore
- recipient
- LOGO
- LINE QR
- image/sticker behavior
- portrait
- property extraction
- save/import
- .ocrletter
- Letter Strip
- print/PDF
- fullscreen

如果 visual redesign 讓其中任何一項退化：

FAIL。

---

# RESPONSIVE / PRINT VALIDATION

本輪不是只看 screen。

必須同時檢查：

## Screen

- desktop editor
- modal
- preset switching
- selected state

## Print/PDF

- A4 physical structure
- front text region
- back property region
- fixed envelope region
- printed-material mark
- no overlap
- no clipping
- no property crossing envelope boundary
- no theme causing postal information unreadable

---

# GATE 2 FUNCTIONAL TESTS

Preview 必須跑：

- existing relevant unit tests
- component tests
- Vite build
- browser smoke / Playwright if available
- print/PDF smoke

不得刪測試換 PASS。

---

# ACCESSIBILITY

檢查：

- contrast
- keyboard
- focus
- tooltip
- hit targets
- selected state
- disabled state
- reduced motion
- postal information readability

不要虛構 AAA。

---

# GATE 2 FINAL REPORT

回報：

```text
LETTER UI PREVIEW READY

Target project:
F:\00-Ticenpi-SaaS\TicenpiLetter

Reference project:
F:\00-Ticenpi-SaaS\TicenpiDM
DM modified: NO

Original:
http://localhost:3013/letter

Preview:
http://localhost:3113/letter
(or actual preview port)

Original Letter source modified: NO

Approved design:
...

Preview files changed:
- ...

Front:
- ...

Back property presets:
- ...

Envelope fixed region:
- ...

Envelope visual presets:
- ...

Printed-material presets:
- ...

Default content:
- ...

DM-referenced interactions:
- ...

Visual parity:
- ...

Existing functionality:
- ...

Tests:
Build:
Browser:
Print/PDF:
Accessibility:

Original dirty WIP preserved: YES
Original local source changed: NO
DM source changed: NO
Docker changed: NO
Staging changed: NO
Production changed: NO
Committed to original: NO
Pushed: NO
```

最後：

`PREVIEW_READY_WAITING_FOR_APPROVE_LETTER_LOCAL_MERGE`

停止。

---

# SECOND APPROVAL GATE

只有使用者明確輸入：

`APPROVE_LETTER_LOCAL_MERGE`

才允許規劃把 Preview 差異合回：

`F:\00-Ticenpi-SaaS\TicenpiLetter`

在此之前：

Original Letter 永遠 READ ONLY。

---

# GATE 3 — SAFE LOCAL MERGE

收到 exact approval token 後：

1. 重新檢查 original Letter git status
2. 確認從 Preview 建立後是否出現新 WIP
3. 逐檔 diff
4. 不整 repo 覆蓋
5. 不 reset
6. 不 clean
7. 不 whole-tree replace
8. 只帶入已核准 UI 差異
9. 同檔有新的 external writer 變更 → HARD STOP + collision report
10. merge 後重新 tests/build/browser/print smoke
11. 確認 DM repo 仍完全 untouched

如果 merge 完成且驗收 PASS：

建立 LOCAL checkpoint commit。

建議 message：

`feat: refine OCR Letter three-fold editor UI`

DO NOT PUSH。

如果 merge 後 FAIL：

不要 commit。

---

# GIT POLICY AFTER PASS

之後：

- 已驗收 PASS 的 milestone → 自動建立 local checkpoint commit
- 下一個大型任務從 checkpoint 開始
- push 仍需使用者明確批准

不要再把多個已驗收大型改版疊在同一個 dirty tree。

---

# HARD STOP CONDITIONS

立即停止：

1. 找不到 ui-ux-pro-max
2. `http://localhost:3013/letter` 不是實際 Letter Editor
3. DM repo 必須被修改才能完成
4. Preview baseline 無法匹配目前 Letter dirty WIP
5. Preview 會污染原本正式本機草稿
6. 必須 DB/schema migration
7. 必須新 scraper/provider
8. 必須 Docker / Staging / Production
9. existing drafts 會被覆蓋
10. autosave / rollback / F5 restore 會退化
11. print/PDF 會破壞三折固定區域
12. property card 需要侵入固定 envelope region
13. 需要猜測 current Letter UI 才能設計
14. active writer 與 merge file collision 無法安全解決

---

# FINAL PRODUCT TEST

只有以下問題能回答 YES 才可宣告成功：

`一個第一次使用的房仲業務，能不能在不學設計軟體的情況下，快速改正文、換業務資料、貼案件網址、挑反面案件版型、挑信封風格，並得到一份看起來可直接列印寄出的三折開發信？`

如果不是 YES：

不要宣告完成。

---

# FINAL RULE

這次不是把 DM 搬進 Letter。

正確流程：

```text
看懂目前 Letter
↓
讀 UI/UX Pro Max
↓
只讀研究 DM 好用的互動與案件流程
↓
理解三折 print constraints
↓
先出 Letter 專用視覺方案
↓
使用者確認
↓
獨立 3113 Preview 實作
↓
3013 Original 完全不動
↓
再次確認
↓
最後才合回 TicenpiLetter
```

任何階段不得跳過 approval gate。
