# DM STAGING LINEAGE → INTEGRATION CLEAN RECONCILIATION REHEARSAL

## 任務目標

目前 forensic audit 已確認：

- `127.0.0.1:8020` 是 ExtractionHub 本機 Uvicorn，不是 DM。
- 目前可信 DM Staging 基準是：
  - branch: `release/dm-central-seat-staging-20260924`
  - commit: `2dd02e2378600f02b9d14590391b9edf84bc14a2`
  - release: `20260924-213157`
- Shared Engine / comparator / Shadow 的 staging lineage 在：
  - `release/dm-shared-engine-staging-20260923`
  - 之後被 `release/dm-central-seat-staging-20260924` 繼承
- `dm/integration-20260920 @ 561bd8c76fec5c0fb86cb9cf4f56734d28367b8a` 是較舊 integration baseline，沒有正式包含後來的 Shared Engine / Central Seat staging lineage。
- 目前主工作目錄有大量 dirty WIP / untracked files，而且 shared engine wheel / lock 已出現 mismatch 風險。
- 目前已部署 Staging 沒有被這批 dirty WIP 汙染。
- forensic report 對 Central Seat 有一個必須再釐清的點：
  - 一處說 staging 已 enforced
  - 一處又描述 assigned/unassigned 還是 shadow observation
  - 因此不能直接假設 authoritative cutover 已完整。

本任務唯一目標：

> 在「完全乾淨、隔離、可丟棄」的 rehearsal worktree 中，重演把可信 Staging lineage 收回 integration branch，確認 Shared Engine、Shadow、wheel/lock、Central Seat、entitlement、platform_admin、assigned/unassigned 全部不退化，再決定是否可以正式 reconcile。

這不是正式 merge 任務。

本輪只做：

CLEAN REHEARSAL
+
DIFF / TEST / BEHAVIOR VERIFICATION
+
GO / NO-GO REPORT

---

# HARD SAFETY RULES

絕對不要：

- 在現有 dirty 主工作目錄直接 merge
- commit 現有 dirty WIP
- reset / clean / stash 現有工作目錄
- cherry-pick 到現有工作目錄
- deploy Staging
- deploy Production
- rollback
- 修改 Staging DB
- 修改 Production DB
- assign / release Seat
- 補 entitlement
- 修改 591 / Sign / Post
- 修改 ExtractionHub source
- 更換 Shared Engine wheel
- 重新 build 一顆新 wheel 當作替代
- 把 dirty lock 視為合法 release evidence

若 rehearsal 需要工作樹：

只能建立新的 temporary worktree。

---

# 已知可信基準

## DM integration baseline

Repo:

F:\00-Ticenpi-SaaS\TicenpiDM

Branch:

dm/integration-20260920

Historical HEAD:

561bd8c76fec5c0fb86cb9cf4f56734d28367b8a

注意：

主 checkout 是 dirty，不能直接操作。

## DM Shared Engine staging lineage

Branch:

release/dm-shared-engine-staging-20260923

Known staging commit:

34a59a7fb50ce5f8b6071e90ab5ee12ca763f3fb

Known source commit in image lineage:

25db5f9b8b65a231dafb7cc8f1f05262fb7519aa

Historical Staging release:

20260923-215914

## DM Central Seat staging lineage

Branch:

release/dm-central-seat-staging-20260924

Trusted commit:

2dd02e2378600f02b9d14590391b9edf84bc14a2

Trusted release:

20260924-213157

## ExtractionHub Shared Engine clean release

Repo:

F:\00-Ticenpi-SaaS\ExtractionHub

Clean source commit:

11762508332bb2508538761302a90f8dca5f2586

Wheel:

ticenpi_shared_engine-0.1.0-py3-none-any.whl

Wheel SHA256:

e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

Shared Engine contract expectations:

- source_clean = true
- semantic contract = 1
- presentation contract = 1
- formatter registry = 1

---

