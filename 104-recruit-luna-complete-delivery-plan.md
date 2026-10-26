# 104Recruit — LUNA 人才搜尋優先完整交付任務

版本：2026-10-26 / SEARCH FIRST V3
指定執行模型：GPT-6 Luna；thinking level：max
工作根目錄：F:\AI\_104Recruit
主頁：http://127.0.0.1:43104/recruiting
104 人才搜尋來源：https://vip.104.com.tw/search/listSearch
本文件本機副本：F:\AI\_104Recruit\docs\planning\17-LUNA-SEARCH-FIRST-DELIVERY-PLAN.md
本輪基線快照：F:\AI\_104Recruit\docs\planning\LUNA-SEARCH-FIRST-SNAPSHOT-20261026.json

你是執行者。直接完成本文件要求的程式、來源連接、資料遷移、頁面、測試與實際驗證，修復發現的問題。不要再只交另一份計畫，也不要把已存在的本機空表單當作完成。工程檢查自行推進，產品使用者不負責逐項點擊驗證。

## 1. 最新需求與任務主軸

使用者已明確糾正：
1. 104 原本是選單、階層多選或固定選項的欄位，本機應採相同選擇模式，避免靠手打地名、職類或列舉值造成錯誤。
2. 能與原站按鈕和控制項完整對應的，就完整對應；標籤、選項、語意、數量限制、取消/確認與實際送出的條件都要一致。
3. 搜尋入口是 https://vip.104.com.tw/search/listSearch；直接向 104 搜尋符合條件的人才。
4. 企業職缺與此人才搜尋沒有必然關聯；取消「先載入企業職缺、選職缺才能篩選」的前置條件。
5. 原需求繼續有效：每日查看額度預設 300 且方案可設定；每天採用 N 筆有效電話；跨日避開歷史已採用的人與電話；最後能複製或匯出給使用者的快速應徵流程。
6. 沿用目前正常登入的 Chrome 與已安裝連接器；不可再次啟動專用自動化 Chrome。
7. 交付完整使用頁面，不把 Gate、逐步驗證、SQL、scope、capability、工程 DTO 或內部階段放在主要操作流程。

本輪主流程：
建立人才搜尋方案 → 用對應 104 的選單設定條件 → 查看條件摘要 → 搜尋 104 人才 → 結果先去重 → 依額度查看必要明細及帳號可見電話 → 產生今日不重複名單 → 複製電話/下載 → 持久保留歷史排除。

「搜尋方案」是本機保存的一組搜尋條件；它不是 104 已刊登職缺。企業職缺只能是進階選填的標籤或歸屬資訊，既不決定能否搜尋，也不決定能否產生名單。先保證完全沒有企業職缺時也能走完人才搜尋流程。

## 2. 權威與授權邊界

- 最新使用者需求與本文件取代舊計畫中「職缺優先」「先選職缺」「只做本機快取預覽」「P12 即停止」等衝突條款。
- 本輪承接已授權的本機實作、目前 Chrome 連接、104 帳號有權查看的人才與電話讀取、額度/去重/本機名單/匯出。不得用舊 allowlist 或舊未實作狀態當作停止施工理由。
- 使用者登入、MFA、CAPTCHA、來源拒絕與帳號實際權限仍按實況處理；不繞過遮罩或邀請限制。
- 不自動發送站內訊息、邀請、簡訊、電話或對外應徵，不購買方案，不接通外部撥號。複製/匯出不等於已撥打。
- 不讀取或匯出帳密、Cookie、登入儲存與其他網站資料；不把候選人個資、實際履歷 URL、電話、企業頁面原文、金鑰或本機私密檔案發布到 GitHub。
- 原站有部分年齡、性別、族群、身心障礙等敏感欄位；本任務對應工作相關搜尋條件，不將這些敏感屬性接入自動篩選、評分、排除或名單分配。
- 本任務只操作 F:\AI\_104Recruit 及其 43104/55432 專案服務；不得更動共用 PostgreSQL 5432、其他專案、Docker/WSL 或 Production。
- 保留既有 dirty WIP；禁止 reset、clean、stash、廣泛 stage、覆蓋還原、自行 commit/push/deploy。本任務不要求發起另一個模型或新對話。
- 真正缺少登入、來源權限、使用者搜尋條件或每日 N 時，完成所有獨立工作後一次列出最小缺口；不要猜使用者要找哪一類人，也不要以缺少 N 為由停止選單、管線或頁面的開發。

## 3. 已核實基線與讀取順序

