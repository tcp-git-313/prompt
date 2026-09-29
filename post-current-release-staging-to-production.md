# Post — Current Release Staging → Production

推薦模型：GPT-5.6 Sol High

## 任務目標

本輪只處理 **Post**。

固定專案：

```text
TARGET_PRODUCT = post
PRODUCT_REPO = F:\00-Ticenpi-SaaS\TicenpiPost
EXPECTED_GITHUB_REPO = tcp-git-313/ticenpi-post
WORKSPACE = F:\00-Ticenpi-SaaS
DEPLOY_REPO = F:\00-Ticenpi-SaaS\deploy
RELEASE_EVIDENCE_ROOT = F:\00-Ticenpi-SaaS\.release-evidence
PRODUCT_EVIDENCE_ROOT = F:\00-Ticenpi-SaaS\.release-evidence\post
```

先驗證 Git remote 唯一對應 `tcp-git-313/ticenpi-post`；若不符，STOP。

**使用者不需要再填任何參數。**

本輪唯一成功流程：

```text
CURRENT COMMITTED PRODUCT SOURCE
→ NEW OR REUSABLE STAGING RELEASE
→ STAGING DEPLOYED
→ STAGING VERIFIED
→ STAGING ACCEPTED
→ IMMUTABLE ARTIFACT / APPROVED PACKAGE LOCKED
→ PRODUCTION PROMOTED
→ PRODUCTION VERIFIED
→ PRODUCTION ACCEPTED
```

本輪不是「補舊 Production evidence」任務，也不是「把歷史 accepted-staging 再部署一次」任務。

## 已知歷史參考（REFERENCE ONLY）

以下資訊只供交叉核對，**不得直接當成本輪 target**：

- 歷史 accepted-staging source 曾為 `27e4efedbd9157293b95b56cd9cecaeeadbf6cac`。
- 該 SHA 只可當 previous accepted / Production reference；如果目前 committed source 更新，必須先建立新 Staging release。

所有 current 值必須執行時重新從 Git、release evidence、runtime 讀取。

---

## 本輪與 Universal Workflow 的關係

目前尚未完成的新 Universal Workflow **不是本輪 release gate**。

```text
UNIVERSAL_WORKFLOW_MIGRATION_DEFERRED = YES
UNIVERSAL_WORKFLOW_MIGRATION_BLOCKS_THIS_RELEASE = NO
LEGACY_IDENTITY_COMPATIBILITY_ALLOWED = YES
```

本輪：
- 不讀取、不套用尚未完成的新 Workflow 當執行正主
- 不因未來 Workflow 尚未完成而 HARD_STOP
- 不執行 Task C
- 不做 Product Onboarding V2 / Product Registry / Central Seat 重構
- 仍保留現行 release 的安全 gate

---

# 1. Current Source Discovery

先在 `F:\00-Ticenpi-SaaS\TicenpiPost` 執行唯讀查核：

```text
REMOTE =
BRANCH =
HEAD_FULL_SHA =
ORIGIN_HEAD =
DIRTY_TRACKED =
DIRTY_UNTRACKED =
HEAD_IN_REMOTE =
CI_STATUS =
```

不要把舊 release branch 名硬當正主。自行以 remote、current docs、release evidence 判斷 canonical release branch；有唯一證據時直接使用。

### Release source 規則

```text
TARGET_SOURCE_COMMIT = 目前 canonical release branch 的 committed HEAD 完整 SHA
```

禁止：
- 直接使用歷史 accepted source 當本輪 target
- 從 dirty working tree 建 release
- 把未 commit WIP 偷帶進 release
- 因舊 Production 還在跑某 SHA，就把該 SHA 當成本輪 target

dirty files：
- 保留原狀
- 不 reset
- 不 stash
- 不刪除
- 不納入 release

若 dirty 內容明顯是本輪要發布但尚未 commit 的功能，輸出：

```text
UNCOMMITTED_RELEASE_CONTENT = YES
RELEASE_BLOCKED = YES
```

停止，因為不能安全推斷哪些 WIP 應發布。

---

# 2. Compare Current Source vs accepted-staging

