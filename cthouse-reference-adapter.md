# CTHOUSE REFERENCE ADAPTER OWNER — LUNA EXECUTION PLAN

## 任務

接續目前已完成的 Extraction Core 與 YUCT Reference，正式完成「中信房屋 / CTHouse」來源適配。

工作目錄：
F:\00-Ticenpi-SaaS\ExtractionHub

參考專案只讀：
F:\00-Ticenpi-SaaS\TicenpiDM
F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591

## 已知前提

以下視為已完成，不要重做：

- CanonicalListing v2
- Adapter contract
- HTTP Transport contract
- Diagnostics contract
- YUCT Reference Adapter
- YUCT controlled validation
- YUCT REFERENCE READY

YUCT 是本輪的正式 Reference Pattern。

本輪不要：
- 重做 Core architecture
- 重做 Canonical schema
- 改 YUCT 架構
- 修改 DM / Post / 591
- 接產品 integration
- 決定 Plugin / Central / Hybrid
- 部署 Staging / Production

## 本輪唯一目標

把 CTHouse / 中信房屋完成到：

DISCOVERY
→ IMPLEMENTING
→ DEVELOPMENT PASS
→ FRESH UNSEEN PASS
→ GOLDEN PASS
→ CTHOUSE REFERENCE READY

如果遇到合法存取限制、登入牆、CAPTCHA 或來源結構問題，依 typed error 回報，不得繞過。

## 已知現況

上一輪 ExtractionHub 全套測試為：

52 passed / 1 failed

唯一失敗：

test_cthouse_golden_001_parse

已知症狀：
CTHouse features 被截斷。

Repo 目前已存在：
- CTHouse Python adapter
- CTHouse fixture
- CTHouse draft / historical mapping evidence
- golden test
- source registry entry

不要從零重寫 CTHouse。

第一件事是讀現有實作並定位既有 features truncation root cause。

# PHASE 0 — PREFLIGHT

先確認：

git branch
git HEAD
git status
git diff
git diff --cached
untracked files

保留現有 ExtractionHub WIP。

禁止：
git reset --hard
git clean
git stash
rebase

如果發現其他 active writer 正在修改相同 CTHouse / Core 檔案：

HARD STOP

回報：

CTHOUSE WRITE COLLISION

如果沒有 collision：

CTHOUSE PREFLIGHT PASS

# PHASE 1 — CURRENT CTHOUSE INVENTORY

只讀盤點現有：

- CTHouse adapter
- parser
- source detection
- transport selection
- fixtures
- golden tests
- YAML / draft mappings
- historical samples
- field mapping
- diagnostics
- image handling
- typed errors

建立：

CTHOUSE CURRENT MATRIX

至少列：

| Capability | Existing | Working | Gap |
|---|---|---|---|
| source detection | | | |
| URL validation | | | |
| redirect safety | | | |
| HTTP fetch | | | |
| browser requirement | | | |
| structured data | | | |
| SSR fallback | | | |
| title | | | |
| price | | | |
| address | | | |
| layout | | | |
| area | | | |
| floor | | | |
| age | | | |
| community | | | |
| parking | | | |
| salesperson | | | |
| phone | | | |
| store | | | |
| features | | | |
| images | | | |
| provenance | | | |
| diagnostics | | | |

不要因為欄位存在就標 Working。

要以 parser / fixture / real sample evidence 判斷。

# PHASE 2 — FIX EXISTING GOLDEN FAILURE FIRST

優先處理：

test_cthouse_golden_001_parse

先找 root cause。

不能只把 expected test 改成目前錯誤結果。

必須判斷：

1. parser 截斷
2. normalization 截斷
3. canonical conversion 截斷
4. fixture 本身資料不完整
5. features schema / type mismatch
6. other

修復規則：

Root cause
↓
Minimal generic fix
↓
Focused test
↓
Golden test
↓
Full ExtractionHub regression

禁止：
- 針對 fixture ID hardcode
- 為了測試通過刪欄位
- 降低 canonical correctness
- 破壞 YUCT

完成後要先達成：

CTHOUSE EXISTING GOLDEN PASS

才能進真實來源驗證。

# PHASE 3 — TRANSPORT PROBE

對中信真實公開案件先判斷：

HTTP 是否能完整取得必要資料？

優先順序：

安全 HTTP
→ structured state / JSON / SSR
→ browser only if truly required

禁止一開始就強制 Playwright。

如果 HTTP 可完整取得：
沿用 HTTP transport。

