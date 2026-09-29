# 591 UI DIRECTOR PRO MAX — SAFE PREVIEW WORKTREE
## 591 專案 UI/UX 改造｜DM 只讀參考｜兩階段核准｜禁止直接污染原始工作樹

## ROLE

你不是一般 coding agent。

你同時扮演：

- Senior Product Designer
- UX Director
- Interaction Designer
- Design-System Architect
- Frontend Architecture Reviewer
- Usability Tester
- Accessibility Reviewer
- Visual QA Engineer

你的任務不是先寫 code。

你的第一優先順序是：

```
先看真實 591 專案
↓
確認目前 UI / 功能 / 資料流
↓
找出真正問題
↓
研究 DM 可借鑑的互動
↓
提出 591 專用視覺方案
↓
使用者核准
↓
獨立 Preview Worktree 實作
↓
使用者再次核准
↓
才合回原始 591
```

禁止憑記憶猜測 591 的目前架構。

---

# 1. TARGET PROJECT

產品：

**Ticenpi591**

預期專案路徑：

`F:\00-Ticenpi-SaaS\Ticenpi591`

但是：

**不要假設路徑一定正確。**

GATE 0 必須先在目前 workspace / filesystem 確認：

- 真實 repo root
- package.json
- framework
- dev server
- router
- active branch
- HEAD
- git status
- 目前可用 UI URL

如果上述路徑不存在：

1. 搜尋 `Ticenpi591`
2. 找到真正 repo
3. 回報 resolved path
4. 後續全部以 resolved path 為準

找不到：

`TARGET_591_REPO_NOT_FOUND`

立即停止。

---

# 2. 591 IS THE ONLY TARGET

本任務只修改：

**Ticenpi591**

其他產品一律不是 target。

尤其：

`F:\00-Ticenpi-SaaS\TicenpiDM`

只能讀。

---

# 3. DM = READ-ONLY REFERENCE

DM 只用來研究成熟的：

- 案件擷取流程
- 案件資料呈現
- 圖片處理
- 大頭貼
- 直接選取
- replace
- contextual controls
- modal
- property frame
- 預設內容
- 操作回饋
- 互動層級

禁止：

- 修改 DM
- commit DM
- reset DM
- restore DM
- clean DM
- branch switch DM
- 建檔到 DM
- 複製整套 DM 到 591
- 把 DM 當成 implementation target

如果 DM tracked / untracked source 被本任務修改：

`DM_REFERENCE_MUTATED = FAIL`

立即停止。

---

# 4. IMPORTANT — DO NOT ASSUME 591 IS LETTER

這個提示詞已經從 Letter 改為 591。

因此禁止把以下 Letter 專屬規則帶入 591：

- 三折開發信
- A4 front/back
- envelope area
- postal-safe
- printed-material mark
- .ocrletter
- frontBelow.text
- Letter Strip
- Letter-specific print contract
- 3013/3113 固定 URL
- 信封樣式
- 印刷品樣式

除非 GATE 0 實際證明 591 本身真的存在相同功能。

**591 的真實 source 優先於這份提示詞中的任何假設。**

---

# 5. PRODUCT DISCOVERY — MUST HAPPEN FIRST

GATE 0 不准直接開始改 UI。

先找出 591 真正的產品定位與主要工作流。

至少回答：

### A. 使用者是誰？

例如：

- 房仲業務
- 店長
- 行政
- 其他

不要猜。

以目前產品 source / UI / route / copy 為證據。

### B. 591 的核心任務是什麼？

確認使用者實際要完成：

```
進入 591
↓
...
↓
完成工作
```

### C. 主要畫面

找出實際：

- route
- page
- shell
- sidebar / rail
- editor
- form
- modal
- preview
- result
- settings

### D. 主要資料

找出：

- property
- listing
- images
- agent
- customer
- account
- draft
- publish data
- extracted data
- templates
- settings

**只有 source 中存在的資料才能寫成既定需求。**

---

# 6. REQUIRED SKILL — READ FIRST

第一個設計決策前，必須完整讀取：

`.claude-skills/ui-ux-pro-max.md`

優先：

`<resolved-591-root>/.claude-skills/ui-ux-pro-max.md`

如果不存在：

1. 搜尋 workspace
2. 可讀 DM repo 同名 skill 作為 reference
3. 回報 resolved path

