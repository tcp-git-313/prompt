# Ticenpi591：完成中央商用權限、Docker 升級與使用者可執行的 Production 切換

指定執行模型：LUNA（Codex 模型選單 `gpt-6-luna`），thinking level：max。
使用繁體中文，回報簡短；你是執行者，請實際完成工程、驗證與交付，不只提出計畫。

## 一、任務與完成界線

接續已進行的 Ticenpi591 主線整併工作，依中央 DELIVERY_WORKFLOW S0–S12 補齊缺口，做到「使用者拿到一行指令，即可按已驗證的流程切換 Production」的交付狀態。

本次明確授權你：
- 修正以下列出的已知 compose identity 缺口、591 的程式／測試／CI／部署設定／操作腳本／文件。
- 在各 repo 原有主線直接 commit、push；591 與 deploy 都是 main。不開任何新分支。
- 部署及驗證隔離的 591 Staging，完成符合真實證據的 release 記錄。
- 對 Production 做必要的唯讀盤點，準備完整且可審查的一鍵切換與回滾腳本。

你不得直接執行任何 Production 寫入。Production 安裝部署腳本、建立網路／WARP relay／防火牆、備份、建立或搬移 env、pull 映像、建立容器、停止 systemd、改 symlink／manifest、切流量等操作，全部只能放入交付給使用者執行的腳本。

不改 Supabase schema／資料／Seat／entitlement／設定，不改 Cloudflare、DNS、OAuth 設定。需要這些操作時，只能列出精確的人工作業與依賴，不代做。既有 Staging 夾具登入沿用已核准的安全方式，不新建帳號、不寄信、不改帳密、不指派席位、不輸出 token。

「可以執行 Production 切換」不等於「Production 已部署／已 ACCEPTED／1088 已完成上架」。本次範圍是 591 的存取控制與交付能力；付款、定價、方案權益管理另有中央規格，不擅自新增方案資料或金流。

已知缺口屬於本次要修正的工作，不能再次僅因發現這些缺口而停止。真正與核准目標不同的來源漂移、未知 WIP、機密缺失、真人登入／MFA、外部權限或需要禁止範圍寫入時，停止受影響步驟並明確回報，不自行繞過。可獨立完成的程式、腳本、測試與文件應繼續完成。

## 二、先讀與權威順序

先讀下列檔案，再開始改動：
1. `F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md`（B8、B9、B21，並注意文件日期）。
2. `F:\00-Ticenpi-SaaS\deploy\docs\system\DELIVERY_WORKFLOW.md`（完整 S0–S12、單一主線規則）。
3. `F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md` 的 591 卡。
4. `F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_TO_STAGING.md`、`STAGING_TO_PRODUCTION.md`、`DEPLOYMENT_IDENTITY.md`。
5. `F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TEST_IDENTITY_CONTRACT.md`。
6. `F:\00-Ticenpi-SaaS\TicenpiDM\docs\ACCESS_DENIAL_SCREEN_CONTRACT.md`。
7. `F:\00-Ticenpi-SaaS\TicenpiPost\backend\app\core\seat.py`。
8. 591 現有 `_charter`／W6 執行紀錄與 repo 適用的 AGENTS.md。

以本提示詞當次授權為界線；最新實查優於過期文件。舊 W6 的真人 gate 不可直接當成已完成，但也不要重跑已被本次有效證據覆蓋的階段。

## 三、2026-10-02 交接快照（開工必須重新唯讀核實）

### Git 與已做工作
- 591 repo：`F:\00-Ticenpi-SaaS\Ticenpi591`，remote `tcp-git-313/ticenpi591`，主線 `main`。
- 交接時 main 與 origin/main 相同：`bd771c937700112a20ad3261c2c716667a3009db`。
- Production 程式 WIP 已有先前的選擇性提交；Staging RC 已經由 merge commit `0f87fdb` 併入 main。
- `codex/591-w6-rc-20260925` 已在本機與 GitHub 刪除，不要重建。
- 尚有其他歷史分支／worktree；不在本次授權刪除範圍。
- 交接時工作樹只有兩個追蹤檔修改：
  - `591/posted_history.json`：runtime 資料，保留，不擅自提交或還原。
  - `scripts/591_seat_e2e.py`：已將錯誤 `/api/autopost/access`、`/api/autopost/health` 改為 `/api/access`、`/api/health`，是本任務尚未提交的修正；先讀 diff 後接續。
