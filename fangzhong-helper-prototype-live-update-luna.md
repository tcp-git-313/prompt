# 任務：房仲好幫手新版實站調查與既有 Prototype 增量更新

指定執行模型：LUNA。建議 thinking：high；如介面不支援，使用 LUNA 實際可選的最高推理設定，不自行換模型。

這是可直接執行的更新任務，不是只寫計畫或分析報告。請先調查目前競品，再修改使用者指定的既有本地原型，實際開啟、操作、比較、修正並交付。這份提示詞本身的發布與驗證，不表示原型已完成更新。

## 1. 本次目標與範圍

競品：<https://helper.fangzhong.workers.dev/beta>。

使用者明確指出的主更新目錄：

`C:\Users\Tu\Documents\Codex\2026-10-08\https-helper-fangzhong-workers-dev-beta\outputs\prototype`

目標是把這份 2026-10-08 的獨立可操作原型，增量更新到本次實際調查到的競品介面、資訊架構、路由、操作流程、元件、響應式行為與資料狀態。先忠實還原，保留可操作性，不自行改成另一種 SaaS 介面。

研究、差異、版本、QA 與交付紀錄放在：

`C:\Users\Tu\Documents\Codex\2026-10-08\https-helper-fangzhong-workers-dev-beta\outputs\prototype-refresh-20261023`

已存在的 `prompt-preparation` 是本任務準備資料；不要覆寫或刪除。執行時間若已不是 2026-10-23，仍保留這個任務目錄，在其下新增實際日期的 run 子目錄，所有觀察寫實際時間（Asia/Taipei），不要假稱提示詞日期就是調查日期。

本任務不修改 HUB、Ticenpi SaaS、正式資料庫、Production 或其他既有獨立產品；不做公開部署、不提交產品 Git、不推送產品程式、不安裝/重配競品本機程式。可以修改主更新目錄內的本地原型程式、文件、自己的合成測試與更新所需素材。

## 2. 原提示詞是什麼：已找到的直接來源

原始使用者貼上的要求位於：

`C:\Users\Tu\.codex\attachments\df58cfa1-1df9-4d9b-863b-3a2b1f549362\貼上的文字.txt`

原文開頭是「我要你做的不是『參考這個網站的風格』，也不是『做一個類似版本』」，接著要求完整產品逆向分析，重建視覺、版型、互動、頁面結構、操作流程、狀態變化、頁面連動都盡可能 1:1 的可操作版本。

原任務允許在第三方 API、登入、付款或真實資料不能執行時，以 Mock／Local State 模擬使用者流程；同時要求研究、Site Map、User Flow、Component Inventory、Design Tokens、桌機與手機對照、逐流程 QA。原任務明確要求先研究再 coding，不能自行美化或只還原首頁。

原始貼文未指定當時執行模型，不能據此推斷舊原型是由哪個模型完成。LUNA 是使用者這一次指定的執行者。

本檔最後附原提示詞完整文字，僅統一換行；原文裡的「把網址貼在這裡」由本任務的競品網址取代。附錄是歷史要求，不覆蓋本檔的增量更新範圍、資料保護、版本與驗收規則。

## 3. 已在 2026-10-23 實際核對的基線

以下只是本任務準備時的直接觀察，執行時必須重新確認；不是已完成新版全站驗收。

### 本地舊原型

- `package.json`：`fangzhong-helper-local-clone`，版本 0.1.0，啟動命令 `node server.cjs`，原設計不需 npm 安裝。
- `routes.json` 實際有 95 筆；`pages.json` 實際有 320 個模板鍵；`site-map-observed.json` 有 46 筆。它們是不同口徑，不能相加，也不能把 95 URL 當成 95 個完整能力。
- 主要檔案：`app.js`、`enhancements.js`、`public-tools.js`、`floor-editor.js`、`import-posts.js`、`prototype.css`、`index.html`、`server.cjs`、`pages.json`、`routes.json`、`design-tokens.json`、`assets-map.json`、`file-manifest.json`、`start.bat`、`README.md`，以及 `assets`、`reference`、`qa`。
- 現行前端是獨立原生 JS 與模板狀態處理，不是現成 React 工程。不要為了習慣而先整包改寫成 React／Next.js／另一套模板框架。
- README 明確標示 PARTIAL；登入、實際刊登、真實 LINE 傳訊、謄本 OCR、AI 人像分割、即時查詢等多項是本地模擬或近似流程。舊 QA 的分數也有低於 90 的項目，不可繼承為本輪通過。
- `app.js` 使用 Local Storage key `fangzhong-local-prototype-v1`；其中種子資料有原帳號業務資料。更新時使用合成 fixture，不把真實姓名、電話、Email、房源、客戶或公開分享 token 寫進新測試或上傳到公開 repo。
- `pages.json`、`routes.json` 以 UTF-8 讀取都沒有 U+FFFD 替換字元。終端中文顯示亂碼時先檢查輸出編碼，不要誤判檔案毀損後全檔轉碼。
- 主目錄 README 連到 `PRODUCT-ANALYSIS.md`、`QA.md`，但準備時該目錄沒有這兩份檔。F 槽舊版有中文文件；請補正主目錄文件與有效連結，不宣稱不存在的檔案已交付。

