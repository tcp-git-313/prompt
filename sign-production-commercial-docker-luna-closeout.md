# TicenpiSign：LUNA 接續中央商用權限＋Docker，做到使用者可執行 Production

指定執行模型：GPT-6 Luna；建議 thinking level：xhigh。
工作語言：繁體中文。你是執行者，從中斷點接續，不重做已完成的工作。

## 1. 最終目標與授權

完成 TicenpiSign 的 main 整理、Production 切換／回滾腳本修正與測試、Staging 固定 Seat 夾具真人 E2E、正式 record-accepted，最後交付「使用者在 Windows PowerShell 7 貼一行即可依序部署並切換 Production」的可執行腳本與預期結果。

本任務已授權你：

- 在 Sign main 做必要程式／測試／部署設定／文件修改、選擇性 commit、push、讀 CI、部署與驗收 Sign Staging。
- 修正本提示詞已列出的切換／回滾缺陷；不要再把這兩項已知問題當成新發現而只停在報告。
- 備份、逐檔比對、可逆封存 WIP，保留獨有成果；不自行丟棄使用者尚未確認的過時清單。
- 唯讀查看 Production 健康、容器、systemd、nginx、port、機密存在性與公開 project ref。
- 只有必要時才修改中央 deploy repo 的 Sign 服務頁或與 Sign 閘門直接相關的最小程式，須附影響範圍及回歸測試；不可改其他產品設定。

未授權你：

- 執行任何 Production 寫入，包括新 env 建立、promote、啟動／停止 Production 容器、nginx 切換、systemd stop/disable、回滾、Production 證據寫入。這些只能由使用者執行你交付的腳本。
- 寫入 Supabase（含 Staging），或修改 Cloudflare、DNS、Google OAuth。若必要，準備精確最小步驟交使用者操作，完成後唯讀重驗。
- 新建分支、新建其他 chat、委派子代理、修改 591 或其他產品。
- git reset --hard、git clean、restore-all、全域 stash、git add -A、force push、刪除資料庫／R2／Docker volume、變更或輸出密鑰。

遇到已知、屬本任務範圍內的問題，修正並驗證，持續做到目標。遇到新的來源漂移、無法識別的 WIP、schema／RPC 不相容、權限或真人登入邊界，停止該依賴路徑並短報具體證據，繼續不依賴它的工作；不得自行變通繞過閘門。

不得把 CI 綠燈、單元測試、health 200、fixture snapshot、模擬演練當成 Staging ACCEPTED 或 Production 已完成。

## 2. 必讀資料，依此順序

1. `F:\00-Ticenpi-SaaS\artifacts\sign-resume-audit-20261002\RESUME.md`
2. 同目錄 `main-wip-status.txt`
3. `F:\00-Ticenpi-SaaS\deploy\docs\system\TICENPI_SYSTEM_CURRENT_STATE.md`（B7、B8、B9、B21）
4. `F:\00-Ticenpi-SaaS\deploy\docs\system\DELIVERY_WORKFLOW.md`（S0–S12、單一主線規則）
5. `F:\00-Ticenpi-SaaS\deploy\docs\system\PRODUCT_INTEGRATION_CONTRACT.md` 的 Sign 卡
6. 同資料夾 `STAGING_TO_PRODUCTION.md`、`STAGING_TEST_IDENTITY_CONTRACT.md`、`DEPLOYMENT_IDENTITY.md`
7. 新版 Sign 交接：目前在 `F:\00-Ticenpi-SaaS\.worktrees\sign-main-next\docs\handoffs\SIGN-PROJECT-HANDOFF-2026-09-28.md` 的 10-02 節；主資料夾尚未同步，不能只讀主資料夾的舊版本。
8. `F:\00-Ticenpi-SaaS\TicenpiDM\docs\ACCESS_DENIAL_SCREEN_CONTRACT.md`、`F:\00-Ticenpi-SaaS\TicenpiPost\backend\app\core\seat.py`
9. Sign 的 auth、runtime identity、前端 access_denial、compose、CI、sign-ops、sign-production-cutover，以及中央 promote/release/deploy/首次納管流程。

有日期的本輪實查優先於舊文件。中央文件目前仍包含過時 Sign 狀態；不得因舊卡寫 systemd 就撤銷已完成的 Docker 設定，也不得因 services.yaml 寫 Docker 就說 Production 已切換。