- deploy repo：`F:\00-Ticenpi-SaaS\deploy`，remote `tcp-git-313/ticenpi-platform`，主線 main；交接讀到 HEAD `1f86cf7`，這是其他 Sign 工作的提交，不可覆寫。
- deploy 已存在他人的未追蹤文件，包含 `HANDOFF-20260922-cookie-isolation-and-ci-gate.md`、`HANDOFF-20260928-admin-workbench-t1-merge.md`、`docs/system/SPEC-C-PRODUCT-BACKEND-SEAT-STATUS.md`；保留，不納入你的 commit。

### 映像、CI 與 Staging
- 映像：`ghcr.io/tcp-git-313/ticenpi-591@sha256:c67fd2cae4dfbfdcb9a2e8182b6813554da041ce219d2d67dc21a6efadbce7bb`。
- 映像真正 source commit：`f9b3b6964fe4bf40cbdf97740af303e66e19e0fb`。
- 映像 source 的 CI run：`37000522652`；本輪 gh 查驗 overall success，backend、frontend、policy、image-scope、arm64-image 全 completed/success。
- VPS Docker image 的 OCI revision label 已讀回等於上述 f9b3b69；RepoDigests 已讀回等於上述完整 digest。
- Staging service：`591runtimestaging`；公開網域 `https://591-staging.ticenpi.com`；host `127.0.0.1:8891` → container 8890。
- Staging current：`/opt/ticenpi/591-runtime-staging/releases/20261002-193155`。
- 線上 `/api/health`、`/api/live`、`/runtime-config.js` 本輪均 HTTP 200，container healthy。
- health identity：environment staging、release `20261002-193155`、commit `bd771c937700`、project `jlsqjvehwblkeuycjoyj`、seat_required true、auth jwks-es256。
- 注意：health 的 bd771c9 是部署設定來源，映像 source 是 f9b3b69。不得混用這兩個 commit 作 artifact 證據；修正後要明確區分 source／deploy-config／runtime identity。
- 注意：Staging health 的 `dry_run_submit=false`，compose 也如此，但 services.yaml 的 notes 仍寫 dry-run only。這是需核對既有授權與文件的差異，不可偷偷改成正式送出或假裝是 dry-run。

### Seat E2E 與驗收
- 已存在 `F:\00-Ticenpi-SaaS\.release-evidence\591\seat-e2e-20261002.json`。
- 該紀錄時間 `2026-10-02T11:33:31+00:00`，runtime 為上述 Staging release：
  - assigned → `/api/access` 200，state allowed。
  - unassigned → 403，error SEAT_NOT_ASSIGNED，access.state no_seat。
  - invalid JWT → 401。
  - no token → 401。
- overall PASS；這是既有紀錄，重用前核對測試對象和產物是否相同。改了受測路徑後必須重測。
- 交接時 `accepted-staging.json` 不存在；現有 history 有舊 `20260927-040154.DEPLOYED.json`，不能冒用為新版本驗收。
- 前一段工作摘要說過登入／發送正常；只有能追溯的真實證據才可以引用，不把摘要、health 200 或 Seat API PASS 等同於 Google 回跳、瀏覽器核心功能、真人 E2E 全過。