### 其他資料夾的身分與保護

| 路徑 | 已核對的身分 | 本次處理 |
|---|---|---|
| 使用者指定的 `outputs\prototype` | 原始本地互動原型 | 本次實際更新目標 |
| `F:\CLONE房仲好幫手` | 舊獨立拷貝；準備時 10 個核心檔與主目錄逐 byte 相同 | 只讀比較來源；不預設同步、不整包覆蓋 |
| `F:\CLONE房仲好幫手\reconstruction-20261016` | 後續獨立 Node／SQLite／Agent 重建；README 記載較新歷史版本 v1.1462–v1.1480、39 頁／104 份 JS／58 份規格；各數量與驗收範圍不同 | 只讀參考；不修改其 app/data/specs/tests，不把它當舊原型的可刪目錄 |
| `F:\Clone解析` | 已拆出的多個後續功能專案 | 保持原狀；不能用舊原型或整包鏡像覆蓋 |
| ChatGPT 專案 mirror 下的 `sources` | 同步參考資料 | 一律只讀，不改名、不移動、不刪除 |

若使用者之後另要求同步 F 槽，先比較檔案與版本、備份並使用明確檔案 allowlist；本提示詞不授權順手同步那些獨立成果。沒有外部資料夾也不應阻塞主目錄更新。

### 已發現的啟動風險

舊 `server.cjs` 與 `start.bat` 寫死 `127.0.0.1:9488`。準備時 9488 確實有其他程序監聽，瀏覽器該埠顯示的是 Ticenpi Post，而非此原型。

- 不要 kill 9488 的程序，不要重用那個 tab 當 Clone 已開啟的證據。
- 不要假定 9490 可用，它也是後續重建專案的歷史入口。
- 為本原型增加可設定 PORT（參數或環境變數），只監聽 loopback；啟動前驗證占用，選實際可用埠，把真正 URL 寫入文件。
- 啟動器需先確認服務啟動成功及身分吻合，再開預覽；處理啟動失敗、退出與重複啟動，不要先開到別人的服務。
- 埠或 `localhost`／`127.0.0.1` 改變會改變 Local Storage origin。不得從占用 9488 的其他產品讀資料或「自動搬移」。已有本原型資料須提供可回復的匯出／匯入／schema migration；新測試使用獨立合成資料與清楚的測試 origin。

### 競品目前版本與可見變化

2026-10-23 已使用原有登入的 Chrome 正常讀取競品。初讀既有首頁仍顯示 v1.2126／ef0c3603／本機最新版 v0.224；新開公告頁及首頁重載後，顯示：

**網站 v1.2132，部署 2026.10.23 17:50，版本標記 4589bff4；本機程式最新版 v0.226。**

這是頁面顯示的網站版本與「本機最新版」，不是已驗證已安裝的本機版本；請分別記錄。執行時若已更新，使用實際新版本，不鎖死 v1.2132。

歷史公告頁當次沒有公告，不能用它證明沒有更新；不能編造 release notes 或功能正式上線日期。

以下路徑已從正常首頁或 FB 導覽看到並核對舊 95 筆 route 清單不存在（其中一部分已進入新頁查看介面）：

