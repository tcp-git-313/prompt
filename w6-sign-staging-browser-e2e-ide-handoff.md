# W6-SIGN：交由另一個 IDE 執行 Staging 瀏覽器 E2E

## 任務與權限

你是執行者，請使用你可用的瀏覽器操控能力，接續 **SIGN-only Staging** 的真人介面流程。使用者已授權 SIGN Staging 測試；請自己完成可自動完成的瀏覽器操作與取證，只有登入、MFA、驗證碼等真正需要帳號持有人介入時才請使用者操作。不要請使用者重做你能做的點擊、選檔、檢查或截圖。

這是瀏覽器 E2E 交接，**不是**改程式、修 CI、部署或正式驗收授權。你完成後提供可核對的證據，原 SIGN release 執行者再更新矩陣並判定是否 ACCEPTED。不得自行執行 `record-accepted`，不得把健康檢查、CI 或單一成功畫面當作全套 E2E PASS。

**唯一可操作站點：** `https://sign-staging.ticenpi.com/`。`https://*-staging.ticenpi.com/` 是 Supabase Redirect URL 白名單樣式，不是真正可開啟的網址；不要把星號輸入瀏覽器。Staging Supabase ref 是 `jlsqjvehwblkeuycjoyj`，若實際頁面、登入導向或 API 指向 Production ref `zfpkxsulehkbyuqkkcml`，立刻停止。任何環境遇到舊 ref `pygkhgkbmsczkgceifqo` 也停止。

## 已有現況：先唯讀重驗，不要照抄成 PASS

- SIGN Staging release `20260927-005459` 已部署；此任務只驗瀏覽器，不重部署、不碰 591-core。現有矩陣位於 `F:\00-Ticenpi-SaaS\.release-evidence\sign\artifacts\20260927-005459\e2e-matrix.md`，目前整體 `PARTIAL`。
- `platform_admin` 已用 Chrome 登入過，並建立合成測試草稿 `CASE-20260927-001`，標題 `W6-E2E-20260925-SYNTHETIC`。請在已登入的 Staging 案件清單依案號／標題找這一件；先確認仍是 draft、已上傳頁數。不要無故另建一件，也不要刪除或修改既有非測試案件。
- 兩頁、無個資的合成 PDF 已備妥：`F:\00-Ticenpi-SaaS\.tmp\sign-w6-e2e-two-page.pdf`。先確認檔案存在、可讀且確為兩頁，再用瀏覽器正常選檔／上傳。不得使用 `samples/` 或真客戶文件。
- Staging 中 SIGN 商用夾具已經建立並唯讀複查：business Customer 的 SIGN entitlement 為 active、`seat_limit=2`；指定成員 A 已 assigned、成員 B 未 assigned。**不要重建或改動 entitlement、Seat、membership、角色或 test_access**；不要碰其他產品。詳見既有 `fixture-snapshot.md`。測試身分以 `F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TEST_IDENTITY_CONTRACT.md` 的 A/B 角色為準；該文件 §3 是較早的歷史快照，不可用來推翻目前的 `fixture-snapshot.md`。
- 現行計畫瀏覽器驗收規格見 `F:\_prompt-publish\W6-SIGN-luna-executor-plan.md` §16。若本提示與該計畫的權限／HARD_STOP 衝突，以計畫為準。
- 先前一個瀏覽器自動化介面的檔案選擇呼叫回了 `Not allowed`，**這只證明該介面被拒，不證明檔案不存在，也不證明使用者沒開擴充功能權限**。使用你 IDE 正常支援的檔案選擇器或瀏覽器操作；不得修改權限、繞過安全限制、偽造上傳成功。若仍被拒，記下準確錯誤並停止該路徑。

## 禁止事項與資料保護

