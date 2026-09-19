# DM POST-REFERENCE INTEGRATION AUDIT OWNER

## Mission

以 TicenpiPost 已證明成功的「分支/Worktree → Owner commits → Integration branch → 全面驗證 → Local Docker → Freeze candidate → CI ARM64 → Staging → SAME digest Production」流程為基準，對 TicenpiDM 截至今天的所有工作做一次完整盤點、排查、整合與確認。

本任務不是重新開發 DM，也不是再做大範圍猜測性修復。

你要完成三件事：

1. 找出「今天以前到現在」DM / central deploy / Hub 所有相關 Git commits、branches、worktrees 與它們的實際用途。
2. 找出「已處理但尚未 Git commit / 尚未 merge / 尚未 push / 只存在 runtime / 只存在 untracked files」的工作。
3. 依 POST 的成功整合方式，建立一個可信的 DM integration candidate，確認沒有漏掉功能、重複修改、錯誤 branch、stale Docker、runtime-only hotfix 或未追蹤的重要檔案。

先調查，證據完整後才整合。
禁止一開始直接 merge / cherry-pick / reset。

---

# A. POST 正確流程 — 參考基準

## A1. POST Wave1 Integration 已證明流程

已知成功基準：

BRANCH:
repair/post-v2-integration

WORKTREE:
F:\00-Ticenpi-SaaS\TicenpiPost-integration

Wave1 Integration HEAD:
b3cc93de970f4c7562eff3a4d5e17614462ae8bb

Wave1 integrated commits：

- G0 6e18ef7 → 2212964
  fix(post): decouple product access from studio heartbeat

- G1 e309021 → 6a9bba9
  fix(post): restore account lifecycle and duplicate binding safety

- G2 badb987 → 4d34fd2
  fix(post): preserve studio lease as session state

- G3 0475368 → 3466261
  fix(post): restore resilient facebook group selectors

- G4 1f48c16 → f13b13c
  fix(post): restore direct local extension binding

- G5 7cb0717 → ad1cc0c
  fix(post): isolate local development runtime

- G6 1fb5392 → b3cc93d
  ci(post): produce immutable release artifacts

Wave1 evidence：
- ownership audit PASS
- no cherry-pick conflicts
- targeted backend PASS
- full backend only pre-existing failures
- frontend tests PASS
- typecheck PASS
- build PASS
- extension validation PASS
- docker compose config PASS
- release script validation PASS
- local Docker build PASS
- local health PASS
- final worktree clean
- Production untouched
- READY_FOR_WAVE2=YES

另有已知後續 POST Wave2 integration：
- integration result commit / head reported as 53fce622
- backend 642 tests PASS with 4 pre-existing failures
- frontend/typecheck/build/publish/runtime/config/health-smoke/local Docker verified
- real Facebook action remained outside that specific integration proof

## A2. POST 流程中不可省略的原則

DM 必須比照：

1. 每個 Owner 在獨立 branch/worktree 工作。
2. Owner commit 要單一責任、可追溯。
3. Integration branch 不直接混入 unrelated dirty files。
4. 先做 ownership/file overlap audit。
5. 再依依賴順序 cherry-pick/merge。
6. conflict 必須理解語意後解，不可 ours/theirs 全吃。
7. targeted tests + full regression。
8. Local Docker 必須由 integration candidate 重建。
9. Local Docker 還是舊 UI / stale image 時不得往下。
10. Local PASS 後才 Freeze Git SHA。
11. GitHub Actions 從 exact frozen SHA 建 ARM64 release image。
12. VPS 不從 extracted source tree build release image。
13. Staging 部署 exact CI ARM64 digest。
14. Staging Accepted 後 Production promote SAME ARM64 digest，不 rebuild。
15. Production / Staging 只換 runtime config/secrets，不換 application artifact。

這是本任務判斷 DM 是否「流程正確」的基準。

---

# B. DM 已知 Git / Runtime 歷史 — 必須逐一驗證，不可盲信

以下是今天對話中已知資訊。它們是「待核對 inventory seed」，不是可以不查就直接 merge 的指令。

