# Task B — Deployment Identity Independent Adversarial Review

推薦模型：MiMo-V2.6-Pro High / Max

## 前置

必須先取得 Task A 完整輸出；可同時取得 D1/D2/D3 evidence。
並讀 F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md。

若 Task A 尚未完成：
REVIEW_BLOCKED = MISSING_TASK_A
停止。

本輪唯讀。你是 reviewer，不是第二個全面調查員；只讀必要 source 驗證爭議點，不重新掃整個 Repo。

## 已核准 baseline

五鍵屬 Deployment Identity，不屬 Central Seat；五鍵全部永久進 shared/.env / requiredEnv 是錯模型；目標方向為 STATIC_SHARED_ENV + DEPLOY_TIME_IDENTITY_METADATA = EFFECTIVE_DEPLOYMENT_ENV；Production same-artifact promotion 不 rebuild。Product/Service Phase 1 不重做。

## 審查

1. 對五鍵重新核對 Task A 分類：
RUNTIME_REQUIRED / DEPLOY_TIME_INJECTED / DERIVED / REDUNDANT / UNRESOLVED。
不能只因 validator 說 required 就當 runtime-required，應看實際 consumer/injection/release flow。

2. 檢查 source-of-truth duplication：
accepted-staging evidence、command target、services.yaml、SUPABASE_URL、DB config、shared/.env、compose env、runtime labels、deployment identity JSON。
指出任何可獨立漂移的重複 truth。

3. 確認 Docker→Docker 仍保證 STAGING ACCEPTED ARTIFACT = PRODUCTION PROMOTED ARTIFACT。
不得 Production rebuild、不得從 mutable branch 重推 source identity、不得用 mutable tag 取代 digest、不得改寫 accepted-staging evidence。

4. 確認移除錯誤 requiredEnv 後仍 fail closed：
environment mismatch
accepted commit != runtime commit
accepted digest != running digest
Production Supabase mismatch
DB target mismatch（若有獨立語意）
release identity 無法對應 evidence

5. 確認 rollback 還原自己的 artifact/config/release identity/environment/Supabase/DB，不繼承失敗新版 identity。

6. 審查 TICENPI_GIT_SHA vs TICENPI_COMMIT_SHA canonical 與 compatibility。

7. 審查 DATABASE_TARGET 判定。

8. 文件只審：
new-workflow.md
PRODUCT_TO_STAGING.md
STAGING_TO_PRODUCTION.md
SYSTEM_OWNERSHIP.md
PRODUCT_INTEGRATION_CONTRACT.md
TICENPI_SYSTEM_CURRENT_STATE.md

9. Scope control：不得重做 Central Seat、Commercial Core、Product Registry、Product/Service classification、tenant/RLS、OAuth。

## 最後只輸出

REVIEW_RESULT = APPROVED / CHANGES_REQUIRED
VARIABLE_CLASSIFICATION_REVIEW =
SOURCE_OF_TRUTH_REVIEW =
IMMUTABLE_PROMOTION_REVIEW =
FAIL_CLOSED_REVIEW =
ROLLBACK_REVIEW =
SHA_ALIAS_REVIEW =
DATABASE_TARGET_REVIEW =
DOC_DELTA_REVIEW =
OVERREACH_FOUND =
REQUIRED_PLAN_CHANGES =
EXECUTION_PLAN_APPROVED = YES/NO

只在 Executor 不需要重新設計時，EXECUTION_PLAN_APPROVED = YES。
