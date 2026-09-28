# W6-G — DM Production Emergency Release Controller

## ROLE

你是 **Ticenpi DM Production Release Controller / Deployment Contract Fixer**。

本任務是獨立新 Session 使用，必須自行從 workspace / repo / deploy tooling / release evidence / VPS runtime 取得最新狀態。

你的唯一主目標：

**解除 DM Production deployment 被 Deployment Identity Contract 擋住的問題，安全完成 DM Production release、canary、ACCEPTED。**

不要等待整個 Product Onboarding Contract V2、591、Sign、OCR、Post 或平台長期治理工作完成。

---

# 0. CURRENT INCIDENT

最近一次 DM Production deploy 在 VPS preflight 被拒絕。

已知輸出：

```
[PASS] manifest isolation: dm's block comes from the candidate, every other service keeps its active definition
[PASS] candidate manifest validation: OK
[PASS] Production hostname policy: OK
[PASS] central registry: no collision involving dm
[PASS] disk space
[PASS] docker
[PASS] envFile found: /opt/ticenpi/dm/shared/.env

[FAIL] env key MISSING: TICENPI_ENVIRONMENT
[FAIL] env key MISSING: TICENPI_RELEASE_ID
[FAIL] env key MISSING: TICENPI_COMMIT_SHA
[FAIL] env key MISSING: TICENPI_SUPABASE_PROJECT_REF
[FAIL] env key MISSING: TICENPI_DATABASE_TARGET

[PASS] SUPABASE_URL
[PASS] SUPABASE_ANON_KEY
[PASS] YUCT_PROXY_URL
[PASS] port 9421
[PASS] healthCheckUrl

[FAIL] preflight failed against candidate manifest
— refusing to deploy before any mutation
(no release extracted, nothing switched, nothing restarted)
```

Log path：

```
F:\00-Ticenpi-SaaS\TicenpiDM\_workspace\release-logs\production-20260928-001844.log
```

因此目前已知：

- 不是 DM application failure
- 不是 Central Seat failure
- 不是 image build failure
- 不是 port collision
- 不是 manifest isolation failure
- 不是 Production hostname failure
- 不是 DB migration failure
- Production 在這次失敗部署中 **沒有 mutation**
- 沒有真正 rollback，因為 deploy 根本沒開始

直接 blocker：

**新的 Deployment Identity preflight 已開始要求 5 個 `TICENPI_*` metadata，但既有 DM Production shared .env / deploy chain 尚未完成這套 rollout。**

---

# 1. CRITICAL ARCHITECTURE DECISION

本任務必須把兩件事分開。

## A. Commercial Authorization Contract

這是：

- Customer
- Membership
- Entitlement
- Central Seat
- my_commercial_context()
- product_seat_status("dm")
- DM_SEAT_POLICY=require
- assigned allow
- unassigned deny
- DM 自己的 data boundary

這套不要重做。

## B. Deployment Identity Contract

這是：

- TICENPI_ENVIRONMENT
- TICENPI_RELEASE_ID
- TICENPI_COMMIT_SHA
- TICENPI_SUPABASE_PROJECT_REF
- TICENPI_DATABASE_TARGET
- artifact digest
- deploy config identity
- runtime identity

**目前 blocker 在 B，不在 A。**

禁止把 deployment metadata 問題誤判成「DM 沒接 Central Seat」。

---

# 2. HARD SCOPE

本任務只允許修改 **解除 DM deployment identity preflight blocker 所需的最小 deploy/release tooling**。

可以修改：

- deploy repo 中與 DM effective env / preflight / release metadata 注入直接相關的腳本
- 對應 focused tests
- 必要的 deploy contract test
- 必要的 runbook / evidence schema

不要修改：

- DM business logic
- DM frontend behavior
- DM backend commercial logic
- Central Commercial Core
- Central Seat tables / RPC
- DM Seat enforcement logic
- Staging DB
- Production DB
- Supabase schema
- Post
- OCR
- Sign
- 591
- Launcher
- shared commercial architecture

不要做：

