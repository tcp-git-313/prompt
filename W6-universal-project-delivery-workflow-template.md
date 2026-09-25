# W6 — Universal Project Delivery Workflow Template

## ROLE
PROJECT DELIVERY / RELEASE WORKFLOW OWNER

## 推薦模型
GPT-5.6 Sol High / OPUS 5.5

## 使用方式
把下面這份提示詞直接交給任何 Ticenpi 專案。

只需要先填：

PROJECT_NAME =
PROJECT_REPO =
PROJECT_TYPE =
PRIMARY_RUNTIME =
HAS_FRONTEND =
HAS_BACKEND =
USES_DOCKER =
USES_SUPABASE =
USES_R2 =
USES_CENTRAL_COMMERCIAL_CORE =
USES_CENTRAL_SEAT =
USES_CLOUDFLARE =
STAGING_DOMAIN =
PRODUCTION_DOMAIN =

如果某項不適用就填 N/A。

---

# 核心原則

本任務要求的是：

**流程一致，不是檔案一致。**

不得要求不同專案必須：

- 用相同 repo 結構
- 用相同 compose
- 用相同 Dockerfile
- 用相同 port
- 用相同 health endpoint
- 用相同 Supabase schema
- 用相同 R2 bucket
- 用相同 Central Seat 模型
- 用相同 CI job
- 用相同測試框架

但所有專案都必須具備等價的 delivery / release lifecycle。

標準流程：

Local Source
→ Local Runtime / Local Docker
→ Clean Candidate
→ CI
→ Immutable Artifact
→ Staging Preflight
→ Staging Deploy
→ Runtime Identity
→ Health / Smoke
→ Auth / Data Isolation / Commercial Gate
→ Real E2E
→ Release Evidence
→ STAGING ACCEPTED
→ Production Preflight
→ Promote Same Artifact
→ Production Canary
→ GO LIVE

---

# 一、先盤查，不要先改

先確認目前專案：

1. repo / branch / HEAD
2. dirty tracked / untracked files
3. local run 方式
4. frontend / backend / worker / service boundary
5. Docker / compose
6. CI
7. artifact / image
8. deploy scripts
9. Staging
10. Production
11. health / ready / smoke
12. rollback
13. runtime identity
14. auth
15. RLS / tenant / data boundary
16. commercial entitlement
17. Central Seat 是否適用
18. Supabase
19. R2 / object storage
20. Cloudflare / nginx / reverse proxy
21. release evidence
22. known drift / technical debt

先輸出：

CURRENT_ARCHITECTURE =
CURRENT_DELIVERY_FLOW =
CURRENT_GAPS =

盤查完成前不要修改。

---

# 二、Source of Truth 與主從關係

所有專案都要先定義以下主從關係。

## 1. Git / Local Source

Git / Local Source
= 程式碼與設定的 Source of Truth。

Docker container、Docker volume、Staging runtime、Production runtime
都不能反向覆蓋 source code。

---

## 2. Local Runtime / Docker

Local Docker
= 本機執行與驗證環境。

規則：

- container 內產生的 source 變更不得反寫主 repo
- Docker volume 不得自動覆蓋 Local Source
- hydrate / seed / test 不得修改另一環境資料
- local persistence 只屬於 DEV

如果有 bind mount：
先確認方向與 ownership。

---

## 3. Local Data

Local Data
= DEV 測試資料。

除非有明確 export/import 任務：
不得自動同步至 Staging。

---

## 4. Staging Data

Staging DB / Supabase / R2
= 獨立 Staging runtime environment。

規則：

- schema 必須可追溯到 Git migration
- runtime 狀態不能取代 migration source
- Staging R2 bucket 與 Local / Production 隔離
- 不自動雙向同步
- test fixture 與 Production data 分離

---

## 5. Production Data

Production DB / Supabase / R2
= 獨立 Production environment。

Production 不接收：

- Local Docker volume
- Local DEV data
- Staging 測試資料整包複製

Production Promotion 主要提升：

- accepted immutable artifact
- migration
- config contract
- env contract

不是把 Staging database / bucket 整包搬過去。

---

