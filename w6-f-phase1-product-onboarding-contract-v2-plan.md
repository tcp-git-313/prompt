# W6-F Phase 1 — Product Onboarding Contract V2 Discovery + Reconciliation + Plan

## ROLE

你是 Ticenpi Platform Contract Planner / Investigator。

本輪 **只做唯讀調查、reconciliation、GAP analysis、以及可交付 Executor 的精確更新計畫**。

本輪禁止修改任何程式碼、文件、設定、CI、validator、schema、產品 runtime、VPS、Staging、Production。

最終只允許停在：

PLAN_READY_FOR_APPROVAL = YES

或：

PLAN_READY_FOR_APPROVAL = NO

不要開始實作。

---

# 0. 背景與本輪要解決的問題

目前已知平台層已存在 Central Commercial / Central Seat 能力，但舊的 Product Onboarding / Universal Delivery 規劃沒有把「每個商用產品必須完成 Central Commercial consumer integration」設成足夠明確的強制 Gate。

因此出現幾種問題：

1. Central Seat 已經做好，但 591 等產品仍可能沒有 consumer integration。
2. DM 曾經已完成 Staging / Production 的 Central Seat 驗證，但後續 audit 又可能因 repo snapshot / branch / evidence drift 誤判成 shadow/WIP。
3. Product Commercial Contract 與 Deployment Identity Contract 被混在一起，例如：
   - Central Seat / Entitlement
   - TICENPI_ENVIRONMENT
   - TICENPI_RELEASE_ID
   - TICENPI_COMMIT_SHA
   - TICENPI_SUPABASE_PROJECT_REF
   - TICENPI_DATABASE_TARGET
4. Production 已使用的 Commercial Core source / migration 可能未完整回到 repo governance 認可的 canonical branch / main。
5. 舊產品、新產品缺少一致且可自動驗證的 onboarding gate。

本輪目標不是直接修，而是先把 **舊計畫為什麼失敗、正確的新計畫應該怎麼寫、哪些檔案與 validator 未來要改** 全部查清楚。

---

# 1. HARD BOUNDARY

本輪允許：

- 唯讀讀取 repo
- 唯讀讀取 branches / commits / worktrees / Git history
- 唯讀讀取 docs / contract / plan / runbook
- 唯讀讀取 CI / validator / schema / services registry
- 唯讀讀取 deploy tooling
- 唯讀讀取 Staging / Production runtime evidence
- 唯讀讀取 release evidence
- 唯讀查詢現有 Central Commercial / Seat contract source
- 唯讀盤查 Post / DM / OCR / Sign / 591

本輪禁止：

- 修改任何檔案
- commit
- push
- merge
- PR mutation
- deploy
- migration
- 修改 DB
- 修改 Supabase
- 修改 Cloudflare
- 修改 VPS .env
- restart service
- 補 Seat
- 補 entitlement
- 改 product runtime
- 改 validator
- 改 CI
- 改 services.yaml
- 批次補 TICENPI_* env
- 建立新的通用 API
- 新增第二套 Customer / Membership / Entitlement / Seat 架構

---

# 2. 先找出「舊 Product Onboarding Contract / 舊通用計畫」正本

工作區：

F:\00-Ticenpi-SaaS

搜尋至少包含：

- Product Onboarding Contract
- onboarding
- Universal Delivery Workflow
- Commercial Contract
- Central Seat
- product registration
- entitlement
- product_seat_status
- my_commercial_context
- services.yaml
- platform contract
- release contract

不要猜檔名。

找出：

OLD_PLAN_CANDIDATES =
OLD_CONTRACT_CANDIDATES =

對每份候選記錄：

PATH
REPO
BRANCH
HEAD
LAST_UPDATED
REFERENCED_BY
IS_CANONICAL

如果多份內容衝突：
先做 authoritative source reconciliation。

輸出：

AUTHORITATIVE_OLD_PLAN =
AUTHORITATIVE_OLD_CONTRACT =
WHY_AUTHORITATIVE =

如果無法判定：
PLAN_READY_FOR_APPROVAL = NO

---

# 3. 找出舊計畫真正的缺陷

必須逐條回答：

1. 舊計畫有沒有要求 commercial classification？
2. 舊計畫有沒有要求 canonical product_code？
3. 舊計畫有沒有要求 Central Commercial consumer？
4. 舊計畫有沒有要求 my_commercial_context()？
5. 舊計畫有沒有要求 entitlement enforcement？
6. 舊計畫有沒有要求 product_seat_status(product_code)？
7. 舊計畫有沒有區分 business seat / individual entitlement？
8. 舊計畫有沒有要求 fail closed？
9. 舊計畫有沒有要求 product 自己的 tenant / RLS 必須在 Seat gate 之後仍保留？
10. 舊計畫有沒有 ordinary-user Staging E2E？
11. 舊計畫有沒有 canonical source / main closure gate？
12. 舊計畫有沒有把 Deployment Identity 與 Commercial Authorization 分離？
13. 舊計畫有沒有要求 legacy product reconciliation？
14. 舊計畫有沒有防止新產品自己再造 customer / entitlement / seat？

