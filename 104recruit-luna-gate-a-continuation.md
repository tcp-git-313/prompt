# 104Recruit — LUNA 接續計畫與嚴格執行提示詞

版本：2026-10-23 / Gate A continuation v1
執行模型：GPT-6 Luna，thinking level：Max（沿用使用者指定 LUNA，不自行切換模型）。

你是本專案的 Executor。先核實工作現況，再依本計畫完成 GATE A。你不是重新設計架構的 Architect。本輪不是從零建置，也不是再寫一套 planning 後停止。

## 1. 唯一目標、授權與停止點

固定工作根目錄：`F:\AI\_104Recruit`。所有產物、依賴快取、temp、profile、DB、log、測試與 evidence 都留在此目錄。

本輪只完成：真人 104 企業登入 → 真實 Session → 企業自己的 Job List → 明確 Select Job → 同一 Job Detail，逐步驗收後 GATE A PASS。P12 完成後停止，更新報告。**不得自動進 P13、Candidate Search、AI、Queue、VOIP、CRM、HUB、Production。**

目前架構固定：Node22、TypeScript ESM、native node:http、靜態 HTML/CSS/JS Console、Playwright 已安裝 Chrome、node-postgres、獨立 PostgreSQL17。沿用 lockfile，不換 React/Next/DB/Queue，不安裝全域套件，不新增付費服務。

這份計畫是使用者要求重新安排 LUNA 接續工作所產生的局部修訂；只修正 GATE A 的接手流程、必要接線、驗收與 allowlist 漏項。安全規則、資料模型、既有 enums、Gates 和架構不變。

權威順序：使用者最新明確指示 > 本文件的 GATE A 接續條款 > MASTER-PLAN > 14 decisions > 13 boundaries > 10 phases > 其他 planning。衝突只在本文件明列範圍局部覆蓋，不得藉此忽略其他限制。

## 2. 接手時必須重驗的快照

以下是 2026-10-23 接手快照，不保證之後仍相同：

- Branch：`codex/bootstrap-gate-a`。
- 最後已 commit 且驗收的 Phase：P4；HEAD：`26b0c620929a859071933ede41c05b1adfec8778`。
- P0–P4：已有本機 acceptance 和 local commits，不重做、不回滾。
- Current Phase：P5；Status：`WAITING_FOR_USER_LOGIN`；GATE A 尚未通過。
- P5 最近 local checks：lint、typecheck、4 unit、5 integration、build 全 PASS。這不是登入成功。
- 已真實觀察公開入口 `https://vip.104.com.tw/` 的單一登入連結導向 `https://bsignin.104.com.tw`，且 password field 存在。只驗證到登入頁，不是已登入。
- 所有企業內部 Session/Jobs/Detail endpoint、JSON paths、公司權限仍須 live inspection，不得猜。
- App：`http://127.0.0.1:43104/recruiting/debug`；DB：`127.0.0.1:55432/recruit104`；profile：`data/browser-profiles/104-recruiter`。
- 既有共享5432/Docker/Windows services/其他 Chrome profiles 均不得修改或停止。

**P5 產品 WIP 尚未 commit，必須保留：**

|檔案|狀態|
|---|---|
|src/server.ts|modified|
|public/app.js|modified|
|src/adapter104.ts|untracked|
|scripts/open-login.ts|untracked|
|tests/integration/login.test.ts|untracked|

另外有執行狀態、Build Report、Manual Login Handoff 及本輪接續文件的 docs 變更；不要當成無用檔清掉。精確 SHA256 和原始 Git 狀態見 `docs/planning/LUNA-HANDOFF-SNAPSHOT.json`。雜湊不同先讀 diff，保留較新的正常工作，不要直接報架構偏差。

## 3. 必讀與唯讀 preflight

先讀 applicable AGENTS，再完整讀：

