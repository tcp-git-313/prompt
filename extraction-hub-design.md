# ExtractionHub 設計規格

## 1. 定位

ExtractionHub 是獨立於 TicenpiPost 的通用網站結構化資料擷取平台。

固定專案位置：

E:\workflow-hub-data\workspace\ExtractionHub

ExtractionHub 不屬於 TicenpiPost，也不把 Ticenpi 的 Listing 當平台核心。

目標：

~~~text
URL
↓
辨識網站 / 品牌 / 技術平台
↓
比較不同擷取策略
↓
取得 Raw Data
↓
Mapping
↓
Normalization
↓
Canonical Data
↓
依 Project Schema 輸出
~~~

同一個 Site Adapter 可以支援：

- TicenpiPost：房仲 14 欄 + images
- Project B：6 欄
- 未來其他不同 Schema

---

## 2. 專案邊界

唯一寫入路徑：

E:\workflow-hub-data\workspace\ExtractionHub

TicenpiPost 若存在：

只允許唯讀參考：

- Listing contract
- normalize
- validate
- image rules
- Golden Samples
- architecture

兩個專案未來以 Versioned HTTP API 整合。

禁止：

- symlink
- submodule runtime 共用
- 互相直接 import
- 把 ExtractionHub 寫進 TicenpiPost

---

## 3. 核心分層

### 3.1 Site / Platform Adapter

負責：

「這個網站實際提供什麼資料、從哪裡取得」。

### 3.2 Canonical Data

負責把不同網站的表示方式統一。

例如：

~~~text
2000
2000萬
NT$20,000,000
~~~

如果 Evidence 證明都是同一價格，Canonical 都轉成：

~~~text
amount = 20000000
currency = TWD
~~~

### 3.3 Schema

負責：

「某個專案需要哪些欄位」。

### 3.4 Project Mapper

把 Canonical Data 轉成專案模型。

例如：

Ticenpi：

~~~text
price_raw = 2000萬
price_wan = 2000
~~~

其他專案：

~~~text
price = 20000000
~~~

---

## 4. Schema 與 Adapter 必須分離

Site Adapter 不應專門輸出 Ticenpi Listing。

應：

~~~text
Website
↓
RawSiteData
↓
CanonicalData
↓
Schema A / Schema B / Schema C
~~~

Schema 可自訂：

- 欄位名稱
- 型別
- 必填
- alias
- canonical unit
- validator
- description

支援：

string / integer / decimal / boolean / enum / date / object / array / images[]。

---

## 5. 第一批房仲網站

### P0

1. 591房屋交易 — 591.com.tw
2. 永慶房屋 — yungching.com.tw
3. 樂屋網 — rakuya.com.tw
4. 台灣房屋 — twhg.com.tw
5. 住商不動產 — hbhousing.com.tw
6. 中信房屋 — cthouse.com.tw
7. 太平洋房屋 — pacific.com.tw
8. 東森房屋 — etwarm.com.tw
9. 有巢氏 — u-trust.com.tw / buy.u-trust.com.tw
10. 信義房屋 — sinyi.com.tw

### P1

- 永義房屋
- 永慶不動產
- 台慶不動產
- 21世紀不動產
- 大家房屋
- 群義房屋
- 全國不動產
- 南北房屋
- 好房網

Bootstrap 前都要再次驗證：

- 官方 Domain
- 買屋入口
- Redirect
- Detail URL
- 是否共用集團平台

---

## 6. 品牌與技術平台不能一對一綁死

建立：

- BrandProfile
- PlatformFingerprint

因為不同品牌可能共用：

- API
- HTML
- React / Angular
- Gallery
- 物件 Backend

因此：

~~~text
品牌 A ─┐
品牌 B ─┼→ 共用 Platform Adapter
品牌 C ─┘
       + Brand Override
~~~

避免 copy/paste 多份 parser。

---

## 7. AUTO / MANUAL 雙模式

### AUTO

輸入 URL 後判斷：

