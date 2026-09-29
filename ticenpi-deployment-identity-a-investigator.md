# Task A — Deployment Identity Remaining-Gaps Audit

推薦模型：Kimi K3 Max

## 固定背景

Workspace: F:\00-Ticenpi-SaaS
Deploy repo: F:\00-Ticenpi-SaaS\deploy
System docs: F:\00-Ticenpi-SaaS\deploy\docs\system
新版規範: F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md
Release evidence: F:\00-Ticenpi-SaaS\.release-evidence\
已知線索: fix/dm-deployment-identity-20260929 @ 1bae728

已核准 baseline，不重做：
- TICENPI_ENVIRONMENT / RELEASE_ID / COMMIT_SHA / SUPABASE_PROJECT_REF / DATABASE_TARGET 屬 Deployment Identity，不屬 Central Seat。
- 五鍵一律永久放 shared/.env / requiredEnv 是錯模型。
- 目標方向：STATIC_SHARED_ENV + DEPLOY_TIME_IDENTITY_METADATA = EFFECTIVE_DEPLOYMENT_ENV。
- Docker→Docker Production 必須沿用 Staging ACCEPTED immutable artifact，不 rebuild。
- Product / Service Phase 1、Product Registry、Central Seat 架構本輪不重做。

本輪唯讀：不修改、不 commit、不 push、不 deploy、不補 shared/.env。

## 任務

1. 追真實 call chain：
promote.ps1 → deploy.ps1 → server/preflight.sh → server/deploy.sh → compose/runtime → server/audit.sh → server/rollback.sh。

另查 server/deployment_identity.py、server/manifest_tool.py、scripts/validate-services-manifest.py、schemas/services-manifest.schema.json、tools/release_evidence_tool.py、release.ps1、services.yaml，以及 DM staging/production compose、health/ready/runtime identity consumer。

2. 對五鍵及 TICENPI_GIT_SHA 分別列：
VARIABLE / DEFINED_AT / SOURCE / INJECTED_AT / VALIDATED_AT / RUNTIME_CONSUMER / AUDIT_CONSUMER / ROLLBACK_CONSUMER / HEALTH_CONSUMER / CURRENT_REQUIRED_ENV / CURRENT_SHARED_ENV_REQUIRED。

3. 用 git log / blame / show 找出誰開始要求五鍵進 requiredEnv：
INTRODUCING_COMMIT / INTRODUCING_FILE / INTRODUCING_RULE。

4. 精確解釋最近 DM Production preflight 為何因 /opt/ticenpi/dm/shared/.env 缺五鍵而在 mutation 前 STOP：
DM_FAILURE_BRANCH / TOOLING_COMMIT / PREFLIGHT_VERSION / EXACT_CHECK / EXPECTED_SOURCE / ACTUAL_SOURCE。
判斷該失敗是早於 1bae728、正是 1bae728、或 partial rollout。

5. 實查 accepted-staging.json、history/*.json、release.ps1、promote.ps1、release_evidence_tool.py。
確認實際已有 source commit、artifact digest/id、deploy config commit、release id、runtime identity verified，以及 promote 是否已鎖 accepted artifact、不 rebuild、已知道 accepted source/release identity。

6. 每鍵分類只能選：
RUNTIME_REQUIRED / DEPLOY_TIME_INJECTED / DERIVED / REDUNDANT / UNRESOLVED。
並列 CANONICAL_SOURCE / DERIVED_FROM / MUST_EXIST_IN_SHARED_ENV / RUNTIME_NEEDS_VALUE / WHY。

7. 專查 TICENPI_GIT_SHA vs TICENPI_COMMIT_SHA：consumer、canonical、compatibility、是否造成 health/runtime/deployment identity mismatch。

8. 專查 TICENPI_DATABASE_TARGET 是否有 Supabase project 之外獨立語意；若沒有，判斷 DERIVED 或 REDUNDANT，不得直接刪除。

9. 只做 Deployment Identity 文件 delta：
new-workflow.md
PRODUCT_TO_STAGING.md
STAGING_TO_PRODUCTION.md
SYSTEM_OWNERSHIP.md
PRODUCT_INTEGRATION_CONTRACT.md
TICENPI_SYSTEM_CURRENT_STATE.md
STAGING_TEST_IDENTITY_CONTRACT.md 原則上 NO_CHANGE。

## 最後只輸出

CURRENT_CALL_CHAIN =
DM_FAILURE_EXACT_CAUSE =
INTRODUCING_COMMIT =
VARIABLE_MATRIX =
WRONG_REQUIREDENV_RULE =
PREFLIGHT_CURRENT_MODEL =
ACCEPTED_STAGING_ALREADY_KNOWS =
PROMOTE_ALREADY_KNOWS =
SHA_ALIAS_RESULT =
DATABASE_TARGET_RESULT =
CODE_FILES_TO_FIX =
DOC_SECTIONS_TO_FIX =
MINIMAL_FIX_PLAN =
TEST_PLAN =
BASELINE_CONTRADICTION = NONE / details
PLAN_READY_FOR_REVIEW = YES/NO

若 Executor 還需要自己猜 material design，PLAN_READY_FOR_REVIEW = NO。