## B1. Staging / runtime baseline

5e6803461cb8...
- staging candidate
- frontend runtime-config injection related
- staging release 20260918-222526 曾運行此 source identity

5151699
- disposable runtime contract related
- 曾用於 local/staging runtime contract work
- 必須查實際 diff / ancestry / 是否已被後續 commit包含

## B2. Production startup fix

ef2e668f75df97c7b40e7c3fb7195386a7719905
known purpose:
fix(dm): unblock production container startup

已知包含/相關：
- shell script CRLF/LF fix
- .gitattributes *.sh eol=lf
- docker-compose runtime contract adjustment
- TICENPI_COMMIT_SHA / TICENPI_GIT_SHA fallback

Production release 20260919-110713 曾回報：
commit ef2e668f75df

注意：
後續發現 root TicenpiDM/docker-compose.yml 可能因此漂移成 production profile。
必須確認 ef2e668 是否把 canonical Local Docker flow 污染成 production runtime contract。
不可直接假定 ef2e668 整體都要保留或整體 revert。

## B3. Entitlement security fix

86f2c97

known purpose:
Backend Entitlement enforcement

已知結果：
- designs CRUD protected
- variants protected
- scrape protected
- ai-draw protected
- images protected
- removebg protected
- valid entitled ALLOW
- no entitlement DENY
- expired DENY
- wrong product DENY
- invalid JWT DENY
- trial backend bypass reported NO
- tests PASS

這是目前 DM 商用安全的重要 commit。
必須確認：
- commit diff
- parent/base
- 是否已被後續 candidate 包含
- 是否與 AI/Capture/Release Gates 有 overlap

## B4. Release Gates — DM

2325cca

branch reported:
dm/release-gates-wave1

base reported:
86f2c97

known purpose:
- Capture Gate A/B release-blocking
- auth invalid JWT release-blocking
- auth no-entitlement release-blocking
- auth entitled release-blocking
- app.release_gates runner
- Gate C hook
- AI E2E hook
- hardcoded JWT secret removal / safe secret loading
- fail-closed validation
- isolated runner 22/22 PASS

必須確認：
- commit still exists
- exact files
- ancestry includes 86f2c97
- untracked gate_*.py/test_*.py 是否已被吸收、superseded 或仍散落在 root

## B5. Release Gates — central deploy

f8d4f0b

reported branch:
dm/release-gates-wave1

reported base:
216f709

known purpose:
- services.yaml DM profile → dm-gates-full
- critical-smoke profile wiring
- deploy.sh / audit.sh profile allowlist
- release blocking on nonzero
- central pipeline integration

必須確認 central deploy 是獨立 Git repo 還是 monorepo path，並建立正確 repo/branch ancestry。

## B6. Capture production incident / runtime changes

Known incident:
Production Capture 曾因 DM_CORS_ORIGINS 不含 https://dm.ticenpi.com 而 browser preflight 失敗。

Known runtime fix:
- Production shared .env 增加 production origin
- deployment release 20260919-110713
- Gate A/B PASS
- Browser Gate C 尚需/曾需真人 E2E

重要：
這裡可能有 runtime-only change、.env.example change、未 commit change。
必須找：
- 哪些有 Git commit
- 哪些只存在 VPS env
- 哪些存在 local .env.example
- 哪些尚未 Git
- 不得把 secret value 寫入報告

## B7. AGNES / AI work

已知今天後段 Agent 曾回報：
- AGNES_API_KEY 原本「在 .env」
- 問題不是重新索取 key，而是 env → backend runtime wiring/loading/injection
- 後續 Agent 又回報「AGNES PRESENT、AI PASS」
- classification complete / real defects fixed locally
- CI harness exit 0
- 但對話中沒有可靠提供最終 AI fix commit SHA

因此這一項必須視為：
POSSIBLY_FIXED_LOCALLY_BUT_COMMIT_ID_UNKNOWN

