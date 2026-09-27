# DM UPDATED STAGING — FULL CLOSEOUT OWNER
## Shared Engine + Shadow Comparator + Central Seat E2E + Rollback/Restore

## 任務目標

前一個 DM reconciliation / deploy 任務目前正在更新 DM Staging。

本任務 **只能在前一個 deploy 已完全結束後開始**。

你要接手「更新後的 DM Staging」作為唯一基準，完成剩餘所有驗收與必要的最小修正，直到可以明確判定：

- DM integration lineage 已對齊
- Shared Engine clean artifact 正確
- platform_admin 真實 Staging 權限正常
- Central Seat authoritative 行為正常
- YUCT Legacy primary 正常
- Core / Shared Engine Shadow 正常
- Semantic Comparator / Presentation 正常
- Staging Shadow Console 有真正可用的管理入口
- rollback → restore drill 完成
- Production 完全未動

最後目標：

> 將 DM / ExtractionHub Shared Engine + Central Seat 的 Staging closeout 一次完成，不再留下「只有 backend API、沒有 UI」、「只有 unit test、沒有 live Staging evidence」、「只有 rollback target、沒有實際 rollback」這類缺口。

---

# 已知歷史基準

以下只能當歷史 expected，開始時必須重新讀取更新後 runtime，不可直接假設仍相同。

## Shared Engine clean release

ExtractionHub clean source commit:

11762508332bb2508538761302a90f8dca5f2586

Wheel:

ticenpi_shared_engine-0.1.0-py3-none-any.whl

Wheel SHA256:

e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

Contracts:

- semantic = 1
- presentation = 1
- formatter registry = 1
- source_clean = true

## Historical trusted DM Staging lineage

Shared Engine branch:

release/dm-shared-engine-staging-20260923

Central Seat branch:

release/dm-central-seat-staging-20260924

Historical trusted Staging commit:

2dd02e2378600f02b9d14590391b9edf84bc14a2

Historical release:

20260924-213157

注意：

本任務開始時，DM 已可能更新到新的 commit / release / digest。

所以：

> LIVE RUNTIME EVIDENCE 優先於上述歷史值。

---

# 重要已知事實

之前已查清：

1. `127.0.0.1:8020` 是 ExtractionHub 本機開發 Uvicorn。
2. 8020 **不是 DM Staging 驗收入口**。
3. 正確交付鏈：

```
ExtractionHub source
→ clean commit
→ hosted CI wheel
→ immutable wheel SHA
→ DM pin
→ DM CI
→ ARM64 Docker image
→ DM Staging
→ Shadow / Comparator / Auth / Seat E2E
```

4. 歷史上 backend 已存在：

- `/api/admin/shared-engine/identity`
- `/api/admin/extraction-shadow/report`
- `/api/admin/extraction-shadow/runs`
- `/api/admin/extraction-shadow/runs/{run_id}`

5. 之前查到 `/admin/extraction-shadow` 並沒有真正的 frontend Shadow Console page，只有一般 frontend fallback。
6. 之前 Staging 的 Shadow enable config 曾被發現未顯式啟用。
7. Central Seat policy 曾被 live runtime 證明為 `require`，不要再引用舊文件的 `shadow` 當真實狀態。
8. platform_admin 真實 Staging 登入與 live identity 尚未完整 closeout。
9. YUCT 真人 UI + Shadow comparator 尚未完整 closeout。
10. rollback / restore 尚未實際演練完成。

---

# SESSION / COLLISION RULE

開始前確認前一個 deploy owner 已經完全結束。

必須取得：

- DEPLOY PASS / FAIL
- new release
- new commit
- backend digest
- frontend digest
- health result
- CI result
- deploy gate result

如果 deploy 還在執行：

HARD STOP

```
PREVIOUS DEPLOY STILL ACTIVE
```

本任務開始後：

> 你是唯一 DM Writer / Deploy Owner。

其他 Session 只能 read-only。

---

# PHASE 0 — UPDATED RUNTIME BASELINE

從 live Staging 重新取得：

https://dm-staging.ticenpi.com/

至少確認：

- environment
- release
- commit
- backend digest
- frontend digest
- x-ticenpi-release
- /api/health
- deployed Supabase environment identity
- DM_REQUIRE_AUTH
- DM_SEAT_POLICY
- Shadow enable/config
- Shared Engine package identity if accessible

輸出：

## UPDATED STAGING BASELINE

Release:
Commit:
Backend digest:
Frontend digest:
Health:
Seat policy:
Shadow enabled:
Environment:

不得沿用舊值。

---

# PHASE 1 — VERIFY DEPLOYED LINEAGE

確認更新後 Staging source commit：

