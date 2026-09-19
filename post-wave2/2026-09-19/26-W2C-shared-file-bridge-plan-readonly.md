# W2C-SHARED-PREP — Central Deploy Shared-File Bridge Plan

- 抬頭：CENTRAL DEPLOY SHARED FILE BRIDGE PREPARATION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- 模式：STRICT READ ONLY

---

你現在是：

CENTRAL DEPLOY SHARED FILE BRIDGE PREPARATION OWNER

這是與其他任務平行的 READ-ONLY bridge planning。

不要修改 Central Deploy main。
不要修改 deploy-w2-post。
不要 commit。

## KNOWN GOOD CENTRAL DEPLOY LINEAGE

Isolated worktree：

F:\00-Ticenpi-SaaS\deploy-w2-post

branch：
repair/post-immutable-deploy

已完成：

W2C immutable delivery:
429898662fb487eb4bafe222dd11e60453a8a204

W2C evidence/GHCR bridge:
808fe81e082602c50a4c3770487b1d67b9755612

W2C staging non-overlap fix:
39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

這三段視為 frozen。

## CURRENT SHARED-WRITER PROBLEM

Central Deploy main：

F:\00-Ticenpi-SaaS\deploy

目前 DM/ORC 有未提交共享檔案修改，已知至少：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml

另可能有：

- DM_RELEASE_GATES.md
- server/warp-proxy-relay-dmruntimestaging.service

因此目前不能直接做 Post shared-file bridge。

## GOAL

只讀分析「當 DM/ORC shared changes 收斂後，Post 最小 bridge 要怎麼套上去」。

Post staging bridge需要：

1. services.yaml
   - post / postruntimestaging criticalSmokeProfile 註冊
2. server/critical-smoke.sh
   - post profile adapter/case
3. server/deploy.sh
   - valid_smoke_profiles 接受 Post profile
4. server/audit.sh
   - valid_smoke_profiles 同步
   - required_services 是否需要 postruntimestaging，依現行 architecture判斷

你的任務是建立 exact merge plan，而不是修改檔案。

## HARD RULES

STRICT READ ONLY。

禁止：

- add
- commit
- stash
- reset
- restore
- clean
- checkout mutation
- cherry-pick
- merge
- rebase
- worktree mutation
- push
- SSH write
- VPS mutation

## STEP 1 — CAPTURE BOTH SIDES

讀：

### Main dirty side
F:\00-Ticenpi-SaaS\deploy

對以下檔案取得：

- HEAD version
- working-tree version
- diff from HEAD

檔案：

server/deploy.sh
server/audit.sh
server/critical-smoke.sh
services.yaml

### Frozen Post bridge side
F:\00-Ticenpi-SaaS\deploy-w2-post @ 39ecc94

讀：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml
- server/immutable_image.py
- deploy.ps1

注意 shared files在 39ecc94仍未加入 Post smoke registration；要根據現有 patterns設計 proposed delta。

## STEP 2 — IDENTIFY ACTIVE DM/ORC CHANGES

逐檔分類 main dirty diff：

- DM
- ORC
- shared framework
- unknown

不要只看 filename，要看實際 diff內容。

確認是否有：

- new smoke profiles
- valid_smoke_profiles changes
- required_services changes
- new service definitions
- WARP relay / runtime-staging entries

## STEP 3 — DEFINE POST PROFILE CONTRACT

根據 TicenpiPost W2E smoke contract設計：

建議 profile：
post-core

若現有 repo命名/模式有更明確 canonical naming，沿用現有 pattern。

精準定義：

- service=post
- service=postruntimestaging
- compose project mapping
- base URL mapping
- Post smoke script invocation adapter

不得自行更改 Post W2E scripts。

## STEP 4 — MINIMAL SHARED FILE DELTA

針對每個檔案輸出：

### services.yaml
要新增/改哪些 key

### server/critical-smoke.sh
要新增哪些 case/function/adapter

### server/deploy.sh
要增加哪些 valid_smoke_profiles

### server/audit.sh
要增加哪些 valid_smoke_profiles
required_services 是否要變，若要，說明理由

要求：

- 不破壞 DM/ORC current dirty changes
- 不移除既有 profiles
- deploy.sh / audit.sh profile whitelist 必須一致或說明刻意差異
- unknown profile fail closed

## STEP 5 — MERGE RISK

判斷 Post bridge與目前 DM/ORC dirty hunks：

- NO_OVERLAP
- SAME_FILE_DIFFERENT_HUNK
- SAME_HUNK_CONFLICT
- SEMANTIC_CONFLICT

逐檔列出。

## STEP 6 — SAFE EXECUTION PLAN

如果 DM/ORC 之後 commit：

列出最安全串行方式：

1. refresh main HEAD
2. 建 fresh isolated bridge worktree
3. cherry-pick frozen W2C commits或基於已整合 Central Deploy target
4. 套 Post shared-file delta
5. tests
6. commit
7. 不直接 Production

不要現在執行。

## OUTPUT

```
TASK=W2C_SHARED_FILE_BRIDGE_PLAN

MODE=READ_ONLY

CENTRAL_MAIN_HEAD=

CENTRAL_MAIN_DIRTY_FILES=

FROZEN_W2C_HEAD=39ecc94fdaed5cd2950b277cc15d1b39da28e6c1

POST_SMOKE_PROFILE=

POST_SERVICE_MAPPING=

POST_RUNTIME_STAGING_MAPPING=

FILE_PLAN_services_yaml=

FILE_PLAN_server_critical_smoke=

FILE_PLAN_server_deploy=

FILE_PLAN_server_audit=

DEPLOY_AUDIT_PROFILE_WHITELIST_CONSISTENT=
YES/NO

REQUIRED_SERVICES_CHANGE_NEEDED=
YES/NO

REQUIRED_SERVICES_REASON=

DM_ORC_OVERLAP_services_yaml=
NO_OVERLAP / SAME_FILE_DIFFERENT_HUNK / SAME_HUNK_CONFLICT / SEMANTIC_CONFLICT

DM_ORC_OVERLAP_critical_smoke=
...

DM_ORC_OVERLAP_deploy_sh=
...

DM_ORC_OVERLAP_audit_sh=
...

SAFE_SERIAL_MERGE_PLAN=

BLOCKERS=

MUTATION_PERFORMED=NO

READY_TO_EXECUTE_SHARED_BRIDGE_AFTER_DM_ORC_SETTLES=
YES/NO
```
