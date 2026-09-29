# Legacy Production Baseline — OCR

推薦模型：GPT-5.6 Sol High

## 任務

本輪只處理 **OCR**。

這不是新版 Workflow 導入任務；這一輪的唯一目標是把目前已完成 Staging 驗證、符合現行 release contract 的版本建立成正式 Production baseline。

**本輪禁止讀取、套用或依賴 `new-workflow.md`。**
新的 Universal Workflow、Deployment Identity 正式重構、Product Onboarding V2、Product Registry、跨產品架構整理全部延後到下一個本機新版本。

---

## 0. 固定目標

```text
TARGET_PRODUCT = ocr
PRODUCT_REPO = F:\00-Ticenpi-SaaS\TicenpiLetter
EXPECTED_GITHUB_REPO = tcp-git-313/ticenpi-orc
WORKSPACE = F:\00-Ticenpi-SaaS
DEPLOY_REPO = F:\00-Ticenpi-SaaS\deploy
SYSTEM_DOCS = F:\00-Ticenpi-SaaS\deploy\docs\system
RELEASE_EVIDENCE_ROOT = F:\00-Ticenpi-SaaS\.release-evidence
PRODUCT_EVIDENCE = F:\00-Ticenpi-SaaS\.release-evidence\ocr
```

先驗證 PRODUCT_REPO 存在且 Git remote 對應 OCR/Letter 正主 repo。Canonical product code 固定為 `ocr`。若 deploy/service 中仍有歷史 immutable service id `orc`，本輪只視為 legacy service identifier，不趁機改名。

已知 evidence hint（執行時仍以 JSON 原值為準）：

```text
KNOWN_ACCEPTED_STAGING_RELEASE_ID = 20260925-021446
KNOWN_ACCEPTED_SOURCE_COMMIT = 723bef7391ce99c2542d591e6b04335e8321add4
KNOWN_ACCEPTED_DEPLOY_CONFIG_COMMIT = 2ab8520b07eb498d6a78ca73b5ddc6a7cfcf93c3
KNOWN_ACCEPTED_STATUS = ACCEPTED
KNOWN_RUNTIME_IDENTITY_VERIFIED = true
KNOWN_HEALTH_VERIFIED = true
KNOWN_AUTH_VERIFIED = true
KNOWN_COMMERCIAL_VERIFIED = true
KNOWN_E2E_VERIFIED = true
```

backend/frontend digest 必須直接讀 `accepted-staging.json` 完整值。

---

## 1. Scope

只允許處理 OCR。

不要順手處理其他產品，不要 merge Universal Workflow workstream，不要重構 Central Seat / Commercial Core，不要做 unrelated cleanup。

本輪成功定義：

```text
CURRENT VERIFIED RELEASE
→ PRODUCTION DEPLOYED
→ PRODUCTION VERIFIED
→ PRODUCTION ACCEPTED
```

並記：

```text
RELEASE_MODE = LEGACY_PRODUCTION_BASELINE
WORKFLOW_MIGRATION_DEFERRED = YES
NEXT_LOCAL_RELEASE_REQUIRES_UNIVERSAL_WORKFLOW_MIGRATION = YES
```

---

## 2. Current State Reconciliation

先只讀確認：

```text
TARGET_PRODUCT
PRODUCT_REPO
CURRENT_BRANCH
CURRENT_HEAD
DIRTY_STATE
DELIVERY_MODEL
STAGING_RELEASE_ID
STAGING_STATUS
STAGING_SOURCE_COMMIT
STAGING_BACKEND_DIGEST
STAGING_FRONTEND_DIGEST
STAGING_DEPLOY_CONFIG_COMMIT
STAGING_RUNTIME_IDENTITY_VERIFIED
STAGING_HEALTH_VERIFIED
STAGING_AUTH_VERIFIED
STAGING_COMMERCIAL_VERIFIED
STAGING_E2E_VERIFIED
CURRENT_PRODUCTION_RELEASE
CURRENT_PRODUCTION_SOURCE
CURRENT_PRODUCTION_ARTIFACT
ROLLBACK_TARGET
```

資料來源優先順序：

