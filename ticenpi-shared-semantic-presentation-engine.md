# TICENPI SHARED SEMANTIC + PRESENTATION ENGINE OWNER — ZERO-COLLISION PLAN

## 任務目標

目前 ExtractionHub 已有：

- YUCT REFERENCE READY
- CTHouse adapter / Golden / regression 已完成，但 live gate 受目前測試環境 IP 限制
- HBHousing / 住商 REFERENCE READY
- CanonicalListing v2
- FieldState / provenance / diagnostics / typed errors
- DM 已有 Shadow Compare / Admin Console 基礎

現在要建立真正「跨產品共用」的：

1. Semantic Normalizer
2. Semantic Comparator
3. Formatter Registry
4. Presentation Profile Contract
5. Product-specific Presentation Profiles 的接入邊界

核心架構：

各來源 Adapter
↓
Extraction Core
↓
CanonicalListing
↓
Shared Semantic Layer
├─ Normalizer
└─ Comparator
↓
Shared Presentation Engine
├─ Formatter Registry
└─ Presentation Profile Contract
↓
各產品 Profile
├─ DM
├─ Post
├─ 591
└─ future products

本任務的核心原則：

- Canonical semantic contract = 共用
- Presentation = 各產品可不同
- Comparator 不得依賴 UI display string
- Raw evidence 必須保留
- 不同產品不必使用相同顯示方式
- 不把 Presentation 邏輯塞回 Adapter
- 不把 DM 的顯示規則寫死成全平台規則

---

# 零碰撞執行策略

## 允許平行，但只允許 READ-ONLY 平行

為了把 write collision 風險降到最低：

可以平行執行：

A. ExtractionHub semantic / canonical audit
B. DM Shadow / display audit
C. Post / 591 consumer requirement audit

以上三個工作：

- 只能讀
- 不准改檔
- 不准 commit
- 不准 migration
- 不准自動 fix

三個 audit 完成後，才進入單一 Writer 實作。

## 禁止平行 Writer

本任務任何時間只允許一個 writer 修改 shared semantic / presentation 相關檔案。

如果偵測到：

- 其他 Session 正在改相同 repo / 相同檔案
- 其他 branch/worktree 對相同 shared package 有 active writer
- repo dirty changes 無法判斷 ownership

HARD STOP。

不要用 worktree / branch 來合理化同檔案平行寫入。

本任務要的是：

PARALLEL READ
→ SINGLE WRITER IMPLEMENTATION

而不是：

PARALLEL WRITE

---

# 工作目錄

優先調查：

F:\00-Ticenpi-SaaS\ExtractionHub
F:\00-Ticenpi-SaaS\TicenpiDM
F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591

如果實際存在共用 package / platform shared library：

必須先找出。

不要自行假設：

TicenpiShared
shared/presentation
shared/core

一定存在。

不要為了本任務直接新建新 repo。

---

# PHASE 0 — GLOBAL PREFLIGHT

對所有相關 repo 先記錄：

- repo path
- branch
- HEAD
- git status
- git diff
- git diff --cached
- untracked files
- current active writer evidence

禁止：

git reset --hard
git clean
git stash
rebase

保留既有 WIP。

輸出：

## GLOBAL WRITE MAP

| Repo | Branch | HEAD | Dirty | Active Writer | Writable This Task |

規則：

- ExtractionHub / shared package：只有單一 writer phase 才可寫
- DM / Post / 591：本輪先只讀；除非後面明確進 consumer integration phase
- 如果目前有其他 Session 在寫同 repo，該 repo 本輪保持 read-only

---

# PHASE 1A — READ-ONLY AUDIT: EXTRACTIONHUB

只讀。

盤點：

- CanonicalListing v2 schema
- Raw vs normalized fields
- FieldState semantics
- provenance model
- YUCT normalized values
- CTHouse normalized values
- HBHousing normalized values
- existing normalizer helpers
- existing comparator helpers
- existing formatting helpers
- existing package boundaries
- tests / golden patterns

特別找真實例子：

LAYOUT
- 3房2廳2衛
- 3房(室)2廳2衛
- spacing / slash variants

PRICE
- 萬
- TWD integer
- comma variants

AREA
- 坪字
- number only

FLOOR
- 8樓/15樓
- 8F/15F
- source-specific representation

PARKING
- 坡平
- 坡道平面
- 無車位
- unknown

ADDRESS
- whitespace
- punctuation
- administrative naming

