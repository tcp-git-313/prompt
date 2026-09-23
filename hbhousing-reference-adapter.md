# 住商不動產 REFERENCE ADAPTER OWNER — LUNA EXECUTION PLAN

## 任務目標

接續目前已完成的 Extraction Core、YUCT Reference，以及已完成診斷但受環境 IP 限制的 CTHouse。

本輪唯一目標：

> 正式完成「住商不動產」來源 Adapter，從既有 Draft / Mapping / Registry 成果出發，完成 Development、Fresh Unseen、Golden 與完整回歸，達到 REFERENCE READY。

本輪不要：

- 重做 Extraction Core
- 重做 CanonicalListing
- 重做 Adapter / Transport / Diagnostics contract
- 修改 YUCT
- 修改 CTHouse
- 修改 DM / Post / 591
- 決定 Plugin / Central / Hybrid
- 部署 Staging / Production
- 同時開始永慶、台灣房屋、591、樂屋等其他來源

---

## 工作目錄

主要：

F:\00-Ticenpi-SaaS\ExtractionHub

參考專案，只讀：

F:\00-Ticenpi-SaaS\TicenpiDM
F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591

實際 repo / folder / source naming 以工作樹為準。

不要猜網站 domain、adapter 檔名或 registry key。

---

# 已知前提

以下視為既有成果：

- CanonicalListing v2 已建立
- FieldState 已建立：
  - PRESENT
  - NOT_PRESENT
  - EXTRACTION_FAILED
  - UNKNOWN
- Adapter contract 已建立
- HTTP Transport contract 已建立
- Diagnostics contract 已建立
- typed errors 已建立
- YUCT REFERENCE READY
- CTHouse features golden defect 已修
- CTHouse live gate 目前因 environment-specific ACCESS_DENIED 暫停
- ExtractionHub 現有住商 Draft / structured mapping / 歷史 sample evidence 已存在

本輪不能因為 CTHouse 被擋，就修改共同 Transport 安全政策去遷就來源。

---

# PHASE 0 — PREFLIGHT

先確認：

git branch
git HEAD
git status
git diff
git diff --cached
untracked files

保留現有 WIP。

禁止：

git reset --hard
git clean
git stash
rebase

確認目前是否有其他 active writer 正在修改：

- 住商 adapter / draft
- registry
- core
- transport
- canonical
- diagnostics
- tests / fixtures

若有真正 collision：

HARD STOP

輸出：

HBHOUSING WRITE COLLISION

若沒有：

HBHOUSING PREFLIGHT PASS

---

# PHASE 1 — 找到「住商」在 Repo 裡真正的身份

不要直接假設 source id 一定叫：

hbhousing
住商
housefun

請從現有：

- sites.yaml / source registry
- draft adapters
- mapping rules
- fixtures
- old docs
- history
- tests

查出：

1. 正式 source id
2. 品牌名稱
3. domain / aliases
4. 是否已有 adapter
5. 是否只有 YAML / Draft
6. 是否已有 samples / fixtures
7. 是否已有 publish status
8. 是否已有 historical field-state

建立：

HBHOUSING SOURCE IDENTITY

格式：

Source ID:
Brand:
Domains:
Aliases:
Existing adapter:
Draft:
Samples:
Registry status:

若既有 source identity 有衝突：

先修 registry identity，不要建立第二個住商來源。

---

# PHASE 2 — EXISTING INVENTORY

完整盤點住商現有成果。

分類：

REUSE
PATCH
REPLACE
DEFER

至少檢查：

- source detection
- URL validation
- redirect policy
- transport candidate
- structured data mapping
- SSR mapping
- DOM mapping
- canonical mapping
- features
- community
- parking
- salesperson
- phone
- store
- images
- provenance
- diagnostics
- typed errors
- fixtures
- golden
- draft / publish gate

原則：

能沿用就沿用。

不要因為目前沒有 Python adapter 就把 draft 成果當不存在。

---

# PHASE 3 — TRANSPORT PROBE

先用合法公開案件做最小探測。

目的：

回答：

> 住商用安全 HTTP 是否能完整取得需要的房源資料？

優先順序：

STANDARD_HTTP
↓
structured state / JSON / SSR
↓
Browser only if truly required

禁止：

- 一開始就用 Playwright
- stealth
- anti-detection
- CAPTCHA solving
- proxy rotation
- private login
- challenge bypass
- TLS disable
- verify=False
- ssl=False

若 STANDARD_HTTP 能取得完整必要資料：

Transport = STANDARD_HTTP

若 HTTP 可取得頁面但缺 JS render 後才出現的重要資料：

先提供證據，再考慮 Browser。

若來源回：