- 位於目前 integration lineage
- 包含 Shared Engine staging lineage
- 包含 Central Seat staging lineage
- 不包含 dirty wheel / dirty lock
- 不來自舊 561bd8c-only baseline

讀：

- current integration remote
- deployed commit
- git ancestry
- worktree topology
- CI artifact provenance
- image labels / release manifest

建立：

```
integration
↓
Shared Engine
↓
Central Seat
↓
current deployed candidate
```

若 lineage 不完整：

HARD STOP

```
UPDATED STAGING LINEAGE INVALID
```

---

# PHASE 2 — SHARED ENGINE ARTIFACT IDENTITY

使用 Staging 真實 platform_admin session 驗：

GET /api/admin/shared-engine/identity

這次不接受：

- localhost bypass
- anonymous request
- copied JWT
- hand-written Authorization
- fake admin fixture

必須：

Google / Supabase real Staging login
→ backend platform_role=platform_admin
→ admin endpoint
→ 200

Expected Shared Engine identity：

source_clean = true

source_commit =
11762508332bb2508538761302a90f8dca5f2586

wheel_version =
0.1.0

wheel_sha256 =
e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

semantic = 1
presentation = 1
formatter registry = 1

若不同：

不要自動換 wheel。

先分類：

- DEPLOYED_OLD_ARTIFACT
- DIRTY_ARTIFACT
- LOCK_MISMATCH
- IDENTITY_ENDPOINT_BUG
- OTHER

---

# PHASE 3 — PLATFORM_ADMIN REAL STAGING E2E

明確分開：

## 正式 Staging admin

```
DM_REQUIRE_AUTH=1
real Google login
platform_role=platform_admin
```

## Local dev bypass

```
localhost
DM_REQUIRE_AUTH=0
LOCAL_DEV_PLATFORM_ADMIN=1
```

兩者不可混用。

Live Staging 必須驗：

- platform_admin recognized
- protected admin API allow
- Shared Engine identity allow
- Shadow admin APIs allow
- entitlement / Seat exemption符合中央 contract

如果 admin session 無法取得：

不要要求使用者貼 token / password。

使用正常瀏覽器 login flow。

---

# PHASE 4 — CENTRAL SEAT LIVE AUTHORITATIVE E2E

先讀 live：

DM_SEAT_POLICY

如果：

`require`

才可宣稱 authoritative active。

如果不是 require：

停止 Seat E2E，回報：

```
SEAT AUTHORITATIVE NOT ACTIVE
```

不要自行偷偷改 config。

## 測試矩陣

優先使用已存在的安全 Staging 測試 customer / members。

需要至少：

### Business Member A

- active business membership
- active DM entitlement
- DM Seat assigned

Expected:

`GET /api/designs → ALLOW / 200`

### Business Member B

- same business customer
- active membership
- same customer entitlement
- NO DM Seat

Expected:

DENY

且 denial 必須是 Seat-related typed reason，不應變成 PRODUCT_ENTITLEMENT_REQUIRED。

### Individual customer

如有既有測試身份：

- active DM entitlement
- Seat not required

Expected:

ALLOW

### platform_admin

Expected:

ALLOW according to central exemption contract

## 資料變更規則

如果缺 DM entitlement / assigned Seat：

不要直接改 table。

只能使用既有正式：

- platform_admin admin RPC
- admin_set_entitlement
- assign_product_seat / release_product_seat
- 或 repo 已定義正式管理流程

每一筆 Staging data change 必須記錄：

- before
- operation
- after
- rollback method

Production 絕對不能碰。

如果無安全測試身份：

允許標：

```
LIVE SEAT E2E BLOCKED — TEST IDENTITY MISSING
```

不要亂建立未知資料。

---

# PHASE 5 — SHADOW BACKEND ENABLEMENT

重新確認 live Staging：

- Shadow backend code present?
- Shadow enable setting?
- Core extractor available?
- Legacy primary still enabled?

目標架構：

```
User YUCT request
↓
Legacy primary → user result
↓
background Shadow Core
↓
Canonical
↓
Semantic comparator
↓
Presentation
↓
Shadow record
```

若 Shadow config = OFF：

先確認這是：

- missing deployment env
- intentional policy
- old config
- current release regression

如果只是 Staging closeout 必須啟用的 config，且 repo / deployment contract 已明確定義：

可以在 Staging 正式 manifest/env flow 中啟用。

禁止：

- Production
- hot edit container
- ad hoc shell export

若需 deploy config change：

走正式 Staging deploy。

---

# PHASE 6 — SHADOW CONSOLE FRONTEND

重新查更新後 frontend。

如果 current release 已經有真正的：

`/admin/extraction-shadow`

驗證即可。

