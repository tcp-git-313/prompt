# W2C-STG-v2 — Staging Immutable Binding + Source Pinning (No Shared-Writer Files)

- 抬頭：CENTRAL DEPLOY STAGING IMMUTABLE NONOVERLAP OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- Central Deploy worktree：F:\00-Ticenpi-SaaS\deploy-w2-post
- Start commit：808fe81e082602c50a4c3770487b1d67b9755612

---

你現在是：

CENTRAL DEPLOY STAGING IMMUTABLE NONOVERLAP OWNER

這是 W2C targeted follow-up，目標是修：

1. postruntimestaging 接受 evidence.service=post 的明確 product/service identity mapping
2. immutable deployment 不得從 mutable working tree package compose/config，必須綁定 evidence.git_sha / release source snapshot

但目前 Central Deploy main 有 DM/ORC active dirty writer，且實際 dirty files 包含：

- server/audit.sh
- server/critical-smoke.sh
- server/deploy.sh
- services.yaml
- DM_RELEASE_GATES.md
- server/warp-proxy-relay-dmruntimestaging.service

因此本 task **不得修改任何 shared dirty file**。

## START STATE

使用既有 clean isolated worktree：

F:\00-Ticenpi-SaaS\deploy-w2-post

branch:
repair/post-immutable-deploy

起始 HEAD 必須：

808fe81e082602c50a4c3770487b1d67b9755612

git status --porcelain 必須 empty。

如果不符：
STOP。

## EXCLUSIVE WRITE OWNERSHIP

本 task 只允許修改：

- server/immutable_image.py
- deploy.ps1
- server/tests/test_immutable_image.py
- server/tests/test_w2c_*.py
- 必要的新 helper / tests（但不得碰 shared dirty file）

以下全部 READ ONLY：

- server/deploy.sh
- server/audit.sh
- server/critical-smoke.sh
- services.yaml
- rollback.sh
- DM/ORC files

若完成 target 必須修改任何 READ ONLY 檔：

HARD STOP

輸出：

CROSS_WRITER_REQUIRED=<file + reason>

不要自行越界。

## TARGET A — PRODUCT/SERVICE IDENTITY

目前：

G6 evidence.service=post

Production target service=post

Runtime staging target service=postruntimestaging

不可直接用：

evidence.service == requested service

也不可完全關掉 service validation。

建立明確 fail-closed mapping，例如：

deployment service post
→ evidence product post

deployment service postruntimestaging
→ evidence product post

unknown / unrelated service
→ reject

優先把 mapping 放在 immutable validation helper，而不是 shared services.yaml。

要求：

- post + evidence post → PASS
- postruntimestaging + evidence post → PASS
- postruntimestaging + evidence dm → FAIL
- unknown target → FAIL
- 不允許任意 alias

## TARGET B — RELEASE SOURCE PINNING

Immutable mode不得使用 mutable current sourcePath 作為 release compose/config來源。

目前 deploy.ps1若從 $cfg.sourcePath package release config，必須修正。

優先選擇「不修改 server/deploy.sh」即可完成的方案。

可接受模式：

A. deploy.ps1 從 TicenpiPost repo exact evidence.git_sha 建立 clean git archive / temporary snapshot
或
B. deploy.ps1 要求明確 approved source snapshot path，並驗證 snapshot commit identity == evidence.git_sha

要求：

evidence.git_sha
==
release source/config snapshot git SHA

且：

- source snapshot clean
- compose/config來自該 snapshot
- mutable working tree不能 fallback
- mismatch在任何 SCP/SSH/remote mutation前 fail closed
- image仍是 exact @sha256
- VPS仍不 build

如果 Central Deploy現有資訊不足以安全定位 TicenpiPost repo / source snapshot：

STOP並輸出 prerequisite。

不得猜 path。

## IMPORTANT

不要：

- 修改 server/deploy.sh
- 修改 services.yaml
- 修改 smoke profile
- SSH
- VPS write
- docker login
- push
- Production/Staging deploy
- TicenpiPost source

只做 local fixture / dry-run / tests。

## VALIDATION

至少：

### identity mapping
1. post → evidence post = PASS
2. postruntimestaging → evidence post = PASS
3. postruntimestaging → evidence dm = FAIL
4. unknown target = FAIL

### source pinning
5. evidence SHA == snapshot SHA = PASS
6. SHA mismatch = FAIL before remote mutation
7. dirty/mutable working source不能被 immutable mode使用
8. legacy source mode behavior不被破壞
9. token不輸出
10. immutable reference exact digest規則保留

## COMMIT

若可在 exclusive files完成：

fix(deploy): bind post staging identity to release source

不要 push。

若需要 shared dirty file：

不要 commit partial architecture；HARD STOP。

## REQUIRED OUTPUT

```
TASK=W2C_STAGING_NONOVERLAP_FIX

START_COMMIT=808fe81e082602c50a4c3770487b1d67b9755612

BRANCH=
WORKTREE=
COMMIT=

FILES_CHANGED=

SHARED_DIRTY_FILES_MODIFIED=
NO

SERVICE_IDENTITY_MAPPING=

POST_PRODUCTION_MAPPING=
PASS/FAIL

POST_RUNTIME_STAGING_MAPPING=
PASS/FAIL

UNRELATED_SERVICE_REJECTED=
PASS/FAIL

RELEASE_SOURCE_MODE=

SOURCE_SNAPSHOT_BOUND_TO_GIT_SHA=
PASS/FAIL

MUTABLE_WORKTREE_USED_IN_IMMUTABLE_MODE=
YES/NO

SOURCE_EVIDENCE_MISMATCH_FAIL_CLOSED=
PASS/FAIL

EXACT_DIGEST_PRESERVED=
YES/NO

LEGACY_SOURCE_MODE_PRESERVED=
YES/NO

CROSS_WRITER_REQUIRED=

REMOTE_ACTION=NO

TESTS=

READY_FOR_SHARED_FILE_BRIDGE=
YES/NO
```