- redirect
- hostname
- URL pattern
- framework fingerprint
- PlatformFingerprint
- brand markers

輸出：

- site_id
- platform_family
- confidence
- reasons

建議：

- >= 0.95：AUTO_CONFIRMED
- 0.70–0.949：SUGGESTED
- < 0.70：NEEDS_SITE_SELECTION

### MANUAL

使用者可手動指定房仲品牌。

Manual 優先，但必須檢查 URL 是否相容。

---

## 8. Site Discovery

不要「隨便點一顆 button」。

Discovery 應找：

~~~text
官方首頁
↓
買屋列表
↓
物件卡
↓
Detail URL pattern
↓
代表性樣本
~~~

每站至少 5 個 Sample，最好 10 個。

樣本要包含不同：

- 縣市
- 價格
- 房型
- 樓層
- 大樓 / 公寓 / 透天 / 土地

---

## 9. Adaptive Extraction Strategy

不能先假設某站一定 HTTPX 或一定 Playwright。

至少比較：

1. HTTP_STANDARD_STATIC
2. HTTP_BROWSER_COMPAT_STATIC
3. HTTP_EMBEDDED
4. HTTP_API
5. BROWSER_NETWORK
6. BROWSER_DOM

核心原則：

「能完整且正確取得資料的最低成本方法」。

---

## 10. HTTP_STANDARD_STATIC

使用 httpx。

分析：

- SSR HTML
- JSON-LD
- Embedded JSON
- script JSON

優點：

最快、資源最低。

如果完整正確，Production 優先使用。

---

## 11. Browser-compatible HTTP

建立 HttpTransport interface。

至少：

- STANDARD_HTTP：httpx
- BROWSER_COMPAT_HTTP：評估 curl_cffi

用途：

當普通 HTTP Client 與主流瀏覽器在公開內容上得到不同結果時，提供另一種 Transport。

不得設計為：

- CAPTCHA bypass
- 登入破解
- Access-Control bypass
- challenge token replay

遇到明確限制必須 STOP / REVIEW。

---

## 12. HTTP_EMBEDDED

自動找：

- __NEXT_DATA__
- __NUXT__
- ng-state
- Angular hydration
- Redux
- Apollo cache
- INITIAL_STATE
- application/json script

結構化資料優先於 DOM。

---

## 13. HTTP_API

如果網站頁面實際使用公開 API：

- REST
- XHR JSON
- GraphQL

可以 Benchmark 直接 HTTP 呼叫。

但要保存：

- endpoint
- method
- functional params
- response schema
- evidence

不能保存 CAPTCHA / challenge / 登入用短期 token 當 bypass rule。

---

## 14. Browser as Teacher

Playwright 不只是 Production Crawler。

更重要用途：

~~~text
Browser
↓
Network Inspector
↓
找到真正 API / JSON
↓
Cross Sample 驗證
↓
建立 HTTP Strategy
~~~

例如 Browser 發現：

~~~text
API price = 2000
頁面 = 2000萬
~~~

跨多 Sample 確認單位後：

~~~text
Source = API.price
Site Override = ×10000
~~~

之後正式 Production 可直接 HTTP。

---

## 15. Browser DOM

最後 fallback。

優先：

- label/value
- table
- heading relation
- definition list
- accessible semantics

避免：

- nth-child
- hashed CSS
- build-generated class

---

## 16. Strategy Benchmark

Bootstrap 階段各方法都測。

每個 Strategy 保存：

- sample_url
- transport
- request_ms
- browser_startup_ms
- navigation_ms
- parse_ms
- normalize_ms
- total_ms
- fields_found
- fields_normalized
- fields_validated
- fields_matched
- required_coverage
- image coverage
- errors
- warnings

UI 要能顯示：

~~~text
Method                 Verified    Images   Avg
HTTP API               14/14       21/21    0.24s
HTTP Embedded          14/14       21/21    0.41s
HTTP HTML               9/14       4/21     0.31s
Browser Network        14/14       21/21    3.8s
Browser DOM            14/14       21/21    5.2s
~~~

