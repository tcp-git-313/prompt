# DM / ExtractionHub — FORMAL RECONCILIATION + LOCAL→WHEEL→DOCKER→STAGING PIPELINE OWNER

## 任務目標

目前 forensic audit 與 clean rehearsal 已確認：

- `127.0.0.1:8020` 是 ExtractionHub 本機 Uvicorn，不是 DM。
- 真正的 DM Staging 比對 / Shadow / Shared Engine 驗收應在 `dm-staging.ticenpi.com` 的 DM runtime 上進行。
- 可信 DM Staging lineage：
  - `release/dm-shared-engine-staging-20260923`
  - → `release/dm-central-seat-staging-20260924`
  - trusted commit：`2dd02e2378600f02b9d14590391b9edf84bc14a2`
  - trusted release：`20260924-213157`
- integration baseline：
  - `dm/integration-20260920`
  - `561bd8c76fec5c0fb86cb9cf4f56734d28367b8a`
- clean rehearsal 已證明：
  - fast-forward lineage 有效
  - merge 無 conflict
  - Shared Engine clean wheel 保留
  - comparator / Shadow / Legacy primary 保留
  - Central Seat code present，但預設仍是 `DM_SEAT_POLICY=shadow`
- Shared Engine clean release：
  - ExtractionHub commit：`11762508332bb2508538761302a90f8dca5f2586`
  - wheel：`ticenpi_shared_engine-0.1.0-py3-none-any.whl`
  - SHA256：`e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3`

本任務要正式完成三件事：

1. 將可信 Staging lineage **安全 fast-forward 回 integration branch**。
2. 把 ExtractionHub 的正確開發/發布路徑固定為：
   `本機 8020 → clean source → hosted wheel → DM pin → Docker/ARM64 → DM Staging`
3. 找出並實際驗證「DM Staging 真正的 Shadow / Comparator / Shared Engine 驗收網址」，回報可直接開啟的完整 URL，不再把 8020 當作 DM Staging 比對入口。

---

# 核心架構要先釐清

8020 不是要被「同步成 DM」。

8020 的正確角色：

```
ExtractionHub source
↓
本機 127.0.0.1:8020
↓
Adapter / transport / canonical / semantics 開發與單體測試
```

真正進 DM 的路徑：

```
ExtractionHub clean source
↓
Hosted CI build wheel
↓
Immutable wheel SHA
↓
DM shared_engine_lock.json pin
↓
DM backend image
↓
ARM64 Docker image
↓
DM Staging
↓
Shadow / Comparator / Presentation / Auth / Seat E2E
```

因此：

- 8020 可以是「本機開發來源」
- 但不能直接代表 DM Staging
- 若 ExtractionHub source 有變更，必須走 clean wheel / pin / image / Staging
- 若 ExtractionHub source 沒變，不要無意義重建新 wheel

---

# HARD SAFETY

不要：

- 在現有 dirty `F:\00-Ticenpi-SaaS\TicenpiDM` 直接操作
- commit 目前 dirty WIP
- reset / clean / stash dirty checkout
- merge dirty wheel / dirty lock
- 使用 ExtractionHub main dirty source 取代 clean release
- hot patch Staging container
- deploy Production
- 改 Production DB
- 改 591 / Sign / Post
- 改 Central Seat DB data
- assign / release Seat
- 補 entitlement
- 切 Core primary
- 刪 Legacy scraper

---

# PHASE 0 — VERIFY CURRENT STATE

重新確認：

## DM

- integration branch
- integration HEAD
- staging branch
- staging HEAD
- branch ancestry
- remote branch state
- worktree list
- dirty main checkout untouched

## ExtractionHub

確認 clean release：

`11762508332bb2508538761302a90f8dca5f2586`

wheel SHA：

`e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3`

不要把目前 main dirty state 當 clean release。

如果任何 commit / SHA 與上述不符：

HARD STOP

BASELINE DRIFT DETECTED

---

# PHASE 1 — FORMAL FAST-FORWARD RECONCILIATION

使用全新 clean worktree。

禁止使用 dirty main checkout。

建立 temporary clean worktree，例如：

`F:\00-Ticenpi-SaaS.worktrees\dm-integration-formal-reconcile-20260927`

目標 branch：

