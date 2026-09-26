# 591 Build-to-Go-Live — Luna 執行計畫

> 版本：2026-09-26
> 執行模型：GPT-6 Luna（max thinking）
> 目標：承接既有 591 RC，完成 Staging 建置、部署、真人驗收與不可變證據；取得使用者另行 Production 授權後，完成同一已驗收 artifact 的 Production 上市、canary、證據與可回滾交付。

## 0. 角色與固定輸入

你是 Ticenpi 591 的交付執行者。這不是重新規劃任務；先重驗、再沿用已完成成果，只修已列出的缺口。

```text
PRODUCT = 591
PRODUCT_REPO_READ_ONLY = F:\00-Ticenpi-SaaS\Ticenpi591
RC_WORKTREE = F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc
RC_BRANCH = codex/591-w6-rc-20260925
CENTRAL_DEPLOY = F:\00-Ticenpi-SaaS\deploy
EVIDENCE_ROOT = F:\00-Ticenpi-SaaS\.release-evidence\591
STAGING_SUPABASE_REF = jlsqjvehwblkeuycjoyj
PRODUCTION_SUPABASE_REF = zfpkxsulehkbyuqkkcml
STAGING_SERVICE = 591runtimestaging
PRODUCTION_SERVICE = 591
STAGING_DOMAIN = 591-staging.ticenpi.com
PRODUCTION_DOMAIN = 591.ticenpi.com
STAGING_RELEASE_ROOT = /opt/ticenpi/591-runtime-staging
STAGING_PORT = 8891
PRODUCTION_PORT = 8890
```

不得讀取或引用 `F:\_prompt-publish\W6-SIGN-*.md`。它們屬於另一個產品。

## 1. 開始前必讀與證據優先序

完整讀取下列檔案，不可只讀摘要：

1. `F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md`
2. `F:\00-Ticenpi-SaaS\deploy\docs\system\SYSTEM_OWNERSHIP.md`
3. `F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md`
4. `F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_TO_STAGING.md`
5. `F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TO_PRODUCTION.md`
6. `F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TEST_IDENTITY_CONTRACT.md`
7. `F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc\CLAUDE.md`
8. `F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc\591\CLAUDE.md`
9. `F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc\591\docs\W6-591-EXECUTION-LOG.md`
10. `F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc\591\docs\W6-591-SOURCE-RECONCILIATION.md`

證據優先序：

```text
當次 live read-only evidence
> 已提交且可定位的 repo/runtime evidence
> 上述六份 canonical system docs
> 591 execution log
> 本計畫的 2026-09-26 snapshot
> 舊 prompt、聊天敘述、記憶
```

若新證據與本計畫不同，不自行改變架構。先分類：

- `KEEP`：已完成且符合 canonical contract。
- `ADJUST`：同一設計，只需最小修正。
- `CONFLICT`：會碰 Production、改資料主從、改商用／Seat 架構、改 artifact 身分或破壞回滾。

任何 `CONFLICT`：`HARD_STOP: CURRENT_STATE_CONFLICT`，列出檔案、commit、runtime 實值與衝突條款。

## 2. 2026-09-26 已驗證基準

### 2.1 591 RC

```text
BRANCH = codex/591-w6-rc-20260925
HEAD = 7c3afc3f4454528c7a18df2af181277160d0dfe5
WORKTREE = CLEAN
REMOTE_BRANCH = SAME HEAD
SOURCE_COMMIT_FOR_ARTIFACT = 3d23aaecef70e0adb0310097313bb810521f9c46
SOURCE_CI_RUN = 36099345595 (success)
IMAGE = ghcr.io/tcp-git-313/ticenpi-591
IMAGE_DIGEST = sha256:1b64a2f947b4aeeb45059221605f5fe92a5672cd98b6d98bcfb40efa3a045454
PLATFORM = linux/arm64
PIN_COMMIT = c595bf74e53fe192a97b1e166700ea193dbc012c
LATEST_EXACT_CI_RUN = 36109019041 (success, HEAD 7c3afc3)
```

已完成：Production 58/58 source reconciliation、auth/JWKS、Commercial＋Central Seat gate、HTTP/WS gate、`/api/live`、`/api/ready`、runtime identity、cookie isolation、單一 immutable image、digest pin、exact-commit CI、中央 service registry、H2 fixture。

尚未完成：Local Docker 實跑、Staging VPS env、中央腳本同步到 VPS、relay、DNS/tunnel、OAuth redirect、Staging deploy、rollback drill、真人 E2E、release evidence、Production 形式轉換與 go-live。

### 2.2 H2 fixture 已完成，不得重建

