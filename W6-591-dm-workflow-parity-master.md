# W6 — 591 Workflow Parity with DM（流程對齊，不照抄檔案）

## ROLE
591 RELEASE / DELIVERY WORKFLOW OWNER

## 推薦模型
GPT-5.6 Sol High / OPUS 5.5
若只是盤查與規劃，可用 MiMo V2.6 Pro 或 KIMI 3。

## 核心原則
本任務的目標不是把 DM 的檔案、目錄、compose、測試名稱、port、env、RPC 或程式碼複製到 591。

目標是讓 **591 的完整交付流程與 DM 的成熟流程一致**，但每個細節要依 591 自己的架構調整。

一句話：

**Workflow parity, not file parity.**

你必須保留 591 現有合理架構，只補齊或修正「流程能力」與「release contract」。

---

# 一、先盤查 591 現況，不要先改

工作區：
F:\00-Ticenpi-SaaS

591 repo：
先從 workspace 實際定位 591 repo，不要猜路徑。

Deploy：
F:\00-Ticenpi-SaaS\deploy

先確認：

1. repo / branch / HEAD
2. dirty tracked / untracked files
3. 是否已有 isolated worktree 習慣
4. local run 方式
5. frontend / backend / service boundary
6. CI workflow
7. Docker build
8. GHCR image
9. staging compose / production compose
10. deploy.ps1 / VPS deploy chain
11. health / ready / smoke
12. rollback
13. runtime identity
14. release evidence
15. auth / RLS / tenant / commercial model
16. 591 是否需要 Central Commercial Core / Seat
17. Staging 與 Production 的 domain / port / env / Supabase / DB
18. 是否有 shared cookie / SSO / nginx / Cloudflare 特殊需求
19. 目前已知技術債或 drift

輸出：

CURRENT_591_ARCHITECTURE =
CURRENT_591_DEPLOYMENT_FLOW =
CURRENT_591_GAPS =

在盤查完成前不要修改。

---

# 二、用 DM 當「流程標準」，不是檔案標準

591 最終必須具備與 DM 等價的下列流程階段。

## Stage 1 — Source Baseline

必須能確認：

SOURCE_REPO
SOURCE_BRANCH
APP_SOURCE_COMMIT
WORKTREE_CLEAN

如果主工作區有 unrelated WIP：
建立 isolated branch / worktree。

不要清理或覆蓋既有 WIP。

---

## Stage 2 — Local Development / Tests

591 自己決定需要哪些：

- unit tests
- integration tests
- frontend tests
- backend tests
- auth tests
- RLS / tenant tests
- scraper / adapter tests
- smoke tests
- contract tests

不要機械照抄 DM test suite。

要求是：

**591 自己的 critical behavior 必須在 Local 可重現、可自動驗證。**

輸出：

LOCAL_TESTS = PASS/FAIL

---

## Stage 3 — CI

591 必須有可重現 CI。

CI 最終至少能證明：

APP_SOURCE_COMMIT
CI_RUN_ID
CI_RESULT
BACKEND_IMAGE_DIGEST（若有）
FRONTEND_IMAGE_DIGEST（若有）

如果 591 是單一 image，就只保留一顆 digest。

不要為了跟 DM 一樣而硬拆 frontend/backend image。

輸出：

CI = PASS/FAIL
CI_ARTIFACT_IDENTITY =

---

## Stage 4 — Immutable Artifact

Staging / Production 都必須使用 immutable artifact。

優先：

image@sha256:...

禁止只靠：

latest
staging
main
mutable tag

標準 provenance：

SOURCE SHA
→ CI
→ IMMUTABLE DIGEST
→ DEPLOY CONFIG PIN
→ RUNNING DIGEST
→ RUNTIME SOURCE IDENTITY

要求流程一致，但 artifact 形式可依 591 調整。

---

## Stage 5 — Deploy Config

591 必須把：

APP SOURCE
ARTIFACT
DEPLOY CONFIG
RUNTIME RELEASE

四個 identity 分開。

至少記錄：

APP_SOURCE_COMMIT
IMAGE_DIGEST(S)
DEPLOY_CONFIG_COMMIT
RELEASE_ID
ROLLBACK_RELEASE_ID

不要假設 compose 結構要跟 DM 一樣。

---

## Stage 6 — Staging Preflight

