# LUNA 修復任務：Ycut 地址語意比對、分欄核對與 API key 永久儲存

推薦模型：LUNA，High thinking。
工作與交付根目錄：F:\yuct_api
任務性質：修復現有可執行服務，完成必要測試與指定地址的真實驗收；不是重新建置整套服務。
證據盤點日期：2026-10-26，Asia/Taipei。

## 1. 使用者最新要求與優先序

使用者實際測試另一筆地址又失敗，要求重新確認後修復：

新北市淡水區中正東路一段3巷14弄17號16樓之6

使用者截圖及資料庫的實際送出值是：

新北市淡水區中正東路一段3巷14弄17號16樓-6

同時使用者明確要求：本機 API key 按下儲存後必須持續保存，重新整理頁面、重開同一瀏覽器及本機服務重啟後都不應要求重貼，直到使用者主動清除或更換。這項新指示取代先前「只留在頁面記憶體」的設計。

目前服務已存在，包含前端、REST API、前端 Chrome 擴充套件橋接、PDF 下載、解析與私有資料庫。不要重新執行原始建置提示詞、改用專用 Playwright 登入瀏覽器、換專案路徑或改成新框架。

這次請實際修復程式及前端，不要只回傳分析、提示詞或建議，也不要只改錯誤訊息就宣稱地址問題已解決。

## 2. 已實際重新確認的故障證據

### 2.1 真實失敗紀錄

資料庫位置：

F:\yuct_api\data\private\service.sqlite3

已讀取的失敗 query_id：

aa893cbc-1a44-4dda-bafb-741138bac665

該筆紀錄：

- input_address：新北市淡水區中正東路一段3巷14弄17號16樓-6
- normalized_address：仍是新北市淡水區中正東路一段3巷14弄17號16樓-6
- status：not_found
- error_code：null
- result.candidates 恰有一筆候選。
- 候選 address：新北市淡水區中正東路一段3巷14弄17號16樓之6
- 最後更新：2026-10-26T07:27:59+00:00，即 Asia/Taipei 15:27:59。
- 此筆停在搜尋候選比對，尚未到建物明細、既有 PDF 判斷或 PDF 下載。因此現有證據不能用來宣稱此建物沒有既有 PDF。

同一程式對兩個值的離線重現結果：

~~~json
{
  "input_suffix": "16樓-6",
  "candidate_suffix": "16樓之6",
  "exact_matches_result": false,
  "input_preview": {
    "city": "新北市",
    "district": "淡水區",
    "road": "中正東路",
    "section": "一段",
    "lane": "3巷",
    "alley": "14弄",
    "house_number": "17",
    "house_subnumber": null,
    "floor": null,
    "floor_subnumber": null,
    "unparsed": "16樓-6",
    "complete": false
  },
  "candidate_preview": {
    "floor": "16",
    "floor_subnumber": "6",
    "unparsed": null,
    "complete": true
  }
}
~~~

已確認的程式缺口：

1. normalize_address 目前只處理全半形、空白及以「之」表示的部分中文樓層數字。
2. 樓層尾端「樓-6」尚未作有語意範圍的正規化。
3. address_parts 的樓層正規式只接受「樓之6」，因此畫面樓層顯示空白，尾端留在 unparsed。
4. exact_matches 直接比較上述不同的 normalized 字串。
5. 前端把 not_found、ambiguous、no_existing_pdf、failed 共用一個「此地址目前沒有可下載的唯一既有謄本」fallback。這會把地址比對問題說成 PDF 不存在。
6. API key 目前只有頁面 JavaScript 記憶體變數，沒有前端持久儲存；按鈕仍叫「暫存並連線」。這不符合使用者最新要求。

### 2.2 已有可工作的回歸基準

已授權、曾由真實 REST 流程成功完成的地址：

新北市淡水區民權路19號22樓之1

成功 query_id：

2bbe6ce1-37c9-4751-86f3-65eb09e27547

另有使用者後續成功紀錄：

c024a7c1-1225-4248-b4f5-dfdeba5b349d

已驗證的第一筆成功紀錄：

