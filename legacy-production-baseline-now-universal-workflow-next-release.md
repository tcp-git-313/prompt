# Legacy Production Baseline — Per-Product Production Promotion

## 使用方式

**一個產品、一個 IDE 任務、一個 release evidence chain。**

不要把 Post / DM / OCR / Sign / 591 全部塞進同一個執行任務。

建議順序：

```text
DM
→ Post
→ OCR
→ Sign
→ 591
```

其中：

- DM / Post / OCR：優先處理，若目前是 Docker → Docker 且 Staging 已 ACCEPTED，可走 same-artifact promotion。
- Sign / 591：若仍存在 Staging Docker → Production systemd、不可重現 package、Production runtime 不等價等 blocker，必須獨立停下，不得因為前面產品成功就一起硬上。

每開一個 IDE 任務，只處理一個 `TARGET_PRODUCT`。

---

# 0. 本輪輸入

開始前先把下面兩個值改成該產品：

```text
TARGET_PRODUCT = <dm | post | ocr | sign | 591>
PRODUCT_REPO = <實際 repo path>
```

目前常用路徑：

```text
dm   = F:\00-Ticenpi-SaaS\TicenpiDM
post = F:\00-Ticenpi-SaaS\TicenpiPost
ocr  = F:\00-Ticenpi-SaaS\TicenpiLetter
```

Sign / 591 必須由 IDE 先自行確認實際 repo path，不要猜。

共用：

```text
WORKSPACE = F:\00-Ticenpi-SaaS
DEPLOY_REPO = F:\00-Ticenpi-SaaS\deploy
SYSTEM_DOCS = F:\00-Ticenpi-SaaS\deploy\docs\system
RELEASE_EVIDENCE = F:\00-Ticenpi-SaaS\.release-evidence
```

---

# 1. 任務目標

本輪只做：

```text
CURRENT STAGING ACCEPTED RELEASE
→ PRODUCTION BASELINE
```

本輪不要同步導入：

- Universal Workflow migration
- Deployment Identity V2 正式重構
- Product Onboarding V2
- Product Registry
- Product / Service architecture redesign
- Central Seat redesign
- Commercial Core redesign
- unrelated refactor
- unrelated cleanup

這些全部延後到下一個本機新版本。

本輪成功後：

```text
RELEASE_MODE = LEGACY_PRODUCTION_BASELINE
WORKFLOW_MIGRATION_DEFERRED = YES
NEXT_LOCAL_RELEASE_REQUIRES_UNIVERSAL_WORKFLOW_MIGRATION = YES
```

---

# 2. 先做 Current Product Reconciliation

只針對 `TARGET_PRODUCT`。

確認：

```text
TARGET_PRODUCT =
PRODUCT_REPO =
CURRENT_BRANCH =
CURRENT_HEAD =
DIRTY_STATE =
DELIVERY_MODEL =
STAGING_RELEASE_ID =
STAGING_STATUS =
STAGING_SOURCE_COMMIT =
STAGING_BACKEND_DIGEST =
STAGING_FRONTEND_DIGEST =
STAGING_DEPLOY_CONFIG_COMMIT =
STAGING_RUNTIME_IDENTITY_VERIFIED =
STAGING_HEALTH_VERIFIED =
STAGING_AUTH_VERIFIED =
STAGING_COMMERCIAL_VERIFIED =
STAGING_E2E_VERIFIED =
CURRENT_PRODUCTION_RELEASE =
CURRENT_PRODUCTION_SOURCE =
CURRENT_PRODUCTION_ARTIFACT =
ROLLBACK_TARGET =
```

資料來源優先順序：

1. `.release-evidence\<product>\accepted-staging.json`
2. `.release-evidence\<product>\history\*.json`
3. product repo current source/config
4. deploy repo / services.yaml
5. Production runtime / health / ready
6. existing handoff only as secondary evidence

若不同來源衝突：

```text
SOURCE_CONFLICT = YES
```

停止，不要自行挑一份當真。

---

# 3. Staging Eligibility Gate

只有以下全部成立，才可以繼續：

```text
STAGING_ACCEPTED = YES
STAGING_RUNTIME_IDENTITY_VERIFIED = YES
STAGING_HEALTH_VERIFIED = YES
STAGING_AUTH_VERIFIED = YES
STAGING_COMMERCIAL_VERIFIED = YES   # 適用時
STAGING_E2E_VERIFIED = YES
SOURCE_IDENTITY_KNOWN = YES
ARTIFACT_IDENTITY_KNOWN = YES
ROLLBACK_TARGET_KNOWN = YES
```