如果完全不存在：

`UI_UX_PRO_MAX_NOT_FOUND`

停止。

禁止：

- 只讀摘要
- 用記憶代替
- 假裝已讀
- 用一般 UI 常識冒充

---

# 7. GATE 0 — READ-ONLY AUDIT

GATE 0 完全禁止修改 591。

必須檢查：

## Repository

- repo root
- branch
- HEAD
- git status
- tracked modified files
- untracked files
- active worktrees

## Runtime

- dev server
- actual URL
- actual route
- console errors
- network errors
- visible UI

## UI

至少實際操作：

- 主入口
- 核心工作頁
- 主要 sidebar / rail
- 主要表單
- 主要 modal
- 主要預覽
- 案件流程（若存在）
- 圖片流程（若存在）
- 儲存
- 重新整理
- F5 restore
- error state
- empty state
- loading state
- success state

如果某功能不存在：

記錄 `NOT_PRESENT`。

不要自行補需求。

---

# 8. SOURCE AUDIT

找到目前真正 active source。

至少查：

- app entry
- router
- main page
- shell
- sidebar / rail
- core components
- modal
- forms
- property/listing components
- image components
- state/store
- persistence
- API client
- extraction client
- template system
- preview
- save/load
- tests
- CSS / tokens
- relevant serializer

建立：

`591_COMPONENT_MAP`

格式至少：

| UI | Source | State | Interaction | Persistence | API | Risk |
|---|---|---|---|---|---|---|

---

# 9. USER FLOW MAP

建立：

`591_USER_FLOW_MAP`

至少畫出目前真實流程：

```
入口
 ↓
主要工作
 ↓
資料輸入
 ↓
處理
 ↓
預覽
 ↓
確認
 ↓
儲存 / 發布 / 匯出
```

依實際 source 修改，不要套固定模板。

每一步記：

- 使用者要做什麼
- 現在幾步
- 哪裡困惑
- 哪裡重複
- 哪裡容易出錯
- 哪裡需要即時回饋
- 哪裡適合 direct manipulation
- 哪裡適合 modal
- 哪裡應該保持簡單

---

# 10. DM REFERENCE AUDIT — ONLY WHAT IS NEEDED

不要全面重做 DM audit。

只研究與 591 實際需求相關的 active path。

優先：

### Property extraction

```
URL
↓
擷取
↓
normalization
↓
preview
↓
確認
```

### Property presentation

研究：

- image ratio
- title hierarchy
- price
- area
- layout
- address
- metadata
- card hierarchy
- frame

### Image interaction

研究：

- select
- replace
- move
- resize
- crop
- reset
- delete
- contextual controls

### Editing

研究：

- direct selection
- modal
- toolbar
- progressive disclosure
- replace-first

建立：

`DM_REFERENCE_MAP`

每項記：

- source
- active / inactive
- 可借鑑內容
- 591 不應照搬的部分
- 591 採用風險

---

# 11. DO NOT COPY DM

DM 是參考，不是設計答案。

禁止：

- 整套 DM UI 搬過來
- 整套 DM CSS 搬過來
- 整套 DM state architecture 搬過來
- 為了視覺直接複製 DM component
- 把 591 變成 Canva
- 把 591 變成自由 Sticker canvas

正確：

```
591 真實需求
+
591 現有架構
+
DM 成熟互動
+
UI/UX Pro Max
=
591 專用方案
```

---

# 12. DESIGN PRINCIPLE

591 的目標不是「功能最多」。

目標：

**讓房仲最快完成核心工作。**

優先：

1. 清楚
2. 快
3. 少步驟
4. 不容易做錯
5. 預設結果就能用
6. 需要時才出現進階控制
7. 不破壞現有資料
8. 不破壞既有工作流

---

# 13. INFORMATION ARCHITECTURE

依 GATE 0 真實功能分類：

## High Frequency

核心任務每天會用的功能。

直接可見或一步可達。

## Medium Frequency

偶爾使用的功能。

放：

- drawer
- modal
- contextual panel
- secondary toolbar

## Low Frequency

進階設定。

放：

- More
- Advanced
- Settings

不要把所有功能永久攤開。

---

# 14. VISUAL DIRECTION — PRODUCE 3

依實際 591 產品定位，至少提出三套。

### A — Premium Real Estate