1. 不碰 Production：不登入、不部署、不寫入、不操作正式簽署；不碰 VPS、Cloudflare、DNS、Supabase 後台或 SQL Editor。不得操作 591-core、591 專案或其他產品。
2. 不改任何 repo 原始碼、CI、設定、release history 或既有驗收矩陣；不 commit、不 push、不跑 release acceptance 命令。只允許在既有合成 SIGN Staging 測試案件及必要的成員 A 合成案件上完成 UI 流程。若要保存證據，可在上述 release 的 artifacts 下新增**經遮罩**的 browser-e2e 證據檔，不覆寫既有檔案；也可只在回覆中交付證據。
3. 不使用 service-role、SQL 直寫、私有 API 代替正常使用者流程；不讀、不複製、不貼出 OAuth callback 完整網址、URL fragment、access/refresh/provider token、cookie 值、Authorization header、簽署連結的 token 或任何秘密。即使瀏覽器／工具輸出自動包含這些資訊，也必須先遮罩再回報。簽署連結只能在本次測試的瀏覽器內轉交至無痕視窗，不得貼進 GitHub、截圖、聊天或日誌。
4. 所有案件、文件與簽名都須明確為**合成測試資料**；簽名可畫 `TEST`／測試筆劃，不得冒用真人簽名。不要向任何真實客戶發訊息。若 UI 要求填收件者，僅用已核准的測試身分；無法確認時停下，不猜地址。
5. 不自行變更瀏覽器安全設定、擴充功能權限或登入帳號。若需要 Google 登入／MFA／驗證碼，先停在該畫面，請使用者本人完成，之後你再接手驗證。不要把 admin 帳號拿來充當 assigned／unassigned Seat 測試。

## 執行順序

### 0. 確認環境與基線

在目前 Chrome 視窗確認實際 host 是 `sign-staging.ticenpi.com`，顯示的登入身分是 `platform_admin`，既有合成案件存在且狀態、頁數與上述一致。若瀏覽器並未共享原本登入狀態，請使用者完成登入；不要取用或輸出 session token。先記下時間、URL **不含 query/hash/token**、案件案號／狀態、Staging release 識別。若原案件已被其他人改動，先唯讀記錄，停止涉及該案件的寫入並回報。

### 1. Admin：上傳、框選、發送

1. 在 `CASE-20260927-001` 的草稿詳情頁，按「上傳圖片/PDF」，選 `F:\00-Ticenpi-SaaS\.tmp\sign-w6-e2e-two-page.pdf`。該 UI 先把檔案放「待上傳佇列」；還要按「上傳 1 份檔案」。等待完成並確認「已上傳文件（共 2 頁）」及兩頁預覽。只見檔名進佇列、沒有伺服器儲存證據，不能算 PASS。若頁數非 2 或上傳錯誤，先記錄 UI 錯誤與可見的 HTTP 狀態；不要重複亂傳。
2. 進「框選簽名位置」，在第 1、2 頁各放 1 個框（共 2 個），按「儲存簽名框」，重新讀頁確認兩框都仍在，位置合理且未蓋住測試文字。不要只看暫時畫面。
3. 回案件詳情頁，確認案號、兩頁、兩框，按一次「發送簽署請求」。此操作只針對合成案件。記錄 UI 的 `sent` 狀態、連結效期與是否顯示簽署連結；**不得在回報中包含連結 token**。如果點擊後沒有明確成功結果，先查明狀態，不要直接按「重新產生連結」或重複發送。

### 2. 匿名簽署：完整正向與負向

在**新的無痕視窗／隔離 profile** 打開剛才產生的 Staging 簽署連結，保持不登入。確認兩頁顯示正確、兩處簽名標示可見；依 UI 開啟簽名板，畫合成測試筆劃，完成、在最終預覽看到簽名套用至兩個位置，再按「最終確認，送出」。等待背景 PDF 合成，確認最終狀態與「下載已簽署文件」，實際下載並以 PDF 檢查頁數為 2、兩處測試簽名可見。若處理仍在進行，只記 PENDING，不把 `202` 或 spinner 當完成。

負向測試：在送出前後選擇安全時點，複製測試連結於**同一個測試瀏覽器內**，對 token 做一字元竄改，確認拒絕；提交完成後重開原連結，確認不能再次簽署／重送。只回報 token 已遮罩的路徑形狀 `/s/<redacted>`、可見錯誤與 HTTP 狀態。不得用隨機掃描或嘗試他人的連結。

### 3. 成員 A/B：Seat 與案件隔離

使用彼此獨立的瀏覽器 profile，依本地 Staging Test Identity Contract 確認哪個是指定成員 A、哪個是 B；如需登入，請使用者完成登入／MFA，你繼續測。不要從 email 字樣或列表順序推測 user_id，不變更 Seat。

