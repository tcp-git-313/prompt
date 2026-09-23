# DM STAGING FINAL HUMAN VALIDATION + ROLLBACK CLOSEOUT OWNER

## 任務目標

接續目前已完成的 Shared Engine Clean Release + DM Staging Evidence。

目前狀態：

- ExtractionHub clean release：PASS
- Hosted wheel CI：PASS
- DM wheel pin：PASS
- Hosted DM ARM64 image：PASS
- DM Staging deploy：PASS
- Staging health：PASS
- dm-gates-full：25/25 PASS
- Legacy YUCT auth regression：PASS
- Production：未部署
- Post / 591：未修改
- Extraction Core：尚未切 Primary

目前只剩最後 closeout：

1. 真正登入 DM Staging
2. 以 platform_admin 驗證 protected Shared Engine runtime identity
3. 真人瀏覽器操作 YUCT UI
4. 驗證 Legacy 仍為正式使用者路徑、Shadow 不影響結果
5. 執行 Staging rollback drill
6. 回復 candidate release
7. 再次確認 live identity / health
8. 將狀態從 PARTIAL 升級為 READY 或明確 BLOCKED

本任務不要重做：

- wheel build
- ExtractionHub release
- DM image build
- Shared Engine architecture
- Semantic / Presentation Engine
- Adapter
- CI pipeline

除非 live evidence 證明現有 artifact 或 deployment 有問題。

---

# 已知 Release Evidence

## ExtractionHub

Clean source commit：

11762508332bb2508538761302a90f8dca5f2586

Hosted CI run：

35863227635

Artifact：

extractionhub-shared-engine-11762508332bb2508538761302a90f8dca5f2586

Wheel：

ticenpi_shared_engine-0.1.0-py3-none-any.whl

Wheel SHA256：

e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

Expected：

source_clean = true

semantic contract = 1
presentation contract = 1
formatter registry = 1

---

## DM

Image source commit：

25db5f9b8b65a231dafb7cc8f1f05262fb7519aa

Manifest commit：

34a59a7fb50ce5f8b6071e90ab5ee12ca763f3fb

Hosted ARM64 CI：

35865720063

Manifest CI：

35868383441

Backend image：

ghcr.io/tcp-git-313/ticenpi-dm@sha256:51efc74c75e35d0d857434e3f113bb0546c9072b2b12398f4c33b1555ffb0e83

Frontend image：

ghcr.io/tcp-git-313/ticenpi-dm@sha256:0842f90d476ab7fa4b74d1c70781cc5bd1b1a7513848e926e5f5c9c148230093

Current Staging release：

20260923-215914

Previous Staging release：

20260922-131327

Staging URL：

https://dm-staging.ticenpi.com/

---

# PHASE 0 — PREFLIGHT

先確認：

- current Staging release
- running backend digest
- running frontend digest
- /api/health
- external HTTPS
- manifest identity
- no active deploy
- no other writer changing DM Staging
- rollback target still exists

不要修改 Production。

不要修改 Production DB。

如果 current Staging 已不是：

20260923-215914

或 running digest 與 expected 不符：

HARD STOP

輸出：

DM STAGING CANDIDATE DRIFT

不要直接 rollback 或重新 deploy。

---

# PHASE 1 — LOGIN INTERACTION

使用正常 Staging Google / Supabase login flow。

不要：

- 注入 JWT
- 手工偽造 session
- 使用 bypass header
- 改 DB role
- 使用 loopback-only admin shortcut
- 修改 platform_admin 資料

如果需要使用者本人完成 Google login：

停在登入頁。

明確告訴使用者：

請完成登入後告訴我。

登入完成後從同一 browser session 繼續。

不要重新開匿名 session。

---

# PHASE 2 — VERIFY PLATFORM_ADMIN SESSION

登入完成後確認目前 session：

- authenticated
- role = platform_admin
- environment = staging

使用現有正式 admin / current-user endpoint。

不要只靠 UI 顯示「Admin」字樣判定。

如果 session 不是 platform_admin：

HARD STOP

輸出：

STAGING LOGIN OK — NOT PLATFORM_ADMIN

不要修改使用者角色。

---

# PHASE 3 — PROTECTED SHARED ENGINE IDENTITY

在同一個已登入 platform_admin session 呼叫：

GET /api/admin/shared-engine/identity

必須取得 200。

核對至少：

source_clean
source_commit
wheel_version
wheel_sha256
semantic_contract_version
presentation_contract_version
formatter_registry_version
build_identity
loaded_from

Expected：

source_clean = true

source_commit =
11762508332bb2508538761302a90f8dca5f2586

wheel_version =
0.1.0