在真正 deploy 前至少檢查：

- target = Staging
- domain
- port
- env
- DB / Supabase project
- required secrets presence
- compose/service identity
- image digest
- rollback target
- manifest collision
- unrelated service drift
- cookie / SSO isolation
- nginx / reverse proxy requirements

如果 591 使用 shared cookie / .ticenpi.com SSO：
特別驗證 Staging cookie 不會污染 Production。

輸出：

STAGING_PREFLIGHT = PASS/FAIL

---

## Stage 7 — Staging Deploy

目標操作介面：

.\deploy.ps1 591 staging

若現有 deploy.ps1 介面不同：
優先建立相容 wrapper，不破壞既有 chain。

遵守目前 HANDOFF 權限：
若 deploy.ps1 的 confirmed mutation 必須由真人執行，AI 只做到 DryRun / Preflight，然後提供唯一正式指令。

---

## Stage 8 — Runtime Identity

部署後不能只看 HTTP 200。

至少驗：

RUNNING_IMAGE_DIGEST
RUNTIME_SOURCE_COMMIT
DEPLOY_CONFIG_IDENTITY
ENVIRONMENT
DOMAIN
PORT
DB/SUPABASE TARGET
POLICY FLAGS

並證明：

CI digest
=
deploy pin
=
running digest

輸出：

RUNTIME_IDENTITY = PASS/FAIL

---

## Stage 9 — Health / Ready / Smoke

依 591 架構定義自己的：

health
ready
deep health
smoke

不要照抄 DM endpoint。

但要求一致：

- process alive
- runtime ready
- external dependencies 有合理驗證
- critical user flow smoke
- fail closed
- 不因 deep check 阻塞單 worker

輸出：

HEALTH = PASS/FAIL
SMOKE = PASS/FAIL

---

## Stage 10 — Auth / Data Isolation / Commercial Gate

591 必須先決定它自己的授權模型。

可能是：

A. Central Seat + 591 tenant/data scope
B. entitlement only
C. user-owned
D. 其他已存在的 canonical model

不要為了跟 DM 一樣而硬加 Seat。

如果 591 屬於商用產品且應納入 Central Commercial Core：
才整合 canonical：

JWT
→ commercial context
→ entitlement
→ Seat（若 business 模型需要）
→ 591 data boundary

並驗證：

assigned/authorized → allow
unassigned/unauthorized → deny
invalid/no JWT → 401
cross-tenant → deny
commercial backend unavailable → fail closed

輸出：

AUTH_MODEL =
COMMERCIAL_MODEL =
AUTH_E2E = PASS/FAIL

---

## Stage 11 — Real Staging E2E

使用普通測試帳號 / 真 session。

依 591 真實功能設計 E2E，不照抄 DM 的 /api/designs。

至少：

- 正常登入
- critical 591 workflow 成功
- authorized user allow
- unauthorized user deny
- data isolation
- browser/session isolation
- no Staging/Production crossover

輸出：

STAGING_E2E = PASS/FAIL

---

## Stage 12 — Release Evidence

591 要接入通用 release evidence。

目標：

F:\00-Ticenpi-SaaS\.release-evidence\591\

至少保存：

product
environment
status
source_commit
image_digest(s)
deploy_config_commit
release_id
rollback_release_id
ci_run_id
runtime_identity_verified
health_verified
auth_verified
commercial_verified
e2e_verified
accepted_at

規則：

DEPLOYED ≠ ACCEPTED

只有 mandatory Staging E2E PASS 才能：

STAGING_RELEASE_ACCEPTED = YES

---

## Stage 13 — Status / Promote

目標操作：

.\release.ps1 591 status

顯示：

STAGING_RELEASE_ACCEPTED
SOURCE_COMMIT
IMAGE_DIGEST(S)
DEPLOY_CONFIG_COMMIT
STAGING_RELEASE_ID
ROLLBACK_RELEASE_ID

Production 操作：

.\promote.ps1 591 production

規則：

- 只讀 accepted Staging evidence
- Production 不 rebuild
- 同一 immutable digest
- Production preflight
- rollback ready
- product-scoped deployment
- failure rollback only 591

---

## Stage 14 — Production Preflight

在任何 Production mutation 前：

