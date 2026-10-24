# LUNA 建置任務：Ycut 地址查詢、既有謄本 PDF 與 REST API

推薦執行模型：LUNA，thinking level：High。
任務性質：在本機建置並實測可操作的服務；不是僅提供設計、樣板或模擬畫面。
盤點基準日期：2026-10-25，Asia/Taipei。此日期代表前次觀察，不保證網站今天仍相同。

## 1. 使用者要的結果

使用者要讓其他程式輸入完整地址，服務即可：

1. 使用有權限的登入 session 查詢指定網站。
2. 精確找到地址對應的建物。
3. 取得「查看謄本」對應的既有 PDF。
4. 保存未修改的原始 PDF、雜湊、來源日期與取得紀錄。
5. 解析文字、表格及必要的影像欄位，輸出可追溯的結構化 JSON。
6. 提供 REST API 給其他程式使用，包含查詢、狀態、JSON、原始 PDF 下載及重新解析。
7. session 過期或遇到 CAPTCHA／OTP／2FA 時，交由使用者人工驗證，完成後保存並重用 session。

請持續完成本機 MVP、適當測試與真實範例驗收。不要停在架構文件、空端點、假資料或「後續可以實作」。

本次不包含正式環境部署、既有 SaaS 接線、公開網路服務、MCP、付費新謄本申請或大量批次調閱。

## 2. 先讀現有證據，不要重猜 selector 或假設流程

前次盤點資料的本機絕對路徑：

C:\Users\Tu\.codex\.chatgpt-projects\g-p-6ad327ac26d48191ae9a774c65889bc2\output\address-lookup

重要檔案：

- discovery-report.md：實站盤點、已知缺口與 REST 規劃。
- detail-response.redacted.json：真實明細 API 回應，文件 URL 已去敏。
- network-evidence.json：明細 API 的去敏 Network 證據。
- original\F_27_FE_1294_00911000-20230913.pdf：已下載原始 PDF。
- sample-result.json：前次範例解析，含明確缺漏；不是正式 API 回應契約。
- extracted-text.txt、extracted-tables.json：逐頁擷取與表格。
- pdf-page-1.png、pdf-page-2.png、detail-page.png：核對圖片。
- inspect_sample.py：單份範本的探針，不是正式 parser。
- openapi.json：草案，未部署；可沿用語意，但須補足正式 schema 與驗證。
- build_api_draft.py：草案產生器，不是後端服務。

上述檔案視為唯讀輸入，不要覆寫。sources/ 及任何同步專案素材也一律唯讀。
可以將必要樣本複製到新服務的私有測試資料目錄，須核對 SHA256；不要提交含個資的樣本、解析結果或截圖到公開 Git。

如果本機參考路徑不存在：
- 不要宣稱已讀取或重新創造同名「證據」。
- 本提示詞已提供足夠流程資訊，可先建置不依賴該檔的部分。
- 透過使用者已授權網站重新取得同一範例，或清楚列出尚缺的樣本。
- 缺少真人登入時，交付可登入的本機瀏覽器入口，讓使用者完成；不要要求把密碼貼進對話。

### 強制交付目錄

最終服務專案根目錄指定為：

`F:\yuct_api`

- 服務原始碼、README、啟動腳本、設定範例、OpenAPI、測試及交付報告均放在此目錄之下。完成回報中的服務檔案路徑也必須以 `F:\yuct_api` 開頭。
- 開始寫入前，先確認 F: 存在、讀取 `F:\AGENTS.md`（若存在）、檢查 `F:\yuct_api` 是否已建立，並查看其中檔案、repo、branch、dirty state 與 WIP。
- 若目錄不存在，建立 `F:\yuct_api`。若目錄已是本服務 repo，沿用並保護既有 WIP；若目錄存在但為空，直接在該目錄建立服務。
- 若目錄已含不相關的 repo、使用者資料或不相容內容，不得清理、覆寫或自行改用別的根目錄；先保護既有內容，說明具體衝突，暫停衝突檔案的寫入並繼續可獨立完成的工作。
- 私有 PDF、解析結果、資料庫、browser profile、session 及其他秘密都儲存在此目錄下的私有資料路徑；若使用Git，確認這些資料已被忽略且不會被 stage 或提交。
- 前次盤點資料仍位於下方舊工作區路徑，屬唯讀來源；不得把工作目錄改回舊路徑。