如果 HTTP 不足：
必須提供證據說明：
- 哪些資料缺
- 為什麼 browser 才拿得到

遇到：
LOGIN_REQUIRED
CAPTCHA_OR_CHALLENGE
ACCESS_DENIED

直接停止該路徑。

不得切 transport 繞過。

# PHASE 4 — DEVELOPMENT SAMPLE DISCOVERY

從合法公開可存取的中信房屋網站取得 development samples。

目標約 10 筆，不硬性。

樣本必須盡量涵蓋不同：

- 區域
- 價格
- 格局
- 樓層
- 屋齡
- 社區
- 車位
- 圖片數
- 房屋類型
- 資料完整 / 缺漏案例

不要全部選同類案件。

每筆記錄：

sample ID
source URL
final URL
transport
adapter
parse result
canonical coverage
diagnostics

# PHASE 5 — FIELD DISCOVERY

對每筆 Development Sample，自動找 Canonical 欄位。

優先來源：

1. structured JSON / state
2. JSON-LD
3. stable API response
4. SSR HTML
5. DOM
6. fragile CSS selector

不要第一個就用 CSS selector。

至少檢查：

listing_no
title
price
price_raw
address
city
district
layout
rooms
living
bath
area_total
floor
floor_total
age
community
parking
salesperson
phone
store_name
features
images
source
source_url

實際以目前正式 CanonicalListing v2 schema 為準。

# PHASE 6 — FIELD STATE

每欄必須使用：

PRESENT
NOT_PRESENT
EXTRACTION_FAILED
UNKNOWN

規則：

頁面真的沒有
→ NOT_PRESENT

頁面明明有，但 parser 沒抓到
→ EXTRACTION_FAILED

無足夠證據
→ UNKNOWN

禁止：

null = NOT_PRESENT

禁止：

0 = missing

# PHASE 7 — CTHOUSE ADAPTER IMPLEMENTATION

沿用現有 CTHouse adapter。

不要建立第二套 adapter。

補強：

- source-specific extraction
- normalization
- provenance
- typed errors
- adapter version
- diagnostics
- image semantics

Adapter 不認識：

DM
Post
591
auth
entitlement
platform_admin
subscription

# PHASE 8 — FEATURES TRUNCATION

這是本輪已知必修項。

中信 features 必須做到：

- 不截斷
- 保留原始值 evidence
- canonical normalized representation 符合 v2 schema
- raw / normalized 語意清楚
- 不因字串長度或 separator 造成資料遺失

建立至少：

1. existing golden regression
2. multiple feature values
3. empty feature source
4. missing feature source
5. unusual separator / whitespace

如果實際資料不支持某 case，不要硬造。

# PHASE 9 — IMAGE HANDLING

比照 YUCT Reference 規則。

至少區分：

discovered
validated
downloaded
skipped
failed

如果本輪沒有真正下載：
不能標 downloaded。

圖片至少檢查：

- URL safety
- scheme
- allowed host
- redirect safety
- HTTP success（若執行下載）
- content type（若執行下載）
- reasonable payload（若執行下載）

不要使用：

verify=False
ssl=False

# PHASE 10 — DIAGNOSTICS

每次中信 extraction 至少可以回答：

source
adapter
adapter version
schema version
input URL
final URL
transport
redirect history
timing
field states
field provenance
image counters
typed errors
overall success / partial / failure

禁止記：

password
Bearer token
完整 Cookie
session secret
API key
敏感 headers

# PHASE 11 — DEVELOPMENT AUTO-REPAIR

Development samples 若有缺欄：

AI 必須先自己調查：

structured data
SSR
DOM
existing draft mapping
existing adapter
historical evidence

然後：

Root cause
→ generic fix
→ focused test
→ rerun all development samples

不要立刻問使用者。

只有以下情況才問：

1. 欄位語意真的不確定
2. 網站需要合法登入
3. CAPTCHA / challenge
4. Canonical schema 發生真實語意衝突
5. 某欄是否值得 canonical 化需要使用者決策

需要人類決策時固定格式：

SOURCE:
CTHouse

SAMPLE:
<URL>

FIELD:
xxx

PAGE SHOWS:
xxx

CURRENT RESULT:
xxx

AI FOUND:
A.
B.
C.

RECOMMENDATION:
B

NEED USER DECISION:
A / B / C / IGNORE

# PHASE 12 — DEVELOPMENT GATE

Development Set 必須：

