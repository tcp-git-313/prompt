# TICENPI SHARED ENGINE CLEAN RELEASE + DM STAGING EVIDENCE OWNER

## 任務目標

接續目前已完成的 Shared Engine Distribution + Packaging 工作。

目前已知：

- Shared Semantic / Presentation Engine 已完成。
- ExtractionHub wheel builder / reproducibility / identity 已完成。
- DM 已能在本機 Docker image 內載入 pinned wheel。
- Post / 591 cross-language consumer contract 已完成，尚未接入。
- Shared Engine source 目前來自 dirty ExtractionHub working tree。
- 上一輪 wheel identity：
  - semantic contract = 1
  - presentation contract = 1
  - formatter registry = 1
  - wheel version = 0.1.0
- 上一輪 local wheel / Docker candidate 僅代表本機驗證，不是正式 release evidence。
- 尚缺：
  1. clean source commit
  2. hosted CI wheel artifact
  3. DM pin clean CI artifact
  4. hosted DM ARM64 image
  5. DM Staging deploy
  6. live Staging shared-engine identity
  7. auth-protected Legacy scrape regression
  8. Staging rollback drill / rollback evidence

本任務唯一目標：

> 建立完整、可重查、可回滾的 Shared Engine release evidence chain，並在 DM Staging 證明實際 runtime 使用該 clean immutable artifact。

正式證據鏈必須是：

Clean ExtractionHub commit
↓
Hosted CI
↓
Immutable wheel artifact + SHA256
↓
DM lock/vendor pin
↓
Hosted DM ARM64 image
↓
DM image digest
↓
Staging deploy
↓
Live /api/admin/shared-engine/identity
↓
Legacy scrape regression
↓
Rollback drill
↓
READY

本任務不要：

- 修改 Post
- 修改 591
- 開始永慶 / 台灣房屋
- 修改 Production
- 修改 Production DB
- 刪除 Legacy scraper
- 切 Core Primary
- 發布 Source Registry

---

# 重要安全原則

## Dirty WIP 不得被破壞

目前 ExtractionHub 與 DM 都有既有 dirty WIP。

禁止：

git reset --hard
git clean
git stash
git rebase
強制 checkout 覆蓋檔案
修改或刪除無法確認 ownership 的 WIP

如果 Shared Engine changes 與其他既有 WIP 混在同一檔案中：

必須逐段判斷 ownership。

如果無法安全拆分：

HARD STOP

輸出：

CLEAN RELEASE OWNERSHIP AMBIGUOUS

不要為了做 clean commit 破壞其他工作。

---

# 平行策略

本任務允許有限度平行：

## TRACK A — ExtractionHub Clean Release

可獨立進行：

- Shared Engine change inventory
- clean release reconstruction
- clean commit
- hosted CI wheel artifact

## TRACK B — DM Legacy Auth Regression

可與 Track A 平行，但只能修改 / 測試：

- DM auth-protected legacy scrape regression
- test harness / fixture
- 不碰 shared engine wheel lock/vendor
- 不碰 Docker packaging
- 不碰 shared_engine.py packaging identity contract

兩 Track 不得修改同一 repo 同一檔案。

當 Track A 產出 hosted CI wheel 後：

停止平行。

後續必須串行：

Track A clean wheel
↓
DM pin
↓
DM hosted ARM64 build
↓
Staging deploy
↓
identity
↓
rollback

---

# 工作目錄

ExtractionHub：

F:\00-Ticenpi-SaaS\ExtractionHub

DM：

F:\00-Ticenpi-SaaS\TicenpiDM

只讀：

F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591
F:\00-Ticenpi-SaaS\ticenpi-platform
F:\00-Ticenpi-SaaS\deploy

實際 repo path 以現場為準。

---

# PHASE 0 — GLOBAL PREFLIGHT

對 ExtractionHub / DM 記錄：

- branch
- HEAD
- git status
- git diff
- git diff --cached
- untracked files
- active writer evidence
- current CI status
- remotes
- current release docs

確認：

- 沒有 staged changes
- 沒有 active writer 修改本任務 target files
- Post / 591 保持 read-only
- Production 不在 scope

輸出：

## GLOBAL RELEASE PREFLIGHT

| Repo | Branch | HEAD | Dirty | Active Writer | Safe |

如有 collision：

HARD STOP

---

# PHASE 1 — SHARED ENGINE CHANGE INVENTORY

在 ExtractionHub 中找出上一輪 Shared Engine Distribution 實際新增 / 修改的檔案。

至少應包含或核對：