---

## 17. Strategy Selection

不能只用一個加權分數決定。

先 Gate：

1. Required fields 達標。
2. Validation 達標。
3. Images 達標。
4. Cross Sample 穩定。

通過後才比較：

- P50 / P95 latency
- failure rate
- resource cost

例如：

HTTP 0.2s 但 13/14 → 淘汰。

HTTP Embedded 0.4s 14/14 → 候選。

Browser 5s 14/14 → fallback。

---

## 18. Preferred + Fallback Chain

每站保存：

~~~text
preferred_strategy
fallback_chain
~~~

Production：

- 先跑 Preferred
- Required fields 全 VALIDATED 即完成
- 不要每次 Production 都跑所有 Strategy

全 Benchmark 只在：

- Bootstrap
- Manual Rebenchmark
- Drift Investigation

---

## 19. High Difficulty Site

例如 591：

~~~text
difficulty = HIGH
~~~

但 HIGH 不代表直接跳過 HTTP。

代表：

- HTTP 快速試一次
- 無效不要大量 retry
- 縮短浪費時間

Benchmark Budget 可配置：

- max attempts
- timeout
- max total seconds

遇到 challenge / Access Denied：

記錄後依政策 fallback 或停止。

---

## 20. Access Policy

至少：

- OK
- RATE_LIMITED
- LOGIN_REQUIRED
- CAPTCHA
- ACCESS_DENIED
- BLOCKED
- ROBOTS_RESTRICTED
- UNKNOWN_CHALLENGE

不得把破解 CAPTCHA、繞 Access Control、無限 Proxy Rotation 設成核心能力。

Proxy 只作為可配置正常出口。

---

## 21. Rate Limit 與 Cache

每 Site 設定：

- max_concurrency
- requests_per_minute
- backoff
- min_delay

建立短期 Cache：

- HTML
- JSON
- API response
- Probe result

同一 URL Benchmark 不要反覆下載。

---

## 22. Response Guard

HTTP 200 不一定代表正常頁。

Guard 要辨識：

- 403
- 429
- 503
- challenge page
- login page
- error page
- empty SPA shell
- 異常 redirect
- 預期欄位全部消失

---

## 23. Raw → Canonical Pipeline

固定順序：

~~~text
Raw Value
↓
Parser
↓
Universal Normalizer
↓
Site Override
↓
Canonical Value
↓
Schema Mapping
↓
Project Mapper
~~~

不可隨意交換。

---

## 24. Normalization

必須處理：

- price
- area
- age
- floor
- layout
- address
- number
- date
- boolean
- enum
- text
- images

支援：

- trim
- regex_extract
- replace
- number_parse
- multiply
- divide
- unit_convert
- enum_map
- split
- join
- floor_parse
- layout_parse
- address_parse

---

## 25. 價格範例

網站 A：

~~~text
raw = 2000
context = 總價（萬）
~~~

網站 B：

~~~text
raw = 2000萬
~~~

網站 C：

~~~text
raw = NT$20,000,000
~~~

Canonical 全部：

~~~text
amount = 20000000
currency = TWD
~~~

Ticenpi Mapper：

~~~text
price_wan = 2000
~~~

Project B：

~~~text
price = 20000000
~~~

---

## 26. Normalization Studio

UI 必須讓非工程師調整。

例如：

~~~text
Field: 委託價
Source: data.house.price
Raw: 2000
Detected Unit: 萬
Canonical: 20,000,000 TWD

Transform:
value × 10000
~~~

提供：

- 測試
- 套用目前 Sample
- 套用全部 Samples
- 儲存 Site Rule

Mapping 錯與 Normalization 錯必須分開。

---

## 27. Universal Probe / Candidate Pool

Probe 不直接決定 Mapping。

每個 Candidate 保存：

- value
- source_type
- source_url
- json_path
- selector（必要時）
- label
- nearby_text
- unit
- evidence
- confidence