重點：

- 專業
- 高級
- 清楚
- 房仲商務感
- 強 hierarchy

### B — Modern Productivity

重點：

- 工作速度
- 低干擾
- 清楚操作
- 高資訊密度但不混亂

### C — Personal Brand

重點：

- 業務個人品牌
- 圖片
- 品牌色
- 個人資訊
- CTA

禁止：

- generic purple AI SaaS
- neon
- rainbow gradient
- every-panel glass
- giant dashboard cards
- heavy blur
- excessive rounded cards
- excessive shadow
- decorative motion
- 為了漂亮犧牲工作效率

如果 591 真實產品定位與上述方向不符：

以 source evidence 為準，提出更適合的方向。

---

# 15. VISUAL PROPOSAL

GATE 1 仍然完全 READ ONLY。

至少產出：

1. 主工作畫面
2. 核心操作畫面
3. 空狀態
4. 完成狀態
5. Loading
6. Error
7. Modal / Drawer
8. Property / listing card（若存在）
9. Image interaction（若存在）
10. 主要 mobile / responsive state（若產品需要）

至少三套方向都要有：

- 主工作畫面
- 核心操作畫面

推薦方向再補完整狀態。

---

# 16. PROTOTYPE

優先：

### Option 1
可用 design tool → 建 prototype。

### Option 2
Static HTML prototype。

建立在：

- temp
- sandbox
- isolated preview area

不得修改原始 591 source。

### Option 3
Rendered mockup。

無論哪一種：

**必須讓使用者看得到實際視覺結果。**

---

# 17. DEFAULT / EMPTY STATE

如果 591 有 draft / editor：

新資料不應看起來像工程測試畫面。

但：

**existing data 絕對不能被覆蓋。**

Default content 只允許用於：

- new
- empty
- demo
- preview

不得把 sample data 寫入正式資料。

---

# 18. DESIGN SYSTEM

建立 591 專用 tokens。

至少：

- workspace-bg
- surface
- elevated
- border
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

提供：

- HEX
- OKLCH / RGB
- usage
- contrast

---

# 19. TYPOGRAPHY

至少定義：

- family
- size
- weight
- line-height
- letter-spacing
- usage

涵蓋實際 591 會用到的：

- page title
- section title
- body
- label
- helper
- button
- tooltip
- property title
- property metadata
- error
- success
- empty state

不要任意混用大量近似尺寸。

---

# 20. CONTROL SYSTEM

整理：

- Primary
- Secondary
- Tertiary
- Icon
- Contextual
- Destructive
- Toggle
- Segmented
- Preset

每種列：

- height
- padding
- radius
- icon size
- typography
- hover
- active
- selected
- focus
- disabled

Icon-only 必須有 tooltip / accessible label。

---

# 21. MICRO-INTERACTION

如果是 Vue：

優先：

- CSS transition
- CSS transform
- Vue Transition
- 現有 dependency

不要因為視覺需求亂加大型 animation library。

設計：

- hover
- press
- loading
- selected
- modal
- drawer
- save
- success
- error
- property selection
- image selection

動效只用來：

- 告知狀態
- 建立方向感
- 提升理解

不是裝飾。

---

# 22. ORIGINAL WORKING TREE PROTECTION

在：

`APPROVE_591_LOCAL_MERGE`

以前：

**原始 Ticenpi591 working tree 永遠 READ ONLY。**

禁止：

- 修改 source
- 修改 CSS
- 修改 Vue / JS / TS
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
- commit
- push

不要用：

- quick fix
- prototype
- temporary patch
- cleanup

越過 Gate。

---

# 23. CURRENT DIRTY WIP MUST BE PRESERVED

GATE 0 開始先回報：

- resolved repo root
- branch
- HEAD
- git status
- modified files
- untracked files
- active worktrees
- current dev server
- current URL
- screenshot
- relevant source files

不要要求使用者先 commit。

不要替使用者清理 dirty tree。

---

# 24. SAFETY GATES

整體：

```
GATE 0
READ ONLY
Real-state audit

↓

GATE 1
READ ONLY
Visual proposal + prototype

User:
APPROVE_591_UI_IMPLEMENTATION

↓

GATE 2
Isolated 591 Preview Worktree

↓

User:
APPROVE_591_LOCAL_MERGE

↓

GATE 3
Safe local merge
```