在 LUNA 當前工作區先確認實際路徑、AGENTS.md、Git 狀態、現有檔案與可用工具。
依「強制交付目錄」檢查並使用 `F:\yuct_api`，不要將服務交付在 LUNA 當前工作區或其他目錄。
若當前目錄是同步素材鏡像，仍將服務建置在 `F:\yuct_api`；只把必要的唯讀證據複製到服務私有測試資料夾。
保護任何既有 WIP；不要 reset、clean、stash、廣泛 stage，亦不要修改無關服務或 repo。
不要讓找不到舊樣本變成停止所有可進行工作的理由。

## 3. 目標網站與真實範例

網站基底：
https://is.ycut.com.tw

查詢入口：
https://is.ycut.com.tw/is/development/address/search

已授權實測地址：
新北市淡水區民權路19號3樓

前次精確命中的明細：
https://is.ycut.com.tw/is/development/address/detail/F_27_FE_1294_00911000

實際操作：
- 依畫面選新北市、淡水區、民權路。
- 門牌輸入19，樓層輸入3。
- 搜尋後有20筆候選，包括同號「3樓之1」等，以及其他巷弄近似結果。
- 點精確地址後，網站另開明細分頁。
- 明細右側「公有雲他店申請」有既有紀錄，點「查看謄本」另開 PDF 分頁。
- 「本店申請」前次沒有紀錄，不代表不能使用目前帳號有權存取的他店既有紀錄。
- 「立即申請」是不同操作，本任務不得自動點擊或觸發新申請。

前次建物識別：
- adrlanddoorsId：F_27_FE_1294_00911000
- buildno：00911000
- sectionNo：1294
- sectionName：馬偕段
- PDF 內顯示建號：00911-000
- 地號顯示：0196-0000

識別碼保留字串及前導零，不要把建號、地號或登記次序當整數儲存。

## 4. 已觀察的 Network 與已知未知

### 已確認

登入後瀏覽器取得 HTTP 200 JSON：

GET /api/buildingDesc/list?address={URL編碼完整地址}&pageIndex=1&pageSize=50

回應外層包含：
id、apiVersion、method、status、data

搜尋結果位於：
data.result[]

候選欄位包含：
adrlanddoorsId、buildno、address、totalPin、totalFloor、toFloor、fromFloor、buildAge、isFavorite

明細端點：

GET /api/buildingDesc/detail/{adrlanddoorsId}

明細請求實際觀察到：
websiteName: Development_WS

搜尋端點的 websiteName 值前次未另存證，須重新觀察；不要假設所有端點共用同一 header。

明細 data 包含：
adrlanddoorsId、address、city、area、road、sectionNo、sectionName、landnoList、
buildDate、totalFloor、toFloor、fromFloor、masterPing、totalPing、
outbuildingPing、commonPing、usename、material、transcriptList

既有 PDF 資料直接位於：
data.transcriptList[]

每筆觀察到：
transcriptID、status、lord、owner、message、acquiredTime、
isOwn、pdfFile、isOldTranscript、isStore

文件主機：
docs.evertrust.com.tw

前次 pdfFile 回傳 HTTP URL，但實際瀏覽器及獨立 HTTP 下載皆使用 HTTPS。
請從當次合法明細回應取得文件 URL，不要拼湊、枚舉或硬編碼文件 token。

前次後端直接 GET 已觀察到的 HTTPS 文件 URL，未帶瀏覽器 Cookie：
- HTTP 200
- Content-Type：application/pdf
- 176254 bytes
- PDF 簽頭有效
- 共2頁

此樣本 SHA256：
4a1a8e3ba366a97d2d3363871136f092e163e192bad50c7f2173bf9f9abc6455

此雜湊只用於驗證前次既有樣本，不要求未來網站回傳的新文件永遠相同。

### 必須誠實保留的未知

- 未登入的獨立搜尋請求前次回302；不得把重新導向登入 HTML 當成功 JSON。
- 已登入瀏覽器請求未觀察到 Authorization header，存在 Cookie／OIDC session。
- 哪些 Cookie 必要、session 有效期、刷新順序、裝置或 IP 綁定尚未確認。
- 前次沒有驗證獨立後端 HTTP client 攜帶授權 session 能否成功重放搜尋及明細。
- 前次沒有在新 Playwright profile 重新登入，也沒有測試 OTP／2FA／CAPTCHA。
- 文件 URL 前次無 Cookie 可下載，不表示所有文件皆相同，也不表示 URL 永久有效。
- 檔案／API 存在不等於網站授權條件已確認；遇到實際權限限制或禁止自動化條款，記錄並停止受限的那部分，不繞過。

