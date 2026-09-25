# DM UI DIRECTOR — CLAUDE UX REDESIGN / DEFAULT TEMPLATE / STICKER-FIRST EDITOR

## ROLE

你現在不是一般 coding agent。

你要同時扮演：

- Senior Product Designer
- UX Director
- Interaction Designer
- Frontend Architecture Reviewer
- Usability Tester
- Design-System Auditor

你的任務不是「把現在 UI 稍微美化」。

你的任務是重新檢視目前 DM 編輯器，讓它變成：

- 大眾第一次打開就看得懂
- 操作步驟盡量少
- 直接在 A4 上操作
- 不需要理解複雜設定才能做出像樣成品
- 新使用者一進來就能看到完整、有質感的正反面預設內容
- UI 看起來像成熟的商業設計工具，而不是工程後台

---

# WORKING REPO

`F:\00-Ticenpi-SaaS\TicenpiDM`

## REFERENCE PROJECTS / SCREENS

主要參考：

- DM:
  `http://localhost:9422`

- OCR / Letter:
  `http://localhost:3013/專案`
  `http://localhost:3013/專案設計`

- Letter source:
  `F:\00-Ticenpi-SaaS\TicenpiLetter`

這些參考專案只能唯讀，不要修改。

---

# TOOLS TO USE

如果可用，優先使用：

1. Playwright MCP / browser automation
2. Claude browser / computer use
3. filesystem / repo inspection
4. screenshot / vision inspection
5. Figma MCP（如果已連線）
6. existing local browser/dev server

如果 Playwright 可用：

- 實際打開 localhost
- hover
- 點擊
- 打開 drawer
- 打開 modal
- 操作圖片
- 操作文字
- 截圖
- 比較 DM 與 OCR/Letter

不要只讀程式碼就宣稱 UI 好不好用。

如果 Figma MCP 可用，可以用來快速建立 wireframe / layout proposal，但不能把 Figma 當成唯一交付；最後仍需對應回實際 DM code architecture。

---

# SAFETY / SCOPE

只允許修改：

`F:\00-Ticenpi-SaaS\TicenpiDM`

禁止修改：

- `F:\00-Ticenpi-SaaS\TicenpiDM_release_wt`
- TicenpiLetter
- OCR / 3013 reference project
- Docker
- VPS
- Staging
- Production
- ExtractionHub
- backend business logic
- Supabase schema
- deployment scripts

不要 commit。
不要 push。
不要 deploy。

目前 DM 有 dirty WIP。

嚴禁：

- git reset
- git reset --hard
- git restore .
- git checkout .
- git clean
- stash apply/pop
- branch switch
- 覆蓋既有未提交 UI WIP

開始前先回報：

- repo
- branch
- HEAD
- git status
- current dirty files
- active worktrees
- possible file collisions

---

# PRIMARY DESIGN PROBLEM

目前 DM 的主要問題不是缺功能，而是：

1. A4 正反面一打開太像空白紙
2. 缺少預設完整成品的感覺
3. 使用者不知道「這個位置可以放什麼」
4. 新手要自己從零配置，很累
5. 圖片 / 大頭貼 / LOGO / QR / Sticker 操作概念不一致
6. 有些設定層級不直覺
7. 操作按鈕過多或角色不清楚
8. 正反面 A4 視覺層次不足
9. 使用者很難預想完成後作品會長什麼樣
10. 整個 UI 比較像「功能都有」，但不像「產品會引導你完成設計」

---

# CORE PRODUCT PRINCIPLE

這個 DM 不應要求一般使用者「設計」。

它應該讓使用者：

```text
選一個好看的預設
↓
替換圖片
↓
修改文字
↓
拖一下位置
↓
完成
```

也就是：

**Replace-first / Direct-manipulation / Template-first**

而不是：

```text
空白紙
↓
自己新增
↓
自己排
↓
自己決定格式
↓
自己猜哪裡該放什麼
```

---

# UX RULES

所有 UI 決策都依照以下原則：

## High-frequency action
直接可見 / 直接操作

## Medium-frequency action
Popover / Drawer / contextual menu

## Low-frequency action
Advanced / More / secondary menu

