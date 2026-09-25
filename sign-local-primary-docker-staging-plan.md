# TicenpiSign：本機主來源、Local Docker、獨立 Staging 完整計畫暨執行提示詞

> 交辦模型：GPT-6 Astra；建議 thinking：xhigh。這份文件是供使用者確認的完整計畫，也是確認後可直接交給工程 AI 執行的提示詞。確認前只做唯讀查證與可逆的本機準備；Production 不在本任務範圍。

## 你要達成的結果

將 TicenpiSign 依 TicenpiDM 已採用的模式整理成一條可追溯的流程：**同一個本機 canonical source → Local DEV → Local Docker → 乾淨 release candidate → exact-commit CI → 不可變 ARM64 映像 → 獨立 Staging → 真人驗收與凍結**。完成後，使用者能明確辨識本機資料、Local Docker 資料及 Staging 資料各自在哪裡、由誰擁有、如何備份／還原，以及 Docker 目前跑的是哪一份來源。

「本機資料為主要資料」的具體規則如下，不得自行改寫成「容器資料或雲端資料覆蓋本機」：

1. **原始碼、規格、測試與建置輸入**：以本機 SIGN repo 為主；每次驗證只指定一個 canonical source root。`LocalSync` 可選目前主工作樹，`ReleaseCandidate` 可選經核對的隔離工作樹，但同一次的 Local DEV 與 Local Docker 必須指向所選的同一個 root 與相同內容指紋。本機 DEV 和 Local Docker 的內容要可用 SHA、dirty state 與指紋相互對照。不可從舊 worktree、VPS、快取、其他產品或手動複製的殘缺資料夾建置。
2. **本機文件與測試資料**：先盤點 SIGN 實際本機資料路徑、junction、`backend/storage`、上傳／PDF／暫存，以及哪些屬於使用者資料。原始資料不進 Git 或 Docker image。Local Docker 使用獨立持久化位置或經核對的本機資料快照；其寫入不得靜默覆蓋本機主資料。若要共享可寫目錄，先提供逐路徑風險、備份與回滾方案，等待使用者確認。
3. **關聯資料庫**：SIGN 目前有 Supabase `sign` schema，不能把「本機為主」誤解成直接改寫 Production Supabase。Local DEV 的資料目標須明確標示；Local Docker 的正式驗收 profile 只允許綁已核對的 Staging 專案 `jlsqjvehwblkeuycjoyj`，或另設可丟棄本機 DB。任何真實資料複製到測試環境都需先完成脫敏、用途及回收核准。
4. **Staging 運行資料**：使用獨立 Supabase、金鑰、R2 bucket／prefix、token pepper、Compose project、port、volume、網域與測試身分。它不是本機主資料的自動同步副本；不得反向覆蓋本機，也不得連向 Production 專案 `zfpkxsulehkbyuqkkcml`。
5. **Production**：現行 `sign.ticenpi.com` 為 systemd 服務，保留現狀。Production 仍按其已核准的 Supabase／R2 來源運作。不得因本計畫對它做資料搬移、部署、DB migration、DNS 或流量切換。

## 已知基準，開始時仍須即時重驗

- `TicenpiSign` 主工作樹有未提交 WIP；`main`／`origin/main` 在上次盤點時是 `e4c7432620fa38bde221c8a9ca6aa911c3c80a8a`。不可 reset、clean、stash、廣泛 stage 或覆蓋這些 WIP。
- 隔離工作樹 `F:\00-Ticenpi-SaaS\.worktrees\sign-staging-readiness` 已有來源快照 commit `492aa733df1c2c87995c9d0d20fa685c182686c8` 及 Staging cookie／前端測試修正 commit `b06062489d5783159a63f4d739dea2ed2e1f0b5c`；它們尚未推送，也不是完整 release candidate。19 個程式／DB 檔在上次盤點時與 VPS 正式 release 的 SHA-256 相同。
- 前端在該隔離分支曾達 48/48 測試通過，使用非真實 placeholder anon key 的 Vite build 通過。後端現有測試會建立 Supabase 測試使用者，不能直接用 `npm test` 當安全的本機／CI 門檻。
- 中央部署 `F:\00-Ticenpi-SaaS\deploy\services.yaml` 已有 `sign` Production，尚無 SIGN 的獨立 Staging 條目。上次盤點時 `sign-staging.ticenpi.com` 無 DNS 解析；不要憑此過期快照推論現在仍然如此。
- SIGN 用 `sign` 作 Launcher／產品代碼；本機 WIP 已有 caller-token `my_commercial_context()` 與 Studio 心跳，中央 Seat 的 assigned／unassigned 強制決策尚未證明。SIGN 的公開 `/s/<token>` 簽署流程不得被業務員 Seat／Studio 閘門誤擋。
- 參考 `F:\00-Ticenpi-SaaS\deploy\PRODUCT_TO_STAGING.md`、DM 的 `scripts/start_local_docker.ps1`、`docker-compose.yml`、`deploy/runtime-staging/compose.yml`，以及 SIGN 的 `docs/03-operations/SIGN-STAGING-GATE-2026-09-25.md`。沿用 DM 的原則，不盲目複製 DM 的 port、資料庫表、profile 或產品測試。