讀：

```text
F:\00-Ticenpi-SaaS\.release-evidence\post\accepted-staging.json
```

如果檔案不存在：

```text
EXISTING_ACCEPTED_STAGING = NONE
```

這**不是**整個任務的 blocker；代表本輪必須先建立新的 Staging release。

如果存在，讀完整：

```text
ACCEPTED_SOURCE_COMMIT
ACCEPTED_RELEASE_ID
BACKEND_DIGEST
FRONTEND_DIGEST
DEPLOY_CONFIG_COMMIT
runtime_identity_verified
health_verified
auth_verified
commercial_verified
e2e_verified
```

分流：

### Case A — current source 已經是最新 ACCEPTED

若：

```text
TARGET_SOURCE_COMMIT = ACCEPTED_SOURCE_COMMIT
status = ACCEPTED
mandatory verified fields = true
artifact/package identity complete
```

則：

```text
STAGING_REUSE_ALLOWED = YES
```

不要沒必要地 rebuild / redeploy Staging。

### Case B — current source 比 accepted 更新或 accepted 不存在

若：

```text
TARGET_SOURCE_COMMIT != ACCEPTED_SOURCE_COMMIT
```

或 accepted 不存在：

```text
NEW_STAGING_RELEASE_REQUIRED = YES
```

**必須建立新 Staging release。**

舊 accepted-staging 只可當：
- previous release reference
- rollback/reference evidence

不得作為本輪 Production promotion source。

---

# 3. Build / Test / CI Gate

若需要新的 Staging release：

先確認目前產品 canonical build/test/CI path。

最低要求：

```text
SOURCE_COMMIT = TARGET_SOURCE_COMMIT
LOCAL_TESTS = PASS
CI_EXACT_COMMIT = PASS
ARTIFACT_OR_PACKAGE_TRACEABLE_TO_SOURCE = YES
```

Docker：
- 必須產生 immutable digest
- 不得只靠 mutable tag

systemd/package：
- package hash 必須固定
- package source 必須可追溯
- dependency lock / build input 必須可證明

禁止從 Production 重新 build 來補 artifact identity。

---

# 4. Staging Deployment

使用目前 repo / deploy repo 已存在的 canonical Staging path。

優先使用現行 canonical `deploy.ps1 post staging`；若 repo 現有 operator wrapper 只是封裝該路徑，可使用既有 wrapper，不另造。

不得自行發明第二套 release script。

Staging target 必須確認：
- environment = staging
- Staging Supabase / DB
- domain / port
- secret presence
- auth config
- commercial / Seat config（適用時）
- storage / external integration（適用時）
- Production 不受 mutation

---

# 5. Legacy TICENPI_* Deployment Identity Compatibility

相關欄位：

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

本輪允許針對 legacy identity contract 做**最小、release-scoped、deploy-time compatibility**。

禁止：
- 為了過 gate 永久人工把五鍵寫死 shared/.env
- 關掉整個 preflight
- 跳過 environment / commit / Supabase / DB / artifact 驗證
- 建立另一份 permanent source of truth

正確方向：

```text
static runtime config/secrets
+
release/deploy-time identity metadata
→ effective runtime/deployment identity
```

compatibility 只解除 legacy identity blocker，不得解除其他安全 gate。

### 若 compatibility 需要更新共用 VPS deploy tooling

不要因「這是共用 VPS 寫入」這句話本身就要求使用者手動跑 PowerShell。

只有在以下全部成立時，可在本任務內繼續既有 release chain：

```text
CHANGE_SCOPE = Deployment Identity compatibility only
SOURCE_DIFF = reviewed
TESTS = PASS
REMOTE_CURRENT_VERSION = compared
BACKUP_BEFORE_REPLACE = ready
ROLLBACK = ready
NO_DB_CHANGE = YES
NO_SECRET_CHANGE = YES
NO_DNS_CHANGE = YES
NO_PORT_CHANGE = YES
NO_RUNTIME_ARCHITECTURE_CHANGE = YES
```

