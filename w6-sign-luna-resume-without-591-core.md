# 交辦：W6-SIGN 接續完成建置與交付｜避開 591-core 共用檔衝突｜GPT-5.6 Luna

```text
PROJECT_NAME   = SIGN
PROJECT_REPO   = F:\00-Ticenpi-SaaS\TicenpiSign
RC_WORKTREE    = F:\00-Ticenpi-SaaS\.worktrees\sign-w6-rc
RC_BRANCH      = codex/sign-w6-rc-20260925
PLAN           = F:\_prompt-publish\W6-SIGN-luna-executor-plan.md
EXECUTE_PROMPT = F:\_prompt-publish\W6-SIGN-luna-execute-prompt.md
MODE           = RESUME + EXECUTE
DATE_BASELINE  = 2026-09-26 Asia/Taipei
```

使用者已核准 W6-SIGN v2 計畫與 Production 方案 A。本提示詞是目前執行狀態的接續交辦，新增一條跨產品協作邊界：**SIGN 執行者不得修改、恢復、刪除、測試或安裝 `591-core`；該 profile 由並行中的 591 執行者負責。**

這不是授權跳過 SIGN smoke。`sign-core` 仍是 SIGN Staging／Production 的必要驗收；在共用平台腳本尚未由其 owner 整合完成前，SIGN 可以繼續所有不碰該共用檔的工作，但不得開始 Staging deployment。

## 1. 開工前完整閱讀

依序完整讀完，不得只讀摘要：

1. `F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md`
2. 同資料夾：
   - `SYSTEM_OWNERSHIP.md`
   - `PRODUCT_INTEGRATION_CONTRACT.md`
   - `PRODUCT_TO_STAGING.md`
   - `STAGING_TO_PRODUCTION.md`
   - `STAGING_TEST_IDENTITY_CONTRACT.md`
3. `F:\_prompt-publish\W6-SIGN-luna-executor-plan.md`（v2，23 節）
4. `F:\_prompt-publish\W6-SIGN-luna-execute-prompt.md`
5. `C:\Users\Tu\.claude\CLAUDE.md` 最上方 CURRENT STATE。

規則優先序：使用者本次「避開 591-core」指示 ＞ v2 的權限／Production Gate／HARD_STOP ＞ `docs/system` 最新合約 ＞本提示詞的狀態摘要。即時重驗與摘要不符時，以實際值為準並回報 drift。

## 2. 跨產品共享檔案邊界（最高優先）

### 2.1 SIGN 執行者禁止事項

不得修改或提交下列檔案：

- `F:\00-Ticenpi-SaaS\deploy\server\critical-smoke.sh`
- `F:\00-Ticenpi-SaaS\deploy\server\deploy.sh`

不得：

- 新增、恢復、刪除或重寫 `591-core` 分支；
- 從任何舊 commit cherry-pick／複製 `591-core`；
- 修改 591 repo、591 worktree、591 container、591 port、591 evidence；
- 以 SIGN 名義安裝目前不完整的共用 smoke 腳本到 VPS；
- 把 `services.yaml` 的 `criticalSmokeProfile: sign-core` 清空、改名或 broad-exclude；
- 用健康 200、手動 `docker exec`、mock 或跳過 smoke 取代 deploy pipeline 的 `sign-core` PASS。

### 2.2 591 執行者的責任

591 執行者自行在中央 `ticenpi-platform` repo 處理 `591-core`。SIGN 執行者不審核 591 功能語意，只在依賴交付後做以下唯讀相容性確認：

1. 中央 repo 的 `server/critical-smoke.sh` 同時存在 `591-core)` 與 `sign-core)`；
2. `server/deploy.sh` allowlist 同時含 `591-core` 與 `sign-core`；
3. `bash -n` 兩支腳本 PASS；
4. SIGN 相關分支仍是：
   - `signruntimestaging` → compose project `ticenpi-sign-runtime-staging`
   - `sign` → compose project `ticenpi-sign`
   - backend container → `node src/critical-check.mjs`
5. 591 執行者回報的中央 commit 已推送到其指定 canonical branch。

禁止 SIGN 執行者替 591 補 commit。若依賴尚未完成，標：

```text
PLATFORM_SMOKE_DEPENDENCY = WAITING_591
```

然後繼續不依賴 VPS smoke 安裝的 SIGN-only 工作；到 Staging deployment barrier 才停止。

### 2.3 VPS 安裝責任

共用腳本只能安裝一次，而且必須是 owner 已整合、同時保留所有既有 profile 的版本。VPS 寫入仍由使用者執行。SIGN 執行者只負責：