- semantics.py
- presentation.py
- package builder / wheel builder
- package metadata
- reproducibility tests
- consumer JSON contract
- ADR-001
- CI workflow
- current-state / history
- package identity tests

不要憑提示詞假設檔名。

建立：

## SHARED ENGINE RELEASE FILESET

每個檔案標記：

OWNED_BY_SHARED_ENGINE
UNRELATED_EXISTING_WIP
MIXED
UNKNOWN

只有：

OWNED_BY_SHARED_ENGINE

可以直接進 release。

MIXED：

必須逐 hunk 拆分。

UNKNOWN：

HARD STOP 或先取得證據。

---

# PHASE 2 — RECONSTRUCT CLEAN EXTRACTIONHUB RELEASE

目標：

建立一個 clean source state，使：

git status
→ clean

且該 clean state 只包含本次 Shared Engine release 必要變更與其必要依賴。

優先安全策略：

1. 使用 fresh worktree / release branch 從正確 base commit 建立隔離工作區。
2. 將已確認 owned 的 Shared Engine changes 精準帶入。
3. 不動原 dirty checkout。
4. 在隔離 worktree 跑完整測試。
5. 只有證據完整才 commit。

禁止：

- 在原 dirty checkout reset
- stash 原 WIP
- commit 無關改動
- wholesale copy 整個 working tree
- 把 unknown untracked files 一起提交

若 Shared Engine changes 依賴尚未 commit 的 Core changes：

必須明確列出依賴。

若這些依賴本身屬同一 Extraction Platform 必要 release：

可納入，但必須：
- 說明理由
- 測試
- 列入 release fileset

若依賴 ownership 不明：

HARD STOP。

---

# PHASE 3 — CLEAN COMMIT IDENTITY

完成 clean release commit 後必須證明：

git status --porcelain
→ empty

記錄：

CLEAN_SOURCE_COMMIT
CLEAN_SOURCE_BRANCH
TREE_SHA if available

Shared Engine runtime identity 必須支援：

source_clean = true
source_commit = exact clean commit

如果 source_clean = false：

禁止 build release artifact。

---

# PHASE 4 — EXTRACTIONHUB FULL GATE

在 clean worktree 執行：

1. Shared Engine unit tests
2. Semantic tests
3. Presentation tests
4. wheel reproducibility
5. package identity tests
6. consumer contract tests
7. Canonical tests
8. Transport tests
9. Diagnostics tests
10. Images tests
11. YUCT regression
12. CTHouse regression
13. HBHousing regression
14. Full pytest
15. git diff --check

要求：

0 failures

已知 warnings 可記錄，但不得新增 unexplained failures。

只有全部 PASS：

允許 push clean release branch / commit。

---

# PHASE 5 — HOSTED CI WHEEL

Push 後使用 hosted CI。

不要使用 local wheel 取代 hosted artifact。

Hosted CI 必須產出：

- wheel filename
- package version
- source commit
- semantic contract version
- presentation contract version
- formatter registry version
- wheel SHA256
- build identity
- CI run id
- artifact id / downloadable artifact evidence

要求：

CI source commit
=
CLEAN_SOURCE_COMMIT

CI 必須 PASS。

如果 hosted CI 無法執行：

BLOCKED

不能用 local build 假裝。

---

# PHASE 6 — VERIFY HOSTED ARTIFACT

下載 hosted CI artifact 到隔離 temp 位置。

驗證：

- SHA256
- wheel metadata
- package version
- source commit metadata
- import
- semantic contract
- presentation contract
- formatter registry

重新執行最小 smoke：

3房2廳2衛
vs
3房(室)2廳2衛
→ NORMALIZED_MATCH

presentation：

compact
→ 3房2廳2衛

slash
→ 3/2/2

若 artifact identity 不符：

HARD STOP。

---

# PHASE 7 — TRACK B: DM LEGACY AUTH REGRESSION

此 Track 可在 Phase 1–6 同時進行，但不能碰 wheel pin / Docker packaging files。

目標：

解決上一輪尚未完成的 auth-protected Legacy scrape regression。

先找真實 auth contract：

- route auth dependency
- platform_admin behavior
- normal entitled user behavior
- unauthenticated behavior
- staging test identity / fixture strategy
- existing test token / JWT helper

不要把舊「anonymous /api/scrape 應成功」測試硬改成 200。

必須先確認產品現在正確 contract。

如果 route 正確應 auth-protected：

更新測試為正確身份。

驗至少：

Unauthenticated
→ expected deny

Authorized / entitled user
→ Legacy scrape path executes

platform_admin
→ according to current product contract

