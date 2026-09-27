# EXTRACTIONHUB CORE CLEAN RELEASE → DM STAGING FULL SHADOW CLOSEOUT OWNER

## 任務目標

直接把目前卡住的 ExtractionHub Core / YUCT adapter 正式做成可被 DM Staging 使用的乾淨交付物，接著完成 DM pin、CI、ARM64 Docker、Staging deploy、Shadow 啟用、Shadow Console、真實 platform_admin、YUCT 真人 UI、Legacy/Core Comparator、Central Seat E2E，最後做到 Staging closeout。

本任務允許：

- 修改 ExtractionHub
- 修改 DM
- 建立 clean commit
- 建立 hosted CI artifact
- 建立 ARM64 image
- 更新 DM pin / lock
- 更新 Staging manifest/config
- 部署 DM Staging
- 在 Staging 啟用 Shadow
- 進行 Staging 測試資料的正式 admin RPC 操作
- 執行 rollback → restore drill

本任務禁止：

- 修改 Production
- 修改 Production DB
- 部署 Production
- 修改 Post / 591 / Sign / ORC
- 使用 dirty source 直接 build release artifact
- hot patch container
- 直接把 localhost:8020 當成 Staging runtime
- 切 Core primary
- 刪除 Legacy scraper
- 要求使用者貼 JWT / Cookie / Password

最終目標：

> 讓 DM Staging 真正可以在同一次 YUCT 真人操作中：
>
> Legacy primary → 使用者結果
>
> 同時 background Core → Canonical → Semantic Comparator → Presentation → Shadow run
>
> 並可從真正的 /admin/extraction-shadow UI 查看比較結果，完成 Central Seat 真人驗證與 rollback/restore。

---

# 已知目前狀態

## DM Staging

目前已知 runtime：

Release:
20260927-121024

Commit:
184aab64e649092ba602a7dc4f386489456439a0

Backend digest:
sha256:d36d030355764c26036c6c5fb51d378562c8cd00b78e6c3434dc29c907010192

Frontend digest:
sha256:50063e1153d50604187c70afd4f201963ffd9133294721b4a2f0941095f8284a

Health:
PASS

Seat policy:
require

Shadow:
enabled=false

Supabase Staging project:
jlsqjvehwblkeuycjoyj

## Shared Engine clean release

ExtractionHub clean source:

11762508332bb2508538761302a90f8dca5f2586

Current clean wheel:

ticenpi_shared_engine-0.1.0-py3-none-any.whl

Current clean wheel SHA256:

e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

Contracts:

semantic = 1
presentation = 1
formatter registry = 1
source_clean = true

但目前正式 clean wheel 只有 semantic / presentation 交付，不含可供 DM Staging 載入的 Extraction Core / YUCT adapter。

## DM Shadow Console candidate

已有候選修正 commit：

07b1a499269756b422fa4ed2b845f94143115027

CI：

36295904567

Frontend Vitest:
134 / 134 PASS

Build:
PASS

ARM64 backend image:
sha256:dded4a4a463dd44a923f1b56e26fec0409e9005d183f1ca5ae345cea1a192e84

ARM64 frontend image:
sha256:a470f73ec326d77988171c9c5b15fbdc9f00c70412fa092f84b31eb261c48c9c

這組 candidate 尚未部署。

---

# 核心問題

目前 DM Staging 的 Shadow provider 想執行 Extraction Core，但現行 bridge / provider 仍依賴 local source 路徑。

也就是目前有：

```
ExtractionHub local source
↓
本機可跑 Core
```

但 Staging Docker 裡沒有：

```
F:\00-Ticenpi-SaaS\ExtractionHub
```

所以 Staging 無法正式載入 Core。

要改成：

```
ExtractionHub clean source
↓
正式 package artifact
↓
DM pinned dependency
↓
DM Docker
↓
Staging
```

---

# 執行策略

允許平行，但只允許：

## READ-ONLY PARALLEL AUDITS

