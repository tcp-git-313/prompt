# 跨房仲案件收集平台 — LUNA 獨立建置任務 V1

推薦模型：LUNA；Thinking：High。
唯一建置根目錄：F:\跨房仲案件收集平台
角色：實作者。依 P0–P6 逐段完成可操作、可驗證的本機平台，不只寫方案。

## 1. 任務定位：收集案件，不是重做案件擷取工具

使用者要的主流程：
選來源／地區 → 掃描房仲官網案件清單 → 自動收錄案件 → 增量重掃 → 新案／價格異動／上下架追蹤 → 同案候選與跨站比價 → 案件庫查詢。

V1 核心是「清單發現與持續收錄」。不得把「貼一筆網址再擷取詳情」或「匯入假資料展示畫面」交成案件收集平台。

本專案完全獨立：
- 不建在 ExtractionHub，不 import／HTTP 呼叫／安裝 editable 指向 ExtractionHub，不引用其 venv、資料庫、schema 或執行程序。
- 不讀寫或修改 F:\00-Ticenpi-SaaS\ExtractionHub 及其他 Ticenpi 專案。
- 不把比房網當資料源；只借鑑其來源刊登、歸檔、比價、歷史的產品行為。
- 初版直接從清單取得必要欄位。深度詳情、完整相簿、電話、社區解譯、調電傳、DM／LINE 整合不在 V1。
- HTML 卡片／官方允許資料的解析屬本平台的來源 adapter，不代表與另一個「擷取資料」專案綁定。

V1：中古屋買賣、預設新北市蘆洲區、兩個來源、單機 Windows 本機使用。來源第一順位為信義 sinyi 與住商 hbhousing。先驗證這兩站實際清單；若受限，候補僅限永慶 yungching、台灣房屋 taiwanhouse，逐站記錄停止與替換理由。最多評估四站，不無限找替代站。

兩個實際可收集來源是驗收目標，尚未有 live 保證。只有一站通就報一站；全部受阻時仍完成離線核心但 RESULT=PARTIAL、AUTO_COLLECTION=BLOCKED。禁止用匯入／fixture 成功宣稱官網收集成功；禁止默默換區、換成租案或重用過期 API。

## 2. 固定技術選擇與精實邊界

- Python 3.11+，使用本專案 .venv；FastAPI + Uvicorn。
- httpx + selectolax + Pydantic；SQLite（Python stdlib sqlite3）持久化。
- 靜態 HTML/CSS/JavaScript，同一 origin，繁體中文介面，不需 Node build。
- 可選 Playwright 僅在允許的公開清單確需 JavaScript 渲染時新增；不使用 stealth、fingerprint、代理輪換或繞過模組。
- 不引入 Scrapy／Crawlee 與另一套採集框架，不用 Redis、Celery、Postgres、向量庫、LLM、圖片去重、雲端服務或付費 API。
- 排程為本程式內建的單一持久任務迴圈，不註冊 Windows 工作排程器／服務，不改系統開機項。
- 所有新增／修改、安裝依賴、資料與證據只能在 F:\跨房仲案件收集平台；只允許必要的 Python 官方／套件 registry 下載。

建議最小結構（都是本次待建的新檔，不是既有基線）：
F:\跨房仲案件收集平台\pyproject.toml
F:\跨房仲案件收集平台\requirements.lock.txt
F:\跨房仲案件收集平台\src\property_collector\app.py
F:\跨房仲案件收集平台\src\property_collector\models.py
F:\跨房仲案件收集平台\src\property_collector\repository.py
F:\跨房仲案件收集平台\src\property_collector\network.py
F:\跨房仲案件收集平台\src\property_collector\collector.py
F:\跨房仲案件收集平台\src\property_collector\matching.py
F:\跨房仲案件收集平台\src\property_collector\scheduler.py
F:\跨房仲案件收集平台\src\property_collector\sources\（base.py 與實際兩站）
F:\跨房仲案件收集平台\src\property_collector\static\
F:\跨房仲案件收集平台\config\sources.json
F:\跨房仲案件收集平台\data\collector.sqlite3
F:\跨房仲案件收集平台\tests\
F:\跨房仲案件收集平台\docs\
F:\跨房仲案件收集平台\scripts\
F:\跨房仲案件收集平台\啟動平台.bat
F:\跨房仲案件收集平台\停止平台.bat