必查：
- git diff
- reflog
- branches/worktrees
- recent commits
- config.py / env loader / compose env / health changes
- AI tests
- AGNES health additions
- whether fix is committed
- whether pushed
- whether included in integration candidate

如果只是 local process env 或手動 restart，不能當成 Git fix。

## B8. Local runtime / Docker work

已知：
- Local Dev frontend localhost:9422
- Local Dev backend 127.0.0.1:8000
- 9422 曾因 dev server未啟動而 closed，後來手動啟動 Vite + uvicorn 後 PASS
- 這是 process/runtime action，不一定有 Git change

Old Local Docker:
localhost:9421
- stale image
- old frontend bundle
- running local/disposable env
- on-disk root docker-compose.yml 已漂移成 production profile
- stale running container env 與 on-disk compose 不一致

後續總控要求：
- 重建 9421 from integrated candidate
- 9421 old UI/stale image = HARD STOP
- Local Docker PASS 才可以 freeze/push

必須確認「今天到底有沒有真的完成新版 9421 rebuild」。
不能只看 health=200。
必須看：
- frontend bundle/build identity
- candidate source SHA
- image ID
- runtime config
- no production targets
- AGNES presence
- Capture/AI/auth pass

## B9. Hub

Known audit:
- Hub DM card found
- launch URL 曾指到 Tailscale/local :9421，而不是 dm.ticenpi.com
- Hub release/status stale
- Hub possibly development-workflow Hub, not customer-product Hub

今天曾派出 Hub sync fix task，但對話中沒有可靠回報最終 Hub fix commit SHA。

因此：
HUB_FIX_STATUS = UNKNOWN

必查：
- Hub git root
- branches/worktrees
- recent commits
- registry/open_url
- release/status changes
- tests/invariants
- committed/uncommitted/pushed state

不要把 entitlement logic塞進開發工作流 Hub，除非 evidence證明是 customer product Hub。

## B10. SSOT docs

Known docs problem:
- TicenpiDM/_charter/current-state.md outdated
- history.md incomplete
- dispatch.md stale
- staging-closeout docs existed on side branch
- staging closeout / Tailscale incident had docs commit 4917eb9
- runtime-staging compose related commit d0a2640
- branch staging-closeout-20260918-docs existed

必須確認：
- 4917eb9 是否已 merge
- d0a2640 是否在哪個 branch
- production branch/current integration 是否包含 docs
- docs 是否又被今天後續工作更新
- 不要把 runtime truth反寫成錯誤「Commercial Go-Live Ready」

## B11. Known untracked / dirty history

曾明確回報：
- root 有 8 個 untracked test files
- 類型含 gate_*.py / generate_test_token.py / test_*.py
- 它們在 entitlement commit 時是 pre-existing/unrelated
- release-gate工作後部分功能可能已被正式 tracked runner取代

必須逐檔分類：
KEEP / TRACK / SUPERSEDED / INCIDENT_ONLY / DELETE_CANDIDATE

本任務禁止直接 delete。
先報告證據與建議。

另有：
- ci-artifacts 曾移出 repo 到 .incident-cache
- Local Vite/uvicorn process startup不是 Git change
- VPS .env runtime fixes不是 Git commit
- Browser/OAuth read-only investigation不是 Git change

不要把 runtime actions誤當 source commits。

---

# C. Phase 1 — 全面 Git / Worktree Inventory（只讀）

先掃描：

F:\00-Ticenpi-SaaS\TicenpiDM
F:\00-Ticenpi-SaaS\
F:\HUB
以及 git worktree list 找到的所有相關 worktrees。

對每個 Git repo輸出：

- repo root
- remote origin
- current branch
- HEAD SHA
- upstream
- ahead/behind
- git status --porcelain
- git worktree list
- local branches
- relevant remote branches
- recent 40 commits --oneline --decorate --graph
- tags/releases if relevant

不得只查 current worktree。

---

# D. Phase 2 — Commit Purpose / Ancestry Matrix

對至少以下 SHA逐一 verify：

POST reference:
- b3cc93d
- 53fce622