### Production 唯讀快照
- VPS：`ubuntu@147.224.255.234`，先 live-check target。
- `ticenpi-591.service` ActiveState=active。
- WorkingDirectory=`/opt/ticenpi/591/current/591`。
- ExecStart=`/opt/ticenpi/591/current/591/.venv/bin/python -m uvicorn app:app --host 127.0.0.1 --port 8890`。
- Production current 指向 `/opt/ticenpi/591/releases/20260912-163944`。
- services.yaml 的 `"591"` 仍是 systemd-python，尚未轉 Docker。
- 中央 Production Supabase ref：`zfpkxsulehkbyuqkkcml`；Staging：`jlsqjvehwblkeuycjoyj`。

## 四、已知 S4 缺口：必須修好並往下做

591 的兩份檔案：
- `deploy/runtime-staging/compose.yml`
- `deploy/runtime-production/compose.yml`

目前都缺 `TICENPI_DATABASE_TARGET`。Production 的 `TICENPI_COMMIT_SHA` 還是 `${TICENPI_GIT_SHA:?...}`，沒有 literal accepted artifact source commit。Production compose 註解引用的 `deploy/runtime-production/CUTOVER.md` 也不存在。

修正要求：
1. 依當前 services.yaml `platformContract.policy` 推導 DATABASE_TARGET literal，不猜值、不寫 NOT_APPLICABLE。
2. identity block 明確包含環境、project ref、database target、release 與 commit；依目前中央規格處理 GIT_SHA alias。
3. Production `TICENPI_COMMIT_SHA` 固定為最後 accepted artifact 的完整 40hex source commit；若沿用本映像就是 f9b3b6964fe4bf40cbdf97740af303e66e19e0fb，不是 bd771c9。
4. 不把任何部署身分鍵塞入 shared env 或 requiredEnv。
5. 查清中央注入／驗證路徑，避免拿 deploy-config commit 冒充 artifact commit。
6. 只改 config／腳本／文件且映像 source fingerprint 未變時可以沿用 digest；若映像內容改變，重新從乾淨 source 建映像、等 exact-commit CI 全綠，再重走 Staging 驗收。Production 永遠不重建。

## 五、依順序執行

### A. 開工盤點與 S0–S3 證據

git fetch 兩個 repo，確認仍在 main，記錄 HEAD／origin/main／dirty diff／已部署版本。git fetch 不代表可以 pull 覆蓋共享 WIP。新出現的未知修改不納入你的提交。

先輸出簡短 CHECKS／GAPS；不要重做已完成 merge、刪 RC 分支或任意更換 Production 程式。

檢查後端所有受保護 HTTP API 與 WebSocket 皆經中央商用＋Seat 閘門：使用呼叫者自己的 JWT，`my_commercial_context()`＋`product_seat_status('591')`，fail closed。health/live/runtime-config 等必要公共端點依合約保留；列出明確 allowlist，不能把整個 /api 放行。

Seat 預設 require；skip 只允許明確本機，且 URL／project ref 任一指向 Production 時拒絕啟動；Staging／Production 不得 skip、shadow 或 dev bypass。中央 timeout、HTTP error、未知／缺欄位、過期快取都不得放行。檢查 WebSocket handshake 與後續長連線授權行為，不能只測 REST。

前端保持 `ticenpi.access_denial.v1`，相容頂層 access 與 detail.access；尚未開通與尚未指派席位的文案／重新確認／改帳號／客服按鈕與 Post／ORC、DM 契約一致。中央不可用只能顯示不可用與重試；401 回登入；不得顯示 user/customer/email／token 或上游原文。

執行完整 repo 要求的測試、前端 typecheck／tests／build、後端 contract gate，以及與這次修正相應的部署測試。參照 `.github/workflows/ci.yml`，檢查所有被 merge 進來的測試目錄，不把 skipped 算 PASS。不要以「CI 已綠」替代本次新增修改的驗證。

選擇性 git add、commit、push main；禁止 git add -A／reset／clean／restore-all。runtime JSON 保留。需要乾淨來源使用 detached worktree（不建立 branch），完成後僅移除自己的臨時 worktree。