## 3. 2026-10-02 中斷點，必須重新核對

下列是前輪實查基線，不能直接當本輪現況：

- Sign repo：`F:\00-Ticenpi-SaaS\TicenpiSign`，GitHub `tcp-git-313/ticenpi-sign`，主線 main。
- 遠端 main：`5d26339a1e5a1ee003bd5ff7e6db282d4a38e838`。
- 主資料夾 main：`e4c7432620fa38bde221c8a9ca6aa911c3c80a8a`，46 項未提交；未清除或覆蓋。
- `.worktrees\sign-main-next` 是乾淨 detached worktree，HEAD=5d26339。RC 已快轉到遠端 main，`codex/sign-w6-rc-20260925` 本機與遠端已刪除，不要重建。
- 舊 WIP 備份：`F:\Ticenpi-git-archive\sign-main-wip-20261002-e4c7432.patch`，202471 bytes，SHA256 `e9fbc45e175a05fdbc19e6bfd0fb5a86357da4eca6bbae9c751f4817caf33f11`。
- ignored storage 備份：`F:\Ticenpi-git-archive\sign-main-ignored-storage-20261002.tgz`，4519 bytes，SHA256 `0a3147e49a7269aea087e3e0551fa6798d607a8f424a7cdbf3ef46268f80522e`；含 backend/src/storage 三個程式檔，不是完整客戶 runtime 資料備份。
- 原 patch `git apply --reverse --check` exit 0，只檢查沒有套用。
- 已保留 charter／交接／ops launcher／gitignore 的 commit：1df3b9f。
- 商用權限／access_denial 實作：83f405e；Docker／操作腳本來源：`193315d46213daefd127e719d61065178d62c420`；digest pin：`61c100058062cb0b416b32e692c0b88ac3f22396`。
- source CI：37000626375 success；main CI：37001659663 success。前輪本機 backend 24/24、frontend 76/76、build 與 bundle 22 files PASS。
- 中央 deploy 本機 main=1f86cf7，Sign Production services.yaml 已改 docker-compose，其他產品不可改。
- Staging release=`20261002-193123`，DEPLOYED、NOT_ACCEPTED。history 的 commercial_verified=false、e2e_verified=false。
- `20261002-192900` config SHA 曾填錯，VOID-NOTE 已作廢；history 不改寫、不再使用。
- Staging backend digest：`sha256:1a84e8727edb5bdd55084a6065923b0316616fa4c181bdf44fb6b96b99420b2e`。
- Staging frontend digest：`sha256:73a1300df837b3206dce7654d4695759b5f4475b60544e9f6e2e0c87727def58`。
- Staging health：env=staging、release_id=20261002-193123、identity.git_sha=61c100058062、project=jlsqjvehwblkeuycjoyj、Seat=require、Studio gate=off。此 git_sha 是 deploy-config，不要誤當映像 source。
- Staging `/api/access` 無 token=401 NO_TOKEN、壞 token=401 BAD_TOKEN；固定 A/B 真人 Seat 與完整簽署流程未驗收。
- 09-27 fixture snapshot 曾有 sign active／seat_limit=2／A assigned／B unassigned；必須重驗，不能再依 09-25 舊卡無 entitlement 就補一次。
- Production：systemd ticenpi-sign active/enabled，nginx :8901→127.0.0.1:8900；舊 health=200/env=production/沒有新版 identity。
- Production Supabase ref 正確 zfpkxsulehkbyuqkkcml，但旧 env PUBLIC_BASE_URL、SIGN_PUBLIC_BASE_URL 仍是 sign.tplex.top，P0 尚未在線上解決。
- Production Docker 尚無容器，127.0.0.1:8903 空閒，新 shared env 尚未建立。
- 目前 promote DryRun 正確擋下：no accepted-staging.json。
- VPS=ubuntu@147.224.255.234。只能沿用既有 SSH/scoped loader，不找、搬、印密鑰。

## 4. N0：確認來源與安全完成單一 main

先 fetch Sign 與必要的中央 deploy，列 HEAD、origin/main、工作樹狀態、worktree、遠端分支、目前 CI/release。若有人新增改動，先逐檔識別，不能覆蓋。

驗證兩份舊備份 SHA，另外建立本輪完整備份清單：tracked binary patch、untracked 檔案及 SHA、被 ignore 的必要程式。不得覆蓋原備份；不要把 secrets／真實文件上傳 GitHub。

