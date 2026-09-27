# W6-E — Ticenpi Runtime Identity Env Contract Audit（唯讀，可與 main merge 平行）

## ROLE
Runtime Identity / Deployment Contract Auditor

## 推薦模型
MiMo V2.6 Pro / KIMI 3
若遇到複雜跨 repo 契約矛盾，再升級 GPT-5.6 Sol High。

## 任務背景

目前正在進行另一條平行任務：

- 商用核心分支要 merge 回 `main`
- 使用者已選擇：先併進 main
- 即使 merge 後 CI 因既有 env contract 缺口暫時紅燈，也先把它當成「已知待辦」記錄
- 本任務不得阻塞、修改或介入該 merge 任務

本任務只做一件事：

**唯讀查清楚以下 5 個 `TICENPI_*` 變數，究竟是現有 canonical runtime contract，還是新 CI policy 才新增的 metadata contract。**

目標變數：

- TICENPI_ENVIRONMENT
- TICENPI_RELEASE_ID
- TICENPI_COMMIT_SHA
- TICENPI_SUPABASE_PROJECT_REF
- TICENPI_DATABASE_TARGET

不要先假設它們應該補進所有服務。
先證明「誰定義、誰讀、缺少會怎樣、目前哪裡真的依賴」。

---

# HARD BOUNDARY

本任務完全唯讀。

禁止：

- 修改任何 repo
- 修改 main
- 修改 merge branch
- commit
- push
- 修改 CI
- 修改 deploy scripts
- 修改 VPS .env
- restart container
- deploy
- mutation
- 修改 Staging / Production
- 補任何 TICENPI_* 值
- 把 CI exclusion 放寬
- 自行建立新 contract

允許：

- repo search
- git history / blame / log
- CI workflow / logs
- deploy scripts
- compose / env template
- runtime inspect
- VPS 唯讀 env key presence 檢查
- Staging / Production 唯讀 runtime metadata
- existing evidence / reports / docs

Secrets：
只能確認 key 是否存在、來源與用途。
不得輸出 secret value。

---

# 1. PIN 本次 Audit 的基線

因為 main merge 可能正在平行進行，本任務開始時先記錄：

AUDIT_STARTED_MAIN_HEAD =
AUDIT_STARTED_COMMERCIAL_BRANCH_HEAD =
AUDIT_STARTED_MERGE_BRANCH_HEAD =
AUDIT_TIMESTAMP =

後續所有結論都要註明基於哪個 snapshot。

如果執行中 main HEAD 改變：
不要重頭跑，也不要阻塞。

只在最後補：

MAIN_MOVED_DURING_AUDIT = YES/NO
FINAL_MAIN_HEAD =

必要時只做 narrow delta check。

---

# 2. 找出每個變數的定義來源

對每個變數搜尋：

- source
- deploy repo
- compose
- shell / PowerShell
- CI workflow
- docs
- env templates
- health / runtime identity
- release tooling
- tests

逐一輸出：

VARIABLE =
FIRST_INTRODUCED_COMMIT =
DEFINED_BY =
DOCUMENTED_BY =
VALIDATED_BY =
INJECTED_BY =

不要只看目前檔案。
必要時用 git log / blame 判斷它是何時、為了什麼加入。

---

# 3. 找出真正 consumer

對每個變數找實際 read/consumer。

區分：

RUNTIME_APPLICATION
DEPLOY_SCRIPT
CI_POLICY
HEALTH_CHECK
AUDIT / EVIDENCE
TEST_ONLY
DOCUMENTATION_ONLY

逐一輸出：

READ_BY =
READ_LOCATION =
READ_TIME =
RUNTIME_CRITICAL = YES/NO
DEPLOY_CRITICAL = YES/NO
CI_ONLY = YES/NO

如果根本沒有 runtime consumer：
不要把它標成 runtime required。

---

# 4. 缺少時會怎樣

對每個變數確認：

DEFAULT_BEHAVIOR_IF_MISSING =
FAIL_CLOSED =
FAIL_OPEN =
FALLBACK_VALUE =
CONTAINER_STARTS =
DEPLOY_BLOCKED =
CI_BLOCKED =

要用 code / test / script 證據，不要猜。

特別確認：