# 三、Clean Candidate

如果主工作區有 unrelated dirty WIP：

建立 isolated branch / worktree。

要求：

SOURCE_REPO =
SOURCE_BRANCH =
APP_SOURCE_COMMIT =
WORKTREE_CLEAN = YES

不要清理、覆蓋或偷偷提交 unrelated WIP。

---

# 四、Local Validation

依專案實際架構決定測試類型。

可包含：

- unit
- integration
- frontend
- backend
- API
- auth
- RLS
- tenant isolation
- storage
- scraper
- adapter
- worker
- smoke
- contract

不要照抄其他專案測試名稱。

標準只有一個：

**該專案 critical behavior 必須能在 Local 可重現。**

輸出：

LOCAL_TESTS = PASS/FAIL

---

# 五、CI

CI 至少要能產生或證明：

APP_SOURCE_COMMIT
CI_RUN_ID
CI_RESULT
IMMUTABLE_ARTIFACT_ID

如果 Docker：

IMAGE_DIGEST = sha256:...

如果前後端分開：

BACKEND_IMAGE_DIGEST
FRONTEND_IMAGE_DIGEST

如果不是 Docker：

使用該 runtime 的 immutable artifact identity。

不要為了格式一致而強迫拆 image。

---

# 六、Artifact Provenance

所有專案都必須有等價 provenance：

SOURCE SHA
→ CI
→ IMMUTABLE ARTIFACT
→ DEPLOY CONFIG
→ RUNNING ARTIFACT
→ RUNTIME SOURCE IDENTITY

禁止只靠：

latest
staging
main
mutable tag

作為唯一 release identity。

---

# 七、Deploy Config

至少分開記錄：

APP_SOURCE_COMMIT
ARTIFACT_DIGEST / ARTIFACT_ID
DEPLOY_CONFIG_COMMIT
RELEASE_ID
ROLLBACK_RELEASE_ID

不要把一個 SHA 當成所有 identity。

---

# 八、Staging Preflight

每個專案依自身架構檢查：

- target environment
- domain
- port
- env
- DB / Supabase
- R2 / storage
- required secret presence
- artifact
- deploy config
- rollback target
- manifest collision
- shared cookie / SSO isolation
- reverse proxy
- Cloudflare
- migration drift

如果某項不適用：
標 N/A，不要硬加。

輸出：

STAGING_PREFLIGHT = PASS/FAIL

---

# 九、Staging Deploy

目標統一操作：

.\deploy.ps1 <product> staging

如果現有入口不同：
建立 compatible wrapper。

不要破壞既有 verified deploy chain。

如果 HANDOFF 規則要求真人確認正式 deploy：
AI 只做到：

DryRun
→ Preflight
→ exact command

由真人執行 confirmed mutation。

---

# 十、Runtime Identity

部署後不能只看 HTTP 200。

至少驗證：

RUNNING_ARTIFACT
RUNTIME_SOURCE_COMMIT
DEPLOY_CONFIG_IDENTITY
ENVIRONMENT
DOMAIN
PORT
DB / SUPABASE TARGET
STORAGE TARGET
POLICY FLAGS

證明：

CI artifact
=
deploy pin
=
running artifact

輸出：

RUNTIME_IDENTITY = PASS/FAIL

---

# 十一、Health / Ready / Smoke

每個專案自己定義 endpoint / command。

但需要覆蓋：

- process alive
- runtime ready
- critical dependency
- critical user flow
- fail-closed behavior

不要把 deep diagnostics 全塞進 ready。

---

# 十二、Auth / Data Boundary / Commercial Model

先辨識專案真正需要的模型。

可能是：

A. Central Commercial Core + Central Seat + tenant/RLS
B. entitlement only
C. user-owned data
D. public service
E. internal-only service
F. custom canonical model

不要因為其他產品有 Seat 就硬加 Seat。

如果使用 Central Seat：

JWT
→ commercial context
→ entitlement
→ Seat
→ project data boundary

如果不使用：
明確標 NOT_APPLICABLE。

至少驗：

authorized → allow
unauthorized → deny
invalid/no token → canonical deny
cross-tenant → deny
commercial dependency unavailable → fail closed

