# STG-PREFLIGHT — VPS / GHCR / Runtime Readiness

- 抬頭：POST STAGING VPS PREFLIGHT OWNER
- 模型：MiMo-V2.5
- 預計工程大小：中型
- 模式：STRICT READ ONLY

---

你現在是：

POST STAGING VPS PREFLIGHT OWNER

這是與 W2INT / W2G / W2C 並行的 Staging host readiness 檢查。

不是 deploy。
不是 mutation。

## GOAL

在真正 immutable staging deploy 前，確認 VPS / credential / runtime 基礎條件是否已準備：

1. postruntimestaging 目標 port / compose project 是否可用
2. Docker / compose / architecture 是否符合 artifact
3. GHCR private image read credential 的「實際注入路徑」是否存在
4. GHCR_READ_TOKEN / GHCR_USERNAME 是否在正確 secret source 中「存在」
5. 不輸出 secret value
6. staging health port 19419 是否被意外佔用
7. rollback/release directories 基礎結構是否存在
8. 不進行 docker login / pull / restart

## KNOWN TARGET

POST runtime staging：

service id:
postruntimestaging

compose project:
ticenpi-post-runtime-staging

expected loopback binding:
127.0.0.1:19419:9418

expected health:
http://127.0.0.1:19419/api/health

VPS：
147.224.255.234

不要假設 public DNS route存在。

## HARD RULES

STRICT READ ONLY。

禁止：

- docker login
- docker pull
- docker compose up/down
- restart
- systemctl restart/start/stop
- file write
- chmod/chown
- secret export
- echo token
- cat secret values
- SSH remote mutation
- deploy.ps1 execution
- deploy.sh execution
- GitHub/GHCR mutation
- Production mutation

## STEP 1 — TRACE CREDENTIAL FLOW

先從 Central Deploy code/readme只讀追蹤：

GHCR_READ_TOKEN
GHCR_USERNAME

實際由哪一層提供：

- local Central Secret Store
- deploy.ps1 process env
- SSH forwarded env
- uploaded env file
- VPS secret file
- systemd EnvironmentFile
- other

不得猜。

輸出 exact presence path / mechanism。

## STEP 2 — PRESENCE-ONLY SECRET CHECK

只確認 key 是否存在。

允許輸出：

GHCR_READ_TOKEN_PRESENT=YES/NO/UNKNOWN
GHCR_USERNAME_PRESENT=YES/NO/UNKNOWN

禁止輸出值、長度、prefix、hash。

如果需要讀 secret store：

只能 presence-only。

## STEP 3 — VPS READ-ONLY CHECK

若既有 SSH access可用，以 read-only commands確認：

- uname -m
- docker version
- docker compose version
- df -h relevant mount
- docker ps metadata only
- listener on 127.0.0.1:19419
- existing postruntimestaging containers / compose labels
- release root directories / current symlink metadata
- permissions metadata必要時可看，但不改

不得讀 container secrets。

## STEP 4 — ARTIFACT PLATFORM

確認 Post CI artifact platform與 VPS architecture是否匹配。

若 CI target arm64、VPS非 arm64：

BLOCKER。

## STEP 5 — PORT / STACK COLLISION

確認：

19419 是否：

- free
- already owned by intended post runtime-staging
- owned by unrelated service

如果 unrelated：

BLOCKER。

不要 stop它。

## STEP 6 — GHCR AUTH EXECUTION PRECONDITION

確認 Central Deploy immutable mode執行到 docker pull之前：

- credential presence check確實會發生
- missing credential fail closed
- token以 password-stdin使用
- token不會進 argv/log

這部分只讀 source + presence evidence。

## OUTPUT

```
TASK=STG_VPS_GHCR_PREFLIGHT

MODE=READ_ONLY

VPS_REACHABLE=
YES/NO

VPS_ARCH=

DOCKER_AVAILABLE=
YES/NO

DOCKER_COMPOSE_AVAILABLE=
YES/NO

CI_ARTIFACT_PLATFORM=

PLATFORM_MATCH=
YES/NO

PORT_19419_STATE=
FREE / INTENDED_POST / COLLISION / UNKNOWN

EXISTING_POST_RUNTIME_STAGING=

RELEASE_ROOT_PRESENT=
YES/NO/UNKNOWN

CURRENT_SYMLINK_STATE=

GHCR_CREDENTIAL_FLOW=

GHCR_READ_TOKEN_PRESENT=
YES/NO/UNKNOWN

GHCR_USERNAME_PRESENT=
YES/NO/UNKNOWN

TOKEN_VALUE_EXPOSED=
NO

GHCR_FAIL_CLOSED_CHECK=
PASS/FAIL/UNKNOWN

PASSWORD_STDIN_CONTRACT=
PASS/FAIL/UNKNOWN

BLOCKERS=

MUTATION_PERFORMED=NO

READY_FOR_IMMUTABLE_STAGING_DEPLOY=
YES/NO
```