TRACK A — ExtractionHub package / Core boundary
TRACK B — DM Shadow provider contract
TRACK C — DM Staging deployment/runtime config
TRACK D — Central Seat / auth E2E prerequisites

四個 Track 完成後：

```
ROOT CAUSE / CONTRACT SYNTHESIS
↓
SINGLE WRITER ExtractionHub
↓
Hosted CI artifact
↓
SINGLE WRITER DM pin/integration
↓
CI / ARM64
↓
STAGING DEPLOY
↓
LIVE E2E
↓
ROLLBACK / RESTORE
```

禁止多個 writer 同時改相同 repo。

---

# 工作目錄

ExtractionHub:

F:\00-Ticenpi-SaaS\ExtractionHub

DM:

F:\00-Ticenpi-SaaS\TicenpiDM

可信 clean worktree / release worktree可使用，但不得污染現有 dirty checkout。

如需修改：

一律建立新的 clean worktree。

---

# PHASE 0 — PREFLIGHT

## ExtractionHub

記錄：

- branch
- HEAD
- git status
- worktree list
- dirty files
- clean release ancestry
- package metadata
- wheel build config
- hosted CI workflow

## DM

記錄：

- integration branch
- HEAD
- current deployed Staging commit
- Shadow Console candidate branch/commit
- worktree list
- dirty checkout status

禁止：

- reset --hard
- clean
- stash
- force push

如果發現 active writer 正在改同一批檔案：

HARD STOP

```
WRITE COLLISION
```

---

# TRACK A — EXTRACTIONHUB CORE PACKAGE AUDIT

只讀確認目前 ExtractionHub 的 Core 結構。

至少找：

- run_extract
- extraction request contract
- extraction context contract
- adapter registry
- YUCT adapter
- transport
- canonical model
- diagnostics
- semantic / presentation package
- packaging include/exclude rules

目標是回答：

1. YUCT adapter 已存在在哪裡？
2. Core entrypoint 是什麼？
3. DM provider 期待的 contract 是不是：
   `run_extract(request, context=context)`
4. 現有 wheel 為什麼沒有 Core？
5. 是 packaging exclude 還是 Shared Engine wheel 本來刻意只包 semantic/presentation？
6. 最小安全方案是：
   - 擴充現有 wheel
   - 或建立同 repo 的正式 Core wheel
   - 或其他既有 package target

不要先假設。

原則：

> 不要為同一份 Core 建第二套 source-of-truth。

---

# TRACK B — DM SHADOW PROVIDER CONTRACT

只讀追：

DM Shadow
→ provider.py
→ shared_engine.py
→ Core invocation

確認：

- provider 目前怎麼找 Core
- local-only bridge限制
- request schema
- context schema
- expected return object
- error behavior
- fallback behavior
- Legacy isolation
- Shadow recorder contract

建立：

## DM ↔ CORE CONTRACT

Input:
Output:
Error:
Timeout:
Diagnostics:
Canonical:
Raw preservation:

這份 contract 必須由現有 code 證明。

---

# TRACK C — STAGING DEPLOY CONTRACT

只讀確認：

- Shadow enable env
- sample rate
- provider selection
- DM_SEAT_POLICY
- runtime env source
- compose / manifest
- Docker build
- ARM64 workflow
- deploy gates
- rollback tooling

建立：

## STAGING CONFIG

Shadow enabled:
Sample rate:
Core provider:
Seat policy:
Auth required:

---

# TRACK D — CENTRAL SEAT / AUTH PREREQUISITES

只讀確認 live Staging 有可安全測的：

- platform_admin
- business customer
- active members
- DM entitlement
- assigned Seat
- unassigned member

如果缺測試資料：

規劃正式 admin RPC 操作。

不要直接改 table。

---

# PHASE 1 — DEFINE CLEAN CORE RELEASE CONTRACT

根據 Track A/B 結果，決定正式交付物。

優先順序：