| 新的本地路由缺口候選 | 本輪準備已觀察 | 仍要補驗 |
|---|---|---|
| `/beta/dev-letter` | 開發信頁，整社區／指定屋主、三種範本、附近成交條件、文字編輯、儲存列印、寄送紀錄、分享入口 | 新建到儲存重開、人工改文保留、成交來源與欄位、輸出、分享狀態 |
| `/beta/stats` | 初始 Loading 後出現月份選擇、客源／狀態、成交、聯絡帶看、完成待辦、刊登分項及計數說明 | 換月、空資料、計數／去重規則與資料更新連動 |
| `/beta/land-tax` | 土地電子謄本選擇、手動土地輸入入口，土增稅與房地合一稅估算流程說明 | 全表單、文件解析、計算與例外；頁面說明不代表演算法已驗證 |
| `/beta/greeting-card` | 春節／端午／中秋、LINE 正方形／社群直式、範本、上傳人像與自訂範本、文字和位置調整、單張與批次下載入口 | 合成圖片實際匯出、尺寸、alpha、壞圖、容量及重開；當次未上傳或下載 |
| `/beta/feedback` | 分類、訊息輸入與送出、我的訊息；載入後表單才出現 | 本地送出／回覆／未讀；原站不送訊息給管理者 |
| `/beta/facebook/remote` | FB 導覽可見手機遙控入口 | 探索實際流程；不觸發真實發文 |
| `/beta/facebook/properties` | FB 建立文案流程改指向物件入口 | 舊文章／文案群組與物件來源、相容／轉址 |
| `/beta/facebook/care` | 每日養帳入口 | 計畫、限制、取消、歷史；不對外按讚留言 |
| `/beta/real-price` | 首頁可見實價登錄入口，舊 route 清單沒有此路徑 | 完整 UI／查詢／匯出流程，不用同名歷史報告冒充 |

「舊 route 清單沒有」只證明這份原型缺映射，不等於競品功能直到今天才誕生。`reconstruction-20261016` 已記錄部分功能，不能重複當全新發現。

另有已觀察的差異：

- 首頁是買賣物件稱呼、新增網址入口當次只列 591、問題與建議導覽、更新的今日待處理與新手步驟。
- FB 總覽是建立物件文案 → 匯入社團 → 排程發布；載入完成後導覽有物件、手機遙控、每日養帳等項，初載與完成狀態不完全相同。不能擷取 Loading 畫面當正式模板。
- 官方手冊現在涵蓋買賣／租賃版本差異、客戶工具／回收桶、委託提醒／週報、成交後關懷、外站配案、平台修改上下架、Marketplace 管理、手機遙控、清舊文、LINE 狀態查詢與網址上架、DM 線上分享等。這些只屬「官方文件宣稱」線索，必須逐一從實際頁面驗證；租賃入口可能受帳號版本影響，不能因當前帳號沒顯示就宣稱已移除。
- 官網「怎麼用」可能第一次進頁自動開啟；避免教學遮罩擋住操作後，錯誤判定按鈕壞掉。

本輪準備沒有新增／修改競品紀錄、沒有送出刊登或訊息、沒有建立公開分享，也沒有實測上述新增能力的最終輸出。不要將這份準備清單當成已完成全站操作。

## 4. 先建立不可混淆的版本與證據基準

開始時讀取適用 AGENTS.md；做檔案清冊、Git status（有 Git 才讀）、主更新目錄雜湊、啟動方式及資料保存方式。不要 reset、clean 或覆蓋使用者的未提交工作。

先在主更新目錄以外做備份，保留本次會改的程式與模板、文件和相關素材，寫相對路徑、大小、SHA256。沒有 Git 也要有可回復備份；不要為這個任務順手初始化或整包提交 Git。

明確分開三個舊基線：

1. 2026-10-08 原型（95 route／320 模板）。
2. 2026-10-14 競品能力報告（49 卡／302 條件），路徑 `outputs\競品能力深度拆解20261014`，只能當歷史研究。
3. 2026-10-16 後續獨立重建（58 規格等），不是本次主更新目錄。

不得把它們相加當覆蓋率，不能把歷史 PASS、原型 PASS MOCK 或文件／程式存在，升級成目前競品真人成功。

建立新的 `source-version-manifest.json`：時間、來源網址、網站版本／部署標記、頁面本機最新版、實際本機版本（未讀到則 Unknown）、頁面／截圖／資產 hash、擷取工具、viewport、觀察層級和限制。網站在途中更新就分批凍結版本，補 delta；不能把不同部署的模板湊成「同一版完整還原」。

## 5. Browser Investigator：先重新調查競品

優先使用使用者已登入的 Chrome 競品分頁，從目前的 URL 與登入狀態辨識；不要依賴這份提示詞中的暫時 tab ID 或過去帳號名稱。瀏覽器工具以工作區實際可用的文件為準，不編造 API。

如果需要登入，讓使用者在同一可讀取瀏覽器登入；已有登入就直接繼續。不要另開隔離 profile 後，因為沒登入又要求使用者重做。無法讀取特定頁時先做其他可讀頁，記錄確切阻塞。

如有可用 REA 與既有 loopback CDP，可被動取得版本、DOM、已交付程式、必要 network metadata，保留 Evidence ID、來源 digest、截斷與未觀察限制；普通點擊與輸入用當前可用的瀏覽器操作工具。沒有 CDP 或某 provider 不可用就使用正常瀏覽器取證，不為此重配瀏覽器或安裝整套工具。不要為研究啟動競品安裝器、原始程式或 Mac 安裝命令。