可在上述目錄增必要 __init__.py、fixtures、migration、.gitignore，不做多層框架或先造未用抽象。不要讓一個檔案包下所有抓取／SQL／API／UI。

## 3. 探索依據：已知與未知

2026-10-25 比房網的正常登入操作已驗證：
- 列表由自己的後端查詢，20 筆／頁；蘆洲同條件歸檔 1,247 筆、不歸檔 6,300 筆。
- 主列表、跨站比價、在售／下架紀錄分開提供；一個來源可有多筆同戶刊登。
- 「相關物件」只是路名、型態、坪數區間搜尋，不等於同一戶。
- 不同來源可開不同價格；來源代碼在篩選／結果中甚至不是同一命名空間。
- update_time 的樣本差距無法證明採集頻率。
- 沒看見其真正採集程序、更新排程、精確同戶演算法或資料合作契約。

因此：本平台自行命名 source_key；自行保存 listing 與 entity；不照抄其私有端點、數字代碼、時間週期或宣稱相同正確率。不得取得／重播其會員 Cookie 或 API，不將探索資料匯入正式案件庫。

研究全文可選只讀：
C:\Users\Tu\.codex\.chatgpt-projects\g-p-6ad327ac26d48191ae9a774c65889bc2\output\ibigfun-research-2026-10-25\ibigfun-aggregation-report.md
這份舊研究後段曾提到 ExtractionHub；本任務以本文件的獨立專案定位為準。研究檔缺少仍可用本節繼續。

開源參考僅限概念或有明確授權的局部程式：
https://github.com/shhhum/taipei-real-estate/tree/311bcd24ff9549562c43265666db7a5d19af9b9e
該 repo 是台北商用租案、網址去重；不是現成買賣案件收集平台，不可直接套其租案 URL。若借程式需保留 MIT notice，禁止執行其 hooks 或整包引入 Airtable 流程。
HTTPX 官方文件：https://www.python-httpx.org/advanced/clients/
EasonLin0716/rent-crawlers 有非商用限制，不使用；tw-rent-radar 所述 stealth 手段不使用。

## 4. P0 — 目錄與環境核對，固定執行計畫

發布本文件前已查驗：F:\跨房仲案件收集平台 原先不存在；只為交付提示詞建立 docs，沒有產品程式、資料庫、測試或 live 成功基線。

1. 重查根目錄及適用 AGENTS；若有使用者新檔／既有 WIP，先讀、記錄、保護，不 reset／clean／stash 或覆蓋。
2. 新建專案自己的 .venv，檢查 Python 版本、依賴與中文路徑；不更動全域或其他專案環境。可 git init 本地空 repo，不 commit、push 或建立遠端 repo。
3. 寫 docs\PLAN.md（P0–P6、來源順序、Allowed Root、驗收矩陣）、docs\CONTRACT.md、docs\PREFLIGHT.json。自行核對後直接開始 P1，不逐階段要求使用者再次批准。
4. 若有真正權限或執行依賴阻礙，說明 CURRENT / EXPECTED / CONFLICT / MINIMUM_OPTIONS；只停止受影響部分，繼續獨立可完成部分。
5. 不為了對接其他產品而擴大範圍。上一版 extractionhub-property-aggregation-v1-luna-build.md 已被本文件取代，不得執行。

## 5. P1 — 真正的清單收集器（本次最重要工作）

先用正常瀏覽器查看官網買賣清單，選新北市蘆洲區，記錄真實 URL、有效篩選、來源物件 ID、下一頁行為、卡片欄位與取得方式。只觀察公開可訪問資料，不猜私有 API／分頁網址，不照搬租屋爬蟲的假設。