1. 優先沿用現有 ExtractionHub package / wheel architecture
2. 不建立 duplicate engine
3. 不手動 copy source 進 DM
4. 不依賴 localhost source path
5. artifact 必須可由 hosted CI重現
6. artifact 必須可 immutable pin

正式 Core artifact 必須至少包含：

- YUCT adapter
- adapter registry
- run_extract
- request/context contract
- canonical model
- diagnostics
- semantic
- presentation

如果 semantic/presentation 仍維持原 wheel，而 Core 用另一正式 package：

必須清楚定義：

- package names
- versions
- compatibility
- lock identity
- DM pin

不要產生兩份 semantic implementation。

---

# PHASE 2 — EXTRACTIONHUB SINGLE-WRITER IMPLEMENTATION

建立 clean worktree。

實作：

- Core package inclusion
- YUCT adapter inclusion
- public entrypoint
- stable DM-compatible contract
- identity metadata
- package versioning
- deterministic build

要求：

```
run_extract(request, context=context)
```

或 DM 現行已證明的等價正式 contract。

保留：

- raw
- canonical
- diagnostics
- field state
- typed error

不得：

- 把 Legacy DM scraper搬進 ExtractionHub
- 改 Legacy primary
- hardcode DM-specific UI formatting進 Core
- 使用本機絕對路徑

---

# PHASE 3 — EXTRACTIONHUB TESTS

至少：

## Core

- registry
- YUCT adapter
- request/context
- canonical output
- diagnostics
- typed errors

## YUCT

- existing fixtures
- golden
- image behavior if contract requires

## Semantic

- EXACT_MATCH
- NORMALIZED_MATCH
- MISMATCH
- UNCOMPARABLE

## Presentation

- DM compact layout
- existing formatters

## Packaging

安裝正式 wheel 到全新 disposable venv/container。

禁止 source checkout fallback。

測：

```
import package
run_extract(...)
semantic
presentation
```

如果 package install後仍偷偷 import repo source：

FAIL。

---

# PHASE 4 — CLEAN COMMIT + HOSTED CI ARTIFACT

只有所有本機測試 PASS：

- clean commit
- push feature/release branch
- hosted CI
- build artifact
- hosted install test
- identity artifact

記錄：

- source commit
- package version
- artifact filename
- SHA256
- source_clean
- CI run
- contract versions

Hosted CI 未 PASS：

不得進 DM。

---

# PHASE 5 — DM PIN NEW CORE ARTIFACT

建立新的 clean DM worktree，基於目前最新 integration。

不要碰 dirty main checkout。

整合：

- 新 Core artifact
- existing Shared Engine semantic/presentation identity
- Shadow provider

要求：

DM 不再：

- 掃 local ExtractionHub path
- import F:\...
- 依賴 localhost 8020

DM 必須：

```
installed immutable package
→ provider
→ run_extract
```

更新：

- lock
- identity
- Docker build dependency
- tests

如果 Core 和 semantic/presentation 分兩個 package：

lock 必須同時 pin兩個 artifact identity。

---

# PHASE 6 — INTEGRATE SHADOW CONSOLE CANDIDATE

確認 commit：

07b1a499269756b422fa4ed2b845f94143115027

是否已在目前 integration lineage。

如果尚未：

把它以最安全方式整合到 clean candidate。

不得直接 cherry-pick 若 ancestry已包含等價改動。

先 diff / merge-base。

Shadow Console 必須：

`/admin/extraction-shadow`

platform_admin only。

至少顯示：

- runs
- run detail
- Legacy raw
- Core raw（若目前 recorder缺，補正式 snapshot）
- Canonical
- comparison status
- DM final display
- field-level status

若 backend目前只存 Core Canonical，沒有 Core raw：

本任務允許補：

`core_raw_snapshot`

但必須：

- schema/versioned
- no secrets
- migration safe
- test coverage

---

# PHASE 7 — DM TESTS

至少：

## Backend

- Core provider
- package identity
- Shadow
- Legacy isolation
- auth
- entitlement
- Central Seat
- platform_admin
- admin APIs