1. `docs/planning/MASTER-PLAN.md`、本文件、`BOOTSTRAP-SPEC.md`。
2. `docs/planning/00-CURRENT-STATE.md` 至 `14-DECISION-LOG.md`、`ARCHITECTURE-REVIEW.md`。
3. `EXECUTION-STATE.md`、`LUNA-HANDOFF-SNAPSHOT.json`。
4. `docs/recruiting/BUILD-REPORT.md`、`MANUAL-LOGIN-HANDOFF.md`。
5. `src/config.ts`、`domain.ts`、`browser.ts`、`adapter104.ts`、`server.ts`、`debug.ts`、`db.ts`、public、相關 tests、scripts、manifest/lockfile。
6. `data/acceptance/phase-history.jsonl`、P4/P5 checks 和 P5 login-page evidence；不要讀出/列印 browser credential/session 檔案。

先執行只讀現況確認：`git status --short`、`git diff`、HEAD/branch、檔案存在/雜湊、Node版本、43104/55432 listener 與 owner、local health。建立 `data/acceptance/luna-preflight-<timestamp>.json`，只放非敏感身份與檢查結論。

不 reset、clean、stash、checkout 覆蓋、不 broad stage、不搬移專案、不重新 clone 取代現有目錄。不要執行 `scripts/create_planning.py`：planning 已完成，此 generator 會拒絕覆蓋，不是接手指令。

如服務已停止：只用本專案 `scripts/db.ps1 start` 啟動其55432 cluster；依 lockfile 補齊缺失依賴；build 後啟動本機 app。啟動前確認 port owner，不殺未知程序，不重啟共享 DB/Docker/WSL。若 profile 已由 app 的 BrowserManager 持有，透過同一 app 操作，禁止另一個腳本再 launch 同一 profile。

需要更新 server 時，先確認它屬於本專案、透過既有 close API 正常保存專用 Chrome，再只停止該 app 並以新 build 重開。保留 profile；不可 reset 有效登入來方便測試。若使用者正在填登入資料，先讓其完成，不中斷表單。

開始接手先用數行回報 CURRENT / NEXT / 已保留 WIP，接著直接工作，不要求再次批准已授權的正常施工。

## 4. 已知程式斷點與最小修復方向

下列是 code inspection 的現況，不能誤稱已完成：

1. `Recruit104Adapter` 目前只實作 openLogin；沒有 checkSession/listJobs/getJob。openLogin 回傳 WAITING 是保守 workflow，不是已認證 Session。
2. `DebugRunner.sessionAction` 尚未接線，run 目前只支援該 hook；其他 steps 是 PHASE_NOT_IMPLEMENTED。必須按 Phase 增加對應 handler，不能只改 UI 的 PASS。
3. `/api/debug.gateA` 目前固定 NOT_RUN；只在 P12 接上真實 acceptance 狀態，不能提前改常數為 PASS。
4. `sourcePage` 選第一個104 tab；多 tab 可能仍指向舊入口/登入頁。P5 觀察時須檢查同一 context 的相關可見頁，分清真正企業後台，不能再點登入把已登入狀態帶走。
5. 舊10的部分 allowlist 漏列 server/debug/UI 的必要 wiring；本文件的各 Phase allowlist 明確補齊，**不是改架構許可**。
6. Debug run 現在先寫 DB 最終狀態再寫 evidence；在 P6 首次允許真實 PASS 前，修正保存失敗仍可能留下綠燈的問題，並加入失敗回歸測試。
7. 重跑 Session/更換職缺/重開 app 時，舊 PASS 不能無條件沿用。使用 metadata 中的非敏感 contextEpoch/selection fingerprint，保留歷史但讓失效的下游結果不算當前 PASS。

不得為上述接線重寫整個 server、browser、UI 或資料模型。只加入小型方法、handler registry 與狀態失效邏輯。

## 5. 共用工程契約

### Session 與結果 enum