Shadow enabled/disabled
→ Legacy user-visible result 不變

Core Shadow failure
→ Legacy result 仍可回傳

完成後輸出：

DM LEGACY AUTH REGRESSION PASS

若缺合法 test identity：

BLOCKED
並說明缺什麼。

---

# PHASE 8 — DM PIN CLEAN CI ARTIFACT

只有 Hosted CI wheel 驗證 PASS 後才修改 DM packaging。

更新：

- wheel vendor / artifact
- lock
- expected SHA256
- expected source commit
- expected contract versions
- build identity

移除上一輪 dirty-source artifact reference。

要求：

DM lock 中的：

source_commit
=
CLEAN_SOURCE_COMMIT

wheel_sha256
=
HOSTED_CI_WHEEL_SHA256

source_clean
=
true

不要接受 local candidate wheel。

---

# PHASE 9 — DM LOCAL REGRESSION WITH CLEAN ARTIFACT

跑：

- shared engine identity tests
- semantic compare
- presentation profile
- admin identity API
- auth / RLS
- Shadow tests
- Legacy auth regression
- frontend Shadow Console tests
- relevant backend tests
- git diff --check

必要案例：

Legacy：
3房2廳2衛

Core：
3房(室)2廳2衛

Status：
NORMALIZED_MATCH

Canonical：
3/2/2

DM compact：
3房2廳2衛

DM slash：
3/2/2

Comparator unchanged：
PASS

---

# PHASE 10 — DM DOCKER BUILD

使用 clean hosted artifact 建 DM image。

要求：

- image 不依賴 host ExtractionHub path
- Docker build 先驗 wheel SHA
- installed Shared Engine identity 可查
- runtime source_clean = true
- runtime source_commit = CLEAN_SOURCE_COMMIT

本地 image smoke：

GET /api/admin/shared-engine/identity

使用合法 platform_admin 測試身份。

回傳至少：

semantic_contract_version
presentation_contract_version
formatter_registry_version
source_commit
source_clean
wheel_version
wheel_sha256
build_identity
loaded_from

預期：

source_clean = true

---

# PHASE 11 — HOSTED DM CI ARM64 BUILD

只有 local clean-artifact Docker PASS 後才 push DM release candidate。

Hosted DM CI 必須：

- build target ARM64 image
- use exact pinned wheel
- pass tests
- publish immutable image
- output image digest
- record DM source commit
- record Shared Engine source commit
- record wheel SHA

要求：

HOSTED_DM_IMAGE_DIGEST

不是 local digest。

如果 hosted build 未跑：

不能進 Staging deploy。

---

# PHASE 12 — PRE-STAGING RELEASE EVIDENCE

組合 release identity：

DM_SOURCE_COMMIT
SHARED_ENGINE_SOURCE_COMMIT
SHARED_ENGINE_WHEEL_VERSION
SHARED_ENGINE_WHEEL_SHA256
DM_IMAGE_DIGEST
CI_RUN_ID

全部必須 immutable / exact。

確認：

Production unchanged。

確認：

Staging target 是正確隔離 service。

不得部署到 Production service。

---

# PHASE 13 — STAGING DEPLOY

部署 DM Staging candidate。

遵守 repo 現有 deploy / preflight / manifest isolation 規則。

部署前：

- service target
- current digest
- candidate digest
- rollback digest
- env contract
- required secrets
- health path
- release identity

若 preflight 發現：

- unrelated manifest changes
- target mismatch
- missing secret
- dirty central deploy config
- wrong service

HARD STOP。

不要強行部署。

---

# PHASE 14 — STAGING LIVE IDENTITY

部署成功後，從 live Staging 查：

/api/admin/shared-engine/identity

必須確認：

source_clean = true
source_commit = CLEAN_SOURCE_COMMIT
wheel_sha256 = HOSTED_CI_WHEEL_SHA256
wheel_version = expected
semantic contract = expected
presentation contract = expected
formatter registry = expected

並核對 DM /api/health 或 release identity：

DM image digest
release
commit
environment

若任何 identity mismatch：

FAIL
並停止後續。

---

# PHASE 15 — STAGING FUNCTIONAL SMOKE

用 Staging 合法身份驗：

1. unauthenticated protected route deny
2. normal authorized flow
3. platform_admin admin identity route
4. Legacy YUCT scrape path
5. Shadow disabled behavior
6. Shadow enabled behavior（若 Staging flag允許）
7. Shared Engine semantic compare
8. Presentation profile compact
9. Presentation profile slash
10. Core Shadow failure 不影響 Legacy user-visible result

