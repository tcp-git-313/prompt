# W6-SIGN：Staging「框選簽名位置」讀取圖片失敗—瀏覽器診斷與安全續測

## 你要做的事

你是另一個可操控瀏覽器的 IDE 執行者。請接手目前 SIGN-only Staging 的合成測試案件，**直接用已開啟的瀏覽器進行唯讀診斷**，查清「已上傳兩頁 PDF，但框選簽名位置沒有圖片」的第一個失敗點。不要把一般瀏覽器操作交回使用者；只有登入、MFA、驗證碼等必須由帳號持有人完成的步驟才請使用者協助。

若只是短暫載入／簽章連結過期，且你在正常重新進入頁面後實際看見兩頁預覽，才繼續原本的 SIGN Staging E2E。若仍無圖，停在診斷結果，不硬放框、不發送簽署連結、不假報 PASS。

## 本次範圍與現有證據

- 唯一可操作站點：`https://sign-staging.ticenpi.com/`。`https://*-staging.ticenpi.com/` 只是 Redirect URL 白名單樣式，不能直接開。
- 既有合成案件：`CASE-20260927-001`，標題 `W6-E2E-20260925-SYNTHETIC`。請在已登入的 `platform_admin` Staging profile 找到這一件並核對；**不新建、不刪除、不重新上傳**。若案件不是 draft、頁數不是 2、或不是此測試案，停止並回報實況。
- 兩頁合成 PDF：`F:\00-Ticenpi-SaaS\.tmp\sign-w6-e2e-two-page.pdf`。先前用本機 `pdfjs-dist` 解析得到 2 頁；Staging 公開 PDF worker 檔也曾回 HTTP 200。這兩件事**不證明**目前瀏覽器能取回 R2 檔案或完成渲染，不能代替現場 Network/Console 證據。
- Staging release：`20260927-005459`，目前 E2E 尚未完成、驗收不是 ACCEPTED。既有完整流程在 `https://raw.githubusercontent.com/tcp-git-313/prompt/main/w6-sign-staging-browser-e2e-ide-handoff.md`；本提示只處理當前預覽 blocker，成功恢復後才接回原流程。
- 已部署原始碼的預覽路徑：`frontend-sales/src/views/FieldPlacementView.vue` 向 `GET /api/cases/:id/documents` 取得 `kind=pdf` 的 `original_url`，再由 `frontend-sales/src/lib/pdfRender.js` 用 pdf.js 在瀏覽器把 PDF 轉成頁面圖；例外被 catch 後 `imageSrc` 變成空值，UI 不一定顯示根因。案件詳情頁的「共 2 頁」只是文件 metadata，不代表預覽圖片已可讀。

## 絕對邊界

1. **只診斷 SIGN Staging**：不碰 Production、591-core、VPS、Cloudflare 設定、Supabase 後台、R2 bucket 設定、DNS、部署、CI、repo 程式碼或 release history。不要因為懷疑 CORS 就自行修改 Cloudflare/R2；任何後台寫入由使用者另行授權和執行。
2. 不呼叫 service-role、不用 SQL Editor、不自行取得／重放 bearer token。不改瀏覽器安全設定或擴充功能權限，不繞過已拒絕的自動化 API。
3. **禁止輸出機密**：不要複製完整 `original_url`、R2 presigned URL、URL query/signature、OAuth callback/hash、`/s/<token>`、Authorization header、cookie 值、access/refresh token、完整 HAR。只報 host、路徑形狀（移除 case UUID 與 query）、HTTP 狀態、必要的非敏感 response header 與錯誤文字。截圖前遮罩網址列和 Network 的完整 Request URL。
4. 不使用 `samples/` 或真客戶文件。不得為了讓測試過關，轉成其他格式、刪除重傳、重建案件、繞過 UI 或在看不到文件時亂點簽名框。
5. 如看到 Production Supabase ref `zfpkxsulehkbyuqkkcml`，或任何環境看到舊 ref `pygkhgkbmsczkgceifqo`，立刻停止並回報。正確 Staging ref 是 `jlsqjvehwblkeuycjoyj`。

## 具體操作：請照順序執行

### 1. 確認頁面

在 admin 的 Staging 瀏覽器 profile 開 `CASE-20260927-001` 詳情，記錄顯示的狀態、文件名稱／種類、上傳頁數；進「框選簽名位置」確認是第 1、2 頁都無圖，還是只有其中一頁。不要在破圖／空白圖上點擊放框。若需要重新登入，只讓使用者本人完成 Google/MFA，之後由你繼續。

### 2. 捕捉一次乾淨的瀏覽器證據

開啟 DevTools 的 Network 和 Console，Network 啟用 Preserve log；在仍登入的前提下，從案件詳情**正常重新進入**「框選簽名位置」，必要時只重整一次。不要人工開啟、分享或重放帶簽章的 R2 URL。

檢查並只回報以下安全欄位：

