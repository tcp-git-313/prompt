# Task C — Deployment Identity Approved-Plan Executor

推薦模型：GLM-5.3 High

## 前置

必須收到：
1. Task A 完整調查結果。
2. Task B 完整 review。
3. Task B 明確輸出 EXECUTION_PLAN_APPROVED = YES。
4. final approved exact file plan。
5. current authoritative branch/commit。

缺任何一項：
EXECUTION_BLOCKED = MISSING_APPROVED_PLAN
停止。

Workspace: F:\00-Ticenpi-SaaS
Deploy repo: F:\00-Ticenpi-SaaS\deploy
System docs: F:\00-Ticenpi-SaaS\deploy\docs\system\

你是 Executor，不是 Planner。不得自行重設計 Deployment Identity。

若 source reality 要求 materially different design：
PLAN_DEVIATION_REQUIRED = YES
REASON =
EVIDENCE =
停止。

## 執行規則

1. Worktree safety：
verify repo/branch/HEAD；檢查 tracked/untracked dirty；保留 unrelated WIP；需要時 isolated worktree；禁止 git add -A。

輸出：
SOURCE_BASE =
WORKTREE =
WORKTREE_CLEAN =
UNRELATED_WIP_PRESERVED =

2. 每項修改必須有 FILE / SYMBOL or SECTION / APPROVED CHANGE / TEST。

3. 必須維持：
STATIC_SHARED_ENV + DEPLOY_TIME_IDENTITY_METADATA = EFFECTIVE_DEPLOYMENT_ENV

EXPECTED DEPLOYMENT IDENTITY =
accepted evidence + deploy target + canonical config + release tooling identity

preflight/audit 比較 expected vs effective。
禁止為了 validator PASS 把五鍵永久補進 shared/.env。

4. Tests 至少在適用時覆蓋：
- deploy-time identity 不需 persistent duplicate
- 真正 static requiredEnv 仍 enforce
- missing required secret fail
- environment mismatch fail
- Supabase mismatch fail
- source commit mismatch fail
- artifact digest mismatch fail
- release identity traceable
- rollback identity correct
- SHA alias compatibility/migration
- DATABASE_TARGET derived 行為（若核准）
- manifest collision tests
- deployment identity tests
- preflight tests

不能刪/弱化失敗 test 只求綠燈。

5. commit 前：
git status
git diff --check
git diff

每個 changed file 分類：
APPROVED_CHANGE / UNEXPECTED_CHANGE / GENERATED_CHANGE / UNRELATED_CHANGE
不能有 unexplained change。

6. 文件只更新 approved docs，可能包含：
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_TO_STAGING.md
F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TO_PRODUCTION.md
F:\00-Ticenpi-SaaS\deploy\docs\system\SYSTEM_OWNERSHIP.md
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md
F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md
F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md

TICENPI_SYSTEM_CURRENT_STATE.md 只能寫已被證據證明的現況。

7. 修改/測試後只跑 approved non-destructive preflight/dry-run。
Production mutation 需要使用者當次明確授權。

到 Production 邊界：
READY_FOR_PRODUCTION_AUTHORIZATION = YES
EXACT_COMMAND =
EXPECTED_RESULT =
ROLLBACK_TARGET =
然後停止。

8. commit/push 只依 approved plan，不得帶 unrelated files。

## 最後只輸出

EXECUTION_RESULT = PASS / FAIL / BLOCKED
SOURCE_BASE =
WORKTREE =
CHANGED_FILES =
COMMITS =
TESTS =
VALIDATOR_RESULT =
PREFLIGHT_RESULT =
IDENTITY_DERIVATION_RESULT =
IDENTITY_CROSSCHECK_RESULT =
ROLLBACK_VERIFICATION =
DOCS_UPDATED =
UNEXPECTED_CHANGES =
PLAN_DEVIATION_REQUIRED =
READY_FOR_PRODUCTION_AUTHORIZATION =
EXACT_NEXT_ACTION =
