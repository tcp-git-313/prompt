# W2C-START — Central Deploy Immutable Image Delivery from Isolated Base

- 抬頭：TICENPI CENTRAL DEPLOY IMMUTABLE IMAGE DELIVERY OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：大型
- Repository：F:\00-Ticenpi-SaaS\deploy
- SAFE_BASE_COMMIT：216f709cde8ce0d95b60f120997e13c650029a1a

---

你現在是：

TICENPI CENTRAL DEPLOY IMMUTABLE IMAGE DELIVERY OWNER

這是正式 Implementation。

## VERIFIED PRECONDITION

前一輪 read-only audit 已確認：

CURRENT_BRANCH=main
CURRENT_HEAD=216f709cde8ce0d95b60f120997e13c650029a1a

main working tree 目前 dirty，但 dirty changes 全屬 DM / ORC：

- server/critical-smoke.sh
- services.yaml
- server/warp-proxy-relay-dmruntimestaging.service

W2C target files：

- deploy.ps1
- server/deploy.sh
- 必要 immutable helper

目前都沒有 dirty。

Audit 結論：

ACTIVE_WRITER_OVERLAP=NO
ISOLATED_W2C_BASE_POSSIBLE=YES
CENTRAL_DEPLOY_WRITER_CONFLICT=NO
SAFE_BASE_COMMIT=216f709cde8ce0d95b60f120997e13c650029a1a

因此：

你被授權從 exact SAFE_BASE_COMMIT 建立 isolated worktree。

不要再因 main working tree dirty 而 STOP。

但嚴禁修改、清理、stash、reset main working tree。

## CREATE ISOLATED WORKTREE

建立：

worktree:
F:\00-Ticenpi-SaaS\deploy-w2-post

branch:
repair/post-immutable-deploy

直接從 exact commit：

216f709cde8ce0d95b60f120997e13c650029a1a

建立後確認：

git rev-parse HEAD
=
216f709cde8ce0d95b60f120997e13c650029a1a

git status --porcelain
=
empty

否則 STOP。

## GLOBAL HARD RULES

禁止對 F:\00-Ticenpi-SaaS\deploy main working tree 執行：

- reset
- restore
- clean
- stash
- checkout mutation
- commit
- merge
- rebase

禁止修改 DM / ORC dirty files。

禁止：

- SSH
- VPS write
- Production deploy
- restart
- GitHub push
- GHCR push

## EXCLUSIVE OWNERSHIP

本任務只允許修改：

- deploy.ps1
- server/deploy.sh
- 確實必要的新 immutable-artifact helper

只讀：

- services.yaml
- server/audit.sh
- server/critical-smoke.sh
- current release/deployment contract docs

不得修改 services.yaml。

## CURRENT PROBLEM

現行 Docker deployment default path 仍可能是：

source/package
→ VPS
→ docker compose --build

這無法保證：

CI artifact == Staging artifact == Production artifact。

## TARGET

新增 explicit immutable-image deployment mode。

輸入必須包含：

1. Approved release evidence
2. Exact immutable image reference：

ghcr.io/.../ticenpi-post@sha256:<64hex>

驗證後：

- pull exact immutable reference
- deploy without VPS build
- preserve release identity/evidence
- preserve rollback

## LEGACY COMPATIBILITY

保留既有 LEGACY_SOURCE_MODE。

其他產品尚未全部 migration，不得移除舊 source deployment path。

Post immutable mode 與 legacy mode 必須明確分流。

## FAIL CLOSED BEFORE MUTATION

以下任一情況必須在任何 remote mutation 前拒絕：

- image ref missing
- image ref 不是 @sha256
- digest malformed
- service mismatch
- evidence source SHA mismatch
- evidence artifact digest mismatch
- unsupported deployment mode
- immutable artifact 無法 resolve（測試用 mock / dry-run）

## PRESERVE

不得破壞：

- versioned release directory
- atomic current symlink
- manifest snapshot
- rollback
- preflight
- ApprovedPackage / evidence validation
- existing service compatibility

## VPS BUILD RULE

IMMUTABLE_IMAGE_MODE：

VPS_BUILD=NO

不得執行：

docker compose build
docker build
任何以 VPS source 重建 image 的步驟。

LEGACY_SOURCE_MODE 可保持原行為。

## VALIDATION

只做 local/static/mock/dry-run。

至少驗：

- PowerShell syntax / dry-run path
- shell syntax
- valid exact digest accepted
- tag-only image rejected
- malformed digest rejected
- evidence mismatch rejected
- wrong service rejected
- immutable mode command graph 無 build
- legacy mode still available
- rollback path仍存在

不得真正 SSH。

## COMMIT

feat(deploy): support immutable image delivery

不要 push。

## REQUIRED OUTPUT

```
TASK=W2C_IMMUTABLE_DEPLOY

SAFE_BASE_COMMIT=216f709cde8ce0d95b60f120997e13c650029a1a

BRANCH=
WORKTREE=
COMMIT=

FILES_CHANGED=

MAIN_DIRTY_TREE_TOUCHED=
NO

DM_ORC_FILES_TOUCHED=
NO

LEGACY_SOURCE_MODE_PRESERVED=
YES/NO

IMMUTABLE_IMAGE_MODE=
PASS/FAIL

EXACT_DIGEST_REQUIRED=
YES/NO

EVIDENCE_MATCH_REQUIRED=
YES/NO

VPS_BUILD_IN_IMMUTABLE_MODE=
YES/NO

FAIL_CLOSED_BEFORE_MUTATION=
PASS/FAIL

ROLLBACK_COMPATIBLE=
YES/NO

SERVICES_YAML_CHANGED=
NO

REMOTE_ACTION=
NO

TESTS=

FOLLOWUP_BRIDGE_REQUIRED=

READY_FOR_INTEGRATION=
YES/NO
```