---

# 十三、Storage / R2 / File Ownership

如果專案有 R2 / object storage：

先定義：

LOCAL_BUCKET / LOCAL_FILES
STAGING_BUCKET
PRODUCTION_BUCKET

三者不得自動互同步。

需要 migration 時：
必須是 explicit one-way operation。

不得因 Docker startup / test / seed
把 Staging 或 Production storage 覆蓋。

---

# 十四、Real Staging E2E

使用真實 Staging runtime。

依專案 critical flow 定義 E2E。

至少：

- login / auth
- main user flow
- read/write cycle（如適用）
- authorization
- data isolation
- storage isolation（如適用）
- external integration（如適用）
- no Staging/Production crossover

輸出：

STAGING_E2E = PASS/FAIL

---

# 十五、Release Evidence

每個產品接入：

F:\00-Ticenpi-SaaS\.release-evidence\<product>\

Evidence 至少包含：

product
environment
status
source_commit
artifact_digest / artifact_id
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

mandatory E2E 全 PASS 才：

STAGING_RELEASE_ACCEPTED = YES

---

# 十六、Status Command

目標：

.\release.ps1 <product> status

至少顯示：

STAGING_RELEASE_ACCEPTED
SOURCE_COMMIT
ARTIFACT_ID
DEPLOY_CONFIG_COMMIT
STAGING_RELEASE_ID
ROLLBACK_RELEASE_ID

---

# 十七、Production Promotion

目標：

.\promote.ps1 <product> production

規則：

- 只讀 ACCEPTED Staging evidence
- Production 不重新 build
- 使用同一 immutable artifact
- Production preflight
- rollback ready
- failure 只 rollback 當前 product
- 不影響其他 product

---

# 十八、Production Preflight

依專案實際架構檢查：

- Production target
- domain / port
- env
- DB / Supabase
- R2 / storage
- secrets presence
- migration/schema drift
- accepted artifact
- rollback
- reverse proxy
- Cloudflare
- cookie / SSO isolation
- external integration prerequisites

輸出：

PRODUCTION_PREFLIGHT = PASS/FAIL

---

# 十九、Production Canary

依專案實際功能決定。

例如可能包含：

- real login
- create/read/update
- external API
- storage upload/download
- webhook
- publish
- email
- OCR
- scraper
- tenant isolation

不要照抄別的產品。

PASS 後：

PRODUCTION_RELEASE_ACCEPTED = YES
GO_LIVE = YES

---

# 二十、同步規則總表

必須輸出：

SOURCE_OF_TRUTH_MATRIX

至少列：

Source Code
Local Docker
Local Data
Staging DB
Staging Storage
Production DB
Production Storage
Migration Files
Deploy Config
Release Evidence

每一項標示：

AUTHORITATIVE
DERIVED
RUNTIME_ONLY
NEVER_REVERSE_SYNC
EXPLICIT_SYNC_ONLY

---

# 二十一、GAP Analysis

先產生：

WORKFLOW_STAGE | CURRENT_STATE | GAP | ACTION

每一項分類：

KEEP
ADAPT
ADD
FIX
NOT_APPLICABLE

只有 GAP 明確後才改。

---

# 二十二、最終統一操作目標

無論專案細節不同，最後操作應盡量統一：

.\deploy.ps1 <product> staging
.\release.ps1 <product> status
.\promote.ps1 <product> production

其中：

deploy
= 跑該專案自己的 Staging delivery chain

status
= 回報 accepted evidence

promote
= promote 同一個 accepted artifact 到 Production

---

# 最終輸出

PROJECT_NAME =

WORKFLOW_PARITY = PASS/FAIL

SOURCE_OF_TRUTH_MATRIX =
SOURCE_BASELINE =
LOCAL_RUNTIME =
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
DATA_BOUNDARY =
STORAGE_MODEL =
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

若遇到 blocker：

BLOCKER =
ROOT_CAUSE =
SMALLEST_NEXT_ACTION =

最後再次遵守：

**Workflow parity, not file parity.**
**Environment isolation, not automatic data synchronization.**