## 先交付的計畫書與決策表

執行前更新一份中文計畫書，逐項列 `目前狀態／證據／修改範圍／驗收方式／回滾方式／依賴者`。至少確認以下決策；證據不足填 `UNKNOWN`：

| 決策 | 預設方案 | 驗收條件 |
| --- | --- | --- |
| Canonical source | 每次指定一個 SIGN 本機 root；主工作樹供 LocalSync，隔離 worktree 供乾淨候選 | 同一次 Local DEV／Docker 使用同 root 與 fingerprint；CI 對上候選 commit；dirty LocalSync 不會被當 release |
| Local Docker 用途 | `LocalSync` 可驗當前 WIP；`ReleaseCandidate` 必須 clean | build label、health、驗證腳本同時顯示 root、SHA、fingerprint、state、purpose |
| 本機業務資料 | 本機主資料保留在原路徑；Docker 用獨立 volume 或一次性快照 | 路徑、容量、備份與還原已核對；重建容器不遺失資料，不會覆蓋主資料 |
| Staging runtime | 新增獨立 Docker Compose，Production systemd 不動 | 獨立 port、volume、Supabase、R2、env、DNS、OAuth、cookie 和回滾路徑 |
| 商用產品代碼 | `sign` | Launcher、Commercial、Seat RPC、服務登記及測試一致 |
| 外部簽署 | token 邊界，不要求業務員 JWT／Seat／Studio | 有效／過期／重放／越權 token 與 PDF 下載路徑通過測試 |
| 升級範圍 | 先到 Staging freeze | Production 19–22 不執行；另開變更窗口 |

## 分段執行：01–18 到 Staging

### 01–04　來源、隔離與契約

1. 唯讀查 SIGN 原始碼、主工作樹狀態、隔離分支、Production release 身分、中央部署登記和資料路徑。按檔案雜湊核對 WIP 與實際運行版本；列出不能對上的檔案。保存清單但不要把密鑰／客戶文件內容輸出。
2. 保護主工作樹 WIP，在隔離 worktree 做後續變更。凍結 SIGN 的業務員、Studio、外部客戶簽署、PDF、audit、R2、Local DEV／Docker／Staging 邊界。
3. 把 Launcher id 與 Commercial `product_code=sign`、中央 Seat 的狀態／豁免／錯誤語意核對到同一份契約。若平台 ADR 與即時 DB 契約衝突，停在契約關，不查詢或建立測試身分。

### 05–06　Auth、Entitlement、Seat 與資料邊界

4. 業務 API 依序驗 JWT、Commercial entitlement、中央 Seat、Studio 心跳，再建立 SIGN profile；使用 caller JWT 呼叫中央 RPC，不把 service-role、任意 user id 或 email 當 Seat 授權替身。RPC 失敗、未知狀態、過期或未指派須 fail closed。platform admin 等豁免只按中央契約處理。
5. `/s/<token>`、對外簽署 API、signer PDF 下載、背景工作與 health 分別定義准入，維持公開簽署路徑可用。測跨業務員／跨案件、token 過期／重放與權限撤銷。檢查 `sign` schema 的 grants、RLS、service-role 使用與 migration 可重放性；不碰 Production DB。

### 07　Local DEV／Local Docker 同源與資料持久化

6. 建 SIGN 專屬本機啟動入口、Dockerfile、Compose、資料目錄清單與驗證腳本。Local DEV 與 Local Docker 的來源檔案必須同 root；運行中的開發服務若來自其他 root，停止 Docker build 並指出 PID／路徑。源指紋含待建置的未提交程式檔，排除 env、secret、客戶資料、node_modules、dist 與 log。
7. `LocalSync` 可從 dirty source 做唯讀建置快照，但輸出標 `dirty/local-only`，不可推送、不可用於 Staging；`ReleaseCandidate` 必須 clean、從同 SHA 重建。容器資料使用獨立命名 volume 或經使用者確認的唯讀資料快照；持久化、備份／還原、容器重建及資料一致性都要實測。不得用 `down -v`、清理主資料或共享 Production volume。
8. 把後端純單元／合約測試與需要可寫 Supabase 的整合測試分開。預設 `npm test` 不得建立真實使用者或修改 Staging／Production。整合測試先確認 project ref、資料名稱空間、fixture 所有權與清理方式。前端測試、建置、cookie 隔離、掃描手動／自動 A/B 回歸均需實際證據；手機真機結果與單元測試分開報告。

