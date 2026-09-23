# DM — EXTRACTION CORE YUCT SHADOW INTEGRATION OWNER

## 工作目錄

DM：
F:\00-Ticenpi-SaaS\TicenpiDM

Extraction Core：
F:\00-Ticenpi-SaaS\ExtractionHub

ExtractionHub 僅作 Integration dependency / reference。

不要修改：
F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591

## 任務背景

ExtractionHub 已完成：
- CanonicalListing v2
- Adapter contract
- HTTP Transport contract
- Diagnostics contract
- YUCT Reference Adapter
- YUCT controlled validation

目前 YUCT Reference 狀態：
YUCT REFERENCE READY

本任務不是再開發 YUCT Adapter。
本任務是：
將 DM 現有 YUCT scraper 與新的 Extraction Core 接成安全的 Shadow Mode。

## 最終目標

目前正式行為維持：

DM 使用者
↓
DM Legacy YUCT Scraper
↓
結果回給使用者

新增：

同一個 URL
↓
背景執行 Extraction Core
↓
CanonicalListing
↓
比較 Legacy Result
↓
記錄 Shadow Result
↓
platform_admin Dashboard

非常重要：

Extraction Core 在 Shadow Mode：
- 沒有決定權
- 不得修改使用者看到的結果
- 不得阻塞 Legacy Scraper
- 不得讓 Legacy Request 因 Shadow failure 失敗
- 不得自動 fallback / replace Legacy
- 不得刪除 Legacy YUCT scraper

## 核心行為

最終流程：

User submits YUCT URL
↓
DM existing scrape API
↓
Legacy YUCT Scraper
↓
legacy_result
├────────────────────────→ return to user
│
└→ background Shadow Run
    ↓
    Extraction Core
    ↓
    CanonicalListing
    ↓
    Legacy Mapper
    ↓
    Comparator
    ↓
    Shadow Recorder
    ↓
    /admin/extraction-shadow

最重要原則：

使用者的正式 response 必須由 Legacy scraper 決定。
Shadow 不可以增加使用者等待時間。

## PHASE 0 — PREFLIGHT

開始前先確認：

DM：
git branch
git HEAD
git status
git diff
git diff --cached

ExtractionHub：
git branch
git HEAD
git status

並確認目前是否有：
- 其他 active writer
- DM Production deployment 正在進行
- 相同檔案正在被其他 Session 修改

如果發現真正 collision：
HARD STOP

回報：
SHADOW INTEGRATION COLLISION

不要 reset / clean / stash / rebase。

如果沒有 collision：
SHADOW INTEGRATION PREFLIGHT PASS

然後繼續。

## PHASE 1 — 找到 DM 真正的 YUCT FLOW

不要依提示詞猜檔名。

從現有程式實際 trace：

Frontend
↓
scrape API
↓
auth / entitlement
↓
ScrapeService
↓
YUCT extractor
↓
result
↓
frontend apply

上一輪已知可能存在：
routers/scrape.py
services/scrape.py
extractors/yuct/

但以目前 repo 實際內容為準。

找出：
1. 哪裡收到 URL
2. 哪裡完成 auth / entitlement
3. 哪裡真正呼叫 YUCT scraper
4. 哪裡取得 legacy result
5. 哪裡 return 給 frontend

Shadow 必須放在：
auth / entitlement 已經通過
+
legacy extraction 已經開始或完成
之後。

不能繞過 DM 原有商用權限。

## PHASE 2 — 建立 Shadow Integration Layer

不要讓 DM business code 到處直接 import ExtractionHub。

建立一層很薄的 Shadow integration。

概念：

DM
↓
Shadow Bridge
↓
Extraction Core

建議結構依 repo convention 決定。

概念上至少包含：
- shadow_bridge
- legacy_mapper
- comparator
- shadow_recorder

名稱可依現有程式風格調整。
不要為符合提示詞硬建特定資料夾。

### 2A — Shadow Bridge

提供概念：

run_shadow(url, context)

作用：
同一個 YUCT URL 交給 Extraction Core。

Extraction Core 回：
ExtractionResult
CanonicalListing
Diagnostics

