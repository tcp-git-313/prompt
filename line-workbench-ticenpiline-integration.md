# 任務：將 TicenpiLine 功能整合進獨立 LINE 訊息工作台

指定執行者：LUNA（gpt-6-luna），thinking level = max（LUNA MAX）。
環境：Windows／PowerShell。
這是實作任務。請完成可操作的產品、隔離測試及交付文件，不能只交計畫、資料包或未接通按鈕。

## 一、目標

以 F:\LINE訊息工作台 為唯一產品工作目錄，保留目前 Ticenpi 操作介面，將 E:\TicenpiLine 的必要程式「複製到新副本後整合」，完成：

1. 好友資料、好友 UID、好友標籤選取與管理。
2. 訊息範本與範本分組，保留新工作台的逐人變數代入與完整預覽。
3. 原始圖片附件、文字及逐張圖片發送。
4. 即時、指定日期時間、每日／每週週期排程。
5. 任務進度、停止、取消、失敗與結果不確定的處理及結果回寫。
6. 本產品自己的 LINE 登入環境、目前帳號識別與連線狀態。
7. 帳號隔離：好友、UID、標籤、排程與結果均屬於正確 LINE 帳號；來源若已有多帳號流程，不得移植後遺失隔離。

最終使用者只需啟動新工作台，開啟 http://127.0.0.1:19424/ 操作。
由新啟動器管理本產品背景引擎，不依賴舊 TicenpiLine 正在執行，不把呼叫舊 7588 服務當成獨立整合完成。
保留手動文字分享與離線匯出作為備用，但正式流程須實際接到本產品的發送引擎。

## 二、絕對限制

唯一允許新增或修改產品檔案的根目錄：F:\LINE訊息工作台。
E:\TicenpiLine、F:\Clone解析、F:\00-Ticenpi-SaaS、F:\HUB、其他獨立產品，以及 ChatGPT 專案 sources/ 全部只讀。

- 禁止修改、移動、刪除、執行來源專案；禁止啟停來源服務或關閉其 Chrome。
- 禁止讀取或複製來源 DB／SQLite／WAL、.env、憑證、私鑰（例如 .pem）、cookies、token、登入快照、Chrome profile、私人日誌或使用者附件。
- 可以靜態讀取原始碼、設定結構、文件、相依清單；只複製必要程式及合法資產，保留授權。
- 不整包複製來源。排除 .git、instance、data、logs、快取、__pycache__、舊 venv、舊打包快照與私人資料。必要 Chrome／擴充資產先確認來源、授權與是否含私人資料。
- 不 import 或執行 E:\TicenpiLine 的 Python；避免來源初始化或 __pycache__ 寫入。副本使用 -B／PYTHONDONTWRITEBYTECODE=1，import 只能解析到新副本。
- 不繼承可能指向舊資料庫的 DATABASE_URL、dotenv 或登入設定。DB、排程儲存、附件、probe、日誌、profile 等所有可寫路徑須先核對 resolved absolute path 位於新產品內。
- 不重置、清空或用測試覆蓋現有新工作台 DB；保護既有使用者資料及未提交修改。必要 schema migration 要有備份與失敗回復。
- 不接 SaaS 登入、計費、席次、雲端客戶資料、Google OAuth／雲端表單或其他產品導覽。
- 不部署、不推送產品程式、不建立 CI／Windows 工作排程器任務、不開公開服務、不啟用付費服務。
- 不自行對真人發訊；真人登入與試傳依第九節。
- 不把歷史測試、模擬 sender、HTTP 200 或本機畫面當成真人收到訊息。

## 三、先核對現況

必讀：
- F:\LINE訊息工作台\AGENTS.md
- F:\LINE訊息工作台\README.md
- F:\LINE訊息工作台\TECHNICAL-DIFFERENCES.md
- F:\LINE訊息工作台\驗證紀錄.md
- F:\LINE訊息工作台\adapter\README.md

下列是 2026-10-23 的只讀研究結果。開始時須確認目前程式是否已改變：