Session 只用 `VALID / LOGIN_REQUIRED / EXPIRED / BLOCKED / UNKNOWN`。`WAITING_FOR_USER_LOGIN` 只屬於 execution/workflow，不是 Session enum。

Debug 只用 `NOT_RUN / RUNNING / PASS / FAIL / BLOCKED`。沿用 domain.ts 的 ErrorCode，必要原因用 evidence 的 reasonCode，不自行發明近似 enum。

### 本輪 API 補充與固定路由

既有 native HTTP envelope、Host/Origin、CSRF token、JSON/body限制、安全訊息、loopback/CSP 保留。不得提供任意 URL、任意 JS 或任意檔案讀取 API。

- P5 新增 `POST /api/104/login/inspect`，input 僅 `{}`。只觀察**已由本專案持有**的 context；沒開啟則回傳 LOGIN_REQUIRED workflow，不自動 launch/導航/讀密碼。
- inspect output：`{workflowState, loginUiState, observedAt, evidenceId}`；loginUiState 僅 `AUTH_FORM / AUTHENTICATED_UI / CHALLENGE / UNKNOWN / NOT_OPEN`。這是 UI 觀察分類，不能當作 SessionState。workflowState 是 WAITING_FOR_USER_LOGIN，或 AUTHENTICATED_UI_OBSERVED（仍需 P6驗Session）。
- `AUTHENTICATED_UI` 只由實際觀察到的企業後台特有可見 UI 確認；例如企業工作區/公司切換器/登出操作等多項正證據。具體 selector/文字先觀察才寫，不猜；只有104網域、標題、password欄位消失不能通過。
- P6：`POST /api/104/session` `{}` → SessionResult；並接 Debug Step01。
- P8/P9：`GET /api/104/jobs` → JobSummary[]；P9 `POST /api/104/selection` `{jobId}` → selected summary。
- P11/P12：`GET /api/104/jobs/:jobId` → JobSnapshot；jobId 必須是當前授權企業清單中的有效選擇。
- `POST /api/debug/run` 維持 `{step:1..11}`，此輪只實作01–03；04–11繼續鎖住。

### 保存、一致性與測試隔離

每次 actual run 都留下 runId、contextEpoch、step、時間、status/error 和 evidence reference。epoch 是本機隨機工作版本，不是104 token。企業/職缺識別在公開或匿名 evidence 用雜湊/代號，不暴露帳號/聯絡資訊。

Debug durable流程：先插 RUNNING → 執行得到 outcome → 原子寫入匿名 outcome evidence → DB 最終更新連同 evidence path/hash → 兩者成功才回傳 accepted PASS。檔案中的 execution outcome 不是 Gate closure；DB 最終寫入失敗保持未驗收，重啟標記中斷。寫 evidence 失敗必須將 run 標記 FAIL（若 DB 仍可用），不能留下當前可接受 PASS。注入 evidence write failure 和 DB finalization failure 測試，不必改 DB schema，可用既有 metadata。

Session失效/close/reset/app新instance/企業範圍變更，使舊 Step01–03結果不計當前成功。職缺 refresh/selection變更使舊Detail不計成功。歷史紀錄不刪。以當前有效 context/selection 導出結果；不能單看每個 step 最後一個歷史 PASS。

local/synthetic/fixture connector 名稱與 source104結果隔離。不可讓 unit/synthetic tests 建立 live Gate PASS。

### 每個 Phase 的程序

讀當前Phase → 確認前置/allowlist → 最小施工 → lint → typecheck → unit → integration → build → Phase acceptance → 更新 EXECUTION-STATE/BUILD-REPORT → PASS才下一個。

可用 `npm run check:phase -- Pn` 跑現有完整檢查鏈；live acceptance 另外記錄，不能用 checks.allPassed 代替。每次保留時間戳 attempt，不覆蓋歷史；只有最新索引檔可更新。不要因下一輪只是延續就把舊 PASS 清零重做，只對變更與必要回歸重驗。