## 5. 實作決策與基本技術方向

優先實作：
Playwright 管登入及 session；後端 HTTP client 查搜尋 API、明細 API並下載 PDF。

不要預設全流程瀏覽器點擊。先做一次有證據的 session 重放驗證。
若某一環節因實測證明依賴瀏覽器，僅該環節改為 Playwright；若整體確實無法重放，才採有理由的瀏覽器流程。
「尚未測試」不等於「無 API」或「必須全瀏覽器」。

在沒有既有專案限制時，建議：
- Python、FastAPI、Pydantic。
- httpx 作 HTTP client。
- Playwright 作登入與必要瀏覽器操作。
- pypdf／pdfplumber 作文字與表格解析。
- 本機 OCR 處理影像欄位。
- SQLite 保存 job、文件索引、解析版本與必要狀態；PDF／JSON 用私有檔案目錄。
- 先採單帳號單 worker、受控並行，依真實網站觀察調整。

以上是可執行的預設，不要求無視現有 repo。若現有 stack 可直接支援，選最小一致改動並說明。
查閱所用工具的官方文件、確認實際版本；不要依記憶寫不存在的方法。
不強制容器、Redis、外部資料庫或雲端 OCR，也不要擴成完整 SaaS。

## 6. 執行階段

依序執行，每階段寫入簡短 evidence／handoff。一般可自行決定的實作不必反覆詢問。
需要真人驗證時才請使用者接手，並繼續不依賴登入的 parser、契約與離線測試。

### P0：確認環境與證據

- 確認 workspace／repo／branch／dirty state、Python／依賴／瀏覽器／OCR 可用性。
- 讀取前次證據，將已確認、推論及未驗證分開。
- 用實際網站核對入口、當前登入狀態、可見控制項及 Network。
- 只調查本任務必要的頁面和資料，不讀其他個人檔案或整個瀏覽器 profile。
- 確認資料目錄及秘密儲存邊界，再寫實作。

P0 完成物：當前狀態、實際工作目錄、資料流、端點證據、缺口清單。

### P1：登入 session 與 API 重放

- 建立服務專用的持久 browser profile，不讀取或複製使用者日常 Chrome profile。
- 開 headed 瀏覽器至已授權網站，使用者自行輸入帳密與驗證。
- 不在提示詞、程式碼、命令列、日誌、截圖或測試fixture中留下密碼、Cookie、token。
- 使用受 OS 保護的秘密儲存或加密 storage state；Windows 可採 DPAPI／相應系統儲存。
- profile 本身也包含登入秘密，限制目錄存取，排除 Git 與一般打包輸出。
- 用同一授權範圍的 session 在獨立 HTTP client 重放搜尋和明細；記錄去敏後 method、path、必要 header 名、status、JSON schema。
- session 失效偵測涵蓋302、401、403、登入 HTML、業務錯誤與帳號不匹配。
- 403 未必是登入過期，須區分權限拒絕；不得以反覆登入掩蓋。
- 不做無界自動重登入；需要驗證時設 requires_login，提供人工處理入口。
- session 保存後，在新程序恢復並再次做一筆合法讀取，不能只證明同一瀏覽器當下可查。
- 如果重放失敗，先從當次 Network 檢查 session／header／跳轉差異；不要關閉 TLS、忽略權限或用猜測代替結論。

P1 Gate：登入瀏覽器成功、獨立後端搜尋／明細成功、重啟後重用成功，分別記錄結果。
尚未完成真人登入或重用不能標 PASS。若無法完成，清楚標 WAITING_USER／PARTIAL，繼續能獨立完成的階段。

### P2：地址匹配、既有謄本、原檔儲存

