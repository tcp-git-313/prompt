# Task B — Deployment Identity Independent Adversarial Review

推薦模型：MiMo-V2.6-Pro High / Max

## 任務定位

你是 **Task B Reviewer**。

本輪只做：
- 讀取／核對 Task A、D1、D2、D3 已取得的證據
- 對 Deployment Identity 正式修法做獨立 adversarial review
- 找出矛盾、遺漏、過度修正、source-of-truth duplication
- 判斷未來 Executor（Task C）是否已有不需要重新設計的可執行方案

本輪禁止：
- 修改任何 repo
- commit / push
- deploy Staging
- deploy Production
- 修改 VPS
- 修改 shared/.env
- 合併 branch
- 執行 Task C
- 重掃整個 workspace
- 重做 Central Seat / Commercial Core / Product Registry / tenant / RLS / OAuth

## 與目前產品 release 的關係

**Task B / Task C / 尚未完成的 Universal Workflow 不得自動阻塞目前產品的正常 release。**

目前 DM / Post / OCR / Sign / 591 的 current-release 任務可以各自依既有 release contract：
CURRENT COMMITTED SOURCE → Staging → ACCEPTED → same-artifact/package-equivalent Production。

Task B 是未來 Deployment Identity / Workflow 正式落地的 reviewer，不是目前 release gate。

輸出必須包含：

```text
CURRENT_RELEASES_BLOCKED_BY_TASK_B = NO
TASK_C_EXECUTE_NOW = NO
TASK_C_IS_FUTURE_MIGRATION_WORK = YES
```

除非你發現一個與 Task B 本身無關、但有直接證據會造成資料毀損／環境串錯的實際 release blocker；若有，只能列證據，不得自行部署或修正。

## new-workflow.md 規則

`F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md` 目前只能當：

```text
DRAFT_REFERENCE_ONLY
```

可以用來比對「未來希望達成的方向」，但：
- 不是目前 release 的執行正主
- 不得因文件尚未完成而判 current release HARD_STOP
- 不得把 draft 內容當已落地事實
- 不得要求目前產品先完成 Universal Workflow migration 才能 release

## 已取得的 evidence baseline

不要重新全域掃描。以下是 A / D1 / D2 / D3 已收斂的 baseline，Reviewer 只針對爭議點讀 source 驗證。

### D1 baseline

- canonical `deploy\services.yaml` main 現況沒有五個 TICENPI_* identity key。
- 部分 platform/worktree manifest 曾把五鍵列入 `requiredEnv`。
- `TICENPI_GIT_SHA` 從未屬於該五鍵 requiredEnv。
- deploy-time identity injection 實際發生在 deploy / rollback runtime start path。
- 大量命中來自 clone/worktree copy，不可把數量當多個獨立 source of truth。

### D2 baseline

候選引入點：
- `b699dc2`：早期 requiredEnv validator / deploy-time identity injection 基礎
- `e9b97c1`：首次把五個 identity keys 加入 requiredEnv；**不在 main**
- `07bc10a`：per-service deployment identity；已進 main
- `fa7e324`：accepted-staging / promote evidence；已進 main
- `1bae728`：effective deployment identity preflight 修法；**當時不在 main**

Reviewer 必須重新確認「現在」branch 狀態是否已改變，但不能把歷史 branch 事實錯當 current main。

### Task A baseline

已定位的歷史 DM failure model：

```text
manifest 把五鍵列為 requiredEnv
        ↓
classic preflight 在 shared envFile 檢查每個 requiredEnv
        ↓
deploy-time runtime injection 尚未發生
        ↓
五鍵被誤判 MISSING
        ↓
mutation 前 STOP
```

Task A 方向：

```text
STATIC_SHARED_ENV
+
DEPLOY_TIME_IDENTITY_METADATA
=
EFFECTIVE_DEPLOYMENT_ENV
```

而不是：

```text
五個 TICENPI_* 永久人工寫入 shared/.env
```

### D3 baseline

- release evidence lifecycle schema 目前主要使用 `backend_digest` / `frontend_digest`，不是單一 `artifact_digest`。
- DM / Post / OCR 有 accepted-staging evidence；歷史 Production 多為 DEPLOYED、未 ACCEPTED。
- Sign / 591 的 evidence / delivery model 與 Docker→Docker 產品不同，不能套同一 promotion 假設。
- 文件之間存在命名與 current-state drift，Reviewer 必須區分「文件落後」與「實作錯誤」。

## Reviewer 必查 1 — 五鍵分類

逐一審查：

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

每個 key 都輸出：

```text
RUNTIME_NEEDS_VALUE = YES/NO
MUST_EXIST_IN_SHARED_ENV = YES/NO
SOURCE_OF_TRUTH =
INJECTION_OR_DERIVATION_POINT =
VALIDATION_POINT =
ROLLBACK_BEHAVIOR =
CLASSIFICATION =
  RUNTIME_REQUIRED
  DEPLOY_TIME_INJECTED
  DERIVED
  REDUNDANT
  UNRESOLVED
```

不得因 manifest / validator 宣告 required 就直接推論為 shared.env runtime secret。

## Reviewer 必查 2 — source-of-truth duplication

檢查以下來源是否能獨立漂移：

- accepted-staging evidence
- current product Git source
- deploy target / command target
- services.yaml
- compose / unit config
- shared/.env
- SUPABASE_URL
- database target policy/config
- runtime labels
- .release-identity.env
- .deployment-identity.json
- health/runtime public identity

找出：
- 哪些是 authoritative source
- 哪些應是 derived
- 哪些只是 evidence / observation
- 哪些重複 literal 應被消除或驗證