1. `F:\00-Ticenpi-SaaS\.release-evidence\ocr\accepted-staging.json`
2. 該產品 release evidence history
3. product repo source/config
4. deploy repo / services.yaml
5. Production runtime / health / ready
6. handoff 只可當 secondary evidence

如果來源衝突，輸出 `SOURCE_CONFLICT = YES` 並 STOP，不自行選一份。

---

## 3. Eligibility Gate

OCR 必須以 `.release-evidence\ocr\accepted-staging.json` 為 promotion 正主。不要用 `orc` 當 commercial product_code。

Docker → Docker 時必須：

```text
PRODUCTION_ARTIFACT = STAGING_ACCEPTED_ARTIFACT
```

Production 不得 rebuild，不得從 dirty tree 重建，不得用 mutable tag 取代 accepted digest。

OCR 預期是 Docker → Docker；若實查不是，STOP 並回報 delivery drift。

---

## 4. Production Config Reconciliation

確認：

```text
PRODUCTION_ENVIRONMENT = production
PRODUCTION_SUPABASE_TARGET =
PRODUCTION_DATABASE_TARGET =
PRODUCTION_DOMAIN =
PRODUCTION_PORT =
PRODUCTION_RUNTIME_MODE =
PRODUCTION_SEAT_POLICY =
PRODUCTION_COMPOSE_OR_UNIT =
```

若 Production config 還 pin 舊 digest，而 Staging accepted evidence 已鎖定新 digest：

只允許建立 **config-only Production deploy commit** 將 Production pin 更新成 accepted digest / accepted source identity。

這個 config-only commit：

- 不得修改產品功能
- 不得 rebuild image
- 不得夾帶 refactor
- 必須通過既有 CI / preflight
- 值必須直接取自 accepted evidence，不手抄截斷 digest

任何 legacy `orc` service key 只做部署相容，不在本輪改 commercial product_code 或資料契約。

---

## 5. Legacy Deployment Identity Compatibility

已知五鍵：

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

本輪不要正式重構 Deployment Identity。

禁止為了讓 preflight 變綠而把五鍵永久人工寫進 shared/.env。

若五鍵 presence rule 是唯一 blocker，先證明：

```text
environment      ← Production deploy target
source commit    ← accepted Staging evidence
artifact         ← accepted Staging evidence
Supabase project ← canonical Production config / SUPABASE_URL
database target  ← canonical Production DB policy/config
release id       ← existing release/promote tooling
```

若其中任何一項不能唯一判定，STOP。

如果現有 tooling 已有安全 release-scoped / deploy-time compatibility path，可使用。
如果沒有，不得關掉整個 preflight；只允許最小 compatibility change，且不能變成永久第二份 source of truth。

---

## 6. Production Preflight

執行目前 repo **既有** canonical Production preflight。

必須保留：

- source identity
- accepted artifact / package identity
- Production target
- Supabase target
- DB target
- static secrets/config
- rollback
- domain / port
- auth prerequisites
- commercial / Seat prerequisites
- health prerequisites

只要存在 identity legacy rule 以外的 blocker，就 STOP，不得一起 bypass。

輸出：

```text
PRODUCTION_PREFLIGHT =
LEGACY_IDENTITY_RULE_BLOCKER =
OTHER_BLOCKERS =
READY_FOR_PRODUCTION_PROMOTION =
```

---

## 7. 自動 Production Promotion 授權

使用者已明確授權本任務：

如果：

```text
preflight 全部 PASS
artifact / package identity 已確認
Production target 已確認
rollback 已確認
沒有新的破壞性操作
OTHER_BLOCKERS = NONE
```

則：

**直接執行 Production promotion，不要再次詢問使用者。**

先記錄：

```text
READY_FOR_PRODUCTION_PROMOTION = YES
EXACT_COMMAND =
EXPECTED_SOURCE_COMMIT =
EXPECTED_ARTIFACT =
EXPECTED_PRODUCTION_TARGET =
ROLLBACK_TARGET =
```

然後直接執行。

只有遇到以下任一項才 STOP 等人工確認：