- status：completed。
- 精確建物 ID：F_27_FE_1294_00952000。
- Ycut buildno：00952000。
- PDF 建號：00952-000。
- PDF 門牌：民權路１９號二十二樓之１。
- 原始 PDF 共 3 頁，233587 bytes，受保護下載端點 HTTP 200，下載 bytes 的 SHA256 與保存紀錄相同。
- Ycut 摘要：屋齡 17.8、竣工日期 2008-12-23、建物 33.94 坪、主建 14 坪、附屬 1.47 坪、共有 18.47 坪、用途一般事務所、22樓／共24樓、鋼筋混凝土造。
- 這是該戶的資料，不得把這些數字套用到本次中正東路地址。
- 前次離線測試為 23 passed，另有套件 deprecation warning；這不是本次新修復的驗收結果。
- reports\build-status.md 尚有「17 tests／未驗收／暫存 key」等過時記載，必須按本次實測更新。

## 3. 開始前先確認實際檔案與服務

先讀適用的 AGENTS.md，確認 F:、工作目錄、當前檔案與執行程序。

- 使用 F:\yuct_api，不要將產品寫進提示詞 repo。
- 目前曾觀察此目錄沒有 Git repo；請重新確認，不要直接假設仍無 Git。
- 若有 Git，先檢查 branch/status，保護所有既有 WIP。
- 不執行 reset、clean、stash、廣泛覆寫、廣泛 stage、commit、push 或 deploy。
- 不修改同步專案 sources/ 或其他 SaaS 專案。
- 不清除、替換資料庫、既有 query、PDF、parser revision、API key 或 Chrome 登入資料。
- 不把歷史 not_found 改成成功來製造證據；修復驗收必須建立新的 query，保留歷史紀錄。
- 可以短暫重啟本機服務讓程式生效；先確認無正在處理的 query，完成後恢復服務並回報。

主要檔案：

- F:\yuct_api\src\yuct_api\address.py
- F:\yuct_api\src\yuct_api\api.py
- F:\yuct_api\src\yuct_api\models.py
- F:\yuct_api\src\yuct_api\worker.py
- F:\yuct_api\src\yuct_api\store.py
- F:\yuct_api\src\yuct_api\ycut_client.py
- F:\yuct_api\src\yuct_api\web\index.html
- F:\yuct_api\chrome-extension\background.js
- F:\yuct_api\chrome-extension\options.js
- F:\yuct_api\chrome-extension\manifest.json
- F:\yuct_api\tests\test_address.py
- F:\yuct_api\tests\test_worker.py
- F:\yuct_api\tests\test_store_api.py
- F:\yuct_api\tests\test_ycut_safety.py
- F:\yuct_api\README.md
- F:\yuct_api\openapi.json
- F:\yuct_api\reports\build-status.md
- F:\yuct_api\scripts\start-service.ps1
- F:\yuct_api\scripts\stop-service.ps1
- F:\yuct_api\scripts\session-status.ps1
- F:\yuct_api\scripts\copy-api-key.ps1
- F:\yuct_api\scripts\export-openapi.ps1

使用專案 Python：

F:\yuct_api\.venv\Scripts\python.exe

需要由外部資料夾執行 Python 模組時，正確設定 F:\yuct_api\src 的搜尋路徑或使用現有腳本；不要使用缺少專案依賴的系統 Python。

本機服務：

http://127.0.0.1:18765/

## 4. 修復地址處理：用同一套語意規則貫穿全鏈

修復應從可重現的失敗測試開始，再完成實作。不要對某一個完整地址硬編碼。

### 4.1 保留原文，建立一致的 canonical 與分欄

至少清楚區分：

- 縣市、行政區。
- 街路名稱、路段。
- 巷、弄。
- 門牌號、門牌之號。
- 樓層、樓之號、地下／地上。
- 室號。
- 尚未可靠解析的剩餘文字。

原始輸入須保留。canonical 是比較表示，不得冒充網站原文。

對有明確語意的尾端樓之號，支援：

- 16樓之6。
- 16樓-6。
- 全半形數字。
- 常見 Unicode 連字號，限定在同一個明確樓層子號位置。
- 十六樓之六等合理中文數字表示。

「樓-6 → 樓之6」只能在明確樓層子號的語意位置轉換，不能全字串將 "-" 改成 "之"，不能誤改負樓層、地下樓層、路名或門牌。

同一路段的一段／1段等數字語意可以統一，但只轉換明確的數字段。不得把「三民路」改成「3民路」或改動其他街路專有名稱。

保留必要的子號與巷弄，不以去掉尾端、刪數字、substring、startsWith 或第一筆近似結果來增加命中率。

### 4.2 比對、預覽與 PDF 身分核對共用規則

