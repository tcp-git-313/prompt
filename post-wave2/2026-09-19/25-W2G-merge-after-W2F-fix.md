# W2G-MERGE — Merge CI Evidence Wiring After W2F Fix

- 抬頭：POST CI EVIDENCE INTEGRATION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：小型
- W2G source commit：c2a0086

---

你現在是：

POST CI EVIDENCE INTEGRATION OWNER

這個任務只能在 W2F-FIX2 + W2INT continuation 完成後執行。

不要提前執行。

## TARGET

將已完成並驗證的 CI evidence wiring fix：

c2a0086

安全整合進：

repair/post-v2-wave2-integration-v2

而不重跑 W2G 調查。

## PRECONDITION

Target worktree：

F:\00-Ticenpi-SaaS\TicenpiPost-wave2-integration-v2

branch：

repair/post-v2-wave2-integration-v2

先確認：

- branch 正確
- worktree clean
- no cherry-pick / merge / rebase in progress
- W2F-FIX2 已完成
- typecheck = PASS
- previous continuation validation已完成或至少沒有 unresolved source changes

若任一不符：
STOP。

## SOURCE FIX

W2G commit：

c2a0086

唯一預期修改：

.github/workflows/ci.yml

內容：

在 generate_release_evidence.py invocation加入：

--extension-metadata "${RELEASE_EVIDENCE_DIR}/ticenpi-post-extension-${GIT_SHA}.metadata.json"

## OWNERSHIP CHECK

先：

git show --name-only c2a0086

必須只改：

.github/workflows/ci.yml

若有其他檔：
STOP。

## CHERRY PICK

cherry-pick exact：

c2a0086

若 conflict：
STOP。

禁止：

- ours
- theirs
- manual resolution
- merge
- rebase

## VALIDATION

至少：

1. YAML parse PASS
2. package_extension metadata output path與 generate_release_evidence input path一致
3. generate_release_evidence --extension-metadata present
4. local evidence generation PASS
5. validate_release_evidence PASS
6. git diff --check PASS
7. existing frontend typecheck仍 PASS
8. integration worktree clean after commit

不需要重跑 full backend。

不得：

- push
- workflow_dispatch
- GHCR push
- VPS
- Production
- real FB action

## REQUIRED OUTPUT

```
TASK=W2G_MERGE_AFTER_W2F

TARGET_BRANCH=
START_HEAD=

SOURCE_COMMIT=c2a0086

OWNERSHIP_AUDIT=
PASS/FAIL

CHERRY_PICK=
PASS/FAIL

RESULT_COMMIT=
FINAL_HEAD=

FILES_CHANGED=

CI_GENERATES_EXTENSION_METADATA=
YES/NO

CI_PASSES_EXTENSION_METADATA_TO_EVIDENCE=
YES/NO

LOCAL_EVIDENCE_GENERATION=
PASS/FAIL

LOCAL_EVIDENCE_VALIDATION=
PASS/FAIL

TYPECHECK=
PASS/FAIL

FINAL_WORKTREE_CLEAN=
YES/NO

REMOTE_CI_RUN=
NOT_RUN

PRODUCTION_TOUCHED=NO
VPS_TOUCHED=NO

READY_FOR_STAGING_ARTIFACT=
YES/NO
```
