# Ticenpi591：完成中央商用權限、Docker 升級與使用者可執行的 Production 切換

指定執行模型：LUNA（Codex 模型選單 `gpt-6-luna`），thinking level：max。
使用繁體中文，回報簡短；你是執行者，請實際完成工程、驗證與交付，不只提出計畫。

## 零、執行規則：來源驗證、預設唯讀與資料集中

以下規則適用所有階段；降低錯誤靠實作與測試，不宣稱提示詞能保證模型 100% 正確。

### 0.1 所有新產生的 591 資料集中在唯一專案資料夾

- 本機專案根目錄固定為 `F:\00-Ticenpi-SaaS\Ticenpi591`。不要在 `F:\00-Ticenpi-SaaS` 父目錄或其共用 artifacts／.worktrees／其他位置新建 591 專用資料夾，也不要新建另一個 ticenpi／591 repo 副本。
- 提示詞、發布 payload／驗證紀錄、log、截圖、測試收據、臨時檔與機器可讀 checklist，統一放 `Ticenpi591\_charter\runtime\production-docker\` 下，依用途與時間分層；這是專案內既有 ignored runtime 路徑。
- 新的 release evidence root 固定為 `F:\00-Ticenpi-SaaS\Ticenpi591\_charter\runtime\production-docker\release-evidence`。在本次 PowerShell 程序及子程序設 `TICENPI_RELEASE_EVIDENCE_ROOT`，讓 release.ps1、promote.ps1 讀寫同一個 root；工具會在其下建立 `591\accepted-staging.json`、history、current-production.json 等。不得永久修改使用者／機器環境變數或 PowerShell profile。
- 先前共用的 `F:\00-Ticenpi-SaaS\.release-evidence\591` 只作唯讀歷史證據來源，不搬移、不刪除、不再寫入；不把舊 accepted 複製成新驗收。本輪有效紀錄由中央 release 工具產生到上述專案內 root。
- 使用者 Execute／Rollback 入口也必須設定相同 evidence root。切換到任意 cwd 仍可找回相同證據，不能落回共用預設目錄。
- 中央 deploy.ps1 目前在子程序 TEMP 下打包。以子程序專用 TEMP／TMP 或工具支援的顯式 tempDir，將本次打包／checksum／metadata 放到專案 `_charter\runtime\production-docker\tmp\`；核對所有實際輸出路徑，不永久改系統 TEMP。清理前驗證絕對 target 位於該任務路徑內，只移除本次建立的項目。
- 臨時 detached worktree 放專案內 `_charter\runtime\production-docker\worktrees\`；先驗證 path 受專案根限制、被 Git ignore、無已有 worktree／檔案衝突，並確認打包不含其他 worktree、log 或 runtime 資料。不能滿足就停止此步驟，不改放父目錄。
- 正式腳本放專案 `scripts\`，正式手冊放專案 `docs\` 或 `deploy\runtime-production\`，延用既有結構。不要將測試收據與發布 payload 納入程式碼 commit。
- 中央 deploy repo 的既有 services.yaml、共用腳本、必要回歸測試與系統文件仍在原位做最小修改；這不是允許在 deploy repo 新增另一套 591 專用 artifacts／handoff 目錄。
- 這項本機整理規則不要求搬移遠端 VPS 的 `/opt/ticenpi/...` 路徑，也不允許更動舊 systemd rollback 資料。不要為整理本機資料改壞伺服器布局。

### 0.2 SHA／digest 必須有真實來源，驗證後固定

- 本提示詞列的 SHA、digest、release id 是交接快照，不是目前狀態保證；先讀 CI 產物、Git 物件、OCI labels／RepoDigests 與 release evidence 交叉核實。
- 建立有證據來源的部署選定紀錄，分欄保存 `artifact_source_commit`、`deploy_config_commit`、`ci_run_id`、`image_digest`、`accepted_staging_release_id`、`systemd_rollback_release_id`；不得相互代填。可變更欄位須重新通過其 gate。
- `git rev-parse HEAD` 只描述該 checkout 的 HEAD，不足以認定映像 artifact source。artifact source 應取 CI 產物與 OCI revision 的一致值；Production 使用 accepted-staging.json 中已驗證的 source_commit。
- compose 必須保留完整 `@sha256:...` 固定值，Production TICENPI_COMMIT_SHA 必須保留已驗證 source 的完整 literal；證據與文件也必須保存當時實際值。不採用「所有 SHA／digest／release id 都不能寫死」的建議。
- 操作腳本從已核實的部署選定紀錄／accepted evidence 讀取固定目標，再與真實 CI、compose、runtime 比對。不得每次自動跟隨最新 main／latest／現行容器來替換選定目標；漂移時拒絕執行，不能讓目標跟著錯誤 runtime 改變。
- git source／CI digest／OCI digest 要辨識 manifest index、platform manifest 與 image ID 的差異，避免拿 image ID 直接當 registry digest；全長值及來源存證，不靠模型補齊縮寫。

### 0.3 PowerShell 與 Bash 分開寫、分開驗證

- 本機用 Windows PowerShell，遠端用 Linux Bash；記錄實際 PowerShell 版本，交付命令與腳本必須可在使用者的指定 shell 執行。
- 不把 `ssh ... << 'EOF'` 這種 Bash heredoc 直接寫進 .ps1。遠端腳本使用獨立 LF .sh 檔或 PowerShell 單引號 here-string，再以明確 UTF-8／LF、標準輸入傳給 `ssh ... 'bash -s --'`。
- 優先以 ProcessStartInfo／明確參數傳遞處理 stdin 與 exit code；不能拼接任意輸入成 shell command。不要用 JSON.stringify 充當 shell quoting。
- Bash 程式內不得混入 `$env:`、PowerShell cmdlet 或 Windows 路徑。單引號 here-string 中的 `$VAR`、`$(...)` 留給遠端 Bash 解譯，不讓本機 PowerShell 先展開；動態參數用嚴格 allowlist 與正確 quoting。
- 測試包含 PowerShell parse、bash -n，以及含 `$`、引號、空白路徑和非 ASCII 文本的傳輸測試；不在 output／argv 洩漏機密。

### 0.4 預設唯讀；腳本包含寫入，AI 不執行寫入

- 未給模式時預設 Check／DryRun；Check／DryRun、Execute、Rollback 互斥，明確拒絕衝突參數。不要以傳一個旗標就跳過全部 gate。
- Execute／Rollback 可以且必須包含使用者已要求的 Production 切換／恢復動作。禁止的是 AI 呼叫這些寫入動作，不是禁止生成可用的操作腳本。
- 若使用 SupportsShouldProcess／-WhatIf，所有遠端副作用也須在 ShouldProcess gate 內，不能只加 attribute 就稱安全。採用等價的顯式預設唯讀模式也可，但必須有零遠端副作用測試。
- DryRun 不為模擬而遠端建 temp／lock／backup、scp／pull／start／stop；只讀查驗允許，本機 log 存專案內。任何 user Execute 都先跑實際 gates，不能把 DryRun 的結果永久當通行證。
- 不要求展示或規定模型私有思考。改以可審查的 checklist 文件，逐關記狀態、證據路徑、指令 exit code 與驗證日期；未知填 NOT_VERIFIED，不能先打勾再找證據。

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

所有 release／promote 入口先設定第 0.1 節 evidence root，再重新讀該 root 下的 accepted-staging.json／history／release status，確認 status、e2e_verified、全部旗標、digest、commit、release 相互一致。用不同 cwd 啟動做一次解析一致性測試。只有全部必要路徑實測後才可宣告 STAGING ACCEPTED。

### D. S9–S10：交付一行 Production 指令

建立完整本機操作腳本（建議 `scripts/Invoke-591ProductionCutover.ps1`）與 `deploy/runtime-production/CUTOVER.md`；路徑可依 repo 現有 convention 調整，但最後指令不得有 placeholder。

腳本至少提供互斥的唯讀 Check/DryRun、Execute、Rollback 模式，未指定模式時預設唯讀。AI 只可跑唯讀模式及隔離測試；不能透過另一程序／排程／背景工作觸發 Execute 或 Rollback 寫 Production。生成的腳本應包含真正的切換／回滾程式碼，透過顯式模式與完整 gates 控制；不要把使用者可執行腳本誤寫成只有 echo 的假腳本。

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
- 測試、runtime 證據與發布驗證全部依第 0.1 節存到唯一 Ticenpi591 專案內；正式腳本與文件延用專案結構，不在父目錄／共用 artifacts 產生 591 副本。不把密鑰、完整 token、個資或廣告資料提交。
- 重要進度先短訊息告知，避免長時間無更新；正常已授權修正自行完成，不重複問是否繼續。

## 八、完成條件與最後回報

只有以下都完成才能回報 READY_FOR_USER_PRODUCTION：
1. 全部相關測試通過；required CI 綠；main 已推送且 WIP 保全。
2. Staging 新的必要路徑實測、完整證據及 accepted-staging 真實有效。
3. Production compose/manifest/identity 與 accepted digest/source 完全匹配。
4. 一鍵 Execute/Rollback 腳本已建立、隔離測試通過、真 VPS 唯讀 Check 和 promote DryRun PASS，必要前置已存在或安全納入使用者腳本。
5. 原 systemd 回滾目標與資料保護已核實。
6. AI Production mutation＝NO；Supabase／Cloudflare／DNS mutation＝NO。

最後先完成專案內 `delivery-checklist.json`（格式可依既有 convention）：每項包含 PASS／FAIL／NOT_VERIFIED、evidence path、checked_at；不可只輸出無證據勾選符號。至少核對以下項目：

- [ ] Artifact source／config commit／CI／digest／release 已分欄且交叉核實。
- [ ] Fixed digest 與 Production literal source 正確，沒有跟隨 latest 或任意 HEAD。
- [ ] 本機 PowerShell／遠端 Bash 語法、引號與編碼測試通過。
- [ ] 未指定模式與 DryRun 沒有任何遠端副作用。
- [ ] Execute／Rollback 有真實實作，AI 未執行 Production 寫入。
- [ ] 原 systemd snapshot／Fernet key／未知備份資料完整保護。
- [ ] 新的 591 交付資料與 release evidence 全在唯一專案根內，入口共用同一 evidence root。
- [ ] 未修改 Supabase schema／資料／設定、Cloudflare、DNS；測試登入界線有紀錄。
- [ ] Staging ACCEPTED 的全部必要證據真實存在；缺項未偽填旗標。
- [ ] Source CI、promotion DryRun 與 Production 唯讀 Check 都有實際結果。

若必要項目未 PASS，狀態不能是 READY_FOR_USER_PRODUCTION；精確列缺項，不宣稱 100% 保證。

<output_format>
最後用繁體中文，只輸出以下欄位；不加前言後語。方括號是格式說明，交付時換成真實值，未完成填 `BLOCKED: 原因`，不得保留 placeholder。缺 gate 時「執行」欄明確寫不可執行；回滾尚未建立時也不能虛構指令。

狀態：[READY_FOR_USER_PRODUCTION | WAITING_USER | PARTIAL | HARD_STOP]
Staging：[實際狀態；release／source commit／deploy-config commit／CI run／digest；E2E 結果]
Production：[唯讀實查 systemd 或 docker；preflight 結果；AI 寫入＝NO]
資料：[唯一專案內的新 evidence root／handoff 路徑]
執行：[無 placeholder 的一行 PowerShell 指令，或 BLOCKED 原因]
預期：[成功標記、關鍵 identity／digest，不能稱已在 Production 實測]
回滾：[無 placeholder 的一行指令與舊版恢復結果，或 BLOCKED 原因]
證據：[可點的 handoff／delivery-checklist／測試／release 檔案]
仍待使用者：[Production 執行、S11 真人 canary，及其他真正人工作業]
</output_format>

不要在最後只貼計畫、只說「可以幫你」，或未實測就說「好了」。你要把授權範圍內的所有工作實際完成，停在使用者執行 Production 的界線。

<!-- END TICENPI591 PRODUCTION DOCKER LUNA HANDOFF -->