## Reviewer 必查 3 — preflight 正確模型

確認未來正式模型是否滿足：

```text
EXPECTED_DEPLOYMENT_IDENTITY
=
accepted release evidence
+ deploy target
+ canonical environment config
+ canonical target policy
+ release tooling identity
```

再與：

```text
EFFECTIVE_DEPLOYMENT_IDENTITY
```

比對。

必須 fail closed：
- environment mismatch
- accepted source commit != effective runtime/source commit
- accepted digest != promoted/running digest
- Production Supabase target mismatch
- database target mismatch（若有獨立語意）
- release identity 無法追溯
- rollback identity 來源不明

## Reviewer 必查 4 — immutable promotion

Docker → Docker 必須：

```text
STAGING_ACCEPTED_ARTIFACT
=
PRODUCTION_PROMOTED_ARTIFACT
```

不得：
- Production rebuild
- mutable tag 代替 digest
- 從 current branch 重新推 source identity
- 改寫舊 accepted evidence 假裝是新 release
- 以 config commit 冒充 product source commit

systemd / mixed delivery 必須有 package equivalence proof；沒有就不能假裝 same-artifact。

## Reviewer 必查 5 — rollback

確認 rollback 還原「目標 release 自己的」：
- artifact/package
- config
- release id
- source commit
- environment
- Supabase target
- DB target

不得繼承失敗新版 identity。

## Reviewer 必查 6 — SHA alias

判斷：
- `TICENPI_COMMIT_SHA` 是否為 canonical runtime identity
- `TICENPI_GIT_SHA` 是否只應保留 compatibility / build alias
- 哪些 consumer 仍依賴 alias
- 未來是否需要雙名同值 shim
- 是否存在 alias drift

## Reviewer 必查 7 — DATABASE_TARGET

確認它到底是：
- 真實獨立 runtime contract
- environment + canonical policy 可推導的 target class
- 或可刪除的重複資料

不能只因名字像 DB connection 就把它當 secret。

## Reviewer 必查 8 — 1bae728 類修法

如果目前 repo 仍存在 `1bae728` 或其後繼實作，逐檔 review：
- server/deployment_identity.py
- server/preflight.sh
- server/audit.sh
- server/deploy.sh
- deploy.ps1
- promote.ps1
- tools/release_evidence_tool.py
- tests

確認：
1. 是否真的解掉「requiredEnv 在 injection 前錯檢」。
2. 是否仍保留其他 static requiredEnv 檢查。
3. 是否引入第二份 source of truth。
4. 是否把 product-specific DM workaround 錯做成 universal rule。
5. 是否能被 rollback 正確處理。
6. 是否會誤擋其他產品。
7. 是否需要拆成更小 commit。

## Reviewer 必查 9 — 文件 delta

只審未來應修改的文件：

- new-workflow.md（DRAFT）
- PRODUCT_TO_STAGING.md
- STAGING_TO_PRODUCTION.md
- SYSTEM_OWNERSHIP.md
- PRODUCT_INTEGRATION_CONTRACT.md
- TICENPI_SYSTEM_CURRENT_STATE.md

`STAGING_TEST_IDENTITY_CONTRACT.md` 預期 NO_CHANGE，除非找到直接證據。

文件不得先寫成「已完成」，必須等程式實作與測試真的完成後才更新 current state。

## Reviewer 必查 10 — current release 與 future migration 分離

Task B 必須明確確認：

```text
CURRENT_RELEASE_PATH
!=
FUTURE_DEPLOYMENT_IDENTITY_MIGRATION
```

current product release 可以使用：
- 既有 canonical release tooling
- 現有安全 gate
- release-scoped legacy TICENPI compatibility（必要時）

但 compatibility：
- 不得永久把五鍵寫死 shared/.env
- 不得關掉整個 preflight
- 不得形成第二份 permanent source of truth
- 不得跳過 Staging ACCEPTED / artifact identity / Production verification

## 最終輸出

只輸出一份 reviewer report：

```text
REVIEW_RESULT = APPROVED / CHANGES_REQUIRED

TASK_A_ROOT_CAUSE_REVIEW =
D1_REVIEW =
D2_REVIEW =
D3_REVIEW =

VARIABLE_CLASSIFICATION_REVIEW =
SOURCE_OF_TRUTH_REVIEW =
PREFLIGHT_MODEL_REVIEW =
IMMUTABLE_PROMOTION_REVIEW =
ROLLBACK_REVIEW =
SHA_ALIAS_REVIEW =
DATABASE_TARGET_REVIEW =
ONE_BAE_728_OR_SUCCESSOR_REVIEW =
DOC_DELTA_REVIEW =

CONTRADICTIONS_FOUND =
OVERREACH_FOUND =
MISSING_EVIDENCE =
REQUIRED_PLAN_CHANGES =

CURRENT_RELEASES_BLOCKED_BY_TASK_B = NO
TASK_C_EXECUTE_NOW = NO
TASK_C_IS_FUTURE_MIGRATION_WORK = YES

EXECUTION_PLAN_APPROVED = YES/NO
TASK_C_READY_FOR_FUTURE_EXECUTION = YES/NO
```

只有在未來 Task C 不需要重新做 architecture decision、所有修改點／測試／rollback 都足夠具體時，才：

```text
EXECUTION_PLAN_APPROVED = YES
TASK_C_READY_FOR_FUTURE_EXECUTION = YES
```

否則列出必須先補齊的 plan changes。

**本輪結束後不要自動啟動 Task C。**