DM:
- 5e68034
- 5151699
- ef2e668
- 86f2c97
- 2325cca
- 4917eb9
- d0a2640

Deploy:
- f8d4f0b
- base 216f709 if present

每個 commit輸出：

COMMIT
REPO
BRANCHES_CONTAINING
PARENT
PURPOSE
FILES_CHANGED
DEPENDENCIES
SUPERSEDED_BY
REQUIRED_FOR_FINAL_CANDIDATE
PUSHED_REMOTE = YES/NO
MERGED_INTO_CURRENT = YES/NO

不要根據 commit message猜 purpose；要看 diff。

---

# E. Phase 3 — 找出「做過但沒 Git」的所有工作

這是本任務最重要部分之一。

使用：

- git status
- untracked files
- git diff
- git diff --cached
- reflog
- worktree branches
- recent file mtimes只作輔助
- Docker running image/runtime identity
- local .env presence only
- VPS/runtime evidence only作對照，不把 secret讀出

建立：

UNCOMMITTED_WORK_MATRIX

欄位：

PATH
REPO/WORKTREE
STATUS = modified/untracked/runtime-only/process-only
LIKELY_OWNER
PURPOSE
RELATED_KNOWN_TASK
DUPLICATES_COMMITTED_WORK = YES/NO/UNKNOWN
NEEDS_COMMIT = YES/NO/REVIEW
SECRET_RISK = YES/NO
RECOMMENDED_DESTINATION_BRANCH
EVIDENCE

特別找：

1. AGNES env wiring / config / health / AI tests
2. Local Docker compose/runtime correction
3. Capture gate scripts
4. Browser Gate C scripts
5. AI E2E scripts/hooks
6. Hub sync fix
7. SSOT docs
8. .env.example changes
9. runtime-staging compose changes
10. generated test token scripts / hardcoded secret remnants

---

# F. Phase 4 — 對照 POST 流程做 Gap Analysis

逐項判斷 DM：

OWNER_BRANCH_ISOLATION =
PASS / FAIL

OWNERSHIP_AUDIT =
PASS / FAIL

INTEGRATION_BRANCH_EXISTS =
YES / NO

ALL_REQUIRED_COMMITS_IDENTIFIED =
YES / NO

UNCOMMITTED_REQUIRED_WORK_IDENTIFIED =
YES / NO

TARGETED_TESTS =
PASS / FAIL / STALE

FULL_REGRESSION =
PASS / FAIL / STALE

LOCAL_DOCKER_REBUILT_FROM_FINAL_SOURCE =
YES / NO

LOCAL_DOCKER_STALE =
YES / NO

LOCAL_DOCKER_RUNTIME_LOCAL_ONLY =
YES / NO

FROZEN_CANDIDATE_SHA_EXISTS =
YES / NO

CI_ARM64_BUILT_FROM_FROZEN_SHA =
YES / NO / NOT_YET

STAGING_USES_CI_DIGEST =
YES / NO / NOT_YET

VPS_SOURCE_TREE_BUILD_PRESENT =
YES / NO

SSOT_MATCHES_RUNTIME =
YES / NO

---

# G. Phase 5 — Integration Plan Before Mutation

在任何 cherry-pick/merge 前，先輸出精準 integration order。

範例格式：

DM_INTEGRATION_ORDER:
1. base = ...
2. entitlement 86f2c97
3. release gates 2325cca
4. AGNES fix = <discovered SHA or UNCOMMITTED>
5. local runtime/docker fix = <SHA or UNCOMMITTED>
6. capture related = <only if not already included>
7. docs = after technical acceptance

DEPLOY_INTEGRATION_ORDER:
1. base = ...
2. f8d4f0b
3. later required deploy fixes = ...

HUB_INTEGRATION_ORDER:
...

每一項要說明 WHY。

若 AGNES / Local Docker / Hub 的重要修復仍未 commit：
先不要硬整合。
建立「需要先 commit 的 owner package」清單。

---

# H. Phase 6 — 安全整合

只有 Phase 1–5 證據完整後才允許。

