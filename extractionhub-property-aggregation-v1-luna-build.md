# ExtractionHub 房源聚合 V1 — LUNA 精實建置任務

目標模型：LUNA；Thinking：High。
角色：實作者。依本文件 P0→P6 逐段完成可操作的本機版本，不只交計畫，不擴寫成全台大平台。
本次預設建置位置：現有 ExtractionHub 的新增 aggregation 模組；不是重做擷取核心。
本文件是建置規格，不是比房網後端原始碼，也不宣稱還原其私有演算法。

## 0. 完成定義與範圍

交付「一個行政區、兩個房仲來源」的本機房源聚合工作台，具備：
1. 官方清單的受限發現、已知物件 URL 批次收錄、標準化 JSON 匯入三種入口。
2. 重用 ExtractionHub 既有詳情擷取核心，持久保存來源刊登、欄位證據、價格版本與生命週期。
3. 篩選／排序／分頁、歸檔與不歸檔、同戶各站報價、價格及狀態歷史。
4. 保守同戶候選、人工合併／拆分／否決，決策重跑及重啟後仍有效。
5. 可見來源狀態、每次收錄結果、更新時間與停止原因；具備本機啟停與驗收說明。

初始來源固定 sinyi（信義）及 hbhousing（住商），交易類型只做中古屋買賣，行政區固定新北市蘆洲區。其他行政區與租案不在本次驗收範圍。不得因樣本少或來源被擋而擅自擴抓其他區或換成租案。

「兩站 live 成功」是目標而非已知前提：目前只確認詳情 adapter 存在，未驗證兩站現行清單、robots 或蘆洲覆蓋。來源受限時停止該來源，繼續完成其他合法來源與本機功能，最後判 PARTIAL，禁止拿 fixtures／比房網回傳冒充這兩站已接通。

禁止：部署、push、CI dispatch、建 image、對外綁定服務、修改現有資料庫、改 shared canonical/schema、修其他產品、升級來源成 SUPPORTED、蒐集屋主／電話／電傳、發送訊息。不要建立背景常駐排程。不要引入 Scrapy、Crawlee、Postgres、Redis、向量庫、LLM 去重或圖片辨識；現有 httpx/selectolax/Pydantic/SQLite 足以做這版。

## 1. 實際路徑、基線與保護

已確認本機：
- Repo：F:\00-Ticenpi-SaaS\ExtractionHub
- Remote：https://github.com/tcp-git-313/ticenpi-extractionhub.git
- 觀察時 branch：master；HEAD：711ba0a535417aeebeeb0e012588b21f235ddd0b
- Interpreter：優先檢查 F:\00-Ticenpi-SaaS\ExtractionHub\.venv\Scripts\python.exe，版本以實測為準。
- Repo 存在 dirty WIP，包含 canonical.py、core.py、多站 adapters、review_ladder、workbench 與 tests；另有未追蹤 carport.py、data/ 等。禁止 reset、clean、stash、覆蓋或整理這些工作。
- 本次只發布提示詞；尚未對本產品執行建置或測試。以上 HEAD 是快照，執行前重新核對。

先完整閱讀：
- F:\00-Ticenpi-SaaS\ExtractionHub\operations\HANDOFF-20261021.md
- F:\00-Ticenpi-SaaS\ExtractionHub\_charter\charter.md
- F:\00-Ticenpi-SaaS\ExtractionHub\pyproject.toml

只讀重用的實際程式：
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\core.py：run_extract(request, context=..., transport=...)。
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\models.py：ExtractRequest。
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\canonical.py：CanonicalListing，schema v2 的 price 單位為 TWD。
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\semantics.py：normalize_semantic。
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\transport.py：傳輸與 TransportContext。
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\review_ladder\policy.py：RobotsPolicy、HostLimiter、PolicyStop。
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\review_ladder\models.py：public_url 與停止原因。
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\adapters\sinyi.py
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\adapters\hbhousing.py

既有 store.py 是工作台／任務資料庫，不是這次聚合庫，不得呼叫它建立／寫入聚合資料。
既有 main.py、service.py、static/workbench.js 與共享輪子皆不改。

新功能留在相同 Python package，但用獨立 app 啟動，避免更動現行服務：
- 新入口：extractionhub.aggregation.app:app
- 綁定：127.0.0.1；由新服務選一個可用高位埠，避免與既有 8020／8000 及其他產品衝突。
- 不停止、重啟或借用使用者原本的服務；只管理本任務啟動且有 PID 紀錄的程序。