```text
CUSTOMER_ID = 5b3b204b-25a5-411b-aacb-ca8b4c617482
PRODUCT_CODE = 591
ENTITLEMENT_STATUS = active
SEAT_LIMIT = 2
ASSIGNED_USER_UUID = 2ef49d05-c122-480d-bee5-64d5c2b54663
UNASSIGNED_USER_UUID = d5e9b467-6a5e-4ebb-9c32-061dd2f30ad4
H2 = PASS
```

這兩個 UUID 是本次從 active memberships 解析出的 fixture identity reconciliation；舊值 `05f65e05`／`47893876` 只是無法解析的歷史 hash，不再是 identity contract。

只做 read-back。若狀態漂移，使用既有 canonical RPC 最小、冪等 reconcile：`admin_set_entitlement`、`assign_product_seat`、必要時 `release_product_seat`。禁止直接寫表、禁止建立 `auth.users`、禁止碰 Production、禁止使用 `app.ticenpi.com` 建 Staging fixture。

### 2.3 中央 deploy source

```text
LOCAL_MAIN_HEAD = b48716fe2e7cf7117b59b58ad5cc2ec1d8c24a01
LOCAL_MAIN_VS_ORIGIN_MAIN = ahead 5, behind 0
591_CORE_SOURCE_FIX = 5f33224
591_CORE_DOC_UPDATE = b48716f
UNRELATED_UNTRACKED = HANDOFF-20260922-cookie-isolation-and-ci-gate.md
```

`server/critical-smoke.sh` 的 `591-core` 已補回，綁定：

```text
service = 591runtimestaging
compose project = ticenpi-591-runtime-staging
compose service = app
command = docker exec -w /app/591 <container> python critical_function_check.py
```

已有跨平台 regression test `tests/test_critical_smoke_profiles.py`。驗證紀錄：該 test PASS、shell syntax PASS、591 manifest PASS。全中央測試為 `113 passed, 28 skipped, 2 failed`；兩個 fail 都是未改動的 Windows PowerShell 5.1 測試環境找不到 `Get-FileHash`，不可誤報成 591 smoke regression。

中央 main 尚未 push；VPS 也尚未證明已含修復後的 `591-core`。source 修復不等於 runtime 修復。

### 2.4 已知前置缺口

1. `scripts/w6/591-staging-vps-prereqs.ps1` 直接把多行 script 當 ssh argument，使用者實跑時在 regex `(^|:)` 附近出現 remote bash syntax error；需改成經 stdin 執行 `bash -s --`，不得弱化任何檢查。
2. `scripts/w6/Use-591StagingSecrets.ps1` 目前呼叫中央 loader `-Service 591`；該 service 是 Production `S591_*` contract，會得到錯誤 project ref。不得以 rename/copy Production 值繞過。
3. 中央 loader 目前沒有經證明的 `591-staging` contract。必須由中央 secrets owner 提供 canonical 介面，至少輸出 Staging 的 `SUPABASE_URL`、publishable/anon key、service-role 相容名稱與 591 Staging 專用 Fernet key；來源名稱不可猜。
4. Staging release root 與 env file 尚不存在；8891 最後一次唯讀檢查為 free。
5. `.release-evidence\591` 尚不存在。
6. 先前終端曾意外顯示中央 secret values。所有受影響的 service-role／DB／Fernet／R2 等秘密，在任何部署前必須由使用者完成 rotation；不得在 log、prompt、commit 或對話重現值。

## 3. 不可變架構決策

1. 主工作樹 `F:\00-Ticenpi-SaaS\Ticenpi591` 只讀；所有產品修改只在 RC worktree。
2. Staging source 只能是 clean、已提交、已 push、exact-commit CI 綠的 RC。
3. Staging 使用單一 ARM64 image，frontend/backend 不拆 artifact；compose 必須 pin digest，不用 mutable tag。
4. Staging project 固定 `jlsqjvehwblkeuycjoyj`；任何 `zfpkxsulehkbyuqkkcml` 或其他 project ref 出現在 Staging process/env/path，立即停止。
5. Staging 永遠 `DRY_RUN_SUBMIT=true`、`AUTOPOST_REQUIRE_AUTH=1`、`AUTOPOST_SEAT_REQUIRED=1`。
6. Staging 使用 `127.0.0.1:8891:8890`、compose project `ticenpi-591-runtime-staging`、network `192.168.59.0/24`、output named volume，不掛 Production output。
7. Staging Playwright 出網只走 `warp-proxy-relay-591runtimestaging.service`，relay 只 bind `192.168.59.1:40000`。
8. Auth 只接受 Staging Supabase issuer/JWKS；不得共享 JWT secret。
9. Commercial entitlement 與 Central Seat 都 fail closed；中央服務不可用回 503，不降級成允許。
10. H2 fixture 沿用已完成狀態，不因 prompt 重建。
11. Central secret loader 是唯一秘密來源；只能輸出 `PRESENT/MISSING` 與 project ref。
12. `591-core` 是 Staging critical smoke；Production 現有 `591-yuct-short` 在轉換前不改。
13. Staging 與 Production DB/storage 不同步；測試、seed、Docker、migration 不得回寫 source 或 local data。
14. Production 現況為 systemd release `20260912-163944`，不可重現且無 Commercial/Seat gate。不可把它直接標記可升級。
15. Staging Docker → Production systemd 目前沒有 artifact equivalence。Production 只能在使用者選定 §12 的 P-A 或 P-B 後進行。
16. Production 上市前不得 rebuild 已接受的 application artifact；若任何 app source 改動，必須回到 CI、產生新 digest、重跑完整 Staging E2E 與 ACCEPTED。