- target identity
- domain / port
- env
- DB / Supabase
- secrets presence
- schema/migration drift
- accepted artifact
- rollback target
- manifest collision
- cookie/SSO isolation
- nginx/reverse proxy
- real production prerequisites

輸出：

PRODUCTION_PREFLIGHT = PASS/FAIL

---

## Stage 15 — Production Promotion

只能 promote Staging ACCEPTED artifact。

不要重新 build。

部署後驗：

- runtime identity
- health
- auth
- data isolation
- critical smoke
- rollback
- evidence

輸出：

PRODUCTION_DEPLOY = PASS/FAIL

---

## Stage 16 — Production Canary

依 591 真實產品功能設計真人 canary。

不要抄 DM/POST/OCR 的 canary。

要求：

- 真 Production login
- critical user flow
- write/read cycle（若功能有）
- data isolation
- authorization
- external integration（若有）
- no duplicate / unintended mutation

PASS 後：

PRODUCTION_RELEASE_ACCEPTED = YES
GO_LIVE = YES

---

# 三、必須保留的「一致性」

591 最終必須與 DM 一致的是：

1. Source baseline 可追溯
2. Dirty WIP 隔離
3. Local tests
4. CI
5. Immutable artifact
6. Deploy-config identity
7. Staging preflight
8. Staging deploy
9. Runtime identity
10. Health / smoke
11. Auth / data boundary
12. Real E2E
13. Release evidence
14. ACCEPTED gate
15. status command
16. promote command
17. Production preflight
18. Same artifact promotion
19. Rollback
20. Production canary
21. GO LIVE evidence

這些叫「流程一致」。

---

# 四、不要求一致的東西

以下都可以依 591 自己調整：

- repo 結構
- frontend/backend 是否分開
- Dockerfile 數量
- compose service 名稱
- port
- domain
- health endpoint
- smoke script 名稱
- test framework
- RPC
- database schema
- tenant model
- Central Seat 是否適用
- nginx 設定
- Cloudflare 設定
- CI job 名稱
- release artifact 數量

不要為了形式一致而破壞 591 已有合理架構。

---

# 五、先做 GAP ANALYSIS，再實作

先輸出：

DM_WORKFLOW_STAGE | 591_CURRENT_STATE | GAP | ACTION

全部階段逐項對照。

然後分類：

KEEP
ADAPT
ADD
FIX
NOT_APPLICABLE

只有 GAP 明確後才開始修改。

---

# 六、執行策略

低風險且互不衝突的工作可以平行：

- tests
- CI audit
- release evidence tooling
- docs
- read-only runtime inspection

以下必須序列化：

- DB migration
- Staging deploy
- Production deploy
- shared manifest mutation
- shared cookie / SSO changes

---

# 七、最終目標

我要最後可以只記這三個操作：

.\deploy.ps1 591 staging
.\release.ps1 591 status
.\promote.ps1 591 production

而且：

deploy
→ 自動跑 591 自己的 preflight / release chain

status
→ 告訴我目前 Staging 是否 ACCEPTED

promote
→ 只把 Staging 已驗證的同一 artifact 提升到 Production

---

# 最終輸出

591_WORKFLOW_PARITY = PASS/FAIL

SOURCE_BASELINE =
LOCAL_TESTS =
CI =
IMMUTABLE_ARTIFACT =
DEPLOY_CONFIG =
STAGING_PREFLIGHT =
STAGING_DEPLOY =
RUNTIME_IDENTITY =
HEALTH =
SMOKE =
AUTH_MODEL =
COMMERCIAL_MODEL =
STAGING_E2E =
RELEASE_EVIDENCE =
STAGING_RELEASE_ACCEPTED =
STATUS_COMMAND =
PROMOTE_COMMAND =
PRODUCTION_PREFLIGHT =
PRODUCTION_DEPLOY =
PRODUCTION_CANARY =
GO_LIVE =

KEEP =
ADAPT =
ADD =
FIX =
NOT_APPLICABLE =

CHANGED_FILES =
COMMITS =
CI_RUNS =
EVIDENCE_PATHS =
COMMAND_EXAMPLES =

若遇到真正 blocker：
BLOCKER =
ROOT_CAUSE =
SMALLEST_NEXT_ACTION =

不要回答「591 應該跟 DM 檔案一樣」。
本任務只要求 **工作流與 release contract 等價**，實作細節必須尊重 591 自身架構。
