# DM EDITOR UI V3.2 — WORKSPACE CLEANUP / ICON RAIL / DUAL A4 / UNIFIED IMAGE SYSTEM / 4×12 LAYOUT GRID

## OWNER
DM EDITOR UI V3.2 OWNER

## WORKING REPO
`F:\00-Ticenpi-SaaS\TicenpiDM`

## LOCAL REFERENCE
優先唯讀參考目前本機已存在的實作與畫面：

- `http://localhost:3013/專案`
- `http://localhost:3013/專案設計`
- `F:\00-Ticenpi-SaaS\TicenpiLetter`

如果 3013 對應的實際 source repo 能從目前工作區明確找到，允許唯讀檢查 source。
找不到就只使用目前可見的畫面與 TicenpiLetter，不要自行猜 repo。

---

# GOAL

延續目前 DM Editor UI V3 / V3.1 的本機 WIP。

這一輪只處理：

1. 整體 workspace 版面整理
2. 移除下邊條
3. 無 Top Bar / 100dvh / 外層無垂直 scrollbar
4. 左側約 70px Icon Rail
5. Icon 平常只顯示圖片，hover 才顯示文字
6. `TicenpiDM` 標籤移到左側最上方，不額外產生 top bar 高度
7. 正反雙面 A4 最大化並排
8. A4 頁面加入更立體的陰影 / 底層感
9. 業務圖片 / LOGO / LINE QR / 一般圖片 / Sticker 統一互動模型
10. 不再保留獨立「大頭照」Icon
11. 正面預設版面做上下分層
12. 版面設定必須明確知道套用正面還是反面
13. 版面格線 / 配置能力：橫向最多 4 格、直向最多 12 列

不要擴大成 backend / schema / deployment 任務。

---

# HARD BOUNDARIES

只允許修改：

`F:\00-Ticenpi-SaaS\TicenpiDM`

禁止修改：

- `F:\00-Ticenpi-SaaS\TicenpiDM_release_wt`
- TicenpiLetter 原始碼（只能讀）
- 3013 參考專案原始碼（只能讀）
- Docker image / container
- VPS
- Staging
- Production
- ExtractionHub
- scraper business logic
- RemoveBG backend logic
- Supabase schema
- deployment scripts

不要 commit。
不要 push。
不要 deploy。

---

# PRESERVE CURRENT WIP

目前 repo 已有未提交 UI WIP。

嚴禁：

- `git reset`
- `git reset --hard`
- `git checkout .`
- `git restore .`
- `git clean`
- stash pop/apply
- branch switch
- 整檔覆蓋混有既有 WIP 的核心檔案

開始前先回報：

1. repo root
2. branch
3. HEAD
4. `git status --short`
5. 目前 dirty WIP
6. active worktrees / possible collision
7. 本輪預計修改檔案

如果 writable repo 不是：

`F:\00-Ticenpi-SaaS\TicenpiDM`

立即 HARD STOP。

---

# PHASE 1 — INSPECT FIRST

先讀目前最新本機 source，不要直接照提示詞想像 UI。

至少找出：

1. Editor 外層 layout
2. Icon Rail
3. `TicenpiDM` 標籤
4. Bottom bar / bottom controls
5. A4 front/back container
6. A4 page shadow/background styles
7. Cases drawer
8. Image entry / asset UI
9. LOGO object implementation
10. LINE QR implementation
11. portrait / business photo implementation
12. generic image / sticker implementation
13. transform state
14. z-index / bring forward / send backward
15. F5 restore / autosave path
16. layout/page settings
17. front/back active page state
18. grid / row / column / card layout logic

並唯讀參考：

- `http://localhost:3013/專案` 的 A4 預覽立體感
- `http://localhost:3013/專案設計` 的 LOGO / LINE QR / 其他圖片 / 大頭照操作
- TicenpiLetter 的 Sticker / Transform 互動

先回報：
- 真正 source files
- 哪些既有能力已存在
- 哪些需要補
- 最小修改方案

然後才實作。

---

# A. REMOVE BOTTOM BAR

刪除 DM 編輯器底部整條固定控制列。

要求：

- 不要保留固定 bottom bar
- 不要因移除後留下空白高度
- 原本必要功能若仍需存在，要移到更精簡的位置
- 不得重新做另一條 disguised bottom bar

如果底部原本包含：
- save state
- zoom
- front/back
- other controls

先盤點哪些真的必要，再以：
- floating compact controls
- local corner controls
- page-local controls

等方式重放。