2026-10-26 的本輪本機核對：
- 分支 codex/bootstrap-gate-a，HEAD 26b0c620929a859071933ede41c05b1adfec8778。
- 目前有 96 個已修改/未追蹤檔案納入本輪基線快照；此數量是寫入本文件前的時間點，不是要求工作樹保持零變更。
- 技術為 Node 22、TypeScript ESM、node:http、靜態 HTML/CSS/JS、pg、Zod；沿用 package-lock.json，不為本次表單改造換框架或加入新的常駐服務。
- PostgreSQL 位於 127.0.0.1:55432/recruit104；已有遷移 001–007、SCRAM、應用/遷移角色分離、DPAPI 密鑰與候選人加密。
- 目前服務 build 為 104Recruit-CompleteDeliveryV2.1，health 的 database=true。接手時重新檢查程序與根目錄，不能只相信 PID 或舊 build。
- 原有 21 項單元與 9 項整合測試曾通過；這不是本輪的來源搜尋/電話驗收。
- 主頁是 public\recruiting.html/js/css；診斷頁仍在 /recruiting/debug。
- public\recruiting.html 的 job-select 是 required；locations、must-have、nice-to-have、exclude 是自由輸入；active-days 可輸入 1–365；這與原站不等價。
- src\daily-lists.ts 的 saveCampaign 強制 jobId 並驗證 availableJobs；listCampaigns、previewDailyList、commitDailyList 等以 recruiting_campaign 作為主要關聯。
- src\search.ts 的 campaignCriteriaSchema 是本機規則，locations 以整份 searchable 字串 includes 判斷，會混淆希望工作地/居住地，也可能從不相關文字命中。
- src\daily-lists.ts 的保存/讀取 mapping 目前還沒有完整往返 minExperience、maxExperience、activeWithinDays，必須專門回歸，不能只換表單外觀。
- src\recruiting-jobs.ts 的 startRun 目前沒有真實 worker，會等待缺少的 quotaInspect/searchPage/candidateDetail/contactRead。
- chrome-extension\background.js、console-bridge.js 與 src\chrome-bridge.ts 目前只有 inspect/jobs/detail 固定命令；裝有插件不代表人才搜尋與電話已接通。
- 最新本機 overview 曾顯示 bridge connected=false/UNKNOWN，但使用者現有正常 Chrome 的 104 搜尋表單可顯示登入後企業介面。分頁登入與本機 bridge 連線是兩個狀態，不可誤報使用者沒登入。
- F:\AI\_104Recruit\docs\recruiting\FINAL-ACCEPTANCE.md 明確承認來源未連接；本輪不能把 LOCAL_PRODUCT_READY 當作最終成果。
- 目前通用電話文字/CSV/TSV 已有實作；特定「快速應徵」接收程式規格仍未核實。

接手依序讀完整內容：
1. applicable AGENTS.md、本文件、本輪基線快照，以及 git status/diff。
2. F:\AI\_104Recruit\docs\planning\EXECUTION-STATE.md、MASTER-PLAN.md、14-DECISION-LOG.md。
3. F:\AI\_104Recruit\docs\recruiting\FINAL-ACCEPTANCE.md、BUILD-REPORT.md、CHROME-CONNECTOR.md、QUICK-APPLY-CONTRACT.md。
4. F:\AI\_104Recruit\public\recruiting.html、recruiting.js、recruiting.css。
5. F:\AI\_104Recruit\src\server.ts、chrome-bridge.ts、search.ts、daily-lists.ts、recruiting-jobs.ts、candidates.ts、dedupe.ts、quota.ts、delivery.ts、crypto.ts。
6. F:\AI\_104Recruit\chrome-extension\manifest.json、background.js、console-bridge.js。
7. F:\AI\_104Recruit\db\migrations 下 001–007，scripts\migrate.ts、test-db.ts、backup-db.ps1、secure-local.ps1，start-104recruit.bat。
8. F:\AI\_104Recruit\docs\source\candidate-search-discovery.md、contact-discovery.md、quota-discovery.md；它們的 UNKNOWN 是前輪狀態，應實際發現並更新，不能直接照抄到交付。
9. F:\AI\_104Recruit\docs\planning\16-LUNA-COMPLETE-DELIVERY-PLAN.md 只作舊版歷史與未衝突需求參考；其舊基線不能覆蓋本輪 live code。

快照 SHA256：82f7a099073cc11524823e864045798f6e4326fdd57587c9700a80b269789190。
快照中的 code hash 可用於辨識接手時變動；若有後續使用者 WIP，先比較並保留，不因預期中的變更直接停止或覆蓋。

## 4. 原站已觀察事實與尚未證實事項

本輪透過使用者現有正常 Chrome 開啟官方 https://vip.104.com.tw/search/listSearch，實際觀察到：