Shadow Bridge 必須：
- 有獨立 timeout
- catch 所有 Shadow exception
- Shadow failure 不向上拋到正式 request
- 不包含 DM-specific business mapping
- 不改 legacy result

如果 Shadow Core：
timeout
exception
parse fail

只記錄 Shadow Result。
正式 DM Request 繼續正常。

## IMPORTANT — Plugin / Central 不綁死

目前仍未決定：
PLUGIN
CENTRAL SERVICE
HYBRID

因此 DM 不可以把大量程式綁死 ExtractionHub implementation。

Shadow Bridge 要形成邊界：

DM
↓
ShadowProvider
↓
Extraction Core

本輪使用目前最小、最可靠的方式執行 Core。

但 Product code 不應依賴：
- FastAPI-specific response
- ExtractionHub internal files
- SQLite run record
- 某個 HTTP endpoint layout

如果目前最合理是直接 Python Core：
可以。

如果現有環境已有可靠 wrapper：
可以使用。

但 ShadowProvider 必須保持可替換。

最終報告說明本輪採用哪種方式。

## PHASE 3 — Legacy Mapper

Legacy Result 和 CanonicalListing 欄位名稱不同。

不能直接：
legacy_result == canonical

建立 DM Legacy Comparison Mapper。

只處理：
「房源本身的事實」。

例如：
listing_no
title
price
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
images

實際欄位依 DM 與 Canonical schema 為準。

禁止比較：
- DM canvas
- design state
- draft state
- UI state
- subscription
- auth
- entitlement
- AI state
- 圖片選取 UI

## PHASE 4 — Comparator

建立共同 Comparator。

Comparator 不修改任何結果。
只回答 Legacy vs Core 是否一致。

每個欄位至少有：
MATCH
MISMATCH
UNCOMPARABLE
LEGACY_MISSING
CORE_MISSING
BOTH_MISSING

但是要利用 Canonical FieldState：
PRESENT
NOT_PRESENT
EXTRACTION_FAILED
UNKNOWN

不要把：
UNKNOWN == UNKNOWN
自動算成功。

### 數值比較

價格：
先 normalization 到相同單位 TWD。

例如：
Legacy：1588 萬
Core：15880000 TWD

應判斷：
MATCH

### 坪數

使用 normalized numeric。

避免：
32.5
vs
32.50
被判 MISMATCH。

使用現有 canonical normalization。

不要自行發明過大的容錯。
如需要 tolerance：
明確定義且測試。

### 地址

比較 normalized address。
但同時保留 raw result 供診斷。

不要為了提高 match rate
把完全不同地址過度 normalize 成一樣。

### 格局

優先比較：
rooms
living
bath

不要只比較 raw string。

### 圖片

圖片比較不要只比較：
圖片數量。

至少盡可能比較：
normalized / canonical source URL set

並另外記：
legacy count
core count
count match
set match

如果兩邊圖片語意完全不同：
UNCOMPARABLE

不要假裝 MISMATCH。

## PHASE 5 — Shadow Recorder

每次 Shadow Run 必須保存一筆 sanitized comparison result。

優先檢查 DM 目前是否已有：
- database
- telemetry
- audit table
- metrics storage

可安全沿用。

如果已有適合 persistence：
沿用。

如果沒有：
建立最小專用 persistence。

不要為 Shadow 建大型新平台。

### Shadow Run 至少記錄

run_id
created_at
product = dm
source = yuct
safe source URL / listing identity
DM release / commit（如果現有 runtime identity 可取得）
legacy_success
core_success
legacy_duration_ms
core_duration_ms
core_adapter_version
core_schema_version
comparison summary
matched_count
mismatch_count
uncomparable_count
per-field result
core typed error（如果存在）

注意：
不得記：
- JWT
- Bearer token
- Cookie
- session
- API key
- 完整敏感 headers

## 成功定義

LEGACY SUCCESS：
DM 原本 Legacy extraction 成功產生可用正式結果。

CORE SUCCESS：
Extraction Core 成功完成 extraction，且符合 YUCT required profile。

Partial result 不要一律當 full success。
要依目前 ExtractionResult contract 判斷。

## 四種整體結果

每一次 dual run 分成：

BOTH_SUCCESS
CORE_ONLY_SUCCESS
LEGACY_ONLY_SUCCESS
BOTH_FAILED