Allowed Files（以下是「本次新建」位置，不是聲稱已存在）：
- F:\00-Ticenpi-SaaS\ExtractionHub\src\extractionhub\aggregation\ 下的 Python 與 static 資源。
- F:\00-Ticenpi-SaaS\ExtractionHub\tests\aggregation\ 下的測試及已去識別 fixtures。
- F:\00-Ticenpi-SaaS\ExtractionHub\docs\aggregation\ 下的計畫、契約與驗收證據。
- F:\00-Ticenpi-SaaS\ExtractionHub\scripts\start-aggregation-local.ps1
- F:\00-Ticenpi-SaaS\ExtractionHub\scripts\stop-aggregation-local.ps1
- F:\00-Ticenpi-SaaS\ExtractionHub\data\aggregation-v1\ 僅限本任務新庫與去識別執行狀態。

不得改其他檔案。若存在同名新模組，先讀取、確認是否已有本任務工作，禁止蓋掉；只有真正需要改共享介面或超出 allowlist 才提出 CURRENT / EXPECTED / CONFLICT / MINIMUM_OPTIONS，並停止受影響部分。

## 2. 探索已知事實：規格依據與禁止推論

2026-10-25 使用者登入比房網後，正常 UI 已觀察到：
- POST https://www.ibigfun.com/api/search/list 回 data[]、total_records、current_page、per_page、args；每頁 20 筆。
- 相同蘆洲區條件，expand=0 歸檔 1,247 筆，expand=1 不歸檔 6,300 筆。
- GET https://api.ibigfun.com/on-market/o2o-same?ids=… 批次回跨站 source_id／link／total／add_date。
- 在售歷史與下架歷史分開載入；樣本分別 5 筆與 26 筆。
- 「相關物件」只是路名／型態／坪數區間再查詢，不能視為同一戶。
- 篩選 source_web=370 指 591、source_web=591 指 5168；回應 row.source=591 又代表 591。這些是不同命名空間，自己的模型使用文字 source_key，禁止抄數字做 join。
- update_time 與 add_time 相差約 15 分鐘，不能證明採集週期。
- 官方／同站多筆刊登可能報不同價格；同品牌也可能有多筆同戶刊登。
- 降價功能顯示一般會員無法使用，已停止。不得借 API 略過此限制。

這些介面僅作設計觀察，**不得建置比房網 crawler、重播其會員 API、使用登入 Cookie、將其資料作商用種子庫或把它當本系統上游**。自己的房源從允許的原始官網或明確授權匯入取得。

完整研究與證據只讀：
C:\Users\Tu\.codex\.chatgpt-projects\g-p-6ad327ac26d48191ae9a774c65889bc2\output\ibigfun-research-2026-10-25\ibigfun-aggregation-report.md
C:\Users\Tu\.codex\.chatgpt-projects\g-p-6ad327ac26d48191ae9a774c65889bc2\output\ibigfun-research-2026-10-25\02-api-evidence.json
C:\Users\Tu\.codex\.chatgpt-projects\g-p-6ad327ac26d48191ae9a774c65889bc2\output\ibigfun-research-2026-10-25\06-source-codes.json
若上述研究檔不存在，使用本節自包含事實繼續，不得因此假造 live 樣本。

開源參考：
https://github.com/shhhum/taipei-real-estate/tree/311bcd24ff9549562c43265666db7a5d19af9b9e
其 MIT 程式可參考 adapter 邊界；實際是商用租案，去重主要依網址，不能直接移植成買賣與同戶引擎。
不得執行 repo hooks 或整包安裝不相關程式。EasonLin0716/rent-crawlers 的 LICENSE 禁止商用；tw-rent-radar 所述 stealth 繞過手段不可使用。開源參考不是新增網路取用許可。

## 3. P0 — 讀取、對帳與固定計畫

1. 讀適用 AGENTS，記錄現行 HEAD／branch／git status、執行環境、既有服務占用埠、上述新路徑是否已存在。對 pre-existing dirty code 建 hash 清單，不讀 .env 或憑證，不把 data/ 全庫複製到證據。
2. 以目前 checkout 為讀取基線；不自行切 branch／worktree，不回退到舊 HEAD，也不丟失未追蹤相依檔。
3. 建 docs\aggregation\PLAN.md、CONTRACT.md、PREFLIGHT.json，寫死本文件 P0–P6、Allowed Files、來源／live 狀態與驗收矩陣。
4. 自行檢查後依計畫繼續 P1，日常實作選擇不需再向使用者問確認。舊 handoff 的車位／schema 任務不是本次任務，不得順便繼續。
5. 基線測試只在隔離臨時資料路徑執行；先確認測試不會寫既有 DB 或外連。若無安全條件，記錄 BASELINE_NOT_RUN，不以改寫舊測試方式湊綠燈。

