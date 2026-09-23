# DM SHADOW PRESENTATION PROFILE OWNER — PARALLEL-SAFE IMPLEMENTATION PLAN

## 任務目標

目前 ExtractionHub 正在進行「住商不動產 Reference Adapter / live extraction validation」。

本任務必須可以與該工作**平行執行**，因此：

- 只修改 DM repo
- ExtractionHub 僅可讀
- 不修改任何 Adapter / Canonical / Transport / Diagnostics / Registry
- 不修改住商相關檔案
- 不修改 Post / 591 / ORC / Letter
- 不部署 Production

本輪目標：

> 把 DM Shadow Compare 從「字串直接比對」升級成「Canonical semantic compare + DM Presentation Profile」，並在 Shadow Console 顯示 Legacy / Status / Core Raw / Canonical / DM Final Display。

核心原則：

1. Comparator 回答「資料語意是否相同」
2. Presentation 回答「DM 最後要怎麼顯示」
3. Extraction Core 不負責 DM 顯示格式
4. Raw 必須保留
5. Canonical 必須作為比較標準
6. DM 顯示格式可獨立於其他產品

---

## 工作目錄

主要修改：

F:\00-Ticenpi-SaaS\TicenpiDM

只讀參考：

F:\00-Ticenpi-SaaS\ExtractionHub

禁止修改：

F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591

---

# PHASE 0 — PREFLIGHT / COLLISION CHECK

先確認：

DM：
- branch
- HEAD
- git status
- git diff
- git diff --cached
- untracked files

ExtractionHub：
- branch
- HEAD
- git status

確認目前是否有其他 active writer 正在修改 DM Shadow Console / comparator / admin extraction-shadow 相關檔案。

如果有：

HARD STOP

輸出：

DM PRESENTATION WRITE COLLISION

如果只有 ExtractionHub / 住商 task 正在執行，且沒有修改 DM：

允許平行。

輸出：

PARALLEL SAFE WITH HBHOUSING = YES

禁止：

git reset --hard
git clean
git stash
rebase

保留所有既有 WIP。

---

# PHASE 1 — TRACE CURRENT SHADOW FLOW

先找出目前真實 DM Shadow integration：

Legacy result
↓
Core result
↓
Legacy Mapper
↓
Comparator
↓
Shadow Recorder
↓
Admin API
↓
/admin/extraction-shadow

確認：

- comparator 現在比較哪些欄位
- 哪些欄位目前是 raw string equality
- 哪些欄位已經有 normalized compare
- current persistence schema
- current report API
- current admin UI
- current feature flags

不要先改 code。

輸出：

CURRENT SHADOW COMPARATOR MATRIX

至少：

| Field | Legacy Raw | Core Raw | Current Compare | Canonical Available | Needs Semantic Compare |

---

# PHASE 2 — DEFINE COMPARISON CONTRACT

建立 DM Shadow 專用的 semantic comparison contract。

不要改 ExtractionHub Canonical schema。

Comparison status 固定：

EXACT_MATCH
NORMALIZED_MATCH
MISMATCH
UNCOMPARABLE
LEGACY_MISSING
CORE_MISSING
BOTH_MISSING

語意：

EXACT_MATCH
→ raw/display 值在安全 normalize 後仍完全相同

NORMALIZED_MATCH
→ raw/display 不同，但 Canonical semantic value 相同

MISMATCH
→ Canonical semantic value不同

UNCOMPARABLE
→ 缺少足夠語意資訊，不能安全比較

UNKNOWN vs UNKNOWN
→ 不得自動視為 MATCH

NOT_PRESENT vs NOT_PRESENT
→ 只有 provenance 足夠時才可視為語意相同

---

# PHASE 3 — LAYOUT SEMANTIC NORMALIZATION

先把「格局」做成正式 Reference Implementation。

以下必須視為相同語意：

3房2廳2衛
3房(室)2廳2衛
3房 2廳 2衛
3 房 / 2 廳 / 2 衛

Canonical compare：

rooms = 3
living = 2
bath = 2

結果：

NORMALIZED_MATCH

不要用字串取代規則硬湊成相同。

必須先 parse 成結構：

rooms
living
bath

再 compare。

保留：

legacy_raw
core_raw
legacy_normalized
core_normalized

---

# PHASE 4 — GENERAL SEMANTIC NORMALIZERS

建立可測試、可擴充的 DM Shadow normalizer registry。

優先支援：

layout
price
area
floor
age
address
parking

範例：

PRICE

1680萬
1,680 萬
16800000元
NT$16,800,000

→ canonical semantic：
16800000 TWD

AREA

42.6坪
42.60 坪
42.6

→ canonical semantic：
42.6

FLOOR

8樓/15樓
8 / 15
8F / 15F

→ canonical semantic：
floor=8
floor_total=15

