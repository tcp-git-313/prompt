# W6-F — Update Ticenpi Product Onboarding Contract to Central Commercial V2

## ROLE

你是 Ticenpi Platform Contract Maintainer。

本任務要做的不是替單一產品補功能，而是把既有的 **Ticenpi Product Onboarding Contract** 升級成新的平台級共用契約，讓：

- 舊產品能被重新盤查並補齊
- 新產品從一開始就不會漏接 Central Commercial / Central Seat
- Product auth / tenant / RLS 與 Central Commercial 明確分層
- Deployment Identity 與 Commercial / Seat 不再混為一談
- 未來 Planner / Executor 不需要靠「記得要接 Seat」來維持一致性

---

# 0. 先找出舊契約，不要猜檔案

工作區：

F:\00-Ticenpi-SaaS

先唯讀搜尋：

- Ticenpi Product Onboarding Contract
- onboarding
- product registration
- commercial contract
- entitlement
- Central Seat
- product_code
- service registry
- services.yaml
- platform contract
- release / delivery workflow

目標是找出：

OLD_CONTRACT_PATH =
OLD_CONTRACT_OWNER_REPO =
OLD_CONTRACT_BRANCH =
OLD_CONTRACT_HEAD =
OLD_CONTRACT_REFERENCED_BY =

如果存在多份候選：
先判斷哪一份是 authoritative。

不要新建第二套重複 contract，除非確認舊契約根本不存在。

---

# 1. 本輪邊界

允許：

- 讀 repo / docs / validators / CI / manifest
- 更新 Product Onboarding Contract
- 更新直接依賴這份契約的 validator / schema / contract tests
- 更新必要的範例 / template / docs
- 建立 focused tests
- commit / push 到隔離分支
- 開 PR（若現行 repo governance 要求）

禁止：

- 修改任何產品 runtime 邏輯
- 修改 Post / DM / OCR / Sign / 591 的商業授權程式
- 修改 VPS .env
- 修改 Staging / Production DB
- deploy
- migration
- Cloudflare mutation
- 批次補 TICENPI_* env
- 新增第二套 Customer / Entitlement / Seat 系統

本任務的交付是：
**新的平台契約 + validator / tests + legacy reconciliation 規則。**

不是把每個產品立即改完。

---

# 2. 新契約的核心原則

新的 Product Onboarding Contract 必須明確寫死：

## 2.1 Workflow parity, not file parity

所有產品遵守相同的平台生命週期與安全契約，
但 repo 結構、Docker、port、endpoint、測試框架可以不同。

## 2.2 Central Commercial 是唯一商業授權來源

商用產品不得自行建立另一套：

- customers
- memberships
- subscriptions
- entitlements
- seats

Canonical Central Commercial 介面沿用既有 Supabase RPC / Platform contract。

至少包含現有 canonical 能力：

- my_commercial_context()
- effective_entitlements(...) / effective_entitlement_details(...)
- product_seat_status(product_code)
- admin_list_customer_members(...)
- admin_list_product_seats(...)
- assign_product_seat(...)
- release_product_seat(...)

不要新增一支通用 `TICENPI_API_URL` / `TICENPI_API_KEY`，
除非現有架構真的已經改成有獨立 API gateway，且有直接證據。

產品端預設應透過：

SUPABASE_URL
+ SUPABASE_ANON_KEY / publishable key
+ 使用者自己的 JWT

呼叫 canonical RPC。

---

# 3. Product Commercial Classification

每個產品 onboarding 時必須先分類：

COMMERCIAL_PRODUCT = YES/NO

如果 NO：
必須說明為什麼是 public / internal / infrastructure / other。

如果 YES：
必須再決定：

COMMERCIAL_MODEL =
- CENTRAL_COMMERCIAL

SEAT_MODE =
- BUSINESS_SEAT_REQUIRED
- ENTITLEMENT_ONLY
- NOT_APPLICABLE

不要把「所有商用產品」簡化成一律強制人工 Seat。