檢查並修復下列所有路徑，不要只讓某一個 function 的單元測試通過：

1. POST /v1/addresses/preview。
2. POST /v1/queries 的 normalized_address。
3. Chrome 橋接送給 Ycut 的 address 與原始地址保存策略。
4. POST /v1/bridge/search 的候選決策。
5. worker 對搜尋結果與明細地址的再次核對。
6. adrlanddoorsId 是否等於唯一候選的 ID。
7. PDF 建號與門牌核對。

原文、canonical、網站原文及必要的 normalization changes／match reason 應可追溯。若新增回應欄位，保持既有呼叫端相容並更新 OpenAPI。

對本次地址，預期分欄：

~~~json
{
  "city": "新北市",
  "district": "淡水區",
  "road": "中正東路",
  "section": "一段",
  "lane": "3巷",
  "alley": "14弄",
  "house_number": "17",
  "house_subnumber": null,
  "floor": "16",
  "floor_subnumber": "6"
}
~~~

「16樓-6」及「16樓之6」應得到相同語意與同一唯一建物。若網站實際結果不唯一，回 ambiguous 並列出差異，不能自行選第一筆。

不完整或不確定的解析應保留 unparsed、說明哪個欄位需要修正。不得丟棄未辨識尾端後繼續當精確地址下載。

### 4.3 必須保留的負例

至少確認下列不同物件永遠不能因正規化而錯配：

- 17號16樓、17號16樓之6、17號16樓之7。
- 17之6號16樓與17號16樓之6。
- 17-6號16樓與17號16樓-6，不可混淆門牌子號與樓之號。
- 同路一段與二段。
- 同路3巷與4巷、14弄與15弄。
- 不同行政區的同名路及同號。
- 地下16樓與地上16樓。
- 有室號與沒有室號。
- 「16樓6室」不得在沒有網站語意證據時直接當作「16樓之6」。

PDF 地址省略縣市時，可沿用現有的受限核對方式：先確認建號一致，再比對街路、門牌及樓層完整尾段。不得為了通過新地址而放寬為任意短字串後綴。

## 5. 修復前端分欄與錯誤訊息

在現有頁面做必要修改，不要改成 React／Next.js 或新增無關依賴。

- 完整地址可以一次貼上。
- 分欄清楚展示縣市、行政區、路段、巷、弄、門牌、門牌子號、樓層及樓之號。
- 必要時可修正分欄並更新完整地址；正反向更新使用同一個契約，不產生隱藏的第二套地址。
- 不能讓「16樓-6」留在 unparsed 卻只顯示空白樓層而讓使用者猜原因。
- 顯示輸入與匹配到的網站地址，說明有進行哪個有語意的等價轉換。
- 找不到匹配時，顯示實際候選及可理解的分欄差異；不要把候選當成已匹配。
- 不猜 Ycut selector、不要依卡片座標／DOM 排序填數值。
- 若提供縣市／行政區下拉，資料必須取自已確認來源或完整、可追溯資料集；不要只硬編碼本次兩個地址的選項。

狀態訊息必須對應真正階段：

| status / error | 顯示意思 |
| --- | --- |
| not_found | 沒有找到語意完整相符的地址，尚未判斷是否有 PDF |
| ambiguous | 多個匹配結果或語意不明，需選擇／修正 |
| no_existing_pdf | 地址及明細已匹配，但該建物沒有可用的既有 PDF |
| requires_login | Chrome 登入／驗證尚未完成 |
| document_tls_failed | 文件連線憑證驗證失敗 |
| document_download_timeout | 文件下載逾時 |
| 其他下載或解析錯誤 | 明確顯示失敗階段，保留已取得的摘要與原始 PDF |

移除把上述狀態都說成「沒有可下載的唯一既有謄本」的通用 fallback。

## 6. API key 永久保存：使用者已明確授權

本機前端的 API key 需要在目前 origin 下持久儲存，而不是僅頁面記憶體或 sessionStorage。

可採 localStorage 或等效的持久瀏覽器儲存；優先最小一致實作。使用者已明確要求此行為，不需再詢問是否要儲存。

最低行為：

