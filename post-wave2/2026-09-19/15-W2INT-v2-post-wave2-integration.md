# W2INT-v2 — POST Wave2 Integration Retry

- 抬頭：POST WAVE2 INTEGRATION RETRY OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：大型
- Authoritative Base：b3cc93de970f4c7562eff3a4d5e17614462ae8bb

---

你現在是：

POST WAVE2 INTEGRATION RETRY OWNER

這是正式 Integration Retry。

不是重新設計，不是重新開發，也不是修舊 conflict。

## AUTHORITATIVE BASE

Exact Wave2 base：

b3cc93de970f4c7562eff3a4d5e17614462ae8bb

branch:
repair/post-v2-wave2-base

worktree:
F:\00-Ticenpi-SaaS\TicenpiPost-wave2-base

## IMPORTANT — OLD FAILED INTEGRATION IS EVIDENCE ONLY

舊 worktree：

F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration

目前保留 unresolved conflict evidence。

禁止碰它。

不得：

- cherry-pick --abort
- reset
- restore
- clean
- resolve conflict
- checkout mutation
- commit

本 task 必須建立全新 integration worktree。

## APPROVED EXACT COMMITS

### W2A RECUT

W2A first:
d5a109a75142c23f3373a57480012da6ed476a52

W2A second:
f8c053514992741f953e52a8b68da18120b0740f

這兩個 commit 必須依序套用。

### W2B Publish Safety

e8e300ba65f9645b7aa23146e5164f961b438a98

### W2D Runtime Config

a5ec9345fe2912fa2a2dd72862ddba7024ce4c04

### W2BD Runtime Wiring Bridge

26e6fba85355e5ad3a5f68637dc83bfcde1e8289

注意：
W2BD branch 曾內含 W2B + W2D lineage。
Integration 只 cherry-pick bridge commit 26e6fba，
前提是 W2B 與 W2D 已先套用。

### W2E Health / Smoke

ddedd8c41f47d4dff8c99ad3ccd04b273244845a

### W2F Regression Pack

43186fec18577ec52c897eb498cad9af15d3c51c

### W2F Contract Correction

65ecfaa75fcabb0c5c97cbdf8c931ad859844975

## CREATE FRESH INTEGRATION

建立：

branch:
repair/post-v2-wave2-integration-v2

worktree:
F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2

直接從 exact base：

b3cc93de970f4c7562eff3a4d5e17614462ae8bb

建立。

確認：

git rev-parse HEAD
=
b3cc93de970f4c7562eff3a4d5e17614462ae8bb

git status --porcelain
=
empty

否則 STOP。

## HARD RULES

禁止：

- merge feature branches
- rebase
- reset
- stash
- clean
- bulk port dirty trees
- manual conflict resolution
- ours/theirs
- push
- Production/VPS write
- Supabase write
- real Facebook action

任一 cherry-pick conflict：

立即 STOP。

## PHASE 1 — OWNERSHIP AUDIT

逐 commit 用 git show --name-only 驗證。

Expected ownership：

W2A first/second:
- frontend/src/components/SessionGate.tsx
- frontend/tests/session-gate.test.tsx

W2B:
- backend/app/core/publish_gate.py
- worker/executor/driver/task related files
- publish safety tests