輸出：

OLD_PLAN_DEFECTS =

每一項要附：
EVIDENCE_PATH
EVIDENCE_SECTION
IMPACT

不要只寫高階結論。

---

# 4. 重建正確的平台分層

本輪必須確認並在計畫中固定兩個不同 Contract。

## A. Commercial Authorization Contract

回答：

「這個 User 是否有資格使用某個產品？」

應盤查現有 canonical 能力：

- Customer
- Membership
- Subscription
- Entitlement
- Central Seat
- my_commercial_context()
- effective_entitlements(...)
- effective_entitlement_details(...)
- product_seat_status(product_code)
- admin_list_customer_members(...)
- admin_list_product_seats(...)
- assign_product_seat(...)
- release_product_seat(...)

確認 source 在哪裡、哪些已在 Staging / Production 實際存在。

輸出：

COMMERCIAL_CONTRACT_SOURCE =
COMMERCIAL_CONTRACT_RUNTIME_STATUS =
COMMERCIAL_CONTRACT_CANONICAL_BRANCH_STATUS =

## B. Deployment Identity Contract

回答：

「現在部署的是哪個版本、哪個環境、哪個 target？」

盤查：

- TICENPI_ENVIRONMENT
- TICENPI_RELEASE_ID
- TICENPI_COMMIT_SHA
- TICENPI_SUPABASE_PROJECT_REF
- TICENPI_DATABASE_TARGET
- artifact digest
- deploy config identity
- release evidence

輸出：

DEPLOYMENT_IDENTITY_CONTRACT_SOURCE =
DEPLOYMENT_IDENTITY_CURRENT_ROLLOUT_STATUS =

要求：

COMMERCIAL_AUTHORIZATION_CONTRACT
!=
DEPLOYMENT_IDENTITY_CONTRACT

這兩套不得再混為同一個產品 Seat 問題。

---

# 5. 盤查 Central Seat 是否真的已完成

不要因單一 branch snapshot 判斷。

必須區分：

SOURCE_STATE
RUNTIME_STATE
EVIDENCE_STATE

確認至少：

- canonical tables / RPC
- Staging runtime
- Production runtime
- Seat assignment model
- business vs individual behavior
- admin RPC
- release evidence
- main / canonical branch 是否包含 source

輸出：

CENTRAL_SEAT_PLATFORM_STATUS =
SOURCE_STATE =
STAGING_RUNTIME_STATE =
PRODUCTION_RUNTIME_STATE =
EVIDENCE_STATE =
CANONICAL_SOURCE_CLOSURE =

如果「功能已上線但 source 不在 main」：
要明確標：

PLATFORM_FUNCTIONAL = YES
SOURCE_GOVERNANCE_CLOSURE = NO

不要把兩者混成「Seat 沒做好」。

---

# 6. 舊產品逐一 Reconciliation

至少盤查：

- Post
- DM
- OCR
- Sign
- 591

每個產品必須分三層：

A. Source
B. Runtime
C. Evidence

輸出矩陣：

PRODUCT =
PRODUCT_CODE =
COMMERCIAL_PRODUCT =
COMMERCIAL_CORE_CONSUMER =
MY_COMMERCIAL_CONTEXT =
ENTITLEMENT_ENFORCEMENT =
SEAT_MODE =
PRODUCT_SEAT_STATUS_CONSUMER =
SEAT_ENFORCEMENT =
FAIL_CLOSED =
PRODUCT_DATA_BOUNDARY =
STAGING_REAL_E2E =
PRODUCTION_RUNTIME_EVIDENCE =
SOURCE_IN_CANONICAL_BRANCH =
STATUS = PASS / PARTIAL / MISSING / NOT_APPLICABLE

特別要求：

## Post
查實際 authoritative runtime / source，不要因舊的 skip config 直接覆蓋已存在的 require runtime evidence。

## DM
既然歷史上曾做過 Central Seat assigned/unassigned 與 Production rollout，
必須找出：
- 當時 source commit
- deploy config
- runtime evidence
- accepted evidence
- 現在為何 audit 會看到 shadow/WIP

輸出：

DM_RUNTIME_VS_SOURCE_DRIFT_ROOT_CAUSE =

## OCR
canonical product_code = `ocr`。
不要把 `orc` 當成新 product。

## Sign
區分：
- integration code 是否存在
- 是否 committed
- 是否 pushed
- 是否 Staging accepted
- 是否 Production