# PHASE 0 — VERIFY FORENSIC BASELINE AGAIN

在任何 rehearsal 前，重新確認：

1. `dm/integration-20260920`
2. `release/dm-shared-engine-staging-20260923`
3. `release/dm-central-seat-staging-20260924`
4. 所有相關 worktree
5. commit containment
6. branch ancestry
7. trusted Staging runtime identity

只讀使用：

- git branch --show-current
- git rev-parse HEAD
- git branch -a --contains
- git worktree list --porcelain
- git log --all --graph --decorate
- git merge-base
- git show
- git diff --stat
- git diff --name-status

確認：

`release/dm-central-seat-staging-20260924`
確實是 Shared Engine staging lineage 的後代。

若不是：

HARD STOP

LINEAGE ASSUMPTION INVALID

---

# PHASE 1 — CREATE CLEAN REHEARSAL WORKTREE

建立新的 disposable worktree，例如：

F:\00-Ticenpi-SaaS\.worktrees\dm-integration-reconcile-rehearsal-20260927

基於：

dm/integration-20260920

規則：

- 不使用現有 dirty checkout
- 不改現有 branch pointer
- 不切換主 checkout branch
- 不刪除既有 worktree
- 不覆蓋任何檔案

建立完成後記錄：

Path:
Branch:
HEAD:
Clean:
YES

如果無法保證 clean：

HARD STOP

REHEARSAL WORKTREE NOT CLEAN

---

# PHASE 2 — REHEARSAL MERGE ONLY

在 rehearsal worktree 中：

模擬將：

release/dm-central-seat-staging-20260924

合入：

dm/integration-20260920

可以使用 temporary rehearsal branch。

不要 push。

不要把 rehearsal merge commit 寫回正式 integration branch。

記錄：

- merge-base
- conflicts
- auto-resolved files
- manual conflict candidates

如果發生 conflict：

不要急著解。

先輸出：

## MERGE CONFLICT MATRIX

| File | Integration Side | Staging Side | Risk Domain | Suggested Resolution Source |

Risk Domain 至少分：

- Shared Engine
- Shadow
- Auth
- Entitlement
- Seat
- Admin
- Frontend
- Deploy
- Tests
- Docs

---

# PHASE 3 — SHARED ENGINE INTEGRITY

確認 rehearsal merge 後仍符合 clean release。

檢查：

- shared engine wheel
- shared_engine_lock.json
- lock source commit
- wheel SHA
- source_clean
- semantic contract
- presentation contract
- formatter registry
- identity endpoint implementation
- DM bridge
- Shadow comparator / recorder / runner

要求：

Wheel SHA256 必須仍為：

e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

Source commit 必須仍為：

11762508332bb2508538761302a90f8dca5f2586

若 rehearsal merge 後出現：

- dirty source
- alternate wheel
- alternate lock
- duplicate vendored engine
- source checkout override

直接：

HARD STOP

SHARED ENGINE INTEGRITY REGRESSION

---

# PHASE 4 — COMPARATOR / SHADOW VERIFICATION

確認以下仍存在且行為一致：

- semantic comparator
- NORMALIZED_MATCH
- canonical layout values
- presentation formatters
- DM final display
- Shadow Console
- Legacy raw
- Core raw
- Canonical
- comparison status
- DM presentation output

特別驗：

Legacy:

3房2廳2衛

Core:

3房(室)2廳2衛

Expected:

NORMALIZED_MATCH

Canonical:

3 / 2 / 2

DM display:

3房2廳2衛

如果 fixture / test 已存在，使用既有 fixture。

不要新增假 live evidence。

---

# PHASE 5 — LEGACY / CORE PRIMARY SAFETY

必須證明：

Legacy user-visible primary = YES

Extraction Core primary = NO

Shared Engine / Core / Shadow failure
不得阻斷 Legacy user response。

確認最近 staging lineage 沒有：