ADDRESS

只允許保守 normalize：
- Unicode / whitespace
- 已知等價標點
- 安全行政區格式整理

不得因為模糊地址推測而誤判 MATCH。

PARKING

只有有明確 ontology / mapping 時才 normalize。
例如「坡平」與「坡道平面」可映射到同一正式 token，前提是現有產品語意確認一致。

不確定：
→ UNCOMPARABLE
而不是強行 MATCH。

---

# PHASE 5 — PRESENTATION LAYER

新增 DM Presentation Profile。

這一層只負責：

> Canonical value → DM display string

不要影響 comparator。

不要影響 Extraction Core。

第一版至少支援：

layout
price
area
floor
age
parking

例如：

layout formatter：

layout_zh_compact
→ 3房2廳2衛

layout_zh_spaced
→ 3房 / 2廳 / 2衛

layout_slash
→ 3/2/2

DM default：

layout = layout_zh_compact

---

# PHASE 6 — PRESENTATION PROFILE MODEL

第一版先做到：

Global Default
↓
Product Profile = DM
↓
Optional Template Override（只有 repo 現有 template abstraction 能自然接上才做）

優先權：

Template Override
> DM Product Profile
> Global Default

如果 repo 沒有成熟 template override 機制：

DEFER

不要為本任務新造大型模板系統。

---

# PHASE 7 — PROFILE STORAGE

先調查 DM 現有：

settings
config
DB
admin settings
feature config

如果已有適合的設定儲存：
→ REUSE

如果沒有：

建立最小、typed、allowlisted profile config。

禁止：

- 任意 JS
- 任意 Python
- eval
- user-provided executable formatter

formatter 必須為 allowlist ID。

例如：

product = dm
field = layout
formatter = layout_zh_compact
options = {}
version = 1

若本輪不需要 DB 就不要硬建 DB migration。

若 config file / existing settings 足夠：
優先簡單方案。

---

# PHASE 8 — SHADOW CONSOLE UI

將 Shadow Console 每欄改成：

| DM Legacy | Status | Extraction Core Raw | Canonical | DM Final Display |

例如：

| 3房2廳2衛 | NORMALIZED_MATCH | 3房(室)2廳2衛 | 3 / 2 / 2 | 3房2廳2衛 |

Status 必須清楚顯示：

EXACT MATCH
NORMALIZED MATCH
MISMATCH
UNCOMPARABLE
LEGACY MISSING
CORE MISSING
BOTH MISSING

不要把 NORMALIZED_MATCH 顯示成錯誤或紅色 mismatch。

---

# PHASE 9 — ADMIN DISPLAY FORMAT SELECTION

在 platform_admin Shadow Console 提供「顯示方案」設定。

至少先支援 layout。

例如：

DM Layout Display

- 3房2廳2衛
- 3房 / 2廳 / 2衛
- 3/2/2

Admin 選擇後：

保存的是 formatter ID：

layout_zh_compact
layout_zh_spaced
layout_slash

不是保存房源-specific 字串。

改設定後：

所有 DM canonical layout 都依該 formatter 顯示。

不要一間房存一個 display choice。

---

# PHASE 10 — PREVIEW BEFORE SAVE

設定 UI 最好提供 Preview：

Canonical：

rooms=3
living=2
bath=2

選擇：

layout_zh_compact

Preview：

3房2廳2衛

切換：

layout_slash

Preview：

3/2/2

若成本太高，可先做最小 preview。

---

# PHASE 11 — REPORT AGGREGATION

Shadow report 的 match rate 要把：

EXACT_MATCH
NORMALIZED_MATCH

都算為 semantic match。

但後台統計必須保留兩者分開：

exact_match_count
normalized_match_count
mismatch_count
uncomparable_count

semantic_match_count =
exact_match_count + normalized_match_count

semantic_match_rate =
semantic_match_count / comparable_count

comparable_count 不得包含：

UNCOMPARABLE
BOTH_MISSING

missing semantics 依現有 contract 判斷，不要亂加 denominator。

---

# PHASE 12 — PERSISTENCE

如果既有 Shadow Recorder 已存 per-field comparison：

擴充但不要破壞舊紀錄。

建議 per-field 可表達：

field
legacy_raw
core_raw
legacy_normalized
core_normalized
comparison_status
presentation_formatter
final_display

不要保存：

JWT
cookie
Authorization header
API key
session secret

舊紀錄無 canonical normalized 值時：
允許 legacy fallback / null。
不要 backfill 猜值。

---

# PHASE 13 — PRODUCT BOUNDARY

本輪只實作：

DM Presentation Profile

但 code structure 要允許未來：

POST_PROFILE
591_PROFILE
LETTER_PROFILE
其他產品

不要現在修改其他 repo。

不要要求所有產品相同顯示。

