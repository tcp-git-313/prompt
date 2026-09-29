# Legacy Production Baseline First — Universal Workflow on Next Release

## 目標

本輪只處理一件事：

目前已經完成 Staging 驗證的版本，先建立為 Production baseline。

本輪不要同步導入新的 Universal Workflow、Product Onboarding V2、Deployment Identity V2 或其他跨平台重構。

下一次真的有新的本機功能版本時，再把「新版本 + 新通用 Workflow」一起導入。

## 路徑

Workspace:
F:\00-Ticenpi-SaaS

Deploy repo:
F:\00-Ticenpi-SaaS\deploy

System docs:
F:\00-Ticenpi-SaaS\deploy\docs\system\

Future workflow:
F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md

Release evidence:
F:\00-Ticenpi-SaaS\.release-evidence\

## 本輪原則

THIS RELEASE = LEGACY_PRODUCTION_BASELINE

NEXT LOCAL RELEASE = FIRST_UNIVERSAL_WORKFLOW_GOVERNED_RELEASE

本輪不要重做：
- Product / Service Phase 1
- Product Registry
- Central Seat 架構
- Commercial Core
- Deployment Identity 正式重構
- Universal Workflow migration
- repo cleanup
- unrelated technical debt

## Phase 1 — 找出目前可升 Production 的 exact candidate

自行確認：
- TARGET_PRODUCT
- repo / branch / source commit
- Staging release status
- accepted release evidence
- immutable artifact identity
- Production target
- rollback target
- delivery model

輸出：

TARGET_PRODUCT =
DELIVERY_MODEL =
STAGING_ACCEPTED =
STAGING_RELEASE_ID =
SOURCE_COMMIT =
ACCEPTED_ARTIFACT =
PRODUCTION_CURRENT_RELEASE =
ROLLBACK_TARGET =

若 Staging 尚未 ACCEPTED、artifact/source 不明或 rollback 不明，停止，不要硬上。

## Phase 2 — 保留既有安全 gate

Production promotion 前仍須確認：
- source identity
- immutable artifact
- Production environment
- Production Supabase / DB target
- required runtime secrets/config
- rollback
- health prerequisites
- auth
- entitlement / Central Seat（適用時）

Docker → Docker 必須沿用 Staging ACCEPTED 的同一 immutable artifact，不得 Production rebuild。

若產品仍有 Staging Docker → Production systemd 且無可重現 package/equivalence proof，標記 BLOCKED；不要把這類 blocker 當成 Workflow 問題略過。

## Phase 3 — TICENPI 五鍵只視為目前已知 legacy rollout 問題

相關欄位：

TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET

目前已知方向：

五鍵屬 Deployment Identity；
「五鍵一律永久存在 shared/.env / requiredEnv」不是未來正確模型。

本輪不要正式重構這套模型，也不要為了讓 validator 通過就永久補五個值。

如果 Production preflight 唯一阻塞就是這個 legacy presence rule：

先證明等價 identity 已可由既有 authoritative evidence 確定：
- environment ← Production target
- source commit ← Staging ACCEPTED evidence
- artifact ← Staging ACCEPTED artifact
- Supabase target ← canonical Production config
- DB target ← canonical Production DB config
- release identity ← existing release/promote tooling

若其中任何一項仍無法唯一判定，停止。

若現有 tooling 已有安全的 release-scoped / deploy-time identity injection 或 compatibility path，可沿用既有能力；不要新增永久第二份 source of truth。

如果現有 tooling 沒有安全路徑，不要直接關掉整個 preflight；只輸出最小 compatibility change 與驗證方式，等使用者確認後再 mutation。

## Phase 4 — Production promotion

當且僅當：

STAGING_ACCEPTED = YES
SOURCE_IDENTITY_KNOWN = YES
ARTIFACT_IDENTITY_KNOWN = YES
PRODUCTION_TARGET_KNOWN = YES
ROLLBACK_READY = YES
OTHER_BLOCKERS = NONE

才進入 Production promotion。

使用目前 repo 已驗證的 canonical promote/deploy path，不自行發明第二套部署流程。

在真正 Production mutation 前，輸出：

READY_FOR_PRODUCTION_PROMOTION = YES
EXACT_COMMAND =
EXPECTED_RELEASE =
EXPECTED_ARTIFACT =
ROLLBACK_TARGET =

等待使用者確認後執行。

## Phase 5 — Production 驗證

完成後至少驗：
- runtime environment = production
- source identity 正確
- artifact identity 正確
- Production Supabase / DB target 正確
- health / ready
- auth deny/allow
- entitlement / Seat（適用時）
- 該產品最小核心 canary

DEPLOYED != ACCEPTED。

只有 mandatory Production canary 全部 PASS 後：

PRODUCTION_ACCEPTED = YES
GO_LIVE = YES

## Phase 6 — Evidence

記錄：

RELEASE_MODE = LEGACY_PRODUCTION_BASELINE
WORKFLOW_MIGRATION_DEFERRED = YES
SOURCE_COMMIT =
ARTIFACT_ID =
PROMOTED_FROM_STAGING_RELEASE_ID =
PRODUCTION_RELEASE_ID =
RUNTIME_IDENTITY_VERIFIED =
HEALTH_VERIFIED =
AUTH_VERIFIED =
COMMERCIAL_VERIFIED =
E2E_VERIFIED =
ROLLBACK_RELEASE_ID =

如果本輪使用任何 legacy compatibility mechanism，要明確記錄 scope；不要把它寫成永久標準。

## Phase 7 — 下一個本機新版本的強制 follow-up

成功建立 Production baseline 後留下：

NEXT_LOCAL_RELEASE_REQUIRES_UNIVERSAL_WORKFLOW_MIGRATION = YES

下一個本機版本開始時先讀：

F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md

並在下一個 release candidate 前完成正式 reconciliation：

- Deployment Identity
- requiredEnv vs deploy-time metadata
- Product Onboarding V2（適用時）
- Product / Service registry alignment（若已核准）
- source/runtime/evidence reconciliation

之後才走：

Local
→ CI
→ immutable artifact
→ Staging
→ ACCEPTED
→ Production

## 最後輸出

TARGET_PRODUCT =
DELIVERY_MODEL =
STAGING_ACCEPTED =
SOURCE_COMMIT =
ACCEPTED_ARTIFACT =
PRODUCTION_PREFLIGHT =
LEGACY_IDENTITY_RULE_BLOCKER =
OTHER_BLOCKERS =
READY_FOR_PRODUCTION_PROMOTION =
PRODUCTION_DEPLOYED =
RUNTIME_IDENTITY_VERIFIED =
HEALTH_VERIFIED =
AUTH_VERIFIED =
COMMERCIAL_VERIFIED =
PRODUCTION_CANARY =
PRODUCTION_ACCEPTED =
GO_LIVE =
RELEASE_MODE = LEGACY_PRODUCTION_BASELINE
WORKFLOW_MIGRATION_DEFERRED = YES
NEXT_LOCAL_RELEASE_REQUIRES_UNIVERSAL_WORKFLOW_MIGRATION = YES
EVIDENCE_PATH =
ROLLBACK_TARGET =
EXACT_NEXT_ACTION =
