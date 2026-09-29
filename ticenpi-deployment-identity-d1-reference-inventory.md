# Task D1 — TICENPI Reference Inventory

推薦模型：MiMo-V2.6-Flash / DeepSeek V4 Flash

Workspace: F:\00-Ticenpi-SaaS
Deploy repo: F:\00-Ticenpi-SaaS\deploy
必要時查 DM: F:\00-Ticenpi-SaaS\TicenpiDM

本輪只做 mechanical reference inventory；唯讀，不修改、不 commit、不 push、不 deploy，不做架構決策。

精確搜尋：
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_GIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
requiredEnv
shared/.env

每個命中分類：
READ / WRITE / INJECT / VALIDATE / DERIVE / DISPLAY / TEST_ONLY / DOC_ONLY

輸出表：
VARIABLE | FILE | LINE/SYMBOL | ACTION_TYPE | SHORT_CONTEXT

另列：
FILES_WITH_REQUIREDENV_LOGIC =
FILES_WITH_SHARED_ENV_LOGIC =
FILES_WITH_DEPLOY_TIME_INJECTION =
FILES_WITH_RUNTIME_CONSUMERS =
FILES_WITH_AUDIT_OR_ROLLBACK_CONSUMERS =

禁止決定：
- 哪個欄位該刪
- canonical source of truth
- Production 是否安全
- rollback 架構
- Central Seat / Product Registry

最後：
WORKER_TASK = D1_REFERENCE_INVENTORY
FILES_INSPECTED =
FACTS =
UNRESOLVED =
NO_ARCHITECTURE_DECISION_MADE = YES