exact source commit CI 全綠且產物 digest／OCI revision／source fingerprint 有證據；deploy-config commit 的 required CI 也要過，不把未觸發 image build 當成新產物。現有 image-scope 已改 before..sha／手動 dispatch 強制建置，先核對再使用，不退回 diff-tree HEAD 的 merge commit 漏建做法。

### B. S4：Production Docker manifest 與安全回滾設計

只修改 services.yaml 的 591 自己頁面，其他產品頁面保持原樣；不要為本任務更動共用 platformContract.policy。

591 改 docker-compose，釘與 accepted Staging 完全相同的單一映像；補齊 product/environment、composeFile、composeProject、sourcePath、remoteDir、releaseRoot、envFile、requiredEnv、preserveFiles、127.0.0.1:8890:8890、health、smoke、cookie 等契約欄位。

Docker Production 使用獨立 release root／compose project／network／output volume，不能覆蓋舊 `/opt/ticenpi/591` 的 systemd releases/shared/current。保留舊 unit、原始 env、Fernet key、未知 regions/backup 資料與 systemd release snapshot，禁止首次轉 Docker 就清掉舊版。

既有 candidate 使用 network `ticenpi-591-runtime-production`、192.168.60.0/24、gateway 192.168.60.1、WARP socks5 gateway:40000。這些是候選值，必須唯讀查 subnet／port／relay／防火牆衝突後才固定。保留 8890 loopback，因此既有 Cloudflare 路由應不需修改；實查支持此結論再寫文件。

查明中央 deploy.sh／rollback.sh／audit.sh 是否只為 `591runtimestaging` 特判 env 與 --no-build；如 Production 591 也需要，做最小、有測試的 service-scoped 支援。不得因此對 Production 使用 Staging key／env，也不影響其他產品。中央腳本只在 main commit/push；安裝到 Production VPS 由使用者的一鍵腳本執行，AI 不安裝。

第一次跨 systemd→Docker 的失敗回復與以後 Docker→Docker 回滾分開實作：第一次失敗必須停止候選 Docker、確認釋放 8890、按原 snapshot 恢復舊 manifest／pointer，啟動原 systemd 並驗證健康。不得宣稱中央通用 rollback 能自動跨到沒有 predecessor 的新 Docker release root。成功後持續保留 systemd fallback，不刪舊資料。

跑 manifest_tool check、manifest validator、test_manifest_platform_policy，以及受影響的中央 deployment/identity/rollback 測試。驗證 diff 限定 591；本機 schema PASS 不等於 VPS 已更新。

### C. S5–S7：完成 Staging 驗收

依新 config 先 DryRun、Preflight，再部署 Staging；來源使用乾淨 detached checkout，runtime dirty 只留原 main。任一失敗按中央流程停止／回滾，exit 3 需解決 decision_required 才可 accepted。

重新檢查 running digest、image revision、release、source/deploy-config、environment／project/database identity、port、runtime-config、health／live／ready、public URL 與 canonical audit／591-core。

Seat 夾具：沿用契約指定的 assigned 與 unassigned 一般使用者（admin／test_allow 不能代替）：
- assigned 允許。
- unassigned 403 並呈現 no_seat。
- 壞 token 401。
- 無 token 401。
- 實測受保護業務 API 及 WebSocket，不只 /api/access；正向測試避免新增真實廣告／外部發送，負向不得觸發業務副作用。

固定安全使用 Staging URL／project exact allowlist，機密只進子程序記憶體；中央 loader `F:\HUB\scripts\load-ticenpi-secrets.ps1` 的 broad import 要收斂到所需 Staging key，測完清除，不寫入證據／命令列／repo。不以 service-role 直接呼叫產品受保護 API 充當使用者。既有 `Use-591StagingSecrets.ps1` 會清除 STAGING_*，執行 E2E 前要讀懂它與腳本的變數契約，不能誤用。

取得 DELIVERY_WORKFLOW S7 要求的真實登入回跳、核心功能、授權矩陣、拒絕畫面、cookie／header 等證據。可追溯且同產物的既有真實測試可以引用；缺少必須如實標 NOT_VERIFIED。需要人類 MFA／未提供帳號等，指出唯一必要的人工作業，完成其餘交付；不得假造 `-E2eVerified`。