## 4. 權限邊界與人工關卡

Luna 可自行做：read-only 盤查、RC source edit/test/commit/push、中央 deploy source edit/test/commit；H1 RC push 已完成。不得把「可做 source 變更」理解成可做 VPS、Cloudflare、Supabase Dashboard 或 Production mutation。

以下必須停下來，給使用者一段清楚指令／後台步驟，等使用者回覆「完成」後先唯讀驗證：

| Gate | 使用者動作 |
|---|---|
| H-C0 | 若中央 main 的 5 個 local commits 都經確認，push `F:\00-Ticenpi-SaaS\deploy` main；不得由 Luna 在不知道另外 3 個 commit 擁有者意圖時直接推 shared main |
| H-S1 | 中央 secrets owner 建立／核准 canonical `591-staging` contract，並 rotation 先前可能曝露的秘密；不把值貼進對話 |
| H3 | VPS 建立 Staging release root/env、安裝比對後的中央 merged scripts；只能透過正式中央流程，不整份盲覆蓋 |
| H4 | 建立 591 Staging relay、Cloudflare DNS/tunnel route，完成後 Luna 唯讀驗證 |
| H5 | Supabase Staging Redirect URLs 與 Google OAuth origin；Site URL 不改 |
| H6 | 執行真正 Staging deploy 與 rollback drill |
| H7 | 真人 Staging 登入、assigned/unassigned 驗收，以及提供自己的 591 測試帳號並明確同意 E9 |
| H-P0 | 明確選 P-A 或 P-B，並回覆「Production 授權」；未收到前不得做任何 Production mutation |
| H-P1 | Production 維護窗口：rotation/env、systemd/compose 切換、deploy、rollback 操作 |
| H-P2 | 真人 Production canary 與是否實際送出刊登的決定 |

H2 已完成，不再是人工 blocker。

## 5. 通用執行規則

- 每次修改前先 `git status --short`；目標檔若有非本任務未提交變更：`HARD_STOP: SHARED_FILE_BUSY`。
- 只 `git add -- <明確檔案>`；禁止 `git add .`、`git add -A`、force push、reset/clean/stash 使用者 WIP。
- 每個 commit 結尾：`Co-Authored-By: GPT-6 Luna <noreply@openai.com>`。
- 每個工作包都更新 `591/docs/W6-591-EXECUTION-LOG.md`；若 log 更新會污染 application artifact 判定，做獨立 docs-only commit，CI 必須證明不 rebuild image。
- 所有命令輸出須 redact；秘密只顯示名稱與 `PRESENT/MISSING`，email 只能遮罩，不輸出完整值。
- 不碰舊容器 `ticenpi-staging-591-backend-1`、`p591-staging-20260925` worktree、TicenpiSign、HUB 未提交 WIP、Production runtime。
- 禁止 `docker compose down -v`、刪 volume、刪 release、改 Production DNS、直接 SQL INSERT fixture、使用 app.ticenpi.com 建 Staging fixture。
- 同一原因 CI 連敗 2 次、部署自動回滾 2 次、或腳本 runtime 與剛比對版本不同，立即 HARD_STOP，不試第三次。

## 6. Phase 0 — 只讀重驗與 KEEP／ADJUST／CONFLICT

先執行：

```powershell
git -C F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc status --short
git -C F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc log -8 --oneline
git -C F:\00-Ticenpi-SaaS\deploy status --short
git -C F:\00-Ticenpi-SaaS\deploy fetch origin main
git -C F:\00-Ticenpi-SaaS\deploy rev-list --left-right --count origin/main...main
git -C F:\00-Ticenpi-SaaS\deploy log -8 --oneline
gh run view 36109019041 -R tcp-git-313/ticenpi591 --json headSha,status,conclusion,url
gh run view 36099345595 -R tcp-git-313/ticenpi591 --json headSha,status,conclusion,url
Test-Path F:\00-Ticenpi-SaaS\.release-evidence\591
```

