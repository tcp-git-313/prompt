# DM EDITOR UI V3.1 — STICKER PORTRAIT + FULL TEXT MODAL + LAYOUT DATA CONSOLIDATION

## OWNER
DM EDITOR UI V3.1 OWNER

## WORKING REPO
`F:\00-Ticenpi-SaaS\TicenpiDM`

## PRIMARY REFERENCE REPO
`F:\00-Ticenpi-SaaS\TicenpiLetter`

## SECONDARY VISUAL REFERENCE
使用者提到「瑞特專案」的業務照片設計可作為視覺參考。

先在現有工作區 / repo 中找出實際的「瑞特專案」來源與相關實作。
如果無法明確識別是哪一個專案或檔案，只回報找不到，不要自行猜一個專案代替。

---

# GOAL

延續已完成的 DM Editor UI V3，本輪只做 Local UI/interaction refinement。

已知上一輪已完成：
- A4 文字直接點擊
- centered modal
- live preview
- Apply / Cancel rollback
- autosave safety gate
- 雙面 A4
- 窄案件 drawer
- 長標題截斷 / tooltip

本輪要把這些功能補完整並重新整理：

1. 大頭貼改成真正 Sticker 操作
2. A4 上「版面資料」所有可編輯文字都能直接點擊修改
3. 文字 Modal 補成完整格式工具，不再只有字型 + 字級
4. 文字工具列盡量單排、整齊
5. 不使用永久右側屬性欄
6. 新增「整版面」入口，讓使用者能調整整體版面
7. 「版面風格」只負責 style preset，不再混放文字大小
8. 品牌名稱 / 店名 / 公司 / 姓名 / 電話等文字大小控制全部整合進「版面資料」
9. 新使用者 / 新空白設計的正面預設套用「石墨滴色金」
10. 調整 A4 預設 Typography，讓第一次打開就有整齊比例
11. 案件清單再精簡：只顯示「標題 + 金額」，標題字級比目前 +2px

---

# HARD BOUNDARIES

只允許修改：

`F:\00-Ticenpi-SaaS\TicenpiDM`

禁止修改：

- `F:\00-Ticenpi-SaaS\TicenpiDM_release_wt`
- TicenpiLetter 原始碼（只能讀取參考）
- 瑞特專案原始碼（只能讀取參考）
- Docker image / container
- VPS
- Staging
- Production
- ExtractionHub
- scraper business logic
- RemoveBG backend logic
- deployment scripts
- Production auth

不要 commit。
不要 push。
不要 build Docker。
不要 deploy。

開始前先回報：

1. repo root
2. branch
3. HEAD
4. git status
5. 目前既有 dirty WIP
6. active worktrees / possible collision

如果 writable repo 不是：
`F:\00-Ticenpi-SaaS\TicenpiDM`

立即 HARD STOP。

---

# IMPORTANT — DO NOT REGRESS V3

上一輪已完成的互動不可退回：

- 不要恢復 Top Bar
- 不要建立永久右側屬性欄
- A4 仍然雙面
- 文字仍然直接點 A4 編輯
- 仍然用 centered modal
- live preview 必須保留
- Apply / Cancel rollback 必須保留
- Cancel 不可被 autosave 偷偷存下
- 案件清單仍是 Icon-triggered compact drawer

---

# PHASE 1 — INVESTIGATION FIRST

先讀目前 V3 實作，再修改。

至少確認：

1. `DmEditor.vue`
2. `A4Preview.vue`
3. `preview.js`
4. `DmEditor.ui-v3.test.js`
5. 大頭貼 / 業務照片目前如何 render
6. 圖片 transform state 現在是否已有：
   - x
   - y
   - scale
   - rotation
   - flip
   - zIndex
7. 所有 A4 可編輯文字的來源與 edit-key
8. 哪些「版面資料」文字目前還不能點擊
9. 文字 Modal 現在實際支援哪些格式
10. 「版面風格」現在混了哪些文字尺寸設定
11. 「版面資料」現在有哪些欄位
12. 現有 style preset 中「石墨滴色金」的實際 preset id / key
13. 新使用者 / 新 draft 預設值從哪裡建立
14. 案件清單目前 row 結構
15. 現有 autosave / dirty / hydrated 邏輯

同時唯讀檢查：

`F:\00-Ticenpi-SaaS\TicenpiLetter`

找出 LETTER 實際使用的：

- Sticker 圖片選取框
- resize handles
- rotate handle
- drag behavior
- transform state
- 圖片上直接操作的控制點
- centered text modal
- text formatting toolbar
- live preview
- Apply / Cancel rollback

