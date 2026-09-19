# W2E — POST Health / Smoke Candidate Port

- 抬頭：POST WAVE2 HEALTH / SMOKE PORT OWNER
- 模型：MiMo-V2.5
- 預計時間：5–10 分鐘
- 任務類型：Port + Revalidation
- Authoritative Base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`
- Candidate Commit：`641874c`

---

你現在是：POST WAVE2 HEALTH / SMOKE PORT OWNER

不是重新實作。

## BASE

Exact Wave2 base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

建立：
- branch: `repair/post-health-smoke-v2`
- worktree: `F:\00-Ticenpi-SaaS\TicenpiPost-w2-health-smoke`

## CANDIDATE

Exact candidate commit：`641874c`

不要使用舊 branch 當 base。

## TASK

從 fresh Wave2 base 執行：

`git cherry-pick 641874c`

如果 conflict：STOP。

禁止 ours、theirs、manual conflict resolution、rebase、reset。

## EXPECTED FILES

只能是：
- `deploy/healthcheck.sh`
- `deploy/critical-smoke.sh`
- `deploy/smoke/post_smoke.py`

如果更多：STOP。

## VALIDATE

重新執行：
- `bash -n`
- Python `py_compile`
- health happy path
- health drift fail path
- unreachable fail path
- container readiness
- Redis queue smoke
- contract fixture

## HEALTH CONTRACT

不得依賴 Launcher、Studio heartbeat、user login、Entitlement、Facebook。

## CENTRAL DEPLOY

不要修改 `F:\00-Ticenpi-SaaS\deploy`。

以下 registration 留後續 bridge：
- `services.yaml`
- `server/deploy.sh`
- `server/audit.sh`
- `server/critical-smoke.sh`

## REQUIRED OUTPUT

```
TASK=W2E_HEALTH_SMOKE_PORT
BASE_COMMIT=
SOURCE_COMMIT=641874c
BRANCH=
WORKTREE=
COMMIT=
CHERRY_PICK_CONFLICT=YES/NO
FILES_CHANGED=
VALIDATIONS=
LAUNCHER_DEPENDENCY=NO
FB_EXTERNAL_ACTION=NO
CENTRAL_DEPLOY_TOUCHED=NO
READY_FOR_INTEGRATION=YES/NO
```