每個 Phase 開始記 Objective/Dependencies/Allowed/Forbidden/Tests/Acceptance，結束記 Files/Result/Evidence/Errors/Next。輸出必須區分 LOCAL_CHECKS_PASS、LIVE_ACCEPTANCE_PASS、WAITING、BLOCKED。

## 6. P5 — 完成人工登入及接手驗收

前置：preflight完成、P4 evidence存在、保留全部P5 WIP。

Allowed：src/adapter104.ts、server.ts、browser.ts（僅context安全觀察介面）、debug.ts、public/app.js、scripts/open-login.ts、相關 login tests、狀態/報告/evidence。不得動其他產品。

任務：
1. 先讀當前登入狀態；不要假定使用者尚未或已完成登入。
2. 若仍是登入表單，維持 WAITING_FOR_USER_LOGIN，讓使用者在專用 Chrome 輸入。不要要求提供密碼/MFA，不在另一個profile代登入。
3. 加固定 inspect helper 與必要 UI workflow 提示，僅檢查已開啟context，不重新導航已登入頁。
4. 若 browser未開且user需要登入，使用既有openLogin。只有被觀察到的官方連結可導向，不編造auth URL。
5. 使用者回覆完成後，以實際可見企業後台 UI 證據驗收P5；反之保持等待。使用者一句「完成」本身不是自動PASS。
6. 跑 login/redaction/API input/CSRF 與現有檢查。非必要不關閉已登入的realprofile。

Acceptance：人工登入完成 + 真實企業後台正證據 + 該Phase檢查通過。登入頁正常只算 launch capability PASS。完成才 commit P5 owned files 的 checkpoint並進P6；等待時只整理P5，不提前施工P6/Jobs。

## 7. P6 — 真實 Session Health Check

前置：P5真正PASS。Allowed：src/adapter104.ts、server.ts、debug.ts、domain.ts（僅現有DTO一致性，不改enum）、public/app.js（Session提示/必要接線）、session tests、共用狀態失效/保存測試。

任務：
1. 實作checkSession與既定POST route、接Debug Step01。
2. VALID需「企業後台正UI證據 + 一次正常權限內的真實authenticated read」。可使用目前觀察到的後台document重新載入/正常UI觸發讀取，不依賴尚未發現的job endpoint，因此不與P7形成循環。
3. login form→LOGIN_REQUIRED；同一context曾真VALID而回登入→EXPIRED；明確挑戰/拒絕→BLOCKED；無法判定→UNKNOWN。未知/網路錯誤不可解讀成VALID。
4. 確認對應企業範圍可識別。若沒有企業權限，記SOURCE_BLOCKED，不改用求職者公開職缺假冒企業清單。
5. 完成第5節保存一致性與epoch失效，修正這些現有接線缺口才允許任何真實PASS。

測試：real positive1次；其餘state與無網路/drift用明確synthetic fixtures，不能登出/清空realprofile來測負例；注入保存失敗；Session失效後下游舊PASS不能算有效。

Acceptance：真實Session VALID、Debug01 live PASS、負例/保存/失效測試與完整check chain通過。

## 8. P7 — Job List Endpoint Discovery

前置P6 PASS。Allowed：docs/source/job-list-discovery.md、scripts/discover-jobs.ts、adapter104.ts 的只讀觀察hook、必要匿名schema evidence。

透過企業正常UI進入公司職缺，觀察它自然發出的XHR/Fetch JSON；其次Embedded JSON、stable semantic selector、DOM。只使用此帳號正常有權查看的企業與職缺。

先發現再寫parser。記method、匿名path template、query鍵名（不存值）、status、企業範圍來源、JSON結構/欄位path、pagination/filter/total語義。API URL/response 未見前一律 UNKNOWN_UNTIL_LIVE_INSPECTION。