## Object editing
優先直接點 A4 物件

## Text editing
直接點文字 → centered modal / contextual editor

## Image editing
直接點圖片 → on-canvas handles

## Global/page settings
才放到 compact drawer / layout panel

## Do not
- 不要把所有設定永久攤開
- 不要塞一堆文字按鈕
- 不要讓每個物件類型都有不同操作邏輯
- 不要依賴右側大型 inspector 才能完成基本操作
- 不要讓新手面對空白頁

---

# PHASE 1 — FULL UX AUDIT

先不要改 code。

用 Playwright / Browser 實際操作 DM。

至少操作：

- 每個左側 icon
- hover label
- cases drawer
- A4 front
- A4 back
- text editing
- modal
- image / logo / QR
- portrait
- sticker
- layout/style/data panel
- front/back selection
- page settings
- add-object flow
- save/restore behavior

每一個入口都要紀錄：

- What is it?
- Who needs it?
- High / Medium / Low frequency
- Current clicks required
- Is wording understandable?
- Is location intuitive?
- Does it deserve permanent space?
- Can it be merged?
- Can direct manipulation replace it?
- Is there duplicate functionality?
- Is it visually dominant enough / too dominant?

建立一份：

`CURRENT_UI_AUDIT`

---

# PHASE 2 — COMPARE DM VS OCR / LETTER

實際打開：

- `http://localhost:3013/專案`
- `http://localhost:3013/專案設計`

比較：

## A4 preview
- background
- page shadow
- depth
- margin
- spacing
- focus
- front/back presentation

## Editing
- direct click
- image selection
- sticker controls
- resize
- move
- rotate
- z-order
- reset

## Navigation
- icons
- labels
- hover
- drawer
- layout density

不要直接複製 OCR。

請判斷：

`OCR/Letter 哪些 UX 適合 DM，哪些不適合。`

---

# PHASE 3 — SOURCE AUDIT

讀目前 DM 實際 source。

至少找出：

- DmEditor.vue
- A4Preview.vue
- preview.js
- editor layout CSS
- cases drawer
- image components
- text modal
- default layout state
- front/back state
- style preset definitions
- layout-data definitions
- autosave/hydrated/dirty logic

建立：

`UI_COMPONENT_MAP`

每個畫面元素標記：

- source file
- component
- state
- CSS selector
- current dimensions
- current typography
- current interaction

---

# PHASE 4 — REDESIGN INFORMATION ARCHITECTURE

重新分類所有功能。

建議框架：

## Left Icon Rail
約 68–72px。

只保留真正的一級入口。

例如：

- Cases
- Images / Assets
- Text / Add
- Layout
- Style
- Data
- Settings

不要因為現有功能有 10 個，就保留 10 個一級 icon。

請自行判斷哪些應合併。

Normal:
- icon only

Hover:
- tooltip / flyout label

不要 layout shift。

---

# IMAGE SYSTEM — UNIFY

目前可能有：

- Portrait
- Logo
- LINE QR
- Generic image
- Sticker

從使用者角度，它們都屬於：

`Image / Asset Object`

請設計一套統一 mental model。

例如：

```text
Images
├─ Business Portrait
├─ Logo
├─ LINE QR
├─ Upload Image
└─ Sticker
```

使用者只需要學一套：

- select
- drag
- resize
- rotate
- flip
- bring forward
- send backward
- reset
- delete

不要有獨立 Portrait icon，除非 audit 後有很強理由。

---

# TEMPLATE-FIRST DEFAULT EXPERIENCE

這是本輪最重要部分。

新使用者不要再看到空白 A4。

需要設計：

- Front default composition
- Back default composition
- placeholder objects
- placeholder image
- placeholder text
- business portrait placeholder
- logo placeholder
- QR placeholder
- CTA placeholder
- sample cards / sections

這些 placeholder 必須讓使用者一看就知道：

- 哪裡可以換圖
- 哪裡可以改字
- 哪裡是品牌
- 哪裡是業務資訊
- 哪裡可以放物件

---

# PLACEHOLDER DESIGN PRINCIPLE

Placeholder 不是純灰框。

它應該像「一份看起來已完成 70% 的漂亮作品」。