逐檔分類：已在 main 保留、被 main 新版取代、仍獨有需要保留、用途未明。不要只比較行數或宣稱 main 是超集。特別檢查 auth.js、四個大檔、.gitignore 雙方規則、charter、交接、ops launcher、ignored storage，以及兩支舊操作腳本 fix-prod-tplex-url.ps1／remove-old-worktrees.ps1。

本次可以自主完成備份、比對與可逆封存；原任務要求「過時的列給使用者確認」，因此把要丟棄／替換的精確清單與理由一次呈現，僅在需要丟棄時等待這次確認，不能假設已確認。等待時繼續在既有乾淨 detached worktree 修脚本及做測試，不新建分支。

獲確認且所有獨有內容保留後，才選擇性恢復明確過時的 tracked 檔、移走衝突 untracked 檔到已驗證 archive，再 main fast-forward。本機主資料夾最後須在 main 且包含正式候選版；之後程式與設定都 commit 到 main。臨時 detached commit 若已產生，經驗證後快轉到 main 再推送，不留下只有 detached 才有的成果。

記錄整理前後 status、patch/檔案 SHA、每個保留／取代項目、main 包含原 Production source 的證據。不要清除 backend/storage、samples、.env、node_modules 或其他 ignored runtime 資料。

## 5. N1：修 Production 一鍵腳本，必須真能識別新舊版

沿用 `scripts/sign-production-cutover.ps1`，修最小必要內容，不另造一套跳過中央治理的部署器。

### 必修 A：健康與身分檢查

目前 :80/:109 只 grep env=production，舊 systemd 也符合；:111 接著停止舊服務，會誤判切流成功。

- 使用 JSON parser 讀實際 health 結構，不以 grep env 或 HTTP 200 判成功。
- expected identity 從凍結的 accepted-staging、Production policy、實際新 release metadata 取得；不得手寫假 SHA。
- 明確區分 artifact source commit、deploy-config commit、release id、image digest。若現有 health 只回 config git_sha，不得拿它冒充 accepted source；用可靠 image label/provenance 與必要的最小 health 修改補足。
- 切換前 :8903、切換後公開 sign.ticenpi.com 必須指向同一新 release、Production project、Seat=require、正確 source 與固定 digest；舊版無 identity／錯 release／錯 commit／錯 project／錯 digest 均拒絕。
- 只有外部驗證與必要 smoke 全 PASS，才停止／停用舊 systemd。保留舊程式、env、unit與 nginx 備份，勿刪除。
- curl/JSON parse/timeout/nginx test/reload/systemctl/Docker/git 任一步失敗須非零退出；禁止原樣印整段可能含敏感內容的 response。

### 必修 B：可靠回滾與重入

目前 :126 `docker compose -p ticenpi-sign stop || true` 缺 compose path/env/workdir，失敗被吞。

- 使用實際 release 的精確 `-f`、`--env-file`、project；記錄切換時的 release、舊 systemd 狀態、nginx 備份 SHA與路徑。
- 不用「最新時間檔名」猜回滾備份；選定此次切換的備份並驗 SHA、目標路徑。
- 回滾順序：舊 systemd 恢復並健康→nginx backup 驗證／還原→nginx -t／reload 成功→公開流量確認回舊版→停止此次 Docker 並核對停止狀態。
- nginx -t 或 reload 失敗時，不能繼續停 Docker；Docker stop 失敗不可印回滾成功。所有故障非零退出，保留可供人工處理的安全狀態。
- 首次轉 systemd→Docker 與以後 Docker release rollback 是不同路徑，分別說明；中央 manifest 已是 Docker 时回舊 systemd不能謊稱中央 Docker audit PASS，交付狀態對齊步驟，禁止順手覆蓋其他服務頁。
- 重跑 Check／PrepareEnv／Deploy／Rollback 不覆寫已驗證備份或既存 env，不誤停其他 release；既存 env 只驗證，缺值交使用者，不自行旋轉 secret。

### 一鍵操作與首次納管