|原站欄位/操作|已觀察模式|本機必須採取|
|---|---|---|
|關鍵字|文字輸入；原站提示職務、求才條件、技能，並提示 + 與引號語法|保留原站文字與語法；驗證並預覽，不能自行把逗號或空白改成全部 AND|
|希望職類|職務類別選單，階層與複選，顯示已選擇 0/6，含查找、收合與確定|使用原站職類代碼、相同階層與選擇限制|
|希望工作地|地區類別選單，地區→縣市→區域階層，顯示已選擇 0/5，含確定|獨立的希望工作地選擇器；不可手打地名|
|居住地|獨立於希望工作地的選單入口|獨立欄位；上限與完整階層仍須再核實|
|最近活動日|不拘、1天內、3天內、5天內、7天內、14天內、21天內、30天內|單選固定值；不可輸入 6天/365天等原站沒有的值|
|更多查詢條件|展開/收合|使用相同基本/進階分組|
|總年資|不拘、無工作經驗，年數與比較方式選單；本次展開年數看到 1–25|依當前原站完整列舉與比較方式建模，不能擅自改成任意上下限|
|學經歷|經歷職務、經歷產業、最高學歷、科系、學校關鍵字、留學國家|逐項盤點選單、複選、文字與組合語意|
|希望條件|工作性質、希望產業、希望待遇、上班時段|對應工作相關選項；有原站數值輸入的欄位可保留受控數值輸入|
|其他工作相關條件|就業狀態、駕照、車輛、語文、擅長工具、證照、履歷更新日等|逐項取得現行 options、欄位依賴與選擇模式|
|E-mail 及聯絡電話|不拘、可直接 E-mail 及電話聯繫|建立原生單選；可用此條件提升有可聯繫人才的查詢效率，但不宣稱結果必有完整有效電話|
|搜尋提交|按鈕文字為符合人數及動態人數|本機執行要真的提交這組搜尋條件並讀取原站結果|
|清除條件|原站存在此操作|本機重設需同步清掉所有已套用條件|

觀察限制：
- 本輪只開啟表單與選單查看；沒有送出人才搜尋、開候選人明細、讀電話或採用人才。
- 未驗證完整 taxonomy、代碼、所有分支/上限、預設值、AND/OR、關鍵字完整文法、分頁及來源扣點。
- 5 與 6 是當時實際選單顯示，不代表永久不變；接手時重新觀察並版本化，不硬寫成永遠真理。
- 最近活動日與履歷更新日是不同欄位。當次畫面有預選活動日期，不得假設初始化一定是不拘。
- 不把原站現在顯示的符合人數當成可取得電話數。

## 5. 控制項對應與 100% 的驗收定義

先交付 F:\AI\_104Recruit\docs\source\search-control-matrix.md，內容不含候選人資料。
每個支援欄位一列，至少包含：
fieldKey、原站標籤、原站控制項類型、optionCode、label、parentCode、path、單/複選、maxSelection、預設值、階層與父子選取規則、AND/OR、清除方式、依賴欄位、未知/不拘語意、DOM evidence、觀察時間、schema/catalogVersion、覆蓋狀態。

必要規則：
1. 本機每個已啟用的原站條件都必須做到：選擇值 → 序列化 → 儲存 → 重新載入 → 套到 104 → 從 104 回讀，一致才准提交。
2. 不能把「畫了相似下拉框」寫成 100%。計算欄位/選項覆蓋率並列出未覆蓋項；未知項不得算已完成。
3. 不同欄位即使 label 相同也有不同型別與代碼域；職類代碼不能當產業或地區代碼。
4. 希望工作地與居住地各自保留；不在履歷整段文字中 includes「台北」就視為兩者都符合。
5. 大類選取的意義、父子互斥/合併、選取上限計算，跟原站回讀一致；不能自行把父節點展成所有子節點而超限。
6. taxonomy 查找可輸入名稱來找選項，但確認值只能選真實選項；不可因查不到就新增自由值。
7. 原站本來就是自由文字或數值的欄位不強制假造下拉選項；保留原站語法、格式、限制與空值規則。
8. 不支援/失效選項顯示具體原因並停用該操作；不得偷偷略過、改成不拘、用最相似地名或擴大條件搜尋。
9. 欄位缺漏或原站改版造成回讀不一致時不提交搜尋，保留草稿並指向需要修正的欄位。
10. 原站適用的工作相關欄位逐項列入矩陣；敏感欄位列為不進入自動篩選範圍，不納入宣稱的支援率。
11. 必須驗證不限/清除與 explicit zero 等差別，不能用 truthy 判斷漏掉合法值。
12. 外部 taxonomy 快取有來源、版本、取得時間、內容指紋與更新狀態；已確認快取可供離線編輯草稿，來源執行前必須驗證仍適用。

選項來源只用合法可見原站介面及其可確認的官方可讀資料；不可拿求職者端 104 的相似地區表直接假定與企業人才搜尋一致。不得讀網頁隱藏 app store、登入 token 或猜私有 API。若選單虛擬化/延遲載入，要逐分支觀察，不能把當前可見前十項當完整字典。