不要憑印象重做一套。

瑞特專案若找得到，唯讀確認其「業務照片」視覺設計方式。
只借用視覺 / UX 思路，不要複製無關架構。

Phase 1 先回報：
- 真正 source files
- 可以直接沿用的既有機制
- 本輪最小修改範圍

然後才實作。

---

# A. BUSINESS PORTRAIT / 大頭貼 → STICKER MODE

## Current expected behavior

### 尚未放照片
維持目前 DM 的預設大頭貼 placeholder / 預設框。

不要因為這輪修改把空狀態弄掉。

### 一旦使用者上傳業務照片

預設大頭貼框必須消失。

畫面只留下「業務照片 Sticker」。

概念：

```
沒有照片
→ 顯示原本 placeholder / 大頭貼框

有照片
→ placeholder 消失
→ 只顯示照片 Sticker
```

不要出現：

```
大頭貼框
+
照片貼在框裡
```

使用者明確不要這種模式。

---

# B. PORTRAIT STICKER — DIRECT ON-CANVAS TRANSFORM

業務照片上傳後，要像 LETTER 的 Sticker 一樣直接在 A4 上操作。

至少支援：

- drag / move（如果現有 Letter Sticker 有）
- 放大
- 縮小
- 旋轉

控制點必須直接出現在被選中的照片 Sticker 上。

不要要求使用者打開右側屬性欄。

不要建立永久 inspector。

目標互動：

```
點業務照片
↓
照片進入 selected state
↓
照片周圍顯示操作控制
↓
直接拖曳 / 放大 / 縮小 / 旋轉
```

Transform 行為優先參考 TicenpiLetter 已證明可用的實作。

## Important

- 不要破壞 A4 print coordinates
- 不要讓旋轉後圖片消失
- 不要因 transform 導致 autosave race
- 正面 / 背面如果都允許照片物件，應使用一致 transform 邏輯
- 若 DM 已有 transform state，優先統一使用，不另建第二套

---

# C. BUSINESS PHOTO VISUAL DESIGN

業務照片的預設視覺呈現：

1. 先看目前 DM
2. 再看 TicenpiLetter Sticker UX
3. 再看使用者提到的「瑞特專案」業務照片設計

如果瑞特專案可找到，吸收其適合 DM 的：
- 裁切
- 邊緣
- 視覺比例
- 乾淨程度
- 選取狀態

但不要抄入無關 UI。

如果找不到瑞特專案，保留目前 DM 視覺，不要自行發明新品牌風格。

---

# D. ALL LAYOUT-DATA TEXT MUST BE DIRECTLY EDITABLE FROM A4

使用者要求：

> 版面資料的所有文字，都要可以直接點擊 A4 預覽面修改。

先盤點所有可編輯文字。

至少包括目前實際存在的類型，例如：

- 品牌名稱
- 店名
- 公司名稱
- 主標題
- 副標題
- 業務姓名
- 職稱
- 電話
- LINE / 聯絡文字
- 服務項目
- Slogan
- 卡片標題
- 卡片數字
- 卡片細節
- 卡片價格
- 其他由「版面資料」提供的可見文字

不要只讓部分文字可點。

規則：

```
A4 上看得到
+
本來屬於可編輯版面資料
→ 直接點擊
→ centered text modal
```

不要要求使用者先去左側找對應欄位才能編輯。

---

# E. TEXT MODAL — COMPLETE TOOLBAR

目前 Modal 只有：

- 字型
- 字級

功能不足。

本輪至少補：

- 字型
- 字級
- 粗體
- 斜體
- 底線
- 文字顏色
- 對齊：
  - 左
  - 中
  - 右
- 行距
- 字距

如果目前資料模型已安全支援其他既有文字屬性，也可一併放入，但不要為了「看起來完整」擴大資料 schema。

## Toolbar layout requirement

工具列盡量放在同一排。

目標：

```
字型 | 字級 | B | I | U | 顏色 | 左 中 右 | 行距 | 字距
```

要求：

- 桌面版優先
- 對齊
- 間距一致
- 不要變成很多直向堆疊的卡片
- 不要做成永久右側 inspector
- 必要時可讓低頻工具放 compact popover，但主工具列仍要整齊

---

# F. TEXT MODAL BEHAVIOR MUST REMAIN LETTER-LIKE

保持：

```
點 A4 文字
↓
Centered Modal
↓
修改
↓
A4 Live Preview
```

Apply：
- 保留變更
- 允許正常 autosave

Cancel：
- 完整還原開啟 Modal 前的狀態
- 不可殘留 dirty temporary state
- 不可因 autosave 已先寫入而造成取消失效