- 新增或完善一個使用者入口（例如 `-Step Deploy`），從新 main 路徑一行執行：凍結與 gate Check→安全 PrepareEnv→Production preflight→中央 promote→首次納管必要步驟→確認 :8903 實際已啟動→Cutover→外部 Verify→真實 DEPLOYED 證據。
- 查實中央首次納管 `.cutover-done` 機制：首次部署可能只解壓、不啟動。腳本不可把 promote exit 0 當 Docker 已跑；實作需遵循中央 release/env/identity 契約，先用隔離環境驗證，不在 Production 試做。
- 固定已接受 source/config/digests；交付時寫入可驗證凍結資訊，不在操作當下無條件抓最新 origin/main 後升級。若 main/accepted/manifest 漂移，拒絕並短報，不部署另一份。
- secret 僅在遠端從舊 env 複製必要白名單鍵到新 env；原 env 保留。缺 secret 不自動造值掩蓋問題，暫存檔 trap 清理、權限 600、不輸出值。
- 最終只需使用者貼單行，不先貼變數區，不手改 compose，不要求重新 build；另給單行 Check、Verify、Rollback。
- 證據紀錄使用真實返回的 release id/commit/digest 與工具參數。僅在當次結果已驗證才設對應旗標；Production canary 尚未做時最多 DEPLOYED，不能 record-accepted。

### 必要測試，不碰 Production

為高影響切換腳本做有意義的回歸與隔離演練，至少涵蓋：

1. 舊 systemd env=production、但無新版 identity→拒絕，不能停止舊服務。
2. 新版 release/source/project/digest 任一不符→拒絕。
3. 外部回舊版或 curl/JSON 失敗→還原路由且不宣告成功。
4. nginx -t/reload 失敗→不能執行後續停服務。
5. SSH 任意 cwd 仍用正確 compose/env/project；Docker stop 非零→失敗。
6. 首次納管只解壓→不能宣告容器已啟動。
7. 重複執行、env 已存在、回滾備份缺失／錯 SHA／路徑越界→安全處理。
8. main／accepted 交付後漂移→禁止部署。

執行實際腳本邏輯或共享 helper 的測試，不只掃字串存在，不用 mock success 當遠端真切換驗證。分開標註本機／隔離演練／Production 唯讀／Production 真切換未執行。

## 6. N2：核對商用、P0 與合約完整性

- 業務受保護 API 都走使用者 JWT→my_commercial_context()→product_seat_status('sign')、fail closed；Seat 預設 require，skip 僅明確 local 且不得接 Production。未知格式／逾時／錯誤不可快取放行。
- Production 不採信 Staging test_access=allow，不可留 dev bypass。admin 合約豁免不等於公司 Seat 真人通過。
- `/api/access` 與受保護 API 的 detail.access 符合 ticenpi.access_denial.v1；no_entitlement、no_seat、unavailable 與 401 畫面／重試／改帳號行為一致。
- 匿名客戶簽署 API 沿用簽署 token、效期、案件歸屬限制，不要求客戶買商用 Seat；未帶 token／竄改 token／跨案件必拒絕。health/live 等必要公開端點保留。
- Production SIGN_PUBLIC_BASE_URL 固定 https://sign.ticenpi.com；前端 runtime 注入只接受 zfpkxsulehkbyuqkkcml，Staging 只接受 jlsqjvehwblkeuycjoyj；容器啟動與 bundle/CI 有 ref/舊網域檢查。
- 既有程式已滿足就保留，不重寫。若需修應用來源，必須新 commit→exact CI→新 artifact→新 Staging 驗收，不能繼承舊 ACCEPTED。
- 確认目前无新增 schema。若需要 DB migration，不自行套用或造新 migration 變通，列具體 blocker與使用者步驟。
- services.yaml 只改 sign/signruntimestaging 的必要字段；identity 不放 requiredEnv/shared env；兩邊固定同組不可變 digest；Production :8903只綁127.0.0.1；外部仍 tunnel→host nginx :8901，不動 Cloudflare/DNS。
- 注意 Studio gate：Staging off、Production on。這是已知 config 差異，必須在交付中說明；尚未執行 Production Studio 真登入不可標 PASS。

## 7. N3：S1–S6 實測、CI、釘 digest、部署 Staging

跑與修改相關的完整可執行單元／合約測試、前端 test/build/bundle、新增腳本回歸；不執行會直寫 Supabase 或建帳號的舊 integration tests。

中央配置如有改動，跑 manifest_tool、validate-services-manifest、相關 pytest 與 Sign policy gate，不改其他產品繞過測試。