- 把 Core 切成 primary
- 刪除 Legacy scraper
- 讓 Shadow exception 冒泡到 user response

若任何一項發生：

HARD STOP

LEGACY PRIMARY REGRESSION

---

# PHASE 6 — CENTRAL SEAT AUTHORITATIVE STATUS

這一階段非常重要。

不要只看檔名或 test 名稱。

請追 rehearsal merge 後 `GET /api/designs` 的實際 dependency chain：

Bearer
→ current user
→ platform role
→ commercial context
→ DM entitlement
→ business account?
→ Seat decision
→ allow / deny

回答：

1. Seat 現在是：
   - SHADOW_ONLY
   - AUTHORITATIVE
   - MIXED
   - UNKNOWN

2. business user 有 active DM entitlement 但 unassigned Seat：
   - 是否真的 deny？

3. business user 有 active DM entitlement + assigned Seat：
   - 是否真的 allow？

4. individual customer：
   - 是否依 contract 不需要 Seat？

5. platform_admin：
   - 是否仍依中央 contract exemption？

6. Seat failure 是否被錯誤降級成 observation-only？

建立：

## CENTRAL SEAT DECISION TABLE

| Account Type | Entitlement | Seat | Platform Role | Expected | Actual |

至少：

- business / entitlement yes / assigned / user
- business / entitlement yes / unassigned / user
- business / entitlement no / N/A / user
- individual / entitlement yes / N/A / user
- platform_admin

若目前 staging code 仍只有 observe / shadow：

明確寫：

SEAT AUTHORITATIVE CUTOVER = NOT COMPLETE

不要宣稱已完成。

---

# PHASE 7 — PLATFORM_ADMIN SAFETY

確認：

- platform_admin exemption path
- local-dev-only admin bypass
- DM_REQUIRE_AUTH
- environment gating

特別驗：

`LOCAL_DEV_PLATFORM_ADMIN`

只能在：

- local
- DM_REQUIRE_AUTH=0
- explicit local flag

成立。

不得在：

- Staging
- Production
- DM_REQUIRE_AUTH=1

生效。

若 rehearsal merge 使 bypass 可進 Staging：

HARD STOP

PLATFORM_ADMIN BYPASS LEAK

---

# PHASE 8 — ENTITLEMENT SAFETY

確認：

- `PRODUCT_ENTITLEMENT_REQUIRED` behavior
- `my_commercial_context()`
- product code = dm
- business customer entitlement
- individual entitlement
- inactive entitlement
- missing entitlement

不要修改任何 live data。

所有 behavior 只透過：

- existing unit tests
- disposable local test DB
- fixtures

驗證。

---

# PHASE 9 — TESTS

允許平行執行 read-only test tracks：

T1 — Shared Engine / semantics / presentation

T2 — Shadow / Legacy/Core

T3 — Auth / entitlement / platform_admin

T4 — Central Seat assigned/unassigned

T5 — Frontend build / UI tests

T6 — Cookie isolation / tenant regression

規則：

測試 agent 不准修改 source。

任何 fail：

回報 single planner。

至少跑：

- DM backend relevant full suite
- Shared Engine distribution tests
- semantic/presentation tests
- Shadow tests
- auth tests
- entitlement tests
- Seat tests
- platform_admin tests
- frontend Vitest
- frontend build
- git diff --check

若 repository 有正式 gate：

跑對應 gate。

---

# PHASE 10 — COMPARE REHEARSAL TO TRUSTED STAGING

比較：

Rehearsal merge result

vs

release/dm-central-seat-staging-20260924 @ 2dd02e2

目標不是要求 bit-for-bit 完全相同，而是確認：

- Staging capability 全部保留
- Integration branch 既有後續合法改動沒有被誤刪
- 不帶入 dirty wheel / lock
- 不帶入不相關 WIP
- 不退化 auth / entitlement / Seat
- 不退化 Shared Engine / Shadow

建立：

## RECONCILIATION DELTA