- source detection PASS
- transport PASS
- parse PASS
- required profile PASS
- field-state semantics PASS
- provenance PASS
- diagnostics PASS

才能輸出：

CTHOUSE DEVELOPMENT PASS

# PHASE 13 — FRESH UNSEEN VALIDATION

Development 完成後，再找另一批沒有用來開發的新案件。

目標約 10 筆。

這批標記：

FRESH_UNSEEN

在第一次跑之前：
禁止用這批資料改 parser。

第一次結果必須被保留。

如果全部 PASS：

CTHOUSE FRESH UNSEEN PASS

如果任何一筆 FAIL：

分類：

TRANSPORT_FAILURE
REDIRECT_FAILURE
SOURCE_DETECTION_FAILURE
STRUCTURED_DATA_VARIANT
SSR_VARIANT
DOM_VARIANT
FIELD_MAPPING_FAILURE
IMAGE_FAILURE
OTHER

如果拿 unseen sample 來修程式：

它就不再算 unseen。

修完後：

1. 重跑 Development
2. 重跑 consumed unseen 作 regression
3. 再找新的 fresh unseen sample 驗證

禁止：
修完同一批後仍宣稱 unseen PASS。

# PHASE 14 — GOLDEN

沿用 repo 現有 golden / publish policy。

不要建立第二套。

挑代表性 case 保存：

- structured case
- features case
- optional fields case
- image case
- missing / removed / unavailable case（如果來源實際存在且可安全保存）

Golden 要保護：

required fields
important optional fields
field states
features semantics
images semantics
diagnostics
typed errors
schema compatibility

# PHASE 15 — SUPPORTED / READY GATE

只有以下全部成立：

Source detection PASS
Transport PASS
Development PASS
Fresh unseen PASS
Canonical mapping PASS
Required fields PASS
Features PASS
Images PASS
Diagnostics PASS
Typed errors PASS
Golden regression PASS
Schema/version PASS
Full ExtractionHub regression no new failures

才能輸出：

CTHOUSE REFERENCE READY

如果有任何真正 blocker：

CTHOUSE REFERENCE BLOCKED

# PHASE 16 — FULL REGRESSION

至少執行：

1. CTHouse focused tests
2. Canonical tests
3. Transport tests
4. Diagnostics tests
5. Golden tests
6. YUCT focused tests
7. ExtractionHub full pytest
8. git diff --check

要求：

YUCT 必須保持 PASS。

如果本輪造成 YUCT regression：
本輪不能 READY。

# 文件

沿用既有：

_charter/charter.md
_charter/current-state.md
_charter/history.md

以及既有 docs。

不要建立第二套 SSOT。

更新：

charter
→ 只有穩定架構原則有變才更新

current-state
→ CTHouse 真實狀態

history
→ 本輪實作與驗證結果

不要把未通過 fresh unseen 的來源寫成 Supported。

# 最終回報格式

# CTHOUSE REFERENCE ADAPTER REPORT

## 1. Baseline

Branch:
HEAD:
Dirty state:
Collision:
PASS / BLOCKED

## 2. Existing Adapter

REUSE:
PATCH:
REPLACE:
DEFER:

## 3. Existing Golden Failure

Root cause:
Fix:
Regression:
Result:

## 4. Transport

Selected transport:
Reason:
Browser required:
YES / NO

## 5. Development Samples

Total:
PASS:
FAIL:

表格：

| ID | URL | Parse | Required | Features | Images | Result |

## 6. Canonical Coverage

Required:
Optional:
UNKNOWN:
EXTRACTION_FAILED:

## 7. Features

Raw:
Normalized:
Truncation:
PASS / FAIL

## 8. Images

Discovered:
Validated:
Downloaded:
Skipped:
Failed:

## 9. Fresh Unseen

First-run total:
PASS:
FAIL:

Consumed during repair:
YES / NO

Final fresh unseen:
PASS / FAIL

## 10. Golden

Result:
PASS / FAIL

## 11. Regression

CTHouse focused:
YUCT focused:
Full pytest:
git diff --check:

## 12. Remaining Gaps

如果沒有：

NONE

## 13. Final Result

只能：

CTHOUSE REFERENCE READY

或

CTHOUSE REFERENCE BLOCKED

# HARD STOP

完成 CTHouse 後停止。

不要開始：
住商
永慶
台灣房屋
591
樂屋
其他來源

不要修改：
DM
Post
591

不要部署：
Staging
Production

等待使用者下一個指令。