每站 adapter 契約：
- discover(scope, cursor, budget) → listings、next_cursor、reached_end、scope_verified、evidence。
- 每筆最少 source_key、source_listing_id、canonical_url、title、city、district、observed_at；清單可見則加總價、總坪、格局、樓層、型態與刊登日。
- 欄位有 value/state/provenance；未提供用 UNKNOWN，不能把空字串／0 當已知。
- 若沒有可靠來源 ID，可暫用來源已驗證 canonical URL 的穩定 hash 並標 identity_method=URL_HASH；不可從標題／價格造 ID。
- 詳情未讀不影響清單案件先入庫；enrichment_status=NOT_REQUESTED。
- 新頁要驗證與前頁 ID 差異；重複相同頁面停止為 PAGINATION_STALLED，不得無限翻頁。

同一個 network gateway 統一 HTTP timeout、大小、來源 origin allowlist、robots、限速、redirect、budget 與 typed stop。httpx 明設 follow_redirects=false、TLS verify=true、bounded pool、response streaming cap；每跳重新驗證。拒絕私網／loopback／metadata IP、憑證 URL、未確認網域，避免 SSRF／DNS rebinding；不能只做字串前綴判斷。

首次 live 驗證每站最多 2 頁、總網路請求 20 次（含 robots／redirect／瀏覽器子請求），並發 1、間隔至少 3 秒或來源更嚴格限制；不下載照片。JS 模式若不能攔截並完整計數／驗證子請求，標 BROWSER_COLLECTION_UNVERIFIED，不用不受控瀏覽器偷偷完成數量。

一般收集器支援可配置 max_pages／max_requests 與續掃 cursor；V1 預設 2 頁／20 請求，明示 PARTIAL_SCOPE，不代表已收全區。分頁／cursor 過期則記錄後重新開始去重，不能跳過無法證明已看過的頁面。

401／403、CAPTCHA、challenge、登入／付費牆立即停該來源；429 停本批並保存 Retry-After；robots 無法取得或禁止也停止。不得自動重試換代理、UA、身分、session 或 transport 以規避拒絕。正常公開頁面可用不等於資料再利用契約已取得，來源說明如實記錄可確認的使用限制與未確認事項，不自行宣稱已授權。

config\sources.json 只納入經記錄的精確 origins、買賣清單方式、地區對應與限速。未驗證來源 disabled，不能靠 supported=true 補齊兩站。預留 adapter 擴充介面即可，不一次寫二十家空殼。

## 6. P2 — 案件庫與增量更新

使用獨立 data\collector.sqlite3；測試必用臨時 DB。migration 可重複執行，交易與唯一鍵保護並發／重啟一致性。

至少建立：
- sources：站點能力、政策、最後成功／失敗、enabled。
- listings：listing_uuid、source_key、source_listing_id、canonical_url、transaction_type、最新欄位、first_seen_at、last_seen_at、source_published_at、status。
- observations/versions：job_id、observed_at、stable_content_hash、欄位原文／標準值／證據；同一次 job 重送冪等。
- events：FIRST_SEEN／PRICE_CHANGED／REMOVED_CONFIRMED／REAPPEARED；保留 before、after 與時間。
- jobs/checkpoints：scope、cursor、頁數／請求數、成功／新增／更新／停止原因、coverage_complete。
- entities/memberships/match_decisions：同案群與人工決策。
- schedules：規則、啟用狀態、下次執行、lease。

唯一鍵至少 (source_key, source_listing_id, transaction_type)。source_listing_id 保留字串／前導零／底線。相同 URL 先正規化已確認的追蹤參數，不能刪掉案件識別參數。

價格為 TWD 整數或精確 Decimal，坪數用 Decimal；API 精確值用字串或已定義精度，禁止 binary float 金額比較。買賣總價與月租不得混用；來源 UNKNOWN 價格不生成降價事件。只對真正在來源可見的欄位記變動，收集時間不能冒充刊登／來源更新時間。

同內容重掃只更新 last_seen 與觀察結果，不製造重複案件／版本／價格事件；新價格新增一個版本與事件。卡片欄位暫缺、擷取失敗不能覆蓋先前已知值為空，需保留觀察層衝突／缺值。