不要犧牲 A4 主畫布高度。

---

# B. NO TOP BAR

整個 DM editor 不要 Top Bar。

禁止：
- 固定 header
- page title bar
- brand bar
- 為 `TicenpiDM` 多加一條橫向區域

工作區要從 viewport 頂端直接開始。

---

# C. 100DVH / NO OUTER VERTICAL SCROLL

Editor workspace：

```css
height: 100dvh;
```

目標：

- 外層 body/editor shell 不出現垂直 scrollbar
- 不要整頁上下滾動
- 必要 scroll 只存在於局部：
  - case drawer
  - settings drawer
  - asset drawer

中央 A4 canvas 本身應維持穩定。

如果目前有：
- 100vh
- min-height
- calc(100vh - topbar - bottombar)

請重新整理成符合「無 Top Bar / 無 Bottom Bar」的 100dvh layout。

---

# D. LEFT ICON RAIL ≈ 70PX

左側導覽改為固定窄 Icon Rail。

目標寬度：

`約 70px`

不要死守 70.000px；如果現有 icon/padding 需要 68–72px 才整齊，可用合理實際值。

要求：

- compact
- vertical
- icon centered
- active state clear
- 不顯示永久文字
- 不因此擠壓 A4 過多

---

# E. ICON ONLY + HOVER LABEL

左側 Icon Rail 平常只顯示 icon 圖片。

文字規則：

```
Normal:
[ icon ]

Hover:
[ icon ]  → 顯示目前這個 icon 的文字
```

hover label 可使用：

- tooltip
- small flyout label
- compact floating tag

要求：

- 不常駐
- 不增加 rail 寬度
- 不造成 layout shift
- 不遮住主要 A4 太多
- z-index 正確
- active icon 仍可辨識

禁止把 icon + text 永久並排。

---

# F. TICENPIDM LABEL AT TOP OF ICON RAIL

把 `TicenpiDM` 品牌 / 標籤移到左側 Icon Rail 最上方。

要求：

- 位於 rail 最高處
- 視覺可辨識
- 不建立 top bar
- 不額外增加水平區塊
- 不讓中央 canvas 往下推
- rail 仍保持 compact

若品牌文字過長，可使用：
- compact logo
- stacked mark
- abbreviation + tooltip

但不要自己換品牌名稱。

---

# G. DUAL A4 MAXIMIZED SIDE-BY-SIDE

中央工作區：

- 正面 A4
- 反面 A4
- 並排
- 最大化利用 viewport

目標：

- Rail 之外空間盡量給 A4
- 左側 drawer 關閉時，A4 應明顯放大
- 正反面都同時可見
- 不改成單面 pager

如果 viewport 不足：
- 優先等比例縮放兩張 A4
- 不要讓整頁出現垂直 scrollbar
- 不要把 A4 做到過小

---

# H. A4 DEPTH / PAPER SHADOW

參考：

`http://localhost:3013/專案`

A4 preview 要有更清楚的紙張層次。

目標：

- A4 white paper
- subtle border
- shadow
- background separation
- slight depth

不要過度：

- heavy dark shadow
- neon
- exaggerated 3D
- large decorative frame

要像設計軟體裡「紙張浮在 workspace 上」。

正面與反面都一致。

---

# I. IMAGE SYSTEM — REMOVE SEPARATE PORTRAIT ICON

圖片相關功能統一入口。

不要再保留獨立「大頭照」Icon。

圖片素材概念統一包含：

- LOGO
- LINE QR
- 其他圖片
- 業務照片 / 大頭照

左側以「圖片 / 素材」入口統一管理。

---

# J. REFERENCE 3013 IMAGE DESIGN

參考：

`http://localhost:3013/專案設計`

已知使用者認可的模式：

## LOGO

已實際驗證：

- 直接點選 PASS
- 縮小 PASS
- 放大 PASS
- 移動 PASS
- F5 restore PASS

功能：

- Move
- Resize
- Bring forward
- Send backward
- Reset

## LINE QR

已實際驗證：

- 直接點選 PASS
- 縮小 PASS
- 放大 PASS
- 移動 PASS
- F5 restore PASS

功能：

- Move
- Resize
- Z-order
- Reset

本輪不要把這些已經能用的功能改壞。

---

# K. BUSINESS PHOTO / PORTRAIT REDESIGN

業務圖片 / 大頭照重新整理成與圖片系統一致的物件。

不要再把它當「固定大頭貼框 + 獨立控制面板」。

目標：