如果仍沒有 UI：

本任務允許補上最小正式 Shadow Console frontend。

因為完整驗收需要可操作的 DM Staging 比對入口。

## UI 最低需求

Route:

`/admin/extraction-shadow`

platform_admin only。

至少顯示：

- latest runs
- run id
- source URL / safe listing identifier
- created time
- Legacy raw
- Core raw
- comparison status
- Canonical
- DM final display
- field-level result
- EXACT_MATCH
- NORMALIZED_MATCH
- MISMATCH
- UNCOMPARABLE
- LEGACY_MISSING
- CORE_MISSING
- BOTH_MISSING

不得顯示：

- JWT
- cookie
- secret
- raw credentials

API 沿用既有 backend：

- /api/admin/extraction-shadow/report
- /api/admin/extraction-shadow/runs
- /api/admin/extraction-shadow/runs/{run_id}

不要再造第二套 comparator API。

UI 必須使用現有 authenticated API client。

## UI Gate

如果補 UI：

- frontend tests
- frontend build
- auth/admin route test
- no Production deploy
- formal Staging CI / image / deploy

---

# PHASE 7 — DISCOVER AND REPORT EXACT STAGING URLS

完成後一定要回報真正可用：

Root:

`https://dm-staging.ticenpi.com/`

Shadow Console:

`https://dm-staging.ticenpi.com/admin/extraction-shadow`

Shared Engine Identity API:

`https://dm-staging.ticenpi.com/api/admin/shared-engine/identity`

Shadow Report API:

`https://dm-staging.ticenpi.com/api/admin/extraction-shadow/report`

Shadow Runs API:

`https://dm-staging.ticenpi.com/api/admin/extraction-shadow/runs`

Run detail:

`https://dm-staging.ticenpi.com/api/admin/extraction-shadow/runs/{run_id}`

但只有在 live route 驗證存在後，才可以把 Shadow Console 標成 READY。

---

# PHASE 8 — YUCT HUMAN UI SMOKE

從真正 DM Staging UI：

1. real login
2. 進正常 DM 功能
3. 使用合法公開 YUCT URL
4. 正常提交
5. 等 user-visible Legacy result
6. 確認頁面正常

不能用：

- fixture-only API
- backend curl 代替 UI
- 8020 代替 DM

驗：

Legacy result = PASS

---

# PHASE 9 — SHADOW / COMPARATOR LIVE VERIFICATION

YUCT 真人操作完成後：

進：

`/admin/extraction-shadow`

找到剛剛同一次 run。

至少核對：

- Legacy raw
- Core raw
- status
- Canonical
- DM final display

若案例中：

Legacy：
`3房2廳2衛`

Core：
`3房(室)2廳2衛`

Expected：

`NORMALIZED_MATCH`

Canonical：

`3 / 2 / 2`

DM final：

`3房2廳2衛`

如果真人案例不是這組：

不要偽造。

驗真實字段即可。

---

# PHASE 10 — LEGACY PRIMARY SAFETY

Live Staging 必須證明：

- Legacy primary = YES
- Core primary = NO
- Shadow failure不影響 user result

做一個可控的 Shadow failure test，只能用既有安全 fixture / injected test mode。

不要破壞 live service。

如果無安全方法：

使用既有 regression test evidence + live architecture evidence，標記 live forced failure NOT_RUN。

---

# PHASE 11 — FULL REGRESSION BEFORE ROLLBACK

在目前 candidate 上跑 canonical gates。

至少：

## Backend

- Shared Engine
- Shadow
- Auth
- entitlement
- Seat
- platform_admin
- Legacy/Core regression

## Frontend

- auth
- admin Shadow Console
- YUCT import
- full Vitest
- build

## Infra

- cookie isolation
- tenant isolation
- dm-gates-full
- health
- git diff --check
- runtime identity

如果 source changed：

要求：

- clean commit
- hosted CI
- ARM64 images
- immutable digests
- Staging deploy evidence

---

# PHASE 12 — ROLLBACK TARGET VALIDATION

不要盲用歷史 target。

先從 deploy history 找：

- current release
- immediate valid previous release
- image digests
- manifest snapshot
- health evidence

確認 rollback target：

- 存在
- artifact 可取得
- deploy tooling 支援
- 不會影響其他 services

輸出：

```
CURRENT_RELEASE=
ROLLBACK_TARGET=
ROLLBACK_SAFE=YES
```

否則：

HARD STOP

```
ROLLBACK TARGET INVALID
```

---

# PHASE 13 — ACTUAL ROLLBACK DRILL

只有前面所有 candidate gates PASS 才執行。

流程：

