# W2C — Central Deploy Immutable Image Delivery

- 抬頭：TICENPI CENTRAL DEPLOY IMMUTABLE IMAGE DELIVERY OWNER
- 模型：DeepSeek V4.1 Flash
- 預計時間：25–40 分鐘
- 任務類型：Implementation
- Repo：`F:\00-Ticenpi-SaaS\deploy`

---

你現在是：

TICENPI CENTRAL DEPLOY IMMUTABLE IMAGE DELIVERY OWNER

這是正式 Implementation。

## REPOSITORY

Central Deploy repo：`F:\00-Ticenpi-SaaS\deploy`

注意：不是 TicenpiPost repo。

不要使用 `repair/post-v2-wave2-base` 作為 git base。

## PRECONDITION

先確認：

`git -C F:\00-Ticenpi-SaaS\deploy status --porcelain`

必須 empty。

記錄 CURRENT_BRANCH、DEPLOY_BASE_COMMIT。

同時確認目前沒有其他 Agent 正在修改 Central Deploy repo。

如果 dirty 或存在另一 active writer，立即 STOP，輸出：

`CENTRAL_DEPLOY_WRITER_CONFLICT=YES`

不要 pull、reset、rebase。

## CREATE

從 DEPLOY_BASE_COMMIT 建 isolated worktree：

`F:\00-Ticenpi-SaaS\deploy-w2-post`

branch：

`repair/post-immutable-deploy`

## EXCLUSIVE OWNERSHIP

本任務只允許修改：
- `deploy.ps1`
- `server/deploy.sh`
- 確實必要的新 immutable-artifact helper

只讀：
- `services.yaml`
- `server/audit.sh`
- `server/critical-smoke.sh`

本任務不修改 `services.yaml`。

## CURRENT PROBLEM

舊 Docker path：

source/package → VPS → docker compose --build

無法保證 CI artifact == Staging artifact == Production artifact。

## TARGET

新增 `IMMUTABLE_IMAGE_MODE`。

輸入 release evidence 以及：

`ghcr.io/.../ticenpi-post@sha256:<64hex>`

驗證後 VPS pull exact immutable reference，然後 deploy without build。

## LEGACY COMPATIBILITY

其他產品仍可能依賴 source deploy。

所以 `LEGACY_SOURCE_MODE` 必須保留，不得直接移除舊路徑。

## FAIL CLOSED BEFORE MUTATION

以下任一情況：
- missing image ref
- not `@sha256`
- invalid sha256
- wrong service
- evidence mismatch
- unresolvable artifact

都必須在任何 remote mutation 前停止。

## PRESERVE

保留：
- release directory
- current symlink
- rollback
- manifest snapshot
- preflight
- ApprovedPackage verification

## NO REMOTE ACTION

禁止 SSH、VPS write、Production deploy、restart、GHCR push、GitHub push。

只做 local fixture、mock、dry-run、static/script validation。

## DO NOT MODIFY

不要修改 TicenpiPost source、Post compose、Post health scripts。

不要處理 AUTO_PUBLISH、Studio、CommercialGate、Sign。

## COMMIT

`feat(deploy): support immutable image delivery`

不要 push。

## REQUIRED OUTPUT

```
TASK=W2C_IMMUTABLE_DEPLOY
DEPLOY_BASE_COMMIT=
BRANCH=
WORKTREE=
COMMIT=
FILES_CHANGED=
LEGACY_SOURCE_MODE_PRESERVED=YES/NO
IMMUTABLE_IMAGE_MODE=PASS/FAIL
EXACT_DIGEST_REQUIRED=YES/NO
VPS_BUILD_IN_IMMUTABLE_MODE=YES/NO
FAIL_CLOSED_BEFORE_MUTATION=PASS/FAIL
ROLLBACK_COMPATIBLE=YES/NO
SERVICES_YAML_CHANGED=NO
REMOTE_ACTION=NO
CENTRAL_DEPLOY_WRITER_CONFLICT=YES/NO
FOLLOWUP_BRIDGE_REQUIRED=
READY_FOR_INTEGRATION=YES/NO
```