這四個數字就是 Dashboard 最上層的重要指標。

## PHASE 6 — Background Execution

Shadow 不得增加正式 request latency。

優先使用 DM 已存在的：
- background worker
- queue
- task mechanism

如果沒有：
可使用現有 backend framework 的 best-effort background task。

因為 Shadow 是非關鍵觀察工作，
允許偶爾因 process restart 遺失。

不要為 Shadow 額外建立：
- Redis cluster
- 大型 queue platform
- 新 microservice

除非 repo 本來已經有並且可直接沿用。

### Legacy Response Rule

概念必須等同：

legacy_result = await legacy_extract(url)

schedule_shadow_in_background(
    url,
    legacy_result
)

return legacy_result

而不是：

legacy_result
↓
等待 Core
↓
等待 compare
↓
return

## PHASE 7 — Feature Flags

加入明確 Shadow controls。

依現有 config conventions。

至少需要概念：
EXTRACTION_SHADOW_ENABLED
EXTRACTION_SHADOW_SAMPLE_RATE
EXTRACTION_SHADOW_SOURCES

預設：
EXTRACTION_SHADOW_ENABLED=false

不能因部署新 code
自動開啟 Shadow。

### Sample Rate

接受：
0.0 ～ 1.0

例如：
0.10 = 10%
0.50 = 50%
1.00 = 100%

抽樣應盡量 deterministic / testable。
不要散落 random() 在 business logic。

### Source Gate

目前只允許：
YUCT

不要讓未知來源或其他網站誤進 Shadow comparison。

## PHASE 8 — PLATFORM_ADMIN REPORT API

建立：
platform_admin only
的報告 API。

建議：
GET /api/admin/extraction-shadow/report

但如果 DM 已有 admin route convention：
沿用既有 convention。

不要重造 admin auth。

### Report 支援

至少：
24 hours
7 days
30 days
all

以及：
source filter

目前：
YUCT

### 最上層 Summary

回：
total_shadow_runs
legacy_success
core_success
both_success
core_only_success
legacy_only_success
both_failed
shadow_error_count

### 每欄 Match Rate

至少包括：
price
address
area_total
layout
floor
age
community
parking
images

有資料再包含：
salesperson
phone
store_name

### Match Rate 不能造假

對每個 field 回：
comparable_count
match_count
mismatch_count
uncomparable_count
legacy_missing_count
core_missing_count

match_rate 定義：
match_count / comparable_count

沒有 comparable case 時：
match_rate = null

不要：
0 / 0 = 100%

## PHASE 9 — PLATFORM_ADMIN DASHBOARD

我要直接在瀏覽器看到結果。

建立 platform_admin-only：

/admin/extraction-shadow

如果 DM 現有 admin routing 有別的 convention：
沿用。

普通客戶不可進入。

### Dashboard 最上方

顯示：

YUCT SHADOW REPORT

Time Range：
24h / 7d / 30d / All

Source：
YUCT

並顯示：
Dual Runs
Legacy Success
Core Success
Both Success
Core Only
Legacy Only
Both Failed

### 欄位一致率

例如：

價格       99.7%
地址      100.0%
坪數       99.3%
格局      100.0%
樓層       98.7%
屋齡       99.1%
社區       97.3%
車位       98.5%
圖片       96.8%

每個欄位旁邊也顯示：
matched / comparable

例如：
299 / 300
99.7%

不要只顯示百分比。

### Dashboard 第二區

顯示：
Recent Mismatches

列表例如：
Run
Listing
Field
Legacy
Core
Core State
Adapter Version
Time

例如：

7W6DJv
community
Legacy: XX花園
Core: UNKNOWN
State: UNKNOWN

### 敏感資料

Dashboard 不得顯示：
- token
- cookie
- secret
- auth headers

公開房源 URL 可以依現有安全規則提供 sanitized / clickable reference。

### Dashboard 第三區

顯示：
Failure Breakdown

例如：
PARSE_FAILED 3
TIMEOUT 1
FETCH_FAILED 2
UNKNOWN 0

以及：
Core Only Success
Legacy Only Success

可以點進去看是哪幾筆案件。