- 進入圖片素材系統
- 可以直接在 A4 上點選
- selected state 清楚
- 操作方式接近 LOGO / QR / Sticker

沒有上傳照片時：
- 可以保留目前合理 placeholder

一旦放入圖片：
- 不要保留多餘的大頭照框
- 圖片本身成為可操作 Sticker / image object

視覺與操作優先參考 3013 / TicenpiLetter。

---

# L. GENERIC IMAGE / STICKER STATE

一般圖片 / Sticker 的核心 state 應統一確認：

- `x`
- `y`
- `width`
- `height`
- `rotation`
- `flipX`
- `zIndex`

如果目前還有必要的：
- aspect ratio
- source URL
- crop
- opacity

可以保留既有 state，但不要為本輪無端擴 schema。

---

# M. ON-CANVAS IMAGE OPERATIONS

一般圖片 / Sticker / 業務照片至少支援：

- Drag / Move
- Resize
- Rotate
- Horizontal Flip
- Bring Forward
- Send Backward
- Delete
- Reset Position / Transform

LOGO / LINE QR 保留既有已通過能力。

目標不是所有圖片都完全一模一樣，而是互動模型一致：

```
點圖片
↓
Selected
↓
on-canvas controls / compact contextual actions
```

不要建立永久右側屬性欄。

---

# N. F5 RESTORE MUST REMAIN SAFE

所有圖片物件的：

- position
- size
- rotation
- flip
- z-index

如果目前已持久化，必須繼續 F5 restore。

不允許為了 UI 整理破壞：

- autosave
- draft restore
- hydrated guard
- dirty guard

如果某種 transform 目前無 persistence，先回報，不要假裝已完成。

---

# O. FRONT DEFAULT LAYOUT — TOP / BOTTOM LAYERS

正面預設版面重新整理成明確「上下分層」。

這不是要求重做 schema。

先用目前現有 layout capability 整理：

```
TOP ZONE
- 品牌 / 主標 / 主視覺 / 核心資訊

BOTTOM ZONE
- 業務資訊 / 聯絡資訊 / 卡片 / CTA / secondary content
```

實際內容以目前 DM 真正有的欄位為準。

目標：

- 上下資訊層級清楚
- 視覺更穩
- 不要所有內容擠成一團
- 初始版面更像完整成品

不要覆蓋既有使用者 custom draft。

只調整新 / default layout。

---

# P. FRONT / BACK TARGET MUST BE EXPLICIT

所有「版面級」設定都必須清楚知道作用對象。

UI 至少要顯示：

- 正面
- 反面
- 如確實存在 global，再顯示整體

不要讓使用者不知道目前在改哪一面。

要求：

- selected page state 明確
- setting panel header / tab / segmented control 可辨識
- 切換正反面時，設定值應跟著正確 page context
- 不得誤把正面設定套到反面
- 不得把反面設定寫進正面 state

先沿用現有資料架構，不要為此建立大規模 schema migration。

---

# Q. LAYOUT GRID — MAX 4 COLUMNS × 12 ROWS

版面配置能力目標：

- 橫向最多 4 格
- 直向最多 12 列

先檢查目前是否已有：

- columns
- rows
- card count
- grid template
- block layout
- layout presets

如果已有 grid：
- 把限制明確設成 max 4 × 12
- UI 不允許超過
- preview 正確

如果沒有真正 grid engine：
- 不要本輪硬造一套完整自由 grid system
- 用現有 layout/preset capability 做最小安全實作
- 若無法在不改 schema 的情況下完成，HARD STOP 並回報

---

# R. RESPONSIVE / VIEWPORT RULES

Desktop editor 優先。

正常桌面尺寸要保證：

- 70px rail
- 雙 A4
- 無 outer vertical scroll
- drawers 可以 local scroll

不要為了小螢幕把 desktop 版面弄壞。

若 viewport 太窄：
- 可以等比例縮 A4
- 可以 overlay drawer
- 但不要讓 Top Bar / Bottom Bar 回來

---

# S. DO NOT REGRESS TEXT EDIT V3

目前已完成的：

- A4 文字直接點擊
- centered modal
- live preview
- Apply
- Cancel rollback
- autosave safety

全部保留。

本輪不是重做文字編輯。

若修改 layout 影響 click target / modal position，必須修到原功能繼續 PASS。

---

# T. TESTS

先跑現有 relevant tests 建 baseline。

完成後至少新增 / 更新：

## Workspace
- no top bar
- no bottom bar
- workspace 100dvh
- no outer vertical overflow