例如：

~~~text
area_main

Candidate A
18.23
API.data.mainArea

Candidate B
主建物18.23坪
DOM

Candidate C
35.52
API.totalArea
~~~

---

## 28. Auto Mapping

優先：

- deterministic rules
- aliases
- label
- unit
- value type
- range
- JSON path
- cross sample
- cross strategy

最後才用 LLM Ranking。

---

## 29. LLM 邊界

LLM 可以：

- Candidate semantic ranking
- 提議 mapping
- 解釋歧義

LLM 不得：

- 生成不存在的值
- 補價格
- 補坪數
- 虛構圖片
- 直接發布 Production Adapter

硬規則：

LLM 不可成為 AUTO_CONFIRM 的唯一證據。

---

## 30. Confidence

Field Confidence 與 Strategy Selection 分開。

Field Confidence 建議包含：

- Source Reliability
- Semantic Match
- Cross Sample Consistency
- Unit Evidence
- Cross Strategy Agreement
- Validation

必須顯示 Breakdown。

Hard Cap：

- Unit-sensitive numeric field 無單位 Evidence → 最大 0.79
- Validation failed → 最大 0.49
- Cross Sample <80% → 不得 AUTO_CONFIRMED
- 只有 LLM inference → 不得 AUTO_CONFIRMED

Levels：

- >=0.95 AUTO_CONFIRMED
- 0.80–0.949 PROVISIONAL
- <0.80 NEEDS_REVIEW

---

## 31. Field Status

不能只看非空。

狀態：

- MISSING
- FOUND
- NORMALIZED
- VALIDATED
- MATCHED

只有 VALIDATED / MATCHED 才算 Verified Coverage。

---

## 32. Teach Mode

人工介入方式：

1. 從 Candidate Pool 選正確值。
2. Browser Inspector 點畫面上的值。

如果點到「主建物18.23坪」：

平台先反查：

- API
- Embedded JSON
- Hydration Data

若找到 data.mainArea=18.23：

保存 JSON Path。

上游真的找不到才保存 DOM selector。

---

## 33. Image Intelligence

優先：

- API gallery
- Embedded JSON gallery
- JSON-LD
- DOM gallery
- Browser gallery

保存：

- raw_url
- canonical_url
- displayed_dimensions
- best_source_dimensions
- order
- source
- confidence

排除：

- logo
- icon
- avatar
- banner
- 廣告
- 門市照
- 經紀人照片
- QR
- tracking pixel
- 全站共用素材

小尺寸只是 Negative Signal，不是一刀切。

80×60 thumbnail 若有 1600×1200 原圖仍應保留。

---

## 34. Adapter

優先 Declarative Adapter：

~~~text
adapters/sinyi.yaml
~~~

包含：

- site_id
- brand
- platform_family
- domains
- detail patterns
- preferred strategy
- fallback
- transport
- field mappings
- normalizers
- units
- image strategy
- validators
- confidence
- samples
- version
- last_verified_at

只有 YAML 無法表達才新增 custom code。

---

## 35. Adapter Lifecycle

- DRAFT
- VALIDATING
- PUBLISHED
- DEPRECATED

新 Learn 只能產生 DRAFT。

Regression 通過才能 Publish。

必須 version + rollback。

---

## 36. Queue / Scheduler

FastAPI 不同步跑重工作。

架構：

~~~text
Frontend
↓
FastAPI
↓
run_id
↓
Redis + arq
↓
Worker
↓
Extraction
↓
DB
↓
SSE / Polling
~~~

arq：

- Bootstrap
- Benchmark
- Regression
- Drift
- Extraction Run
- Scheduled Validation

Crawlee RequestQueue：

只管理 crawl URL lifecycle。

兩者不可混用。

---

## 37. Scheduler

至少：

- Periodic Drift Check
- Golden Sample Regression
- Adapter Health Check
- Relearn Candidate Job

ExtractionHub 不能只有 On-demand API。

