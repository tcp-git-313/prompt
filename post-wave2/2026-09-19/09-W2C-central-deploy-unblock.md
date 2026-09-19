# W2C-UNBLOCK — Central Deploy writer/dirty-tree reconciliation

- 抬頭：CENTRAL DEPLOY WRITER STATE AUDITOR
- 模型：MiMo-V2.5
- 預計工程大小：小型
- Repository：F:\00-Ticenpi-SaaS\deploy
- Mode：STRICT READ ONLY

---

你現在是：

CENTRAL DEPLOY WRITER STATE AUDITOR

這不是 Implementation。

目前 W2C 因 Central Deploy repo dirty 而正確 HARD STOP。

已知：

CURRENT_BRANCH=main
DEPLOY_BASE_COMMIT=216f709cde8ce0d95b60f120997e13c650029a1a

dirty files：
- M server/critical-smoke.sh
- M services.yaml
- ?? server/warp-proxy-relay-dmruntimestaging.service

其他 worktrees：
- codex/b1-env-contract-20260913
- codex/p14-db-20260913

## GOAL

判斷：

1. dirty changes 是否屬於另一個 active task / product
2. dirty files 的內容與 ownership
3. 是否存在安全的 clean commit/ref 可作 W2C base
4. 是否可以建立 isolated W2C worktree，而完全不碰 dirty main
5. 若不可以，明確指出唯一 blocker

## HARD RULES

STRICT READ ONLY。

禁止：

commit
stash
reset
restore
clean
checkout mutation
merge
rebase
cherry-pick
branch -f
worktree remove
push
pull
修改任何 file

## CHECKS

只讀輸出：

- git status --short
- git diff --stat
- git diff -- server/critical-smoke.sh
- git diff -- services.yaml
- untracked service file metadata / content summary（不得輸出 secret）
- git worktree list --porcelain
- git branch -vv
- relevant recent git log

判斷每個 dirty file 最可能屬於：

POST
DM
591
ORC
Sign
Shared deploy infrastructure
UNKNOWN

不要自行推斷作者動機；只依 path/content/ref evidence 分類。

## CLEAN BASE RULE

若 main HEAD 本身 clean commit 可直接作為 isolated worktree base，且 dirty changes 只存在於 main working tree：

回報：

ISOLATED_W2C_BASE_POSSIBLE=YES

但不要真的建立。

若 active shared-writer task 正在修改 W2C 需要的 same files：

回報：

CENTRAL_DEPLOY_WRITER_CONFLICT=YES

## IMPORTANT

W2C implementation ownership預定：

- deploy.ps1
- server/deploy.sh
- 必要 immutable helper

目前 dirty：

- services.yaml
- server/critical-smoke.sh
- warp-proxy-relay-dmruntimestaging.service

如果沒有 overlap，也要判斷是否仍存在 repo-level active writer policy conflict。

## REQUIRED OUTPUT

```
TASK=W2C_CENTRAL_DEPLOY_UNBLOCK

CURRENT_BRANCH=
CURRENT_HEAD=

DIRTY_FILES=

WORKTREES=

DIRTY_FILE_OWNERSHIP=

W2C_TARGET_FILES_CURRENTLY_DIRTY=
YES/NO

ACTIVE_WRITER_OVERLAP=
YES/NO/UNKNOWN

ISOLATED_W2C_BASE_POSSIBLE=
YES/NO

SAFE_BASE_COMMIT=

CENTRAL_DEPLOY_WRITER_CONFLICT=
YES/NO

BLOCKER=

SAFE_NEXT_ACTION=
WAIT / START_W2C_FROM_ISOLATED_BASE / OTHER

MUTATION_PERFORMED=NO
```