## Frontend

- Shadow Console
- admin gate
- runs
- detail
- YUCT import
- full Vitest
- build

## Full gates

- dm-gates-full
- cookie isolation
- tenant isolation
- git diff --check

全部 PASS 才可 build images。

---

# PHASE 8 — HOSTED DM CI + ARM64

產出：

- backend ARM64 image
- frontend ARM64 image
- immutable digests
- CI run IDs
- source commit
- artifact provenance

不得 local-only image deploy。

---

# PHASE 9 — STAGING MANIFEST / CONFIG

準備新 Staging candidate。

要求：

```
DM_REQUIRE_AUTH=1
DM_SEAT_POLICY=require
Shadow enabled=true
Shadow sample rate=100% for controlled Staging validation
```

如果現有 config nomenclature不同，以 repo實際 contract為準。

Shadow 100% 只限 Staging controlled validation。

Production 不動。

更新 Staging manifest：

- backend digest
- frontend digest
- release
- commit
- env/config

先跑 deploy preflight。

若 unrelated manifest drift：

HARD STOP

不要覆蓋其他產品。

---

# PHASE 10 — DEPLOY DM STAGING

走正式 deploy tooling。

要求：

- service = dmruntimestaging
- isolated deploy
- immutable digests
- health
- release header
- runtime identity
- post-deploy audit

部署後：

輸出：

```
STAGING_RELEASE=
STAGING_COMMIT=
BACKEND_DIGEST=
FRONTEND_DIGEST=
HEALTH=PASS
```

---

# PHASE 11 — REAL PLATFORM_ADMIN LOGIN

使用正常 Google / Supabase login。

不要：

- localhost bypass
- copied token
- manual Bearer
- user-supplied token

驗：

```
/api/admin/shared-engine/identity
→ 200
```

並核對：

- source_clean
- source commit
- package versions
- package SHA
- semantic
- presentation
- formatter
- Core identity/version if separate artifact

Shadow admin APIs：

- report → 200
- runs → 200

---

# PHASE 12 — CENTRAL SEAT LIVE E2E

因 Staging policy=require：

必須實測。

## Business Member A

active DM entitlement
assigned Seat

Expected:

`/api/designs → 200`

## Business Member B

same business customer
active membership
no Seat

Expected:

Seat-related deny

不能是 entitlement missing。

## Individual

如果已有安全測試 account：

active entitlement
Seat not required

Expected allow。

## platform_admin

Expected allow / exemption。

若資料缺：

只能用正式 admin RPC。

允許：

- admin_set_entitlement
- assign/release product seat

若 repo實際名稱不同，以正式 contract為準。

每筆變更記：

before
operation
after
rollback

Production 不動。

---

# PHASE 13 — YUCT HUMAN UI

真正從：

https://dm-staging.ticenpi.com/

登入一般有權限帳號。

在 DM UI：

- 貼合法公開 YUCT URL
- 正常提交
- 等 Legacy user-visible result

要求：

Legacy user-visible result = PASS

不能：

- 用 fixture API代替
- 用 8020代替
- 用 backend curl代替 UI

---

# PHASE 14 — LIVE SHADOW / COMPARATOR

YUCT 真人 run後：

進：

https://dm-staging.ticenpi.com/admin/extraction-shadow

找同一次 run。

至少驗：

- Legacy raw
- Core raw
- Canonical
- comparison status
- DM final display

要看到真實 live Core run。

如果案例剛好：

Legacy:
3房2廳2衛

Core:
3房(室)2廳2衛

Expected:

NORMALIZED_MATCH

Canonical:
3 / 2 / 2

DM final:
3房2廳2衛

若真人案例不同：

驗實際欄位，不偽造。

---

# PHASE 15 — LEGACY PRIMARY SAFETY

必須證明：

Legacy primary = YES

Core primary = NO

Shadow failure不得影響 user response。

使用：

