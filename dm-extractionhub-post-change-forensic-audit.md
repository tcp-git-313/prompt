# DM / ExtractionHub Post-Change Forensic Audit — 8020, Staging Branch, Shared Engine, Central Seat

## 任務目標

目前有人表示「已經做好」，但提供的本機網址是：

http://127.0.0.1:8020/

使用者懷疑這可能是舊版 DM / 舊 runtime / 舊 worktree 的端口。

本任務 **先不要修、不要部署、不要切換 branch、不要改資料**。

唯一目標是做一次完整的 forensic audit，回答清楚：

1. 127.0.0.1:8020 現在到底是什麼程式、哪個 repo、哪個 worktree、哪個 commit、哪個 branch、哪個 build。
2. 對方「做好了」到底改了哪些檔案、commit、branch、worktree、runtime。
3. 這些改動有沒有覆蓋、破壞或繞過之前已完成的 Shared Engine / DM Staging 整合。
4. 有沒有讓 Central Seat / entitlement / platform_admin 權限鏈退化或失效。
5. 之前 Shared Engine semantic / presentation 比對、Shadow Console、DM bridge 到底是在：
   - 哪個 branch
   - 哪個 worktree
   - 哪個 commit
   - 哪個 Staging release
   上完成。
6. 使用者記得「比對是在 DM-STAGING 分支」，請用 git / worktree / commit / deploy evidence **確認，不要靠印象**。
7. 最後產出「目前真正可相信的基準」以及「8020 是否只是舊版或錯誤啟動」。

本任務是 **調查任務**。

在 root cause 與 topology 沒查清楚前：

- 不准修改 source
- 不准 commit
- 不准 merge
- 不准 cherry-pick
- 不准 reset / clean / stash
- 不准 deploy
- 不准 rollback
- 不准改 Staging DB
- 不准改 Production / Production DB
- 不准改 Central Seat data
- 不准補 entitlement / Seat
- 不准為了讓測試過而關閉 auth

---

# 已知歷史基準（只作比對線索，必須重新驗證）

以下是過去紀錄，不可直接當成目前狀態：

## ExtractionHub Shared Engine clean source

Clean source commit：

11762508332bb2508538761302a90f8dca5f2586

Wheel：

ticenpi_shared_engine-0.1.0-py3-none-any.whl

Wheel SHA256：

e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3

Hosted wheel CI：

35863227635

## DM Shared Engine Staging candidate

DM image source commit：

25db5f9b8b65a231dafb7cc8f1f05262fb7519aa

Manifest commit：

34a59a7fb50ce5f8b6071e90ab5ee12ca763f3fb

Backend ARM64 digest：

sha256:51efc74c75e35d0d857434e3f113bb0546c9072b2b12398f4c33b1555ffb0e83

Frontend ARM64 digest：

sha256:0842f90d476ab7fa4b74d1c70781cc5bd1b1a7513848e926e5f5c9c148230093

Staging release：

20260923-215914

Previous rollback target：

20260922-131327

Historical integration branch seen previously：

dm/integration-20260920

Historical local HEAD seen previously：

561bd8c76fec5c0fb86cb9cf4f56734d28367b8a

## Historical Shared Engine implementation locations

ExtractionHub:

- src/extractionhub/semantics.py
- src/extractionhub/presentation.py

DM bridge historically included:

- src/backend/app/shadow/shared_engine.py

Historical functionality included:

- semantic comparator
- canonical values
- presentation formatter
- Shadow Console
- platform_admin-only Shared Engine identity/profile-related admin surface
- YUCT Legacy remains primary
- Extraction Core remains Shadow / non-primary

以上全部要重新確認目前是否還存在、在哪個 branch/worktree、是否被改壞。

---

# 執行策略：平行 READ-ONLY，禁止平行 Writer

可以平行執行 5 個只讀 Track：

TRACK A — 8020 Runtime / Process Forensics  
TRACK B — Git / Branch / Worktree / Commit Topology  
TRACK C — Shared Engine / Shadow / Staging Lineage  
TRACK D — Central Seat / Entitlement / platform_admin Regression Audit  
TRACK E — Diff / Test / Runtime Comparison

所有 Track：

- READ ONLY
- 不改 source
- 不 commit
- 不 deploy
- 不切 branch
- 不 checkout
- 不 reset
- 不 stash
- 不 clean
- 不改 DB