### 調查順序

1. 首頁、App Shell、側欄、手機導航、全部功能、官方手冊目錄、公告、版本。
2. 新 route 候選與 FB 導覽變化，先確認新增／修改／轉址／移除／權限差異。
3. 物件（含可見交易類型）、網址匯入、照片／封面、文案；FB／平台刊登；LINE。
4. 客戶、需求、聯絡、帶看、委託、週報、配案、預約與待辦／統計之間的連動。
5. 單／多物件 DM、開發信、屋主／買方報告、賀卡、謄本、稅務、實登、學區、社區、影像與格局等工具。
6. 設定、品牌、分享與失效、搜尋／備份、裝置頁，以及實際發現的其他入口。

這不是固定 49 或 58 項的上限。從目前頁面可發現的站內連結與可見互動擴展，將資料型 ID 正規化為 route pattern，別把每一個私人房源當成一種新功能。不要猜 API、批次掃描 private ID、探測管理員頁或繞過權限。

### 每頁／每個能力的紀錄

記錄入口、URL／pattern／query、前往與返回路徑、breadcrumb、App Shell／active nav、主要 CTA、次要 CTA、欄位與必填／預設／驗證、Tab、Modal／Drawer、Popover／Tooltip、disabled／hover／focus、教學遮罩、成功與錯誤訊息、Loading 到完成、保存與重新開啟、資料連動、使用者最後得到的成品。

逐一考慮 Empty、Loading、Success、Error、Partial Failure、Retry、Cancel、Resume、Duplicate、Permission、Expired、Limit、Mobile；未觀察者寫 Unknown，不能補想像中的流程。特別量測舊輪缺失的 click 數、必填／人工輸入數、context switch、等待與手動恢復；不能拿箭頭分段數當 click 數。量不到就列量測缺口與原因。

技術推定分成 Confirmed（直接證據）、Highly Likely（行為推論）、Possible（候選設計）、Unknown。前端呼叫了 API 不能證明後端完成；HTTP200／202、queued、pasted、草稿或綠色裝置燈都不是外站成功。

### 外部副作用的界線

本次原站操作以正常只讀探索為主。可在本地 Clone 隔離測試新增、修改、刪除、排程、分享、預約等完整合成流程；原站若需要保存合成草稿，須先確認符合使用者本次授權，並保存清楚的清理清冊，不更動既有真資料。

不要為了重建而送出新的 FB／LINE 訊息、平台刊登、按讚留言、意見給管理員、付款、接受新條款、產生新配對碼／NAS key／OAuth scope 或公開報告。上輪對特定 CAP 動作的同意，不能被擴充為新的對外測試。確有驗收需要時，先準備具體預覽，對精確對象與內容取得所需授權；其餘不依賴此動作的工作繼續。

不要重開已撤銷的兩份 CAP 公開分享，也不要替使用者修改原站全域設定。原站付費／法定條款不複製成已同意狀態。

## 6. 差異矩陣：研究完成一批，才實作一批

先產出人可讀 `新版差異.md` 與可重算 `route-delta.json`／`capability-delta.json`。

每列至少：穩定 ID、能力／頁面、舊本地證據、新觀察來源與版本、差異類型（added／changed／removed／redirected／permission-dependent／unchanged／unknown）、欄位／互動／狀態／樣式／輸出差異、對應本地檔案、復用或修改方案、相容策略、優先級、待驗證、完成狀態、證據路徑。

資料形狀是範例：

```json
{
  "id": "delta-fb-properties",
  "routePattern": "/beta/facebook/properties",
  "sourceVersion": "本次實際值",
  "difference": "added",
  "observedScope": ["navigation", "loaded-page"],
  "sourceTerminalStatus": "UNKNOWN",
  "localFiles": [],
  "implementationStatus": "TODO",
  "localQaStatus": "NOT_RUN",
  "evidenceRefs": []
}
```

不要先為填滿差異表而假定所有候選都是新增、所有手冊能力都成功，或所有沒找到的路徑都已刪除。與 10/16 研究重疊者標明既有歷史證據及仍需重測的部分。

## 7. Implementer：在原型上更新，不把研究當交付終點

優先保留主目錄現有可用互動與獨立架構，以小批次增量修改；每批只處理可解釋的頁面／能力。`pages.json` 約 15.7 MB，先看鍵、欄位與目標片段，不要把全檔灌進對話再盲目重寫。

### 應更新的資料與畫面