- 輸入完整地址，保留原文及正規化結果。
- 只做有規則且可測試的全半形、空白、已知中文樓層數字轉換；不丟掉巷、弄、號之、樓之、地下室或室號。
- 精確比對唯一候選；零筆回 not_found，多筆精確候選回 ambiguous，回傳必要候選資訊供呼叫者選擇。
- 「19號3樓」不得錯配「19號3樓之1」或「某巷19號3樓」。
- 按實際分頁欄位遍歷必要範圍，不假設第一頁就是全部。
- 從精確匹配建物的明細取得 transcriptList，不依地址猜文件 URL。
- 同時支援帳號有權存取的本店與公有雲既有紀錄，不假設只有 isOwn=true 才能用。
- 多份既有紀錄先依可用狀態與 acquiredTime 選最新可用；若語意不明或時間無法判讀，保留差異並回待核對，不猜。
- 無既有PDF回 no_existing_pdf，不觸發「立即申請」或新申請／扣款。
- 文件URL只允許已確認的 HTTPS 文件主機；每次重新導向也檢查目的地。
- 禁止任意使用者 URL、file path、內網位址、非預期 scheme 與任意轉址，避免變成任意下載代理。
- 檔案下載用不帶登入站 Cookie 的獨立 client；只有實證必要且授權範圍明確才調整，不外洩 Cookie 到文件站。
- 設定可調整的 timeout、最大檔案大小、有限重試及退避；429遵守 Retry-After。
- 驗證HTTP狀態、Content-Type、PDF簽頭與可開啟頁數；HTML、空檔或截斷檔都不是成功。
- 先下載臨時檔，完整驗證後原子寫入。原始 bytes 不修改，保存 SHA256、大小、頁數及來源識別。
- 使用 document_id 及安全檔名，不直接用地址當路徑。
- 儲存下載時間、網站 acquiredTime、PDF內各頁查詢時間；不可互相覆蓋。
- 文件敏感 URL 不出現在一般日誌、公共 repo、API 回應或前端可見訊息。
- 預設使用最新「既有」紀錄，不保證是今天的登記狀態。

P2 Gate：真實範例精確命中，原始PDF成功落地、可讀且有雜湊；無結果／歧義／無既有PDF有明確行為。

### P3：正式 parser 與局部 OCR

不能把 inspect_sample.py 包一層 API 就宣稱正式 parser。
建立可版本化、可重解析的 parser，辨識文件類型與區段，支援重複列、跨頁及多所有權人資料結構。
文件格式不支援時回明確狀態，不回看似成功的空JSON。

優先結構：
- document/source：文件類型、來源站、建物ID、申調時間、各頁查詢時間、下載時間、parser版本與hash。
- building：建號、地段地號、門牌、登記日期／原因、用途、建材、層數／層次、完工日期、主建／層次面積。
- accessories[]：用途、面積與單位。
- common_parts[]：建號、面積、持分分子／分母、原文與頁碼。
- owners[]：登記次序、原樣遮蔽的姓名／識別、地址、權利範圍、登記／原因日期、權狀字號。
- other_registrations[]：其他登記事項，先忠實保留文字，無法可靠拆解時不要補造。
- 每個欄位保留 value、raw、unit、page、method、status；信心分數只有真實可計算時才提供，不能一律填1。
- 缺漏／待OCR／來源空白／不支援／解析失敗分開，不用同一個 null 掩蓋原因。
- ROC 日期轉西元須有規則，保留原文；查詢時間使用 Asia/Taipei 時區。
- 遮蔽值保持遮蔽，不推測缺字，不用其他資料補還原。

前次樣本已核對：
- 第1頁：建物標示部；第2頁：建物所有權部。
- PDF內建物總面積46.28平方公尺，陽台5.6平方公尺。
- 共有部分有2筆，持分與面積各自保留，不直接相加成私人面積。
- 完工日期為民國097年12月23日，即2008-12-23。
- 網頁總建23.73坪、主建14坪、附屬1.69坪、共有8.03坪，與PDF欄位口徑不同，必須分別保存。
- 網頁用途「見其他登記事項」，此PDF用途「一般服務業」；不能用網站值覆蓋歷史PDF。
- 網站 acquiredTime 是2023/09/13 12:31:13。
- PDF第1頁查詢時間2023/09/13 12:31:37，第2頁12:31:47。
- 第2頁所有權人「地址」畫面可見，pypdf和pdfplumber文字結果卻空白，該頁有影像物件。