不要切 Core Primary。

不要移除 Legacy。

---

# PHASE 16 — ROLLBACK DRILL

本輪只做 Staging rollback。

先記：

PREVIOUS_STAGING_IMAGE_DIGEST

流程：

Candidate deploy
↓
Functional smoke PASS
↓
Rollback to previous immutable digest
↓
Health PASS
↓
Identity = previous release
↓
Re-deploy candidate
↓
Health PASS
↓
Identity = candidate

如果現有 deployment policy 不允許安全 staging rollback drill：

不要自行創造危險流程。

輸出：

ROLLBACK DRILL BLOCKED

並說明原因。

Production 不參與 rollback drill。

---

# PHASE 17 — FINAL STAGING AUDIT

確認：

- Staging current digest
- Shared Engine live identity
- DM commit
- clean source commit
- wheel SHA
- release identity
- health
- smoke
- rollback evidence

Production：

UNCHANGED

Post：

UNCHANGED

591：

UNCHANGED

Production DB：

UNCHANGED

---

# PHASE 18 — DOCUMENTATION

更新既有 SSOT：

ExtractionHub：
- current-state
- history
- ADR / distribution docs

DM：
- current-state
- history
- deployment evidence / handoff existing location

記錄：

- clean commit
- CI run
- wheel identity
- DM image digest
- Staging deploy release
- live identity
- rollback result

不要建立第二套 SSOT。

---

# FINAL REPORT FORMAT

# TICENPI SHARED ENGINE CLEAN RELEASE + DM STAGING REPORT

## 1. Baseline

ExtractionHub:
Branch:
HEAD:
Dirty original checkout:

DM:
Branch:
HEAD:
Dirty original checkout:

Production changed:
NO

## 2. Clean Release Isolation

Fresh worktree:
YES / NO

Release fileset:
PASS / BLOCKED

Unrelated WIP included:
NONE / FOUND

Clean git status:
PASS / FAIL

## 3. ExtractionHub Release

Clean source commit:
Source clean:
true / false

Full tests:
Wheel reproducibility:
Consumer contract:
YUCT:
CTHouse:
HBHousing:
git diff --check:

## 4. Hosted Wheel CI

CI run:
PASS / FAIL

Wheel version:
Wheel SHA256:
Build identity:
Source commit:
Artifact ID:

Exact clean commit:
PASS / FAIL

## 5. DM Legacy Auth Regression

Unauth:
Authorized user:
platform_admin:
Shadow failure preserves Legacy:
Result:
PASS / BLOCKED

## 6. DM Artifact Pin

Expected source commit:
Expected wheel SHA:
Expected contract versions:
source_clean:
true / false

Local tests:
PASS / FAIL

## 7. DM Docker

Local build:
PASS / FAIL

Host path dependency:
NONE / FOUND

Runtime identity:
PASS / FAIL

## 8. Hosted DM ARM64

CI run:
DM source commit:
Shared source commit:
Wheel SHA:
Image digest:
PASS / FAIL

## 9. Staging Deploy

Target:
Candidate digest:
Previous digest:
Release:
Health:
PASS / FAIL

## 10. Live Shared Engine Identity

semantic:
presentation:
formatter registry:
source_commit:
source_clean:
wheel_version:
wheel_sha256:
build_identity:
loaded_from:

Identity match:
PASS / FAIL

## 11. Staging Functional Smoke

Auth:
Legacy scrape:
Shadow:
Semantic compare:
Presentation compact:
Presentation slash:
Legacy unaffected by Core failure:

Result:
PASS / FAIL

## 12. Rollback Drill

Candidate → previous:
PASS / FAIL / BLOCKED

Previous identity:
PASS / FAIL

Previous → candidate:
PASS / FAIL / BLOCKED

Candidate identity restored:
PASS / FAIL

## 13. Isolation

Post writes:
NONE / FOUND

591 writes:
NONE / FOUND

Production writes:
NONE / FOUND

Production DB writes:
NONE / FOUND

## 14. Remaining Gaps

NONE
或逐項列出。

## 15. Final Result

只能：

TICENPI SHARED ENGINE CLEAN RELEASE + DM STAGING READY

或

TICENPI SHARED ENGINE CLEAN RELEASE + DM STAGING BLOCKED

---

# HARD STOP

本輪完成後不要：

- 部署 Production
- 修改 Production DB
- 修改 Post
- 修改 591
- 開始永慶
- 開始台灣房屋
- 切 Core Primary
- 刪 Legacy scraper
- 發布 Source Registry

Staging READY 後等待使用者核准下一步。