## PHASE 10 — API / DASHBOARD AUTH

必須使用 DM 現有正式：
platform_admin
授權。

不得另外建立：
- SHADOW_ADMIN_KEY
- 秘密 URL
- header bypass

測試：
platform_admin → 200
一般 DM entitlement user → DENY
未登入 → DENY

## PHASE 11 — TESTS

至少建立：

1. Shadow disabled → 完全不呼叫 Core
2. Sample rate 0 → 不執行
3. Sample rate 1 → 執行
4. Shadow Core success → Legacy response 不變
5. Shadow Core fail → Legacy response 不變
6. Shadow timeout → Legacy response 不變
7. Comparator price normalization
8. Comparator area normalization
9. Comparator address
10. FieldState comparison
11. Images comparison
12. Sensitive diagnostics sanitization
13. Report aggregation
14. Match-rate denominator
15. platform_admin report access
16. normal user denied
17. unauthenticated denied

## PHASE 12 — LOCAL / STAGING VALIDATION

先不要直接開 Production Shadow。

使用目前已確認正常的 YUCT controlled URLs 做 integration test。

至少驗：
Legacy
+
Core
+
Compare
+
Persist
+
Dashboard

完整走一次。

重要：
不能只 mock Core。

可以有 unit test mock。

但 Integration Gate 至少一次：

真實 DM Legacy YUCT
+
真實 Extraction Core YUCT
+
真實 Comparator

一起跑。

## PHASE 13 — Shadow Rollout

程式完成後：

Shadow feature 預設 OFF。

先回報。

不要自行 Production enable。

未來核准後 rollout：

Stage 1
10%

Stage 2
50%

Stage 3
100%

每階段觀察：
- legacy/core success
- field match rate
- failure types
- latency
- resource consumption

重要：
Shadow 不是替換。

即使 100% shadow，
仍然是：

Legacy Primary
Core Shadow

不要自動變：
Core Primary

## PHASE 14 — Performance

Shadow 會多跑一次擷取。

因此至少量：
- Legacy extraction duration
- Core extraction duration
- DM request latency before / after
- backend CPU / memory（能取得則記錄）
- Shadow task concurrency

避免 Shadow 造成：
- 大量同時 request
- memory spike
- 外部網站過量 request

必要時加入：
bounded concurrency

但不要建立複雜平台。

## FINAL REPORT

固定格式：

# DM YUCT SHADOW INTEGRATION REPORT

## 1. Baseline

DM branch:
DM commit:

ExtractionHub branch:
ExtractionHub commit:

Collision:
PASS / BLOCKED

## 2. Legacy Flow

實際 DM YUCT flow。

## 3. Shadow Architecture

ASCII 圖。

## 4. Shadow Provider

目前如何呼叫 Extraction Core：

LOCAL CORE
HTTP
OTHER

以及為什麼。

確認：
可替換 = YES / NO

## 5. Comparator

列：
fields
normalization
image comparison
FieldState handling

## 6. Persistence

Shadow Result 存在哪。

確認：
No secrets stored = PASS / FAIL

## 7. Dashboard

URL:
/admin/extraction-shadow
或實際 route

API:
實際 route

Auth:
platform_admin only

## 8. Tests

列：
focused tests
full tests
integration tests

## 9. Controlled Integration

Controlled YUCT URLs：

Legacy:
Core:
Compare:
Persist:
Dashboard:

結果。

## 10. Performance

Legacy latency:
Shadow Core latency:
User request added latency:
Memory / CPU evidence:

如果沒有 runtime measurement：
NOT MEASURED

不要猜。

## 11. Production Defaults

Shadow enabled:
FALSE

Sample rate:
實際預設

Source:
YUCT

## 12. Remaining Blockers

沒有則：
NONE

## 13. Final Result

只能：

DM YUCT SHADOW READY

或

DM YUCT SHADOW BLOCKED

## HARD STOP

完成：
Shadow Bridge
Legacy Mapper
Comparator
Recorder
Admin API
Admin Dashboard
Tests
Controlled integration

後停止。

不要：
- enable Production Shadow
- 切 Core Primary
- 刪 Legacy scraper
- 修改 Post
- 修改 591
- 開始其他 Adapter integration

等待使用者核准。