- 安裝前唯讀比對 VPS 與 owner 交付版本；
- 確認 diff 沒有預期外變更；
- 準備包含備份、`install`、`bash -n`、SHA256 與 profile presence 的完整 U2-b 指令；
- 標 `WAITING_USER: U2-b` 後停止；
- 使用者完成後用唯讀 SSH 驗證。

在 owner 合併版未就緒前，先前任何 U2-b 指令都作廢，不得要求使用者執行。

## 3. 已完成、必須沿用的 SIGN 證據

先唯讀重驗；相符就 KEEP，不重做、不 rebase、不改寫歷史。

### 3.1 SIGN RC

```text
RC HEAD / origin = 6fbfed5e3533b53bc9bc8be2118983a874749fee
branch           = codex/sign-w6-rc-20260925
CI gate run      = 36109828775（backend/frontend/policy/arm64-images 全綠）
README ref fix   = 2b8c68825f0538984ea0546e479f6fcf3b05b8ab
compose pin base = 5396835da942c0486c832c56ba86c5f8f0af582d
deploy config    = 6fbfed5e3533b53bc9bc8be2118983a874749fee
```

部署使用的 frozen artifact 不是 config-only CI 新建的未使用映像。正確 provenance：

```text
SOURCE_COMMIT = 5396835da942c0486c832c56ba86c5f8f0af582d
CI_RUN_ID     = 36109267877
BACKEND       = sha256:919cb75d464a8dc0e8192aa0ed3c61937ab67d980552a17493ec5414271ee5bd
FRONTEND      = sha256:9ddd6a2fb31b8a546443cd238db79adb5bcc3790f48974f6ece75a280747c233
DEPLOY_CONFIG_COMMIT_STAGING = 6fbfed5e3533b53bc9bc8be2118983a874749fee
CONFIG_CI_RUN = 36109828775
```

依 `PRODUCT_TO_STAGING.md` 的 config/artifact 分離規則，不得因 `6fbfed5` 的 config-only CI 又產生新映像，就把 compose 改釘那些新 digest；否則形成 digest recursion。

### 3.2 Phase 3R／4R

- Phase 4R policy blocker 已修：`deploy/README-VPS.md` 的 active stale ref 已改為 Production canonical `zfpkxsulehkbyuqkkcml`，沒有改 auth.js 或 broad exclusion。
- Local DEV 已證明 jlsq／JWT fail-closed／無 token 401。
- Local Docker ReleaseCandidate 已用 PowerShell 7 跑到：

```text
NGINX HEADER POLICY: PASS
VERIFY_DOCKER_IS_LATEST = PASS
```

技術 verifier 可記 PASS；但依最新 `PRODUCT_TO_STAGING.md`，真人登入與核心流程仍是 PARTIAL／待 Staging fixture，不得把 local health 當真人 E2E。

### 3.3 中央 deploy repo

已存在的 SIGN 相關 commit：

```text
f0c262603509816c1f7925176f58a3817257d0c9  platform: sync critical-smoke.sh with VPS (W5-A Option A)
71acc90588814e7e994da81df78ab07ff64e368d  platform: add sign-core critical smoke profile
2273b3623c47b404bcf1f38afe8ee0bb68b976d5  sign: register isolated Staging runtime signruntimestaging
```

目前本機 deploy HEAD 曾觀察為 `5d292b2`，`origin/main` 為 `2273b36`，且有另一執行者新增 system docs commit。這些檔案屬使用者／其他執行者；SIGN 不 push、不重排、不改寫。每次讀 deploy repo 先 `git status --short` 與 `git log -8`。

SIGN 已完成的中央功能：

- `services.yaml`：`signruntimestaging`，Docker，port 8902，domain `sign-staging.ticenpi.com`，`authCookie: none`，`criticalSmokeProfile: sign-core`；
- operator mapping：sign staging／production；
- release evidence tool 支援 product sign；
- 平台測試曾為 `114 passed, 28 skipped`（需使用正確 PowerShell module path）。

上述共享檔案若與即時狀態不同，不自行修復；分類 KEEP／ADJUST／CONFLICT 後回報。

### 3.4 HUB secret contract

```text
local commit = 33d5da7304aeed56c1943c8cbc58c6833b413d9a
origin/main  = 6e4cb5c3223ecda824f17d56565b847cf15b6662（先前觀察值，需重驗）
```

