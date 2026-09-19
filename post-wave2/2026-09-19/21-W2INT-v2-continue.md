# W2INT-v2-CONTINUE — Resume POST Wave2 Integration Validation

- 抬頭：POST WAVE2 INTEGRATION CONTINUATION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中大型
- Mode：Resume Existing Integration, Do Not Restart

---

你現在是：

POST WAVE2 INTEGRATION CONTINUATION OWNER

這不是重新建立 Integration。

你要從先前已建立的 W2INT-v2 worktree/branch 繼續完成尚未完成的 validation phases。

## KNOWN CONTEXT

先前任務：

W2INT-v2 — POST Wave2 Integration Retry

已使用 fresh integration lineage，且另一個 read-only session曾觀察到：

branch:
repair/post-v2-wave2-integration-v2

HEAD 可能為：
e53162f

但這個 HEAD 尚未由正式 REQUIRED OUTPUT 鎖定。

先前執行進度：
- integration commits 已經套用到某個程度
- targeted backend tests 已跑
- full backend suite 執行時工具回傳空白，可能 timeout
- 尚未正式完成 frontend/runtime/health-smoke/local-docker/final-cleanliness 回報

Central Deploy Evidence Bridge 已完成，不屬於本任務：
- branch: repair/post-immutable-deploy
- commit: 808fe81e082602c50a4c3770487b1d67b9755612
- 不要重做
- 不要碰 Central Deploy repo

## TARGET WORKTREE

優先使用既有：

F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2

branch:
repair/post-v2-wave2-integration-v2

## STEP 1 — RESUME STATE VERIFICATION

先只讀確認：

git rev-parse --show-toplevel
git branch --show-current
git rev-parse HEAD
git status --porcelain
git log --oneline --decorate -15

確認：

- branch 正確
- 沒有 CHERRY_PICK_HEAD
- 沒有 MERGE_HEAD
- 沒有 rebase in progress
- worktree clean

如果存在 unresolved git operation：

STOP。

不得自行 abort/reset/resolve。

如果 branch/worktree不存在：

STOP。

不要重建。

記錄：

CURRENT_HEAD=

## STEP 2 — VERIFY EXPECTED INTEGRATION COMMITS PRESENT

確認 current HEAD ancestry中包含 W2INT-v2預期整合成果：

W2A RECUT:
d5a109a75142c23f3373a57480012da6ed476a52
f8c053514992741f953e52a8b68da18120b0740f

W2B:
e8e300ba65f9645b7aa23146e5164f961b438a98

W2D:
a5ec9345fe2912fa2a2dd72862ddba7024ce4c04

W2BD:
26e6fba85355e5ad3a5f68637dc83bfcde1e8289

W2E:
ddedd8c41f47d4dff8c99ad3ccd04b273244845a

W2F:
43186fec18577ec52c897eb498cad9af15d3c51c

W2F-FIX:
65ecfaa75fcabb0c5c97cbdf8c931ad859844975

注意 cherry-pick 後 SHA 可能不同。

以 commit message / patch-id / ancestry內容確認，不要求 SHA 原樣存在。

若缺少任一整合內容：

STOP。

不要自行補 cherry-pick。

## STEP 3 — FULL BACKEND SUITE

先確認沒有殘留 pytest process屬於此 worktree。

如果沒有：

重新跑 full backend suite。

允許較長 timeout，但不要 background fire-and-forget。

記錄：

total passed
total failed
failed test names
duration

歷史已知 baseline failures只有以下可標 PRE_EXISTING：

- test_extract_api golden fixture
- test_frontend_invariants backdrop-filter
- test_invariants playwright scan
- test_invariants DOM selector scan

但必須與 authoritative base / Wave1 behavior一致。

任何新增 backend failure：

STOP。

## STEP 4 — FRONTEND VALIDATION

跑：

targeted SessionGate tests
wave2-access-regression
full npm test
npx tsc --noEmit
npx next build

驗證 access matrix：

signed_out → BLOCK
no_tenant → BLOCK
not_entitled → BLOCK
real auth failure → BLOCK