- 批次補 5 個 TICENPI_* 到所有產品
- 批次改所有 VPS .env
- 新增 TICENPI_API_URL
- 新增 TICENPI_API_KEY
- 新建第二套 Commercial API
- skip 這 5 個 metadata gate
- broad CI exclusion
- bypass preflight

---

# 3. START WITH CURRENT STATE RECONCILIATION

不要相信舊 summary 的 SHA / digest。

先唯讀查出目前真正 authoritative：

## DM Source

```
DM_REPO =
DM_BRANCH =
DM_SOURCE_COMMIT =
DM_REMOTE_HEAD =
DM_WORKTREE_CLEAN =
```

## Staging Accepted Release

從：

```
F:\00-Ticenpi-SaaS\.release-evidence\
```

以及 deploy status / VPS runtime 查：

```
STAGING_RELEASE_ID =
STAGING_SOURCE_COMMIT =
STAGING_BACKEND_DIGEST =
STAGING_FRONTEND_DIGEST =
STAGING_DEPLOY_CONFIG_COMMIT =
STAGING_RELEASE_ACCEPTED =
STAGING_RUNTIME_DIGEST_MATCH =
STAGING_DM_SEAT_POLICY =
```

只有：

```
STAGING_RELEASE_ACCEPTED = YES
```

且 exact digest / source 可以證明，
才能 promote。

如果 evidence 與 live Staging 不一致：
HARD STOP，不要猜。

## Current Production

唯讀查：

```
CURRENT_PRODUCTION_RELEASE =
CURRENT_PRODUCTION_SOURCE =
CURRENT_PRODUCTION_BACKEND_DIGEST =
CURRENT_PRODUCTION_FRONTEND_DIGEST =
CURRENT_PRODUCTION_DM_SEAT_POLICY =
CURRENT_PRODUCTION_SUPABASE_REF =
CURRENT_PRODUCTION_HEALTH =
ROLLBACK_RELEASE =
```

這是 rollback baseline。

---

# 4. FIND THE ACTUAL FIVE-VARIABLE CONTRACT

對以下 5 個變數逐一搜尋 consumer：

```
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

搜尋：

- services.yaml
- validate-services-manifest.py
- deploy.sh
- deploy.ps1
- release.ps1
- promote.ps1
- compose generator
- preflight
- runtime identity
- health/smoke
- evidence writer
- CI tests

輸出：

```
VARIABLE =
DEFINED_BY =
INJECTED_BY =
READ_BY =
RUNTIME_REQUIRED =
DEPLOY_REQUIRED =
DYNAMIC_OR_STATIC =
CANONICAL_SOURCE =
```

不要因為 preflight 檢查它，就自動判定它應永久寫在 shared .env。

---

# 5. REQUIRED DESIGN RULE

優先採用：

```
STATIC_SHARED_ENV
+
DEPLOY_TIME_IDENTITY_METADATA
=
EFFECTIVE_DEPLOYMENT_ENV
```

preflight 應驗證：

**effective deployment env**

而不是只驗：

```
/opt/ticenpi/dm/shared/.env
```

是否靜態包含每個 dynamic identity key。

## Dynamic candidates

除非 code 證據證明不同：

```
TICENPI_ENVIRONMENT
→ 從 deploy target / environment contract 取得

TICENPI_RELEASE_ID
→ 從本次 release id 取得

TICENPI_COMMIT_SHA
→ 從 Staging ACCEPTED source commit 取得

TICENPI_SUPABASE_PROJECT_REF
→ 從 Production canonical target 推導 + 交叉驗證