- 新工作台：Node 24、內建 HTTP、node:sqlite、原生 JavaScript UI；已有名單、CSV、範本、逐人渲染、附件、任務快照與回執。
- 現有三條入口必須分清：
  A. app/public/app.mjs：line.me/R/msg/text 官方手動文字分享。
  B. adapter/：Node/CDP 個人 LINE 自動化驅動，真實選擇器仍為 UNVERIFIED，未真人驗收。
  C. app/backend/ticenpi-line-adapter.mjs：TicenpiLine multipart 資料包，只匯出，未呼叫發送 API。
- TicenpiLine：Python Flask、SQLAlchemy／SQLite、APScheduler、React／TypeScript；Selenium 控制 LINE Chrome 擴充，觀察 TalkService/sendMessage XHR 回應。
- POST /send-message 使用 multipart：friends（好友 UID）、items、file_N、file_info_N、schedule_time、interval_time、task_name，以及 recurrence_type／recurrence_days／recurrence_time。
- friend_group_ids 是好友標籤群組，不是 LINE 聊天群組。個人 LINE 好友 UID 不可任意當成官方 Messaging API userId。
- 原 API success 可能只表示背景工作已啟動；客戶端接受也不代表對方收到或已讀。
- 上傳 parser 接受文件／影片，不代表 sender 能發送；已檢查即時 sender 對 file item 走圖片處理。不得照搬這項缺陷後宣稱完整附件支援。
- 歷史隔離測試有38項通過，是舊版本證據，不能充當本輪整合驗收。

優先只讀檢查：
- E:\TicenpiLine\app\services\send_message\（dto.py、api.py、service.py、utils.py、scheduler.py）
- E:\TicenpiLine\app\line_extension\（line_extension.py、messageBackground.py、ensure_chrome_for_testing.py、必要帳號與路徑模組）
- E:\TicenpiLine\app\utils\path_utils.py
- E:\TicenpiLine\app\core\current_account.py
- E:\TicenpiLine\app\config.py、requirements.txt
- E:\TicenpiLine\frontend\src\（頁面、API 與型別）

先盤點實際存在的好友、標籤、範本分組、帳號、排程與傳送模組；未查到的能力標 Unknown，不補造。

## 四、整合方式

沿用既有結構，完成最小可維護的整合，不重寫整個產品。
建議新增 engine/ticenpiline/ 放必要 Python 副本；如需 venv，只建在本產品 engine/venv/。
新 DB、排程、附件、帳號、profile、日誌全部在本產品 data/ 之下，彼此用途分明。
文件與來源 manifest 放 docs/；必要 fixture 放 tests/。

工作台 Node 代理至自己的 Python worker，建議 127.0.0.1:19429。
先核對埠是否空閒；其他程式占用就清楚報錯或採用文件化的新本機埠，不能終止它。
不得退回連舊 TicenpiLine 7588。

worker 要有 /api/environment 或同等身分端點，回傳角色、絕對根目錄、版本、模擬／真實模式。
Node 確認是本產品 worker 後才允許讀寫或發送。
worker 的寫入／發送端點需有內部授權並拒絕不可信跨來源請求，不能只靠 localhost 就允許其他網頁觸發發送；內部 token 不寫入公開前端或診斷回覆。
移植程式中的 DB、scheduler store、上傳／下載、帳號、probe、Chrome 路徑全部改到副本；先檢查，再啟動。

複製必要來源前後建立 SHA-256 manifest，結束時核對被複製來源是否變更。
若來源因外部程序變更，只記錄與停止依賴受影響檔案的步驟，不回寫來源。

以移植 TicenpiLine sender 為正式路徑。既有 Node/CDP 驅動保留為明確的未驗收備用，不能正式路徑失敗就自動切換，造成同一訊息重送。

原碼存在不等於目前 LINE 擴充相容。先核對可用環境與合法安裝方式；若版本失效或不可取得，完成其餘可測部分並提供證據、最小修復方案，不假造登入或發送。

## 五、功能要求

### 5.1 好友與標籤

- 從本產品的新登入環境取得真實好友資料，不讀舊 DB。
- 同步由使用者明確操作，顯示帳號、同步時間、錯誤與未登入狀態。
- 既有工作台聯絡人／CSV 可明確對應好友 UID；顯示名稱只供候選比對，同名不能自動猜。
- UID 必須屬於目前 LINE 帳號；未知好友、缺 UID、帳號不符或同名未釐清，停在準備階段。
- 好友標籤對應正確好友集合，建立任務時固定名單；後續改標籤不偷偷改已確認收件人。
- 示範好友與帳號清楚標記，不能被真實 sender 接受。