執行前：
1. 比對 VPS 現有腳本 hash / diff。
2. 備份原檔。
3. 只安裝必要檔案。
4. 安裝後重新比對 hash。
5. 先跑 read-only preflight。
6. 有任何非預期差異立即 rollback tooling。

若執行環境／安全分類器本身禁止你寫 VPS：

```text
TOOL_PERMISSION_BLOCKED = YES
USER_APPROVAL_MISSING = NO
```

不要把工具限制誤報成「使用者還沒授權」；輸出唯一必要的 exact manual action。

---

# 6. Staging Verification / ACCEPTED

新 Staging release 至少驗：

- runtime identity / release traceability
- health / ready
- no-token deny
- invalid-token deny
- real auth
- commercial entitlement
- Seat allow/deny（適用時）
- data isolation（適用時）
- Post 真實 auth、商用/Seat、資料隔離及核心發文/物件流程的最小 E2E

只有 mandatory verification 全 PASS：

```text
STAGING_DEPLOYED = YES
STAGING_VERIFIED = YES
STAGING_ACCEPTED = YES
```

### accepted-staging evidence 更新規則

不得在 Staging 尚未 ACCEPTED 前覆寫 canonical `accepted-staging.json`。

新 release ACCEPTED 後：

1. 先保存 release-specific / history evidence。
2. 再依現有 evidence convention 更新 canonical `accepted-staging.json`。
3. `source_commit` 必須等於 `TARGET_SOURCE_COMMIT`。
4. digest/package identity 必須是本輪實際產物。
5. 不得沿用舊 release artifact identity。
6. 不得改寫舊 history evidence。

---

# 7. Production Eligibility

Production 的唯一 promotion source：

```text
NEWEST VALID STAGING ACCEPTED EVIDENCE
```

要求：

```text
PRODUCTION_SOURCE_COMMIT
=
STAGING_ACCEPTED_SOURCE_COMMIT
```

Post Docker→Docker 時 Production 必須使用最新 Staging ACCEPTED 的同一 immutable digest。

Production 禁止：
- 直接從 current branch build
- Production rebuild 已 ACCEPTED 的 Docker artifact
- 使用 mutable tag
- 跳過新的 Staging ACCEPTED
- 用 previous Production baseline 當本輪 source
- 把 config commit 當 product source commit

---

# 8. Production Config Reconciliation

先確認：

```text
PRODUCTION_ENVIRONMENT = production
PRODUCTION_SUPABASE_TARGET =
PRODUCTION_DATABASE_TARGET =
PRODUCTION_DOMAIN =
PRODUCTION_PORT =
PRODUCTION_RUNTIME_MODE =
PRODUCTION_SEAT_POLICY =
PRODUCTION_DELIVERY_MODEL =
ROLLBACK_TARGET =
```

若 Production pin 落後，只做 accepted digest/source 的 config-only reconciliation，不把舊 Production source 當本輪 target。

若只是正常 promotion 所需 config-only pin：

- accepted digest/package → Production config
- accepted source identity → Production identity pin

可自動建立 config-only commit，但：
- 不改產品功能
- 不混 unrelated refactor
- 必須 CI PASS
- 必須保留 rollback

---

# 9. Production Preflight

必須驗：

- exact accepted source
- exact accepted artifact/package
- anti-rollback / no stale release
- environment
- Supabase
- DB
- secret/config presence
- domain/port
- auth
- commercial / Seat
- rollback
- health prerequisites

如果除了 legacy TICENPI identity compatibility 之外還有 blocker：

```text
OTHER_BLOCKERS != NONE
READY_FOR_PRODUCTION_PROMOTION = NO
```

STOP，不得一起 bypass。

---

# 10. Production Mutation Authorization

使用者已授權：

若：

```text
PRODUCTION_PREFLIGHT = PASS
ACCEPTED_SOURCE_CONFIRMED = YES
ACCEPTED_ARTIFACT_OR_PACKAGE_CONFIRMED = YES
TARGET_CONFIRMED = YES
ROLLBACK_READY = YES
OTHER_BLOCKERS = NONE
NO_NEW_DESTRUCTIVE_OPERATION = YES
```

則：