ACCESS_DENIED
LOGIN_REQUIRED
CAPTCHA_OR_CHALLENGE

依 typed error 停止該路徑。

不要換 transport 繞過。

---

# PHASE 4 — DEVELOPMENT SAMPLE DISCOVERY

在合法公開可存取的情況下，建立 Development Set。

目標約 10 筆，不是硬性數字。

樣本應盡量涵蓋：

- 不同區域
- 不同價格
- 不同格局
- 不同樓層
- 不同屋齡
- 有 / 無社區
- 有 / 無車位
- 不同圖片數量
- 不同房屋類型
- 資料完整案例
- 資料缺漏案例

不要全部選相同類型。

每筆記錄：

Sample ID:
Source URL:
Final URL:
Transport:
Adapter:
Parse result:
Canonical coverage:
Diagnostics:
Image result:

---

# PHASE 5 — FIELD DISCOVERY

以正式 CanonicalListing v2 為目標。

不要憑提示詞新增 schema。

至少確認來源能否提供：

listing_no
title
price
price_raw
address
city
district
road
layout
rooms
living
bath
area_total
floor
floor_total
age
building_type
features
community
parking
salesperson
phone
store_name
images
source
source_url

實際欄位依 repo 現有 CanonicalListing 為準。

---

# PHASE 6 — DATA SOURCE PRIORITY

AI 每個欄位依以下順序找來源：

1. stable structured JSON / page state
2. JSON-LD
3. stable public API payload
4. SSR HTML
5. DOM
6. fragile selector

不要第一個就用 CSS selector。

如果同一欄位有多個來源：

選最穩定且可測試者。

保留 provenance。

---

# PHASE 7 — FIELD STATE SEMANTICS

每欄必須正確使用：

PRESENT
NOT_PRESENT
EXTRACTION_FAILED
UNKNOWN

規則：

頁面明確沒有 / 不適用
→ NOT_PRESENT

頁面明明有但 parser 沒抓到
→ EXTRACTION_FAILED

證據不足
→ UNKNOWN

有值且來源可證明
→ PRESENT

禁止：

null = NOT_PRESENT

禁止：

0 = missing

---

# PHASE 8 — 建立 / 補強住商 Adapter

沿用現有架構。

如果目前只有 Draft / YAML mapping：

將成熟、可證明的 mapping 接入正式 Python Adapter / dispatch。

但不要建立第二套 Source Adapter framework。

Adapter 必須：

- 使用既有 BaseAdapter / ExtractionResult
- 不依賴 FastAPI
- 不依賴 DM / Post / 591
- 不認識 auth / entitlement / platform_admin
- 使用 injectable Transport
- 回 CanonicalListing
- 回 Diagnostics
- 回 typed errors
- 有 adapter version
- optional field 失敗可 partial，不可整包 generic error

---

# PHASE 9 — NORMALIZATION

不得把來源字串直接硬塞 canonical。

依現有 v2 rules 做：

Raw + Normalized

例如：

price_raw
price normalized

address_raw
address normalized

layout_raw
rooms / living / bath

floor_raw
floor / floor_total

area raw / normalized

features raw / normalized

parking raw / normalized

如果現有 schema 已有相同欄：

沿用。

不要重造欄位。

---

# PHASE 10 — IMAGES

比照 YUCT Reference 標準。

至少區分：

discovered
validated
downloaded
skipped
failed

不要：

只找到 URL
→ 標 downloaded

圖片至少遵守：

- HTTPS / safe scheme
- allowed host
- redirect safety
- MIME / signature（若執行 payload validation）
- payload size limit
- timeout

禁止：

verify=False
ssl=False

如果本輪只做到 URL / metadata 層：

如實回報。

不要假裝已下載。

---

# PHASE 11 — DIAGNOSTICS

每次住商 extraction 至少能回答：

request ID
source
adapter
adapter version
schema version
input URL
final URL
redirect history
transport
fallback reason
timing
field state
field provenance
image counters
typed errors
overall success / partial / failure

禁止保存：

password
Bearer token
完整 Cookie
session secret
API key
敏感 headers

---

# PHASE 12 — DEVELOPMENT AUTO-REPAIR

Development samples 若有：

欄位漏抓
解析失敗
格式差異
圖片問題

AI 先自己調查：

- structured data
- SSR
- DOM
- existing draft mapping
- historical evidence
- current adapter

流程：

Root cause
↓
Minimal generic fix
↓
Focused test
↓
Rerun all Development samples

禁止：

- listing ID hardcode
- URL token hardcode
- sample-specific selector
- 降低 required profile 只為了過測試

---

# 只有這些情況才問使用者

