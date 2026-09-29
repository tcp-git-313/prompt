# Task A Resume — Deployment Identity Remaining-Gaps Audit After D1

推薦模型：Kimi K3 Max

## 任務定位

D1 已完成，不要再掃整個 workspace，不要重做 reference inventory。

本輪直接把 D1 最終結果視為 ACCEPTED EVIDENCE，繼續完成原本缺失的 Task A：

- 真實 call chain
- requiredEnv root cause
- DM Production failure exact cause
- accepted-staging / promote already-known identity
- 五鍵分類
- SHA alias
- DATABASE_TARGET
- exact code/doc fix plan

本輪唯讀：
不修改、不 commit、不 push、不 deploy、不補 shared/.env。

---

## 0. 路徑

Workspace:

F:\00-Ticenpi-SaaS

Deploy repo:

F:\00-Ticenpi-SaaS\deploy

System docs:

F:\00-Ticenpi-SaaS\deploy\docs\system\

新版 Workflow:

F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md

Release evidence:

F:\00-Ticenpi-SaaS\.release-evidence\

已知 Deployment Identity worktree/commit 線索：

fix/dm-deployment-identity-20260929 @ 1bae728

---

## 1. D1 已確認事實 — 不重查

D1 已完成全 workspace static reference inventory。

以下視為 accepted evidence，除非你找到直接 source contradiction，否則不要重新掃描：

1. canonical F:\00-Ticenpi-SaaS\deploy\services.yaml 目前沒有任何 TICENPI_* 放在 requiredEnv。

2. 只有 4 份 manifest 把五個 identity keys 放進 requiredEnv：
   - ticenpi-platform\services.yaml
   - .worktrees\admin-workbench-platform\services.yaml
   - .worktrees\dm-deployment-identity-20260929\services.yaml
   - .worktrees\w2-platform-c1-commercial-admin\services.yaml

3. TICENPI_GIT_SHA 從未被放進 requiredEnv。

4. current deploy\server\preflight.sh 沒有 executable hardcoded shared/.env；它從 manifest envFile 取得 env file，再檢查 requiredEnv。

5. 舊 backup preflight 曾 hardcode：
   $RELEASE_ROOT/shared/.env
   但 current preflight 不再 hardcode。

6. deploy-time identity injection 已存在於：
   deploy\server\deploy.sh start_runtime
   deploy\server\rollback.sh start_runtime

   已確認會注入：
   TICENPI_RELEASE_ID
   TICENPI_COMMIT_SHA
   TICENPI_GIT_SHA

7. current canonical deploy repo 本身沒有 runtime READ/DERIVE consumer；真正 runtime consumers 在各 product repo / worktree。

8. workspace 中 WRITE=0：沒有程式永久寫入這五鍵到 shared/.env。

9. 目前最大未解矛盾：
   canonical deploy\services.yaml 不要求五鍵，
   但 ticenpi-platform / Deployment Identity worktree manifest 要求五鍵。

10. 72% references 是 clone/worktree copies，不要再把 clone 數量當成獨立實作。

---

## 2. 目前最重要的假說

D1 已顯示一個很可能的 failure mechanism：

services manifest 把五鍵列入 requiredEnv
→ preflight 在 runtime start 之前先檢查 envFile
→ 五鍵此時尚未由 deploy.sh start_runtime 注入
→ preflight FAIL
→ deploy-time injection 永遠沒有機會發生

你必須用實際 source / command path 驗證這個假說。

不要直接把它當結論。

---

## 3. Task A1 — 找出 DM Production failure 實際用了哪份 manifest

這是本輪最高優先。

追：

promote.ps1
→ deploy.ps1
→ manifest selection / copy / SCP
→ server/preflight.sh
→ actual service block
→ actual envFile
→ requiredEnv

回答：

