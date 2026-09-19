# W2C-AUTH — Secure GHCR Credential Transport to VPS Deploy Process

- 抬頭：CENTRAL DEPLOY GHCR CREDENTIAL TRANSPORT OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- Start commit：39ecc94fdaed5cd2950b277cc15d1b39da28e6c1
- Worktree：F:\00-Ticenpi-SaaS\deploy-w2-post

---

你現在是：

CENTRAL DEPLOY GHCR CREDENTIAL TRANSPORT OWNER

這是 W2C 的 non-overlap follow-up。

已知 preflight：

- Central Secret Store：
  F:\Secrets\ticenpi.env
- key：
  GHCR_READ_TOKEN
  GHCR_USERNAME
- VPS：
  147.224.255.234
- VPS sshd AcceptEnv 只有 LANG / LC_*
- sudoers env_reset，沒有 GHCR env_keep
- VPS 沒有 GHCR secret file / systemd EnvironmentFile
- immutable_image.py / server/deploy.sh 已定義 GHCR env contract
- token必須用 docker login --password-stdin
- 目前 deploy.ps1 完全沒有把 GHCR_* 傳到 VPS

因此 immutable deploy會在 GHCR auth-check fail closed。

## START STATE

Worktree：

F:\00-Ticenpi-SaaS\deploy-w2-post

branch：

repair/post-immutable-deploy

HEAD 必須：

39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

working tree 必須 clean。

否則 STOP。

## SHARED-WRITER BOUNDARY

Central Deploy main目前 DM/ORC dirty，包含 shared files：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml

因此本 task禁止修改上述檔案。

## WRITE OWNERSHIP

優先只允許修改：

- deploy.ps1
- dedicated tests / helper

若要安全完成 credential transport 必須修改 server/deploy.sh：

STOP。

輸出：

CROSS_WRITER_REQUIRED=server/deploy.sh + exact reason

不要越界。

## GOAL

建立：

Central Secret Store
→ local deploy.ps1 process
→ SSH transport
→ remote deploy process environment
→ ghcr-auth-check
→ docker login --password-stdin

完整安全鏈。

要求：

- token不在 CLI argv
- token不在 process command line
- token不 echo
- token不寫 release evidence
- token不寫 manifest
- token不寫普通 log
- token不需要 sshd AcceptEnv
- token不依賴 sudoers env_keep
- 預設不持久化到 VPS filesystem
- remote process結束後 credential不留在 shell profile/systemd/env file
- missing token fail closed before pull/deploy mutation

## STEP 1 — INSPECT ACTUAL SSH EXECUTION PATH

先讀 deploy.ps1：

確認目前：

- ssh command怎麼組
- remote deploy.sh如何被呼叫
- 是否經 sudo
- stdin目前是否有其他用途
- scp/ssh helper是否會 log arguments
- PowerShell transcript/log是否會包含 stdin payload

不要猜。

如果現有 transport架構不允許安全 stdin credential envelope：

STOP並提出最小可行 prerequisite。

## STEP 2 — PREFERRED TRANSPORT

優先設計「stdin credential envelope / ephemeral process env」。

例如概念：

local deploy.ps1 presence-only load secrets
→ SSH stdin傳送
→ remote wrapper read secret into env
→ exec deployment process

但不要機械照抄。

必須依現有 SSH helper/privilege model選擇實作。

禁止把 token：

- 拼進 ssh remote command string
- 放在 -ArgumentList
- 放在 URL
- 放在 command-line env assignment
- 寫進 temp .env unless no safer option and task stops for approval

## STEP 3 — CENTRAL SECRET STORE

若 repo已有標準 loader：

沿用。

若沒有：

只實作最低必要 parser，且：

- 不輸出值
- 不輸出長度
- 不輸出 prefix/hash
- duplicate key / malformed line fail closed
- GHCR_READ_TOKEN required
- GHCR_USERNAME optional with existing default behavior

不得修改 F:\Secrets\ticenpi.env。

## STEP 4 — DRY-RUN CONTRACT

Dry-run不得需要真 token。

可以使用 fixture placeholder確認：

- secret走 stdin channel
- argv不含 secret
- rendered command/log不含 secret

真實 deploy mode：

missing secret必須在任何 SSH/deploy mutation前 STOP。

## STEP 5 — TESTS

至少：

1. GHCR_READ_TOKEN present → transport plan PASS
2. missing token → FAIL CLOSED before SSH mutation
3. username absent → existing safe default behavior
4. token never appears in argv
5. token never appears in rendered logs
6. token never written to evidence/manifest
7. stdin payload only reaches intended remote wrapper/process
8. malformed secret store → FAIL
9. legacy-source mode不被破壞
10. immutable dry-run仍不執行 remote mutation

不執行：

- real docker login
- real GHCR pull
- SSH mutation
- VPS deploy
- push

## COMMIT

若可只改 non-shared files完成：

fix(deploy): securely transport ghcr credentials

不要 push。

## REQUIRED OUTPUT

```
TASK=W2C_GHCR_CREDENTIAL_TRANSPORT

START_COMMIT=39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

BRANCH=
WORKTREE=
COMMIT=

FILES_CHANGED=

SECRET_SOURCE=

TRANSPORT_MODE=

GHCR_READ_TOKEN_REQUIRED=
YES/NO

GHCR_USERNAME_OPTIONAL=
YES/NO

TOKEN_IN_ARGV=
YES/NO

TOKEN_IN_LOG=
YES/NO

TOKEN_PERSISTED_ON_VPS=
YES/NO

SSHD_ACCEPTENV_REQUIRED=
YES/NO

SUDO_ENV_KEEP_REQUIRED=
YES/NO

MISSING_TOKEN_FAIL_CLOSED_BEFORE_REMOTE_MUTATION=
PASS/FAIL

LEGACY_SOURCE_MODE_PRESERVED=
YES/NO

SHARED_DIRTY_FILES_MODIFIED=
NO

CROSS_WRITER_REQUIRED=

REMOTE_ACTION=
NO

TESTS=

READY_FOR_VPS_DEPLOYER_ROLLOUT=
YES/NO
```