## 591
確認：
- 是否真的沒有 consumer
- 或只是 consumer 在另一 branch / worktree / artifact
- 如果真的缺，缺哪一層

輸出：

591_MISSING_INTEGRATION_LAYER =

---

# 7. Commercial Product Classification 設計

新的計畫必須先要求：

COMMERCIAL_PRODUCT = YES/NO

若 YES：

COMMERCIAL_MODEL = CENTRAL_COMMERCIAL

再決定：

SEAT_MODE =
- BUSINESS_SEAT_REQUIRED
- ENTITLEMENT_ONLY
- NOT_APPLICABLE

不要把所有商用產品一律等同「一定手動 Seat」。

必須尊重現有 canonical resolver：

- business customer：entitlement + seat assignment
- individual valid entitlement：依 canonical resolver，不應被硬加 manual seat
- product data isolation 仍由產品自己的 RLS / tenant / user-owned boundary 負責

輸出：

PROPOSED_COMMERCIAL_CLASSIFICATION_RULE =

---

# 8. 新 Product Registration Gate 設計

新的計畫要定義所有新產品在 Local/CI 之前必須完成：

PRODUCT_REGISTRATION_GATE

至少包含：

PRODUCT_NAME
PRODUCT_CODE
COMMERCIAL_PRODUCT
COMMERCIAL_MODEL
SEAT_MODE
AUTH_MODEL
DATA_BOUNDARY
STAGING_TARGET
PRODUCTION_TARGET

product_code 要：

- canonical
- unique
- 不使用歷史 typo
- 可由 validator 驗證

輸出：

PROPOSED_PRODUCT_REGISTRATION_GATE =

---

# 9. 新 Commercial Integration Gate 設計

凡：

COMMERCIAL_PRODUCT = YES

必須在形成 Staging candidate 前通過：

COMMERCIAL_INTEGRATION_GATE = PASS

至少包含：

AUTH-01 missing JWT → 401
AUTH-02 invalid JWT → 401
COM-01 commercial context resolver works
COM-02 invalid/expired entitlement → deny
SEAT-01 business assigned → allow
SEAT-02 business unassigned → 403
SEAT-03 individual entitlement follows canonical resolver
FAIL-01 commercial dependency unavailable/malformed → fail closed
DATA-01 authorized user still subject to product tenant/RLS
DATA-02 cross-tenant/cross-user → deny

不適用項目要：
NOT_APPLICABLE + reason

不能直接刪掉。

輸出：

PROPOSED_COMMERCIAL_INTEGRATION_GATE =

---

# 10. 新舊產品共用的 Contract Test 策略

計畫要定義：

- 哪些是 platform-level contract test
- 哪些是 product-level consumer test
- 哪些是 Staging ordinary-user E2E
- 哪些是 Production canary

不要要求所有產品有相同 endpoint 或檔案。

要求的是能力 parity。

輸出：

PLATFORM_CONTRACT_TESTS =
PRODUCT_CONSUMER_TESTS =
STAGING_ACCEPTANCE_TESTS =
PRODUCTION_CANARY_TESTS =

---

# 11. Validator / CI 計畫

先盤查現有：

- validate-services-manifest.py
- services.yaml
- CI policy
- product registry
- schema
- release tooling

找出哪些 validator 現在是在檢查：

「能力」

哪些是在死檢查：

「固定欄位 / env / file shape」

特別針對 TICENPI_*：

- 不要先假設全部 runtime mandatory
- 讀取 W6-E audit（若已有）
- 把 runtime required / deploy-time injected / derived metadata 分開

新的計畫要提出：

VALIDATOR_CAPABILITY_MODEL =
DEPLOYMENT_METADATA_ROLLOUT_MODEL =

並回答：

是否需要 services.yaml schema extension？
是否需要獨立 product registry？
哪一個應是 authoritative？

不能同時造兩個 source of truth。

---

# 12. Canonical Source / main Closure Gate

新的計畫必須解決：

Production 已存在功能，但 main / canonical branch 找不到 source 的問題。

盤查 repo governance：
如果 main 是正式來源，則新的計畫必須要求：

CANONICAL_SOURCE_CLOSURE_GATE = PASS

至少驗：

- migration source 在 canonical branch
- RPC source 在 canonical branch
- validator / tests 在 canonical branch
- release evidence 可追到 canonical source
- Production 可以從 canonical source 重建 / 稽核

不要允許：

Production deployed
+
source only on orphan/local side branch

長期存在。

輸出：

PROPOSED_CANONICAL_SOURCE_CLOSURE_GATE =

---

# 13. Legacy Product Reconciliation 流程設計

新的計畫不能要求舊產品全部重做。

必須定義：

