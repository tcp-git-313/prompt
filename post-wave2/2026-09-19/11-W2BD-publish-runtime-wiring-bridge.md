# W2BD-BRIDGE — Publish Action Gate Runtime Wiring

- 抬頭：POST WAVE2 PUBLISH RUNTIME WIRING OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- Wave2 Base：b3cc93de970f4c7562eff3a4d5e17614462ae8bb
- W2B Candidate：e8e300ba65f9645b7aa23146e5164f961b438a98
- W2D Candidate：a5ec9345fe2912fa2a2dd72862ddba7024ce4c04

---

你現在是：

POST WAVE2 PUBLISH RUNTIME WIRING OWNER

這是 W2B + W2D 完成後的 serial bridge。

不是重新設計 Publish Safety。

## GOAL

W2B 已新增 action-level safety gates：

- TICENPI_AUTO_PUBLISH_POST
- TICENPI_AUTO_PUBLISH_COMMENT
- TICENPI_AUTO_PUBLISH_MARKETPLACE
- TICENPI_AUTO_PUBLISH_RELIST
- TICENPI_AUTO_PUBLISH_DELETE

目前 source 已 fail-closed。

但 runtime 尚未完整 wiring，導致：

master TICENPI_AUTO_PUBLISH 單獨開啟時，
所有 action 仍保持 dry-run。

這在安全上正確，但在正式可運作上尚未完整。

本 task 負責：

1. 將 5 個 action gate env 正確 wire 到 worker runtime。
2. Local / runtime-staging / Production 都維持 explicit + fail-closed。
3. config drift 能偵測這些 worker gate。
4. 不允許任何 action 預設 ON。

## BASE / BRANCH

從 exact Wave2 base：

b3cc93de970f4c7562eff3a4d5e17614462ae8bb

建立 fresh worktree：

F:\00-Ticenpi-SaaS\TicenpiPost-w2-publish-runtime

branch：

repair/post-publish-runtime-wiring

然後依序 cherry-pick：

1. e8e300ba65f9645b7aa23146e5164f961b438a98
2. a5ec9345fe2912fa2a2dd72862ddba7024ce4c04

如果任一 conflict：

STOP。

禁止自行 ours/theirs。

## OWNERSHIP FOR THIS BRIDGE

因為這是已核准的 serial bridge，允許修改最低必要的：

- root docker-compose.yml（Local worker env wiring）
- deploy/staging/compose.yml
- deploy/runtime-staging/compose.yml
- deploy/staging/.env.example
- deploy/runtime-staging/.env.example
- root .env.example（若存在且確實是 Local env contract）
- backend/app/core/config_drift.py
- dedicated regression tests for config drift / env wiring

不要修改：

- publish_gate.py semantics
- executor/driver business logic
- auth
- identity
- studio
- frontend
- extension
- Central Deploy repo

## FROZEN SAFETY CONTRACT

Master gate：

TICENPI_AUTO_PUBLISH

Action gates：

TICENPI_AUTO_PUBLISH_POST
TICENPI_AUTO_PUBLISH_COMMENT
TICENPI_AUTO_PUBLISH_MARKETPLACE
TICENPI_AUTO_PUBLISH_RELIST
TICENPI_AUTO_PUBLISH_DELETE

所有 action gate default：

OFF / empty / false

不得：

- default 1
- default true
- fallback to master value
- master ON 自動隱式 arm 所有 actions

## RUNTIME TARGET

### Local

Local Google OAuth / membership bootstrap / no Studio product gate 維持不變。

只把 action gate variables 傳入 worker。

Local 預設仍 fail-closed。

### runtime-staging

action gate variables 必須存在於 worker env contract。

預設 OFF。

### Production

action gate variables 必須存在於 worker env contract。

預設 OFF。

Production 是否實際 enable 任一 action 是後續 staging / release approval 決策。

本 task 不啟用。

## CONFIG DRIFT

檢查：

backend/app/core/config_drift.py

將實際會影響 worker external-write behavior 的 5 個 action vars 納入對應 worker watched keys。

不得加入不相關設定。

## ENV EXAMPLES

更新對應 .env.example：

加入 5 個 action vars，值保持空 / 0 / false 的安全範例。

不得放任何 secret。

## VALIDATION

至少：

1. docker compose config — Local
2. docker compose config — Production compose
3. docker compose config — runtime-staging compose
4. 無 action vars時全部 OFF
5. 單獨 POST=1 不影響 COMMENT / MARKETPLACE / RELIST / DELETE
6. master=0 時即使 action=1 仍不得 external write
7. config drift watched keys 包含 5 個 action gates
8. secret scan
9. git diff --check
10. relevant backend tests

不要 real Facebook action。

## IMPORTANT

若 W2B 和 W2D cherry-pick 後出現 source collision：
STOP。

若需修改 W2C / Central Deploy：
CROSS_REPO_BRIDGE_REQUIRED
不要越界。

## COMMIT

fix(post): wire publish action gates into runtime

不要 push。

## REQUIRED OUTPUT

```
TASK=W2BD_PUBLISH_RUNTIME_WIRING

BASE_COMMIT=b3cc93de970f4c7562eff3a4d5e17614462ae8bb

W2B_SOURCE_COMMIT=e8e300ba65f9645b7aa23146e5164f961b438a98
W2D_SOURCE_COMMIT=a5ec9345fe2912fa2a2dd72862ddba7024ce4c04

BRANCH=
WORKTREE=

CHERRY_PICK_W2B=
PASS/FAIL
CHERRY_PICK_W2D=
PASS/FAIL

BRIDGE_COMMIT=

FILES_CHANGED=

LOCAL_WORKER_ACTION_ENV=
PASS/FAIL

PRODUCTION_WORKER_ACTION_ENV=
PASS/FAIL

RUNTIME_STAGING_WORKER_ACTION_ENV=
PASS/FAIL

ACTION_DEFAULT_FAIL_CLOSED=
PASS/FAIL

MASTER_ALONE_ARMS_ACTIONS=
NO

CONFIG_DRIFT_WATCHED_KEYS=
PASS/FAIL

ENV_EXAMPLES_UPDATED=
PASS/FAIL

SECRET_LITERAL_FOUND=
YES/NO

TESTS=

REAL_FB_ACTION=
NO

CENTRAL_DEPLOY_TOUCHED=
NO

CROSS_REPO_BRIDGE_REQUIRED=

READY_FOR_POST_WAVE2_INTEGRATION=
YES/NO
```