例如：

- 真實感示意圖片
- 虛擬標題
- 虛擬價格
- 虛擬業務資訊
- 示意 QR
- 示意 Logo
- 示意卡片

但要清楚表示可替換。

例如：

`點擊替換主圖`

或 hover 顯示：

`更換圖片`

---

# DESIGN AT LEAST 3 TEMPLATE DIRECTIONS

不要一開始就鎖死一個 layout。

至少先提出三種：

## A. Editorial / Magazine
- 主圖主導
- 強烈 typography
- 有節奏的大小圖片
- 高質感

## B. Real-estate Commercial
- 物件資訊清楚
- 圖片與資訊平衡
- 業務/QR/CTA 清楚

## C. Personal-brand / Social
- 業務 portrait 比較突出
- Sticker 感
- 活潑但不亂
- CTA 強

每一套都要有：

- front
- back
- object hierarchy
- editable placeholders
- expected user flow

---

# DO NOT FORCE 2×3

使用者之前舉過 2×3 六物件，只是例子。

不要把：

`6 cards / 2×3`

當成固定需求。

應根據 template 決定：

- 1 hero + 3 cards
- 2×2
- asymmetrical
- editorial
- 4-column
- mixed sizes

但 layout system 上限可保留：

- max 4 columns
- max 12 rows

---

# FRONT / BACK MUST BOTH LOOK COMPLETE

Front 不能只是資料文字。

Back 也不能只是空白。

兩面都要像完成品。

## Front
應有明確視覺主次：

- brand
- hero visual
- primary message
- selected property content
- CTA / business presence

## Back
可以承擔：

- more properties
- services
- additional images
- business information
- QR / contact
- supporting content

兩面應有一致 design language。

---

# A4 PREVIEW VISUAL QUALITY

參考 OCR/Letter 的「紙張感」。

要求：

- workspace background
- white paper
- subtle border
- layered shadow
- spacing
- visual depth

不要只是白色 rectangle。

但也不要：

- 重陰影
- 過度 3D
- 花俏漸層污染設計

---

# DIRECT MANIPULATION

盡量讓編輯發生在 A4 上。

## Text
點文字：
- centered edit modal
- live preview
- formatting
- apply/cancel

## Image
點圖片：
- on-canvas handles
- drag
- resize
- rotate
- quick action menu

## Placeholder
點 placeholder：
- replacement action first
- 不要先讓使用者進複雜設定

---

# REDESIGN BUTTON HIERARCHY

對所有 buttons 做一次 hierarchy cleanup。

分類：

## Primary
真正主要流程

## Secondary
常用輔助

## Tertiary / icon
輕量操作

## Dangerous
delete/reset

確認：

- consistent height
- consistent icon size
- readable labels
- hit area sufficient
- hover/focus state
- no ambiguous icon-only action without tooltip

不要只是保持歷史尺寸。

---

# DESIGN TOKENS

盤點現有：

- spacing
- radius
- shadows
- font size
- button height
- icon size
- panel width
- drawer width
- modal width
- background colors

重新建立一致性。

避免出現：

- 31px button
- 36px button
- 42px button
- 47px button

除非有合理 hierarchy。

建立一個 compact UI token proposal。

---

# DO NOT REMOVE FUNCTIONALITY FOR BEAUTY

如果某功能看起來醜，但確實重要：

不要刪功能。

可以：

- move
- merge
- hide under popover
- contextualize
- rename

功能完整性 > 純視覺。

---

# PHASE 5 — DESIGN PROPOSAL BEFORE IMPLEMENTATION

先產出一份：

`DM_UI_REDESIGN_PROPOSAL`

包含：

1. Current problems
2. New information architecture
3. New icon rail
4. A4 workspace
5. image/sticker system
6. text editing
7. page targeting
8. template system
9. 3 template directions
10. button hierarchy
11. design tokens
12. before/after user flow

如果 Figma MCP 可用：
- 建 wireframe / editable design

如果不可用：
- 用 HTML/CSS prototype 或清楚 screenshot/mockup

---

# PHASE 6 — CHOOSE AND IMPLEMENT

不要等使用者逐一回答小問題。

根據：