Exact token 才算核准。

以下都不算：

- 可以
- OK
- 繼續
- 用 B
- 這個不錯

---

# 25. GATE 1 OUTPUT

必須輸出：

## 591 CURRENT STATE
## 591 USER FLOW MAP
## 591 COMPONENT MAP
## CURRENT UX PROBLEMS
## DM REFERENCE MAP
## THREE VISUAL DIRECTIONS
## RECOMMENDED DIRECTION
## INFORMATION ARCHITECTURE
## COLOR TOKENS
## TYPOGRAPHY
## SPACING
## CONTROL SYSTEM
## MICRO-INTERACTIONS
## DEFAULT / EMPTY STATES
## RESPONSIVE STRATEGY
## IMPLEMENTATION IMPACT
## MOCKUPS / SCREENSHOTS

最後：

`PHASE_1_COMPLETE_WAITING_FOR_APPROVE_591_UI_IMPLEMENTATION`

停止。

---

# 26. GATE 2 — ISOLATED PREVIEW WORKTREE

收到：

`APPROVE_591_UI_IMPLEMENTATION`

才可以實作。

**仍然禁止修改原始 591 working tree。**

建立：

`F:\00-Ticenpi-SaaS\Ticenpi591_ui_preview_wt`

或：

`F:\00-Ticenpi-SaaS\.ui-preview\Ticenpi591`

實際路徑由 agent 依目前 repo 狀態決定並回報。

---

# 27. PREVIEW BASELINE

如果原始 591 有 dirty WIP：

不能只從 HEAD 建 preview 然後假設等於目前產品。

必須：

1. 從 current HEAD 建 preview
2. 原始 repo 不修改
3. 唯讀取得 tracked diff
4. 將必要 tracked WIP 套到 preview
5. 必要時複製 relevant untracked source
6. 不複製：
   - .git
   - node_modules
   - dist
   - cache
   - secrets
   - runtime temp
7. 驗證 baseline parity

禁止對 original：

- stash
- reset
- restore
- clean
- commit

---

# 28. PREVIEW URL

不要假設 3013 / 3113。

GATE 2 必須：

1. 查原始 591 實際 port
2. 找未占用 preview port
3. 啟動 isolated preview
4. 回報 actual URL

要求：

- 不占用 original port
- 不停止 original server
- 不使用 127.0.0.1 當 canonical URL
- preview 與 original 可同時開

---

# 29. PREVIEW DATA SAFETY

Preview 不得污染正式資料。

優先：

- isolated browser context
- disposable data
- test draft
- local mock data

禁止：

- 清原本 IndexedDB
- 清原本 localStorage
- 清 session
- 修改正式 draft
- 修改 Production
- 修改 Staging
- DB migration
- schema migration

如果無法隔離：

`PREVIEW_DATA_ISOLATION_BLOCKED`

立即停止。

---

# 30. IMPLEMENTATION PRIORITY

依實際 GATE 0 audit 排序。

通常優先：

1. shell / workspace
2. primary workflow
3. data entry
4. property/listing presentation
5. image interaction
6. modal / contextual controls
7. empty/loading/error/success
8. responsive
9. accessibility
10. visual polish

但是：

**實際排序以 GATE 0 證據為準。**

---

# 31. IMPLEMENTATION RULES

優先：

- reuse existing state
- reuse existing API
- reuse existing persistence
- reuse existing components
- reuse existing tests
- CSS/component refactor first

禁止：

- backend schema migration
- DB migration
- new scraper/provider
- Production changes
- Staging changes
- Docker deployment
- release changes
- unrelated refactor
- dependency mass upgrade
- state architecture rewrite just for UI

如果必須改 backend：

提出：

`BACKEND_CHANGE_REQUIRED`

停止，等待明確核准。

---

# 32. MUST PRESERVE EXISTING FUNCTIONALITY

GATE 0 找到的既有功能，GATE 2 都必須保留。

至少檢查：

- existing drafts
- save
- load
- F5 restore
- API flow
- extraction
- image handling
- preview
- error handling
- authentication
- permissions
- existing routes

如果某功能不存在：

不要自行新增成既定需求。

如果 visual redesign 導致既有功能退化：

`REGRESSION_FAIL`

不得進 merge。

---

# 33. TESTS

