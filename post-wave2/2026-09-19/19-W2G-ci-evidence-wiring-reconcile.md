# W2G — POST CI Evidence Wiring Reconcile

- 抬頭：POST RELEASE EVIDENCE WIRING OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- Mode：Targeted Verify + Minimal Fix

---

你現在是：

POST RELEASE EVIDENCE WIRING OWNER

這是正式 targeted verify + minimal fix。

目前兩份 read-only 報告互相矛盾：

REPORT A（STG_BRIDGE_PREP）：
- final/integration-v2 CI 呼叫 generate_release_evidence.py 時沒有 --extension-metadata
- package_extension.py 也沒有 --metadata-output
- 因此 CI 無法產生 authoritative release evidence

REPORT B（STG_ARTIFACT_READINESS）：
- 宣稱 ci.yml 已把 extension metadata linked into release evidence
- 宣稱沒有 artifact blocker

不能同時成立。

你的任務是以「目前真正的 W2INT-v2 branch HEAD」為唯一證據，裁決並在必要時做最小修正。

## TARGET REPO

F:\00-Ticenpi-SaaS\TicenpiPost

## STEP 1 — VERIFY CURRENT W2INT-v2

只讀確認：

branch:
repair/post-v2-wave2-integration-v2

輸出：

CURRENT_W2INT_HEAD=
CURRENT_W2INT_WORKTREE=
WORKTREE_CLEAN=

若 branch 不存在、worktree dirty、或仍在 cherry-pick/rebase/merge state：

STOP。

不得猜 final HEAD。

## STEP 2 — INSPECT EXACT CI AT CURRENT HEAD

只讀 current HEAD 的：

- .github/workflows/ci.yml
- scripts/release/package_extension.py
- scripts/release/generate_release_evidence.py
- scripts/release/validate_release_evidence.py
- scripts/release/release_evidence.schema.json

精準確認：

1. package_extension.py 是否需要/支援 --metadata-output
2. CI 是否實際產生 extension metadata JSON
3. generate_release_evidence.py 是否 REQUIRE --extension-metadata
4. CI 是否把實際 metadata path傳入 generate_release_evidence.py
5. post-release.json 是否包含 extension metadata
6. validate_release_evidence.py/schema 是否要求或接受 extension object

不要只看 script能力。
要看「CI wiring實際是否閉環」。

## STEP 3 — CLASSIFY

若 CI 已完整閉環：

CI_EVIDENCE_WIRING=PASS

不要改 source。

只輸出 exact line/step evidence。

若 CI 缺任何 required wiring：

CI_EVIDENCE_WIRING=FAIL

進入 minimal fix。

## STEP 4 — MINIMAL FIX IF REQUIRED

禁止直接修改 W2INT-v2 integration branch。

從 CURRENT_W2INT_HEAD 建 fresh isolated worktree：

branch:
repair/post-ci-evidence-wiring

worktree:
F:\00-Ticenpi-SaaS\TicenpiPost-w2-ci-evidence

只允許修改：

.github/workflows/ci.yml

若 script本身真的有 bug才可修改：
scripts/release/*

但優先只修 workflow wiring。

目標閉環：

package_extension.py
→ extension zip
→ extension metadata JSON
→ generate_release_evidence.py --extension-metadata <metadata>
→ post-release.json
→ validate_release_evidence.py
→ schema PASS

## HARD RULES

禁止：

- Production
- VPS
- GitHub push
- GHCR push
- workflow_dispatch
- real remote CI mutation
- backend/frontend/runtime source變更
- deploy/*
- docker-compose.yml

## LOCAL VALIDATION

至少：

- YAML parse
- package_extension local deterministic package
- metadata JSON exists
- generate_release_evidence with metadata PASS
- validate_release_evidence PASS
- missing metadata negative case FAIL if contract requires it
- git diff --check

不得偽造 remote GHCR actual digest。

使用 local fake-shaped sha256 fixture即可做 wiring test。

## COMMIT

若需修正：

fix(ci): wire extension metadata into release evidence

不要 push。

若不需修正：

不要建立 commit。

## REQUIRED OUTPUT

```
TASK=W2G_CI_EVIDENCE_RECONCILE

CURRENT_W2INT_HEAD=
CURRENT_W2INT_WORKTREE=
WORKTREE_CLEAN=

PACKAGE_EXTENSION_METADATA_OUTPUT=
PRESENT/MISSING

GENERATE_EVIDENCE_EXTENSION_METADATA_REQUIRED=
YES/NO

CI_GENERATES_EXTENSION_METADATA=
YES/NO

CI_PASSES_EXTENSION_METADATA_TO_EVIDENCE=
YES/NO

CI_EVIDENCE_WIRING=
PASS/FAIL

REPORT_A_CORRECT=
YES/NO/PARTIAL

REPORT_B_CORRECT=
YES/NO/PARTIAL

FIX_REQUIRED=
YES/NO

BRANCH=
WORKTREE=
COMMIT=

FILES_CHANGED=

LOCAL_EVIDENCE_GENERATION=
PASS/FAIL

LOCAL_EVIDENCE_VALIDATION=
PASS/FAIL

REMOTE_CI_RUN=
NOT_RUN

PRODUCTION_TOUCHED=NO
VPS_TOUCHED=NO

READY_FOR_STAGING_ARTIFACT=
YES/NO
```