1. 欄位語意真的無法判斷
2. 網站需要合法登入
3. CAPTCHA / challenge
4. Canonical schema 有真實語意衝突
5. 某欄是否值得 canonical 化需要產品決策

需要使用者時固定格式：

SOURCE:
住商

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

不要丟一大堆 code 給使用者。

---

# PHASE 13 — DEVELOPMENT GATE

Development Set 必須：

- source detection PASS
- transport PASS
- parse PASS
- required profile PASS
- field-state semantics PASS
- provenance PASS
- images semantics PASS
- diagnostics PASS
- typed errors PASS

才能輸出：

HBHOUSING DEVELOPMENT PASS

---

# PHASE 14 — FRESH UNSEEN VALIDATION

Development 完成後，另外找一批：

沒有用來開發
沒有用來修 parser
沒有被提前看過來調整 mapping

的全新案件。

目標約 10 筆。

標記：

FRESH_UNSEEN

第一次執行結果必須保存。

如果全部 PASS：

HBHOUSING FRESH UNSEEN PASS

若有 FAIL：

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

如果拿 unseen sample 修程式：

該 sample 立即標：

CONSUMED_DURING_REPAIR

不得再算 fresh unseen。

修完後：

1. 重跑 Development
2. 重跑 consumed samples 作 regression
3. 再找新的 fresh unseen samples

禁止：

修完同一批
→ 宣稱 fresh unseen PASS

---

# PHASE 15 — GOLDEN

沿用 repo 現有 Golden / publish policy。

不要建立第二套。

代表性案例至少考慮：

- structured case
- SSR / fallback case（若存在）
- optional fields case
- image case
- features / parking / community case
- missing field case
- removed / unavailable case（只有真實存在且可安全保存時）

Golden 要保護：

required fields
important optional fields
field states
provenance
image semantics
diagnostics
typed errors
schema compatibility

不要機械式把所有 Development URL 都做成 Golden。

---

# PHASE 16 — FULL REGRESSION

至少執行：

1. 住商 focused tests
2. Canonical tests
3. Transport tests
4. Diagnostics tests
5. Image validation tests
6. Golden tests
7. YUCT focused tests
8. CTHouse focused tests
9. ExtractionHub full pytest
10. git diff --check

要求：

- YUCT 不可 regression
- CTHouse 既有已修 features 不可 regression
- 不得因住商而破壞 Core Contract

---

# PHASE 17 — READY GATE

只有以下全部成立：

Source detection PASS
Transport PASS
Development PASS
Fresh unseen PASS
Canonical mapping PASS
Required fields PASS
Images PASS
Diagnostics PASS
Typed errors PASS
Golden PASS
Schema/version PASS
Full regression PASS

才能：

HBHOUSING REFERENCE READY

如果有 blocker：

HBHOUSING REFERENCE BLOCKED

不要因：

adapter exists
draft exists
development pass

就標 Ready。

---

# 文件

沿用現有：

_charter/charter.md
_charter/current-state.md
_charter/history.md

以及 repo 既有 docs。

不要建立第二套 SSOT。

更新：

current-state
→ 住商真實狀態

history
→ 本輪實作 / 驗證紀錄

charter
→ 只有穩定架構原則真的改變才更新

---

# FINAL REPORT FORMAT

# HBHOUSING / 住商 REFERENCE ADAPTER REPORT

## 1. Baseline

Branch:
HEAD:
Dirty state:
Collision:
PASS / BLOCKED

## 2. Source Identity

Source ID:
Brand:
Domains:
Aliases:
Registry status:

## 3. Existing Work

REUSE:
PATCH:
REPLACE:
DEFER:

## 4. Transport

Selected:
Reason:
Browser required:
YES / NO / NOT_PROVEN

Access policy:
PASS / BLOCKED

## 5. Development Samples

Total:
PASS:
FAIL:

表格：

| ID | URL | Parse | Required | Images | Result |

## 6. Canonical Coverage

Required:
Optional:
UNKNOWN:
EXTRACTION_FAILED:

列出重要缺口。

## 7. Provenance

Structured:
SSR:
DOM:
Other:

Result:
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

PASS / FAIL

## 11. Regression

HBHousing focused:
Canonical:
Transport:
Diagnostics:
Images:
YUCT focused:
CTHouse focused:
Full pytest:
git diff --check:

## 12. Remaining Gaps

如果沒有：

NONE

## 13. Final Result

只能：

HBHOUSING REFERENCE READY

或

HBHOUSING REFERENCE BLOCKED

---

# HARD STOP

完成住商後停止。

不要開始：

- 永慶
- 台灣房屋
- 591
- 樂屋
- 信義
- 其他來源

不要修改：

- DM
- Post
- 591

不要部署：

- Staging
- Production

等待使用者下一個指令。