DM_PRODUCTION_COMMAND =
LOCAL_MANIFEST_PATH =
LOCAL_MANIFEST_COMMIT =
REMOTE_MANIFEST_PATH =
REMOTE_SERVICE_KEY =
REMOTE_ENVFILE =
REMOTE_REQUIREDENV =
FIVE_KEYS_PRESENT_IN_ACTUAL_REQUIREDENV = YES/NO

特別確認：

DM 那次 Production preflight 到底使用：

A.
F:\00-Ticenpi-SaaS\deploy\services.yaml

B.
F:\00-Ticenpi-SaaS\ticenpi-platform\services.yaml

C.
.worktrees\dm-deployment-identity-20260929\services.yaml

D.
VPS 上較舊 / 較新的 manifest snapshot

E.
其他

不要從資料夾名稱猜，要沿 command / SCP / release snapshot 證明。

---

## 4. Task A2 — 驗證 preflight-before-injection 時序

實查：

server\preflight.sh
server\deploy.sh
server\rollback.sh

回答 exact order：

1. candidate manifest load
2. envFile resolution
3. requiredEnv validation
4. Deployment Identity expected-value derivation（若存在）
5. runtime env injection
6. docker compose/systemd start
7. audit

輸出：

PREFLIGHT_EXECUTION_ORDER =

IDENTITY_INJECTION_HAPPENS_BEFORE_OR_AFTER_REQUIREDENV_CHECK =

CAN_DEPLOY_TIME_METADATA_SATISFY_PREFLIGHT_TODAY = YES/NO

WHY =

如果 requiredEnv 在 injection 之前檢查，而五鍵只在 start_runtime 才注入，要明確指出這就是 contract wiring mismatch。

---

## 5. Task A3 — 找五鍵 requiredEnv 規則的 introducing commit

只針對真正含五鍵的 authoritative candidate manifests / validator 做 git history。

使用：

git log
git blame
git show

回答：

INTRODUCING_COMMIT =
INTRODUCING_BRANCH =
INTRODUCING_FILE =
INTRODUCING_DIFF =
INTRODUCING_INTENT =

並區分：

A. manifest data 加入五鍵
B. validator 強制要求五鍵
C. deployment_identity.py / effective-env fix

不要把三者混成一個 commit。

---

## 6. Task A4 — accepted-staging / promote 已經知道什麼

實查：

F:\00-Ticenpi-SaaS\.release-evidence\

release.ps1
promote.ps1
tools\release_evidence_tool.py

優先 DM。

確認實際欄位名稱：

SOURCE_COMMIT_FIELD =
ARTIFACT_FIELD =
DEPLOY_CONFIG_COMMIT_FIELD =
STAGING_RELEASE_ID_FIELD =
RUNTIME_IDENTITY_VERIFIED_FIELD =

回答：

ACCEPTED_STAGING_ALREADY_KNOWS =
PROMOTE_ALREADY_KNOWS =

並明確列：

- accepted source commit?
- backend/frontend digest?
- deploy config commit?
- staging release id?
- production target?
- production artifact reuse?
- rebuild blocked?

---

## 7. Task A5 — 五鍵逐一分類

對：

TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET

每一個只能選：

RUNTIME_REQUIRED
DEPLOY_TIME_INJECTED
DERIVED
REDUNDANT
UNRESOLVED

輸出：

VARIABLE =
CLASS =
CANONICAL_SOURCE =
DERIVED_FROM =
RUNTIME_CONSUMERS =
VALIDATION_CONSUMERS =
MUST_EXIST_IN_SHARED_ENV = YES/NO
MUST_EXIST_IN_EFFECTIVE_RUNTIME_ENV = YES/NO
WHY =

注意：

「runtime 需要讀到」
不等於
「shared/.env 必須人工永久保存」。

---

## 8. Task A6 — TICENPI_GIT_SHA vs TICENPI_COMMIT_SHA

D1 已確認 deploy.sh / rollback.sh 兩個都會注入。

現在只查真正 runtime consumer 和 compatibility。

回答：