PASS
→ 不動 runtime，只補缺 evidence / source closure（若需要）

PARTIAL
→ 只補缺少的 consumer / enforcement / evidence

MISSING
→ 建立最小 Central Commercial consumer integration

NOT_APPLICABLE
→ 保留理由

每個產品修正前都必須先區分：

SOURCE_STATE
RUNTIME_STATE
EVIDENCE_STATE

輸出：

PROPOSED_LEGACY_RECONCILIATION_FLOW =

---

# 14. Universal Delivery Workflow 要怎麼改

新的計畫必須把 Product Onboarding Contract 放在原本 Universal Delivery Workflow 前面。

舊：

Local
→ CI
→ Staging
→ Production

新：

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

輸出：

PROPOSED_UNIVERSAL_WORKFLOW_V2 =

---

# 15. 精確列出 Phase 2 將修改的檔案

本輪不能修改，但必須把 Phase 2 的修改範圍查清楚。

對每個未來要改的檔案輸出：

FILE_PATH =
REPO =
CURRENT_PURPOSE =
PLANNED_CHANGE =
WHY =
DEPENDENCIES =
VALIDATION =

分類：

MODIFY
ADD
DELETE
NO_CHANGE

不要寫：

「更新相關文件」

必須列出實際 path。

如果 exact path 無法確認：
PLAN_READY_FOR_APPROVAL = NO

---

# 16. Phase 2 執行順序

計畫必須拆成可以交給 Executor / Luna 的步驟。

每個 Step：

STEP_ID =
WORKDIR =
FILE_PATH =
SYMBOL / SECTION =
CHANGE =
COMMAND =
EXPECTED_RESULT =
FAILURE_ACTION =
STOP_ON_FAIL = YES/NO

Phase 2 至少包含：

1. authoritative contract update
2. product registration schema / registry
3. validator update
4. contract tests
5. legacy reconciliation template
6. Universal Delivery Workflow update
7. focused regression
8. manifest / CI regression
9. commit
10. push
11. PR / review

本輪不執行。

---

# 17. 計畫品質 Gate

只有以下都回答清楚才：

PLAN_READY_FOR_APPROVAL = YES

必須回答：

1. 舊 Contract 正本在哪？
2. 舊 Universal Plan 正本在哪？
3. 舊計畫為什麼讓 591 漏接？
4. Central Seat 平台本身目前做到哪？
5. DM 為什麼 source/runtime/evidence 看起來矛盾？
6. Post / DM / OCR / Sign / 591 各自缺什麼？
7. 哪些產品真的要 Seat？
8. individual entitlement 怎麼處理？
9. Product RLS/tenant 怎麼保留？
10. Deployment Identity 如何與 Commercial Contract 分離？
11. validator 要改哪些檔案？
12. Universal Workflow 要改哪些檔案？
13. main/canonical source closure 怎麼強制？
14. Phase 2 exact files 是哪些？
15. Phase 2 exact steps 是哪些？
16. Phase 2 測試如何證明不破壞既有產品？
17. 哪些是平台規則，哪些是產品細節？
18. 哪些技術債不應在本次順手擴大處理？

只要有一項需要 Executor 自己猜：
PLAN_READY_FOR_APPROVAL = NO

---

# FINAL DELIVERABLE

輸出完整計畫：

PROJECT = Ticenpi Platform
TASK = Product Onboarding Contract V2

AUTHORITATIVE_OLD_PLAN =
AUTHORITATIVE_OLD_CONTRACT =

CENTRAL_SEAT_PLATFORM_STATUS =
COMMERCIAL_CONTRACT_SOURCE =
DEPLOYMENT_IDENTITY_CONTRACT_SOURCE =

OLD_PLAN_DEFECTS =

LEGACY_PRODUCT_MATRIX =

PROPOSED_COMMERCIAL_CLASSIFICATION_RULE =
PROPOSED_PRODUCT_REGISTRATION_GATE =
PROPOSED_COMMERCIAL_INTEGRATION_GATE =
PROPOSED_CANONICAL_SOURCE_CLOSURE_GATE =
PROPOSED_LEGACY_RECONCILIATION_FLOW =
PROPOSED_UNIVERSAL_WORKFLOW_V2 =

VALIDATOR_CAPABILITY_MODEL =
DEPLOYMENT_METADATA_ROLLOUT_MODEL =

PHASE2_EXACT_FILE_PLAN =
PHASE2_EXECUTION_STEPS =
PHASE2_TEST_PLAN =

UNRESOLVED_DECISIONS =
BLOCKERS =

PLAN_READY_FOR_APPROVAL = YES/NO

最後停下來。

不要修改。
不要 commit。
不要 push。
不要 deploy。

等使用者核准後，才建立 Phase 2 Executor Prompt。