## Icon rail
- width around intended value
- `TicenpiDM` at top
- labels hidden normally
- label visible on hover/focus
- no layout shift

## A4
- front/back both render
- side-by-side desktop
- paper shadow styles exist

## Image system
- no separate portrait icon
- portrait available under image/assets entry
- LOGO behavior preserved
- LINE QR behavior preserved
- generic sticker controls
- transform state persistence
- F5 restore

## Page targeting
- front settings modify front only
- back settings modify back only

## Grid
- cannot exceed 4 columns
- cannot exceed 12 rows

不要刪測試來讓 build 過。

---

# U. MANUAL LOCAL BROWSER ACCEPTANCE

完成後用本機實際驗收。

## Workspace

確認：

- Top Bar = none
- Bottom Bar = none
- height = 100dvh behavior
- outer vertical scrollbar = none
- Icon Rail ≈70px
- `TicenpiDM` 在最上面

## Icon hover

逐個 icon：

- normal only icon
- hover shows correct text
- mouse leave hides text
- no canvas shift

## A4

- front/back maximize side-by-side
- both have subtle paper depth/shadow
- text direct edit still works
- modal still centered

## Images

LOGO：
- select
- move
- resize
- z-order
- reset
- F5 restore

LINE QR：
- same checks

Business portrait：
- under image/assets
- no separate portrait icon
- direct select
- resize
- move
- rotate if implemented through sticker model
- restore

Generic image/sticker：
- drag
- resize
- rotate
- flipX
- bring forward
- send backward
- delete
- reset

## Page target

- select front
- change layout setting
- only front changes

- select back
- change layout setting
- only back changes

## Grid

- 4 columns accepted
- 5 columns rejected / unavailable
- 12 rows accepted
- 13 rows rejected / unavailable

---

# HARD STOP CONDITIONS

遇到以下情況停止，不自行擴大：

1. 要完成 4×12 必須重做 backend/schema
2. 圖片統一必須破壞既有 draft schema
3. portrait / logo / QR state 互相不相容且需要 migration
4. active writer 正在改同一核心檔案
5. 需要修改 Docker / Staging / Production
6. 需要改 release worktree
7. 會破壞既有 F5 restore
8. 會破壞 V3 text modal rollback
9. 找不到 3013 對應 reference source，且僅靠猜測才能完成

---

# ACCEPTANCE CRITERIA

PASS only if:

1. 無 Top Bar
2. 無 Bottom Bar
3. workspace = 100dvh behavior
4. outer vertical scrollbar 無
5. Icon Rail 約 70px
6. `TicenpiDM` 位於 rail 最上方
7. icon 平常只顯示圖片
8. hover 才顯示文字
9. 正反 A4 最大化並排
10. A4 有紙張陰影 / 立體層次
11. 不存在獨立大頭照 icon
12. 業務照片整合進圖片素材系統
13. LOGO 既有能力不退化
14. LINE QR 既有能力不退化
15. generic sticker 支援要求的 transform
16. F5 restore 不退化
17. 正面預設版面上下分層
18. page-level setting 明確區分正面/反面
19. max columns = 4
20. max rows = 12
21. V3 文字直接點擊 / modal / rollback 不退化
22. local tests PASS
23. build PASS
24. local browser acceptance PASS
25. Docker changed = NO
26. Staging changed = NO
27. Production changed = NO
28. commit = NO
29. push = NO

---

# FINAL REPORT

只回報：

```text
DM EDITOR UI V3.2

Repo:
Branch:
HEAD:

Files modified:
- ...

Workspace:
- Top Bar: removed
- Bottom Bar: removed
- Height:
- Outer vertical scroll:
- Icon Rail width:
- TicenpiDM placement:

Icon hover:
- ...

A4:
- dual side-by-side:
- shadow/depth:
- resulting scale behavior:

Image system:
- portrait icon removed:
- portrait integration:
- LOGO:
- LINE QR:
- sticker:
- transform state:
- F5 restore:

Front default layout:
- top zone:
- bottom zone:

Page targeting:
- front:
- back:

Grid:
- max columns:
- max rows:

Regression:
- text direct edit:
- centered modal:
- Apply/Cancel rollback:
- autosave safety:

Tests:
Build:
Browser acceptance:

Existing dirty WIP preserved: YES/NO
Docker changed: NO
Staging changed: NO
Production changed: NO
Committed: NO
Pushed: NO

Remaining issue:
- none / exact blocker
```