- `load-ticenpi-secrets.ps1` 的 `sign-staging` contract 已 value-blind PASS；
- `sync-secrets.ps1` 是既有未追蹤檔，依 v2 規則不得 commit；備份 `sync-secrets.ps1.bak-20260925-sign` 保留；
- HUB 有大量他人 WIP，禁止 reset、stash、clean、整檔 stage 或 broad commit；
- 只印 PRESENT／MISSING 與 project ref，永不印值；
- 未取得使用者明確要求前，不 push HUB。

### 3.5 Evidence

```text
F:\00-Ticenpi-SaaS\.release-evidence\sign\
F:\00-Ticenpi-SaaS\.release-evidence\sign\artifacts\ci-5396835da942c0486c832c56ba86c5f8f0af582d\
F:\00-Ticenpi-SaaS\.release-evidence\sign\artifacts\ci-6fbfed5e3533b53bc9bc8be2118983a874749fee\
```

`ci-539…` 是 frozen deploy artifact 證據；`ci-6fb…` 只證明 config commit CI 全綠，不可偷換 artifact identity。

## 4. 接續執行順序

### Phase R0：恢復與漂移盤點（唯讀）

1. 驗 RC local/origin HEAD、clean status、CI run 與 compose pin。
2. 驗 evidence JSON 的 source SHA、CI run、兩個 digest、platform `linux/arm64`。
3. 驗 deploy repo status/log；不得改 shared smoke files。
4. 驗 HUB 只看狀態與 sign-staging contract；不得印值。
5. 唯讀 SSH 驗 VPS：`/opt/ticenpi/sign-runtime-staging`、8902、現行 deploy script SHA／profile presence。
6. 輸出 KEEP／ADJUST／CONFLICT。若 SIGN 自身 frozen identity drift → HARD_STOP；只有 `591-core` 尚待 owner 時 → `WAITING_591`，不是 SIGN 自行修復授權。

### Phase R1：完成可平行的 SIGN-only 準備

在不碰 VPS／Cloudflare／Supabase 寫入的前提下：

1. `docker compose -f deploy/runtime-staging/compose.yml config --quiet`，機密只以 process placeholder／中央 loader 注入，不寫 `.env`。
2. `python server/manifest_tool.py check --manifest services.yaml --service signruntimestaging`。
3. 平台 operator/evidence tests 全綠。
4. `load-ticenpi-secrets.ps1 -Service sign-staging -CheckSecretsOnly` value-blind PASS。
5. `sync-secrets.ps1 -Service sign-staging` 不帶 `-Confirm`，只做 DryRun；確認 remote path 與 required key 名稱，不印值。
6. 準備 U3 DNS、U4 Supabase Site／Redirect URLs、U5 身分登入所需的人工作業清單，但不可代替使用者寫後台。

### Phase R2：等待共享平台依賴

當 591 執行者回報完成後，SIGN 只讀驗 §2.2 五項。若任一項失敗：

```text
BLOCKER = PLATFORM_SMOKE_COMBINED_VERSION_NOT_READY
ROOT_CAUSE = owner 交付版本未同時保留既有 profile 與 sign-core
EVIDENCE = commit / file / line / bash-n output
SMALLEST_NEXT_ACTION = 回 591／platform owner 修正；SIGN 不動共享檔
```

驗證 PASS 後才準備新的 U2-b。不要沿用 2026-09-25 的舊安裝指令或舊 SHA256。

### Phase R3：使用者節點與 Staging 部署

所有 VPS／Cloudflare／Supabase 後台寫入由使用者執行；每次先給完整備份、執行、驗證、預期輸出，標 WAITING_USER 後停止。

依序：

1. U2-b：安裝 owner 合併後的共用部署腳本；Luna 唯讀驗 SHA、`bash -n`、兩個 profile presence。
2. U2-a：同步 sign-staging env；Luna 唯讀驗檔案存在、mode/owner、requiredEnv PRESENT、URL 含 jlsq 且不含 zfpk/pygk，不讀值。
3. U3：DNS CNAME；U4：Supabase Staging Site URL／Redirect URLs；可合併請使用者處理。
4. `deploy.ps1 sign staging -DryRun` 與 `-Preflight` 可由 Luna唯讀執行；真正 deploy/rollback rehearsal 由使用者執行（U2-c）。
5. 部署成功必須證明：source/artifact/config/runtime 四段 identity、兩個 running digest、`/api/health.identity`、runtime-config 只含 jlsq、`sign-core` PASS、audit PASS。
6. cloudflared ingress／restart 為 U2-d，由使用者執行；Luna唯讀驗公開可達性。

任何 deployment 在 `PLATFORM_SMOKE_DEPENDENCY=WAITING_591` 時都禁止開始。

### Phase R4：Staging fixture 與 E2E