- 更新 `site-map-observed.json`、`routes.json`、`pages.json`：新頁、新狀態、導航、query variants、來源版本及證據。保留合理向後相容；舊路徑應按實際轉址或明示已停用，不全部指到首頁。
- 更新 `app.js` 的 route resolution 與事件連接、`enhancements.js`、相關工具程式與 CSS；如需新模組可分檔，不堆成另一個難維護巨檔。
- 新路徑必須有具體頁面與可操作流程，不可全部導向「整合操作／本地設定」占位畫面；導航 active state、返回與 query 狀態要一致。
- App Shell、列表、表單、Modal／Drawer、Toast、Loading／Empty／Error、微動畫、focus／keyboard、桌機與手機行為以新觀察為準。
- 實際記錄 page max-width、側欄／topbar／bottom nav、grid、gap、padding、字級／行高、圓角／border／shadow、按鈕和輸入尺寸；修正 `design-tokens.json`，不任意換設計。
- `assets-map.json` 與 `file-manifest.json` 要對應最終檔案，避免新模板引用舊 hash 不存在素材。對來源可用的圖示／字型／素材保持來源與授權紀錄；不把競品交付腳本當依賴執行。

### 使用者流程必須成立

新增後列表增加、編輯後 detail 更新、刪除確認後列表消失、搜尋／篩選／排序有效；跨頁資料引用一致；保存後重載可讀回；取消不把半成品當已保存。DM／報告／賀卡／工具要有對應的真實本地輸出檔，不能只是成功 Toast。

本機 Mock 可保留原站可見流程，但必須在適當位置清楚標示本地演練及外部結果未連接，不把模擬 posted／sent 混進真實成功統計。不能把去背的顏色法包裝成 AI 分割成功；不能將近似稅務／房貸運算宣稱為原站完整演算法或現行法律正確性。條件未知時保留未驗證狀態，不先補造結果。

各版本的 local state 有 schemaVersion 與 migration，舊資料不自動丟棄、不每次啟動重置；新空白帳號的 seed 使用合成資料。若 schema 不相容，保留舊值、可下載備份與明確恢復指引，錯誤不可靜默吞掉。

模板取自已授權可見行為與結構研究，實作腳本獨立撰寫。擷取的新模板移除原站 `<script>`、事件屬性、真正 form action、追蹤請求、實際帳號／客戶值及私密 token；舊樣本保留歷史證據，不將其私人資料另行公開。檢查 CSS URL、iframe、worker、圖片、fonts 與連結，避免原型載入時悄悄操作競品後端。測試預設不需競品 Cookie，不把憑證拷貝到 Clone。

原站仍需外部整合的功能，提供有輸入、校驗、預覽、狀態轉換、歷史／恢復的獨立本地演練；不能用一段通用文字或假按鈕冒充完整還原。對未觀察的進階規則標記差距；不趁這次重做真正的第三方發布 Agent 或整套 SaaS。

## 8. 驗收：有畫面不算完成

### 功能／回歸

每個 changed／added 能力實際跑入口 → 輸入 → 預覽／確認 → 本地執行 → 成功／失敗 → 歷史 → 重載／修改／恢復；測試合理的邊界輸入、必填、壞檔、重複與部分失敗。外部來源不能測的終態保持 Unknown。

保留並回歸原型已可操作的物件、客戶、待辦、照片／封面、文案、FB／平台演練、LINE 演練、單／多 DM、CSV／XLSX、格局圖／PNG／SVG、亮度 ZIP、預約轉待辦等既有能力，避免新版模板使舊事件 handler 失效。

用路由、UI 狀態與保存讀回確認功能，不以 HTTP200 或頁面非空當 PASS。未知路由要明示未找到，不能 generic fallback 令路由測試虛假通過。追蹤 console error、missing asset、JS 初始化錯誤、CSP 違規及對原站／非預期外部端點請求。

### 輸出

下載須真正取得檔案，檢查 MIME、格式、大小、尺寸、檔案數、ZIP CRC、中文與缺圖；PNG 透明度及主體保留要另驗，不拿「按了下載」當成功。PDF／列印由本地實際輸出與頁面結果驗證，取消列印不能算 PDF 成品。不能沿用歷史輸出檔當本輪結果。

### 桌機／手機與視覺

至少測 1920、1440、1280、768、430、390 寬度；記錄真正 clientWidth／scrollWidth、viewport、縮放／deviceScaleFactor、載入完成狀態與資料 fixture。未真正套用 viewport 的截圖不得作手機證據。

