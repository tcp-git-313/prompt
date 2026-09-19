# STG-PREP-C — Artifact / GHCR / Release Evidence Readiness

- 抬頭：POST STAGING ARTIFACT READINESS OWNER
- 模型：MiMo-V2.5
- 預計工程大小：中型
- 模式：READ ONLY

---

你現在是：

POST STAGING ARTIFACT READINESS OWNER

這個任務與 W2INT-v2 平行。

目的：

在 W2INT-v2 最終 HEAD 產生之前，先確認從 Git commit 到可部署 immutable artifact 的整條 Artifact Factory 是否已準備好。

不是 Deploy。
不是 Production。
不得 push。

## CURRENT STATE

POST G6 artifact factory 已整合於 Wave1 lineage。

Central Deploy immutable delivery已完成：

429898662fb487eb4bafe222dd11e60453a8a204

Evidence/GHCR bridge已完成：

808fe81e082602c50a4c3770487b1d67b9755612

## GOAL

確認 W2INT-v2 一旦 PASS 並取得 final HEAD 後，可以最短路徑完成：

clean commit
→ GitHub CI
→ GHCR immutable digest
→ post-release.json
→ extension zip metadata
→ Central Deploy evidence validation
→ staging exact digest deploy

## HARD RULES

READ ONLY。

禁止：

- 修改 repo
- commit
- push
- workflow dispatch
- GHCR push
- docker login remote
- SSH
- VPS write
- Production
- Staging deploy

## INSPECT POST

確認：

### CI trigger

- 哪些 branch/event 會 build
- push / PR 行為
- artifact push 條件
- evidence upload 條件
- 是否需要正式 release/tag

### Image identity

確認：

- image repository
- SHA tag
- actual digest capture
- immutable_reference
- source_state
- git_sha
- run_id/run_url/workflow
- platform/tests

### Extension artifact

確認：

- manifest version
- extension version
- git SHA
- SHA256
- zip packaging
- evidence linkage

### Fail closed

確認：

- dirty source拒絕
- digest capture failure拒絕
- evidence validation failure拒絕

## INSPECT CENTRAL DEPLOY

只讀確認：

808fe81 lineage對正式 G6 schema支援：

- source_state clean
- git_sha
- artifact_digest
- immutable_reference
- service/environment
- GHCR auth contract

## GHCR AUTH

確認 repo/document/config中：

- GHCR_READ_TOKEN 預期來源
- GHCR_USERNAME behavior
- docker login --password-stdin
- secret不得進 argv/log/evidence

不要讀或輸出 secret value。

若無法證明 credential實際存在：

標：

GHCR_CREDENTIAL_PRESENCE=UNKNOWN

不要猜。

## REMOTE CI STATUS

如果目前沒有 W2INT-v2 final commit：

REMOTE_CI_FOR_FINAL_HEAD=WAITING_FOR_W2INT

這是正常狀態。

不要為此判 FAIL。

## OUTPUT

```
TASK=STG_ARTIFACT_READINESS

MODE=READ_ONLY

POST_CI_WORKFLOW=

CI_TRIGGER_MODEL=

CLEAN_SOURCE_GATE=
PASS/FAIL

GHCR_IMAGE_REPOSITORY=

DIGEST_CAPTURE_LOGIC=
PASS/FAIL

IMMUTABLE_REFERENCE_LOGIC=
PASS/FAIL

RELEASE_EVIDENCE_SCHEMA=
PASS/FAIL

EXTENSION_ARTIFACT_FACTORY=
PASS/FAIL

CENTRAL_DEPLOY_G6_SCHEMA_COMPAT=
PASS/FAIL

GHCR_AUTH_CONTRACT=
PASS/FAIL

GHCR_CREDENTIAL_PRESENCE=
YES/NO/UNKNOWN

TOKEN_EXPOSURE_RISK=

REMOTE_CI_FOR_FINAL_HEAD=
WAITING_FOR_W2INT / AVAILABLE

BLOCKERS_BEFORE_FIRST_STAGING_ARTIFACT=

MUTATION_PERFORMED=NO

READY_TO_BUILD_FINAL_STAGING_ARTIFACT_AFTER_W2INT=
YES/NO
```
