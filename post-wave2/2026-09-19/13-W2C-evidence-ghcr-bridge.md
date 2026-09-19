# W2C-BRIDGE — Exact Release Evidence + GHCR Pull Contract

- 抬頭：CENTRAL DEPLOY POST EVIDENCE BRIDGE OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：中型
- Central Deploy candidate：429898662fb487eb4bafe222dd11e60453a8a204
- Post G6 evidence producer：b3cc93de970f4c7562eff3a4d5e17614462ae8bb lineage

---

你現在是：

CENTRAL DEPLOY POST EVIDENCE BRIDGE OWNER

這是 W2C immutable deployment 後的正式 contract bridge。

目標不是新增另一套 release format。

目標是讓 Central Deploy 精準接受 TicenpiPost G6 已產生的 release evidence schema，並定義 GHCR private image pull 所需的 credential contract。

## REPOSITORY

Central Deploy isolated worktree：

`F:\00-Ticenpi-SaaS\deploy-w2-post`

branch：
`repair/post-immutable-deploy`

起始 HEAD 必須是：

`429898662fb487eb4bafe222dd11e60453a8a204`

working tree 必須 clean。

否則 STOP。

不要碰 main dirty working tree。

## AUTHORITATIVE POST EVIDENCE FORMAT

TicenpiPost G6 已產生：

- `release-evidence/post-release.json`
- `scripts/release/release_evidence.schema.json`
- `scripts/release/generate_release_evidence.py`
- `scripts/release/validate_release_evidence.py`

已知正式欄位包含/使用：

- git_sha
- source_state
- artifact_digest
- immutable_reference
- workflow run identity
- image repository/tag metadata
- extension artifact metadata

Central Deploy 不得自行再發明另一套 aliases 作 primary contract。

## TASK 1 — INSPECT EXACT G6 SCHEMA

只讀取得 TicenpiPost Wave1/Wave2 base中上述 schema / generator。

建立一份 precise field mapping。

Central Deploy helper目前若接受：

service
source_sha
image_digest

等 aliases，只能作 backward compatibility，不能取代正式 G6 field names。

## TASK 2 — FORMAL EVIDENCE VALIDATION

調整：

server/immutable_image.py

以及必要 tests，使 Post immutable mode至少嚴格驗：

- source_state == clean
- git_sha present and valid
- artifact_digest == digest from immutable_reference
- immutable_reference is exact @sha256
- service/product identity resolves to Post
- evidence mismatch fail closed

若 G6 schema本身有正式 version / artifact identity欄位，沿用，不新增平行欄位。

## TASK 3 — GHCR READ AUTH CONTRACT

W2C已確認 private GHCR image需要 login。

本 task只定義安全 credential interface，不放 secret。

優先設計：

- GHCR_READ_TOKEN
- GHCR_USERNAME / actor if required

要求：

- token不得出現在 CLI args
- token不得寫入 release evidence
- token不得寫入 manifest
- token不得 echo/log
- login應使用 stdin，例如 docker login --password-stdin
- immutable pull前完成 auth
- missing required auth時 fail closed

如果既有 Central Secret Store有既定命名，優先沿用既有 naming，不自行重複造 secret。

## TASK 4 — NO REMOTE ACTION

只做 local fixture / mock / dry-run。

不得：

- 真實 docker login GHCR
- SSH
- VPS mutation
- Production deploy
- GitHub push
- GHCR push

## OWNERSHIP

允許修改：

- server/immutable_image.py
- deploy.ps1
- server/deploy.sh
- W2C dedicated tests
- 必要的新 helper/tests

禁止修改：

- services.yaml
- server/critical-smoke.sh
- DM/ORC files
- TicenpiPost repo source

## IMPORTANT

不要處理：

Post smoke registration
services.yaml
production enablement
AUTO_PUBLISH
Sign
DM/ORC

這些不是本 task。

## TESTS

至少：

1. real-shaped G6 evidence fixture → PASS
2. legacy alias fixture（若保留）→ compatibility PASS
3. source_state != clean → FAIL
4. git_sha malformed → FAIL
5. artifact_digest != immutable_reference digest → FAIL
6. tag-only ref → FAIL
7. wrong service/product → FAIL
8. missing GHCR credential contract → FAIL before remote mutation
9. token never appears in rendered/logged command
10. immutable dry-run still has:
    docker pull exact digest
    compose up -d --no-build
11. legacy source mode unaffected

## COMMIT

`fix(deploy): align post evidence and ghcr pull contract`

不要 push。

## REQUIRED OUTPUT

```
TASK=W2C_EVIDENCE_GHCR_BRIDGE

START_COMMIT=429898662fb487eb4bafe222dd11e60453a8a204

BRANCH=
WORKTREE=
COMMIT=

FILES_CHANGED=

G6_SCHEMA_USED_AS_PRIMARY=
YES/NO

PRIMARY_EVIDENCE_FIELDS=

LEGACY_ALIASES_PRESERVED=
YES/NO

SOURCE_STATE_CLEAN_REQUIRED=
YES/NO

GIT_SHA_REQUIRED=
YES/NO

DIGEST_BINDING=
PASS/FAIL

EXACT_IMMUTABLE_REFERENCE=
PASS/FAIL

GHCR_AUTH_CONTRACT=

TOKEN_IN_CLI_ARGS=
NO

TOKEN_LOGGED=
NO

FAIL_CLOSED_BEFORE_REMOTE_MUTATION=
PASS/FAIL

LEGACY_SOURCE_MODE_PRESERVED=
YES/NO

SERVICES_YAML_CHANGED=
NO

MAIN_DIRTY_TREE_TOUCHED=
NO

REMOTE_ACTION=
NO

TESTS=

FOLLOWUP_SMOKE_REGISTRATION_BLOCKED_BY_DIRTY_MAIN=
YES/NO

READY_FOR_CENTRAL_DEPLOY_INTEGRATION=
YES/NO
```