Canonical 規則要尊重目前平台模型：

- business customer：依有效 entitlement + product seat assignment
- individual valid entitlement：依 canonical resolver 行為，不應被硬要求手動 Seat
- product isolation 保持存在

---

# 4. Product Registration Contract

新的契約必須要求每個產品至少能被唯一描述：

PRODUCT_NAME =
PRODUCT_CODE =
COMMERCIAL_PRODUCT =
COMMERCIAL_MODEL =
SEAT_MODE =
AUTH_MODEL =
DATA_BOUNDARY =
STAGING_TARGET =
PRODUCTION_TARGET =

PRODUCT_CODE 必須：

- canonical
- lowercase（若目前平台規範如此）
- 全平台唯一
- 不得用 UI 名稱或歷史 typo 當另一個 product

已知 OCR canonical product_code = `ocr`。
不要重新引入 `orc` 作為另一個產品代碼。

不要強迫這些欄位一定寫在 services.yaml；
先檢查目前 registry/schema 是否適合。

若 services.yaml 目前不適合承載 Product Contract：
應提出最小 schema extension 或獨立 canonical registry，
但只能有一個 authoritative source。

---

# 5. Mandatory Commercial Integration Gate

任何：

COMMERCIAL_PRODUCT = YES

的產品，在形成 Staging ACCEPTED candidate 前，
必須通過：

COMMERCIAL_INTEGRATION_GATE = PASS

這個 gate 至少驗證：

## 5.1 Auth entry

- 使用者 JWT 由 canonical auth 驗證
- invalid JWT → 401
- missing JWT → 401

## 5.2 Commercial context

產品能取得 canonical commercial context。

MY_COMMERCIAL_CONTEXT_CONSUMER = YES

## 5.3 Entitlement

產品會確認該 product_code 的有效 entitlement。

ENTITLEMENT_ENFORCEMENT = YES

## 5.4 Seat

如果：

SEAT_MODE = BUSINESS_SEAT_REQUIRED

則必須：

PRODUCT_SEAT_STATUS_CONSUMER = YES

business assigned → allow
business unassigned → 403

如果：

SEAT_MODE = ENTITLEMENT_ONLY

則不應硬要求 manual Seat。

## 5.5 Fail closed

Central Commercial / Seat dependency 無法取得可靠結果時：

FAIL_CLOSED = YES

不能默默 skip / allow。

## 5.6 Product data boundary

Central Seat 只負責「可不可以進產品」。

通過後仍必須進入產品自己的：

- tenant boundary
- RLS
- user-owned data boundary
- domain authorization

不得把 Seat 當成產品資料隔離的替代品。

---

# 6. 強制分離兩種 Contract

新契約必須明確區分：

## A. Commercial Authorization Contract

回答：

「這個 User 有沒有資格使用這個產品？」

內容：

- Customer
- Membership
- Entitlement
- Seat
- Commercial context
- Product access gate

## B. Deployment Identity Contract

回答：

「現在部署的是哪個版本、哪個環境、接哪個 target？」

可能包含：

- environment
- release id
- commit sha
- Supabase project ref
- database target
- artifact digest
- deploy config identity

目前看到的：

- TICENPI_ENVIRONMENT
- TICENPI_RELEASE_ID
- TICENPI_COMMIT_SHA
- TICENPI_SUPABASE_PROJECT_REF
- TICENPI_DATABASE_TARGET

屬於 Deployment Identity Contract，
不是 Central Seat API 設定。

在 W6-E contract audit 尚未得到 canonical rollout 結論前：

- 不要把這 5 個欄位寫成所有產品的無條件 runtime mandatory contract
- 不要因 Commercial Integration Gate 而要求批次補這 5 個 .env
- validator 要能區分 runtime required / deploy-time injected / derived metadata

---

# 7. 舊產品 Reconciliation 規則

更新後契約必須附帶 Legacy Product Reconciliation Matrix。

至少盤查：

- Post
- DM
- OCR
- Sign
- 591

每個產品輸出：