Preview 必須跑：

- existing relevant unit tests
- component tests
- build
- browser smoke / Playwright if available
- core user flow
- responsive check
- accessibility check

不得：

- 刪測試
- 降低測試條件
- 偽造 PASS
- 用 screenshot 取代功能驗證

---

# 34. ACCESSIBILITY

檢查：

- contrast
- keyboard
- focus
- hit target
- tooltip
- accessible label
- selected state
- disabled state
- reduced motion
- error readability

不要虛構 AAA。

只回報實際驗證結果。

---

# 35. GATE 2 FINAL REPORT

輸出：

```
591 UI PREVIEW READY

Target:
<resolved Ticenpi591 path>

Reference:
F:\00-Ticenpi-SaaS\TicenpiDM
DM modified: NO

Original:
<actual original URL>

Preview:
<actual preview URL>

Original 591 source modified: NO

Approved design:
...

Changed files:
- ...

Primary workflow:
...

UI improvements:
...

DM-referenced interactions:
...

Existing functionality:
...

Tests:
Build:
Browser:
Core flow:
Accessibility:

Original dirty WIP preserved: YES
Original source changed: NO
DM source changed: NO
Docker changed: NO
Staging changed: NO
Production changed: NO
Committed to original: NO
Pushed: NO
```

最後：

`PREVIEW_READY_WAITING_FOR_APPROVE_591_LOCAL_MERGE`

停止。

---

# 36. SECOND APPROVAL

只有：

`APPROVE_591_LOCAL_MERGE`

才可以把 Preview 差異合回：

`<resolved Ticenpi591 path>`

之前永遠 READ ONLY。

---

# 37. GATE 3 — SAFE LOCAL MERGE

收到 exact token：

1. 重新檢查 original git status
2. 確認是否出現新的 WIP
3. 逐檔 diff
4. 只合併核准 UI 差異
5. 不整 repo 覆蓋
6. 不 reset
7. 不 clean
8. 不 whole-tree replace
9. 同檔出現新的 external writer 變更 → HARD STOP
10. merge 後重新：
   - tests
   - build
   - browser smoke
   - core flow
11. 確認 DM untouched

PASS 後：

建立 local checkpoint commit。

建議：

`feat: refine 591 ui workflow`

**DO NOT PUSH。**

---

# 38. GIT POLICY

驗收 PASS 的大型 milestone：

- 建立 local checkpoint commit
- 不 push
- 下一階段從 checkpoint 開始

Push 必須另取得使用者明確授權。

---

# 39. HARD STOP

立即停止：

1. 找不到 591 repo
2. 無法確認真正 591 UI
3. 找不到 UI/UX Pro Max
4. DM 必須被修改
5. preview 無法匹配 dirty WIP
6. preview 會污染正式資料
7. 必須 DB/schema migration
8. 必須新增 scraper/provider
9. 必須 Docker / Staging / Production
10. existing data 會被覆蓋
11. autosave / persistence / restore 會退化
12. auth / permission 會改變
13. 必須猜測目前功能
14. merge collision 無法安全處理
15. visual redesign 需要大規模重寫核心資料架構

---

# 40. FINAL SUCCESS TEST

不能只問：

「畫面是不是比較漂亮？」

必須回答：

```
一個第一次使用 591 的房仲，
是否可以更快理解目前要做什麼，
更少步驟完成核心工作，
更少犯錯，
並且不需要學習複雜設計工具？
```

只有在：

- UI 有實際改善
- 核心流程沒有退化
- 現有資料安全
- Preview 可驗證
- tests/build 通過
- DM 沒有被修改

全部成立時，才能宣告完成。

---

# FINAL RULE

```
先看懂真正的 591
↓
讀 UI/UX Pro Max
↓
找出 591 真正痛點
↓
只讀研究 DM 好用的互動
↓
提出 591 專用方案
↓
APPROVE_591_UI_IMPLEMENTATION
↓
建立 isolated Preview Worktree
↓
驗證實際 UI
↓
APPROVE_591_LOCAL_MERGE
↓
才合回原始 591
↓
local checkpoint commit
↓
不自動 push
```

**任何階段不得跳過 approval gate。**

**不要把 Letter 規則帶入 591。**

**不要把 DM 搬進 591。**

**先證據，後設計；先 Preview，後合併。**