### 5.2 範本、文字與圖片

- 保留工作台變數與逐人預覽，補齊來源存在且屬本輪的範本／範本分組操作。
- 每人渲染後獨立建立訊息序列，不能把甲的稱呼送給乙。
- 保存原始圖片與 SHA-256，來源檔不覆寫；發送前核對內容、格式與存在性。
- 依預覽順序逐張送圖；文字與每張圖片都要有結果。
- 文件／影片若不支援，在任何部分按送出前即拒絕；不能先送文字再發現附件不支援。
- UI 只列實際支援格式，不把文件轉圖片後假稱已發送原始文件。

### 5.3 真正發送與結果回寫

必須接通：工作台 UI → 本產品 worker → 移植 sender → 逐項結果 → 工作台 DB。
下載 ZIP、輸出 request.json 或 mock HTTP 成功不算接通。

- 任務包含穩定 ID、帳號、收件人快照、內容摘要、worker 任務 ID 與逐項結果，重複點擊不能啟動兩次。
- 原 success 不直接映射成 delivered。至少區分：待準備、待執行、排程中、處理中、取消要求、已取消、客戶端接受、失敗、結果不確定。
- 按送出前失敗可標 failed；已按送出後逾時、程序中斷、圖片只送一部分則標 uncertain／partial，保留逐項證據，不自動重送。
- 發送前寫入持久日誌與防重複識別，重啟不得把不確定項目當成新工作。人工重試要清楚指出可能重複部分。
- 取消阻止尚未發出的工作；進行中顯示取消要求與實際停止點，不能宣稱收回已發出的訊息。
- 日誌不含完整登入 token／憑證，避免洩漏私人訊息。

### 5.4 排程

- 支援一次性、每日、每週，使用 Asia/Taipei。
- 每次執行有穩定執行 ID 與固定名單／內容，重啟或重複註冊不能重送。
- 排程建立、修改、取消與結果在新工作台操作。
- 未登入、帳號不符、worker 不可用時停等或報錯，不暗中換帳號／引擎。
- 開頁、啟動產品、安裝相依、測試不會建立或發出新訊息；只有使用者明確確認的發送／排程可進入真實執行。
- 不啟動兩個 scheduler 同時執行同一任務；原 Node 到期待處理狀態與 Python scheduler 權責需明確。

## 六、UI 與啟動

保持目前 Ticenpi 藍白灰，不加競品全站頂欄／側欄。
流程：確認帳號／好友 → 名單與標籤 → 文案與圖片 → 逐人預覽 → 即時／排程 → 進度與結果。
技術埠、JSON、資料包、worker 診斷放進階收合，正常操作使用清楚中文。

發送確認前顯示帳號、收件人／人數、完整文字、圖片順序、時間及方式。
未登入、缺 UID、不支援附件、worker 身分不符要直接說明原因。

啟動.cmd 保持單一入口，自動管理本產品 Node 與 Python worker。
背景服務／helper 使用隱藏視窗。使用者要操作的 LINE 登入視窗由使用者明確開啟。
同產品已啟動就沿用；異產品占埠報錯，不全域 taskkill。
停止只處理經 PID／命令列／根目錄核對屬於本產品的程序。
維持 loopback；不公開管理 API 或 Chrome 除錯埠。

## 七、必要驗證

全部使用本產品下的隔離暫存 DB／fixture，不讀寫現有使用者任務、不讀舊 DB。
模擬模式禁止真實網路發送，即使系統裝有或登入 LINE 也一樣。

必要檢查：
1. 錯誤根目錄、外部 DB 設定、錯誤 worker 身分、他人占埠均被拒絕。
2. Node → 實際 Python 子程序 → 移植服務 → 明確模擬 sender → 回報 → 工作台 DB。不能只 mock 掉 HTTP 後聲稱整合成功。
3. 兩位收件人得到不同稱呼與正確 UID；缺 UID、同名、跨帳號與標籤變更不誤送。
4. 原圖位元組、摘要、缺檔、不支援文件、第二張圖失敗、回應逾時與按送出後中斷。
5. 重複請求、重複啟動、雙 scheduler、重啟後不確定項目不被再送。
6. 一次性、每日／每週、台北日期、取消、未登入與 worker 失效。
7. 原工作台名單、預覽、草稿、附件、手動分享與匯出的必要回歸。
8. 單一啟動器不終止其他程式；既有產品服務不受影響。
9. 可用瀏覽器工具操作實際新版名單、預覽、排程及結果，保存截圖；程式測試不替代畫面驗收。