1. 按鈕改成「儲存並連線」。
2. 儲存後在同一 origin 重新整理、新分頁及之後重開瀏覽器可自動讀取並連線。
3. 不設自動失效時間；只在使用者主動清除、更換，或瀏覽器本身清除網站資料時移除。
4. 本機服務短暫停止、重新啟動、暫時網路錯誤或 Ycut session 過期不能抹掉已保存 API key。
5. API key 真正回401時標示需更新；不要把瞬斷當成 key 無效，也不要默默刪掉使用者要求永久儲存的值。
6. 「清除已儲存 key」明確移除持久儲存及當前記憶體值，回到未設定狀態。
7. 顯示目前是否已有儲存，不顯示完整 key；正常連線後輸入框可清空或遮蔽。
8. Key 不放在 URL／hash／query string、HTML 原始碼、一般 log、console、完整 JSON 回應、截圖或公開測試證據。
9. 保留 Windows DPAPI 保護的主 key 檔：F:\yuct_api\data\private\api-key.dpapi。不因前端持久化而重新產生或輪替 key。
10. Chrome 橋接擴充套件目前自己的儲存設定要保留，不要求使用者重裝或重貼設定來掩蓋前端問題。
11. 只儲存本機服務 API key；不新增 Ycut 帳號密碼、Cookie、Authorization 或 OTP 的擷取／儲存。
12. 更新 UI 說明與 README，移除「重新整理就清除」「僅記憶體暫存」等過時文案。
13. 指出瀏覽器 origin 改變（例如127.0.0.1改為localhost或換port）會是不同網站儲存範圍，不假稱跨 origin 共用。

持久瀏覽器儲存有本機裝置與同源頁面存取的風險，依本次明確需求實作即可；不把資料送到雲端、不擴成帳密管理產品。

## 7. 保留現有成功流程與資料來源界線

現在資料流：

本機前端 → FastAPI REST／SQLite queue → 已安裝 Chrome 橋接
→ 使用者目前登入的 Ycut 頁面呼叫搜尋／明細
→ 後端下載既有 PDF → 私有保存 → 本機解析／OCR。

已觀察的 Ycut：

- 搜尋：GET /api/buildingDesc/list?address=...&pageIndex=...&pageSize=...
- 搜尋清單 data.result[]，總坪數欄位為 totalPin。
- 明細：GET /api/buildingDesc/detail/{adrlanddoorsId}。
- 明細的總坪數為 totalPing，另有 masterPing、outbuildingPing、commonPing、buildAge、buildDate、usename、material、fromFloor、toFloor、totalFloor。
- 既有 PDF 來源是精確匹配明細的 transcriptList[]。
- PDF 下載只允許既有白名單 docs.evertrust.com.tw，保留每次轉址檢查、大小／MIME／PDF簽頭／頁數／SHA256 驗證。
- 目前已使用 truststore==0.10.5 及 Windows 系統憑證驗證解決先前 Missing Subject Key Identifier 問題；保持 HTTPS、憑證與 hostname 驗證。不得改成 verify=False、CERT_NONE 或接受任意主機。
- 搜尋及明細讀取發生在目前已登入的前端 Chrome，不是後端重放日常Chrome Cookie。
- CAPTCHA／OTP／2FA 由使用者人工處理，不繞過，不反覆要求提供帳密。
- 不觸發「立即申請」或付費新謄本。

維持：

- building.ycut_summary 與 field_sources 的逐欄追溯。
- API 原始欄位及既有呼叫契約。
- 原始 PDF 與 parser revision。
- F:\yuct_api\data\private\site-details 的私有來源快照。
- 網站總坪數與 PDF 原表面積分開。PDF total_area_sqm 是主建物原表總面積，不是含共有部分的網站總建坪。
- 下載／解析失敗仍保留已取得的建物摘要及保存成功的 PDF。
- 原始識別碼與建號的前導零。
- 只綁本機127.0.0.1；本次不部署VPS、不開外網、不擴充多租戶。

## 8. 必須執行的測試與驗收

### A. 地址測試矩陣

使用可讀的 table-driven 或其他行為測試，同時涵蓋正規化、分欄與候選決策，不要只測一個 helper。

至少涵蓋：

- 本次完整地址的「16樓-6」與「16樓之6」。
- 中文樓層／樓之號、全半形數字、明確樓之位置的常見連字號。
- 一段／1段等明確數字段表示法。
- 含路段、巷、弄、門牌、樓之號的完整組合。
- 同一輸入經 preview、create_query、bridge search、worker detail、PDF identity 的語意一致。
- 第4.3節的所有負例及零筆／多筆匹配。
- 不得硬編碼本次完整地址、候選ID、樓層或結果數字。