- DB migration
- destructive schema change
- secret 新增／修改／輪替
- DNS 變更
- port 變更
- runtime architecture 變更
- 非預期 source 修改
- 非預期 config 修改
- 不屬於 accepted Staging release / 正常 Production promotion 的 mutation

正常的 config-only pin reconciliation（accepted digest/source identity → Production config）不算新的破壞性操作，但仍須通過既有測試、CI、preflight。

---

## 8. Production Execution

OCR Docker → Docker：優先使用 canonical `promote.ps1 ocr production`；若現行 deploy tooling 的 immutable service key 仍要求 `orc`，只能在 source 證據證明後使用該 legacy service key，不得把 product_code 改回 orc。

只使用現有 canonical release/deploy path，不自行發明第二套 Production deploy script。

---

## 9. Production Verification

部署後不要停在 DEPLOYED。

至少驗：

```text
RUNNING_ENVIRONMENT = production
RUNNING_SOURCE_COMMIT = expected
RUNNING_ARTIFACT_OR_PACKAGE = expected
RUNNING_SUPABASE_TARGET = expected Production
RUNNING_DATABASE_TARGET = expected Production
RUNNING_RELEASE_ID = traceable
```

並執行：

- health / ready
- no token deny
- invalid token deny
- Production real auth path
- entitlement / Seat allow-deny（適用時）
- 該產品核心 canary

不得用 Production test bypass 代替真人行為。

若最後只差真人登入：

```text
PRODUCTION_DEPLOYED = YES
PRODUCTION_ACCEPTED = NO
HUMAN_CANARY_REQUIRED = YES
```

不要假裝 GO_LIVE。

全部 mandatory verification PASS 後才：

```text
PRODUCTION_ACCEPTED = YES
GO_LIVE = YES
```

---

## 10. Evidence

不得覆寫 accepted-staging evidence。

依現行 release evidence convention 寫入 Production evidence，至少記：

```text
TARGET_PRODUCT = ocr
RELEASE_MODE = LEGACY_PRODUCTION_BASELINE
WORKFLOW_MIGRATION_DEFERRED = YES
PROMOTED_FROM_STAGING_RELEASE_ID =
SOURCE_COMMIT =
BACKEND_DIGEST =
FRONTEND_DIGEST =
PRODUCTION_RELEASE_ID =
RUNTIME_IDENTITY_VERIFIED =
HEALTH_VERIFIED =
AUTH_VERIFIED =
COMMERCIAL_VERIFIED =
E2E_VERIFIED =
PRODUCTION_ACCEPTED =
ROLLBACK_RELEASE_ID =
LEGACY_COMPATIBILITY_USED =
LEGACY_COMPATIBILITY_SCOPE =
NEXT_LOCAL_RELEASE_REQUIRES_UNIVERSAL_WORKFLOW_MIGRATION = YES
```

---

## 11. 最終輸出

```text
TARGET_PRODUCT = ocr
PRODUCT_REPO =
DELIVERY_MODEL =
STAGING_ACCEPTED =
STAGING_RELEASE_ID =
SOURCE_COMMIT =
ACCEPTED_ARTIFACT_OR_PACKAGE =
PRODUCTION_CONFIG_STATUS =
PRODUCTION_PREFLIGHT =
LEGACY_IDENTITY_RULE_BLOCKER =
LEGACY_COMPATIBILITY_USED =
OTHER_BLOCKERS =
READY_FOR_PRODUCTION_PROMOTION =
EXACT_COMMAND =
PRODUCTION_DEPLOYED =
RUNTIME_IDENTITY_VERIFIED =
HEALTH_VERIFIED =
AUTH_VERIFIED =
COMMERCIAL_VERIFIED =
HUMAN_CANARY_REQUIRED =
PRODUCTION_ACCEPTED =
GO_LIVE =
RELEASE_MODE = LEGACY_PRODUCTION_BASELINE
WORKFLOW_MIGRATION_DEFERRED = YES
NEXT_LOCAL_RELEASE_REQUIRES_UNIVERSAL_WORKFLOW_MIGRATION = YES
EVIDENCE_PATH =
ROLLBACK_TARGET =
EXACT_NEXT_ACTION =
```