生命週期與採集健康分開：listing.status=ACTIVE／MISSING_PENDING／REMOVED_CONFIRMED／UNKNOWN；job.status=RUNNING／COMPLETED／PARTIAL／BLOCKED／FAILED／CANCELLED 等。
只掃兩頁或預算中斷不能把未見案件判成下架；完整有效同範圍掃描的缺席最多 MISSING_PENDING。REMOVED_CONFIRMED 需明確來源移除證據，404 也須排除故障／權限頁。BLOCKED 不清空案件、不產生下架事件。

## 7. P3 — 同案歸檔、跨站比價與決策恢復

V1 先保守可解釋，不用 AI：
- 重複刊登 ID 的去重由唯一鍵完成。
- 跨來源同戶只產生候選，預設不自動合併。已知同縣市／區，再用路名／社區、面積、格局、樓層形成候選及理由；缺值不可當相等。
- 同路、同社區、相近坪數不等於同一戶；不同門牌／戶號／樓層為衝突。保留原地址，不補造門牌。
- 人工 merge／split／reject 保存決策、理由、規則版本；合併仍保留同品牌的多筆刊登。
- 群組顯示每筆來源報價、原始連結、最後觀察時間及價格區間，不創造單一「正確價」。
- 人工拆分／否決後重掃與重啟仍有效，不能下次自動合回。交易化、冪等，遇過期群組版本回衝突，不部分寫入。
- 不用 A像B、B像C 的連通關係自動傳遞合併；實價成交資料與在售刊登分開，V1 不接實價庫。

## 8. P4 — 可操作的案件收集工作台

FastAPI 服務 127.0.0.1，預設 port 19431，啟動前查占用；若非本平台程序占用則選下一個空位並寫 runtime.json，禁止盲殺占用者。同一 origin 提供 UI/API，禁止寬鬆 CORS；限制 Host、寫入端點同源／本機操作 token，防其他網頁驅動本機爬取。token 不寫日誌。

主要畫面與驗收順序：
1. 收集任務：來源、行政區、手動「開始收集」、停止、續掃；顯示本批限制、來源狀態與收集進度。
2. 案件庫：新案、價格異動、來源、地區、總價、關鍵字、狀態、排序／分頁；可切歸檔／不歸檔。
3. 比價與歷史：展開來源刊登表、原站連結、價格／狀態事件及 observed_at。
4. 同案候選覆核：merge／split／reject。
5. 排程：明確啟用／暫停、間隔、下次時間、上次結果；預設停用。

JSON／CSV 匯入只作除錯／離線復原輔助，另頁標「匯入資料」，不得作首頁主要收集方式或預塞比房案件。測試資料採明顯虛構內容，只存在測試／demo DB；正式 DB 初始為空，畫面不能把 demo 算作 live 案件。

API 建議固定於 /api/v1：
GET sources、POST jobs、GET jobs/{id}、POST jobs/{id}/cancel、POST jobs/{id}/resume；
GET listings、GET listings/{id}/history、GET entities/{id}、GET candidates、POST decisions；
GET/PUT schedules、GET health。用 Pydantic 驗證，規格與實作對齊於 docs\CONTRACT.md。
GET 查詢不觸發外網；jobs 進背景工作並持久化，HTTP 立即回 job_id，不能卡住 API。取消只在安全分頁邊界停止並保存 cursor。

page_size 預設 20／上限 100，sort 白名單＋穩定 ID tie-break；server 先篩選再算群組／分頁，分別回 total_listings／total_entities。展開群組可顯示其他來源但標示不符合目前篩選。空值／未知不能當價格 0 通過條件。

上游標題／地址／URL 一律轉義，拒絕 javascript URL、SQL sort 注入與任意檔案讀寫。不收集／持久化聯絡人、電話、登入 Cookie、屋主資料。不要以繞過驗證或變換身分求資料量。

## 9. P5 — 單程式排程與可恢復工作