GIT_SHA_CONSUMERS =
COMMIT_SHA_CONSUMERS =
CANONICAL_FIELD =
ALIAS_COMPATIBILITY_REQUIRED = YES/NO
CURRENT_MISMATCH_RISK =
MIGRATION_PLAN =

特別檢查 health / ready / runtime identity 是否可能讀不同欄位。

---

## 9. Task A7 — TICENPI_SUPABASE_PROJECT_REF

確認 Production 已有的 canonical config 是否能從：

SUPABASE_URL
service target
deployment config

唯一推導 project ref。

輸出：

SUPABASE_REF_CLASS =
CANONICAL_SOURCE =
DERIVED_FROM =
SHARED_ENV_DUPLICATION_REQUIRED = YES/NO

---

## 10. Task A8 — TICENPI_DATABASE_TARGET

只回答實際語意。

如果 product DB 就是該 Supabase project 的 Postgres：

判定它是否只是 DERIVED / REDUNDANT。

如果它還表示獨立 database name / host / logical target：

指出 canonical source。

輸出：

DATABASE_TARGET_CLASS =
CANONICAL_SOURCE =
DERIVED_FROM =
INDEPENDENT_SEMANTIC = YES/NO
KEEP_OR_REMOVE_REASON =

---

## 11. Task A9 — 正確 preflight 模型

根據 actual source 判斷，是否應收斂為：

STATIC_SHARED_ENV
+
DEPLOY_TIME_IDENTITY_METADATA
=
EFFECTIVE_DEPLOYMENT_ENV

再建立：

EXPECTED_DEPLOYMENT_IDENTITY
=
accepted release evidence
+ deploy target
+ canonical environment config
+ release tooling identity

最後：

EXPECTED_DEPLOYMENT_IDENTITY
vs
EFFECTIVE_DEPLOYMENT_IDENTITY

輸出：

CURRENT_MODEL =
TARGET_MODEL =
MINIMAL_CODE_CHANGE =

不要全面重寫 deployment tooling。

---

## 12. Task A10 — 文件 delta

只檢查：

F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_TO_STAGING.md
F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TO_PRODUCTION.md
F:\00-Ticenpi-SaaS\deploy\docs\system\SYSTEM_OWNERSHIP.md
F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md
F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md

STAGING_TEST_IDENTITY_CONTRACT.md 原則上 NO_CHANGE。

每份只列：

DOCUMENT =
SECTION =
CHANGE_REQUIRED =
CURRENT_WRONG_RULE =
CORRECT_RULE =

---

## 13. 禁止事項

不要：

- 再掃 31,771 files
- 重做 D1
- 重做 Product/Service Phase 1
- 重查 Central Seat 架構
- 修改任何檔案
- 補 shared/.env
- deploy
- commit
- push
- 因為看到五鍵就假定五鍵全部要刪
- 因為 runtime consumer 存在就假定 shared/.env 必須保存

---

## 14. 最終只輸出

CURRENT_CALL_CHAIN =

DM_FAILURE_EXACT_CAUSE =

ACTUAL_MANIFEST_USED =

PREFLIGHT_EXECUTION_ORDER =

INTRODUCING_COMMITS =

VARIABLE_MATRIX =

WRONG_REQUIREDENV_RULE =

PREFLIGHT_CURRENT_MODEL =

ACCEPTED_STAGING_ALREADY_KNOWS =

PROMOTE_ALREADY_KNOWS =

SHA_ALIAS_RESULT =

SUPABASE_REF_RESULT =

DATABASE_TARGET_RESULT =

CODE_FILES_TO_FIX =

DOC_SECTIONS_TO_FIX =

MINIMAL_FIX_PLAN =

TEST_PLAN =

D1_CONTRADICTION = NONE / details

BASELINE_CONTRADICTION = NONE / details

PLAN_READY_FOR_REVIEW = YES/NO

如果仍有任何 material implementation detail 需要 Executor 猜：

PLAN_READY_FOR_REVIEW = NO