- 缺少 TICENPI_ENVIRONMENT 是否會阻止 app 啟動？
- 缺少 TICENPI_RELEASE_ID 是否只是少 runtime identity metadata？
- 缺少 TICENPI_COMMIT_SHA 是否會影響 health / audit？
- TICENPI_SUPABASE_PROJECT_REF 是否真的由 runtime 做 cross-environment guard？
- TICENPI_DATABASE_TARGET 是否有實際 consumer，還是只由 policy 檢查？

---

# 5. 檢查各產品目前狀態

至少盤查：

- Post
- DM
- Sign
- 591
- OCR

如果還有同一 policy 涵蓋的服務，也列出。

對每個服務分別檢查：

SERVICE =
STAGING_ENV_FILE_PATH =
PRODUCTION_ENV_FILE_PATH =

TICENPI_ENVIRONMENT_STAGING_PRESENT =
TICENPI_RELEASE_ID_STAGING_PRESENT =
TICENPI_COMMIT_SHA_STAGING_PRESENT =
TICENPI_SUPABASE_PROJECT_REF_STAGING_PRESENT =
TICENPI_DATABASE_TARGET_STAGING_PRESENT =

TICENPI_ENVIRONMENT_PRODUCTION_PRESENT =
TICENPI_RELEASE_ID_PRODUCTION_PRESENT =
TICENPI_COMMIT_SHA_PRODUCTION_PRESENT =
TICENPI_SUPABASE_PROJECT_REF_PRODUCTION_PRESENT =
TICENPI_DATABASE_TARGET_PRODUCTION_PRESENT =

只報 PRESENT / MISSING / NOT_APPLICABLE。
不要打印 secrets 或 sensitive values。

對非秘密 metadata（environment / project ref 類）若需要比對，可以輸出 canonicalized identity，但不要輸出其他 credential。

---

# 6. 查 runtime 是否已經有「等價資訊」

即使沒有 TICENPI_*，也要查產品是否已透過其他欄位提供相同資訊。

例如可能已有：

- ENV
- APP_ENV
- RELEASE
- RELEASE_ID
- COMMIT
- GIT_SHA
- SUPABASE_URL
- DATABASE_URL
- product-specific release metadata

逐服務輸出：

EQUIVALENT_ENVIRONMENT_FIELD =
EQUIVALENT_RELEASE_FIELD =
EQUIVALENT_COMMIT_FIELD =
EQUIVALENT_SUPABASE_IDENTITY =
EQUIVALENT_DATABASE_TARGET =

如果 TICENPI_* 只是重複 existing metadata：
明確標：

DUPLICATES_EXISTING_CONTRACT = YES

---

# 7. 驗證 CI policy 本身

找出新 CI policy 的 exact file / script / test。

確認：

CI_POLICY_PATH =
CI_POLICY_INTRODUCED_BY =
CI_POLICY_SCOPE =
CI_POLICY_SERVICES =

回答：

1. policy 是單純檢查 key presence？
2. 還是會核對 value 與 services.yaml / runtime？
3. 是否所有服務真的共享同一 contract？
4. 是否有 rollout / compatibility 邏輯？
5. 是否把「新 metadata 標準」誤當成「既有 runtime mandatory contract」？

輸出：

CI_POLICY_TYPE =
PRESENCE_ONLY =
VALUE_VALIDATION =
BACKWARD_COMPATIBILITY =
ROLLOUT_STRATEGY_EXISTS =

---

# 8. 比對 deploy / runtime contract

對每個服務回答：

A. 目前 deploy 成功是否依賴這 5 個 TICENPI_*？
B. 目前 runtime 正常是否依賴這 5 個？
C. health / smoke 是否依賴？
D. release evidence 是否依賴？
E. rollback 是否依賴？

輸出：

SERVICE =
DEPLOY_REQUIRES_TICENPI_METADATA =
RUNTIME_REQUIRES_TICENPI_METADATA =
HEALTH_REQUIRES_TICENPI_METADATA =
EVIDENCE_REQUIRES_TICENPI_METADATA =
ROLLBACK_REQUIRES_TICENPI_METADATA =

---

# 9. 確認 BLOCK-ARM64-005 引用是否真的相關

如果 repo / docs / scripts 提到 BLOCK-ARM64-005：

查出事件的直接 root cause。