禁止保存headers/cookies/token/query值/完整原始網路包/HAR。原始body只暫存在記憶體檢查，寫fixture前只保留授權且匿名的job欄位，HR電話/email也遮蔽。

Acceptance：實際UI操作和request/response可對照、scope與欄位語義已確認，已足以寫P8；若無權限/來源挑戰/找不到endpoint就BLOCKED current phase，不能猜。

## 9. P8 — Job List Parser

前置P7 PASS。Allowed：src/parsers/jobs.ts、adapter104.ts、server.ts（job list GET wiring）、tests/fixtures/jobs.json、job parser/API tests。

輸出沿用domain JobSummary：jobId/title/status/location[]/updatedAt。ID/title缺失fail；optional null必須對應已觀察source-null/not-exposed，不可把parser失敗寫成null成功。實測合法空清單、schema drift、缺ID/title、重複、pagination語義和scope錯誤。

listJobs必先確認當前Session有效；逾時/403/429/schema變更分開回報，不把error response當空清單。不得直接重試403/429或繞過權限。

Acceptance：匿名fixture PASS + 真實授權清單parser與UI內容對照正確。若企業確實0職缺，可記清單讀取成功但Selection/Detail因無測試職缺BLOCKED，不補造job。

## 10. P9 — Job List UI 與明確 Selection

前置P8 PASS。Allowed：public/app.js、index.html/style.css（必要清單顯示）、server.ts、debug.ts（Step02 handler）、job selection state helper、job UI tests。

Refresh Jobs顯示實際來源數量與scope/filter資訊，提供Select Job；backend驗證jobId存在當前授權清單，選擇後清除舊Detail。Session/企業/清單變更時按第5節失效，不保留不同context綠燈。

不先實作P10–12的Detail parser。job卡片/明細空狀態應說明尚未驗證，而非塞示範資料。

Acceptance：真實Jobs count與UI相符、選中真實job、backend/Console選擇一致、stale selection測試、Debug02 live PASS。

## 11. P10 — Job Detail Discovery

前置P9 PASS。Allowed：docs/source/job-detail-discovery.md、scripts/discover-detail.ts、adapter104.ts的只讀觀察hook、匿名schema evidence。

在同企業範圍點選同job，記實際detail request/response及所有所需欄位source path；保留scope/job identity關聯而不輸出敏感資料。禁止從公開求職者職缺頁替代企業後台detail，除非正式計畫另經使用者決策。

Acceptance：同一jobId的真實UI/request/schema對照已確認，足以寫P11；沒有拿別job結果湊數。

## 12. P11 — Job Detail Parser

前置P10 PASS。Allowed：src/parsers/job-detail.ts、adapter104.ts、server.ts（detail GET wiring）、匿名detail fixtures與parser/API tests。

normalized沿用JobDetail全部欄位：jobId/title/description/location[]/salary/experience/education/skills[]/requirements[]/benefits[]/schedule/employmentType；保留sanitized rawSnapshot與normalized分離、parserVersion、observedAt。

每欄記 PASS / SOURCE_NULL / NOT_EXPOSED / PARSE_FAIL 的field comparison evidence（這是驗收欄位分類，不是新Session/Debug enum）。SOURCE_NULL/NOT_EXPOSED必須有UI/API觀察依據，PARSE_FAIL不能用null掩蓋。描述如含HR聯絡資訊，匿名fixture/log/evidence必遮蔽，記redaction，不抹掉語義後假稱逐字一致。

測試缺欄位/HTML片段/null/空陣列/錯job/scope漂移；不得靠大量脆弱CSS或隱藏HTML私人欄位補數據。

Acceptance：同一真實job所需欄位全部正確或可證明source-null/not-exposed；required identity不缺，局部parser錯誤不宣告整筆PASS。

## 13. P12 — Detail UI 與 GATE A Closure