## 6. 搜尋方案、型別與資料遷移

採最小相容改造：
- 沿用 recruiting_campaign 的內部主鍵及既有 candidate_use/daily_list_item 關聯，將業務語意改為獨立的搜尋方案。
- source_job_id 在既有 schema 已可空；取消後端、API、前端對 jobId/availableJobs 的硬性要求。企業/工作區邊界仍必須驗證，不能隨取消職缺要求而解除公司隔離。
- 新增結構化 source_filters、filters_schema_version、catalog_version、獨立 source/local rules 與需要的 profile name/版本欄位，先檢查已有欄位避免重複造輪子。
- 主 UI 用 profileName；對外企業職缺名稱只能來自可選的真實關聯。禁止用虛構 jobId 或把搜尋方案名稱冒充已刊登職缺。
- 既有強制 job_title 欄位如保留相容用途，要明確 mapping；新版結果與匯出提供搜尋方案欄，沒有職缺時職缺欄為空。更新 payload/CSV 契約與相容性測試。
- 舊自由文字條件保留，不直接猜代碼轉換；可確定一對一者提出轉換預覽，模糊或不存在者標示需重新選擇。不得未經核對就用模糊對應執行真實來源搜尋。
- 舊名單、quota ledger、candidate ID、電話指紋、DNC 與已採用資料不可因改成搜尋方案而重建/清空。

型別契約：
- sourceFilters 是嚴格 schema：每個欄位採具名型別、原站 optionCode 或經驗證的自由值；reject unknown fields，限制大小/長度/數量。
- 將希望職類、希望工作地、居住地、活動日期、更新日期、年資 mode/operator/value、聯絡可用性分開建模。
- 實際代碼由來源觀察確定，本文件不捏造名稱像 1/7/30 就代表官方 value。
- localRules 與 sourceFilters 分開；加分、排除與本機分數預設收合且可關閉。原站結果不能再被偷偷套上預設 70 分而全部變成不符合。
- 本機額外硬條件如啟用，送出前摘要明確顯示；來源無該條件時只能在已合法取得資料上評估，不能宣稱已推入 104。
- 開始 run 保存搜尋方案 ID/版本、catalogVersion、canonical filters hash、原站回讀 hash、來源公司 scope、policy 版本；執行中編輯只影響新 run。
- 每一個欄位都做保存/重新載入 round-trip；修掉前輪經驗與活動欄位保存 mapping 遺漏。

必要新增 migration 從 008 或接手時最新序號接續；禁止改寫已套用 001–007。先備份並驗證，再用 migration role 套用，app role 僅取得必要權限。提供遷移前後筆數與既有名單/額度/去重鍵不變證據。

## 7. 現有 Chrome 連接器的實作要求

沿用 manifest 的限定網域與使用者選定的原站分頁，實作版本化固定命令：
- session/企業辨識。
- 讀取搜尋表單 schema/catalog。
- 套用已驗證 sourceFilters。
- 回讀原站當前已選條件。
- 取得搜尋符合人數（若原站提供）。
- 提交搜尋、讀結果頁、下一頁/游標。
- 僅從本次已觀察結果選定候選人明細。
- 讀取該帳號可見聯絡權限與完整電話。
- 來源額度觀察，支援「來源未提供」與「尚未觀察」的不同狀態。

命令實際名稱可依現有實作統一，但必須具備 strict schema、request/operation ID、protocolVersion、capabilities、scope、criteria hash、短效單次票券與結果大小限制。
- 不提供任意 URL、任意 JavaScript、任意 selector 或通用 eval 的遠端執行入口。
- 候選人 ID/導覽連結必須來自該 run 合法看到的結果；後端固定 allowlist 驗證 route/origin，禁止猜 ID、掃描明細或任意導向。
- token/ticket/epoch 過期只能阻止新來源動作，不能刪掉業務名單或額度。健康檢查不能每次清掉正在執行的搜尋。
- 導覽後重新注入/綁定，處理 SPA、內容未穩定、虛擬清單、MV3 service worker 重啟與 tab 關閉。
- 使用者在原站改條件/換企業/導覽別頁時先停 source dispatch，保留可恢復游標；不能搶回分頁覆蓋真人輸入。
- 104 登入已有效但本機 bridge 未連接，要顯示「連接目前 Chrome」，不能寫成帳密錯誤或已登出。
- extension 更新後檢查實際載入版本和能力，不能只更新磁碟檔就聲稱已生效。可沿既有權限更新，避免新增加 cookies/debugger/all-sites 權限。
- 透過產品一次連接/開始後自動執行後續步驟；不得要求使用者每讀一頁都點插件或在 debug 重複確認。
- 不使用另一個內部瀏覽器的 session 冒充此 Chrome 連接器已完成驗收。
- 瀏覽器控制應遵守當前可用工具/技能的規則；使用者接管或停止控制時立即停止該 UI 操作，保留狀態並改做獨立本機工作，不能繞過接管。