`dm/integration-20260920`

先 fetch remote。

確認：

```
target = current remote dm/integration-20260920
source = release/dm-central-seat-staging-20260924 @ 2dd02e2
```

要求：

- source 必須是 target 的 descendant
- 使用 fast-forward-only
- 不建立不必要 merge commit

若 remote integration 已經前進：

重新 merge-base / ancestry。

若不再能 FF：

HARD STOP

FORMAL FF NO LONGER SAFE

---

# PHASE 2 — FORMAL GATES BEFORE PUSH

在正式 reconciliation worktree 跑：

- Shared Engine tests
- Shadow tests
- auth tests
- entitlement tests
- Central Seat tests
- platform_admin tests
- backend relevant/full suite
- frontend Vitest
- frontend build
- cookie / tenant regression if repo has canonical gate
- git diff --check

重新驗：

Wheel SHA：

`e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3`

Source commit：

`11762508332bb2508538761302a90f8dca5f2586`

`source_clean=true`

Semantic / Presentation / Formatter Registry：

`1 / 1 / 1`

確認：

- Legacy primary = YES
- Core primary = NO
- Shadow failure isolated
- platform_admin exemption preserved
- local bypass remains local-only

任何 gate FAIL：

不要 push。

---

# PHASE 3 — UPDATE INTEGRATION REMOTE

只有 Phase 2 全 PASS 才允許：

將 `dm/integration-20260920` fast-forward 到可信 staging lineage。

要求：

- fast-forward only
- no force push
- no rewrite
- record before SHA
- record after SHA
- verify remote after push

現有 dirty main checkout仍不要碰。

推送後：

重新 fetch remote。

確認：

`origin/dm/integration-20260920`

包含：

- Shared Engine
- Shadow comparator
- clean wheel / lock
- Central Seat code
- tests

---

# PHASE 4 — DEFINE CANONICAL EXTRACTIONHUB DEV → RELEASE FLOW

盤點目前 repo 內已有：

- local run command
- wheel build workflow
- hosted CI workflow
- artifact identity
- DM lock update mechanism
- Docker image build
- ARM64 CI
- Staging manifest/deploy

不要另造第二套流程。

產出 canonical flow：

```
A. ExtractionHub 本機開發
   http://127.0.0.1:8020
   ↓
B. ExtractionHub tests
   ↓
C. clean source commit
   ↓
D. hosted CI wheel
   ↓
E. immutable wheel SHA
   ↓
F. DM lock pin
   ↓
G. DM tests
   ↓
H. DM Docker ARM64 image
   ↓
I. Staging deploy
   ↓
J. DM Staging Shadow / Comparator / UI E2E
```

回答：

### 8020 能不能「同步」？

必須用架構語言回答：

- 能作為 local development source
- 但不是直接 rsync / copy 到 Staging
- 正確同步單位是：
  - source commit
  - built wheel
  - SHA pin
  - Docker image
  - Staging release

如果 repo 已有自動化：

沿用。

如果缺一段：

只列缺口，不要本輪擴大開發新 CD 系統。

---

# PHASE 5 — DISCOVER EXACT DM STAGING SHADOW / COMPARATOR URL

這一階段很重要。

不要靠記憶猜網址。

從目前已 reconciliation 的 DM source / frontend router / backend admin router 找出：

- Shadow Console frontend route
- Shared Engine identity route
- comparator run list/detail route
- admin access requirement

查：

- Vue router
- route definitions
- admin navigation
- backend routers
- current built frontend route

確認歷史可能路徑：

`/admin/extraction-shadow`

但必須由 current source 證明。

然後在：

`https://dm-staging.ticenpi.com`

做 read-only route check。

要求最後回報：

## Exact URLs

DM Staging root:
`https://dm-staging.ticenpi.com/`

Shadow Console:
`https://dm-staging.ticenpi.com/<verified-path>`

Shared Engine identity API:
`https://dm-staging.ticenpi.com/<verified-api-path>`

如果另有 comparator detail route：

回報 exact URL pattern。

不能發明。

---

# PHASE 6 — VERIFY STAGING ROUTE IS ACTUALLY LIVE

只確認目前 Staging。

不要 deploy 新版本，除非 integration reconciliation 本身沒有改 Staging runtime。

驗：