## 4. P1 — 來源介面與受限發現

新增 sources.py，只做 sinyi、hbhousing，定義 discover(scope, cursor, budget) 與 extract_detail(url, budget)。詳情包裝既有 run_extract；不要複製現有解析器。

先從原始官網正常 UI 找到「買賣／新北市／蘆洲區」的真正清單 URL 與下一頁；檢查顯示條件、來源物件 ID 與當頁差異。不得套用商用租案 repo 的 URL 或猜 API／分頁格式。清單 adapter 只讀允許 HTML、頁面內嵌資料或已確認官方允許介面。

清單與詳情都必須先經同一來源的 robots／host allowlist／限速與 challenge 判斷。既有 RobotsPolicy 有 EH_REVIEW_IGNORE_ROBOTS 環境變數入口：本模組若發現該值為 1，直接拒絕 live job 並回 CONFIG_UNSAFE；不得啟用、依賴或修改該跳過機制。

預設來源均 disabled；P1 記錄允許的買賣清單、精確 origins、政策證據、界面與來源權利依據後，才能在本機明確開啟。robots 可讀／允許不等於已證明資料再利用授權；沒依據就保留 disabled，仍完成匯入／offline 流程。

每站每個手動 live job：最多清單 2 頁、詳情 10 筆、總計 20 次 HTTP（含 robots、redirect 與核心自動資產驗證；不得把子請求藏在預算外），並發 1，間隔至少 3 秒或來源更嚴格限制。所有網路請求經可注入計數 transport。若核心會另開連線而無法完整計數，先停該 live 路徑，不能假稱有完整 budget。初版不下載圖片。

401／403／CAPTCHA／challenge／登入牆／付費限制為 terminal；429 停本批，記 Retry-After；robots 取得失敗停止該來源。禁止換代理、指紋、UA、身分、session 或 transport 迴避拒絕。先驗證 origin、解析後主機與 public IP，redirect 每跳再驗，禁止 loopback、私網、雲端 metadata、任意使用者網址及憑證 URL。

未能確認清單時可支援「允許的已知詳情 URL 清單」；狀態必須顯示 DISCOVERY_UNVERIFIED，不可稱自動清單同步完成。依賴注入的 fixture transport 必須與 live 明確隔離，fixture job 不得觸發任何外網。

## 5. P2 — 聚合庫、正規化與生命週期

新增 models.py、repository.py、ingest.py。使用新 SQLite DB：
F:\00-Ticenpi-SaaS\ExtractionHub\data\aggregation-v1\aggregation.sqlite3
DB 路徑可由本模組獨立設定覆寫；測試用臨時目錄。禁止修改 store.DB、現有 extractionhub.sqlite3 或 review 庫。

最小資料表／同義模型：
- sources：source_key、adapter_version、enabled、policy evidence、live_status。
- listings：UUID、source_key、source_listing_id、canonical_url、transaction_type、current_version_id、last_seen_at、listing_status；唯一 (source_key, source_listing_id, transaction_type)。
- observations：job_id、listing_id、observed_at、fetch_status、content_hash、必要欄位與 provenance；同一 job 相同輸入冪等。
- versions／events：stable field hash、PRICE_CHANGED／FIRST_SEEN／REMOVED_CONFIRMED／REAPPEARED；同內容重跑不增版本／事件。
- entities、memberships：物件群組與多筆來源刊登的對應。
- match_decisions：merge／split／reject、涉及 listing IDs、理由、規則版本、建立時間；保存否決與拆分限制。
- jobs：scope、budget used、source、開始／結束、成功／失敗／停止原因、coverage_complete。

source_listing_id 永遠字串，保留底線、字母與前導零。總價用 TWD 整數或 Decimal 的精確字串，面積用 Decimal；禁止 binary float 金額計算。price_wan 只作顯示。地坪、建坪、主建、附屬、公設、車位面積分欄，不猜缺值。