## 8. 真正向 104 搜尋的執行管線

1. 驗證已綁企業、目前登入與連接器版本，取得搜尋表單與適用選項。
2. 使用者以原生對應控制項建立方案；顯示清楚條件摘要。
3. 開始前以持久 run 鎖與 criteriaVersion 固定方案；清除原站沒有在本次方案表示的殘留條件。
4. 在官方 listSearch 表單套用值，回讀每個已支援欄位；完整比較預期/實際 canonical filters。
5. 一致後才透過原站正常按鈕提交，依真正導覽到的結果讀取；不得憑空拼查詢參數或猜搜尋 API。
6. 結果摘要先讀穩定 ID 與可見工作相關欄位，先排除歷史採用與 DNC，再決定必要明細。
7. 結果來源原生條件已完成的部分不以模糊本機字串重新解釋；額外本機條件需獨立列出理由。
8. 依預算、電話權限與目標讀取必要候選人/電話；不為了搜尋先掃讀全部企業職缺。
9. 正常分頁要有已看頁/候選人鍵、排序/criteria hash、游標與結果變動偵測，防止重複頁與無限循環。
10. 達到今日 N、額度用完、來源結果結束、權限不足、資料格式改變、使用者暫停/取消或登入失效時，持久記錄停止原因與可恢復位置。

「搜尋 104」和「重新篩選已取得資料」是兩個明確按鈕。前者必須走真實來源；後者只檢查快取。來源符合人數、本頁結果數、本機快取人數、已查看、可用電話與採用筆數各自呈現，不能互換。

清單搜尋或符合人數計算是否耗額度仍需觀察；不知道成本時不能將預覽包裝成免費來源搜尋。

## 9. worker、重啟與每日執行

- 實作真正持久 worker；可用既有 PostgreSQL lease/鎖與同一 Node 服務，無須新增 Redis/BullMQ。
- 狀態涵蓋 queued/running/paused/waiting_login/waiting_connection/quota_exhausted/completed/partial/failed/cancelled；UI 用業務文字顯示。
- 開始、繼續、暫停、停止均版本化且冪等；兩個分頁/兩個程序同一來源 scope 同時最多一個來源動作。
- 來源 action 派送前先持久記錄 DISPATCHING；逾時、crash 或回應遺失視為可能已執行，不以新 key 自動再發一次。
- 重啟恢復尚未發出的工作；不盲目重派已可能耗點的動作。中途回應仍須正確入帳，再停止後續動作。
- 來源拒絕/限流/異常頁採有限退避並暫停；不得換帳號、開新 profile 或換企業繞過。
- 使用者要每日自動執行時使用已存在 scheduler 設定，實作並驗證後才能啟用 UI；以 Asia/Taipei + profile/day 唯一性防重。
- 若目前 Chrome 關閉或未登入，每日任務顯示等待瀏覽器；待同日連接恢復依 policy 繼續，不假稱瀏覽器關閉仍可背景取得來源。
- 不以自動排程為由安裝 OS 服務/排程器或在來源操作時突然打開專用 Chrome。

## 10. 額度：沿用帳本並補齊來源語意

- 查看上限預設 300，只是可調整本機保護上限；不同方案可提高或降低。移除 UI/schema 中將所有方案/目標硬編碼最多 300 的不當限制，改從有效 policy 推導。
- 查詢清單、符合人數、翻頁、候選人明細、電話揭露可能有不同成本；透過合法可見資訊觀察並分類，不因「沒有成功解析」就算免費。
- quotaInspect 應能回報 AVAILABLE/NOT_PROVIDED/UNKNOWN 及成本信心。來源沒有顯示剩餘數不是連接器永遠未實作；要實作缺值語意並依可靠成本上界及本機上限決策。
- 可以核實成本上界時先保守預留；上界也無法可靠界定時只停止該來源動作，顯示實際缺口，不硬填 cost=0 或假裝精確扣點。
- 來源有剩餘額度時使用較低可用值；以 observation watermark 避免已包含的事件又扣一次，並保守處理使用者手動/其他工具消耗。
- 保留 RESERVED → DISPATCHING → CONFIRMED/CHARGED_UNKNOWN；只有能證明未派送/不耗點才釋放。
- 不確定已發出的操作不自動重試。相同操作/相同結果重送只能記一次本機帳，不承諾外部 exactly-once。
- 改方案、切搜尋方案、換分頁、重連/重啟都不能歸零；降低上限低於已用量時剩餘=0。
- 本機業務日 Asia/Taipei，來源重置視窗若不同再同時約束；跨午夜未派送預留重新檢查，已派送歸原操作並對帳不確定跨日費用。
- 讀取 X 筆和採用 N 筆不同；N 可以因快取節省查看而不必硬限制等於當日來源查看上限。
- N 未設定不執行大批自動採用；N=0 不讀來源。設定頁仍可正常編輯選單、保存草稿與查看歷史。