1. `GET /api/cases/<case-id>/documents`：HTTP status；回應中是否有測試案的 PDF 文件；`kind`、`page_count`、`document_pages.length`；`original_url` 是「有／無」，**絕對不要抄值**。如 API 失敗，記 status 和可見的非敏感錯誤碼。
2. 若 `original_url` 有值，Network 是否出現向 R2／儲存空間**取原 PDF**的請求：只記**遮罩帳戶識別碼後的 host**（例如 `<account>.r2.cloudflarestorage.com`）、HTTP status 或瀏覽器顯示的阻擋原因、`Content-Type`、`Access-Control-Allow-Origin` 是否存在及是否允許 `https://sign-staging.ticenpi.com`；不要記路徑、query、簽章或完整 response body。
3. pdf.js 主 chunk／worker 請求是否成功（只記檔案類型、status）；Console 的**第一條相關錯誤**和錯誤類型，例如 CORS、403、404、Failed to fetch、InvalidPDF、worker 初始化、canvas。貼錯誤前移除任何完整 URL、token、query 或個資。
4. 直接觀察第 1、2 頁是否實際顯示內容，以及 `<img>` 是有正常圖還是破圖／空白。不能因頁數 label 出現就判圖片載入成功。

若工具會自動把完整 Network JSON、HAR、Request URL 或 session 輸出到對話，**不要使用那個匯出方式**；改以你自己讀取後回報上述少量安全欄位。

### 3. 用證據分類，不猜根因

- `/documents` 非 200、文件／頁面 metadata 異常或 `original_url` 缺失：記為 API／文件資料路徑問題，不碰資料庫或重傳。
- 原 PDF 請求為 403／404／簽章過期：只允許從案件頁**正常重新進入一次**，讓應用程式取得新 presigned URL；若仍失敗，停止。不得手工修改、延長或重放簽章網址。
- Network 顯示 CORS 阻擋或 PDF 請求雖有 HTTP 200、但缺少允許 Staging origin 的 CORS header：記為「疑似 R2 CORS」，附瀏覽器錯誤與 header 證據；**不要自行改 R2/Cloudflare**。
- 原 PDF 可由瀏覽器成功取得且 CORS 正常，但 pdf.js chunk／worker 或解析、canvas 報錯：記為前端預覽渲染路徑問題，附第一條 Console 錯誤；不要以本機解析成功否定瀏覽器錯誤。
- 上述請求都正常但仍無圖：記 `ROOT_CAUSE=UNKNOWN`，附可見證據，不自行猜測或亂改。

### 4. 只有預覽真恢復才續測

如果正常重新進入後，**兩頁內容都清楚可見**，確認仍是 draft、仍是原合成案件，才依前述完整 E2E 提示詞在每頁放 1 個簽名框、儲存、重新讀頁確認兩框持久存在，接著發送測試連結並續測。若這次只是偶發恢復，照實註明曾失敗及重試次數，不把問題藏起來。

如果仍無圖，`TWO_FIELDS_SAVED=BLOCKED`、`SEND_LINK=NOT_RUN`、`STAGING_E2E=PARTIAL`、`STAGING_RELEASE_ACCEPTED=NO`。不要做任何後續簽署或把「健康檢查／上傳成功／頁數顯示」拿來補 PASS。

## 交付格式

用繁體中文、先白話說明結論，再填：

```
CASE_NO = CASE-20260927-001
STAGING_HOST = sign-staging.ticenpi.com
CASE_STATUS =
DETAIL_PAGE_COUNT =
FIELD_PAGE_1_VISIBLE = YES/NO
FIELD_PAGE_2_VISIBLE = YES/NO
DOCUMENTS_API_STATUS =
PDF_METADATA = kind:<值>, page_count:<值>, document_pages:<數字>, original_url:PRESENT/MISSING
PDF_FETCH_HOST = <遮罩帳戶識別碼後的 host；沒有請求就 NONE>
PDF_FETCH_STATUS_OR_BLOCK =
PDF_RESPONSE_CONTENT_TYPE =
PDF_CORS_ALLOW_STAGING = YES/NO/UNKNOWN
PDFJS_CHUNK_WORKER = <各 status，未觀測則 UNKNOWN>
FIRST_RELEVANT_CONSOLE_ERROR = <已遮罩；沒有則 NONE>
DIAGNOSIS = API_METADATA / STORAGE_FETCH / R2_CORS_SUSPECTED / PDFJS_RENDER / UNKNOWN / TRANSIENT_RECOVERED
RETRY_COUNT =
TWO_FIELDS_SAVED = PASS/BLOCKED/NOT_RUN
SEND_LINK = PASS/NOT_RUN
STAGING_E2E = PASS/PARTIAL/FAIL
STAGING_RELEASE_ACCEPTED = NO（本任務無權自行核准）
BLOCKER =
ROOT_CAUSE = <證據足夠才寫；否則 UNKNOWN>
SMALLEST_NEXT_ACTION =
EVIDENCE = <遮罩後截圖或觀測時間與狀態；不含 signed URL、token、cookie>
```

若同時有其他角色測試已做，也只能逐項附上實際證據，不能把它們當成此預覽問題的修復。切勿在回報中貼出完整 Network Request URL 或任何簽章參數。