選擇性 commit/push main，等待 exact commit CI 全部 completed/success，驗 artifact provenance→source→backend/frontend digest。禁止拿「分支最新綠燈」代替 exact SHA。

config-only 釘 digest 若 CI仍重建另組映像，不追逐新 digest，引用原已驗證 artifact；不要為本任務重設整套 CI。應用／image inputs 變動则必须接受新 source artifact。

打包用 main 的乾淨 detached worktree；查 status、SHA，依序 DryRun→Preflight→Sign Staging deploy。已有 release完全符合且无需部署則驗現有版本，不無故重部署。

輸出 source/config/CI/run/release/backend/frontend digest/health/smoke/audit；health/git_sha語義和來源說清楚。遇 exit 3 SUCCESS_DECISION_REQUIRED 列原因，未決定前不驗收；rollback 只有 verified healthy 和正確 target digest 才通過。

## 8. N4：固定 Seat 夾具真人 E2E → record-accepted

先唯讀重驗 Staging。固定 customer=STG-E2E-B-20260907；A=job0975805890@gmail.com，B=tcp.ai313@gmail.com。兩人應一般 user／非 admin／test_access=null、同一 active business Customer、sign entitlement active/seat_limit至少2、A assigned、B unassigned。

不得使用 platform_admin 取代 Seat A/B，不拿 email／陣列位置猜 user_id；以中央實際回傳精確識別。

若缺 fixture，由使用者在 Staging 以 canonical admin_set_entitlement／admin_list_customer_members／assign_product_seat 操作，提供讀前→最小操作→讀後步驟；你不寫 Supabase、不直寫表、不改角色或其他產品。若已符合就不要補。

使用可用的浏览器工具，遵守對應 skill，先取得真實 URL/state。真人 Google 密碼／MFA／驗證碼由使用者輸入，你不得索取秘密、建立新帳號、繞過登入或重複用已失敗的上傳方式。

只在需要時一次短列使用者操作，停在正確畫面等其完成；完成後主動恢復驗收，不把「請你測試」當最終交付。不因登入受阻就捏造 E2E或宣稱已達READY。

每項使用合成資料並附遮罩證據：

- A 真登入，/api/access=allowed、業務 API=200，建立自己的案件；不得讀 admin/B 的案件。
- B 真登入，/api/access與業務資料 API=403、access_denial no_seat，畫面「尚未指派使用席位」，無案件資料洩漏。
- 商用未開通／無有效 Customer／停用或過期等有可用合法測試身分時驗拒絕；不能為湊矩陣自己更動中央。必要身份缺失讓使用者安排，區分「單元測試覆蓋」與「真人覆蓋」。
- 無 token=401、壞 JWT=401、平台測試 deny 拒絕；allow/admin只驗明文授權矩陣，不當Seat測試。
- 合成兩頁 PDF 上傳→看得到每頁預覽→框選→發送／複製正確 Staging 簽署連結→無痕匿名客戶看PDF→簽署→背景產物→下載，實際檢查簽名位置與頁數。
- 竄改／replay token與跨案件／跨業務資料存取依合約拒絕；不貼原token、signed URL、headers或cookies。
- Staging cookie 隔離、登入登出循環、nginx header policy，不能影響 Production cookie。
- 記錄每項 PASS/FAIL/WAITING_USER，截圖／狀態碼／release/source/config及清理／保留的合成案件。

只有當現行 release 的正式 S7 矩陣全通過，才依實際工具介面 record-deployed及record-accepted；不帶未驗證旗標。正確區分 source與config，不手寫SHA、不改舊history、不用作廢release。

驗證 `.release-evidence\sign\accepted-staging.json` status=ACCEPTED，runtime/health/auth/commercial/e2e都true；完整source/digest與運行容器、CI、證據一致。

## 9. N5：交付 Production 操作包，只準備不執行

將Production compose釘到同一組accepted digest，Production commit literal=accepted source，release id由中央注入，project=zfpkxsulehkbyuqkkcml、database target正確、Seat=require、Studio=on、正式網址。

做到 main與remote main包含完成程式和部署配置、CI exact SHA綠、主資料夾新脚本實際存在；使用者不能被交付到已移除的worktree路徑。