記錄命令、實際結果、失敗、修正、證據路徑及哪些是模擬。
不以舊38項測試數字代替本輪；不寫只重述實作、沒有實際風險價值的測試。

## 八、執行順序與兩次失敗規則

1. 讀文件與專案狀態，盤點來源功能及依賴。
2. 在 docs/TICENPILINE-FEATURE-MAP.md 列：功能、來源程式、目標位置、必需資料、目前狀態、驗收方式。
3. 寫最小計畫後繼續一般可逆實作，不交計畫就停。
4. 先完成副本、路徑隔離與 worker 身分，再串 UI／發送／排程。
5. 完成離線整合、畫面與啟動驗證，再交真人驗收步驟。

同一實作問題經兩次有針對性的修復仍失敗，立即停止該部分反覆嘗試，交回 SOL。
寫 docs/SOL-HANDOFF.md：重現步驟、兩次嘗試、錯誤、檔案、最小修復選項。
若無法切換模型，明確回報交接點，不假裝已交給另一模型；可繼續不依賴該問題的工作。
不得重寫整套、mock 掉失敗或切回舊服務來掩蓋問題。

## 九、真人驗收界線

要求使用者介入前，完成所有可離線完成的程式、UI、獨立啟動與驗證。
只在以下必要動作要求使用者：
- 在新產品自己的 LINE 官方畫面登入；AI／程式不代填或收集帳密。
- 指定本人或明確授權的單一測試對象，核對 UID 與內容。
- 確認此次真實試傳。未指定對象與確認時，不可對任何好友或標籤群組發送。

先準備一則短文字與一張無私人資料的圖片及收件核對步驟。
真人實測分開記錄：按送出、客戶端接受、對方實際看到，不互相替代。
未登入時，完成其他工作並交 WAITING_USER 的最少必要動作，不宣稱通用發送完成。
首次試傳不能用群發或週期排程。

## 十、本輪範圍邊界

- 不改成官方帳號 Messaging API，不申請 OA／憑證，不收集 LINE 密碼。
- Flex 電子名片屬另一產品「社區與 LINE 名片」的 LIFF 分享流程，不把 Flex JSON 塞文字／檔案端點。
- PDF／Word／Excel／影片本輪不宣稱支援；未有完整實作與實際驗收就明確拒絕。
- 不自動匯入舊好友 DB 或舊登入狀態。
- 不修改其他七個產品、共用擷取器、HUB、SaaS。

## 十一、必交付

更新：
- F:\LINE訊息工作台\README.md
- F:\LINE訊息工作台\TECHNICAL-DIFFERENCES.md
- F:\LINE訊息工作台\驗證紀錄.md
- F:\LINE訊息工作台\docs\TICENPILINE-FEATURE-MAP.md
- F:\LINE訊息工作台\docs\TICENPILINE-INTEGRATION.md
- F:\LINE訊息工作台\docs\TICENPILINE-SOURCE-MANIFEST.json
- 畫面證據；SOL-HANDOFF.md 僅在兩次修復失敗時建立。

最終繁體中文回報：
1. 新啟動入口及 http://127.0.0.1:19424/。
2. 功能｜實作狀態｜離線證據｜真人驗收狀態對照表。
3. 實際新增／修改檔案、獨立資料與登入環境位置。
4. 執行過的測試、證據路徑、來源檔案核對結果。
5. 已整合、尚缺、WAITING_USER、交回 SOL 分別說明；不得將待真人驗收寫成已完成通用發送。
6. 使用者接下來需要做的最少操作。

請從盤點與核對開始，接著持續實作至本輪範圍完成；缺真人登入或收件人時，先完成其他可做工作再交最後驗收動作。

<!-- END OF TASK: line-workbench-ticenpiline-integration -->