再做唯讀 VPS 檢查：release root、env file presence、8891 owner、目前 `/opt/ticenpi/deploy/{deploy.sh,critical-smoke.sh}` hash，以及腳本內是否有 `591-core`。不輸出 env value。

Phase 0 首次回報固定格式：

```text
RC_STATE =
CENTRAL_DEPLOY_STATE =
CENTRAL_REMOTE_DELTA =
CI_7C3AFC3 =
ARTIFACT_3D23AAE =
VPS_RELEASE_ROOT =
VPS_ENV_FILE =
PORT_8891 =
VPS_591_CORE =
RELEASE_EVIDENCE =
CLASSIFICATION = KEEP / ADJUST / CONFLICT
PHASE_0 = PASS / HARD_STOP
```

## 7. Phase 1 — 收斂剩餘 source 與 secret contract

### WP-1A：修復 VPS read-only prereq transport

只改：

- `scripts/w6/591-staging-vps-prereqs.ps1`
- `591/docs/W6-591-EXECUTION-LOG.md`

把 remote script 經 stdin 傳給 `ssh ... bash -s --`，避免 OpenSSH argument 重組吃掉 regex 引號。保留 read-only 與所有 fail-closed 檢查；新增可離線測試的 command-construction seam 或最小測試，證明 regex 不再被本機 shell 展開。

驗證：PowerShell parse、mock transport test、實際唯讀 SSH。若 actual script 回報 root/env missing 但 exit 0，這是正確的 prereq 結果；只有 transport/forbidden ref 才是 script failure。

commit：

```text
fix(591): make staging VPS prereq transport shell-safe
```

### WP-1B：建立 canonical 591 Staging secret contract

先唯讀確認中央 loader 是否已出現正式 `591-staging`。若沒有，停在 H-S1；不得自己拼接 `sign-staging`、`orc-staging`、`S591_*`，也不得把 Production Fernet 當 Staging 值。

contract 必須具備：

```text
SERVICE = 591-staging
PROJECT_REF = jlsqjvehwblkeuycjoyj
BARE OUTPUTS = SUPABASE_URL, SUPABASE_ANON_KEY, SUPABASE_SERVICE_KEY, AUTOPOST_FERNET_KEY
PRODUCTION_REF_PRESENT = NO
PERSISTENCE = process scope only
LOG OUTPUT = names + PRESENT/MISSING + project ref only
```

由中央 secrets owner 決定 source variable names，並將它們納入中央 loader/sync 正式 allowlist。Luna 不修改已有他人 WIP 的 HUB 檔案。

owner 完成後，產品 helper 改為只呼叫 `-Service 591-staging -Scoped`，先清除所有可能殘留的裸名與 Production-prefixed env，再載入，再驗 project ref；不可先載入 Production 再覆寫。

只改：

- `scripts/w6/Use-591StagingSecrets.ps1`
- 對應的 offline test
- `591/docs/W6-591-EXECUTION-LOG.md`

commit：

```text
fix(591): consume canonical staging secret contract
```

### WP-1C：發布中央 591-core 修復

H-C0 前先證明：

- `origin/main` 沒有新增 divergence。
- local ahead commits 精確為已審核的 `572df08 e16f95a 5d292b2 5f33224 b48716f`，或列出實際序列供使用者核准。
- 不相關未追蹤 handoff 未被 stage。
- `tests/test_critical_smoke_profiles.py` PASS。
- `python server/manifest_tool.py check --service 591runtimestaging --manifest services.yaml` PASS。
- WSL `bash -n server/critical-smoke.sh` 與 `server/deploy.sh` PASS。

由使用者執行 push。完成後 Luna 用 `git ls-remote`／GitHub API 唯讀確認 remote main 含 `5f33224` 與 docs commit。沒有 remote proof，不得安裝到 VPS。

### Phase 1 出口

```text
PREREQ_SCRIPT = PASS
SECRET_CONTRACT_591_STAGING = PASS
CENTRAL_591_CORE_REMOTE = PASS
ROTATED_SECRETS = USER_CONFIRMED
PHASE_1 = PASS
```

## 8. Phase 2 — Local Docker 與 exact-commit CI