**直接執行 Production promotion，不要再停下詢問一次。**

只有遇到以下才 STOP：
- DB migration
- destructive schema change
- secret 新增／修改／輪替
- DNS 變更
- port 變更
- runtime architecture 變更
- 非預期 source 修改
- 非預期 config 修改
- 無法證明 rollback
- 無法證明 artifact/package equivalence

先輸出 exact command / expected identity，然後在同一任務繼續執行。

使用現行 canonical `promote.ps1 post production` path。

---

# 11. Production Verification

部署後不得只停在 DEPLOYED。

至少確認：

```text
RUNNING_ENVIRONMENT = production
RUNNING_SOURCE_COMMIT = STAGING_ACCEPTED_SOURCE_COMMIT
RUNNING_ARTIFACT_OR_PACKAGE = STAGING_ACCEPTED_ARTIFACT_OR_PACKAGE
RUNNING_SUPABASE_TARGET = expected Production
RUNNING_DATABASE_TARGET = expected Production
RUNNING_RELEASE_ID = traceable
```

再驗：
- health / ready
- auth deny/allow
- commercial / Seat
- Post 真實登入、授權、Seat 與最小核心操作 canary

```text
DEPLOYED != ACCEPTED
```

若最後只差真人 OAuth / UI 操作：

```text
PRODUCTION_DEPLOYED = YES
PRODUCTION_ACCEPTED = NO
HUMAN_CANARY_REQUIRED = YES
```

不要宣告 GO_LIVE。

全部 mandatory Production verification PASS 後：

```text
PRODUCTION_ACCEPTED = YES
GO_LIVE = YES
```

---

# 12. 本輪禁止事項

1. 把歷史 accepted source 當成本輪 target。
2. 重部署舊版本來假裝最新版已上線。
3. 只補舊 Production ACCEPTED evidence 就結束。
4. current source 直接跳過 Staging 上 Production。
5. 跳過新的 Staging ACCEPTED。
6. 因未來 Universal Workflow 尚未完成而 HARD_STOP。
7. 關閉整個 preflight。
8. 永久人工寫死 TICENPI_* 五鍵。
9. 為解 identity blocker 順手做架構重構。
10. 修改其他產品。

---

# 13. Final Evidence

依現有 release evidence convention 記錄，不得覆寫舊 history。

最後輸出：

```text
TARGET_PRODUCT = post
PRODUCT_REPO = F:\00-Ticenpi-SaaS\TicenpiPost
TARGET_SOURCE_COMMIT =
CURRENT_BRANCH =
DIRTY_STATE =
EXISTING_ACCEPTED_STAGING_SOURCE =
NEW_STAGING_RELEASE_REQUIRED =
STAGING_RELEASE_ID =
STAGING_DEPLOYED =
STAGING_VERIFIED =
STAGING_ACCEPTED =
STAGING_ACCEPTED_SOURCE_COMMIT =
STAGING_ARTIFACT_OR_PACKAGE =
LEGACY_IDENTITY_COMPATIBILITY_USED =
VPS_TOOLING_CHANGED =
TOOL_PERMISSION_BLOCKED =
PRODUCTION_DELIVERY_MODEL =
PRODUCTION_PREFLIGHT =
OTHER_BLOCKERS =
READY_FOR_PRODUCTION_PROMOTION =
EXACT_PRODUCTION_COMMAND =
PRODUCTION_RELEASE_ID =
PRODUCTION_DEPLOYED =
RUNTIME_IDENTITY_VERIFIED =
HEALTH_VERIFIED =
AUTH_VERIFIED =
COMMERCIAL_VERIFIED =
HUMAN_CANARY_REQUIRED =
PRODUCTION_ACCEPTED =
GO_LIVE =
UNIVERSAL_WORKFLOW_MIGRATION_DEFERRED = YES
UNIVERSAL_WORKFLOW_MIGRATION_BLOCKS_THIS_RELEASE = NO
LEGACY_IDENTITY_COMPATIBILITY_ALLOWED = YES
EVIDENCE_PATH =
ROLLBACK_TARGET =
EXACT_NEXT_ACTION =
```