輸出：

EXTRACTION SEMANTIC MATRIX

| Field | Raw Examples | Existing Canonical | Existing Normalizer | Gap |

不修改程式。

---

# PHASE 1B — READ-ONLY AUDIT: DM

只讀。

找出：

- Shadow Legacy Mapper
- Shadow Comparator
- Shadow Recorder
- Shadow Admin API
- Shadow Console
- current comparison statuses
- current display strings
- current per-field UI
- current admin settings mechanism
- existing profile/config mechanism

特別確認：

目前是否把：

3房2廳2衛
vs
3房(室)2廳2衛

判為 MISMATCH。

找出 root cause：

- raw string equality
- partial normalize
- mapper issue
- UI-only display issue
- persistence model issue

輸出：

DM SHADOW GAP MATRIX

不修改程式。

---

# PHASE 1C — READ-ONLY AUDIT: POST + 591

只讀。

目的：

不要現在接入，而是先知道未來 consumer 對「同一 Canonical value」的顯示需求是否不同。

盤點：

- layout display
- price display
- area display
- floor display
- age display
- parking display
- address display

輸出：

PRODUCT PRESENTATION REQUIREMENTS

| Field | DM | Post | 591 | Shared Semantic? | Product Display? |

如果某產品目前沒有明確格式：

標記：

UNKNOWN

不要猜。

---

# PHASE 2 — ARCHITECTURE DECISION

三個 read-only audit 完成後，才決定共用層放哪裡。

優先順序：

1. 已存在且真正被多產品共用的 shared package
2. ExtractionHub 內可獨立引用、與 FastAPI / adapter 解耦的純 Python module/package
3. 最小新增 shared package

禁止：

- 為了漂亮架構直接新建 microservice
- 新建獨立 repo，除非現有 monorepo / package 結構真的無法安全共用
- 把 Presentation Engine 綁到 FastAPI
- 把 Presentation Engine 綁到 DM

輸出：

SHARED LOCATION DECISION

Chosen location:
Why:
Alternatives rejected:
Import boundary:
Deployment impact:

如果 location 無法安全決定：

HARD STOP

---

# PHASE 3 — FORMAL SEMANTIC CONTRACT

建立共用 Semantic Contract。

## Comparison statuses

固定支援：

EXACT_MATCH
NORMALIZED_MATCH
MISMATCH
UNCOMPARABLE
LEGACY_MISSING
CORE_MISSING
BOTH_MISSING

語意：

EXACT_MATCH
→ canonical semantic 一致，且 display/raw 在安全 normalization 後等價

NORMALIZED_MATCH
→ raw/display 不同，但 canonical semantic value 相同

MISMATCH
→ canonical semantic value 不同

UNCOMPARABLE
→ 證據不足，不可安全比較

UNKNOWN vs UNKNOWN
→ 不得自動 MATCH

NOT_PRESENT vs NOT_PRESENT
→ 只有來源證據足夠才能視為 equivalent

---

# PHASE 4 — NORMALIZER REGISTRY

建立共用、typed、testable 的 normalizer registry。

第一版至少支援：

layout
price
area
floor
age
address
parking

但每個 normalizer 必須以真實 sample evidence 為基礎。

不要做過度推測。

## Layout

以下應能正規化：

3房2廳2衛
3房(室)2廳2衛
3房 2廳 2衛
3 房 / 2 廳 / 2 衛

canonical semantic：

rooms=3
living=2
bath=2

## Price

例如：

1680萬
1,680 萬
16800000元
NT$16,800,000

canonical semantic：

16800000 TWD

## Area

例如：

42.6坪
42.60 坪
42.6

canonical semantic：

42.6

## Floor

例如：

8樓/15樓
8 / 15
8F / 15F

canonical semantic：

floor=8
floor_total=15

## Address

只做保守 normalize：

- Unicode
- whitespace
- known equivalent punctuation
- clearly equivalent administrative formatting

不能靠 fuzzy match 把不同地址判成相同。

## Parking

只有有明確 evidence / ontology 才 normalize。

例如：

坡平
坡道平面

可映射到同 token，前提是真實產品資料已確認同義。

不確定：

UNCOMPARABLE

不要強行 MATCH。

---

# PHASE 5 — SHARED COMPARATOR

Comparator input：

- raw evidence
- canonical semantic value
- FieldState
- provenance

Comparator output：

