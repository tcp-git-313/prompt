# W2C-STG — Staging Immutable Binding + Release Source Pinning

- 抬頭：CENTRAL DEPLOY STAGING IMMUTABLE BINDING OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- Central Deploy branch：repair/post-immutable-deploy
- Start commit：808fe81e082602c50a4c3770487b1d67b9755612

---

你現在是：

CENTRAL DEPLOY STAGING IMMUTABLE BINDING OWNER

這是 Central Deploy W2C 的 targeted follow-up。

目前 read-only Staging Bridge Audit確認兩個尚未解決且不需要碰 dirty main shared files 的 gap：

GAP 1 — STAGING SERVICE ID
- G6 evidence service=post
- deployment target service id=postruntimestaging
- immutable_image.py目前直接比較 evidence.service == requested service
- 因此 staging immutable deploy會 EVIDENCE_SERVICE_MISMATCH

GAP 2 — RELEASE SOURCE PINNING
- deploy.ps1 immutable mode目前仍會從 $cfg.sourcePath package compose/config
- $cfg.sourcePath可能是 dirty/mutable working tree
- 這破壞「source commit / evidence / compose / image 同一 release identity」

本任務只修這兩個 gap。

## REPOSITORY

Central Deploy isolated worktree：

F:\00-Ticenpi-SaaS\deploy-w2-post

branch：
repair/post-immutable-deploy

起始 HEAD 必須：

808fe81e082602c50a4c3770487b1d67b9755612

working tree 必須 clean。

若不符：
STOP。

不要碰：

F:\00-Ticenpi-SaaS\deploy main dirty working tree

## OWNERSHIP

允許修改：

- deploy.ps1
- server/deploy.sh
- server/immutable_image.py
- W2C dedicated tests
- 必要 immutable helper

禁止修改：

- services.yaml
- server/critical-smoke.sh
- server/audit.sh
- DM/ORC dirty files
- TicenpiPost repo

## TARGET A — PRODUCT/SERVICE IDENTITY MAPPING

不要弱化 evidence validation。

建立 explicit mapping：

deployment service:
postruntimestaging

product/evidence service:
post

要求：

- mapping必須明確、fail closed
- 只有已註冊 mapping可以接受
- post evidence不得被任意其他 service重用
- production post → evidence service=post
- runtime staging postruntimestaging → evidence service=post
- unrelated service → FAIL

優先沿用 services registry已存在的 product/service metadata；若沒有合適欄位，可在 helper內建立最小 explicit mapping，但不要修改 services.yaml。

不要用「忽略 service check」解決。

## TARGET B — RELEASE SOURCE PINNING

Immutable mode不得從 mutable working tree取得 compose/config。

需要保證 release config來源綁定 evidence.git_sha / source commit。

選擇 repo現有最小、安全機制，例如：

A. 從指定 git commit archive/export出 release source snapshot
或
B. 要求明確傳入已由 CI/approved package建立的 source snapshot path，並驗證 commit identity

不可：

- 默默 package current dirty sourcePath
- 只驗 image digest但使用另一版 compose
- 在 VPS重新 build

## REQUIRED SOURCE CONTRACT

Immutable deploy開始 remote mutation前，至少能證明：

evidence.git_sha
==
release source/config snapshot git sha

若不一致：
FAIL CLOSED。

若無法在 Central Deploy repo現有能力安全建立 commit snapshot：

STOP並回報需要的最小 external prerequisite。

不要猜。

## PRESERVE

必須維持：

- exact image@sha256
- source_state=clean
- evidence digest binding
- GHCR auth contract
- versioned release directory
- rollback
- manifest
- legacy source mode
- immutable mode --no-build

## NO REMOTE ACTION

只做：

- unit tests
- fixtures
- dry-run
- static validation

禁止：

- SSH
- VPS write
- real docker login GHCR
- Production/Staging deploy
- push

## TESTS

至少：

### service mapping
1. post target + evidence.service=post → PASS
2. postruntimestaging target + evidence.service=post → PASS
3. postruntimestaging + evidence.service=dm → FAIL
4. unknown deployment service → FAIL

### source pinning
5. evidence.git_sha == source snapshot sha → PASS
6. mismatch → FAIL before remote mutation
7. dirty/mutable source fallback不可被 immutable mode使用
8. legacy source mode still works
9. immutable graph仍为 docker pull exact digest + up -d --no-build
10. rollback still valid

## COMMIT

fix(deploy): bind staging service and release source identity

不要 push。

## REQUIRED OUTPUT

```
TASK=W2C_STAGING_BINDING_SOURCE_FIX

START_COMMIT=808fe81e082602c50a4c3770487b1d67b9755612

BRANCH=
WORKTREE=
COMMIT=

FILES_CHANGED=

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

NO_BUILD_PRESERVED=
YES/NO

LEGACY_SOURCE_MODE_PRESERVED=
YES/NO

SERVICES_YAML_CHANGED=NO
CRITICAL_SMOKE_CHANGED=NO
AUDIT_SH_CHANGED=NO

MAIN_DIRTY_TREE_TOUCHED=NO

REMOTE_ACTION=NO

TESTS=

FOLLOWUP_SHARED_FILE_BRIDGE_REQUIRED=
YES/NO

READY_FOR_FINAL_STAGING_BRIDGE=
YES/NO
```
