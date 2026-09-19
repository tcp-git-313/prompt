# W2D — POST Runtime Config / Environment Wiring

- 抬頭：POST WAVE2 RUNTIME CONFIG OWNER
- 模型：DeepSeek V4.1 Flash
- 預計時間：15–25 分鐘
- 任務類型：Implementation
- Authoritative Base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

---

你現在是：POST WAVE2 RUNTIME CONFIG OWNER

這是正式 Implementation。

## BASE

Exact base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

建立：
- branch: `repair/post-runtime-config`
- worktree: `F:\00-Ticenpi-SaaS\TicenpiPost-w2-runtime-config`

直接從 exact base 建立並確認 clean。

## IMPORTANT OWNERSHIP BOUNDARY

你只處理 POST runtime compose / environment wiring。

你不負責 immutable digest deploy engine，那是 W2C。

不要自己發明 IMAGE_DIGEST、GHCR_DIGEST、release evidence、promotion protocol。

## FIRST STEP

確認實際存在的 `deploy/staging/compose.yml`、`deploy/runtime-staging/compose.yml` 以及其他真正屬於 POST runtime 的 compose template。只修改確認屬於 POST 的檔案。

## TARGET

移除或修正危險 hardcoded override，例如：
- `TICENPI_AUTO_PUBLISH: ""`
- `ANTHROPIC_API_KEY: ""`
- `OPENAI_API_KEY: ""`
- `TICENPI_BUDGET_EXEMPT_ACCOUNTS: ""`
- 固定 WARP_PROXY literal

改成正確 env interpolation。

## PRODUCTION AUTH TARGET

正式 Production intended values：
- `TICENPI_REQUIRE_STUDIO=0`
- `TICENPI_COMMERCIAL_GATE_ENABLED=1`
- `TICENPI_ALLOW_TENANT_KEY=0`

但要先辨認每個 compose 的環境身份，不得把 Production target 錯套進 Local / generic Staging。

## AUTO PUBLISH

本任務不得開啟，必須保持 fail closed。

例如 `${TICENPI_AUTO_PUBLISH:-}` 或 repo 現有等價安全模式。

不得設 true / 1。

## SECRETS

禁止寫入 real Supabase key、service role、JWT secret、cookie key、Anthropic/OpenAI key、proxy credential、tenant key、password。只能 interpolation。

## FORBIDDEN

禁止修改：
- `deploy.ps1`
- Central Deploy repo
- `backend/*`
- `frontend/*`
- `extension/*`
- Local `docker-compose.yml`

禁止 Production/VPS action。

## VALIDATION

使用 dummy env 執行 `docker compose config`。

確認 syntax PASS、interpolation PASS、no secret literal、no accidental publish activation。

## COMMIT

`fix(post): make runtime configuration environment-driven`

不要 push。

## REQUIRED OUTPUT

```
TASK=W2D_RUNTIME_CONFIG
BASE_COMMIT=
BRANCH=
WORKTREE=
COMMIT=
FILES_CHANGED=
RUNTIME_ENVIRONMENTS_FOUND=
HARDCODED_OVERRIDES_REMOVED=
PRODUCTION_AUTH_TARGET=
AUTO_PUBLISH_DEFAULT=
SECRET_LITERAL_FOUND=YES/NO
COMPOSE_VALIDATION=
IMMUTABLE_DEPLOY_INTERFACE_CHANGED=NO
CROSS_OWNERSHIP_REQUIRED=
OWNERSHIP_VIOLATION=YES/NO
READY_FOR_INTEGRATION=YES/NO
```