### 08–12　CI、不可變映像、設定與回滾

9. 為 SIGN exact commit 建 CI：安全本機測試、前端建置、靜態安全檢查、ARM64 backend／frontend 映像建置及 digest／provenance。不得把測試所需的 service-role key 烤進 image 或 frontend。config-only commit 只驗證 pinned digest，不重建／推送映像；應用原始碼變更必須走建置路徑。
10. 建 `deploy/runtime-staging/compose.yml`，以 `image@sha256` 固定映像；新增中央 `signruntimestaging` 登記，只改 SIGN Staging 自己的頁面。port 先檢查全平台占用，不自行猜數字。VPS env 存在且 project ref 正確才允許部署；缺值 fail closed。
11. 補 SIGN 的 Staging cookie 守衛、host-only／localStorage 驗證、nginx header policy 與 API 不轉送 Cookie 規則。Studio OAuth redirect 指 Staging 網域；Supabase Staging redirect allowlist 含正確網址。R2、簽署 URL、token pepper、storage 和 volume 必須與 Production 隔離。
12. 在部署前先證明只回滾 SIGN Staging 的映像及設定；migration 另有資料備份／隔離還原方案。清楚區分「程式回滾」和「資料回滾」，不得用退 image 假裝 DB 已復原。

### 13–18　Staging 部署與驗收

13. 完成 DNS／Cloudflare／OAuth／環境檔前置條件後，依序執行乾淨來源 dry-run、唯讀 preflight、exact-commit CI 核對、部署。部署時寫入 source SHA、config SHA、image digest、release id；服務及外部 health 的身分須一致。
14. 確認內外健康、安全標頭、Cookie 隔離、401/403、RLS、外部簽署 token、PDF 合成／下載／R2、audit 及 Studio 心跳。health 200 只算存活，不算業務 E2E。
15. 以經中央核准的非 admin 商業客戶、有效 SIGN entitlement、至少兩位有效成員執行 assigned 與 unassigned 真人 E2E；再驗 platform admin 豁免與到期／停用。不得以 platform_test_allow、mock、RPC 存在或測試帳號截圖取代 Seat E2E。
16. 驗證客戶不登入也能開有效簽署連結、簽名、下載；無效／過期／重放連結按契約拒絕。測試資料的建立／清理需可追溯，不碰真實客戶文件。
17. 保存 source commit、CI run、digest、config commit、DB fingerprint、runtime identity、回滾、assigned／unassigned 及客戶簽署證據，標 `PROVEN／PLAUSIBLE／NOT_SUPPORTED`。只有每項關卡有直接證據才宣布 `STAGING_FREEZE=PASS`。

## 硬性停止與權限邊界

- 使用者目前要的是**完整計畫確認**。在使用者明確確認本計畫前，不執行 Staging DB 寫入、fixture 指派、DNS／Cloudflare 變更、VPS 部署或 Production 變更。允許唯讀盤點、隔離工作樹修正與無外部副作用的本機驗證。
- 機密、MFA、金鑰輸入由使用者控制。不得讀出或貼出 secret；只查存在性、所屬環境與必要的雜湊／指紋。
- 不 reset、clean、stash 或覆蓋 SIGN 主工作樹；不停止其他產品服務，不清除任何 Docker volume，不把 Production 的 service-role 或業務資料灌進 Local Docker／Staging。
- 任何來源身分、環境 ref、Seat 契約、測試身分、schema、CI 或回滾證據不符，直接報 `HARD_STOP` 與可驗證原因；不得用舊文件或推測補成 PASS。

## 執行者最後交付格式

1. 先回報 01–18 每關 `PASS／PARTIAL／FAIL／HARD_STOP`、證據路徑與缺口；19–22 明確維持未執行。
2. 顯示 canonical root、Local DEV／Local Docker 的 source SHA／fingerprint、LocalSync 或 ReleaseCandidate 身分、資料位置／備份及是否曾寫入本機主資料。
3. 若到 Staging，顯示 exact source/config SHA、CI run、ARM64 digest、runtime 身分、DNS／外部 health、Seat 與簽署 E2E、回滾結果；不得把 mock 或 health 說成真人驗收。
4. 列出仍需使用者處理的項目，合併成一次清單；沒有完成的關卡保留原狀，不宣布 Production readiness。

<!-- END SIGN LOCAL PRIMARY DOCKER STAGING PLAN -->