---

## 38. Regression

每站保存 Golden Samples：

- URL
- raw evidence
- expected canonical values
- image expectations

修改：

- Adapter
- Mapping
- Normalizer
- Image Strategy

都必須重跑。

Required regression 失敗禁止 Publish。

---

## 39. Drift Detection

監測：

- field success
- API schema
- JSON path
- image coverage
- fallback usage
- latency
- failure rate
- challenge frequency

例如：

~~~text
昨天 HTTP_API 14/14
今天 HTTP_API 8/14
Browser Network 14/14
~~~

→ STRATEGY_DRIFT
→ 暫時 fallback
→ RELEARN_REQUIRED

不可默默抓錯。

---

## 40. Extractor Studio

至少包含：

### Extract

- URL
- AUTO / MANUAL
- Schema

### Sites

- Brand
- Platform
- Difficulty
- Preferred Strategy
- Coverage
- Images
- Latency
- Adapter Version
- Health

### Site Detail

- Samples
- Probe
- Candidates
- Mapping
- Normalization
- Images
- Benchmark
- Regression
- Drift

### Review Queue

只顯示：

- PROVISIONAL
- NEEDS_REVIEW
- DRIFT

### Schema Manager

可以建立 14 欄、6 欄或任意自訂 Schema。

---

## 41. API

POST /v1/extract

輸入：

~~~text
url
schema_id
schema_version
site_mode = AUTO | MANUAL
manual_site_id optional
~~~

輸出：

- run_id
- detected_site
- site_confidence
- platform_family
- adapter_version
- strategy_used
- transport_used
- raw_fields
- parsed_fields
- canonical_fields
- project_fields
- field_status
- images
- validation
- evidence
- timings
- response_guard

重工作 API：

- /v1/sites/bootstrap
- /v1/sites/{id}/learn
- /v1/sites/{id}/benchmark
- /v1/sites/{id}/validate
- /v1/sites/{id}/regression
- /v1/sites/{id}/publish

全部回 202 + run_id。

---

## 42. Bootstrap 流程

每站：

1. Verify Official Domain
2. Find Buy/List Page
3. Discover Detail Pattern
4. Collect Samples
5. STANDARD_HTTP
6. BROWSER_COMPAT_HTTP
7. Embedded JSON Probe
8. Browser Network Discovery
9. HTTP API Replay
10. Browser DOM
11. Candidate Pool
12. Mapping
13. Normalization
14. Images
15. Cross Sample
16. Strategy Selection
17. Draft Adapter
18. Regression
19. Report

單站失敗不可停止整批。

---

## 43. 驗收

必須實際證明：

1. AUTO 與 MANUAL Site 模式都能運作。
2. 同 Site Adapter 可供兩個不同 Schema 使用。
3. 2000 / 2000萬 / NT$20,000,000 可統一 Canonical。
4. HTTPX / Browser-compatible HTTP / API / Browser 可 Benchmark。
5. Mapping 錯與 Normalizer 錯可分開修。
6. Low Confidence 自動進 Review。
7. LLM 不能自行發布 Adapter。
8. Golden Samples 可 Regression。
9. Drift 可自動偵測。
10. Production 不會每次全 Strategy 重跑。
11. FastAPI 重工作全部進 Queue。
12. 圖片可排除共用與非物件素材。
13. 網站改版時寧可 NEEDS_REVIEW，也不能默默輸出錯誤資料。

---

## 44. 最終核心

ExtractionHub 的目標不是「寫很多爬蟲」。

而是：

~~~text
第一次：
多 Strategy 學習網站
↓
找出資料真正來源
↓
建立 Mapping / Normalizer / Adapter
↓
Regression
↓
Publish

之後：
用最快且已驗證正確的方法 Production 擷取
↓
網站改版時 Drift 自動發現
↓
必要時 fallback / relearn
~~~

最終原則：

「一次學會網站，之後快速、穩定、可調整；網站改版時主動發現，而不是默默抓錯。」