Backdrop：
- 不要因誤點背景就直接關閉

ESC：
- 不要靜默丟掉修改
- 優先維持明確 Apply / Cancel

---

# G. PLAIN-STRING SCHEMA LIMITATION

目前已知：

DM 是 plain-string schema。

本輪不要做 inline rich-text migration。

因此：

- 格式先作用於「整個文字物件」
- 不要求反白幾個字後單獨套不同格式
- 不修改資料模型成 rich-text spans

若要做 range formatting，列為後續獨立任務。

---

# H. ADD "整版面" ENTRY ABOVE "版面風格"

使用者目前沒有方便調整「整體版面」的入口。

在「版面風格」上方新增：

`整版面`

如果目前產品內已有語意相同但名稱不同的功能，優先沿用既有資料與邏輯，只調整入口與命名，不要重造 duplicate settings。

## 整版面負責

只放真正的 page/layout level 設定。

例如目前架構實際已有的：

- 正 / 反面
- 整體版面結構
- 整體頁面配置
- 版面位置 / 區塊配置
- 頁面層級可調參數

不要把單一文字字型放進整版面。

先依現有 capability 實作，不要發明不存在的版面引擎。

---

# I. "版面風格" RESPONSIBILITY CLEANUP

「版面風格」只負責視覺 preset / theme。

例如：

- 石墨滴色金
- 其他 style preset
- palette
- background
- card visual theme
- global visual treatment

不要再混：

- 品牌名稱字級
- 店名字級
- 姓名字級
- 電話字級
- slogan 字級
- card title 字級

這些文字尺寸全部移到「版面資料」。

---

# J. "版面資料" CONSOLIDATION

把目前散在「風格」底下的文字尺寸控制整合到「版面資料」。

例如目前實際存在的：

- 品牌名稱 size
- 店名 size
- 公司名稱 size
- headline size
- subheadline size
- 姓名 size
- 職稱 size
- 電話 size
- 服務項目 size
- LINE text size
- Slogan size
- card title size
- card number size
- card detail size
- card price size

以實際 schema 為準。

不要建立重複欄位。

如果目前同一值存在 camelCase / kebab / duplicate aliases，例如已看到可能有：
- titleJob / title-job
- lineText / line-text
- cardTitle / card-title

先確認真正 source of truth。
不要趁本輪做大規模 schema cleanup，除非不整理就會造成 UI 寫到錯欄位。

---

# K. NEW USER / NEW DRAFT DEFAULT STYLE

新使用者 / 新空白設計的：

`正面預設風格 = 石墨滴色金`

先找現有 preset 的真實 key / id。

禁止用顯示名稱硬寫成新的 duplicate preset。

## Safety

只影響：
- 新使用者初始資料
- 新 draft / 空白設計
- 沒有既有 user customization 的情況

不得覆蓋：
- 已存在 draft
- 已存在 style selection
- 既有使用者資料
- autosave restore

Existing user state must win.

---

# L. A4 DEFAULT TYPOGRAPHY TUNING

目前 A4 初始版面文字比例需要重新整理。

本輪調整的是「預設值」，不是強制覆寫每個既有 draft。

請在實際預覽中調整：

- 品牌名稱
- 店名
- 公司名稱
- headline
- subheadline
- 姓名
- 職稱
- 電話
- services
- line text
- slogan
- card title
- card number
- card detail
- card price

目標：

- 初次打開比例整齊
- 主次層級明確
- 不要電話過大、公司過小等失衡
- 不要造成文字溢出
- 正面「石墨滴色金」預設看起來完整
- 雙面 A4 都不要因預設字級而破版

調整後用實際 Local A4 preview 驗證，不要只看數值。

Existing customized draft must not be overwritten.

---

# M. CASE LIST — SIMPLIFY AGAIN

案件 Drawer 每列只需要：

`標題 + 金額`

不要預設顯示：

- 縮圖
- 房數
- 坪數
- 地址
- 行政區
- 其他 secondary metadata

除非目前 UI 必須有極少的狀態提示，否則不要塞回去。

## Title

案件清單標題：

`目前字級 + 2px`

以實際現有 CSS token / computed size 為準，不要猜 absolute size。

仍然：

- 最多 1–2 行
- 超出 ellipsis
- hover tooltip 顯示完整標題

## Amount

金額：
- 右側清楚顯示
- 不要搶過標題
- 對齊一致

## Drawer size

重新檢查：

- drawer width
- row height
- padding
- font size
- scroll area

目標是「正常、緊湊、好看」，不要因為簡化後反而留一堆空白。