PRODUCT =
PRODUCT_CODE =
COMMERCIAL_PRODUCT =
COMMERCIAL_CORE_CONSUMER =
MY_COMMERCIAL_CONTEXT =
ENTITLEMENT_ENFORCEMENT =
SEAT_MODE =
PRODUCT_SEAT_STATUS =
SEAT_ENFORCEMENT =
FAIL_CLOSED =
PRODUCT_DATA_BOUNDARY =
STAGING_REAL_E2E =
PRODUCTION_RUNTIME_EVIDENCE =
SOURCE_IN_CANONICAL_BRANCH =
STATUS = PASS/PARTIAL/MISSING/NOT_APPLICABLE

注意：

「Central Seat 已經做好」
不等於
「每個產品 consumer 都接好了」。

不要把每個產品的 Seat integration 誤描述成「各自建立 Seat」。

應使用：

Central Seat consumer integration
或
Commercial enforcement integration。

---

# 8. 舊產品補齊策略

對 legacy product：

如果 PASS：
不要重寫。

如果 PARTIAL：
只補缺少的 consumer / enforcement / evidence。

如果 MISSING：
建立最小 Central Commercial integration。

禁止：

- 為了統一形式重構已經正確工作的產品
- 每個產品建立自己的 Customer/Seat table
- 把已有 runtime evidence 因 repo snapshot 過期而直接判定不存在

任何 legacy reconciliation 必須區分：

SOURCE_STATE
RUNTIME_STATE
EVIDENCE_STATE

三者不能混為一談。

---

# 9. 新產品 Onboarding Gate

任何新產品要進入正常 release workflow 前：

PRODUCT_REGISTRATION_GATE = PASS

至少要求：

1. product_code 已註冊
2. commercial classification 已完成
3. seat mode 已決定
4. auth model 已決定
5. data boundary 已決定
6. Commercial integration plan 已存在
7. contract tests 已存在
8. release evidence schema 已接入

如果 COMMERCIAL_PRODUCT = YES：
Commercial Integration Gate 必須是 mandatory。

不能等到 Production 前才發現「591 沒有 consumer」。

---

# 10. Mandatory Contract Tests

對商用產品，contract tests 至少覆蓋：

AUTH-01:
missing token → 401

AUTH-02:
invalid token → 401

COM-01:
valid user + valid commercial context → resolver 可取得

COM-02:
invalid/expired entitlement → deny

SEAT-01:
business assigned → allow

SEAT-02:
business unassigned → 403

SEAT-03:
individual entitlement behavior 符合 canonical resolver

FAIL-01:
commercial dependency unavailable / malformed → fail closed

DATA-01:
authorized product access 不得繞過產品自己的 tenant/RLS

DATA-02:
cross-tenant / cross-user access → deny

產品若不適用某項：
必須標 NOT_APPLICABLE + reason，
不能直接刪掉測試項目。

---

# 11. Real Staging E2E

Staging ACCEPTED 前，
synthetic JWT 只能作 regression evidence。

若產品有真人登入流程，
必須至少有 ordinary-user E2E：

- authorized user
- unauthorized user
- actual product critical API / UI
- correct Staging project
- no Production crossover

對 business Seat 模型：

assigned → product critical API allow
unassigned → product critical API deny

不要用 platform_admin / test_access 取代 ordinary-user acceptance。

---

# 12. Production Gate

Production promotion 前至少要求：

COMMERCIAL_CONTRACT_SOURCE_CANONICAL = YES
STAGING_COMMERCIAL_E2E = PASS
STAGING_RELEASE_ACCEPTED = YES
PRODUCTION_PREFLIGHT = PASS

Canonical source 不得只存在 local-only / orphan side branch，
尤其 Platform Commercial Core 的 migration / RPC / validator 必須回到 repo governance 認可的正式來源。

如果 repo 規則指定 `main` 為正式來源：
Production 所依賴的 canonical contract 必須可從 `main` 重建 / 稽核。

不要再出現：

Production 已套用
但 main 找不到 migration / contract source