```
current candidate
↓
formal rollback
↓
previous release
↓
health
↓
basic DM load
↓
formal restore
↓
current candidate
↓
health
↓
runtime identity
↓
Shared Engine identity
```

要求：

- 不碰 Production
- 不改其他 service
- central deploy isolation PASS
- release headers正確
- digests正確

若 rollback 失敗：

停止，不繼續 restore guessing。

依 deploy tooling既有 recovery contract處理。

---

# PHASE 14 — RESTORE CANDIDATE VALIDATION

restore current candidate 後重新驗：

- health 200
- exact release
- exact commit
- backend/frontend digests
- Shared Engine identity
- platform_admin
- Seat policy
- Shadow enabled
- Shadow Console route
- Legacy YUCT basic smoke

不用完整重跑所有 long tests，但關鍵 live gates必須重驗。

---

# PHASE 15 — FINAL CLOSEOUT

只有以下全部 PASS 才可以宣告 READY：

## DM lineage

PASS

## Shared Engine clean identity

PASS

## platform_admin real Staging

PASS

## Central Seat authoritative

PASS 或明確不在此次 release scope，但不得誤報

## assigned / unassigned

PASS if Seat authoritative is in scope

## Shadow enabled

PASS

## Shadow Console frontend

PASS

## YUCT human UI

PASS

## Comparator live

PASS

## Legacy primary

PASS

## rollback

PASS

## restore

PASS

## Production unchanged

PASS

---

# EXTRACTIONHUB LOCAL 8020 RULE

8020 保留作為：

```
ExtractionHub local developer runtime
```

可用於：

- Adapter development
- transport debug
- canonical extraction
- semantics tests

不能作為：

- DM Staging acceptance
- Shadow Console
- Central Seat validation
- platform_admin validation
- production-like E2E

任何 ExtractionHub source change若要進 DM：

```
clean commit
→ hosted wheel
→ SHA pin
→ DM image
→ Staging
```

禁止直接把 8020「同步」到 Staging container。

---

# FINAL REPORT FORMAT

# DM UPDATED STAGING FULL CLOSEOUT REPORT

## 1. Updated Runtime

Release:
Commit:
Backend digest:
Frontend digest:
Health:
Seat policy:
Shadow enabled:

## 2. Lineage

Integration:
Shared Engine ancestor:
Central Seat ancestor:
Deployed candidate:
PASS / FAIL

## 3. Shared Engine

source_clean:
source_commit:
wheel:
wheel SHA:
semantic:
presentation:
formatter:

Result:

## 4. platform_admin

Real Google login:
platform_role:
Admin API:
Identity endpoint:

Result:

## 5. Central Seat

Policy:
Authoritative active:
Business assigned:
Business unassigned:
Individual:
platform_admin exemption:

Result:

## 6. Shadow Runtime

Backend enabled:
Core available:
Legacy primary:
Shadow isolation:

Result:

## 7. Shadow Console

Frontend route:
Auth gate:
Runs list:
Run detail:
Legacy raw:
Core raw:
Canonical:
Status:
DM final:

URL:

Result:

## 8. YUCT Human UI

URL tested:
Legacy result:
Core shadow:
Run recorded:

Result:

## 9. Comparator

EXACT:
NORMALIZED:
MISMATCH:
Canonical:
Presentation:

Live evidence:

Result:

## 10. Tests

Backend:
Frontend:
Seat:
Auth:
Shadow:
Shared Engine:
Cookie/Tenant:
dm-gates-full:
Build:
diff-check:

## 11. Rollback

Candidate:
Rollback target:
Rollback:
Health:
Restore:
Restored candidate:
Identity after restore:

Result:

## 12. Production Isolation

Production:
UNCHANGED

Production DB:
UNCHANGED

Other products:
UNCHANGED

## 13. Exact URLs

DM Staging:

Shadow Console:

Shared Engine Identity:

Shadow Report:

Shadow Runs:

## 14. Remaining Items

只能列真正未完成事項。

若無：

NONE

## 15. Final Result

只能：

DM / EXTRACTIONHUB STAGING CLOSEOUT READY

或

DM / EXTRACTIONHUB STAGING CLOSEOUT BLOCKED

---

# HARD STOP

不要：

- 在 deploy 尚未完成時開始
- 用 8020 代替 DM Staging
- 用 localhost admin bypass 當 Staging evidence
- 要求使用者提供 token / cookie / password
- 修改 Production
- 修改 Production DB
- 修改其他產品
- 切 Core primary
- 刪 Legacy scraper
- 使用 dirty ExtractionHub source build正式 wheel
- 使用 hot patch 取代正式 CI / Docker / deploy

本任務開始後，完成整個 Staging closeout 或明確停在唯一 blocker，不要再把不同專案線混在一起。