## 11. 跨日去重、電話與快速應徵交付

定義 R_d=第 d 日來源操作，V_d=已查看人才，S_d=確定採用，D_d=已交付。S_d 加入時即建立永久排除，不等到電話接通或 CSV 下載。

- 第一天看 X 人、採用 N 人：第二天搜尋結果含這 N 個來源 ID，在打開明細前就排除；X-N 未採用者可用新鮮快取重新比對。
- 同人換電話仍按 sourceCandidateId 排除；不同 ID 同電話按台灣標準化電話 HMAC 排除；同人已知多號均加入排除鍵。
- 新 ID 的電話若在清單不可見，可能看完明細才知道與舊電話重複；此查看可能耗點，要如實記帳，電話仍不得進名單。
- 源 ID 帶企業可見/來源限制時按已驗證 scope 建模；搜尋方案/企業職缺切換不能清除本工作區歷史。
- 已採用即使取消匯出或沒打電話，也不能自動放回新名單；重取同批不增加採用數。
- 電話只有完整、有效、來源權限允許、新鮮、未 DNC、未歷史採用才可交付；遮罩/需邀請/待開放/撤銷不算可用。
- 可直接 E-mail 及電話聯繫的原生篩選只能提高候選效率，不能代替每人電話可見證據。
- DNC 優先於排序、分數、快取、今日名單與既有批次；匯出時重新檢查。
- 候選人與電話沿既有加密、AAD 與隔離儲存；日誌/測試輸出不出現完整電話或履歷。刪除資料仍保留必要不可逆排除鍵，金鑰遺失不得生成新 key 假裝沒有歷史。
- copy/TXT/CSV/TSV 沿用已實作交付批次、idempotency、manifest hash；保留電話開頭 0 與 +886 格式，正確轉義並防止姓名/方案欄位的試算表公式注入。
- 無職缺時匯出搜尋方案欄，職缺欄為空；不能因 jobTitle 缺值使匯出失敗。
- 在 F:\AI\_104Recruit\docs\recruiting\QUICK-APPLY-CONTRACT.md 更新欄位/編碼/分隔/重取規則。沒有接收端範例前只宣稱通用可複製電話格式；特定快速應徵匯入相容性獨立標示，不能阻擋完成其他流程。
- 不把匯出事件寫成「已撥打」，也不自動發起電話。

## 12. 頁面交付規格

主頁順序：
1. 人才搜尋方案：選擇/新增方案與名稱，不必選企業職缺。
2. 原站基本條件：關鍵字、希望職類、希望工作地、居住地、最近活動日。
3. 更多查詢條件：按原站工作相關分組展開；自訂本機額外規則另放進階收合區並預設不阻擋來源結果。
4. 已選條件摘要：每個 chip 可移除、整組清除、原站對應狀態與儲存狀態；無需使用者了解代碼。
5. 搜尋 104 / 重新篩選已取得資料：清楚區分，顯示目前額度與每日 N，提供開始、暫停、繼續與停止。
6. 原站人才結果：工作相關資訊、匹配依據、歷史排除、電話狀態、查看/採用狀態與分頁。
7. 今日名單、歷史使用、額度方案、複製與下載保留完整可操作。

互動：
- 地區/職類以原站對應的階層多選彈窗/選單；有搜尋找選項、已選數/上限、chips、取消/確認及清除；取消不得偷偷保存暫選。
- 列舉值用原生 select/radio/checkbox 或等價無障礙元件；下拉不得用逗號輸入當退路。
- 表單錯誤緊鄰欄位，提交錯誤有可聚焦摘要和定位；不能只吐一個不可定位 toast。
- 開啟彈窗焦點進入，Esc 關閉並還原焦點；方向鍵/Tab 可完成選擇，已選/失效狀態不只靠顏色。
- 選項載入中/失敗/過期各有具體狀態，不以空清單偽裝「沒選項」；保留已有草稿。
- 長地名/職類 chip 不壓壞布局；驗證訊息與人數更新不造成按鈕跳動；桌面與窄畫面均可操作。
- 每職缺最低/最多名額、權重、優先順序移出主流程；單一方案只顯示今日目標。多方案進階分配才顯示方案最低/上限/權重，總額度與歷史去重仍共用。
- 顯示「來源搜尋完成」不等於「已取得電話」；空結果、無可讀電話、剩餘額度不足、登入失效都有業務原因與能採取的下一動作。
- 不要求使用者打開 /recruiting/debug 或逐步跑工程檢查。