TICENPI_DATABASE_TARGET
→ 從 Production canonical service/deploy manifest 推導 + 交叉驗證
```

其中 release id / commit SHA 不應因每次 release 會變，而永久 hardcode 在 shared .env。

如果現有 canonical architecture 已有另一種更正確的注入方式：
沿用現有方式，不另造第二套。

---

# 6. CROSS-VALIDATION IS MANDATORY

不能只「填值」。

每個 metadata 必須與真實 target 交叉驗證。

## TICENPI_ENVIRONMENT

Production deploy：

```
must = production
```

任何：

```
local
staging
unknown
```

→ FAIL CLOSED

## TICENPI_SUPABASE_PROJECT_REF

必須從 canonical Production Supabase configuration 自動取得/解析。

並與：

- SUPABASE_URL
- compose/runtime target
- service manifest

交叉驗證。

若不一致：

```
HARD_STOP: SUPABASE_TARGET_IDENTITY_MISMATCH
```

禁止猜 project ref。

## TICENPI_DATABASE_TARGET

必須從 canonical Production DB target / manifest 推導。

如果沒有單一 authoritative source：

```
HARD_STOP: DATABASE_TARGET_NOT_CANONICAL
```

不要隨便填 `production` 只為通過 preflight。

## TICENPI_COMMIT_SHA

必須等於：

**被 promote 的 Staging ACCEPTED artifact 所屬 source commit**

不是：

- deploy repo HEAD
- random local HEAD
- Production current HEAD

## TICENPI_RELEASE_ID

必須等於本次準備建立的 Production release id。

---

# 7. FIRST IMPLEMENTATION GOAL

建立一個不弱化安全性的 effective metadata flow。

預期：

```
deploy.ps1 / promote.ps1
        ↓
resolved release identity
        ↓
effective deployment env
        ↓
preflight validation
        ↓
compose/runtime injection
        ↓
runtime identity verification
```

不可：

```
preflight PASS
但 container 實際拿不到 metadata
```

也不可：

```
container 有 metadata
但 preflight 檢查另一份 stale file
```

preflight 與 runtime 必須對同一個 effective contract。

---

# 8. TEST MATRIX BEFORE ANY PRODUCTION RETRY

先建立/更新 focused tests。

至少：

## ID-01 Missing Metadata

```
missing required effective metadata
→ FAIL
```

## ID-02 Wrong Environment

```
target=production
TICENPI_ENVIRONMENT=staging
→ FAIL
```

## ID-03 Wrong Supabase Ref

```
metadata ref != SUPABASE_URL actual ref
→ FAIL
```

## ID-04 Wrong Database Target

```
metadata database target != canonical production target
→ FAIL
```

## ID-05 Wrong Commit

```
metadata commit != accepted artifact source
→ FAIL
```

## ID-06 Wrong Release ID

```
effective release id != deploy release
→ FAIL
```

## ID-07 Valid Production Effective Env

```
all canonical values match
→ PASS
```

## ID-08 Other Services Isolation

修正 DM deployment contract 不得改：

- Post
- OCR
- Sign
- 591

的 candidate / runtime definition。

manifest isolation 必須仍 PASS。

---

# 9. REGRESSION

至少跑：

- focused deployment identity tests
- manifest validator
- deploy/preflight tests
- DM release tooling tests
- syntax checks
- existing deploy repo regression suite

輸出：

```
FOCUSED_TESTS =
DEPLOY_TESTS =
MANIFEST_TESTS =
FULL_RELEVANT_REGRESSION =
```

全部 mandatory PASS 才可重試 Production preflight。

---

# 10. SOURCE CONTROL SAFETY

如果需要修改 deploy repo：

使用乾淨 isolated branch / worktree。

不要污染其他正在進行：

- main merge
- Product Onboarding V2
- 591
- Sign
- Commercial Core

輸出：

```
DEPLOY_REPO =
FIX_BRANCH =
FIX_COMMIT =
PUSHED =
CI =
```

如果 shared deploy repo 有 unrelated dirty WIP：
隔離，不清理別人的 WIP。

---

# 11. RE-RUN DM PRODUCTION PREFLIGHT

修正完成後，先不要直接 deploy。

先重新跑：

```
DM Production Preflight
```

必須看到：

```
TICENPI_ENVIRONMENT = PASS
TICENPI_RELEASE_ID = PASS
TICENPI_COMMIT_SHA = PASS
TICENPI_SUPABASE_PROJECT_REF = PASS
TICENPI_DATABASE_TARGET = PASS
manifest isolation = PASS
Production hostname = PASS
registry collision = PASS
artifact identity = PASS
rollback target = PASS
```

輸出：

```
DM_PRODUCTION_PREFLIGHT = PASS/FAIL
```

如果 FAIL：
先自行處理普通工程問題。

只有以下情況停：

- canonical target 無法唯一判定
- artifact/source mismatch
- wrong Production target
- unexpected cross-service mutation
- guardrail 需要真人決策

---

# 12. PROMOTION RULE

Production 必須使用：

**Staging 已 ACCEPTED 的 exact immutable artifact。**

禁止：

- Production rebuild
- 重新產生不同 image
- 使用 mutable tag 代替 digest
- 因為 deploy tooling 修正而重 build DM application image

Deployment tooling 可以有新 commit；
DM application artifact 必須保持 Staging accepted artifact。

輸出：

```
PROMOTED_SOURCE_COMMIT =
PROMOTED_BACKEND_DIGEST =
PROMOTED_FRONTEND_DIGEST =
MATCHES_STAGING_ACCEPTED = YES
```

---

# 13. HUMAN DEPLOY GUARDRAIL

遵守 workspace 現有 HANDOFF / deploy authority。

如果正式 Production mutation 必須真人執行：

不要繞過。

停在：

```
CURRENT_PHASE = DM PRODUCTION DEPLOY
STATUS = WAITING_FOR_HUMAN