沿用 canonical 的 UNKNOWN／NOT_PRESENT／PRESENT／EXTRACTION_FAILED，不重定義共享欄位；carport 的既有兼容預設 0 只有狀態／provenance 支持時才可解讀，不能把「未知車位價」當「免費車位」。原文與標準值並存，矛盾標記不互蓋。匯入時拒絕電話、Cookie、token、聯絡人等非本任務欄位。

刊登生命週期與採集健康分開：
- listing_status：UNKNOWN／ACTIVE／MISSING_PENDING／REMOVED_CONFIRMED。
- job/fetch status：OK／BLOCKED／RATE_LIMITED／PARSE_FAILED／ROBOTS_UNAVAILABLE 等。
被擋或解析失敗不覆蓋舊值、不清空同戶、不變成下架。兩頁有預算限制的掃描不是全站快照，coverage_complete=false，任何未見物件不得據此下架。只有明確來源下架證據才 REMOVED_CONFIRMED；404 也需判定確實物件移除，不能把來源故障頁誤當刪除。

## 6. P3 — 可解釋同戶候選與人工恢復

新增 matching.py。分清 duplicate listing 與 property matching：
- 同來源 ID 重跑 = 更新相同 listing。
- 不同來源／不同刊登可屬同物件，但保留各自 ID、報價、版本與原始連結。
- 同路段、同社區同格局、相近坪數或相近座標，僅能產生 REVIEW_CANDIDATE。
- V1 預設不自動合併跨刊登；人工確認後才歸成同戶。不要為了展示 6,300→1,247 而調閾值，該比房結果不是本系統 benchmark。
- 候選以已知縣市／區域為硬條件；有可靠社區或路名再縮小，UNKNOWN 不可相等比較。面積差、樓層差等給原值與理由，不輸出虛構機率。
- 地址正規化只做保守全半形／台臺／可確認段巷弄樓層處理；缺門牌或戶別不能補造。
- 明確不同門牌、戶號、樓層衝突要顯示警示；人工合併需明示確認且可恢復。
- merge、split、reject 以 transaction 寫入；同一 listing 同時僅一個群組；不依連通分量把 A像B、B像C 自動串成一群。
- split／reject 的限制持久保存，重跑候選與重啟後不自動合回；多次 merge／split 操作具冪等與版本衝突檢查。
- 群組摘要不創造「正確總價」；顯示各來源價格、區間與 observed_at。

## 7. P4 — 本機 API 與操作介面

新增 app.py、static/，使用既有 FastAPI，所有服務只在本機獨立入口。
最小 API 契約固定：
- GET /api/aggregation/v1/sources
- POST /api/aggregation/v1/jobs：mode=discover|urls|import；scope 固定 V1；回 job_id。
- GET /api/aggregation/v1/jobs/{job_id}
- GET /api/aggregation/v1/listings：grouped、source_key、keyword、min_price_twd、max_price_twd、status、page、page_size、sort。
- GET /api/aggregation/v1/entities/{entity_id}：所有來源刊登與各自價格。
- GET /api/aggregation/v1/listings/{listing_id}/history
- GET /api/aggregation/v1/candidates
- POST /api/aggregation/v1/decisions：merge|split|reject。
- GET /health：只證明本模組程序與 DB，不宣稱 live sources 成功。

JSON 匯入 payload 最多 200 records，欄位採 CONTRACT.md 白名單，單筆保留 source_key、source_listing_id、來源 URL、觀察時間與欄位證據。手動 URL job 每來源仍遵守 10 筆詳情與 20 次總網路 budget。所有 writes 有 idempotency key、輸入限制、明確錯誤碼；資料查詢不觸發外網。

分頁 page_size 預設 20、上限 100，非法值回 422；排序白名單與穩定 tie-break ID；群組數／刊登數分別回傳 total_entities／total_listings。grouped=true 在 server 先套來源／價格等篩選，再算群組與分頁；結果至少一筆匹配才出現。展開可顯示該群其他來源，但標示「不符合目前篩選」。禁止前端拿當頁資料自行算全庫筆數。

工作台路徑 /：
- 上方來源狀態、最後成功更新與「未 live 驗證」提示。
- URL／JSON 匯入與手動更新入口；live job 明示來源、行政區與預算。
- 歸檔開關、來源／價格／關鍵字篩選、分頁。
- 點開群組看來源價格表、來源連結與各筆歷史。
- 候選覆核可合併、拆分、否決，操作後立即可見且重新整理仍保留。
- 不顯示電話／屋主／調電傳；文字與來源 URL 全部安全轉義，防止上游 HTML／javascript URL 注入。
- 禁用寬鬆 CORS；限制 Host／Origin，本機寫入端點有同源保護，避免第三方網頁驅動本機批次抓取。測試與 UI 從同一 origin 操作。