- simplicity
- learnability
- direct manipulation
- visual quality
- compatibility with current architecture
- minimal schema changes

自行選擇最合理的主方案作為 default implementation。

但不要刪除另外兩個 template 的概念；可以保留為後續 preset。

實作時：

- 優先 reuse current state/schema
- 不做 backend migration
- 不做 rich text schema migration
- 不破壞 autosave
- 不破壞 existing draft
- new default only affects new/empty draft
- existing user design must win

---

# NEW USER DEFAULT

新使用者 / 新 draft：

不要空白。

預設直接載入一套完整 front/back template。

要求：

- 有示意圖片
- 有文字 placeholder
- 有業務 placeholder
- 有 CTA
- 有 image slots
- 有完整 hierarchy

讓使用者第一眼就知道：

`這個系統能做出什麼。`

Existing draft 不可被覆蓋。

---

# PHASE 7 — PLAYWRIGHT UX VALIDATION

實作後重新用 Playwright。

測試：

## First-time user
- 第一次打開是否知道下一步做什麼
- 是否看得到完整作品

## Replace flow
- 換主圖要幾步
- 換業務照片要幾步
- 改標題要幾步
- 改 QR 要幾步

目標：
常用操作盡量 <= 2 interactions。

## Image
- select
- move
- resize
- rotate
- z-order
- reset

## Text
- click
- edit
- preview
- apply
- cancel

## Front/back
- clearly distinguish
- editing correct page

## UI
- no outer vertical scroll
- no top bar
- no heavy permanent side panel
- A4 remains dominant

---

# ACCESSIBILITY / GENERAL PUBLIC USABILITY

檢查：

- button target size
- text contrast
- hover
- keyboard focus
- tooltip
- icons meaning
- selected state
- error state
- disabled state

一般使用者不應需要猜 icon 意義。

Icon-only controls 必須有 tooltip。

---

# SUCCESS CRITERIA

成功不是：

`UI 比較漂亮`

而是：

1. 新手打開就知道可以做什麼
2. 正反面一開始就像成品
3. 可以直接替換 placeholder
4. 文字直接點擊修改
5. 圖片直接操作
6. 圖片操作邏輯一致
7. 常用操作更少步
8. A4 有紙張層次
9. icon rail 更乾淨
10. settings 不再搶畫面
11. existing drafts 不壞
12. F5 restore 不壞
13. build/tests pass

---

# HARD STOP

如果遇到以下狀況，停止並報告：

- 必須改 backend schema
- 必須做 destructive migration
- 必須重做所有 draft schema
- 會破壞 existing draft restore
- 會破壞 autosave rollback
- 參考 project 無法讀取，而實作必須靠猜
- active writer 正在改同一核心檔案
- 必須修改 Docker / Staging / Production

---

# FINAL OUTPUT

最終回報分兩部分。

## PART 1 — UX DIRECTOR REPORT

```text
CURRENT PROBLEMS
- ...

NEW INFORMATION ARCHITECTURE
- ...

BUTTON / CONTROL HIERARCHY
- ...

IMAGE / STICKER MODEL
- ...

DEFAULT FRONT TEMPLATE
- ...

DEFAULT BACK TEMPLATE
- ...

OTHER TEMPLATE DIRECTIONS
- ...

NEW USER FLOW
- ...

BEFORE / AFTER CLICK COUNT
- ...

DESIGN TOKENS
- ...
```

## PART 2 — IMPLEMENTATION

```text
Repo:
Branch:
HEAD:

Files changed:
- ...

New default template:
- ...

Icon rail:
- ...

A4 preview:
- ...

Image system:
- ...

Text editing:
- ...

Front/back targeting:
- ...

Existing draft compatibility:
- ...

Tests:
Build:
Playwright validation:

Docker changed: NO
Staging changed: NO
Production changed: NO
Committed: NO
Pushed: NO
```

## FINAL PRODUCT TEST

Before declaring PASS, answer:

`如果一個完全沒看過 DM 的房仲業務，第一次打開，能不能在 3 分鐘內理解怎麼換圖、改字、換業務照片並做出可用的正反面？`

如果答案不是明確 YES，繼續改善，不要宣告完成。