## 13. 實作順序與允許修改範圍

每項完成由執行者自行驗證後繼續；不為每個里程碑要求批准。

|任務|工作與驗收|主要檔案|
|---|---|---|
|T01|重新核對根目錄、WIP、health、DB角色、備份與來源表單；建立本輪實際基線|docs\planning、docs\source、既有讀取工具|
|T02|完成原站欄位/選項/階層/代碼/限制/清除/回讀矩陣及最小去識別 fixtures|docs\source\search-control-matrix.md、src\parsers、tests\fixtures|
|T03|嚴格 sourceFilters/catalog/localRules schema 與版本、保存往返|src\search.ts、新增必要 src\search-filters.ts、src\parsers|
|T04|搜尋方案與企業職缺解耦、舊資料相容遷移、不清歷史|src\daily-lists.ts、src\candidates.ts、src\delivery.ts、db\migrations\008 起|
|T05|版本化 Chrome 搜尋/回讀/分頁/候選人/電話命令，實際載入版本核對|chrome-extension、src\chrome-bridge.ts、src\domain.ts、src\adapter104.ts|
|T06|真正官方表單搜尋管線、條件精確回讀、結果與穩定 ID、額度語意|src\recruiting-jobs.ts、src\quota.ts、必要 src\search-worker.ts、src\parsers|
|T07|持久 worker、lease、暫停/繼續/停止、重啟恢復、排程與重試|src\recruiting-jobs.ts、src\db.ts、src\server.ts、必要新模組|
|T08|跨日/跨方案去重、電話權限、DNC、無職缺交付與格式相容|src\dedupe.ts、src\candidates.ts、src\daily-lists.ts、src\delivery.ts|
|T09|完整原站對應選單、搜尋主頁、名單與設定連動|public\recruiting.html/js/css、必要新增 public\search-controls.js/css|
|T10|單元、隔離DB整合、瀏覽器與錯誤恢復測試，修復失敗|tests、scripts\test-db.ts、test-runner.mjs、必要 package scripts|
|T11|真實 Chrome 原站條件往返、來源搜尋、可見電話與名單驗收|data\acceptance 私密目錄、docs\recruiting 脫敏報告|
|T12|建置/服務更新/實際頁面檢查/BAT 與最終報告|start-104recruit.bat、scripts、README.md、docs\planning、docs\recruiting|

可在 F:\AI\_104Recruit 內新增完成上述工作必需的模組、fixtures、樣式與測試。只更動直接必要範圍；不重寫無關 VoIP、AI 供應商、計費平台或引入新的前端框架。既有舊診斷維持可用，產品主頁不能依賴它的職缺步驟。

## 14. 必測矩陣

|案例|必須成立|
|---|---|
|完全沒有企業職缺 / 職缺清單權限失敗|仍可建立獨立搜尋方案、設定條件、搜尋與產生名單；沒有職缺 API 前置依賴|
|希望工作地與居住地設不同縣市|保存/回載/原站套用各自不混淆；符合判斷不掃整段文字猜地區|
|職類父子/多分支/已選上限|與原站相同規則；超限、重複與非法代碼在前後端都拒絕|
|活動日期、更新日期、年資 operator|只允許來源合法列舉；初始、清除、不拘、無經驗正確；不可靜默變值|
|keyword +、引號、空白與特殊字元|按已觀察原站文法往返；不自行改成 AND 或拼任意 URL|
|完整表單保存 round-trip|所有啟用欄位 reload 後不遺漏，包含以前漏掉的經驗與活動條件|
|catalog 過期、代碼移除、label 同名|不靠 label 猜新值，不落回不拘；提示重新選擇並禁止錯誤提交|
|原站殘留條件 / 真人修改分頁|回讀不一致即停，避免用另一組條件搜尋|
|搜尋 104 與本機重篩|只有前者觸發來源；成本依實況記錄；來源結果不是 fixture 或快取冒充|
|沒有 quota counter|分清 NOT_PROVIDED/UNKNOWN；可靠上界才能保守派送，不能免費化或永遠假報未實作|
|300方案改為其他上限 / 降到已用以下|不硬編碼300；帳本不重置；新的可用餘額正確|
|雙擊開始/雙分頁/雙 worker|同業務操作至多一份 run/帳本採用，scope來源動作串行|
|來源已發送但逾時/程序崩潰|不退還不確定成本，不重發付費動作；可解釋恢復狀態|
|分頁重複/游標失效/結果動態變動|不無限循環、不漏記重複處理，按可驗證狀態停止或恢復|
|第一天 X=100 採用 N=30；第二天來源含此30|按ID在明細前跳過；第一天未採用70可用新鮮快取|
|新ID同電話/同人換電話/同人多號|歷史 ID 或任一電話 HMAC 命中皆不新採用；已發生查看如實計費|
|換方案/重啟/跨日/切搜尋方案|歷史候選人與標準化電話交集為0；名單與帳本不刪除|
|遮罩/邀請/撤銷/DNC/權限過期|無法複製/匯出；本機分數不能覆蓋拒絕|
|重取同批 CSV/TXT、姓名公式字元|不重占名額、保留開頭0、正確轉義與批次指紋；不標示已撥打|
|舊資料升級與最小權限|既有採用/額度/去重/密鑰不變；遷移與app角色分離、不 blanket grant|
|鍵盤/Esc/取消/焦點/手機寬度|選單完整可操作，取消不落地，錯誤可定位|
|Chrome斷線/關閉/MV3重啟/企業切換|暫停來源、隔離企業、不丟業務狀態，不開專用 Chrome|
|排程同日重入/服務重啟|不重開同日任務；未登入顯示等待來源，沒有假背景成功|