對原站與 Clone 用相同尺寸、相同頁面狀態及等價資料做 screenshot 對照，至少首頁、物件列表／編輯、FB 總覽／排程、客戶或待辦抽屜、DM、報告、新賀卡／開發信／稅務／統計，以及手機首頁／功能頁／導航。資料文字與時鐘等動態差異的 mask 規則明列，不大面積遮掉版面問題。

截圖工具慢或失敗就保留 Unknown 與原因，繼續其他驗收；不能換成 DOM 截圖或舊版截圖後宣稱逐像素通過。對本輪新增功能作實質第二輪修正，不用主觀 90／95 分取代操作證據。

視覺、layout、typography、component、interaction、navigation、workflow、responsive 分開報告。分數若是人工估計必須註明，測得的像素差異另列量測方式。任一重要項低於原要求 90、缺新版對照或核心流程有阻塞，整體仍標 PARTIAL，不宣告完整 1:1 或正式能力同等。

## 9. 交付物與續作要求

主更新目錄：更新後可操作程式、有效啟動方式、README、`PRODUCT-ANALYSIS.md`、`QA.md`（或清楚連到真正存在的中文對應文件）、新版 Site Map／Flows／Component Inventory／Design Tokens／route／asset manifest。

研究目錄至少交付：

1. `原始提示詞與更新範圍.md`：原要求來源、本次目標／非目標與各資料夾身分。
2. `新版差異.md`、`route-delta.json`、`capability-delta.json`：每項新舊對照與證據。
3. `source-version-manifest.json`、`evidence-index.json`：時間、版本、來源、hash、工具、coverage／限制，不保存 credential／token 值。
4. `流程驗收.csv`／JSON：Action、Expected、Actual、status、source version、local build/hash、viewport、fixture、證據位置。
5. 桌機／手機對照頁、截圖、真實本地輸出與輸出驗證清單；可離線閱讀。
6. `Mock與未接通清單.md`、`待驗證與阻塞.md`、`測試收尾清冊.md`：本地 PASS、Mock、外部 PASS 與 Unknown 分開。
7. `變更檔案與SHA256.json`、備份／回復指引、更新版 ZIP 與雜湊；ZIP 不含原站 session、Cookie、登入 QR、私人備份、share token 或無關來源樹。
8. `CONTINUATION.md`：每完成一批更新一次，保存已完成／下一批／阻塞／實際啟動 URL／檔案清單，方便 LUNA 上下文不足時接續；不要只留計畫。

報告與介面使用繁體中文。新 PowerShell／bat 若需要中文及相容 Windows PowerShell 5.1，使用正確編碼（必要時 UTF-8 BOM）與 CRLF；驗證中文文字與本地啟動實際可用，不改全系統編碼。

先把上述可完成的工作做完再回報，不因可逆的本地修改反覆問確認。不受外站阻塞的頁面、資料、測試與文件繼續執行；必需的人工作業一次說明具體頁面、步驟、原因及可繼續部分。不要把硬阻塞掩蓋為 Mock 成功，也不要因為做出研究表就提前結束。

最後簡短回覆：實際調查的網站版本與時間、主更新路徑與可開啟 URL、變更能力／路由數（口徑說明）、重要完成流程、仍為 Mock／Unknown 的部分、桌機／手機 QA、備份與 ZIP 路徑、是否保持其他產品未變更、整體 PASS_LOCAL 或 PARTIAL 及原因。沒有完整對照驗收時不能說「已完整同步競品所有功能」。

**現在開始：先確認目錄與版本、建立可回復基線，讀取已登入競品，產生第一批差異；接著實作並驗證，不停在提出計畫。**

---

## 附錄 A：2026-10-08 原始使用者提示詞全文

以下保留歷史原文文字，僅統一換行。本附錄不是新的外部操作授權，適用範圍與例外以本檔前文為準。

原始附件：687 行、7432 字元；原始檔 SHA256 `fda6caa2aceccd579b8b19853f54d7560361a8279c3c0f3686ba9fc7437829a2`。