證據齊全後，依中央 release.ps1 先 record-deployed 再 record-accepted；single image 的 backend/frontend digest 按工具契約填同一 digest。寫入 source_commit＝artifact source、deploy_config_commit＝部署設定 commit、CiRunId＝source 的真實 CI run、release_id＝實際 Staging release，不能抄舊值。

重新讀 accepted-staging.json／history／release status，確認 status、e2e_verified、全部旗標、digest、commit、release 相互一致。只有全部必要路徑實測後才可宣告 STAGING ACCEPTED。

### D. S9–S10：交付一行 Production 指令

建立完整本機操作腳本（建議 `scripts/Invoke-591ProductionCutover.ps1`）與 `deploy/runtime-production/CUTOVER.md`；路徑可依 repo 現有 convention 調整，但最後指令不得有 placeholder。

腳本至少提供互斥的唯讀 Check/DryRun、Execute、Rollback 模式。AI 只可跑唯讀模式及隔離測試；不能透過另一程序／排程／背景工作觸發 Execute 或 Rollback 寫 Production。

使用者 Execute 的單行指令須內建順序，不要求使用者自己拼多條 SSH 指令：
1. 固定 target／service／accepted release／digest／source commit，重新核對主線與乾淨打包 source，拒絕漂移。
2. accepted-staging 與 promote gate 全 PASS，無 build、無 mutable tag；production literal commit＝accepted source。
3. 再驗 Production 現況與舊 systemd rollback target，執行互斥鎖避免重複切換；備份原 manifest／unit 狀態／release pointer 等，保護 env 與原 Fernet key，不能重生 key 導致既有 591 帳密失效。
4. 使用已存在正確 Production 機密，僅按所需欄位安全準備 Docker env；禁止把部署 identity 填 env、禁止輸出值。缺 anon key、relay、registry auth 等必須在停止舊 systemd 前失敗。無法安全自動取得的前置不能偽稱一鍵已就緒。
5. 建立專屬 network／relay／精確防火牆規則並驗證，不重啟或修改其他產品／現有 WARP daemon。隔離、冪等、執行失敗可回收自己建立的項目，不刪別人共用資源。
6. 準備候選 package／digest pull／中央腳本／manifest；需安裝伺服器腳本時比對現況與 published main blob，保留備份。candidate manifest 採中央 deploy 流程，不直接以改 active manifest 繞過 preflight。
7. 完成所有可提前驗證項目才短暫停止舊 systemd，由 `promote.ps1 591 production`／中央 deploy 鏈啟動 Docker；SourceOverride 指向乾淨來源，不能拿 dirty main 打包。不得以裸 docker compose 替代中央 release gate。
8. 驗證 running image digest、container healthy、loopback binding、internal/public health、production project/database identity、runtime commit＝accepted source、critical smoke，以及無 token／壞 token 401；不做未授權真實廣告发布。
9. 成功只記 Production DEPLOYED；若任何切換後 gate 失敗，自動進 systemd fallback，確認舊服務 running/listening、health／public URL、舊 pointer／manifest 正確，輸出回滾成功或失敗。不能把單純命令 exit 0 當回滾成功。
10. 真正 Production Seat／commercial canary、record-accepted 留給使用者依 S11 執行；不自動填 E2eVerified，不使用 Staging 夾具作 Production 真人證據。

另交付一行手動 Rollback 指令，能使用已記下的具體舊 target，在失敗自動回滾之外作人工回退。成功切換後若使用者之後回退，腳本也須處理中央 evidence／manifest／pointer 一致性，不留下同時搶 8890 的 systemd 與 Docker。

## 六、如何驗證一鍵腳本（不能假稱 Production 已測）