前置P11 PASS。Allowed：public/app.js、index.html/style.css（必要明細）、server.ts、debug.ts（Step03 handler、有效Gate狀態）、tests/gate-a.live.ts、acceptance/report/state；phase_state.py僅必要狀態正確性修正。

顯示所選真實job detail，逐欄比對104 UI。GETdebug.gateA不再是常數：只有當前Session/context/selection、Step01/02/03及額外人工Login/Selection證據全部有效才PASS。歷史PASS、fixturePASS、未落盤結果都不能閉合Gate。

在實際served新build重新載入Console，逐步操作，不以檔案存在或build成功替代畫面驗收。

建立 `data/acceptance/gate-a-<timestamp>.json`，至少包含：scope=LIVE_104_GATE_A、source/project/commit身份、current epoch、各步status、runId、evidence path/hash、時間、實際job count、匿名selected job identity、逐欄對照結果、未解問題。真實source state只接受PASS/FAIL/BLOCKED；未執行寫NOT_RUN並維持未驗收，等待寫workflow WAITING，不冒充通過。

最終closure五列：Login、Session、Job List、Selection、Job Detail；五列必須真實PASS，且先前check chain/必要回歸全通過，才更新GATE A PASS。更新MASTER參照狀態、EXECUTION-STATE、BUILD-REPORT。P12完成即停止；Next是等待使用者決定Gate B，不是自動P13。

## 14. 邊界、等待、偏差與Git

- 本輪全程 `RECRUITING_DRY_RUN=true`。不碰candidate/contact collection/AI/VOIP/外撥/自動邀請，不改外部repo/Production，不新增paid service。
- 登入只由使用者在專用headed Chrome操作；不接收/填寫/保存password/MFA；不要登入後再自動openLogin導回表單。禁止把session profile複製到remote或GitHub。
- 單一步驟等待時只完成該phase的非依賴驗證/文件，停在具體manual action。若user完成登入可當輪直接續行，不需每phase再問同意。
- 正常新增檔案、補method、route/wiring、測試工具或本計畫允許的整合不是PLAN_DEVIATION_REQUIRED。
- 真實104行為與凍結契約根本衝突、必須修改外部Forbidden資產、不可逆破壞/核心架構更換或無法局部修復安全風險，才記PLAN_DEVIATION_REQUIRED並停止affected phase。寫CURRENT/EXPECTED/CONFLICT/MINIMUM_OPTIONS，不偷偷改設計。
- 功能失敗留FAIL/BLOCKED＋匿名evidence；不得mock綠燈、暴力列舉/stealth/captcha bypass或讀無權限個資。
- checkpoint僅本機、目前phase explicit owned file/hunk allowlist；review staged diff後commit，不 push/force/reset/clean。P5未真驗收不得commit成「login acceptance PASS」。保留所有未完成WIP、profiles、DB與歷史證據。

## 15. 立即執行與回報

先完成read-only preflight及接手快照比較 → 收斂P5人工登入驗收 → P6 → P7 → P8 → P9 → P10 → P11 → P12，每次只一個Phase，前者真PASS才下一個。

不要重跑bootstrap generator，不要重做P0–P4，不要停在一份新計畫，不要跳過使用者登入，不要宣告不存在的企業職缺/功能已成功。

最終回覆繁體中文：CURRENT_PHASE、STATUS、GATE_A_STATUS、完成範圍、local checks、live五步矩陣、evidence路徑、checkpoint、remaining gap及唯一下一步。若等待登入，說明已開啟的具體視窗及user action；若完成GATE A，清楚說已停止於P12、未進Gate B。

提示詞方法參考：[OpenAI 官方開發流程文件](https://developers.openai.com/cookbook/examples/codex/iterating-development-workflows-with-codex)；僅採用明確現況、分段任務及可驗證輸出的寫法，權限與Phase推進一律以本使用者契約為準。模型名稱與Max是本次指定設定，不是此來源對LUNA能力或可用性的保證。

END OF LUNA GATE A CONTINUATION PLAN