視覺只需沿用既有工作台用語與色系、清楚可操作；不要重設品牌、加地圖、做整套產品改版或安裝設計服務。

## 8. P5 — 有意義驗收，明確分開 offline 與 live

至少完成以下測試，測試只接臨時 DB／fixture transport，禁止污染舊資料：
A. 同來源 ID、含前導零／底線的 ID，重跑無重複 listing 或事件；改價新增單一版本。
B. 同戶不同來源不同價格仍可人工歸檔；同品牌多筆不被字典鍵吃掉。
C. 同社區不同樓層／戶別只作候選；UNKNOWN 不會被當作相等。
D. split/reject 後重跑與重啟不合回；決策衝突回明確錯誤，不部分寫入。
E. BLOCKED、429、robots error、timeout、parse error、有限分頁缺席，不誤判下架、不消滅舊值。
F. 分頁每頁 ID、總數、穩定排序、歸檔 counts、來源＋價格聯合篩選、群組展開語意。
G. URL host／私網／redirect／超額 budget／跨站 Origin／XSS／SQL sort 注入均被拒；fixture 模式零外網。
H. 兩個來源各有清單解析 fixture 與詳情 canonical bridge 測試，驗證真正 source ID／單位與重複下一頁停止，不只 assert 自己造的常數。
I. 用可追溯且去識別的本地樣本確認 UI 流程：匯入→篩選→合併→比價→改價歷史→拆分→重新整理。
J. 開啟實際本機頁面完成上述 browser smoke，保留截圖、URL、job IDs；不能只拿 API 200 或前端靜態截圖宣稱互動成功。

若環境與來源允許，再做受限 live：
- 每來源至多 10 筆詳情與 2 頁清單，驗證來源/物件ID/買賣類型/行政區/價格/面積/樓層/來源連結；逐筆記預算與結果。
- 10 筆全數可在開發時重跑驗證，但不得稱其為全新未見集；本次不做來源 SUPPORTED 升級。
- 不足 10 筆就報實際 n，不補造；來源被擋立即結束該來源，不靠換策略完成數字。
- LIVE_NOT_RUN／LIVE_PARTIAL／LIVE_BLOCKED 與 OFFLINE_PASS 分開，不能宣稱全台、兩站全量或即時同步。

測試前檢查既有 tests 是否會寫 DB／外連；只跑安全的新測試與受影響既有測試，使用完整路徑或明確 tests 目錄，不在 repo root 遞迴收集 data/ 複本。既有 baseline failures 記錄為既有，不修無關程式。

## 9. P6 — 交付、核對與停止

在 docs\aggregation\ 建立：
- PLAN.md、CONTRACT.md：最終實際版本，清掉已放棄的設計。
- VALIDATION.md：P0–P6 結果、測試命令／exit、UI 截圖完整路徑、live/offline 分界與未完成項。
- SOURCE-STATUS.json：兩來源 discover/detail/policy/live 狀態、最後觀察時間與原因。
- HANDOFF.md：可直接複製的 PowerShell 啟動／停止／執行測試指令、完整 Windows 路徑、DB 位置、實際本機 URL、下次工作最小入口。
- PREFLIGHT.json 與 WIP-PRESERVATION.json：前後既有 dirty code hash 對比，確認 allowlist 外無本任務修改。

啟動器使用本機已存在的 Python／依賴，不要求全域安裝；缺套件只在本任務隔離環境處理並記錄，不改共用 venv 或全域環境。啟動後檢查健康與頁面，停止器僅停止本任務已確認的 PID，不能按 port 盲殺其他程序。

最後回覆繁體中文、精簡，固定列：
1. RESULT：PASS_LOCAL 或 PARTIAL；若 live 未驗證必須列明，不能只寫「完成」。
2. 完成能力與仍缺能力。
3. 實際可點本機 URL、啟動／停止／測試的一行 PowerShell 指令。
4. 新增／修改檔完整路徑、測試數字、來源狀態。
5. 前後 HEAD 與 WIP 保護結果；沒有 commit／push／部署。

執行到本機驗收與交付後停止，不自動進下一期。
END_OF_TASK — ExtractionHub Aggregation V1 / LUNA / Local Build Only