| Domain | Trusted Staging | Rehearsal Integration | Difference | Safe? |

Domains：

- Shared Engine
- Shadow
- Legacy primary
- Auth
- Entitlement
- Seat
- platform_admin
- Frontend
- Deploy
- Tests

---

# PHASE 11 — DO NOT FORMALLY MERGE YET

本任務即使全部 PASS：

也不要直接 push 正式 merge。

只輸出：

GO

或：

NO-GO

如果 GO：

給出下一個正式任務需要的：

- source branch
- target branch
- exact verified commits
- expected merge strategy
- expected conflicts
- required post-merge gates

如果 NO-GO：

列出 blocker。

---

# FINAL REPORT FORMAT

# DM STAGING → INTEGRATION CLEAN RECONCILIATION REHEARSAL REPORT

## 1. Baseline

Integration:
Shared Engine staging:
Central Seat staging:
Trusted Staging release:
Current dirty checkout untouched:
YES / NO

## 2. Rehearsal Worktree

Path:
Branch:
Base HEAD:
Clean:
YES / NO

## 3. Lineage

Merge base:
Shared Engine ancestry:
Central Seat ancestry:
Lineage valid:
YES / NO

## 4. Merge Rehearsal

Conflicts:
Files:
Manual resolutions:
Formal integration branch modified:
NO

## 5. Shared Engine Integrity

Wheel SHA:
Expected:
Source commit:
source_clean:
Semantic:
Presentation:
Formatter registry:
Identity implementation:

Result:
PASS / FAIL

## 6. Comparator / Shadow

Semantic comparator:
NORMALIZED_MATCH:
Canonical:
Presentation:
Shadow Console:
DM final display:

Result:
PASS / FAIL

## 7. Legacy Primary

Legacy primary:
YES / NO

Core primary:
NO / YES

Shadow failure isolated:
YES / NO

## 8. Central Seat

Status:
SHADOW_ONLY / AUTHORITATIVE / MIXED / UNKNOWN

Business assigned:
ALLOW / DENY / NOT_TESTED

Business unassigned:
ALLOW / DENY / NOT_TESTED

Individual:
ALLOW / DENY / NOT_TESTED

platform_admin:
ALLOW / DENY / NOT_TESTED

Result:
PASS / FAIL / INCOMPLETE

## 9. Auth / Entitlement

Auth required:
platform_admin exemption:
DM entitlement:
Missing entitlement:
Inactive entitlement:
Local bypass isolation:

Result:
PASS / FAIL

## 10. Tests

Shared Engine:
Shadow:
Auth:
Entitlement:
Seat:
platform_admin:
Frontend:
Build:
Cookie/Tenant:
diff-check:

## 11. Reconciliation Delta

| Domain | Trusted Staging | Rehearsal | Difference | Safe |

## 12. Regression Status

Shared Engine:
NO_REGRESSION / REGRESSION

Central Seat:
NO_REGRESSION / REGRESSION / INCOMPLETE

Auth:
NO_REGRESSION / REGRESSION

Legacy/Core:
NO_REGRESSION / REGRESSION

## 13. Safe Formal Merge Plan

Source branch:
Target branch:
Source commit:
Target baseline:
Expected conflicts:
Required gates:

Do not execute:
YES

## 14. Isolation

Dirty main checkout:
UNCHANGED

Staging:
UNCHANGED

Staging DB:
UNCHANGED

Production:
UNCHANGED

Production DB:
UNCHANGED

ExtractionHub:
UNCHANGED

Other products:
UNCHANGED

## 15. Final Verdict

只能：

RECONCILIATION REHEARSAL GO

或

RECONCILIATION REHEARSAL NO-GO

---

# HARD STOP

本任務不是正式 merge。

即使 GO：

不要 push
不要 merge 正式 branch
不要 deploy
不要改 DB

先交完整 rehearsal evidence，等使用者確認後再做正式 reconciliation。