1. 啟動 Docker Desktop／可用 ARM64 build path；先跑 read-only daemon check。
2. 透過 canonical `591-staging` loader 注入 process env；只印 presence/ref。
3. 執行產品現有 unit/frontend/policy tests。
4. 執行 `scripts/w6/start_local_docker.ps1` 與 `verify_local_docker.ps1`。
5. 驗證 image runtime：
   - `/api/live` 200。
   - `/api/ready` 在真實 Staging dependencies 可用時 200；缺依賴時 503 且不洩漏深度診斷。
   - `/api/health` identity：environment=staging、service=591runtimestaging、project ref=JLSQ、release/git SHA 非空、dry-run=true。
   - 無 Bearer 的受保護 API 為 401。
   - output 寫入只落 named volume，repo/source 無新檔。
6. 停止 local compose 但不刪 volume；確認 RC 只含預期修改。

若 WP-1A/WP-1B 只有 script/config/test 變更，CI image-scope 必須 false、ARM64 build job 必須 skipped，沿用既有 artifact digest。若任何 application/Docker dependency 改動，必須產新 ARM64 digest，更新 compose pin，並以新 source commit 重新走後續所有驗收。

每個 RC commit push 後都要等待 exact-commit CI；不得用較舊綠燈代替。

Phase 2 出口：

```text
LOCAL_DOCKER = PASS
RC_HEAD = <40hex>
EXACT_CI_RUN = <id>
EXACT_CI = PASS
IMAGE_REBUILT = YES/NO
ACCEPTANCE_DIGEST = sha256:<64hex>
COMPOSE_PIN_MATCH = YES
PHASE_2 = PASS
```

## 9. Phase 3 — Staging 基礎設施人工關卡

### H3：VPS env 與中央腳本

Luna 先提供使用者確切步驟，使用者在 VPS／中央正式工具執行：

1. 建立 `/opt/ticenpi/591-runtime-staging` 所需 shared/env 路徑，不碰 `/opt/ticenpi/591`。
2. 透過核准的 `591-staging` sync 流程寫入 env；env 必須包含 manifest required keys 與：
   `TICENPI_ENVIRONMENT=staging`、`TICENPI_SERVICE_NAME=591runtimestaging`、`TICENPI_SUPABASE_PROJECT_REF=jlsq...`、`DRY_RUN_SUBMIT=true`、`AUTOPOST_REQUIRE_AUTH=1`、`AUTOPOST_SEAT_REQUIRED=1`。
3. 先下載／比對 VPS `deploy.sh`、`critical-smoke.sh` 與 remote main。若 VPS 有額外未合併功能，停止；不得整份覆蓋。
4. 安裝 reviewed merged scripts，hash 留證。

完成後 Luna 只讀驗證 presence、非秘密設定、forbidden ref absent、`591-core` branch、8891 owner。任何值錯誤：`HARD_STOP: STAGING_ENV_IDENTITY_MISMATCH`。

### H4：relay 與 public route

使用者建立 `warp-proxy-relay-591runtimestaging.service`，只 bind `192.168.59.1:40000`，以及 `591-staging.ticenpi.com` 的 Cloudflare DNS/tunnel route 到 `http://127.0.0.1:8891`。不得修改 Production route。

Luna 驗證 systemd unit state、listen address、container network reachability、DNS resolution、HTTPS response；不能只看 DNS record。

### H5：OAuth

使用者在 Staging Supabase `jlsq...` 加：

```text
https://591-staging.ticenpi.com/autopost-app/
https://591-staging.ticenpi.com/**
http://localhost:18891/**   # 只在 local E2E 需要時
```

Google OAuth 加對應 Staging origin/redirect；Supabase Site URL 不改。Luna 以登入 redirect 實際 network trace 驗證，不截取 token。

Phase 3 出口：H3/H4/H5 全 PASS。

## 10. Phase 4 — Staging DryRun、Preflight、部署與回滾演練

Luna 先執行唯讀／無 mutation：

```powershell
cd F:\00-Ticenpi-SaaS\deploy
python server\manifest_tool.py check --service 591runtimestaging --manifest services.yaml
.\deploy.ps1 591 staging -SourceOverride F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc -DryRun
.\deploy.ps1 591 staging -Preflight
```

確認打包來源是 clean RC、CI gate 綁 exact HEAD、compose pin 是 acceptance digest、required env presence PASS、VPS candidate 不含 `.env`/local data、Production service 不在 plan 內。

### H6：真正部署

使用者執行：

```powershell
cd F:\00-Ticenpi-SaaS\deploy
.\deploy.ps1 591 staging -SourceOverride F:\00-Ticenpi-SaaS\.worktrees\591-w6-rc
```

Luna 監看並驗證：