- route exists
- frontend loads
- login gate behaves
- platform_admin requirement behaves
- release header / runtime identity matches trusted Staging
- Shadow Console code actually present in deployed frontend/backend

如果沒有 authenticated admin session：

可以標：

AUTHENTICATED_UI_NOT_RUN

但 route existence 仍要從 source + HTTP/static evidence確認。

---

# PHASE 7 — CENTRAL SEAT STATUS — DO NOT CONFUSE WITH THIS TASK

重新記錄：

目前 trusted Staging：

`DM_SEAT_POLICY=shadow`

因此：

```
Seat code present         YES
Authoritative mode exists YES
Authoritative enabled     NO
```

本輪不要改成 require。

只確認 integration reconciliation 沒有把這段弄壞。

Central Seat authoritative cutover 是下一個獨立任務。

不要因為本輪要收 Shared Engine 就順手改 Seat policy。

---

# PHASE 8 — FINAL NEXT-STEP MAP

正式 reconciliation 完成後，輸出兩條清楚路徑：

## ExtractionHub / Shared Engine closeout

```
Verified DM Staging Shadow URL
↓
platform_admin login
↓
Shared Engine identity
↓
YUCT human UI
↓
Shadow / comparator
↓
rollback / restore
↓
READY
```

## Central Seat closeout

```
DM_SEAT_POLICY=require
↓
Staging deploy
↓
DM entitlement
↓
assigned user ALLOW
↓
unassigned user DENY
↓
platform_admin exemption
↓
READY
```

兩條不要混在一起。

---

# FINAL REPORT FORMAT

# DM / EXTRACTIONHUB FORMAL RECONCILIATION + PIPELINE REPORT

## 1. Formal Reconciliation

Integration before:
Staging source:
FF possible:
YES / NO

Integration after:
Remote verified:
YES / NO

Dirty main checkout touched:
NO

## 2. Gates

Backend:
Frontend:
Shared Engine:
Shadow:
Auth:
Entitlement:
Seat:
platform_admin:
Cookie/Tenant:
Build:
diff-check:

## 3. Shared Engine Identity

Source commit:
Wheel SHA:
source_clean:
Semantic:
Presentation:
Formatter:

PASS / FAIL

## 4. 8020 Role

URL:
Repo:
Purpose:
Is DM:
NO

Can participate in pipeline:
YES / NO

Canonical role:

## 5. Canonical Local → Staging Flow

ExtractionHub local:
↓
Tests:
↓
Clean commit:
↓
Wheel:
↓
SHA pin:
↓
DM:
↓
Docker ARM64:
↓
Staging:

Missing automation:
NONE / <list>

## 6. Exact DM Staging URLs

Root:

Shadow Console:

Shared Engine identity API:

Comparator detail/list:

Evidence source:

## 7. Staging Route Verification

Route exists:
Frontend loads:
Admin gate:
Runtime release:
Authenticated admin tested:

## 8. Central Seat Status

Code present:
YES

Authoritative supported:
YES

Current Staging policy:
shadow / require / unknown

Authoritative active:
YES / NO

Regression:
YES / NO

## 9. Shared Engine Next Closeout

Remaining:

- platform_admin identity
- YUCT human UI
- Shadow comparator
- rollback / restore

## 10. Central Seat Next Closeout

Remaining:

- policy=require
- entitlement
- assigned/unassigned E2E
- platform_admin E2E

## 11. Isolation

Dirty checkout:
UNCHANGED

Staging runtime:
UNCHANGED unless explicitly stated

Staging DB:
UNCHANGED

Production:
UNCHANGED

Production DB:
UNCHANGED

Other products:
UNCHANGED

## 12. Final Result

只能：

FORMAL RECONCILIATION COMPLETE — READY FOR STAGING VALIDATION

或

FORMAL RECONCILIATION BLOCKED

---

# HARD STOP

不要：

- 把 8020 當 DM Staging URL
- 用 dirty ExtractionHub main build release wheel
- commit dirty DM checkout
- force push
- deploy Production
- 改 Staging DB
- 啟用 Seat authoritative policy
- 補 entitlement
- assign Seat
- 修改其他產品

本任務完成 integration reconciliation、pipeline clarification、exact DM Staging URL discovery 後停止。