- live architecture evidence
- existing forced failure regression
- 如果有安全 Staging failure injection，再做 controlled validation

不要讓真實使用者流程壞掉。

---

# PHASE 16 — FULL STAGING GATES

部署後跑：

- health
- dm-gates-full
- auth
- entitlement
- Seat
- platform_admin
- Shared Engine identity
- Core identity
- Shadow
- Legacy regression
- frontend
- build provenance
- tenant isolation
- cookie isolation

全部 PASS 才能 rollback。

---

# PHASE 17 — ROLLBACK TARGET VALIDATION

從 deployment history找 immediate valid previous release。

不要硬寫舊值。

確認：

- manifest snapshot
- backend digest
- frontend digest
- image availability
- rollback tooling
- service isolation

輸出：

```
CURRENT_RELEASE=
ROLLBACK_TARGET=
ROLLBACK_SAFE=YES
```

---

# PHASE 18 — ACTUAL ROLLBACK → RESTORE

執行正式：

```
current candidate
↓
rollback
↓
previous release
↓
health PASS
↓
basic DM load
↓
restore candidate
↓
health PASS
↓
runtime identity
↓
Shared Engine/Core identity
↓
Shadow Console
↓
basic YUCT smoke
```

要求：

Production不動。

其他產品不動。

如果 rollback失敗：

停止。

不要自行猜修復。

---

# PHASE 19 — FINAL RESTORE VALIDATION

restore後至少重新驗：

- release
- commit
- backend digest
- frontend digest
- health
- Seat policy=require
- Shadow enabled
- platform_admin identity
- Shadow Console
- one basic YUCT UI smoke
- Shadow run exists

---

# FINAL REPORT FORMAT

# EXTRACTIONHUB CORE → DM STAGING FULL CLOSEOUT REPORT

## 1. ExtractionHub Core Release

Source commit:
Package:
Version:
SHA256:
CI:
source_clean:

YUCT adapter included:
YES / NO

run_extract contract:
PASS / FAIL

## 2. DM Candidate

Commit:
Core pin:
Shared Engine pin:
Shadow Console:
Core raw snapshot:

## 3. DM CI / Images

CI:
Backend digest:
Frontend digest:
ARM64:
PASS / FAIL

## 4. Staging Deploy

Release:
Commit:
Health:
Seat policy:
Shadow enabled:

## 5. platform_admin

Real login:
Identity API:
Shadow APIs:

Result:

## 6. Central Seat

Business assigned:
Business unassigned:
Individual:
platform_admin:

Result:

## 7. YUCT Human UI

URL:
Legacy result:
Core shadow:
Run id:

Result:

## 8. Comparator

Legacy raw:
Core raw:
Canonical:
Status:
DM final:

Result:

## 9. Shadow Console

URL:
Runs:
Detail:
Field states:
Auth gate:

Result:

## 10. Regression

Backend:
Frontend:
dm-gates-full:
Auth:
Entitlement:
Seat:
Cookie:
Tenant:
Legacy isolation:

## 11. Rollback

Candidate:
Target:
Rollback:
Health:
Restore:
Restored release:
Identity after restore:

Result:

## 12. Production Isolation

Production:
UNCHANGED

Production DB:
UNCHANGED

Other products:
UNCHANGED

## 13. Remaining Items

若無：

NONE

## 14. Final Result

只能：

EXTRACTIONHUB CORE + DM STAGING CLOSEOUT READY

或

EXTRACTIONHUB CORE + DM STAGING CLOSEOUT BLOCKED

---

# HARD STOP

不要：

- 用 localhost:8020 當 Staging
- 用 dirty ExtractionHub main build正式 package
- 直接 copy source進 DM container
- hot patch
- 改 Production
- 改 Production DB
- 改其他產品
- 切 Core primary
- 刪 Legacy scraper
- 要求使用者貼 credential

本任務要一路做到 Staging，完成 live Shadow / Comparator / Seat / rollback closeout，或停在唯一可證明 blocker。