必須完成混合內容處理：
1. 先文字與表格擷取。
2. 檢查目標欄位是否缺字、影像覆蓋或文字不足；不能因整頁有文字就跳過OCR。
3. 對所有權人地址等影像欄位，定位並裁切必要區域，使用本機繁體中文 OCR。
4. 將辨識原文、定位頁碼／區域及方法保留，與原頁視覺核對。
5. OCR不確定或不可用時保留 needs_ocr_or_manual_review，不把文字輸出空白當作原文件沒地址。
6. 若 required_fields 包含未確認欄位，任務回 needs_review；非必要欄位可 completed_with_warnings。
7. OCR缺依賴時，先調查可用WinRT OCR／已安裝Tesseract或可用本機引擎；使用官方來源補必要依賴。
8. 不把個資送雲端 OCR／外部 LLM，除非使用者另行授權。
9. 不用人工看過的地址硬編碼成OCR結果。

P3 Gate：主要文字／表格可解析，影像地址有真實OCR結果與核對證據，或誠實保留缺口；不得標成全部通過。
至少以第二份不同布局或匿名化自建fixture驗證不是硬編碼單一PDF；fixture通過不等於第二份真實文件驗收。

### P4：REST API、持久任務與操作方式

API最低需求：

POST /v1/queries
- body：address，選填 required_fields。
- 僅取得既有謄本，不接受任意URL或來源帳號／tenant覆寫。
- 返回202：query_id、status、status_url。
- Idempotency-Key以呼叫者範圍隔離；同key不同payload回409。

GET /v1/queries/{query_id}
- 回任務狀態、輸入、精確匹配、必要候選、文件ID、解析結果、來源時間、warnings與可理解錯誤。
- 只回當前呼叫者有權存取的結果。
- 明確界定正式JSON schema；前次sample-result.json是檢查樣本，不直接當成對外型別。

GET /v1/documents/{document_id}/pdf
- 回未修改的原始PDF，含合理Content-Disposition／ETag。
- 每次驗證資料範圍，不把PDF設為公開靜態目錄。
- 不暴露主機絕對路徑或來源文件token URL。

POST /v1/documents/{document_id}/reparse
- 使用已保存的原檔，指定已安裝parser版本。
- 不重新查網站，不覆寫原PDF；建立新解析版本，保留舊版本。
- 不接受任意程式碼、模組路徑、下載URL或輸入檔路徑。

GET /v1/session/status
- 回ready／requires_login／unknown與最後檢查時間。
- 不回傳Cookie、token或profile內容。

任務狀態：
queued、running、requires_login、completed、completed_with_warnings、
needs_review、not_found、ambiguous、no_existing_pdf、failed

定義每個狀態能否恢復、是否終止及下一步；登入恢復後不得遺留永久running任務。
持久化job及parse revision；程序重啟後有明確恢復策略，不只存在記憶體。
單worker／SQLite可接受，暫不需要外部queue；遇到同帳號並行與session刷新採鎖避免互相覆寫。
不要在async路由直接阻塞整個server執行長時間PDF/OCR；放入受控worker，限制等待佇列。
服務重啟、錯誤與重試不能重複建立新調閱或錯用不同帳號資料。

本機預設僅綁127.0.0.1，服務呼叫驗證與帳號／租戶範圍由伺服器設定。
即使是單使用者MVP，也要有穩定principal與document／job歸屬；不能任意ID就能下載其他人的PDF。
若提供多個principal，測試跨principal存取拒絕；尚未支援多租戶時不要對外宣稱支援。
API key等秘密不進Git；無權限或不存在的ID統一404。

提供Windows可操作入口：
- 初始化與安裝必要依賴。
- 開啟人工登入瀏覽器。
- 啟動／停止本機服務。
- 查session狀態。
- 送出範例地址，等候並取得JSON與原始PDF。
- 重新解析已保存PDF。

採清楚的PowerShell腳本或CLI與中文README即可，不必新增完整前端。
秘密不可經命令列明碼參數、範例或日誌輸出；README用安全輸入方式說明。
提供正式OpenAPI，與路由實際行為一致；不要只複製舊草案。

### P5：測試與真實範例驗收

必要測試聚焦實際風險，不寫僅映射實作的測試：

A. 地址與清單
- 精確地址、樓之／號之差異、巷弄、地下室、全半形、前導零。
- 無結果、多筆精確候選、分頁、無既有PDF、多份既有紀錄。
- 不接受第一筆近似結果，不自動新申請。