- status
- normalized values
- reason
- comparable boolean
- evidence summary

Comparator 不得：

- 根據 DM UI 字串判斷
- 修改 raw
- 修改 canonical
- 執行 product formatter
- 猜缺值

---

# PHASE 6 — SHARED PRESENTATION ENGINE

Presentation Engine input：

Canonical semantic value
+
Presentation Profile
+
Formatter ID
+
typed options

output：

display string

不得改 Canonical value。

第一版 Formatter Registry 至少：

## Layout

layout_zh_compact
→ 3房2廳2衛

layout_zh_spaced
→ 3房 / 2廳 / 2衛

layout_slash
→ 3/2/2

## Price

至少支援：

price_wan_compact
price_twd

不要發明產品尚未需要的 formatter。

## Area

至少支援：

area_ping
area_number

## Floor / Age / Parking

依 read-only audit 的真實產品需求決定。

---

# PHASE 7 — PRESENTATION PROFILE CONTRACT

正式定義：

Global Default
↓
Product Profile
↓
Optional Template Override

優先權：

Template Override
> Product Profile
> Global Default

但如果目前產品不存在成熟 Template Override：

第一版只做：

Global Default
+
Product Profile

Template Override = DEFERRED

不要為了本任務造大型 template system。

---

# PHASE 8 — PRODUCT PROFILE MODEL

Profile 應保存：

product
field
formatter_id
typed options
version
enabled

例如：

product=dm
field=layout
formatter_id=layout_zh_compact
version=1

不要保存：

- arbitrary Python
- arbitrary JS
- eval string
- executable expression

Formatter 必須 allowlisted。

---

# PHASE 9 — FIRST CONSUMER = DM

共用層完成後，DM 作第一個 consumer。

DM 不擁有 Engine。

DM 只擁有：

DM Presentation Profile
+
DM adapter / bridge

Shadow Console 改為顯示：

| DM Legacy | Status | Extraction Core Raw | Canonical | DM Final Display |

必要案例：

Legacy：
3房2廳2衛

Core Raw：
3房(室)2廳2衛

Canonical：
3 / 2 / 2

Status：
NORMALIZED_MATCH

DM Final Display：
3房2廳2衛

如果切換 DM formatter：

layout_slash

DM Final Display：
3/2/2

Comparator status 必須仍是：

NORMALIZED_MATCH

這是本輪最重要 acceptance test。

---

# PHASE 10 — POST / 591

本輪預設：

READ-ONLY REQUIREMENTS ONLY

不要直接修改 Post / 591。

只有 DM consumer 驗證完成，而且 shared API 穩定後，最終報告才提出：

POST INTEGRATION PLAN
591 INTEGRATION PLAN

等待下一輪核准。

---

# PHASE 11 — PERSISTENCE / SETTINGS

先重用現有設定機制。

如果平台已有：

- product settings
- config registry
- admin settings
- DB settings table

優先 reuse。

如果沒有：

先用最小 typed config。

不要一開始就做大型 DB migration。

若確實需要 migration：

必須先證明：

- product profile 需要 runtime editable
- existing config 不足

再提 migration plan。

本輪不要碰 Production DB。

---

# PHASE 12 — ADMIN UI

只在 DM 第一個 consumer 驗證階段加入最小 UI。

至少：

DM Layout Display

選項：

- 3房2廳2衛
- 3房 / 2廳 / 2衛
- 3/2/2

保存：

formatter_id

不是房源-specific 字串。

最好提供 Preview：

Canonical：
rooms=3
living=2
bath=2

Selected formatter：
layout_zh_compact

Preview：
3房2廳2衛

platform_admin 才可修改。

normal user / unauthenticated：
DENY

---

# PHASE 13 — SHADOW REPORT AGGREGATION

Semantic match：

EXACT_MATCH
+
NORMALIZED_MATCH

都算 match。

但統計分開保留：

exact_match_count
normalized_match_count
mismatch_count
uncomparable_count
legacy_missing_count
core_missing_count

semantic_match_count =
exact_match_count + normalized_match_count

semantic_match_rate =
semantic_match_count / comparable_count

UNCOMPARABLE / BOTH_MISSING 不得污染 denominator。

---

# PHASE 14 — TEST MATRIX

## Semantic Normalizer

layout
price
area
floor
age
address
parking

## Comparator

1.
3房2廳2衛
vs
3房(室)2廳2衛
→ NORMALIZED_MATCH