- A（assigned）：登入後 `GET /api/cases` 應為 200 且可看到自己的清單；透過 UI 建一件**明確合成**的草稿案件以驗可建案，保留案號。A 直接請求 admin 合成案件詳情時，應 404 或 403，且不能看到 admin 案件資料。只用瀏覽器正常導航／UI 或開發者工具的已發請求結果，勿另造 bearer token 或調私有資料。
- B（unassigned）：身分登入可成功，但 SIGN 的案件入口／`GET /api/cases` 應 403、錯誤碼 `SEAT_NOT_ASSIGNED`，UI 不應載入案件資料。若 B 看到資料，這是安全性 FAIL：立即停止其他寫入、保存**遮罩後**證據並回報。
- 登入身分若不是合約指定的 A/B，停止該角色測試；不得借用 admin 或其他產品帳號代替。

### 4. Cookie 隔離與選做項目

依計畫，在 Staging 完成 3 次登入／登出週期（可分用不同隔離 profile，但逐次確認身分與結果），檢查 Staging 操作未新增 domain=`.ticenpi.com` 的 `sb-` cookie。不要讀出或報告 cookie 值。Production cookie「未變」只有在已有可比較的安全基線、且可純本機唯讀比較 cookie 名稱／domain 等非敏感 metadata 時才可判 PASS；**不得為了此項去登入、瀏覽或操作 Production**。若沒有可靠基線，標 `PENDING_BASELINE`，不可臆測。

`platform_test_allow/deny` 是選做 U6；若測試身分未建立，記 `NOT_RUN (E2, platform fixture missing)`，不自行建立或替代。

## 證據與停止規則

每一項都要有：`PASS | FAIL | PARTIAL | BLOCKED | NOT_RUN`、觀測時間、操作的 Staging host、身分角色（不含 email）、案件案號、可見 UI 結果、必要時 HTTP status／錯誤碼，以及遮罩後截圖或證據路徑。HTTP 200、健康檢查與 CI 不可代替瀏覽器結果；登入頁正常也不等於 Seat 驗證成功。下載須檢查 PDF 真正兩頁與簽名，不可只看按鈕。

遇到 Production ref、舊 ref、真客戶資料、`samples/`、未知帳號、既有案件非預期變更、跨案件資料外洩、需要擴大權限／改配置／改程式、瀏覽器檔案選擇被拒、或 token／cookie 可能外洩：立即停止受影響路徑，列出 `BLOCKER / ROOT_CAUSE(可確定才寫) / EVIDENCE / SMALLEST_NEXT_ACTION`。不要靠重試、跳過 job、改 fixture 或假資料截圖湊 PASS。沒有需要真人登入／MFA 的事就繼續工作，不要把一般瀏覽器操作丟回使用者。

## 回報格式

請用繁體中文，先說**已完成什麼、還卡什麼**，再逐項列：

```
STAGING_HOST =
RELEASE_ID = 20260927-005459
ADMIN_CASE_NO = CASE-20260927-001
ADMIN_UPLOAD_2_PAGES = PASS/FAIL/PARTIAL/BLOCKED
TWO_FIELDS_SAVED = PASS/FAIL/PARTIAL/BLOCKED
SEND_LINK = PASS/FAIL/PARTIAL/BLOCKED
ANONYMOUS_SIGN_PREVIEW_SUBMIT = PASS/FAIL/PARTIAL/BLOCKED
SIGNED_PDF_DOWNLOAD_2_PAGES = PASS/FAIL/PARTIAL/BLOCKED
TOKEN_TAMPER_AND_REPLAY = PASS/FAIL/PARTIAL/BLOCKED
MEMBER_A_CASES_AND_ISOLATION = PASS/FAIL/PARTIAL/BLOCKED
MEMBER_B_403_NO_DATA = PASS/FAIL/PARTIAL/BLOCKED
COOKIE_ISOLATION = PASS/FAIL/PARTIAL/BLOCKED/PENDING_BASELINE
FIXED_ALLOW_DENY_E2E = PASS/FAIL/NOT_RUN
EVIDENCE = <逐項、遮罩後的時間／截圖路徑／HTTP 狀態／案號>
BLOCKER = <若有>
ROOT_CAUSE = <有證據才填；否則 UNKNOWN>
SMALLEST_NEXT_ACTION = <若有>
STAGING_E2E = PASS/PARTIAL/FAIL
STAGING_RELEASE_ACCEPTED = NO（本任務無權核准；交回原執行者判定）
```

不要在回覆、截圖、檔案或工具日誌保留簽署 token、session、cookie 值、OAuth URL fragment、客戶個資或正式環境資料。若某項未真正觀測，直寫未測／阻塞原因，不得以推測填 PASS。