最後由單一 Planner 統整。

---

# TRACK A — 127.0.0.1:8020 FORENSICS

先確認 8020 是誰在 listen。

使用 Windows 可用工具，例如：

- Get-NetTCPConnection
- netstat -ano
- Get-Process
- Win32_Process / CIM
- tasklist
- PowerShell process command line inspection

記錄：

- PID
- process name
- full command line
- parent PID
- start time
- executable path
- working directory（若可取得）
- listening address
- whether Docker / Python / Node / Uvicorn / Vite / reverse proxy

不要殺 process。

然後 HTTP read-only fingerprint：

- GET /
- GET /api/health（若存在）
- GET /health（若存在）
- GET runtime/version endpoint（只依 repo 實際存在）
- response headers
- x-ticenpi-release
- environment
- release
- commit
- frontend/backend version

不要猜 endpoint，先從 repo / network evidence 查。

回答：

> 8020 到底是不是目前 DM？

分類：

- CURRENT_DM_LOCAL
- OLD_DM_LOCAL
- OLD_RELEASE_RUNTIME
- DIFFERENT_WORKTREE
- DIFFERENT_PROJECT
- UNKNOWN

如果是 Docker：

查：

- container name
- image
- image digest
- labels
- compose project
- mounts
- working directory
- env names only（不要印 secrets）

如果是 Python/Node：

從 command line + working dir 對應 repo/worktree。

---

# TRACK B — GIT / BRANCH / WORKTREE TOPOLOGY

在以下 repo 做 read-only inventory：

F:\00-Ticenpi-SaaS\TicenpiDM
F:\00-Ticenpi-SaaS\ExtractionHub
F:\00-Ticenpi-SaaS\ticenpi-platform

以及實際存在的相關 worktree。

至少執行：

- git status --short
- git branch --show-current
- git rev-parse HEAD
- git log --oneline --decorate -n 30
- git branch -a --contains <known commits>
- git worktree list --porcelain
- git show --stat --oneline <relevant commits>
- git diff --name-status <baseline>..<current>
- git diff --stat <baseline>..<current>

必要時：

- git reflog
- git merge-base
- git log --all --graph --decorate

但只讀。

建立：

## DM WORKTREE MAP

| Path | Branch | HEAD | Dirty | Purpose/Evidence |

特別找：

- branch 名稱包含 staging / dm-staging / shared-engine / shadow / release
- worktree 名稱包含 dm-shared-engine-staging
- commit 25db5f9...
- commit 561bd8c...
- manifest commit 34a59a7...

使用者記得：

> 「之前的比對是在 DM-STAGING 分支」

請證明或否定：

- 是否真的存在名為 DM-STAGING / dm-staging 類 branch
- 是否只是 worktree 名稱
- 是否其實是 release branch / staging worktree
- semantic comparator / presentation / Shadow UI 是在哪一條 lineage 完成

不得只用「看起來像」。

輸出：

DM STAGING LINEAGE = <branch/worktree/commit evidence>

---

# TRACK C — SHARED ENGINE / SHADOW / STAGING LINEAGE

確認 Shared Engine 功能目前在哪裡。

ExtractionHub：

- semantic contract
- presentation contract
- formatter registry
- package metadata
- wheel build config
- identity metadata

DM：

- shared engine bridge
- identity endpoint
- shadow comparison
- canonical output
- DM presentation profile
- Legacy/Core/raw/canonical/final display
- admin-only controls

建立功能矩陣：

| Capability | Historical Baseline | Current Main Checkout | Staging Lineage | 8020 Runtime |

至少：

- semantic comparator
- NORMALIZED_MATCH
- canonical layout
- presentation formatters
- DM final display
- Shadow Console
- Shared Engine identity endpoint
- Legacy primary
- Core shadow-only

確認：

1. 8020 是否包含這些功能？
2. 若 8020 不包含，是不是因為它是舊 commit？
3. 對方「做好了」的改動是在主 checkout 還是另一個 worktree？
4. 新改動是否誤把 Legacy/Core primary 關係改掉？
5. 是否把 wheel integration 移除、改回 vendored/local copy、或改成 source checkout？
6. 是否破壞 pinned wheel SHA / identity contract？

如果發現 Shared Engine source 被 duplicate / fork：