W2D:
- deploy/staging/*
- deploy/runtime-staging/*

W2BD:
- docker-compose.yml
- env examples
- backend/app/core/config_drift.py
- dedicated wiring/drift tests

W2E:
- deploy/healthcheck.sh
- deploy/critical-smoke.sh
- deploy/smoke/post_smoke.py

W2F:
- Wave2 test files only

若任何 commit 超出已核准 ownership：
STOP。

## PHASE 2 — CHERRY PICK ORDER

依序 exact cherry-pick：

1. d5a109a75142c23f3373a57480012da6ed476a52
2. f8c053514992741f953e52a8b68da18120b0740f
3. e8e300ba65f9645b7aa23146e5164f961b438a98
4. a5ec9345fe2912fa2a2dd72862ddba7024ce4c04
5. 26e6fba85355e5ad3a5f68637dc83bfcde1e8289
6. ddedd8c41f47d4dff8c99ad3ccd04b273244845a
7. 43186fec18577ec52c897eb498cad9af15d3c51c
8. 65ecfaa75fcabb0c5c97cbdf8c931ad859844975

任何 conflict：
STOP。

## PHASE 3 — BACKEND VALIDATION

至少跑：

- backend/tests/test_publish_action_gates.py
- backend/tests/test_publish_runtime_wiring.py
- backend/tests/test_config_drift.py
- backend/tests/test_wave2_*.py
- auth / identity / commercial / studio suites
- relevant driver/executor tests

然後跑 full backend suite。

已知 historical 4 failures只有在與 authoritative base / Wave1完全相同時才可標 PRE_EXISTING：

- test_extract_api golden fixture
- test_frontend_invariants backdrop-filter
- test_invariants playwright
- test_invariants DOM selector

任何新增 failure：
STOP。

## PHASE 4 — FRONTEND VALIDATION

跑：

- frontend/tests/session-gate.test.tsx
- frontend/tests/wave2-access-regression.test.tsx
- full npm test
- npx tsc --noEmit
- npx next build

Access matrix 必須：

signed_out → BLOCK
no_tenant → BLOCK
not_entitled → BLOCK
real auth failure → BLOCK

studio_offline → ALLOW WEB
account_mismatch → ALLOW WEB

backend_unavailable → distinct service unavailable state

unknown reason → fail-safe block

## PHASE 5 — PUBLISH SAFETY MATRIX

Master：

TICENPI_AUTO_PUBLISH

Actions：

TICENPI_AUTO_PUBLISH_POST
TICENPI_AUTO_PUBLISH_COMMENT
TICENPI_AUTO_PUBLISH_MARKETPLACE
TICENPI_AUTO_PUBLISH_RELIST
TICENPI_AUTO_PUBLISH_DELETE

必須：

- defaults all OFF
- master alone arms nothing
- action ON + master OFF仍不得 real write
- action gates independent
- unknown action fail closed
- no real FB action

## PHASE 6 — RUNTIME CONFIG

驗 Local / Production / runtime-staging compose。

確認：

Local：
- Google OAuth保留
- TICENPI_LOCAL_DEVELOPMENT=1
- TICENPI_REQUIRE_STUDIO=0
- action gates default OFF

Production：
- TICENPI_COMMERCIAL_GATE_ENABLED=1
- TICENPI_REQUIRE_STUDIO=0
- TICENPI_ALLOW_TENANT_KEY=0
- TICENPI_AUTO_PUBLISH default OFF
- all action gates default OFF

runtime-staging：
- action gates present
- default OFF
- existing synthetic/runtime-staging contract不被破壞

確認：

- no secret literal
- config_drift watches master + all action gates

執行三份 compose config validation。

## PHASE 7 — HEALTH / SMOKE

這次必須補真正 shell syntax check。

優先使用可用的：

- Git Bash
- WSL
- Linux container

執行：

bash -n deploy/healthcheck.sh
bash -n deploy/critical-smoke.sh

以及：

python -m py_compile deploy/smoke/post_smoke.py

再跑安全 fixture/mock：

- health happy
- config drift fail
- unreachable fail
- container readiness
- Redis queue smoke
- contract fixture

Health contract 不得依賴：

- Launcher
- Studio heartbeat
- user login
- Entitlement
- Facebook

## PHASE 8 — LOCAL DOCKER RUNTIME

從此 integration source 建 isolated temporary compose project。

不得碰現有 19418 containers。

使用 unused loopback host port。

驗：

- docker build / compose build PASS
- GET /api/health = 200
- GET / = 200

完成後只清自己建立的：

- containers
- network
- temp artifacts

不得 prune shared volumes/system。

## PHASE 9 — SOURCE CLEANNESS

確認：

git status --porcelain
=
empty

確認沒有：

- old conflict artifacts
- dirty-tree files
- quarantined G6 alternative
- secret files

不要 squash。
不要 push。

## DO NOT TOUCH CENTRAL DEPLOY

Central Deploy W2C 已在另一 repo完成：

429898662fb487eb4bafe222dd11e60453a8a204
808fe81e082602c50a4c3770487b1d67b9755612

本 task 不得修改：

F:\00-Ticenpi-SaaS\deploy
或
F:\00-Ticenpi-SaaS\deploy-w2-post

## REQUIRED OUTPUT

```
TASK=W2INT_V2_POST_WAVE2_INTEGRATION

BASE_COMMIT=b3cc93de970f4c7562eff3a4d5e17614462ae8bb

BRANCH=
WORKTREE=
HEAD=

INTEGRATED_COMMITS=
W2A_FIRST=
W2A_SECOND=
W2B=
W2D=
W2BD=
W2E=
W2F=
W2F_FIX=

OWNERSHIP_AUDIT=
PASS/FAIL

CHERRY_PICK_CONFLICTS=

BACKEND_TARGETED=
BACKEND_FULL=
PRE_EXISTING_BACKEND_FAILURES=
NEW_BACKEND_FAILURES=

FRONTEND_TARGETED=
FRONTEND_FULL=
TYPECHECK=
BUILD=
NEW_FRONTEND_FAILURES=

ACCESS_GATE_MATRIX=
PASS/FAIL

PUBLISH_SAFETY_MATRIX=
PASS/FAIL

RUNTIME_CONFIG=
PASS/FAIL

CONFIG_DRIFT=
PASS/FAIL

HEALTH_SMOKE_SYNTAX=
PASS/FAIL

HEALTH_SMOKE_FIXTURES=
PASS/FAIL

LOCAL_DOCKER_BUILD=
PASS/FAIL/PENDING

LOCAL_HEALTH=
PASS/FAIL/PENDING

LOCAL_UI=
PASS/FAIL/PENDING

REAL_FB_ACTION=NO

OLD_CONFLICTED_INTEGRATION_TOUCHED=
NO

CENTRAL_DEPLOY_TOUCHED=
NO

FINAL_WORKTREE_CLEAN=
YES/NO

PRODUCTION_TOUCHED=NO
VPS_TOUCHED=NO

NEXT_BLOCKERS=

READY_FOR_STAGING_BRIDGE=
YES/NO
```