回答：

BLOCK_ARM64_005_ROOT_CAUSE =
MISSING_VARIABLE =
SAME_AS_THESE_5_VARIABLES = YES/NO
DIRECT_EVIDENCE =

不要把「曾發生某個 env 缺失事故」直接推論成這五個變數全部 runtime mandatory。

---

# 10. 分類這 5 個變數

每個變數只能選一個主要分類：

A. EXISTING_RUNTIME_REQUIRED
B. EXISTING_DEPLOY_REQUIRED
C. EXISTING_IDENTITY_METADATA
D. NEW_CI_GOVERNANCE_METADATA
E. REDUNDANT_WITH_EXISTING_METADATA
F. UNUSED / DOCUMENTATION_ONLY
G. MIXED_BY_SERVICE

輸出：

TICENPI_ENVIRONMENT_CLASS =
TICENPI_RELEASE_ID_CLASS =
TICENPI_COMMIT_SHA_CLASS =
TICENPI_SUPABASE_PROJECT_REF_CLASS =
TICENPI_DATABASE_TARGET_CLASS =

---

# 11. 決定該不該補 VPS .env

對每個服務輸出：

SERVICE =
SAFE_TO_BACKFILL = YES/NO
BACKFILL_NEEDED = YES/NO
BACKFILL_REASON =
SOURCE_OF_TRUTH_FOR_VALUE =
RUNTIME_RESTART_REQUIRED = YES/NO
RISK_IF_WRONG =

重要：

如果 value 沒有單一 canonical source，
SAFE_TO_BACKFILL 必須是 NO。

尤其：

TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET

不得靠猜。

---

# 12. 提出 rollout 方案，但不要執行

最後只能提出方案，不修改。

根據證據選：

## PLAN A — Backfill runtime metadata
適用：
這 5 個是既有 canonical deployment/runtime contract，
只是 VPS env 漏填。

## PLAN B — Transitional CI compatibility
適用：
這是新 governance metadata，
現有服務還沒正式採用。
應先讓 policy 有明確 rollout，不直接修改 5 個 runtime。

## PLAN C — Mixed rollout
適用：
只有部分變數 / 部分服務已有 canonical contract。

## PLAN D — Policy defect
適用：
CI policy 與現有 deploy/runtime contract 明顯不一致。

輸出：

RECOMMENDED_ROLLOUT_PLAN = A/B/C/D
WHY =

不要實作。

---

# 13. 與 main merge 的關係

本任務不能阻止 main merge。

明確回答：

CAN_MAIN_MERGE_PROCEED_WITH_KNOWN_RED_CI = YES/NO

判斷基準：

- merge 本身是否已驗證內容安全
- CI 紅燈是否由既有 contract gap 造成
- 是否會讓 main 產生無法部署/無法救援的狀態
- 是否只是 governance rollout 未完成

如果 YES：
說明這是 known follow-up。

如果 NO：
必須指出具體會破壞什麼，不能只因「CI red」就判 NO。

---

# FINAL OUTPUT

只回：

AUDIT_STARTED_MAIN_HEAD =
FINAL_MAIN_HEAD =
MAIN_MOVED_DURING_AUDIT =

VARIABLE_AUDIT =

TICENPI_ENVIRONMENT_CLASS =
TICENPI_RELEASE_ID_CLASS =
TICENPI_COMMIT_SHA_CLASS =
TICENPI_SUPABASE_PROJECT_REF_CLASS =
TICENPI_DATABASE_TARGET_CLASS =

CI_POLICY_PATH =
CI_POLICY_TYPE =
CI_POLICY_SCOPE =
BACKWARD_COMPATIBILITY =
ROLLOUT_STRATEGY_EXISTS =

BLOCK_ARM64_005_ROOT_CAUSE =
SAME_AS_THESE_5_VARIABLES =

SERVICE_MATRIX =
EQUIVALENT_METADATA_MATRIX =

SAFE_TO_BACKFILL_MATRIX =

RECOMMENDED_ROLLOUT_PLAN =
WHY =

CAN_MAIN_MERGE_PROCEED_WITH_KNOWN_RED_CI =

BLOCKERS =
FOLLOWUPS =

不要修改任何東西。
本任務是唯讀 contract audit，可與 main merge 平行執行。