實跑 `promote.ps1 sign production -SourceOverride <凍結乾淨來源> -DryRun`，須PASS，沒有Production寫入；script Check/唯讀預檢實跑，核對舊systemd/nginx與目標port、不印密鑰。

當新Production env尚未存在，完整Production preflight會等使用者PrepareEnv；不能偽稱已PASS。交付一鍵入口必須先安全建立/驗證env再實跑正式preflight，失敗立即停在切流前。這一項可明示 `PRODUCTION_PREFLIGHT=WAITING_USER_PREPARE_ENV`；其他READY條件仍須全PASS。

新 env只有在使用者執行時才建立；LUNA不執行PrepareEnv/Promote/Cutover/Rollback。Production真切換與canary狀態一律NOT_RUN，不能標ACCEPTED。

必要交付檔案：

1. 修正並測試的 `TicenpiSign\scripts\sign-production-cutover.ps1`（或保留其相容接口的薄入口）。
2. `TicenpiSign\docs\handoffs\SIGN-PRODUCTION-READY.md`：凍結source/config/platform commit、CI、Staging accepted release、digest、env鍵名存在性、路由/port、初次切換與回滾差異、證據限制。
3. 本輪獨立evidence資料夾、E2E matrix、腳本演練log、main/WIP reconciliation、hash清單。history不可改寫。
4. 更新既有Sign交接與中央Sign合約卡/CURRENT_STATE，區分source/Docker-ready/Staging ACCEPTED/Production仍systemd；不得先寫Production已DEPLOYED。

使用者交付最少四條單行命令：

- Check（純唯讀）；
- Deploy（一行依序完成所有已授權給使用者的準備／preflight／promote／切換／驗證，固定accepted候選）；
- Verify（公開identity/digest/smoke與實際服務狀態）；
- Rollback（明確本次舊systemd還原目標，失敗不能假成功）。

各命令附預期結果、非零時停哪裡、能否重跑。Production真人canary與record-accepted另列使用者最小步驟；不把公司1088方案付款自動化或上架頁修改擴成此任務，也不宣稱尚未測的方案銷售／付款全鏈已通過。

## 10. 完成條件與回報格式

不要完成一個phase就結束；持續直到全部可執行工作完成。需要人介入，短列必要操作，等待且继续獨立工作，完成後回到同一任務接續。

只有同時滿足以下條件才能寫 `READY_FOR_USER_PRODUCTION=YES`：

- main/WIP收斂完成，所有獨有成果與archive保留；無未確認丟棄項。
- 兩個已知脚本缺陷及首次納管/凍結/失敗處理通過真正回歸與隔離演練。
- exact CI PASS；source/provenance/digest鏈可核對。
- 現行Staging真人商用/Seat/簽署E2E全PASS且正式ACCEPTED。
- promote DryRun PASS，Production compose與accepted digest完全相同。
- Production Check/可執行唯讀檢查PASS；新env待使用者建立可如實列為唯一前置，腳本已涵蓋并測試该路徑。
- 使用者的一行命令指向真實main檔案，成功條件與回滾腳本完整，尚未執行Production寫入。

否則READY=NO，列具體第一阻擋關卡與使用者最小動作；禁止用「操作包已寫」假裝達標。

最終繁體中文短報：

```text
STATUS = PASS | PARTIAL | FAIL | HARD_STOP | WAITING_USER
READY_FOR_USER_PRODUCTION = YES | NO
MAIN = <實際40hex>
STAGING = <release / ACCEPTED或實際狀態>
SOURCE / CONFIG / PLATFORM = <各實際40hex>
CI = <exact SHA / run / 結果>
DIGESTS = <backend / frontend>
PRODUCTION_PREFLIGHT = PASS | WAITING_USER_PREPARE_ENV | FAIL
PRODUCTION = 舊systemd仍運作；本任務未寫入
使用者執行：<真實完整單行Deploy命令>
預期結果：<身分一致、Docker提供正式流量、舊systemd停止但可回滾、真實DEPLOYED>
回滾：<真實完整單行Rollback命令>
證據與操作手冊：<實際檔案路徑>
未測項：Production真切換／真人canary，以及其他實際未驗證項
```

不要只回答計畫，不要說「可以了」卻沒有測那條路徑。完成準備、Staging ACCEPTED與可核對的一鍵交付後，把Production執行交回使用者。

<!-- END: sign-production-commercial-docker-luna-closeout -->