標記：

SHARED_ENGINE_DRIFT_RISK

---

# TRACK D — CENTRAL SEAT / ENTITLEMENT / PLATFORM_ADMIN REGRESSION AUDIT

這一 Track 非常重要，但只讀。

目標不是重做 Central Seat，而是確認最近改動有沒有把 DM 的授權鏈改壞。

追目前 DM protected route，例如：

GET /api/designs

實際 dependency chain：

Bearer
→ current user
→ platform role
→ commercial context
→ product entitlement
→ Seat（若目前 branch 已接）
→ allow/deny

比較：

1. historical baseline
2. current main checkout
3. Staging lineage
4. 8020 runtime 對應 source

至少確認：

- platform_admin exemption 還在不在
- PRODUCT_ENTITLEMENT_REQUIRED 邏輯有沒有改
- business entitlement 行為有沒有被放寬
- Central Seat helper / RPC 是否被接入或移除
- assigned / unassigned 行為是否被改
- local bypass 是否被誤帶進 Staging code path
- DM_REQUIRE_AUTH semantics 是否改變
- test_access / dev-user / loopback bypass 是否污染正式路徑

特別注意：

之前已知現況曾是：

- platform_admin 先豁免
- 一般 user 走 commercial entitlement
- business customer 的 DM Seat authoritative gate 尚未完整成為 deployed primary gate

如果最新改動聲稱 Central Seat 已完成：

必須拿 code + tests + Staging evidence 證明。

不能因 unit test 名稱存在就判完成。

建立：

## CENTRAL SEAT REGRESSION MATRIX

| Behavior | Historical | Current Checkout | Staging Lineage | 8020 |

至少：

- auth required
- platform_admin exemption
- product entitlement
- business Seat
- assigned
- unassigned
- local bypass isolation

分類：

UNCHANGED
INTENTIONALLY_CHANGED
REGRESSED
UNKNOWN

---

# TRACK E — DIFF / TEST / RUNTIME COMPARISON

先確認「對方做了什麼」。

找：

- recent commits
- uncommitted diff
- generated files
- compose changes
- env/config changes
- frontend port changes
- backend port changes
- runtime launch scripts
- docs claiming done

建立：

## CHANGE INVENTORY

| File | Change | Why | Affects Shared Engine | Affects Auth/Seat | Runtime Impact |

如果 commit 尚未提交：

用 working-tree diff。

如果在其他 worktree：

對該 worktree diff。

不要改。

---

# TEST POLICY

只允許執行不會改資料的 tests。

可以平行跑：

- ExtractionHub semantic/presentation tests
- DM backend auth/entitlement/seat tests
- DM Shadow tests
- DM frontend tests
- cookie isolation tests
- build
- git diff --check

如果 test 會寫 DB：

只可用 disposable/local test DB。

禁止對 Staging DB 做 destructive test。

禁止對 Production。

---

# LIVE STAGING READ-ONLY CHECK

只讀確認目前真正的：

https://dm-staging.ticenpi.com/

至少：

- /api/health
- release
- commit
- headers
- environment

如果已有 authenticated browser session：

只可安全記錄：

- request 是否有 Authorization
- HTTP status
- typed error code

不要輸出 credential。

如果沒有可用登入 session：

不要卡住整個 forensic audit。

Staging auth E2E 可標 NOT_RUN。

---

# 8020 與 STAGING 必須直接比較

最終一定要回答：

## 8020

- repo:
- worktree:
- branch:
- commit:
- release:
- purpose:
- old/current:

## DM Staging

- branch lineage:
- source commit:
- release:
- Shared Engine present:
- Shadow present:
- auth/seat behavior:

然後：

8020 == Staging ?

YES / NO

如果 NO：

Difference reason:

例如：

- 8020 = old local dev
- 8020 = unrelated worktree
- 8020 = pre-shared-engine baseline
- 8020 = local convenience runtime
- Staging = actual Shared Engine candidate

---

# 判斷「有沒有改壞」

不能只看 tests。

必須同時看：

1. git lineage
2. exact diff
3. runtime identity
4. Shared Engine capability matrix
5. Central Seat/auth matrix
6. relevant tests

分類：

NO_REGRESSION_FOUND
REGRESSION_CONFIRMED
PARTIAL_REGRESSION
UNVERIFIED