wheel_sha256 =
e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

semantic contract =
1

presentation contract =
1

formatter registry =
1

如果欄位命名略有不同：

依實際 API schema 判斷。

不要因 field name 不同就擅自修改 backend。

如果 identity 任一核心值 mismatch：

HARD STOP

輸出：

SHARED ENGINE LIVE IDENTITY MISMATCH

並列：

Expected
Actual

不要繼續 rollback。

---

# PHASE 4 — HUMAN UI SMOKE

使用同一個已登入 platform_admin browser session。

不要使用 fixture 取代真人 UI。

實際從 DM UI 跑一次合法、公開、目前可正常使用的 YUCT 房源。

優先使用已存在的 Staging controlled validation URL / known-good YUCT URL。

不要自行高頻探測新房源。

驗：

1. URL 可以提交
2. Legacy YUCT 正常完成
3. 使用者看到的主要結果正常
4. Shadow 開啟時不阻擋 Legacy response
5. Shadow 失敗時 Legacy 仍能正常回應
6. Shadow Console 有資料
7. semantic comparator 正常
8. presentation profile 正常

---

# PHASE 5 — REQUIRED SEMANTIC EXAMPLE

如果目前 Shadow 資料中存在以下語意案例：

Legacy：

3房2廳2衛

Core：

3房(室)2廳2衛

要求：

Status：

NORMALIZED_MATCH

Canonical：

3 / 2 / 2

DM compact display：

3房2廳2衛

如果 UI 支援 slash preview：

3/2/2

且 comparator status 仍：

NORMALIZED_MATCH

如果現場沒有剛好這筆資料：

不要偽造 live evidence。

可以用已存在 unit/integration evidence補充：

LIVE SAMPLE NOT AVAILABLE

但不得宣稱真人 UI 已驗該特定值。

---

# PHASE 6 — VERIFY LEGACY IS STILL PRIMARY

從真人 UI / network / response behavior 確認：

- Legacy YUCT 仍是 user-visible primary result
- Shared Engine / Core 只在 Shadow
- Core failure 不改變正式 Legacy response
- 沒有切 Primary
- 沒有刪 Legacy

輸出：

LEGACY PRIMARY CONFIRMED

或

LEGACY PRIMARY NOT CONFIRMED

若 not confirmed：

HARD STOP。

---

# PHASE 7 — PRE-ROLLBACK SNAPSHOT

只有 Phase 1–6 全部 PASS 才能做 rollback drill。

記錄 candidate：

Release：
20260923-215914

Backend digest：
sha256:51efc74c75e35d0d857434e3f113bb0546c9072b2b12398f4c33b1555ffb0e83

Frontend digest：
sha256:0842f90d476ab7fa4b74d1c70781cc5bd1b1a7513848e926e5f5c9c148230093

Manifest：
34a59a7fb50ce5f8b6071e90ab5ee12ca763f3fb

Rollback target：

20260922-131327

先確認 rollback target snapshot / release files 存在且完整。

如果不存在：

ROLLBACK DRILL BLOCKED

不要猜。

---

# PHASE 8 — STAGING ROLLBACK

只對：

dmruntimestaging

執行既有正式 rollback 流程。

現有記錄指令：

.\deploy.ps1 -Service dmruntimestaging -Rollback -To 20260922-131327 -PythonPath 'C:\Users\Tu\AppData\Local\Programs\Python\Python313\python.exe'

但執行前：

先依現場 deploy tooling / help / current docs 確認參數仍正確。

不要盲目複製舊指令。

rollback 前做：

- dry-run / preflight if supported
- target service confirmation
- current release confirmation
- rollback target confirmation
- unrelated manifest change check

若中央 deploy preflight BLOCKED：

停止。

不要 force。

---

# PHASE 9 — VERIFY PREVIOUS RELEASE

Rollback 完成後確認：

- service healthy
- /api/health 200
- external HTTPS 200
- UI loads
- running digest 對應 previous release
- release identity = 20260922-131327

不要要求 previous release 具有新版 Shared Engine endpoint。

如果 previous release 沒有：

/api/admin/shared-engine/identity

這是合理的。

不要判失敗。

核心是：

rollback 到 previous release 可正常服務。

輸出：

ROLLBACK TO PREVIOUS PASS

或

FAIL

---

# PHASE 10 — REDEPLOY CANDIDATE

使用既有正式 candidate release / immutable digest 流程，把：

20260923-215914

重新恢復為 active Staging。

不要 rebuild image。

不要 rebuild wheel。

不要生成新 artifact。

必須回到同一 immutable candidate。

若工具是以 release promotion / deploy candidate 的方式恢復：