### B. API key 行為測試

以假的 key 測試，不輸出真 key：

- 儲存並連線後讀得到持久值。
- 頁面重新載入後自動恢復。
- 新分頁同 origin 能恢復。
- 網路斷線／503／服務重啟／requires_login 不清除。
- 401 顯示需更新，但不因一般錯誤清掉保存值。
- 主動清除才移除。
- 不在URL、一般log、完整JSON、測試快照出現 key。
- 儲存不可用或quota/security例外時有清楚訊息，不假稱已永久儲存。

盡量在現有使用者 Chrome 做本機 UI 的 reload／新分頁確認，不關閉使用者所有Chrome視窗。若無法實際重開整個使用者瀏覽器，誠實標示這一個動作未實測，並提供可操作驗證步驟；不可拿不相干的全新profile冒充使用者現有Chrome驗收。

### C. 真實地址端到端

使用已授權的這兩筆地址，避免擴成大量或隨機調閱：

1. 新北市淡水區中正東路一段3巷14弄17號16樓之6
2. 相同地址的16樓-6表示法
3. 視需要回歸：新北市淡水區民權路19號22樓之1

每次使用新的冪等鍵與 query，不以舊not_found或舊成功紀錄作新修復結果。

本次地址兩個表示法必須命中同一個唯一建物ID；再以真實明細判斷有無既有PDF，不能預先假定一定有或沒有。

如果有PDF，完成：

前端送出 → API建任務 → Chrome查詢 → 唯一地址／明細ID
→ 下載原始PDF → SHA256／頁數 → 本機解析 → GET JSON → 受保護PDF下載。

如果實際沒有既有PDF，也須真實拿到唯一建物明細後才回 no_existing_pdf，建物摘要必須保留；這個真實結果與原本錯誤 not_found 必須分清楚。

真實畫面至少確認：

- 分欄中的16樓與樓之6。
- 狀態與候選／錯誤說明。
- 本機API key已儲存、重新整理後可自動連線。
- 查詢結果顯示實際建物資料。
- 有PDF時可下載，解析欄位來源、單位正確；無PDF時不出現一整排無解釋的空白欄位。

### D. 檢查與證據

- 執行與變更相稱的既有測試及新增行為測試。
- 檢查前端JavaScript語法及實際渲染。
- 匯出OpenAPI並確認與路由一致。
- 檢查本機服務重啟後健康與Chrome橋接。
- 更新 F:\yuct_api\reports\build-status.md，移除過時數字及尚未驗收描述，改為當次可證明的結果。
- 另提供一份去敏修復報告，建議 F:\yuct_api\reports\address-normalization-persistent-key-fix.md。
- 證據包含測試數量、真實query_id、status、地址匹配摘要、PDF是否存在、頁數／hash核對、保存key的實際驗證範圍。
- 不把健康檢查、mock、fixture或一筆舊成功取代本次完整驗收。

若工具對Chrome操作出現安全政策拒絕，遵從該拒絕，不改走CDP、其他瀏覽器表面或其他技術繞過。特別是先前 chrome://extensions 被URL政策封鎖，不需要也不應為這次修復重試安裝。只有確實被擋的UI操作才能標示限制，其他仍被允許的本機程式與API工作按工具規則完成；不能宣稱未看到的UI狀態已通過。

## 9. 交付與停止條件

完成後留下本機服務可用，提供修正後直接可開的前端網址及本次query結果入口。若為套用修復重啟服務，恢復原先本機埠18765。

以繁體中文簡短回報：

- 真正原因與實際修改。
- 變更檔案的完整Windows絕對路徑。
- 地址等價表示法的正例與不可混淆的負例結果。
- API key是否真的持久保存及清除方式。
- 兩個表示法的真實建物ID是否相同、實際有無PDF。
- 測試、API與UI驗證分別通過到哪裡。
- 還有實際阻擋時，精確說明範圍與必要的人工操作。

除非確實需要使用者完成CAPTCHA／OTP／2FA或工具權限阻擋，不要反覆要求確認一般修復、重貼已保存設定、重填已給的測試地址，或停在「可以繼續」。

不要只因民權路那筆仍成功就關閉任務；本次中正東路地址與永久儲存key都要有各自的實作及驗收證據。

<!-- END OF LUNA YCUT ADDRESS NORMALIZATION AND PERSISTENT KEY REPAIR PROMPT -->