B. Session／HTTP
- 302登入跳轉、401／403、200登入HTML、業務失敗、schema變動。
- session過期及人工重新登入後恢復。
- 已保存session在新程序中的重用。
- 文件過期URL、429／5xx／timeout、有界重試。
- 非允許文件主機／轉址、錯Content-Type、空檔、截斷PDF、大小超限。

C. PDF與OCR
- 真實樣本文字、日期、單位、2筆共有部分、遮蔽值與第2頁影像地址。
- 不混淆網站23.73坪與PDF46.28平方公尺。
- 圖像欄位缺漏可被偵測，OCR不可用時不假稱完整成功。
- 多所有權人／跨頁及不支援格式至少有fixture覆蓋。
- 缺必要欄位時needs_review，原檔hash保持不變。
- reparse不存取網站，保留舊解析版本。

D. API／持久化／資料範圍
- 202任務到明確結果、冪等衝突、程序重啟、worker失敗恢復。
- 查詢／JSON／PDF／reparse均有相同授權限制。
- tokenURL與憑證不出現在API和一般log。
- 無權限資料存取拒絕。

最後用已授權範例地址走完整鏈：
其他程式或CLI送POST → 真實網站搜尋 → 精確明細 → 既有PDF →
保存原始bytes → 解析／OCR → GET結果 → 受保護的PDF下載。

實際連線與真人驗證可能需要使用者操作，先給具體可用的入口及步驟。
不要要求使用者重新輸入已知範例地址、已提供欄位或自行整理資料。
不要為測試額外大量查詢；只做必要、有界的驗證。
不得以mock、fixture、200健康檢查或瀏覽器畫面取代完整鏈驗收。

## 7. 最低交付物

1. 可操作服務原始碼與鎖定／可重現的依賴。
2. 開始使用README、Windows啟動／登入操作入口、設定範例。
3. 實際可用REST API及OpenAPI文件。
4. SQLite schema／migration或同等初始化與恢復方式。
5. session管理、API adapter、地址匹配、下載儲存、parser／OCR、worker清楚分工。
6. 必要測試與實際執行結果。
7. 私有data資料夾：原始PDF、解析JSON、版本、來源日期、SHA256；預設排除Git。
8. 去敏證據：實測端點、狀態、payload/schema摘要、人工驗證需求與真實測試時間。
9. 完成報告／接續handoff：已完成、目前服務URL、操作命令、已驗證範圍、剩餘缺口、下一步。
10. 秘密與個資檔案的排除規則；交付程式與私有runtime資料分開。

若使用Git：
- 初始與結束都回報branch／status。
- 只處理自己建立或明確修改的檔案，不提交來源樣本、秘密、profile、log或PDF。
- 未有明確授權不要push產品repo、開公開服務或部署。
- 本提示詞所在tcp-git-313/prompt是提示詞發布repo，不是產品原始碼目的地。

## 8. 完成回報與誠實界線

以繁體中文簡潔回報：

- CURRENT：實際完成到哪裡。
- PASS：有實際證據的項目。
- PARTIAL：程式完成但真實驗證未完成，或OCR／格式仍有限制。
- WAITING_USER：只有確實需要真人登入／驗證時使用，附已開好的入口及要做的具體動作。
- FAIL／HARD_STOP：精確原因，不隱藏第一個實際阻擋。
- 檔案絕對路徑、啟動方式、API範例、測試與真實驗收證據。
- 是否真的啟動服務、是否重啟驗證session、是否真的完成獨立HTTP重放。
- 原始PDF與JSON是否可取回、影像地址是否已完成OCR核對。
- 未完成項目的接續步驟，不將其寫成已完成。

禁止下列結論：
- 只見到API就說後端session重放已通過。
- 只下載到PDF就說解析全部成功。
- pypdf沒讀到地址就說文件沒有地址。
- 下載時間當成謄本查詢時間。
- 全部stub／mock完成就說服務可用。
- 尚未部署卻給部署成功或虛構網址。
- 缺少工具、權限或登入時捏造成功。
- 看到本提示詞就直接跳過本機／網站現況檢查。

現在請從P0開始，持續建置到本機可操作與真實驗收；只有明確需要人工驗證或超出授權的動作才暫停相關工作，其餘工作繼續完成。

<!-- END OF LUNA YCUT ADDRESS TRANSCRIPT SERVICE BUILD PROMPT -->