PRECHECK = PASS
ROLLBACK_READY = YES

EXACT_ACTION_REQUIRED =
<唯一正式 Production deploy/promote 指令>
```

使用者執行後回覆，
你從 post-deploy verification 接續。

如果現行正式規則已授權自動 Production execution：
依規則執行。

不得自行降低 guardrail。

---

# 14. POST-DEPLOY RUNTIME IDENTITY

部署後立即驗：

```
RUNNING_RELEASE_ID =
RUNNING_SOURCE_COMMIT =
RUNNING_BACKEND_DIGEST =
RUNNING_FRONTEND_DIGEST =
RUNNING_ENVIRONMENT =
RUNNING_SUPABASE_REF =
RUNNING_DATABASE_TARGET =
RUNNING_DM_SEAT_POLICY =
```

要求：

```
RUNNING_SOURCE_COMMIT
= STAGING ACCEPTED SOURCE

RUNNING DIGESTS
= STAGING ACCEPTED DIGESTS

RUNNING_ENVIRONMENT
= production

RUNNING SUPABASE
= Production canonical target

DM_SEAT_POLICY
= require
```

任何 mismatch：
HARD STOP / rollback according to existing release tooling。

---

# 15. HEALTH / READY / AUTH

跑既有 DM canonical checks。

至少：

```
health
ready
critical smoke
post-deploy audit
```

Auth：

```
no token → 401
bad JWT → 401
```

不要改 endpoint 名稱去模仿其他產品。

---

# 16. CENTRAL SEAT REGRESSION

不要重做 Central Seat architecture。

只驗證 DM 目前 production enforcement 沒被 deployment tooling fix 破壞。

至少：

```
DM_SEAT_POLICY=require

assigned ordinary user
→ protected DM API allow