測試規則：
- 合成 fixtures 只用於隔離測試 schema；不能把測試候選人/電話 seed 進主資料庫或展示為真實來源。
- 跑 npm run typecheck、npm run lint、npm run test:unit、npm run test:integration、npm run build 和 git diff --check；按本輪變更增加有意義回歸，不用大量鏡像測試湊數。
- 瀏覽器 E2E 至少用一個包含多個地區/職類、活動日期/年資/聯絡選項的組合，完整保存→回載→原站回讀，並測清除/取消/上限/失效代碼。
- 真實來源驗收先1筆，必要時最多5筆，全部納入正式來源帳本；使用者已保存的條件/N 才是業務依據，不能自訂任意地區/職類與採用數。
- 沒有真實條件或 N 時仍完成全部無依賴開發與來源表單往返；最後只集中索取缺少的業務設定，不捏造人才、不以 synthetic PASS 冒充真實電話驗收。
- 私密真實驗收證據保存在 data\acceptance；公開報告只寫檢查狀態、數量、hash、版本與脫敏資訊，不放真人姓名、電話、履歷連結或帳號識別。

## 15. 完成定義與最終交付

必須同時符合：
A. 使用者能完全不選企業職缺就建立搜尋方案，按原站選單選地區/職類與固定條件。
B. 每個已啟用來源欄位完成代碼驗證與保存/回讀對應；coverage matrix 如實列出支援範圍及未知。
C. 「搜尋104」確實在官方 listSearch 送出本次條件並取得結果，無需使用者在 debug 點步驟。
D. 真正實作候選人/可見電話讀取與 worker；符合條件、有權限且未重複的電話才進今日名單。
E. 300預設可依方案調整，併發/重試/跨日/重啟正確，成本未知不被當成0。
F. 歷史採用 ID/電話跨日、跨搜尋方案永久排除，DNC/權限再檢查生效。
G. 今日名單可複製/下載，無企業職缺仍可輸出；快速應徵特定接收端相容性如實標示。
H. 主頁、例外狀態、排程、歷史與啟動器有實際可操作證據；不是只看程式或 HTTP200。
I. 測試、建置、執行中版本、實際主頁與載入中的extension版本一致；新 migration 已備份後正確套用。

不得以「本機頁面 ready，但人才來源未接線」聲稱完成本任務。若來源真正拒絕/缺少權限/等待使用者業務值，完成其他施工後精確標明真實限制與未驗收項，不能填 PASS。

最終檔案：
- F:\AI\_104Recruit\docs\source\search-control-matrix.md
- 更新 F:\AI\_104Recruit\docs\source\candidate-search-discovery.md、contact-discovery.md、quota-discovery.md
- F:\AI\_104Recruit\docs\recruiting\SEARCH-FIRST-ACCEPTANCE.md
- F:\AI\_104Recruit\data\acceptance\search-first-acceptance-<timestamp>.json（私密、脫敏摘要；不進Git）
- 更新 F:\AI\_104Recruit\docs\recruiting\BUILD-REPORT.md、FINAL-ACCEPTANCE.md、QUICK-APPLY-CONTRACT.md
- 更新 F:\AI\_104Recruit\docs\planning\EXECUTION-STATE.md、MASTER-PLAN.md、14-DECISION-LOG.md 與 README.md，標注本輪取代的職缺優先條款。

最後給使用者的內容保持簡潔：可用頁面 http://127.0.0.1:43104/recruiting、BAT完整路徑、實際完成流程、真實驗收結果與必要缺口。工程矩陣/帳本/測試細節放報告，不讓使用者接手施工。

END OF LUNA EXECUTION PLAN — SEARCH FIRST V3