最新 identity contract 顯示 sign entitlement 尚不存在。原 v2 已核准的範圍是：同一個固定 business Customer 加 `sign` entitlement、`seat_limit=2`、只指派 assigned 成員。

規則：

- 先唯讀快照；
- 只用 canonical platform_admin RPC／本機 Launcher Workbench；Workbench 若尚未支援 sign Seat，就由使用者使用 canonical RPC；
- 不直接改表、不用 service-role 直寫、不用 SQL Editor；
- 不改 Post／DM／OCR／591 entitlement 或 seat；
- 寫入後再唯讀快照；
- assigned 應 ALLOW、unassigned 應 403、壞 JWT 401、無 token 401；
- 真人建案→上傳→框選→發送→無痕簽署→下載與資料清理全部要有證據；
- 未完成真人 E2E 不得 `record-accepted`。

完成後才執行：

```text
release.ps1 sign record-deployed ...
release.ps1 sign record-accepted ... -E2eVerified
release.ps1 sign status
```

必須出現 `.release-evidence\sign\accepted-staging.json`，且 identity／health／auth／commercial／e2e 全 true，才能宣告：

```text
STAGING_RELEASE_ACCEPTED = YES
```

### Phase R5：Production 方案 A

只在 Staging ACCEPTED 後開始。Production 改為同一組 accepted Docker digest、新 releaseRoot `/opt/ticenpi/sign-runtime`、新 port 8903 的藍綠切換，是使用者已選定的方向；但每一道 G1–G5 仍需使用者在對話中當次明確授權。

- 未到 Gate：Production 只允許唯讀 HTTP／SSH。
- G1–G5 不得合併授權、不得用計畫核准代替 Gate 授權。
- Production Supabase／SQL／seed／entitlement／env／DNS／Cloudflare／deploy 都由使用者執行。
- 不得原地改 `/opt/ticenpi/sign`、systemd current 或現行 host nginx。
- Production compose 只能釘 accepted Staging digest，不重建映像。
- P0 pygk bundle、P1 samples 依 v2 原樣列出，在對應 Gate 前不處理。

## 5. Git 與共享工作樹規則

- 不 rebase、不 force-push、不改寫已推 commit；不 reset/clean/stash。
- 只 `git add <明確路徑>`；不得 `git add -A` 或 `git add .`。
- 主工作樹 `F:\00-Ticenpi-SaaS\TicenpiSign` 唯讀（v2 最終 history 一行除外）。
- `backend/storage`、`samples/` 不得寫入、搬移、掛載或同步。
- deploy/HUB 有他人改動時完整保留；SIGN 不接管其他產品檔案。
- commit 訊息照 v2 計畫並加實際模型署名 `[GPT-5.6 Luna]`。
- 若計畫要求修改未列檔案或改變 frozen decision，停下回報，不自行設計。

## 6. 每一步輸出

```text
Phase Rx.y = PASS | PARTIAL | FAIL | HARD_STOP | WAITING_USER | WAITING_AUTH | WAITING_591
EVIDENCE = 指令摘要、完整路徑、commit SHA、CI run id、digest
```

沒有證據不得 PASS。health 200、mock、測試帳號截圖、profile 名稱存在，都不能替代真正 smoke／E2E。

HARD_STOP 時：

```text
BLOCKER =
ROOT_CAUSE =
EVIDENCE =
SMALLEST_NEXT_ACTION =
```

## 7. 最終／停點回報

每次停下列出：

- SIGN RC HEAD、artifact source commit、deploy config commit；
- CI run 與 backend/frontend ARM64 digest；
- deploy/HUB 的 SIGN commit 清單與是否已推；
- `.release-evidence\sign\` 路徑；
- U1–U11、G1–G5 完成／待處理；
- `PLATFORM_SMOKE_DEPENDENCY` 狀態；
- Production mutation 是否為 NONE；
- P0 pygk bundle、P1 samples 原樣狀態。

最後一行只能是：

```text
NEXT = WAITING_591 | WAITING_USER: Ux | WAITING_AUTH: Gx | HARD_STOP | STAGING_RELEASE_ACCEPTED = YES | GO_LIVE = YES
```

## 8. 現在開始

先做 Phase R0 的唯讀恢復盤點，輸出 KEEP／ADJUST／CONFLICT。不要修改 `critical-smoke.sh`、`deploy.sh` 或任何 591 檔案。若 SIGN-only 基線一致，直接做 Phase R1；若最後只剩平台 smoke 依賴，標 `WAITING_591`，不要把它改成 SIGN 自行修復任務。