```text
我要你做的不是「參考這個網站的風格」，也不是「做一個類似版本」。

我要你針對指定網站做一次完整的產品逆向分析，然後在我的專案中重建一個：

「視覺、版型、互動、頁面結構、操作流程、狀態變化、頁面連動都盡可能 1:1 的可操作版本。」

目標網站：
[把網址貼在這裡]

==================================================
最高原則
==================================================

不要憑印象做。
不要看到首頁後就開始寫。
不要只複製顏色和 Card。
不要自己重新設計。
不要把它改造成一般 SaaS Dashboard。
不要用「差不多」代替實際驗證。

你必須真的操作目標網站。

我要的是：

Visual Clone
+
Interaction Clone
+
Navigation Clone
+
Workflow Clone
+
Responsive Clone

也就是：

外觀看起來一樣，
操作起來也要像同一套產品。

如果某個流程因為第三方 API、登入權限、付款、真實資料而無法真正執行，可以使用 Mock Data / Local State / Fake Backend 模擬。

但「使用者看到的操作流程」必須完整成立。

==================================================
第一階段：先完整探索網站
==================================================

在寫任何程式之前，使用 Browser / Playwright / Chrome DevTools 實際操作網站。

至少探索：

首頁
Dashboard
主要功能頁
Sidebar
Topbar
Bottom Dock
帳號選單
設定
列表
詳細頁
新增流程
編輯流程
刪除流程
搜尋
篩選
排序
分頁
Tab
Accordion
Dropdown
Popover
Tooltip
Modal
Drawer
Toast
Notification
Empty State
Loading State
Error State
Success State
Disabled State
Hover State
Focus State
Selected State

如果網站需要登入：

先確認是否有 Demo / Public account / Preview。

如果部分功能無法登入，
至少從公開頁面、官方文件、影片、Screenshot、GitHub、Help Center 等來源補齊。

不要只研究 Dashboard。

我要「整套產品流程」。

==================================================
第二階段：建立完整 Site Map
==================================================

先把網站所有可發現 Route 建立成 Site Map。

格式例如：

/
├── dashboard
├── create
├── library
│   └── /:id
├── history
├── accounts
│   └── /:id
├── automation
│   ├── create
│   └── /:id/edit
├── reports
├── profile
├── settings
└── plans

每個 Route 要記錄：

URL
頁面名稱
入口
可以前往哪裡
可以從哪裡回來
主要 CTA
次要 CTA
Modal
Drawer
Tab
狀態
需要的資料

我要確保最後不是只有幾張互相不連通的 Mock Page。

==================================================
第三階段：建立 User Flow
==================================================

把所有主要流程畫出來。

例如：

Dashboard
→ 點擊新增
→ Create Page
→ 填入資料
→ Next
→ Select Target
→ Preview
→ Confirm
→ Processing
→ Success
→ History
→ Detail

或者：

Dashboard
→ Account
→ Detail
→ Edit
→ Save
→ Toast
→ Detail Updated

所有流程都必須分析。

每一個：

Button
Card
Navigation
CTA
Row
Menu
Icon
Dropdown

都要確認點擊後發生什麼。

禁止做：

「按鈕看起來能按，但沒有任何功能。」

==================================================
第四階段：建立 Component Inventory
==================================================

把目標網站拆成可重複元件。

例如：

App Shell
Sidebar
Sidebar Item
Topbar
Bottom Dock
Search
Command Palette
Page Header
Card
Metric Card
Feature Card
Table
Row
Status Badge
Avatar
Button
Icon Button
Tabs
Input
Textarea
Select
Checkbox
Toggle
Date Picker
Dropdown
Popover
Modal
Drawer
Toast
Tooltip
Progress
Timeline
Skeleton
Empty State
Error State

每個元件都分析：

尺寸
padding
gap
border
radius
shadow
background
hover
active
disabled
animation
icon
font
line-height
responsive behavior

==================================================
第五階段：像素級視覺分析
==================================================

不要只寫：

「它是藍色 Sidebar。」

我要你實際分析：

Page width
Max width
Sidebar width
Topbar height
Bottom Dock height
Card width
Card height
Grid column
Gap
Margin
Padding
Radius
Border opacity
Shadow
Blur
Font size
Font weight
Line height
Icon size
Button height
Input height

建立 Design Token：

--background
--surface
--surface-2
--text
--muted
--border
--primary
--secondary
--success
--warning
--danger

以及：

spacing
radius
shadow
typography
motion

如果可以從 DevTools 取得 CSS，
優先以實際 CSS 為準。

==================================================
第六階段：動態效果也要還原
==================================================

研究：

Hover
Mouse down
Focus
Selected
Active
Loading
Skeleton
Fade
Slide
Scale
Number transition
Modal transition
Drawer transition
Page transition
Menu transition
Dropdown transition

分析：

duration
easing
distance
opacity
scale

例如：

hover:
translateY(-2px)
150ms ease

modal:
opacity 0 → 1
scale .98 → 1
180ms ease-out

不能把所有動畫全部省略。

==================================================
第七階段：Responsive 必須一起複製
==================================================

至少驗證：

Desktop 1920
Desktop 1440
Laptop 1280
Tablet
Mobile 430
Mobile 390

我要確認：

Sidebar 什麼時候收合
Bottom Nav 什麼時候出現
Card 怎麼重新排列
Table 怎麼變
Modal 寬度
Padding 怎麼縮
Font 是否縮
Navigation 是否換型態

禁止只做好 Desktop，
Mobile 隨便堆成一列。

==================================================
第八階段：資料與狀態模型
==================================================

不要把畫面全部寫死。

建立合理的資料模型：

User
Account
Project
Item
Task
Status
History
Notification
Settings
等等。

如果沒有後端，
用：

Mock Data
React State
Local Storage
Fake API
Mock Service

模擬。

但是流程必須成立。

例如：

新增資料
→ 列表真的增加

修改資料
→ Detail 真的更新

刪除資料
→ 列表真的消失

切換狀態
→ UI 真的改變

搜尋
→ 結果真的過濾

Filter
→ 結果真的改變

Pagination
→ 頁面真的切換

Toast
→ 操作後真的出現

不要只做視覺假按鈕。

==================================================
第九階段：Route 連動
==================================================

所有主要頁面必須可以互相操作。

例如：

Dashboard Card
→ Detail

Detail
→ Edit

Edit
→ Save
→ Detail

List
→ Item

Sidebar
→ Page

Notification
→ Target Page

Search
→ Result

不要使用：

href="#"

除非原網站真的如此。

不要建立 Dead Link。

==================================================
第十階段：逐頁 Screenshot 對比
==================================================

完成第一輪後：

不要直接跟我說完成。

請對目標網站與本地 Clone 做 Screenshot 對比。

至少比較：

Dashboard
主要功能頁 1
主要功能頁 2
List
Detail
Modal
Drawer
Mobile Dashboard
Mobile Function Page

採用：

Reference Screenshot
VS
Clone Screenshot

逐項找：

位置不對
尺寸不對
顏色不對
字級不對
Card 太大
間距不對
Sidebar 不對
Icon 不對
Shadow 不對
Button 不對
Animation 不對

然後自己修第二輪。

==================================================
第十一階段：Visual QA
==================================================

第一輪完成後自行評分：

Visual similarity
Layout similarity
Typography similarity
Component similarity
Interaction similarity
Navigation similarity
Workflow completeness
Responsive similarity

0～100 分。

任何一項低於 90，
不要宣告完成。

繼續修。

==================================================
第十二階段：流程 QA
==================================================

用 Browser / Playwright 實際跑完整操作。

例如：

Create
Edit
Delete
Search
Filter
Navigate
Modal
Drawer
Toast
Mobile navigation

為每一個流程建立檢查：

Action
Expected result
Actual result
Pass / Fail

不要只驗證畫面能開。

==================================================
第十三階段：不要擅自美化
==================================================

這個任務不是重新設計。

所以：

不要覺得原網站某個地方不好看就自己改。
不要加入自己喜歡的 Sidebar。
不要自行換 Layout。
不要自行改顏色。
不要自行把 Card 做得比較漂亮。
不要自行把功能合併。

第一個版本的目標是：

「先忠實還原。」

等我確認 1:1 Clone 後，
我們再做 Ticenpi Optimized Version。

==================================================
如果目標網站無法完整查看
==================================================

請綜合使用：

官方網站
產品頁
Help Center
Documentation
YouTube
Product Hunt
GitHub
Blog
Release Notes
App Store screenshots
Google Images
官方 Demo

補足資訊。

但必須標示：

「直接觀察」
「官方資料」
「推測」

不要把推測當成確定功能。

==================================================
最後交付
==================================================

最後我要看到的不是一篇研究報告。

我要的是一套可以操作的 Prototype。

同時附上：

Site Map
User Flow
Component Inventory
Design Tokens
Routes
已完成流程
無法複製的部分
Mock 的部分
Desktop QA
Mobile QA
Visual Similarity Score
Workflow QA

最重要的是：

我打開本地網址後，
應該會覺得：

「這就是同一套網站。」

而不是：

「這是一套參考它做出來的網站。」

==================================================
非常重要
==================================================

整個執行順序：

1. Research
2. Browse every reachable page
3. Map Routes
4. Map User Flows
5. Capture Screenshots
6. Analyze Components
7. Analyze CSS / Design Tokens
8. Implement App Shell
9. Implement Pages
10. Implement Interactions
11. Implement State
12. Connect all Routes
13. Desktop QA
14. Mobile QA
15. Screenshot Comparison
16. Fix Differences
17. Workflow QA
18. Second Visual Pass

不要跳過 Research 就開始 coding。

不要只做首頁。

不要只做到「像」。

我要的是「完整可操作的產品級還原」。
```

<!-- END OF LUNA FANGZHONG PROTOTYPE LIVE UPDATE TASK -->