建立新的獨立 integration worktree/branch，例如：
dm/integration-audit-20260920

規則：
- 不在 dirty canonical worktree直接整合
- 不 reset --hard
- 不 git clean
- 不刪 untracked files
- 不改 Production
- 不 deploy
- 不 push，除非使用者另行批准

依 verified order cherry-pick/merge。

Conflict：
逐檔解析語意。
禁止 ours/theirs全吃。

如果某 commit已被 descendant包含：
不要重複 cherry-pick。

---

# I. Phase 7 — Integration Verification

## DM backend
- auth
- entitlement
- scrape
- AI
- release gates
- targeted tests
- full relevant regression

## Frontend
- tests
- typecheck
- build

## Capture
- Gate A
- Gate B
- confirm /api/scrape remains httpx + existing proxy contract
- 不因本任務亂改 CORS/JWT/proxy

## AGNES
- env source presence value-blind
- runtime presence value-blind
- local AI minimal generation PASS
- no secret in tracked diff

## Local Docker
必須真正重建 localhost:9421 from integration candidate。

驗：
- old stale bundle gone
- new UI/build identity
- /api/health
- local/disposable runtime
- no Production Supabase/origin
- candidate SHA identity
- Capture PASS
- AI PASS
- auth PASS

如果 9421還是舊畫面：
INTEGRATION_ACCEPTED = NO

---

# J. 不要混淆 local digest 與 ARM64 digest

本機 Windows/x86 Docker digest 不要求等於 GitHub Actions ARM64 digest。

正確 invariant：

LOCAL_DOCKER_SOURCE_SHA
=
FROZEN_CANDIDATE_GIT_SHA
=
CI_ARM64_SOURCE_SHA

之後：

STAGING_IMAGE_DIGEST
=
CI_ARM64_IMAGE_DIGEST

再之後：

PRODUCTION_IMAGE_DIGEST
=
STAGING_ACCEPTED_ARM64_DIGEST

VPS禁止 source-tree rebuild。

本任務只做到 Local integration verification，不自動 deploy。

---

# K. 最終報告格式

輸出一份：

## 1. POST_REFERENCE_FLOW
逐項列出 POST 已證明流程與 DM 對照。

## 2. REPOSITORY_MAP
所有 DM / deploy / Hub repo、worktree、branch、HEAD。

## 3. COMMITTED_WORK
表格：
SHA | Repo | Purpose | Branch | Included? | Pushed? | Required?

## 4. UNCOMMITTED_WORK
表格：
Path | Worktree | Purpose | Status | Needs commit? | Secret risk | Destination

## 5. RUNTIME_ONLY_WORK
例如：
- VPS .env
- local Vite/uvicorn process
- Docker container runtime
說明哪些不是 Git。

## 6. DUPLICATES / SUPERSEDED
哪些舊 scripts/commits已被正式 runner取代。

## 7. MISSING_WORK
做過但找不到 Git證據的工作。

## 8. FINAL_INTEGRATION_ORDER
精準 commit order。

## 9. INTEGRATION_RESULT
INTEGRATION_BRANCH =
INTEGRATION_HEAD =
COMMITS_INCLUDED =
CONFLICTS =
TESTS =
LOCAL_DOCKER =
CAPTURE =
AGNES =
AUTH =
WORKTREE_CLEAN =
UNTRACKED_REMAINING =

## 10. RELEASE_READINESS
READY_TO_FREEZE_CANDIDATE =
YES / NO

READY_TO_PUSH_FOR_CI_ARM64 =
YES / NO

READY_FOR_STAGING =
YES / NO

BLOCKERS =
...

## 11. CRITICAL RULE
只有在：
- required commits全齊
- required uncommitted work已處理
- integration tests PASS
- localhost:9421真的是新版 candidate Docker
- Capture PASS
- AGNES PASS
- entitlement/auth PASS
- tracked secrets = none

才可回報：

READY_TO_FREEZE_CANDIDATE = YES

完成後停止。
不要 push。
不要 deploy Staging。
不要 deploy Production。
