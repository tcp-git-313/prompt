# W2C-VPS-PREP — Immutable Deployer VPS Rollout Plan

- 抬頭：CENTRAL DEPLOY VPS CONTROL PLANE ROLLOUT PLANNER
- 模型：MiMo-V2.5
- 預計工程大小：中型
- 模式：STRICT READ ONLY

---

你現在是：

CENTRAL DEPLOY VPS CONTROL PLANE ROLLOUT PLANNER

這個任務只規劃「如何安全把新的 immutable deploy capability 放到 VPS」。

不要實際 rollout。

## VERIFIED CURRENT STATE

VPS：
147.224.255.234

目前：

/opt/ticenpi/deploy/deploy.sh

是舊版 legacy deployer，沒有：

- immutable-image mode
- GHCR auth flow
- immutable_image.py

VPS目前 post runtime staging：
- port 19419
- compose project ticenpi-post-runtime-staging
- current release存在
- legacy source stack正常運作

Local frozen W2C lineage：

F:\00-Ticenpi-SaaS\deploy-w2-post
branch repair/post-immutable-deploy

commits：

429898662fb487eb4bafe222dd11e60453a8a204
808fe81e082602c50a4c3770487b1d67b9755612
39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

另有 GHCR credential transport task可能產生後續 commit。

Central Deploy main目前有 DM/ORC dirty shared files。

## GOAL

設計一個不會覆蓋/遺失 DM/ORC shared changes的 control-plane rollout。

必須回答：

1. 現在 /opt/ticenpi/deploy 是怎麼被更新的
2. 是否已有 versioned deployer release / symlink機制
3. 若沒有，最小安全 rollout是：
   - in-place atomic replacement
   - side-by-side candidate dir
   - versioned control-plane release
   哪個較合適
4. 如何驗證新 deployer而不執行 product deployment
5. 如何 rollback舊 deployer
6. 哪些檔案必須等 shared-file bridge完成後才能正式 rollout
7. immutable_image.py / tests / helper要放哪
8. GHCR credential transport與 deployer rollout的 dependency

## HARD RULES

STRICT READ ONLY。

禁止：

- scp
- rsync
- file write
- chmod/chown
- symlink mutation
- systemctl mutation
- deploy
- docker login
- docker pull
- service restart
- git commit/push
- Central Deploy main modification

只允許：

- local source read
- git read-only commands
- SSH read-only commands
- metadata/hash/stat inspection

## STEP 1 — INSPECT LOCAL DEPLOY INSTALL MECHANISM

找出 Central Deploy如何把：

deploy.ps1
server/deploy.sh
server/audit.sh
server/critical-smoke.sh
helpers

送到：

/opt/ticenpi/deploy

確認是否有：

- bootstrap script
- self-update
- platform deployment service
- rsync/scp step
- install manifest
- version marker
- hash validation
- rollback copy

不得猜。

## STEP 2 — INSPECT VPS CONTROL PLANE

Read-only查看：

/opt/ticenpi/deploy

包括：

- files
- symlinks
- ownership/mode
- version markers
- timestamps
- checksums
- backup dirs
- references from systemd/cron/deploy tooling

不要讀 secret values。

## STEP 3 — COMPARE LOCAL FROZEN W2C VS VPS

列出至少：

- deploy.sh
- immutable_image.py
- audit.sh
- critical-smoke.sh
- rollback/preflight helper
- services.yaml if installed separately

哪些：

- SAME
- DIFFERENT
- MISSING_ON_VPS

注意 DM/ORC dirty main不等於 frozen W2C branch。

## STEP 4 — ROLLOUT STRATEGY

優先：

side-by-side/versioned candidate
→ syntax/unit/dry-run validation
→ atomic cutover only after shared bridge + auth transport PASS

如果現有架構不支援 side-by-side，定義最小 backup/atomic replace。

要求：

- 不影響目前 post runtime-staging
- 不 restart app containers
- 不 deploy product
- rollback能在 deployer validation失敗時立刻回舊版
- current legacy deployment path保持可用

## STEP 5 — VALIDATION PLAN

新 deployer rollout後、真正 Staging deploy前，至少驗：

- bash -n
- python helper --help / validation fixture
- deploy.sh legacy dry-run
- deploy.sh immutable dry-run
- GHCR auth-check missing secret fail closed
- no product container change
- version/hash evidence

## STEP 6 — DEPENDENCY GATES

列出 rollout前必須完成：

- W2C shared-file bridge?
- W2C GHCR credential transport?
- W2G CI evidence merge?
- W2INT final validation?

區分：

MUST_HAVE_BEFORE_CONTROL_PLANE_ROLLOUT
vs
MUST_HAVE_BEFORE_FIRST_IMMUTABLE_STAGING_DEPLOY

## OUTPUT

```
TASK=W2C_VPS_DEPLOYER_ROLLOUT_PLAN

MODE=READ_ONLY

CURRENT_VPS_DEPLOY_ROOT=

CURRENT_VPS_DEPLOYER_VERSION_EVIDENCE=

LOCAL_FROZEN_W2C_HEAD=39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

INSTALL_MECHANISM=

VERSIONED_CONTROL_PLANE_SUPPORTED=
YES/NO

RECOMMENDED_ROLLOUT_MODE=

FILES_TO_INSTALL=

FILES_DIFFERENT_OR_MISSING=

ROLLBACK_MODE=

VALIDATION_BEFORE_CUTOVER=

VALIDATION_AFTER_CUTOVER=

MUST_HAVE_BEFORE_CONTROL_PLANE_ROLLOUT=

MUST_HAVE_BEFORE_FIRST_IMMUTABLE_STAGING_DEPLOY=

DM_ORC_SHARED_CHANGE_RISK=

BLOCKERS=

MUTATION_PERFORMED=NO

READY_TO_EXECUTE_CONTROL_PLANE_ROLLOUT_AFTER_GATES=
YES/NO
```