如果沒有 accepted-staging evidence：

```text
PROMOTION_BLOCKED = NO_ACCEPTED_STAGING
```

停止。

---

# 4. Delivery Model Gate

先判定：

```text
DELIVERY_MODEL =
DOCKER_TO_DOCKER
DOCKER_TO_SYSTEMD
SYSTEMD
STATIC
OTHER
```

## 4.1 Docker → Docker

要求：

```text
PRODUCTION_ARTIFACT
=
STAGING_ACCEPTED_ARTIFACT
```

Production 不得 rebuild。

不得從本機 dirty tree 建新 image。

不得用 mutable tag 代替 accepted digest。

## 4.2 Docker → systemd / systemd

必須證明：

- source commit 相同
- package 可重現
- package hash 可驗
- dependency lock 等價
- Production package 已被 release evidence 綁定

如果不能證明：

```text
PROMOTION_BLOCKED = ARTIFACT_EQUIVALENCE_NOT_PROVEN
```

停止。

不要把這類 blocker 當成 Deployment Identity 五鍵問題。

---

# 5. Production Config Reconciliation

只針對目前產品。

確認 Production config：

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

若是 Docker same-artifact promotion，Production compose 必須 pin accepted digest。

如果 Production compose 還 pin 舊 digest：

不要 rebuild image。

只允許建立「Production deploy-config commit」更新：

```text
old digest
→ accepted Staging digest
```

若其中包含 source identity literal，也必須與 accepted source commit 一致。

這種 config-only commit 不得改產品功能。

---

# 6. Legacy Deployment Identity Compatibility

目前已知五個欄位：

```text
TICENPI_ENVIRONMENT
TICENPI_RELEASE_ID
TICENPI_COMMIT_SHA
TICENPI_SUPABASE_PROJECT_REF
TICENPI_DATABASE_TARGET
```

目前問題是：

某些舊 manifest / preflight 把這些 Deployment Identity metadata 當成 `requiredEnv`，要求永久存在 shared env。

本輪不要做正式 Deployment Identity V2 migration。

本輪也禁止：

```text
為了過 preflight
把五鍵永久人工寫進 shared/.env
```

如果這五鍵是唯一 blocker：

先確認等價 identity 可由 authoritative source 唯一得到：

```text
environment
← Production deploy target

source commit
← accepted-staging source_commit

artifact
← accepted-staging backend/frontend digest

Supabase project
← canonical Production SUPABASE_URL / Production config

database target
← canonical Production database policy/config

release id
← existing release/promote tooling
```

如果任何一項無法唯一判定：

```text
LEGACY_IDENTITY_COMPATIBILITY = BLOCKED
```

停止。

如果現有 tooling 已有安全的 release-scoped / deploy-time injection path，可以使用。

如果沒有：

不要整個關掉 preflight。

先輸出：

```text
TEMP_COMPATIBILITY_CHANGE =
FILES =
EXACT_BEHAVIOR =
WHY_SAFE =
TEST =
ROLLBACK =
```

等待使用者確認後才修改。

---

# 7. Production Preflight

執行目前 repo 已存在的 canonical Production preflight。

必須保留：

- source identity
- accepted artifact
- Production target
- Supabase target
- DB target
- required static secrets/config
- rollback
- domain/port
- auth prerequisites
- commercial / Seat prerequisites
- health prerequisites

只要還有五鍵以外 blocker：

```text
OTHER_BLOCKERS != NONE
```

停止，不得一起 bypass。

輸出：

```text
PRODUCTION_PREFLIGHT =
LEGACY_IDENTITY_RULE_BLOCKER =
OTHER_BLOCKERS =
READY_FOR_PRODUCTION_PROMOTION =
```

---

# 8. Production Mutation Boundary

如果：

```text
STAGING_ACCEPTED = YES
SOURCE_IDENTITY_KNOWN = YES
ARTIFACT_IDENTITY_KNOWN = YES
ROLLBACK_READY = YES
PRODUCTION_PREFLIGHT = PASS
OTHER_BLOCKERS = NONE
```

而且沒有新的破壞性或非預期變更，則：

```text
READY_FOR_PRODUCTION_PROMOTION = YES
EXACT_COMMAND =
EXPECTED_SOURCE_COMMIT =
EXPECTED_ARTIFACT =
EXPECTED_PRODUCTION_TARGET =
ROLLBACK_TARGET =
```

**不要停下來再次詢問。直接執行 Production promotion。**

本輪使用者已授權：

```text
preflight 全部 PASS
+ artifact / target / rollback 已確認
+ 沒有新的破壞性操作
→ 直接 Production promotion
```