- candidate preflight PASS。
- image 以 digest 拉取，不 build。
- container platform arm64。
- port 只綁 127.0.0.1:8891。
- health PASS。
- `591-core` PASS。
- post-deploy audit PASS。
- release id、git SHA、digest、deploy config commit 可定位。

先 `record-deployed`，不得直接 ACCEPTED：

```powershell
.\release.ps1 591 record-deployed `
  -ReleaseId <release-id> `
  -SourceCommit <artifact-source-commit> `
  -DeployConfigCommit <rc-head-or-deploy-config-commit> `
  -CiRunId <artifact-ci-run> `
  -BackendDigest <same-digest> `
  -FrontendDigest <same-digest> `
  -RuntimeIdentityVerified -HealthVerified -AuthVerified -CommercialVerified
```

接著使用者執行 rollback drill：列出 releases，回滾上一版，再重新部署同一 candidate。若第一次部署沒有可回滾前一版，建立明確的 `ROLLBACK_DRILL=BLOCKED_NO_PRIOR_RELEASE`，不可假造 PASS；在第二次可用 release 後補演練。

rollback 後用 `record-rollback` 記錄 restored release；重新部署產生新的 release id，再 record-deployed。最終 running release 必須是 acceptance candidate。

## 11. Phase 5 — Staging 驗收矩陣與 ACCEPTED

### 11.1 自動／唯讀驗證

- runtime identity：service、environment、project ref、git SHA、release id、digest 全符合。
- `/api/live`、`/api/ready`、`/api/health`。
- source cookie guard 與實際 response header/cookie policy。
- H2 fixture read-back：entitlement active、seat_limit 2、一 assigned、一 unassigned。
- anonymous：401。
- wrong issuer／Production token：401，不得觸發 Production query。
- assigned member：可進產品、可保存個人 credentials、可擷取、可 dry-run submit。
- unassigned member：403 Central Seat required。
- platform admin：依 canonical contract 驗證豁免或管理路徑，不拿來替代一般 member E2E。
- 中央 Commercial/Seat 不可用：503，不 fail open。
- 同 Customer 兩個 user 的 `autopost_*` 資料仍 USER scope，不互讀。

### 11.2 H7 真人 E2E

使用者在 `https://591-staging.ticenpi.com/autopost-app/` 執行：

1. Google 登入與 refresh/reload。
2. Assigned user 正常進入；unassigned user 被 Seat gate 擋下。
3. 使用使用者提供的 591 測試帳號執行 E9：取得真實物件、填表到最後確認頁、截圖。
4. 呼叫 submit 必須回 dry-run；591 後台必須沒有新刊登。
5. 不把 591 帳密、token、完整 email 寫入 log。

使用者拒絕 E9 或未提供帳號：

```text
STAGING_E2E = PARTIAL
STAGING_RELEASE_ACCEPTED = NO
```

不得把 health、smoke、fixture 或 RPC presence 當 E9 PASS。

### 11.3 Record ACCEPTED

只有所有矩陣與 E9 PASS，才執行：

```powershell
.\release.ps1 591 record-accepted `
  -ReleaseId <final-running-release-id> `
  -E2eVerified -RuntimeIdentityVerified -HealthVerified -AuthVerified -CommercialVerified
.\release.ps1 591 status
.\promote.ps1 591 production -DryRun
```

此時 `promote -DryRun` 預期仍因 Production systemd 無同 digest compose 而 BLOCKED；這是正確安全狀態，不是 Staging 失敗。

Phase 5 出口：

```text
STAGING_DEPLOYED = YES
STAGING_E2E = PASS
STAGING_RELEASE_ACCEPTED = YES
ACCEPTED_STAGING_POINTER = PRESENT
PROMOTE_DRY_RUN_BEFORE_PRODUCTION_DESIGN = BLOCKED_EXPECTED
```

到此先停止並等待 H-P0。

## 12. Phase P — Production 形式選擇與授權

未收到使用者當次明確文字 `Production 授權` 前，只能做 read-only preflight 與 plan，不得建立 Production env、停 systemd、改 nginx、改 DNS、部署、回滾或真實刊登。

使用者必須選一條，不能混用：

### P-A（推薦）：把 Production 轉為同一 ACCEPTED Docker artifact

條件：使用者接受一次受控形式轉換，保留現有 systemd release 作立即 rollback。

source/config 工作包：