A4 雙面預覽仍要盡量保持可視。

---

# N. NO RIGHT-SIDE PROPERTY PANEL

再次強調：

不要新增永久右側屬性面板。

本輪的 editing model 是：

```
直接點 A4 物件
→ centered modal / on-canvas controls
```

文字：
- centered modal

業務照片 Sticker：
- on-canvas handles

版面級：
- 左側「整版面」

Style：
- 左側「版面風格」

Data：
- 左側「版面資料」

---

# O. A4 DOUBLE-SIDED PREVIEW

必須保留：

- 正面
- 背面
- 目前雙面 A4 結構
- 現有 print dimensions
- 既有 front/back object state

本輪可以修改 A4 內「可點擊 / 可選取 / sticker transform / default typography」行為。

不要把雙面預覽改成單面。

不要為了 Drawer 而改壞 A4 page dimensions。

---

# P. LOCAL TESTS / REGRESSION

沿用目前測試架構。

至少補 / 更新對應測試：

## Portrait Sticker
- no photo → placeholder exists
- uploaded photo → placeholder removed
- sticker selected → handles visible
- scale changes
- rotation changes
- persisted state works
- front/back behavior if applicable

## Text Modal
- all layout-data editable text can open modal
- modal toolbar controls exist
- live preview
- Apply
- Cancel rollback
- autosave safety

## Style / Layout Data
- typography size controls no longer live under style
- layout-data owns text-size controls
- new default front style uses existing 石墨滴色金 preset
- existing saved style is not overwritten

## Case Drawer
- title + amount only
- no thumbnail
- title computed size increased by 2px
- long title ellipsis + tooltip

## Build
- existing frontend tests
- Vite build
- git diff --check

Do not reduce existing test coverage.

---

# Q. MANUAL LOCAL BROWSER ACCEPTANCE

用本機實際頁面驗證。

## 1. Portrait

沒有上傳：
- 原 placeholder 正常

上傳業務照片：
- placeholder 消失
- 只剩 Sticker
- 點 Sticker 有控制點
- 拖曳 / 放大 / 縮小 / 旋轉正常
- 不會旋轉後消失

## 2. Text

正面逐類型點擊：
- brand
- store
- headline
- name
- phone
- card text
- 其他 layout-data visible text

都能：
- 直接點
- 開 centered modal
- 修改
- live preview
- Apply / Cancel

背面有可編輯文字也做同樣驗證。

## 3. Modal toolbar

確認同一列 / 整齊顯示：
- font
- size
- bold
- italic
- underline
- color
- alignment
- line-height
- letter-spacing

## 4. Left UI

確認順序與責任：

```
整版面
版面風格
版面資料
```

至少語意與 UI responsibility 必須清楚，不可互相混用。

## 5. Default

建立真正「沒有既有 draft/customization」的新狀態驗證：
- 正面預設石墨滴色金
- Typography 整齊
- 不破版

再開既有 draft：
- 不得被新預設覆蓋

## 6. Cases

- 只顯示標題 + 金額
- title +2px
- 超長標題不撐高
- tooltip 可看完整
- drawer 整體大小正常

---

# HARD STOP CONDITIONS

遇到以下情況停止並回報，不自行擴大：

1. 要實作 Sticker 必須重做整套 object data schema
2. Apply / Cancel 會被 autosave 破壞
3. 「石墨滴色金」在 repo 內找不到現有 preset
4. 「瑞特專案」無法明確辨識
5. 版面資料的 text keys 有重大 duplicate / schema conflict
6. 需要修改 backend schema
7. 需要修改 Staging / Production
8. 發現 active writer 正在改同一批核心檔案
9. 會破壞既有 draft restore
10. 會改壞 A4 print dimensions

---

# FINAL REPORT

最後只回報：

1. repo / branch / HEAD
2. modified files
3. 大頭貼 Sticker 如何改
4. placeholder → uploaded Sticker 行為
5. on-canvas transform 實際支援項目
6. 哪些 A4 文字已可直接點擊
7. Text Modal 新增哪些 controls
8. toolbar 是否完成單排 / 整齊
9. 「整版面」入口實作方式
10. 「版面風格」與「版面資料」如何重新分工
11. 哪些文字尺寸設定已移到版面資料
12. 石墨滴色金 default 如何只套新使用者 / 新 draft
13. A4 default typography 調整內容
14. 案件清單如何簡化
15. title +2px 的實際前後值
16. tests
17. build
18. local browser acceptance
19. existing dirty WIP 是否完整保留
20. Docker changed = NO
21. Staging changed = NO
22. Production changed = NO