如果有 regression：

只指出：

- 哪個檔案
- 哪個 commit/diff
- 哪個 behavior
- 哪個 test/runtime evidence

本任務不要修。

---

# 特別確認：之前比對到底在哪裡

請專門做一節：

## HISTORICAL COMPARATOR LOCATION

回答：

- semantic comparator implementation repo:
- presentation implementation repo:
- DM bridge repo:
- DM branch:
- DM worktree:
- DM commit:
- Staging release:
- Shadow Console location:
- Was it actually "DM-STAGING branch"?
  - YES
  - NO
  - PARTIALLY / naming confusion

Evidence:

- git branch containment
- worktree mapping
- commit ancestry
- deploy manifest/release identity

這一節不能省略。

---

# FINAL REPORT FORMAT

# DM / EXTRACTIONHUB POST-CHANGE FORENSIC AUDIT REPORT

## 1. Executive Result

8020 classification:
Current trusted DM baseline:
Current trusted ExtractionHub baseline:
Regression status:
Central Seat/auth status:

## 2. 8020 Runtime

PID:
Process:
Command:
Working directory:
Container:
Image/digest:
Repo:
Worktree:
Branch:
HEAD:
Release:
Environment:

Classification:

## 3. DM Git / Worktree Topology

| Path | Branch | HEAD | Dirty | Purpose |

## 4. Historical Comparator Location

Semantic comparator:
Presentation engine:
DM bridge:
Shadow Console:
Branch:
Worktree:
Commit:
Staging release:

Was previous comparison actually on DM-STAGING branch:
YES / NO / PARTIAL

Evidence:

## 5. What Changed

| File / Commit | Change | Shared Engine Impact | Auth/Seat Impact | Risk |

## 6. Shared Engine Regression Matrix

| Capability | Baseline | Current | Staging | 8020 | Result |

Include:

semantic comparator
presentation
NORMALIZED_MATCH
canonical
DM final display
Shadow
identity
Legacy primary
Core shadow-only

## 7. Central Seat / Auth Regression Matrix

| Behavior | Baseline | Current | Staging | 8020 | Result |

Include:

auth required
platform_admin exemption
entitlement
business Seat
assigned
unassigned
local bypass isolation

## 8. Tests

ExtractionHub:
DM backend:
DM frontend:
Shadow:
Seat/Auth:
Cookie:
Build:
diff-check:

## 9. Staging Runtime

URL:
Release:
Commit:
Environment:
Shared Engine evidence:
Auth evidence:

## 10. 8020 vs Staging

Same code:
YES / NO

Same release:
YES / NO

Same auth path:
YES / NO

Same Shared Engine:
YES / NO

Explanation:

## 11. Did The Recent Work Break Anything?

Result:

NO_REGRESSION_FOUND
REGRESSION_CONFIRMED
PARTIAL_REGRESSION
UNVERIFIED

Evidence:

## 12. Central Seat Safety

Central Seat modified:
YES / NO

Entitlement semantics changed:
YES / NO

platform_admin changed:
YES / NO

Seat assigned/unassigned semantics changed:
YES / NO

Risk:

## 13. Safe Next Step

只列下一個最小安全任務。

不要自動執行修復。

## 14. Isolation

Production:
UNCHANGED / UNKNOWN

Production DB:
UNCHANGED / UNKNOWN

Staging DB:
UNCHANGED / UNKNOWN

Other products:
UNCHANGED / UNKNOWN

## 15. Final Verdict

只能選：

FORENSIC AUDIT CLEAN — READY FOR NEXT VALIDATION

或

FORENSIC AUDIT FOUND REGRESSION — FIX REQUIRED

或

FORENSIC AUDIT INCONCLUSIVE — MORE EVIDENCE REQUIRED

---

# HARD STOP

本輪絕對不要：

- 修 code
- merge
- cherry-pick
- checkout 切 branch
- reset
- stash
- clean
- deploy
- rollback
- 改 Staging DB
- 補 entitlement
- assign/release Seat
- 改 Production
- 改 Production DB
- 修改 591 / Sign / Post

先把：

8020 是誰
+
真正 Staging lineage
+
Shared Engine 是否完整
+
Central Seat/auth 是否被改壞
+
之前 comparator 到底在哪個 branch/worktree

全部查清楚，再決定下一步。
