# STG-PREP-B — Staging Bridge Preparation

- 抬頭：POST STAGING BRIDGE PREPARATION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- 模式：STRICT READ ONLY

---

你現在是：

POST STAGING BRIDGE PREPARATION OWNER

這是與 W2INT-v2 平行執行的 Staging Bridge 前置準備。

不是 Deployment。
不是 Production。
不是修改 Central Deploy dirty main。

## CURRENT AUTHORITATIVE STATE

POST Wave2 Integration 正在另一個 Session 執行：

W2INT-v2

因此本 task 不得依賴最終 integration HEAD 已產生。

目前已知：

POST Wave2 authoritative base:
b3cc93de970f4c7562eff3a4d5e17614462ae8bb

POST Health/Smoke candidate:
ddedd8c41f47d4dff8c99ad3ccd04b273244845a

Central Deploy immutable delivery:
429898662fb487eb4bafe222dd11e60453a8a204

Central Deploy evidence/GHCR bridge:
808fe81e082602c50a4c3770487b1d67b9755612

Central Deploy worktree:
F:\00-Ticenpi-SaaS\deploy-w2-post

Central Deploy branch:
repair/post-immutable-deploy

Central Deploy main currently has unrelated DM/ORC dirty changes in:
- server/critical-smoke.sh
- services.yaml
- server/warp-proxy-relay-dmruntimestaging.service

## GOAL

在不修改任何 source 的前提下，把「W2INT-v2 PASS 後需要做的 Staging Bridge」準備完整。

最終 bridge 必須能回答：

1. POST final integration HEAD 如何產出 CI artifact / release evidence。
2. Central Deploy 如何吃 exact immutable image@sha256。
3. Post health/smoke 如何註冊進 Central Deploy。
4. Staging service 應使用哪個 service id / environment / compose。
5. 哪些檔案最後需要修改。
6. 哪些修改現在被 DM/ORC dirty main 暫時阻擋。
7. 哪些 bridge 可以在 isolated worktree 完成。
8. 哪些動作一定要等 W2INT-v2 final HEAD。

## HARD RULES

STRICT READ ONLY。

禁止：

- commit
- stash
- reset
- restore
- clean
- merge
- rebase
- cherry-pick
- branch mutation
- worktree mutation
- push
- pull
- SSH write
- VPS mutation
- Production
- Staging deploy
- GitHub/GHCR mutation

## SOURCES TO INSPECT

### TicenpiPost

只讀：

- .github/workflows/ci.yml
- scripts/release/*
- deploy/staging/compose.yml
- deploy/runtime-staging/compose.yml
- deploy/healthcheck.sh
- deploy/critical-smoke.sh
- deploy/smoke/post_smoke.py
- deployment-contract.md / related deployment docs
- relevant service/environment metadata

### Central Deploy

只讀：

- deploy.ps1
- server/deploy.sh
- server/immutable_image.py
- services.yaml
- server/critical-smoke.sh
- server/audit.sh
- rollback / preflight related scripts

## REQUIRED ANALYSIS

### A. Service identity

確認：

- Post production service id
- Post runtime-staging service id
- service registry keys
- compose paths
- health URL/port
- smoke profile naming

不得猜。

### B. Smoke registration delta

精準列出：

若要把 Post W2E smoke 接進 Central Deploy，需要修改哪些 Central Deploy files、哪些 case/profile/registry entries。

因為 services.yaml 與 server/critical-smoke.sh 目前被 DM/ORC dirty changes佔用：

只產生 proposed minimal delta。

不要修改。

### C. Immutable deployment binding

確認：

Post G6 evidence fields
→ Central Deploy immutable_image.py
→ deploy.ps1
→ server/deploy.sh

能否形成：

source commit
→ exact image digest
→ staging deploy without build

列出任何剩餘 contract gap。

### D. Staging target

確認真正 staging target：

- service name
- environment name
- compose
- host port
- Supabase target
- expected URL

如果資料不足：
標 UNKNOWN，不要猜。

### E. Final bridge sequence

產出最小 serial bridge sequence，例如：

W2INT final HEAD
→ CI
→ release evidence
→ GHCR exact digest
→ Central Deploy registration
→ immutable staging deploy
→ health
→ critical smoke

但只能根據 repo evidence。

## OUTPUT

```
TASK=STG_BRIDGE_PREP

MODE=READ_ONLY

POST_SERVICE_ID=
POST_RUNTIME_STAGING_SERVICE_ID=

PRODUCTION_COMPOSE=
RUNTIME_STAGING_COMPOSE=

STAGING_PORT=
STAGING_URL=

POST_SMOKE_FILES=

CENTRAL_DEPLOY_SMOKE_REGISTRATION_FILES=

SMOKE_REGISTRATION_CURRENTLY_BLOCKED=
YES/NO

BLOCKED_BY_DIRTY_FILES=

IMMUTABLE_DEPLOY_CONTRACT=
PASS/PARTIAL/FAIL

RELEASE_EVIDENCE_CONTRACT=
PASS/PARTIAL/FAIL

REMAINING_CONTRACT_GAPS=

FINAL_BRIDGE_FILES_TO_CHANGE=

CAN_PREPARE_IN_ISOLATED_WORKTREE=

MUST_WAIT_FOR_W2INT_HEAD=

FINAL_STAGING_BRIDGE_SEQUENCE=

MUTATION_PERFORMED=NO

READY_FOR_FINAL_STAGING_BRIDGE=
YES/NO
```
