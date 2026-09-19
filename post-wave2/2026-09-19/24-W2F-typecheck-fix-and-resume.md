# W2F-FIX2 — Fix Wave2 Typecheck Regression and Resume Integration Validation

- 抬頭：POST WAVE2 TYPECHECK FIX + VALIDATION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- Current integration HEAD：e53162ff4a361b8a4608835bfc1b4e10672370d8

---

你現在是：

POST WAVE2 TYPECHECK FIX + VALIDATION OWNER

這是針對 W2INT-v2 唯一新增 frontend failure 的最小修正，然後繼續完成原本 STEP 4–9。

不是重新整合。
不是重新設計。
不是修改 production source。

## CURRENT VERIFIED STATE

Branch：
repair/post-v2-wave2-integration-v2

Worktree：
F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2

Current HEAD：
e53162ff4a361b8a4608835bfc1b4e10672370d8

Worktree clean。

Backend full：
642 passed / 4 pre-existing failures / 0 new failures。

Frontend：
- targeted tests PASS
- full tests PASS
- build PASS
- typecheck FAIL

唯一新增 failure：

frontend/tests/wave2-access-regression.test.tsx:11

TS2345:
Argument of type '"idle"' is not assignable to parameter of type 'SessionStatus'.

Actual SessionStatus：
'loading' | 'signed-in' | 'signed-out'

此 test 來自 Wave2 regression commit，base typecheck 是 PASS。

## STEP 1 — VERIFY STATE

先確認：

git branch --show-current
git rev-parse HEAD
git status --porcelain

必須：

branch = repair/post-v2-wave2-integration-v2
HEAD = e53162ff4a361b8a4608835bfc1b4e10672370d8
worktree clean
no merge/cherry-pick/rebase operation in progress

若不符：
STOP。

## STEP 2 — MINIMAL TEST-ONLY FIX

只允許修改：

frontend/tests/wave2-access-regression.test.tsx

修正 invalid mock：

'idle'

改成符合實際 SessionStatus contract 的合法值。

優先使用：

'signed-out'

但必須先確認該 test case 的 intent。

如果該 case 其實需要其他合法狀態：
使用與測試語意一致的合法 SessionStatus。

禁止修改：

- frontend/src/**
- backend/**
- deploy/**
- docker-compose.yml
- extension/**
- tsconfig.json
- package.json

這是一個 test-only fix。

## STEP 3 — VERIFY FIX

至少跑：

npx tsc --noEmit

以及：

targeted wave2-access-regression test

要求：

TYPECHECK=PASS

若仍有任何新增 TS error：
STOP。

## STEP 4 — COMMIT

Commit：

test(post): fix wave2 access regression session status

不要 push。

記錄：

FIX_COMMIT=

## STEP 5 — RESUME FRONTEND VALIDATION

重新跑：

- frontend/tests/session-gate.test.tsx
- frontend/tests/wave2-access-regression.test.tsx
- full npm test
- npx tsc --noEmit
- npx next build

Access matrix 必須：

signed_out → BLOCK
no_tenant → BLOCK
not_entitled → BLOCK
401/real auth failure → BLOCK
403 → not_entitled block

studio_offline → ALLOW WEB
account_mismatch → ALLOW WEB

backend_unavailable → distinct service unavailable
unknown reason → fail-safe block

若目前 unknown reason 沒有 dedicated unit case：
新增一個 test-only regression case於同一 test file即可。

不得改 production source。

## STEP 6 — PUBLISH SAFETY

驗證：

MASTER:
TICENPI_AUTO_PUBLISH

ACTIONS:
TICENPI_AUTO_PUBLISH_POST
TICENPI_AUTO_PUBLISH_COMMENT
TICENPI_AUTO_PUBLISH_MARKETPLACE
TICENPI_AUTO_PUBLISH_RELIST
TICENPI_AUTO_PUBLISH_DELETE

要求：

- defaults all OFF
- master alone arms nothing
- action ON + master OFF no write
- each action independent
- unknown action fail closed
- no real Facebook action

跑 relevant publish gate/driver tests。

## STEP 7 — RUNTIME CONFIG

驗：

Local:
docker-compose.yml

Production:
deploy/staging/compose.yml

Runtime staging:
deploy/runtime-staging/compose.yml

確認：

Local:
- TICENPI_LOCAL_DEVELOPMENT=1
- TICENPI_REQUIRE_STUDIO=0
- action gates OFF

Production:
- TICENPI_COMMERCIAL_GATE_ENABLED=1
- TICENPI_REQUIRE_STUDIO=0
- TICENPI_ALLOW_TENANT_KEY=0
- master/action gates OFF

Runtime staging:
- action gates present
- default OFF

跑 docker compose config x3。

確認 config_drift watches master + action gates。

## STEP 8 — HEALTH / SMOKE

跑：

bash -n deploy/healthcheck.sh
bash -n deploy/critical-smoke.sh

以及：

python -m py_compile deploy/smoke/post_smoke.py

若 Windows原生 bash不可用：
使用 Git Bash / WSL / Linux container。

不得因 CRLF直接 SKIP。

跑 safe fixture/mock：

- health happy
- config drift fail
- unreachable fail
- container readiness
- Redis queue smoke
- contract fixture

Health不得依賴：

Launcher
Studio heartbeat
login
Entitlement
Facebook

## STEP 9 — LOCAL DOCKER

從 current integration HEAD 建 isolated temporary compose project。

不得碰 existing 19418 stack。

使用 unused loopback port。

驗：

- docker build / compose build PASS
- GET /api/health = 200
- GET / = 200

完成後只清自己建立的 temporary containers/network/artifacts。

禁止 prune shared resources。

## STEP 10 — FINAL CLEANNESS

git status --porcelain 必須 empty。

只有允許的 test-only fix commit可以新增。

不要 push。

## DO NOT TOUCH

Central Deploy：
- F:\00-Ticenpi-SaaS\deploy
- F:\00-Ticenpi-SaaS\deploy-w2-post

不要碰。

Production/VPS/Supabase/real Facebook 也不要碰。

## REQUIRED OUTPUT

```
TASK=W2F_FIX2_AND_W2INT_CONTINUE

START_HEAD=e53162ff4a361b8a4608835bfc1b4e10672370d8

BRANCH=
WORKTREE=

FILES_CHANGED=

INVALID_STATUS_BEFORE=idle
VALID_STATUS_AFTER=

FIX_COMMIT=

TYPECHECK_AFTER_FIX=
PASS/FAIL

UNKNOWN_REASON_TEST_ADDED=
YES/NO

FRONTEND_TARGETED=
FRONTEND_FULL=
TYPECHECK=
BUILD=

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

FINAL_HEAD=
FINAL_WORKTREE_CLEAN=
YES/NO

PRODUCTION_TOUCHED=NO
VPS_TOUCHED=NO

NEXT_BLOCKERS=

READY_FOR_STAGING_BRIDGE=
YES/NO
```