正式原則：

Canonical semantic contract = shared

Presentation profile = product-specific

---

# PHASE 14 — TESTS

至少新增：

## Layout

1.
Legacy: 3房2廳2衛
Core: 3房(室)2廳2衛
→ NORMALIZED_MATCH

2.
Legacy: 3房2廳2衛
Core: 3房2廳1衛
→ MISMATCH

3.
Legacy: 3 房 / 2 廳 / 2 衛
Core: 3房2廳2衛
→ NORMALIZED_MATCH

4.
Unknown / malformed
→ UNCOMPARABLE

## Price

1680萬
vs
16800000
→ NORMALIZED_MATCH

1680萬
vs
1580萬
→ MISMATCH

## Area

42.60坪
vs
42.6
→ NORMALIZED_MATCH

## Presentation

Canonical 3/2/2

layout_zh_compact
→ 3房2廳2衛

layout_zh_spaced
→ 3房 / 2廳 / 2衛

layout_slash
→ 3/2/2

## Isolation

Presentation formatter 改變：
→ comparator result 不得改變

Comparator normalize 改變：
→ raw evidence 不得被覆蓋

## Authorization

platform_admin
→ 可讀 / 改 Presentation Profile

一般 user
→ 不可改

unauthenticated
→ 不可改

---

# PHASE 15 — LOCAL VALIDATION

至少用目前 Shadow Console 的真實案例驗：

Legacy：
3房2廳2衛

Core：
3房(室)2廳2衛

要求：

Status：
NORMALIZED_MATCH

Canonical：
3 / 2 / 2

DM Final Display：
3房2廳2衛

然後切換 formatter：

layout_slash

要求：

Status 仍然：
NORMALIZED_MATCH

DM Final Display：
3/2/2

這證明：

Comparator 與 Presentation 已解耦。

---

# PHASE 16 — PARALLEL SAFETY VERIFICATION

結束前再次確認：

ExtractionHub 的住商 task 沒有被本任務修改。

執行：

git status / diff

ExtractionHub：

NO WRITES FROM THIS TASK

若發現本任務碰到 ExtractionHub：

BLOCKED

回退「本任務自己的」ExtractionHub 修改，但不得 reset 他人的 WIP。

---

# PHASE 17 — REGRESSION

執行 DM 相關：

- Shadow comparator tests
- Shadow report aggregation tests
- Presentation formatter tests
- Admin auth tests
- relevant backend tests
- relevant frontend tests
- existing DM smoke / invariants where applicable
- git diff --check

不得要求 ExtractionHub 住商 task 完成才能通過本任務。

本任務與住商應維持 repo-level isolation。

---

# FINAL REPORT FORMAT

# DM SHADOW PRESENTATION PROFILE REPORT

## 1. Baseline

DM Branch:
DM HEAD:
DM Dirty:
ExtractionHub HEAD:
Parallel safe:
YES / NO

## 2. Current Comparator

Raw string fields:
Existing normalized fields:
Changed fields:

## 3. Semantic Comparator

Statuses:
Layout:
Price:
Area:
Floor:
Age:
Address:
Parking:

## 4. Presentation Engine

Location:
DM profile:
Formatters:
Storage:

## 5. Shadow Console

Columns:

DM Legacy
Status
Extraction Core Raw
Canonical
DM Final Display

Result:
PASS / FAIL

## 6. Admin Selection

platform_admin:
PASS / FAIL

Normal user denied:
PASS / FAIL

Preview:
PASS / DEFERRED

## 7. Required Example

Legacy:
3房2廳2衛

Core:
3房(室)2廳2衛

Status:
NORMALIZED_MATCH / FAIL

Canonical:
3 / 2 / 2

DM default display:
3房2廳2衛

Slash display:
3/2/2

Comparator unchanged after formatter switch:
PASS / FAIL

## 8. Aggregation

Exact matches:
Normalized matches:
Mismatches:
Uncomparable:

Semantic match formula:
PASS / FAIL

## 9. Tests

Comparator:
Presentation:
Aggregation:
Admin auth:
Backend:
Frontend:
git diff --check:

## 10. ExtractionHub Isolation

ExtractionHub writes from this task:
NONE / FOUND

HBHousing task affected:
NO / YES

## 11. Final Result

只能：

DM SHADOW PRESENTATION READY

或

DM SHADOW PRESENTATION BLOCKED

---

# HARD STOP

本任務完成後不要：

- 修改 ExtractionHub
- 修改住商 Adapter
- 修改 YUCT / CTHouse
- 修改 Post / 591 / ORC / Letter
- 抽成全平台 shared package
- 部署 Production
- 切 Core Primary
- 刪 Legacy scraper

等 DM 實際驗證穩定後，再決定是否抽成共用 Ticenpi Presentation Engine。