本機 PowerShell parse／參數測試、shell bash -n、mock SSH 的完整順序測試，以及隔離環境的失敗注入：至少涵蓋缺 env／錯 target／錯 digest／無 accepted／port 被未知服務占用／promote 失敗／Docker 不健康／fallback 失敗／重複執行。

證明 DryRun／Check 不會寫遠端、不會 pull/start/stop/改 env/manifest，不會觸碰其他產品。用隔離的測試 target 或 mock，不使用 Production 演練切換。

在真 Production 僅唯讀 preflight：讀 unit/current/监听、所需文件存在性與 key 有無、受控 fingerprint、network／relay／rule 現況；不 cat 全 env、不 docker inspect 全 environment、不洩漏 auth header。

跑 `promote.ps1 591 production -DryRun` 並保留實際 PASS 證據；這只證明 promotion gate，不等於遠端所有前置完成。你的 Check 必須補足源端與 VPS 唯讀驗證。

若未完成 Staging ACCEPTED 或缺必需人工作業，Execute 必須 fail closed；交付標 WAITING_USER 或 PARTIAL，說明精確缺口，不能宣稱 READY_FOR_USER_PRODUCTION。不要將未知項目塞進一鍵腳本後稱「到時候會知道」。

## 七、提交、文件與可回溯交付

- 只選擇性提交自己的檔案到兩個 repo main，推送後核對 remote HEAD 與 exact-commit CI；共享 workspace 的他人新增 commit／WIP 先識別，不覆寫。
- 依中央範本更新 591 integration card、CURRENT_STATE 的 591 狀態與 B8/B9/B21 的產品範圍；保留 Sign／其他產品現況與歷史。
- B8/B9 在 source 修好但 Production 未執行前，寫「591 source ready／Production still systemd」，不能標整個平台已解決。
- 產生 `docs/PRODUCTION_DOCKER_HANDOFF.md`（或符合現有文件結構的同等檔案），列 S0–S12 已完成／待使用者狀態、來源與設定 commit、CI run、digest、Staging release、E2E 證據、使用者 Execute/Rollback 指令、預期輸出、失敗判讀、需人做的 S11 canary。
- 測試與 runtime 證據存入明確的 artifacts／.release-evidence 路徑，不把密鑰、完整 token、個資或廣告資料提交。
- 重要進度先短訊息告知，避免長時間無更新；正常已授權修正自行完成，不重複問是否繼續。

## 八、完成條件與最後回報

只有以下都完成才能回報 READY_FOR_USER_PRODUCTION：
1. 全部相關測試通過；required CI 綠；main 已推送且 WIP 保全。
2. Staging 新的必要路徑實測、完整證據及 accepted-staging 真實有效。
3. Production compose/manifest/identity 與 accepted digest/source 完全匹配。
4. 一鍵 Execute/Rollback 腳本已建立、隔離測試通過、真 VPS 唯讀 Check 和 promote DryRun PASS，必要前置已存在或安全納入使用者腳本。
5. 原 systemd 回滾目標與資料保護已核實。
6. AI Production mutation＝NO；Supabase／Cloudflare／DNS mutation＝NO。

最後用繁體中文短報告：

狀態：READY_FOR_USER_PRODUCTION | WAITING_USER | PARTIAL | HARD_STOP
Staging：release／source／deploy-config／digest／ACCEPTED 證據
Production：仍是 systemd，AI 未寫入；唯讀 preflight 結果
執行：一行可直接貼上的真實 PowerShell 指令
預期：切換成功時實際會看到的成功標記與關鍵 identity／digest
回滾：一行可直接貼上的真實指令與預期舊版恢復結果
證據：可點的 handoff／測試／release 檔案
仍待使用者：Production 執行及 S11 真人 canary（若另有阻塞，逐項明列）

不要在最後只貼計畫、只說「可以幫你」，或未實測就說「好了」。你要把授權範圍內的所有工作實際完成，停在使用者執行 Production 的界線。

<!-- END TICENPI591 PRODUCTION DOCKER LUNA HANDOFF -->