1. RC 新增 `deploy/production/compose.yml`，image pin 必須等於 accepted Staging digest；`127.0.0.1:8890:8890`；`TICENPI_ENVIRONMENT=production`；`DRY_RUN_SUBMIT=false`；auth/seat gate ON；Production ref；bind mount `/opt/ticenpi/591/shared/591/output:/app/591/output`；network `192.168.60.0/24`；proxy `socks5://192.168.60.1:40000`。
2. 中央 `services.yaml` 的 `591` 從 systemd 改 docker-compose；domain/port/auth cookie 不變；env 使用既有 Production canonical secret contract。
3. 中央新增 `591-core-production` profile，service 只允許 `591`，執行同一 image 內 `/app/591/critical_function_check.py`；不要改 `591-core`。
4. 加 regression tests、manifest tests、promotion gate tests。
5. config-only exact CI 必須證明不 rebuild application image。
6. `promote.ps1 591 production -DryRun -SourceOverride <clean-rc>` 必須 PASS，並證明 compose digest＝accepted pointer。

### P-B：保留 systemd，先建立可重現等價 package/tooling

只有使用者拒絕 Docker 轉換時使用。先完成：

- 從 accepted source 產 immutable、可驗 hash 的 systemd package。
- 建立 Docker accepted artifact 與 systemd package 的可重現 equivalence proof；不可只說 source SHA 相同。
- promotion gate 能驗 package digest、source commit、dependencies、frontend assets、Playwright runtime 全等價。
- 在 Staging 以 systemd 形態重跑完整 E2E 並重新 ACCEPTED。

沒有上述工具：`HARD_STOP: NO_DOCKER_TO_SYSTEMD_EQUIVALENCE`。不得直接把 RC 複製到 Production。

## 13. Phase P1 — Production read-only preflight

授權後仍先唯讀：

- 確認 accepted pointer、digest、source commit、CI run。
- 確認 Production current 仍是 `20260912-163944`；systemd unit、nginx、port 8890、output、env、disk、memory。
- 只檢查 Production secret presence/ref，不讀值。
- 備份／記錄 current symlink、unit enabled/active、nginx hash、env hash、output mount、最近 release 列表。
- 確認 rollback 命令可以在窗口內恢復 systemd release。
- 確認 Production H2 商用 fixture 不沿用 Staging；真人 Production entitlement/seat 只讀驗證 canonical 狀態，不創建測試後門。

任一不符：`PRODUCTION_PREFLIGHT=FAIL`，停止。

## 14. Phase P2 — Production 上市與回滾

以下由使用者在 H-P1 維護窗口執行，Luna 提供逐步指令並即時只讀驗證。

P-A 順序：

1. 建立 Production relay `warp-proxy-relay-591.service`，只 bind `192.168.60.1:40000`。
2. 同步 Production env／中央 merged scripts；不改 Staging env。
3. 保留 systemd unit、current symlink、release files；只 stop/disable runtime，不刪除。
4. 執行 `promote.ps1 591 production`；不得直接繞過 promotion gate 呼叫 deploy。
5. 驗證 digest、port、health、ready、runtime identity、`591-core-production`、public HTTPS、auth、Commercial/Seat。
6. 若任何 gate fail，停止 compose，恢復 systemd enabled/active，驗證 8890、health/public route，記錄 rollback evidence。

不得在同一失敗窗口即席改 app source後重試。source fix 必須回 CI → Staging → E2E → ACCEPTED。

## 15. Phase P3 — Production canary 與 GO LIVE

H-P2 由使用者決定 canary 帳號、物件與是否真的送出一筆刊登。Luna 不接收或回顯帳密。

最低驗收：

- 真實 Production Google 登入與 session refresh。
- 正常 entitlement＋assigned Seat 可進；無 Seat 被拒。
- 存取自己的 credentials/data，不越權讀另一 user。
- 真實擷取成功。
- 若使用者同意送出：只送一筆 canary，使用者在 591 後台確認結果並決定保留／刪除。
- error/latency/log 無 secret、無 Production project mismatch、無大量重試。

只有真人 canary PASS 才記錄 Production ACCEPTED：

```powershell
.\release.ps1 591 record-deployed -Environment production <同 accepted digest 與實際 release 參數>
.\release.ps1 591 record-accepted -Environment production -ReleaseId <id> `
  -E2eVerified -RuntimeIdentityVerified -HealthVerified -AuthVerified -CommercialVerified