完成可測的內建排程功能，但交付時預設 disabled。使用者透過 UI 自行啟用；預設間隔 6 小時、最小 60 分鐘，網站更嚴格者優先。這是本平台初版設定，不是比房網或來源站的更新週期。
- 時間戳存 UTC，介面以 Asia/Taipei 顯示。
- 同來源同 scope 只允許一個工作，SQLite lease 避免重複啟動產生多個收集器。
- 服務重啟把失聯 RUNNING 記成 INTERRUPTED，不假裝成功、不重放一串錯過排程；保留可人工續掃的 checkpoint。
- 預算、每站冷卻與 terminal stop 同手動工作，不因排程重跑略過 blocked 狀態。
- 沒有二十四小時觀測就不宣稱已實測長期穩定；排程測試用可注入時鐘，不能真的等六小時。
- 不建立外部／雲端 automation，不註冊系統常駐服務。

## 10. P6 — 測試、實際啟動與交付

必驗矩陣：
A. 清單解析／有效地區：真正來源 fixture、來源 ID、價格單位、下一頁；假翻頁／重複頁停止。
B. 增量：同批／跨批重掃無重複刊登或事件；改價一次只一事件；缺值不抹除舊資料。
C. 完整性：預算截斷、只有兩頁、scope 不符、429、403、robots failure、timeout 不造成下架或清庫。
D. 群組：不同來源不同價仍可人工歸檔；同品牌多筆保留；同棟不同戶／未知欄位不誤合；split/reject 重啟後有效。
E. API：多條件篩選、穩定分頁、正確 counts、決策衝突、job 冪等／取消／續掃。
F. 排程：停用不執行、可注入時鐘到期、lease 互斥、重啟恢復與冷卻，不爆量補跑。
G. 安全：來源 allowlist、redirect、私網／DNS 驗證、budget 含子請求、XSS／Host／Origin／sort 注入；offline tests 零外網、臨時 DB。
H. 真實 browser smoke：啟動本機頁面，操作收集→案件庫→篩選→比價／歷史；人工覆核可用去識別測試 DB，但要分開標記。保留 screenshot、URL、job IDs 與可追溯查詢結果。

Live 驗收：每個實際採用來源最多 2 頁，遵守 P1 的總網路預算。記錄見到 n 個去重案件／新增 n／更新 n、清單證據、source ID、買賣類型與行政區。若蘆洲確實 0 案，據實報 0，另外用 fixture 驗解析；不得改區湊數。一次成功不能叫全量、全台、永久支援或真人驗收。

新建：
- 啟動平台.bat：不依賴目前目錄，透過專案 scripts 啟動隱藏背景程序並開正確本機 URL，支援中文／空白路徑。
- 停止平台.bat：只停本專案已驗證 PID、啟動時間與命令路徑，不按 port 名盲殺；清楚回報停止結果。
- scripts\start.ps1、scripts\stop.ps1、scripts\test.ps1；tests 統一臨時資料，不能動正式庫。
- docs\VALIDATION.md：命令、exit、測試數字、UI 截圖、live/offline 分界與缺口。
- docs\SOURCES.md＋SOURCE-STATUS.json：每站已查清單、方式、分頁、限制、替換／停止、最後結果。
- docs\HANDOFF.md：完整 Windows 路徑、DB、實際 URL、一行啟動／停止／測試命令、下一個最小任務。

結果只可寫：
- PASS_LOCAL_WITH_LIVE：本機功能與兩來源 live 依本範圍通過；不代表正式部署或全量覆蓋。
- PARTIAL：核心通過但 live／來源數／瀏覽器驗收／其他必要項不足，逐項列缺口。
- BLOCKED：無法建立可操作核心，附具體原因與已完成工作。

最後回覆繁體中文、精簡：RESULT、案件收集能力／缺口、完整專案路徑與可點本機 URL、兩站實際統計、測試與證據、一行啟動／停止命令。不得只回「已規劃」或「頁面完成」。

只完成本機 V1；不 commit／push／部署、不改其他產品、不進入深度詳情擷取 V2。
END_OF_TASK — Cross-Broker Property Collection Platform / F:\跨房仲案件收集平台 / Independent V1