只有遇到以下任一項，才 STOP 等人工確認：

- DB migration
- schema destructive change
- secret 新增／修改／輪替
- DNS 變更
- port 變更
- runtime architecture 變更
- 非預期 source 修改
- 非預期 config 修改
- 任何不在目前 accepted Staging release / 既有 Production promotion 範圍內的 mutation

如果只是既有 release promotion 所需、且可由 accepted Staging evidence 唯一決定的正常 config-only pin 更新，例如：

```text
Production compose digest
→ accepted Staging digest

Production source identity literal
→ accepted Staging source_commit
```

這屬於本輪預期 promotion config reconciliation，不需再次詢問；但必須先通過既有測試／CI／preflight，不得夾帶功能修改。

---

# 9. 直接執行 Production

符合 §8 條件後：

使用現有 canonical promote/deploy path。

Docker → Docker 優先：

```powershell
.\promote.ps1 <product> production
```

或 repo 當前已驗證的等價 canonical command。

禁止自行建立第二套 Production deploy script。

執行後不要停在 DEPLOYED；繼續完成 §10 Runtime Verification 與 §11 Production Acceptance。

---

# 10. Production Runtime Verification

部署完成後至少驗：

```text
RUNNING_ENVIRONMENT = production
RUNNING_SOURCE_COMMIT = expected
RUNNING_ARTIFACT = accepted artifact
RUNNING_SUPABASE_TARGET = expected Production
RUNNING_DATABASE_TARGET = expected Production
RUNNING_RELEASE_ID = traceable
```

接著：

## Health

- health
- ready

## Auth

- no token → deny
- invalid token → deny
- real Production login/JWT → expected allow

## Commercial / Seat

適用時：

- entitled / assigned → allow
- unauthorized / unassigned → deny

不得用 Production test bypass 取代真人驗證。

## Core Canary

跑該產品最小真實核心流程。

---

# 11. Production Acceptance

```text
DEPLOYED != ACCEPTED
```

只有：

- runtime identity PASS
- health PASS
- auth PASS
- commercial PASS（適用時）
- core canary PASS

才能：

```text
PRODUCTION_ACCEPTED = YES
GO_LIVE = YES
```

若需要真人登入但目前無法自動完成：

```text
PRODUCTION_DEPLOYED = YES
PRODUCTION_ACCEPTED = NO
HUMAN_CANARY_REQUIRED = YES
```

不要假裝 GO_LIVE。

---

# 12. Evidence

完成後記錄：

```text
TARGET_PRODUCT =
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
```

Evidence 存在目前 release evidence convention 下。

不要覆寫 accepted-staging evidence。

---

# 13. 下一個本機新版本

本輪 Production baseline 成功後，只留下：

```text
NEXT_LOCAL_RELEASE_REQUIRES_UNIVERSAL_WORKFLOW_MIGRATION = YES
```

**本輪不要讀、不要套用、不要依賴 `new-workflow.md`。**

該文件目前仍屬未來 Workflow 草案／待完成規範，不是本輪 Production baseline 的執行依據。

只有等未來真的出現新的本機版本，而且新的 Universal Workflow 已完成並獲准後，才在下一個 release 任務中讀取並套用：

```text
F:\00-Ticenpi-SaaS\deploy\docs\system\new-workflow.md
```

屆時再執行已核准的 Deployment Identity / Workflow migration。

下一版才正式走：

```text
Local
→ Local runtime
→ Tests
→ CI
→ Immutable Artifact
→ Staging
→ Runtime Identity
→ Auth / Commercial / E2E
→ STAGING ACCEPTED
→ Production Preflight
→ Same Artifact Promotion
→ Canary
→ PRODUCTION ACCEPTED
```

---

# 14. Scope Guard

本輪只處理 `TARGET_PRODUCT`。

不要：

- 順手處理其他產品
- 順手 merge Universal Workflow workstream
- 順手 merge Product Registry
- 順手重構 Central Seat
- 順手修 unrelated repo
- 順手整理所有 manifests
- 順手把 Legacy compatibility 變永久 architecture

若發現另一產品問題：

只輸出：

```text
OUT_OF_SCOPE_FINDING =
AFFECTED_PRODUCT =
FOLLOWUP_REQUIRED = YES
```

不要切換產品繼續做。

---

# 15. 最終輸出

```text
TARGET_PRODUCT =

PRODUCT_REPO =

DELIVERY_MODEL =

STAGING_ACCEPTED =

STAGING_RELEASE_ID =

SOURCE_COMMIT =

ACCEPTED_ARTIFACT =

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
