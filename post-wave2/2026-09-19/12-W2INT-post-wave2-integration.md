# W2INT — POST Wave2 Integration

- 抬頭：POST WAVE2 INTEGRATION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：大型
- Base：b3cc93de970f4c7562eff3a4d5e17614462ae8bb

---

你現在是：

POST WAVE2 INTEGRATION OWNER

這是正式 Integration。

不是重新設計，也不是重新開發。

## AUTHORITATIVE BASE

Wave2 base：
branch: `repair/post-v2-wave2-base`
commit: `b3cc93de970f4c7562eff3a4d5e17614462ae8bb`
worktree: `F:\00-Ticenpi-SaaS\TicenpiPost-wave2-base`

## APPROVED CANDIDATES

W2A:
`0030b44243e301fd89325d187264ab21df12929a`

W2B:
`e8e300ba65f9645b7aa23146e5164f961b438a98`

W2D:
`a5ec9345fe2912fa2a2dd72862ddba7024ce4c04`

W2BD:
`26e6fba85355e5ad3a5f68637dc83bfcde1e8289`

W2E:
`ddedd8c41f47d4dff8c99ad3ccd04b273244845a`

W2F:
`43186fec18577ec52c897eb498cad9af15d3c51c`

W2F-FIX:
`65ecfaa75fcabb0c5c97cbdf8c931ad859844975`

## INTEGRATION ORDER

使用 exact commits，順序固定：

1. 0030b44243e301fd89325d187264ab21df12929a
2. e8e300ba65f9645b7aa23146e5164f961b438a98
3. a5ec9345fe2912fa2a2dd72862ddba7024ce4c04
4. 26e6fba85355e5ad3a5f68637dc83bfcde1e8289
5. ddedd8c41f47d4dff8c99ad3ccd04b273244845a
6. 43186fec18577ec52c897eb498cad9af15d3c51c
7. 65ecfaa75fcabb0c5c97cbdf8c931ad859844975

不要 merge feature branches。

## CREATE

建立：
branch: `repair/post-v2-wave2-integration`
worktree: `F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration`

必須直接從 exact base `b3cc93de970f4c7562eff3a4d5e17614462ae8bb` 建立。

確認 HEAD exact match 且 worktree clean，否則 STOP。

## HARD RULES

禁止 merge feature branches、rebase、reset、stash、clean、bulk port dirty tree、push、Production/VPS write、Supabase write、real Facebook action。

任何 cherry-pick conflict：STOP。
禁止 ours/theirs/manual conflict resolution。

## OWNERSHIP QUICK CHECK

確認：
- W2A = SessionGate + dedicated tests
- W2B = publish gate / worker / driver + tests
- W2D = Post runtime compose/env examples
- W2BD = docker-compose/env wiring + config_drift/tests
- W2E = deploy/healthcheck.sh + deploy/critical-smoke.sh + deploy/smoke/post_smoke.py
- W2F = Wave2 tests only

若越界：STOP。

## BACKEND TESTS

至少跑：
- test_publish_action_gates.py
- test_publish_runtime_wiring.py
- test_config_drift.py
- Wave2 backend regression tests
- Wave1 auth / identity / commercial / studio suites
- relevant driver tests

然後 full backend suite。

只有與 base/Wave1 完全相同的 historical 4 failures可標 PRE_EXISTING：
- extract_api golden fixture
- frontend_invariants backdrop-filter
- invariant playwright scan
- invariant DOM selector scan

任何新增 failure：STOP。

## FRONTEND

跑：
- SessionGate targeted tests
- wave2-access-regression
- full frontend tests
- tsc --noEmit
- next build

驗：
- signed_out → BLOCK
- no_tenant → BLOCK
- not_entitled → BLOCK
- real auth failure → BLOCK
- studio_offline → ALLOW WEB
- account_mismatch → ALLOW WEB
- backend_unavailable → distinct service unavailable state

## PUBLISH SAFETY

驗：
master = TICENPI_AUTO_PUBLISH

actions:
- TICENPI_AUTO_PUBLISH_POST
- TICENPI_AUTO_PUBLISH_COMMENT
- TICENPI_AUTO_PUBLISH_MARKETPLACE
- TICENPI_AUTO_PUBLISH_RELIST
- TICENPI_AUTO_PUBLISH_DELETE

要求：
- default all OFF
- master alone does NOT arm actions
- action alone + master OFF does NOT write
- actions independent
- unknown action fail closed
- no real Facebook action

## RUNTIME CONFIG

驗 Local / Production / runtime-staging compose：

- Local Google OAuth保留
- Local require_studio=0
- Production commercial gate=1
- Production require_studio=0
- Production allow_tenant_key=0
- auto publish default off
- action gates default off
- no secret literal
- config_drift watches master + all action gates

跑 docker compose config 對三個 runtime contract。

## HEALTH / SMOKE

W2E Windows port 時 bash -n曾 SKIP。

Integration 必須補真實 shell syntax validation，優先 Git Bash / WSL / Linux container：

`bash -n deploy/healthcheck.sh`
`bash -n deploy/critical-smoke.sh`
`python -m py_compile deploy/smoke/post_smoke.py`

並跑安全 mock/fixture：
- health happy
- config drift fail
- unreachable fail
- container readiness
- Redis queue smoke
- contract fixture

Health不得依賴 Launcher、Studio、login、Entitlement、Facebook。

## LOCAL DOCKER

從 Wave2 integration source 建 isolated temporary compose project。

不得碰現有 19418 container。

使用 unused loopback port。

驗：
- build PASS
- /api/health = 200
- / = 200

完成後只清理自己的 containers/network/temporary artifacts。

## FINAL

git status 必須 clean。

不要 squash。
不要 push。

## REQUIRED OUTPUT

```
TASK=W2INT_POST_WAVE2_INTEGRATION

BASE_COMMIT=b3cc93de970f4c7562eff3a4d5e17614462ae8bb

BRANCH=
WORKTREE=
HEAD=

INTEGRATED_COMMITS=
W2A=
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

FINAL_WORKTREE_CLEAN=
YES/NO

PRODUCTION_TOUCHED=NO
VPS_TOUCHED=NO

NEXT_BLOCKERS=

READY_FOR_STAGING_BRIDGE=
YES/NO
```