2.
3房2廳2衛
vs
3房2廳1衛
→ MISMATCH

3.
UNKNOWN vs UNKNOWN
→ 不得自動 MATCH

4.
NOT_PRESENT vs NOT_PRESENT
→ provenance-aware

## Presentation

Canonical 3/2/2

layout_zh_compact
→ 3房2廳2衛

layout_zh_spaced
→ 3房 / 2廳 / 2衛

layout_slash
→ 3/2/2

## Isolation

改 formatter：
→ comparator status 不變

改 presentation profile：
→ canonical 不變

改 semantic normalizer：
→ raw evidence 不變

## Regression

YUCT
CTHouse
HBHousing
Canonical
Transport
Diagnostics
Images
DM Shadow
DM auth

不得 regression。

---

# PHASE 15 — PERFORMANCE

量測：

- normalize cost
- compare cost
- format cost
- Shadow request added latency
- batch report aggregation impact

這一層應為 CPU-light 純邏輯。

若引入：

Redis
microservice
remote RPC
browser

都視為架構過度。

除非有實證必要，否則禁止。

---

# PHASE 16 — DOCUMENTATION

沿用現有 SSOT。

ExtractionHub：

_charter/charter.md
_charter/current-state.md
_charter/history.md

DM：

沿用現有 handoff / current-state / history 規則。

只更新必要文件。

不要建立第二套架構文件系統。

---

# PHASE 17 — WRITE ISOLATION CHECK

實作完成前再次確認：

- 沒有其他 Session 修改 shared engine files
- 沒有同檔案 parallel writer
- Post / 591 本輪沒有 write
- Production 沒有 write
- DB Production 沒有 write

輸出：

PARALLEL READ AUDITS:
PASS / FAIL

SINGLE WRITER IMPLEMENTATION:
PASS / FAIL

WRITE COLLISION:
NONE / FOUND

---

# FINAL REPORT FORMAT

# TICENPI SHARED SEMANTIC + PRESENTATION ENGINE REPORT

## 1. Baseline

Repositories:
Branches:
HEADs:
Dirty states:
Active writers:

## 2. Parallel Safety

Read-only audits:
PASS / FAIL

Parallel writers:
0

Single writer:
YES / NO

Collision:
NONE / FOUND

## 3. Shared Location

Chosen:
Reason:
Import boundary:
Deployment impact:

## 4. Semantic Contract

Statuses:
FieldState handling:
Raw preservation:
Provenance handling:

## 5. Normalizers

Layout:
Price:
Area:
Floor:
Age:
Address:
Parking:

## 6. Comparator

Input:
Output:
Semantic match rules:
PASS / FAIL

## 7. Presentation Engine

Formatter Registry:
Profile Contract:
Global defaults:
Product profile:
Template override:
ENABLED / DEFERRED

## 8. DM Consumer

Legacy:
3房2廳2衛

Core:
3房(室)2廳2衛

Canonical:
3 / 2 / 2

Status:
NORMALIZED_MATCH / FAIL

DM compact:
3房2廳2衛

DM slash:
3/2/2

Comparator unchanged:
PASS / FAIL

## 9. Admin

platform_admin:
PASS / FAIL

normal user:
DENY / FAIL

unauthenticated:
DENY / FAIL

## 10. Aggregation

Exact:
Normalized:
Mismatch:
Uncomparable:
Denominator:
PASS / FAIL

## 11. Regression

YUCT:
CTHouse:
HBHousing:
Canonical:
Transport:
Diagnostics:
Images:
DM Shadow:
Full tests:
git diff --check:

## 12. Performance

Normalize:
Compare:
Format:
Added latency:

## 13. Post / 591 Next Integration

Post:
PLAN READY / UNKNOWN

591:
PLAN READY / UNKNOWN

No writes:
PASS / FAIL

## 14. Remaining Gaps

NONE
或逐項列出。

## 15. Final Result

只能：

TICENPI SHARED SEMANTIC + PRESENTATION READY

或

TICENPI SHARED SEMANTIC + PRESENTATION BLOCKED

---

# HARD STOP

本輪完成後不要：

- 直接修改 Post
- 直接修改 591
- 開始永慶 Adapter
- 開始台灣房屋 Adapter
- 發布來源 Registry
- 部署 Production
- 修改 Production DB
- 刪除 DM Legacy scraper
- 切 Extraction Core Primary

等待使用者核准下一步。