studio_offline → ALLOW WEB
account_mismatch → ALLOW WEB

backend_unavailable → distinct service unavailable state
unknown reason → fail-safe block

任何新 frontend failure：
STOP。

## STEP 5 — PUBLISH SAFETY VALIDATION

確認：

MASTER:
TICENPI_AUTO_PUBLISH

ACTIONS:
TICENPI_AUTO_PUBLISH_POST
TICENPI_AUTO_PUBLISH_COMMENT
TICENPI_AUTO_PUBLISH_MARKETPLACE
TICENPI_AUTO_PUBLISH_RELIST
TICENPI_AUTO_PUBLISH_DELETE

必須：

- all defaults OFF
- master alone arms nothing
- action ON + master OFF does not write
- actions independent
- unknown action fail closed
- no real Facebook external action

跑 relevant publish gate/driver tests。

## STEP 6 — RUNTIME CONFIG

驗證三個 runtime contracts：

### Local
docker-compose.yml

要求：
- TICENPI_LOCAL_DEVELOPMENT=1
- TICENPI_REQUIRE_STUDIO=0
- Google OAuth identity保留
- master/action gates default OFF

### Production
deploy/staging/compose.yml

要求：
- TICENPI_COMMERCIAL_GATE_ENABLED=1
- TICENPI_REQUIRE_STUDIO=0
- TICENPI_ALLOW_TENANT_KEY=0
- master/action gates default OFF

### Runtime Staging
deploy/runtime-staging/compose.yml

要求：
- action gates存在
- default OFF
- synthetic/runtime-staging contract不被破壞

跑 docker compose config。

確認：

- no secret literal
- config_drift watches master + action gates

## STEP 7 — HEALTH / SMOKE

補完成真正 syntax validation：

bash -n deploy/healthcheck.sh
bash -n deploy/critical-smoke.sh

若 Windows環境原生 bash不可用：

可用 Git Bash / WSL / Linux container。

不要因 CRLF直接 SKIP。

以及：

python -m py_compile deploy/smoke/post_smoke.py

跑安全 fixture/mock：

- health happy
- config drift fail
- unreachable fail
- container readiness
- Redis queue smoke
- contract fixture

Health不得依賴：

Launcher
Studio heartbeat
user login
Entitlement
Facebook

## STEP 8 — LOCAL DOCKER RUNTIME

從 current integration source 建 isolated temporary compose project。

不得碰既有：

19418
Production
VPS
其他 running Post stack

選 unused loopback port。

驗：

- docker compose build / build PASS
- GET /api/health → 200
- GET / → 200

完成後只清理自己建立的：

containers
network
temporary artifacts

禁止：

docker system prune
docker volume prune
刪 shared volumes

## STEP 9 — FINAL CLEANNESS

git status --porcelain 必須 empty。

不得新增 source修改。

若 validation過程產生 temporary/untracked artifacts：

只刪本次產生的 temporary files。

不要 commit validation-only artifacts。

不要 push。

## DO NOT DO

不要：

- 重跑/重做 Central Deploy Evidence Bridge
- 修改 F:\00-Ticenpi-SaaS\deploy
- Production
- VPS write
- GitHub push
- GHCR push
- real FB action
- Supabase mutation

## REQUIRED OUTPUT

```
TASK=W2INT_V2_CONTINUE

BRANCH=
WORKTREE=
CURRENT_HEAD=

GIT_OPERATION_IN_PROGRESS=
YES/NO

EXPECTED_INTEGRATION_CONTENT_PRESENT=
YES/NO

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

TEMPORARY_COMPOSE_PROJECT=
TEMPORARY_HOST_PORT=

LOCAL_HEALTH=
PASS/FAIL/PENDING

LOCAL_UI=
PASS/FAIL/PENDING

REAL_FB_ACTION=NO

CENTRAL_DEPLOY_TOUCHED=NO

FINAL_WORKTREE_CLEAN=
YES/NO

PRODUCTION_TOUCHED=NO
VPS_TOUCHED=NO

NEXT_BLOCKERS=

READY_FOR_STAGING_BRIDGE=
YES/NO
```