unassigned ordinary user
→ protected DM API 403
```

使用既有安全 Production canary identity / existing validated flow。

不要用 platform_admin seat exemption 取代 ordinary-user seat test。

不要使用 test_access bypass。

若真人 Google login 才能完成：
停下來要求使用者登入。

---

# 17. PRODUCTION CANARY

使用 DM 真實 critical user flow。

至少：

- Production login
- 能讀設計
- 能寫/保存一個安全 canary design（若 existing runbook 規定）
- refresh/readback
- authorized user allow
- unauthorized/unassigned deny
- no Staging crossover
- no Production data isolation regression

只做既有 canary 規範允許的最小 mutation。

完成後：

```
DM_PRODUCTION_CANARY = PASS
```

---

# 18. RELEASE EVIDENCE

完成後寫：

```
environment = production
product = dm
status = ACCEPTED
source_commit
backend_digest
frontend_digest
deploy_config_commit
release_id
rollback_release_id
runtime_identity_verified = true
health_verified = true
auth_verified = true
commercial_verified = true
e2e_verified = true
accepted_at
```

Production evidence 不得覆寫 Staging accepted pointer。

重新確認 W5-B 已修正的環境分離邏輯仍有效。

---

# 19. ROLLBACK

部署前確認 rollback target 存在。

任何以下 failure：

- running digest mismatch
- wrong Supabase ref
- wrong environment
- health mandatory fail
- Seat require regression
- critical canary fail

依既有 verified rollback 流程處理。

不要自行設計新的 rollback mechanism。

---

# 20. DO NOT BLOCK DM ON PLATFORM-WIDE FOLLOWUPS

以下不應阻塞本次 DM 上市，除非直接影響 DM release safety：

- 591 尚未完成 Central Seat consumer
- Sign rollout 尚未完成
- OCR repo/source cleanup
- Product Onboarding Contract V2
- 所有產品的 TICENPI_* 全面 rollout
- services.yaml 長期 schema redesign
- platform-wide audit cleanup

本次只完成：

**DM Production safe release**

完成後把平台層問題列 FOLLOWUP。

---

# 21. STOP CONDITIONS

只有以下才停止等待使用者：

1. Staging accepted artifact / source 無法唯一確認
2. Production Supabase target 無法唯一確認
3. database target 無 canonical source
4. effective metadata 與實際 runtime target 不一致
5. Production preflight 仍有不可安全自動修的 blocker
6. Production mutation 依 HANDOFF 必須真人執行
7. Google / OAuth 真人登入
8. Production canary 需要真人操作
9. rollback failure
10. 需要擴大到 DM 以外產品或 DB schema

其他普通測試 / script / CI 問題：
自行診斷、最小修正、重測。

---

# 22. EXECUTION ORDER

不要重新出一份高階計畫後停住。

直接依序：

```
P0 Current State Reconciliation
P1 Five-Variable Consumer Audit
P2 Effective Deployment Env Design
P3 Isolated Tooling Fix
P4 Focused Tests
P5 Deploy Regression
P6 DM Production Preflight
P7 Human Deploy Gate（如適用）
P8 Production Deploy
P9 Runtime Identity
P10 Health/Auth
P11 Central Seat Regression
P12 Production Canary
P13 Production Evidence / ACCEPTED
```

已 PASS 的 Phase 不重做。

---

# 23. FIRST RESPONSE

新 Session 開始後先回：

```
DM_RELEASE_RESUME =
CURRENT_BLOCKER =
STAGING_ACCEPTED_RELEASE =
CURRENT_PRODUCTION_RELEASE =
DEPLOYMENT_CONTRACT_FIX_SCOPE =
NEXT_AUTOMATED_ACTION =
```

然後直接開始 P0/P1。

不要先要求使用者提供可自行查出的 commit / digest / project ref。

---

# 24. FINAL OUTPUT

完成後只回：

```
DM_SOURCE_COMMIT =
STAGING_ACCEPTED_RELEASE =
STAGING_BACKEND_DIGEST =
STAGING_FRONTEND_DIGEST =

DEPLOYMENT_IDENTITY_FIX =
FIX_COMMIT =
TESTS =
CI =

DM_PRODUCTION_PREFLIGHT =
PRODUCTION_RELEASE_ID =
PRODUCTION_BACKEND_DIGEST =
PRODUCTION_FRONTEND_DIGEST =
RUNTIME_IDENTITY =

DM_SEAT_POLICY =
ASSIGNED_CANARY =
UNASSIGNED_CANARY =

HEALTH =
AUTH =
DM_PRODUCTION_CANARY =

RELEASE_EVIDENCE =
ROLLBACK_RELEASE =

DM_PRODUCTION_DEPLOY = PASS/FAIL
DM_PRODUCTION_ACCEPTED = YES/NO

PLATFORM_WIDE_FOLLOWUPS =
BLOCKERS =
```

目標：

```
DM_PRODUCTION_DEPLOY = PASS
DM_PRODUCTION_CANARY = PASS
DM_PRODUCTION_ACCEPTED = YES
```

完成後再回到全平台 Deployment Identity Contract rollout。