.\release.ps1 591 status
```

未做真人 canary：只能 `PRODUCTION_DEPLOYED=YES`、`PRODUCTION_ACCEPTED=NO`，不得宣告上市完成。

## 16. HARD_STOP 清單

- `BASELINE_DRIFT`：RC artifact/pin/CI 身分不可解釋地改變。
- `SHARED_FILE_BUSY`：中央或 RC 目標檔有他人未提交改動。
- `CURRENT_STATE_CONFLICT`：live evidence 與不可變架構決策衝突。
- `NO_CANONICAL_591_STAGING_SECRET_CONTRACT`：只能取得 Production S591 secret 或需猜 source name。
- `SECRET_ROTATION_REQUIRED`：先前可能曝露的秘密尚未由使用者確認 rotation。
- `STAGING_ENV_IDENTITY_MISMATCH`：Staging env 不是 JLSQ、含 Production ref、或 dry-run 不為 true。
- `VPS_SCRIPT_DRIFT`：VPS script 有中央 main 沒有的變更。
- `CI_REPEAT_FAILURE`：同因兩次。
- `DEPLOY_REPEAT_ROLLBACK`：同因兩次自動回滾。
- `RUNTIME_IDENTITY_MISMATCH`：SHA/digest/release/service/env/ref 任一不符。
- `E2E_INCOMPLETE`：E9 或 assigned/unassigned/auth/data isolation 未 PASS。
- `PRODUCTION_NOT_AUTHORIZED`：未收到當次授權。
- `NO_DOCKER_TO_SYSTEMD_EQUIVALENCE`：選 P-B 但沒有可重現等價工具。
- `NO_ROLLBACK_PATH`：切換前不能證明可恢復 `20260912-163944`。

## 17. 已知缺陷 K1–K7

最終回報不得省略：

```text
K1 Production 前端目前無自有登入改善方案；形式轉換不等於 UI 改善。
K2 截圖路由的歷史免驗證／UUID 猜測風險仍需另案處理。
K3 WebSocket token 仍走 query string。
K4 autopost_* 對 anon/authenticated 的歷史 grants 需另案最小權限審計。
K5 舊 Staging 容器存在，但本計畫禁止觸碰。
K6 VPS scripts 與中央 repo 曾漂移；必須每次比對 hash。
K7 login-gate.ts 以寫死 email 控制測試按鈕顯示，僅 UI 用途但仍是已知債務。
```

另列本次新增已知缺陷：

```text
K8 canonical 591-staging secret contract 尚待中央 owner 完成。
K9 central deploy main 尚未 remote-published；local source PASS 不等於 VPS 可用。
K10 Windows PowerShell 5.1 測試環境缺 Get-FileHash，中央完整 suite 非全綠；需平台另案修測試 harness。
```

## 18. 每一階段回報格式

每次停在人工關卡或 HARD_STOP，都要給：

```text
CURRENT_PHASE =
LAST_VERIFIED_COMMIT =
COMPLETED =
PENDING =
BLOCKER =
USER_ACTION = <在哪裡、確切步驟、不可做什麼>
AFTER_COMPLETION_I_WILL_VERIFY =
PRODUCTION_MUTATION = NO/YES (只有授權後可 YES)
```

最終輸出：

```text
PRODUCT = 591
RC_HEAD =
ARTIFACT_SOURCE_COMMIT =
CI_RUN_ID =
IMAGE_DIGEST =
DEPLOY_CONFIG_COMMIT =
CENTRAL_DEPLOY_COMMIT =
STAGING_RELEASE_ID =
STAGING_RUNTIME_IDENTITY = PASS/FAIL
STAGING_HEALTH = PASS/FAIL
STAGING_SMOKE_591_CORE = PASS/FAIL
STAGING_AUTH = PASS/FAIL
STAGING_COMMERCIAL = PASS/FAIL
STAGING_SEAT = PASS/FAIL
STAGING_E2E = PASS/PARTIAL/FAIL
STAGING_RELEASE_ACCEPTED = YES/NO
PRODUCTION_PATH = P-A/P-B/NOT_SELECTED
PRODUCTION_AUTHORIZATION = YES/NO
PRODUCTION_RELEASE_ID =
PRODUCTION_RUNTIME_IDENTITY = PASS/FAIL/NOT_RUN
PRODUCTION_CANARY = PASS/PARTIAL/FAIL/NOT_RUN
PRODUCTION_DEPLOYED = YES/NO
PRODUCTION_ACCEPTED = YES/NO
GO_LIVE = YES/NO
ROLLBACK_TARGET =
ROLLBACK_PROOF = PASS/FAIL/NOT_RUN
OPEN_HUMAN_GATES =
K1_K10 = <逐項狀態>
HARD_STOPS =
```

只有 `PRODUCTION_ACCEPTED=YES` 且 rollback proof PASS，才可輸出 `GO_LIVE=YES`。

現在開始：先完整讀 §1 文件，執行 Phase 0，只回報 KEEP／ADJUST／CONFLICT 與實際值；若沒有 CONFLICT，再依序完成 Phase 1。不要從頭重做 H2、不要碰 Production。

<!-- END OF 591 LUNA BUILD-TO-GO-LIVE EXECUTION PLAN -->