的狀態。

---

# 13. Validator / CI 設計原則

新的 validator 應檢查「契約能力」而不是死背所有產品檔案長相。

應至少能驗證：

- product registration exists
- unique product_code
- commercial classification exists
- seat mode valid
- required contract tests exist
- canonical Commercial Core source reachable
- deployment metadata contract 依其 rollout policy 驗證

不要：

- 因 DM 使用某個 env / endpoint，就硬要求所有產品完全相同
- 把 deploy-time dynamic metadata 當成所有共享 .env 的靜態必填值
- 用 broad exclusions 讓 validator 失去意義

---

# 14. 與 Universal Delivery Workflow 整合

新的 Product Onboarding Contract 必須成為 Universal Delivery Workflow 的前置 Gate。

最終生命週期：

Product Registration
→ Commercial Classification
→ Commercial Integration Contract
→ Product Data Boundary
→ Contract Tests
→ Local
→ CI
→ Immutable Artifact
→ Staging
→ Real E2E
→ ACCEPTED
→ Production Promotion
→ Canary
→ GO LIVE

也就是：

Universal Delivery Workflow
不再只是
「Local → CI → Staging → Production」。

在 Local/CI 前先證明產品平台契約完整。

---

# 15. 本次更新實作方式

先做：

A. 找舊 Product Onboarding Contract
B. 讀完整內容
C. 建立 OLD → NEW gap table
D. 只在 authoritative contract 上更新
E. 同步直接依賴的 validator / schema / tests
F. 跑 focused tests
G. 跑現有 contract / manifest regression
H. 產出 Legacy Product Reconciliation template

不要在這個任務直接修改五個產品。

---

# 16. 變更前後比較

必須輸出：

OLD_CONTRACT_BEHAVIOR =
NEW_CONTRACT_BEHAVIOR =

至少回答：

1. 舊版為什麼可能讓 591 漏接 Central Seat consumer？
2. 新版用什麼 gate 防止再次發生？
3. 舊產品怎麼補、不重做？
4. 新產品怎麼一開始就強制分類？
5. Central Seat 與 Product RLS 如何分層？
6. Commercial Contract 與 Deployment Identity Contract 如何分離？
7. main / canonical source drift 如何防止？

---

# 17. 驗收標準

只有以下全部成立：

- 舊 authoritative contract 已找到
- 新契約內容已更新
- 不建立第二套 Commercial Core
- Commercial Integration Gate 已成為 mandatory
- Seat mode 有 business / individual 差異
- Product data boundary 保留
- Deployment Identity 分離
- Legacy reconciliation template 存在
- new product registration gate 存在
- validator / tests 與契約一致
- focused regression PASS
- 未修改產品 runtime / DB / VPS

才：

ONBOARDING_CONTRACT_V2 = PASS

---

# FINAL OUTPUT

OLD_CONTRACT_PATH =
OLD_CONTRACT_OWNER_REPO =
OLD_CONTRACT_HEAD =

NEW_CONTRACT_PATH =
NEW_CONTRACT_VERSION =

COMMERCIAL_INTEGRATION_GATE =
PRODUCT_REGISTRATION_GATE =
DEPLOYMENT_IDENTITY_SEPARATED =
LEGACY_RECONCILIATION_TEMPLATE =

VALIDATOR_UPDATED =
TESTS_UPDATED =
TEST_RESULTS =

CHANGED_FILES =
COMMITS =
PUSHED_BRANCH =
PR =

OLD_CONTRACT_BEHAVIOR =
NEW_CONTRACT_BEHAVIOR =

LEGACY_PRODUCT_NEXT_ACTIONS =
NEW_PRODUCT_ONBOARDING_FLOW =

ONBOARDING_CONTRACT_V2 = PASS/FAIL

BLOCKERS =
FOLLOWUPS =

最後不要 deploy。
不要修改 Staging / Production。
不要批次改產品。
本任務只把 Ticenpi Product Onboarding Contract 升級成新的平台級共用契約。