依現有正式流程。

---

# PHASE 11 — VERIFY RESTORED CANDIDATE

Candidate 恢復後再次確認：

- /api/health = 200
- external HTTPS = healthy
- UI = api ok
- release = 20260923-215914
- backend digest = expected
- frontend digest = expected

然後在已登入 platform_admin session：

再次呼叫：

/api/admin/shared-engine/identity

重新核對：

source_clean = true
source_commit = 11762508332bb2508538761302a90f8dca5f2586
wheel_version = 0.1.0
wheel_sha256 = e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3
semantic = 1
presentation = 1
formatter registry = 1

要求：

RESTORED IDENTITY MATCH = PASS

---

# PHASE 12 — POST-ROLLBACK HUMAN SMOKE

Candidate 恢復後：

不要完整重跑所有測試。

只做最小真人 smoke：

1. 已登入狀態仍可正常使用，或重新正常登入
2. platform_admin identity 200
3. DM UI 正常
4. 一次 YUCT Legacy 操作正常
5. Shadow 不影響 Legacy
6. Admin Shadow Console 正常

---

# PHASE 13 — FINAL ISOLATION AUDIT

確認：

Production deploy：
NONE

Production DB changes：
NONE

Post writes：
NONE

591 writes：
NONE

Source Registry publish：
NONE

Core Primary switch：
NO

Legacy deletion：
NO

其他 active service definitions：
UNCHANGED

---

# PHASE 14 — DOCUMENTATION

更新既有：

DM current-state
DM history
Staging deployment evidence / handoff

ExtractionHub 若沒有新 source change：

不要改 source release。

只補必要 evidence reference。

記錄：

- platform_admin login result
- live shared-engine identity
- human YUCT UI smoke
- rollback to previous
- previous health
- candidate restore
- restored identity
- final active release

不要建立第二套 SSOT。

---

# FINAL REPORT FORMAT

# DM STAGING FINAL HUMAN VALIDATION + ROLLBACK REPORT

## 1. Baseline

Current release:
Backend digest:
Frontend digest:
Manifest:
Health:

## 2. Human Login

Google/Supabase login:
PASS / FAIL

Authenticated:
YES / NO

platform_admin:
YES / NO

## 3. Live Shared Engine Identity

HTTP:
200 / FAIL

source_clean:
source_commit:
wheel_version:
wheel_sha256:
semantic:
presentation:
formatter registry:
build_identity:
loaded_from:

Expected match:
PASS / FAIL

## 4. Human YUCT UI Smoke

URL:
Legacy result:
Shadow:
Admin console:
Result:
PASS / FAIL

## 5. Semantic / Presentation

Live NORMALIZED_MATCH example:
PASS / NOT_AVAILABLE / FAIL

Canonical:
DM compact:
DM slash:

Comparator unchanged:
PASS / NOT_TESTED / FAIL

## 6. Legacy Primary

Legacy remains primary:
PASS / FAIL

Core Primary:
NO / YES

## 7. Rollback

From:
20260923-215914

To:
20260922-131327

Preflight:
PASS / FAIL

Rollback:
PASS / FAIL / BLOCKED

Previous health:
PASS / FAIL

Previous release identity:
PASS / FAIL

## 8. Candidate Restore

Restore candidate:
PASS / FAIL / BLOCKED

Release:
Backend digest:
Frontend digest:
Health:

## 9. Restored Shared Engine Identity

HTTP:
200 / FAIL

source_clean:
source_commit:
wheel_sha256:
contracts:

Expected match:
PASS / FAIL

## 10. Post-Restore Human Smoke

Login:
YUCT:
Shadow:
Admin console:

Result:
PASS / FAIL

## 11. Isolation

Production:
UNCHANGED / CHANGED

Production DB:
UNCHANGED / CHANGED

Post:
UNCHANGED / CHANGED

591:
UNCHANGED / CHANGED

Source Registry:
UNCHANGED / CHANGED

## 12. Final Active Staging

Release:
Backend digest:
Frontend digest:
Health:
Shared Engine identity:

## 13. Remaining Gaps

如果沒有：

NONE

否則逐項列出。

## 14. Final Result

只能：

TICENPI SHARED ENGINE CLEAN RELEASE + DM STAGING READY

或

TICENPI SHARED ENGINE CLEAN RELEASE + DM STAGING BLOCKED

---

# HARD STOP

完成後不要：

- 部署 Production
- 修改 Production DB
- 修改 Post
- 修改 591
- 開始新 Adapter
- 切 Core Primary
- 刪 Legacy scraper

如果 READY，等待使用者核准下一階段。
