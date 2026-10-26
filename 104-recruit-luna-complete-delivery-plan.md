# 104Recruit — LUNA 完整交付執行計畫

版本：2026-10-26 / COMPLETE DELIVERY V2
指定執行模型：GPT-6 Luna；建議 thinking level：max。
工作根目錄：F:\AI\_104Recruit
本機服務：http://127.0.0.1:43104
本文件本機副本：F:\AI\_104Recruit\docs\planning\16-LUNA-COMPLETE-DELIVERY-PLAN.md
接手快照：F:\AI\_104Recruit\docs\planning\LUNA-COMPLETE-DELIVERY-SNAPSHOT.json

你是本專案的執行者。讀完本文件後直接實作、驗證、修復，持續到使用者能使用下述完整流程。不要只輸出另一份提案、空畫面、模擬資料或下一個提示詞。本文件中的檢查點由你自行執行，不要求使用者逐關確認。

## 1. 使用者要求與本輪授權

使用者最新要求的完整意思如下：

1. 讓使用者從104企業帳號可使用的人才來源，設定條件，篩選特定求職者，取得帳號確實有權查看的電話，產生可以拿去「快速應徵」使用的每日名單。
2. 一般方案每日查看額度先以300作為可編輯的預設值；不同方案、來源帳號的額度可以不同。
3. 每日查看X筆後，確定採用N筆。後續日子的讀取與採用必須避開歷史已採用的人與電話；重啟、換職缺、改方案都不能清除去重記錄。
4. 每天依條件與每日撥打目標分配不重複電話。少於目標時如實顯示不足原因，不能用歷史重複號碼湊數。
5. 插件已裝、目前Chrome已可讀取職缺。沿用此方式，禁止再開專用自動化Chrome。
6. 使用者要可用的結果與完整頁面；技術階段、手動逐步驗證與施工術語由執行者處理。
7. 本輪可在專案內完成候選人搜尋、合法可見聯絡欄位讀取、必要本機加密儲存、額度、去重、每日名單、匯出與前端。這些授權不再以過往「只到P12」為停止理由。

權威：使用者最新明確要求 > 本文件針對完整交付的修訂 > 現有專案其餘仍適用規則。
本輪的原始對話是授權依據。文件只是工程落地，不自行創造新的對外聯絡、付費或部署授權。
來源真實讀取與外撥開關分開：不要為了讓電話名單可用就把現有dryRun保護整體關掉。候選人/電話讀取是真實資料且可能消耗來源額度；外部訊息、邀請與外撥維持停用，畫面與測試清楚區分。
使用者仍負責登入、MFA或來源要求的真人操作。自動外撥、發邀請、送訊息、修改外部專案、公開部署與新增付費服務不在本輪執行內容；輸出名單供使用者使用。
如真正遇到來源拒絕、缺少權限、登入或必要商業參數，完成所有不依賴此條件的工作後，集中報告最小缺口。不能每完成一個工程檢查就停下來詢問。

## 2. 完整成果與不變量

最終主頁是「人才篩選與每日名單」，主路徑 /recruiting；根路徑 / 進入此頁。保留 /recruiting/debug 為收合的進階診斷，BAT啟動器改開主頁。

使用者流程：
選企業職缺 → 設定篩選條件、查看上限與每日名單目標 → 按一次「開始今日篩選」 → 系統自動搜尋、避開歷史採用、控制額度、讀取必要明細、檢查電話、分配每日名單 → 顯示實際名單 → 複製電話或下載匯入檔 → 記錄交付與後續結果。

核心不變量：
- 額度不能因併發、重試、重啟、跨日、切換職缺而被超扣、漏扣或自動歸零。
- 今日新採用的候選人ID及有效電話，不能出現在同工作區任何職缺的歷史已採用集合。
- 確定採用即開始歷史排除，不必等到撥打成功才排除。
- 已查看但未採用，與已採用，必須使用不同狀態。
- 未知、遮罩、無權限、尚待同意與拒絕聯絡的電話，不得列入可交付名單。
- 今日已產生的名單要能持續查看；Chrome連線或票券過期不應抹去業務資料。
- 本機額度是應用保護上限，不能宣稱它能提高104帳號本來的權限或額度。
- 系統不能保證每300次查看就有指定數量合格電話；如實區分查看、符合條件、可取得電話、已採用、已交付與實際撥打。
- 後端執行鎖、資料庫唯一約束、持久事件帳本是正確性來源；前端停用按鈕只是體驗。
- 真實來源驗收與合成資料測試分開記錄；兩者均不可冒充對方。

## 3. 已核實基線與接手順序

以下在2026-10-26由本機原始碼核實；你仍須接手時重新檢查：

- 專案 F:\AI\_104Recruit；分支 codex/bootstrap-gate-a；HEAD 26b0c620929a859071933ede41c05b1adfec8778。
- 工作樹有大量已修改及未追蹤WIP。完整檔案與SHA256見本輪接手快照。不得reset、clean、stash、覆蓋還原、廣泛stage或自行commit/push。
- Stack：Node22、TypeScript ESM、native node:http、靜態HTML/CSS/JS、pg、Zod；版本沿用 package-lock.json。
- 主DB是 PostgreSQL17，127.0.0.1:55432/recruit104。共享5432、其他Docker、WSL與Windows服務禁止修改。
- 目前前端只有 public/index.html、public/app.js、public/style.css 的連接器診斷工作區。
- 真實runtime用 src/chrome-bridge.ts 的 ChromeBridgeAdapter。src/adapter104.ts 與 src/browser.ts 的舊專用瀏覽器只留合成回歸用途；不得接回真實操作。
- 插件位於 F:\AI\_104Recruit\chrome-extension，manifest版本0.1.0。
- 現有固定命令只有 inspect / jobs / detail；不能把職缺讀取成功當作已經能搜尋人才或讀電話。
- 只允許 vip.104.com.tw、bsignin.104.com.tw 及本機console；沒有cookies、history、debugger、all-sites權限。
- 現有bridge有90秒單次票券、嚴格schema、512 KiB結果上限；即時context在120秒失效。
- 現有bridge的inspect會使後續選取狀態失效；須區分健康重驗與企業/帳號真正變更，避免新管線每次重驗就清掉任務。
- src/domain.ts 的 searchCriteriaSchema.limit 目前是 literal(5)；search/contact/queue/voip正式路由尚以 PHASE_NOT_IMPLEMENTED 拒絕。
- 001-foundation.sql 有candidate、candidate_contact、candidate_outreach、call_task等基礎表，尚沒有每日額度帳本、跨職缺永久採用排除或每日名單交付模型。
- src/db.ts 目前只執行001；任何新migration都必須連同runner實作順序執行及checksum驗證。不得只新增SQL卻未實際套用。
- scripts/db.ps1 用本專案cluster的trust初始化；候選人資料落地前必須完成本機SCRAM、最小權限角色、DPAPI管理密鑰與加密驗收，不能繼續用無密碼superuser處理個資。
- 現有 scripts/migrate.ts 是獨立遷移入口，server的ready/recover不會自動套新SQL；本輪須把受控migration步驟接進啟動器，否則build成功仍會跑舊schema。
- 先前真實職缺驗收22筆並有選取/明細PASS，證據 F:\AI\_104Recruit\data\acceptance\gate-a-live-20261026-125035.json。這是歷史時間點資料，不代表今天連線或候選人/電話驗收已通過。
- F:\00-Ticenpi-SaaS\Ticenpi-Voip 存在，但本輪對README、frontend/backend/docs的窄範圍檢索沒有找到「快速應徵」匯入契約；它不是已驗證的快速應徵系統路徑，禁止直接假設相同。
- 本文件與發布內容不包含求職者資料、企業職缺原文、完整電話或金鑰。

接手依序讀：
1. applicable AGENTS.md、本文件、本輪快照、目前git status/diff。
2. F:\AI\_104Recruit\docs\planning\MASTER-PLAN.md、14-DECISION-LOG.md、EXECUTION-STATE.md。
3. F:\AI\_104Recruit\docs\recruiting\CHROME-CONNECTOR.md。
4. F:\AI\_104Recruit\src\server.ts、chrome-bridge.ts、domain.ts、db.ts、config.ts。
5. F:\AI\_104Recruit\chrome-extension\manifest.json、console-bridge.js、background.js。
6. F:\AI\_104Recruit\db\migrations\001-foundation.sql、scripts\db.ps1、start-104recruit.bat。
7. 原03/07/09資料、佇列、安全規劃，然後依下表局部修訂，不能沿用衝突條款停止。

## 4. 明確取代過時條款

|舊條款|本輪採用|
|---|---|
|P12完成即停止並等下一階段批准|持續完成本文件所有本機成果及驗收，內部里程碑自行推進|
|真實操作使用專用Playwright profile|使用已安裝插件與使用者正常登入的目前Chrome分頁|
|search limit固定5|5只作為初次小樣本驗證上限；正式批次由方案額度、來源權限與每日目標限制|
|使用者逐一執行11步驗證|工程驗證自動執行；產品主頁只有業務操作|
|AI供應商成功評分才能產生名單|規則篩選先完成可用主流程；AI是可關閉的輔助，不作為電話名單必要依賴|
|必須先加Redis/BullMQ才有佇列|本輪使用既有PostgreSQL持久工作與鎖完成單機排程；不新增服務|
|允許未接自動隔日重撥|預設不自動重撥；歷史採用永久排除於新名單；回訪是獨立且須操作者明確指定|
|全部資料跟著短效bridge context清掉|短效來源驗證照常到期；業務名單、額度、去重與歷史保留於DB|
|所有11步與外部真人電話都PASS才算電話名單可用|本輪以真實篩選/可見電話/額度/跨日去重/交付/頁面驗收為完成條件，外部撥號未測狀態單獨列明|
|本機單一帳號可等同正式多租戶RBAC|本輪是本機單一操作者；公司資料與帳本仍隔離，不宣稱完成多使用者上線|

執行者先將修訂寫入現有MASTER、決策紀錄、需求、執行與驗收文件，清楚標注新舊依據；歷史證據不可改寫。
使用者的新要求已是本輪擴大施工範圍的授權，不要再把原allowlist或舊文件的未實作狀態當成需請示理由。
其他安全限制仍適用，尤其來源拒絕、隱藏電話、外部寫入、真人外撥、秘密與Production。

## 5. 業務名詞、集合與跨日去重

固定本機業務時區 Asia/Taipei，儲存時間UTC。所有業務日期由後端時計算，前端日期不決定額度。

- R_d：第d日實際觸發的來源查看行為。這是行為帳本，不能直接等同「成功解析候選人數」。
- V_d：第d日已查看的候選人集合；同人可能對應多次會扣額度的來源行為。
- S_d：第d日確定採用的候選人集合，具有當時有效可讀電話。使用者所說N筆指此集合。
- D_d：第d日已複製/匯出/交付的名單；交付不代表打過電話。
- C_d：由真實接收端回報或操作者標記的撥打紀錄；不能由匯出動作推導。
- H：本工作區累積已採用候選人ID與電話HMAC集合，跨日期、跨職缺、跨來源帳號保留。
- 「檢視候選人」與「揭露電話」若來源會分別扣額度，使用不同quotaBucket與事件，不能混成免費。

主要規則：
1. 尚未確定採用的暫存勾選是草稿，不是S_d。
2. 一旦確定採用/自動納入今日名單，原子建立使用紀錄與永久排除鍵，立刻加入H。
3. 次日預讀搜尋結果若已有穩定sourceCandidateId，先排除H，不能再打開該人的付費明細。
4. 同一人換電話仍以sourceCandidateId排除；不同ID同電話以phoneHmac排除；同人多個已知電話全部建立排除鍵，輸出只選一個有效主電話。
5. 未看過的新ID，只有開明細後才能知道是否與舊電話重複。此情況可能仍消耗一次來源額度，但該電話絕不可進名單。UI與驗收必須承認這個可觀測性限制。
6. X-N的已查看未採用者不永久封鎖；優先使用仍新鮮的本機快取重新比對條件。預設明細快取有效7天，可設定；來源已更新、權限變更或快取過期才按預算重讀。
7. 快取命中不新增來源查看消耗；真正再次開啟來源且來源會計費時照記，不靠本機unique候選人數假裝沒有消耗。
8. 已採用即使後來未匯出、未接通、換職缺或取消當日批次，也不自動移除H。
9. 刪除履歷/電話內容時保留最小的不可逆排除鍵與DNC鍵，避免刪資料造成重複聯絡；金鑰遺失須暫停新名單，不能另生新key當作無歷史。
10. 停止聯絡/DNC優先於分數、排序、快取、今日名單與所有交付；匯出前重新檢查並移除受阻項，保留原因和稽核。
11. 同一家公司/工作區多職缺共用H。不同企業不得互看電話/履歷；不跨不相干企業洩露誰在其他企業名單中。
12. 歷史已採用者的回訪在獨立「回訪」模式處理；本輪只建可記錄的預留契約，預設不產生回訪電話、不占新名單的N。

驗收範例：
- 第一天查看100人，採用A001–A030共30人：H加入這30人及其電話；另70人保留未採用快取。
- 第二天搜尋結果包含上述30人：按ID全部跳過，明細開啟次數為0；從新的人與未採用快取產生名單。
- 新ID B901的電話與A001相同：若名單頁看不到電話，最多在合法查看後辨識；計一次真實查看，採用0人並記PHONE_DUPLICATE。
- 第二天最終名單與歷史30人的人ID交集=0、標準化電話交集=0。
- 第三天重啟服務、換職缺或提高方案上限，上述交集仍為0。

## 6. 額度、方案與來源計費語意

### 6.1 方案與預算

設定表至少包含：planId、名稱、dailyViewLimit、可選dailyContactRevealLimit、dailyPreparedPhoneLimit、dailyDialLimit、businessTimezone、effectiveFrom。
dailyViewLimit預設300，明確標記「使用者設定的本機上限；來源實際可用額度另行核對」。不得寫成所有104方案都保證300。
其他方案欄位可編輯；每日準備電話目標由操作者首次設定整數N，不捏造使用者要撥幾通。
每日可配置N必須落在方案可用上限內。dailyDialLimit只在有真實接收端/手動撥打紀錄時使用，不能拿它充當已實現的外撥控制。
型別固定：dailyViewLimit為非負整數，預設300；dailyContactRevealLimit與dailyDialLimit可為null，表示尚無該項可核實政策，不表示可以任意對來源操作。dailyPreparedPhoneLimit與workspaceDailyTarget於第一次啟用時由操作者設定非負整數，未設定前不自動採用；N=0時不讀來源。即使來源查看上限是300，cache可節省查看，不能自行推論N一定要小於300。
尚未設定N時可看預覽、既有歷史與草稿，主頁直接提示「設定每天準備幾位」；不要要求填工程參數。
降低上限低於已用量時remaining=0，已用量保留；提高上限只增加當日差額，不能重置帳本。
run保留的planVersion是稽核快照，不是沿用過時上限的豁免；每次dispatch及commitUse都重新檢查目前有效policy。降低上限或暫停方案須即時阻擋尚未派送/採用動作。
多職缺、多批次、同日重跑共享相同來源帳號方案額度；不能為每個職缺各給300。
無法核實來源按seat或企業共用時計算成企業共享上限，頁面標示範圍待核實，禁止藉新增帳號標籤複製額度。
額度的穩定key不得使用bindingId、contextEpoch或隨機sessionId；重新連接不能創建新額度日。

### 6.2 來源計費發現

在固定合法操作下觀察搜尋清單、翻頁、履歷明細、電話揭露各自是否扣點，以及當日剩餘值、來源重置時區與重複查看規則。
只透過目前綁定分頁自然顯示的資料與可用操作，不讀帳密/Cookie、不猜私有API、不繞過遮罩/邀請/權限。
第一次實測只用1筆，必要時最多5筆；全數納入正式帳本，驗收不另有免費額度。
先讀取操作前計數，再一個操作，再讀取操作後計數。若來源不提供計數，記錄「未提供」，不能自行聲稱已驗證精確扣點語意。
不確定會否扣點的動作採保守成本上界預留；成本上界也無法可靠界定時暫停該來源操作，其他本機功能繼續完成。
只搜尋清單就會計費時，不能把list層自動算免費；一個操作最多可能扣k筆就先預留k。
額度檢查本身若會扣點也必須計入；不能用持續背景重驗製造額外消耗。
外部人工或其他工具也可能耗用同帳號額度：來源計數低於本機估算時採更低可用值，且顯示本機帳本無法辨識的外部消耗。

### 6.3 預留、提交與不確定結果

為每個可能消耗額度的來源動作建立持久operationId、idempotencyKey、quotaDay、quotaBucket、upperBoundCost及狀態：
RESERVED → DISPATCHING → CONFIRMED / CHARGED_UNKNOWN；只有可證明尚未發出或來源明確不扣的操作才能RELEASED。
- 在DB transaction鎖定來源scope+quotaBucket+quotaDay餘額，扣除預留後寫入operation，commit，再允許插件操作。
- worker領取持久lease；多程序/多分頁同一來源scope同時最多一個來源操作。固定鎖定順序避免死鎖。
- DISPATCHING在傳送前持久化；此後程序崩潰即視為「可能已發出」，不能因沒收到結果而退額度或盲目重發。
- 來源動作已發出但超時/回應解析錯/回傳遺失，計入CHARGED_UNKNOWN，消耗保守成本。來源即使回傳失敗也可能已計費。
- DB可做本機exactly-once帳本；無法跟104做分散式交易，不能承諾外部exactly-once。用同operation至多一次派送、悲觀計費及對帳處理不確定性。
- 純DB暫時錯誤可有界重試；可能扣點的來源操作不得自動重試相同或偽造新key。
- RESERVED未發出且lease逾期可以釋放；DISPATCHING或未知成本只能經來源可驗證對帳修正，留原事件與差額事件。
- 同一結果重送多次只提交一次；不同operationId若指同一logical request仍以idempotencyKey防重。
- 批次停止/暫停，只取消尚未派送動作；已發出動作完成後照帳入庫。
- 來源schema錯誤時已付出的查看仍計費，不把解析失敗當免費。

餘額顯示：
localRemaining = max(0, planLimit - confirmedCost - uncertainCost - reservedCost - knownExternalCost)。
若來源有有效remaining觀測，sourceAvailable從該觀測扣除觀測之後尚未反映的操作與預留，再取min(localRemaining, sourceAvailable)。
必須用watermark/sequence標明觀測已包含哪些本機事件，避免把已反映的用量扣兩次；不可直接從舊snapshot無限重減。
來源只提供總量/已用量時轉成同一模型；語意未知時保守處理並清楚顯示。

跨日：
每次派送前依後端時間檢查quotaDay；過午夜而未派送的舊日預留先釋放，再以新日原子重新預留。
跨午夜的已派送操作保留原派送日帳本，同時依可觀察的來源計費日核對；不確定跨日成本在新日額度也設保守hold直到對帳，不能利用午夜多拿額度。
本機時區固定Asia/Taipei；若來源重置時區/視窗不同，再用來源自己的視窗額度約束取最小值。
當日已採用名單的日期在commit使用紀錄時決定；不能把昨天剩的待處理候選人無限制算進今天。
不得把「每日歸零」做成刪除歷史資料。

## 7. 電話、身分與聯絡權限

只取得該企業帳號實際有權看到的欄位。VISIBLE與GRANTED尚需有效新鮮權限、有效完整號碼、未DNC、未使用、條件符合才可採用。
INVITE_REQUIRED / INVITE_PENDING列在「待開放聯絡」；本輪不自動發送邀請。
UNAVAILABLE / REVOKED / DO_NOT_CONTACT不可揭露、複製、匯出或排入可用名單。
可見性必須針對該企業與該候選人記錄contactEvidence、observedAt、validUntil或來源未提供、parserVersion；分數不能覆蓋權限。
權限快取與履歷快取分開：本機contactEvidenceMaxAge預設24小時，可調整；若來源給更短有效期則取較短值，這是本機新鮮度政策而非來源保證。每次交付要求目前企業的fresh Session，重新檢查已觀察的撤權/DNC；證據過期則明確要求受預算控制的權限更新，不能永遠以首次可見為依據。來源沒有撤權通知時，文件須承認無法保證即時感知所有遠端變更。
來源只有部分電話時記MASKED/不可用的原因，不自動補齊、猜測、改DOM、讀隱藏狀態或旁路端點。
必要電話揭露操作若有獨立扣點/條款或通知副作用，先核實；沒有已授權範圍內的純讀取操作就保留待處理，不擅自點會聯絡候選人的按鈕。

身分：
- sourceCandidateId優先用來源穩定ID；不能用姓名、頁序號、列表index或顯示名替代。
- candidate identity應綁source與workspace/company context；資料庫既有UNIQUE(source,id)需透過新migration改成符合企業隔離的唯一鍵並檢查FK/現有資料。
- 無穩定ID的列不可冒充已去重候選人；列待確認並排除自動採用。
- 電話正規化以TW預設、國家可配置、保留分機為獨立欄位；09與+8869同一手機等價。市話/國碼僅在確定完整可解析時可用。
- 只清除允許的分隔字元，不能把遮罩、文字或非法字元刪掉後拼成看似有效電話。
- phoneHmac以標準化E.164主號產生，分機不產生新的新名單資格；共享總機可能保守排除，顯示同號原因。
- HMAC-SHA256使用獨立DPAPI保護的dedupeKey與version；不用純SHA256或明文電話索引。
- 建立sourceCandidateId HMAC與phone HMAC兩類排除鍵，企業/工作區範圍明確。
- AES-256-GCM加密電話/個資，隨機唯一nonce、tag、keyVersion與包含workspace/candidate/field的AAD；加密key與dedupe key分離。
- 輪替HMAC key須雙版本查詢與受控回填，在全部舊資料可匹配前不可切換而失去排除；缺key直接暫停敏感讀取/交付。

## 8. 來源發現、插件與自動工作執行

### 8.1 真實來源發現

登入及職缺已完成的工作沿用；先以一次fresh Session核對，不反覆要求重新安裝或登入。
在已綁定的正常Chrome104分頁觀察：人才搜尋入口、可用條件、分頁/cursor、穩定候選人ID、明細與聯絡權限/電話呈現、剩餘額度。
來源選擇器、route與資料格式都要來自實際觀察，填入 F:\AI\_104Recruit\docs\source\candidate-search-discovery.md 與 contact-discovery.md；未知就保留UNKNOWN_UNTIL_OBSERVED，不猜端點。
操作候選人資料限當前已配置的職缺條件與小樣本驗收；未配置條件時先完成本機功能，不隨意讀人。
建立allowlist欄位與去識別fixture；真實姓名、電話、完整履歷、URL內秘密不進Git、公開提示詞、diagnostic或LLM請求。
可先用來源搜尋條件減少讀取，再對清單可見欄位本機篩選；需要明細才能判斷的欄位才在預算內查看。
沒有來源schema證據不得把parser或整個功能標成真實PASS。

### 8.2 插件升級

擴充固定動作，如quotaInspect、searchPage、candidateDetail、contactRead；最終命名與DTO在實作契約中統一。
commands set、background handler、bridge請求/回應Zod schema、server路由、前端transport與測試必須一起改。
每個action使用strict discriminated union，不把新的候選人資料塞進原jobDetail fields，不開放任意URL、JS、fetch或方法參數。
新版握手回傳protocolVersion/capabilities，舊插件顯示一次「請重新載入連接器」，不能卡成永遠執行中或回退到開專用Chrome。
沿用同host權限與綁定分頁，不讀Cookie、帳密、Storage裡的登入秘密、不新增all-sites/debugger/history權限。
同一operationId、action、公司、候選人、查詢cursor必須綁進單次票券；禁止拿A人的授權回B人的資料。
候選人與電話payload按一個受控操作/一頁分段傳輸，仍受512 KiB上限或經審查的新分段上限控制，不任意拉大body限制。
extension session只存binding、scope、opaque operation metadata；不持久放履歷/電話。
同來源scope一個inflight，不只使用易失busy布林；搭配後端持久operation與lease。服務重啟後先復原狀態，不能重新播放已派送命令。
更新插件版本、description與本機安裝頁，準確寫明新增候選人與電話資料用途。
使用者已要求這些本機功能；既有插件常規reload可直接協助處理。若Chrome真的要求新增權限或必須真人點擊，僅集中處理該一次必要操作。

### 8.3 自動管線與暫停續跑

後端持久job是權威；前端只負責呈現與傳輸插件命令。選定條件後按一次開始，自動跑下列流程：
1. 檢查DB、密鑰、方案設定、插件版本、公司綁定與fresh來源權限。
2. 讀取當日已採用總量，計算還缺多少。優先取本機未採用的新鮮快取。
3. 搜尋/翻頁，先以ID與list可見資料排除歷史採用、DNC與明顯不合條件者。
4. 原子預留來源額度，序列執行必要明細與可見聯絡讀取。
5. 標準化、規則比對、聯絡權限檢查、候選人及電話雙重去重。
6. 交易內建立當日採用項、永久排除鍵及每日目標用量；併發衝突者記去重結果。
7. 達當日目標/方案上限、來源無結果、來源額度不足或使用者暫停就停止新來源派送，保存cursor、run及各種實際計數。
8. 前端持續顯示結果與短缺原因；續跑只補剩餘目標，不另產生N筆。

狀態至少有QUEUED / RUNNING / PAUSING / PAUSED / WAITING_CONNECTION / WAITING_LOGIN / COMPLETED / PARTIAL / FAILED / CANCELLED。
PARTIAL不是全部成功；帶resumeCursor、已完成計數、stopReason與可否續跑。
pause/cancel與job配置改動有version/If-Match；按開始時凍結criteriaVersion、planVersion與companyScope。中途改條件只能套用下一次受控run。
同公司/來源scope同時只允許一個來源run；重複點擊同idempotencyKey回同run；不同職缺競用時清楚排隊。
本機業務資料不因120秒驗證失效而清空。保留短效票券與freshness約束，每次有實際來源成本的操作前更新合法Session證據。
被動健康檢查同公司時保留持久run與工作選取；企業/帳號真正切換則暫停舊run，不把舊結果接進新公司。
前端分頁背景仍能繼續，不需盯著畫面。現有console-bridge依賴本機頁存活：頁面/Chrome關閉或電腦休眠要暫停並下次續跑，不能宣稱關瀏覽器仍能擷取。
可有本機每日排程：預設關閉，操作者設定時間後開啟；到時若Chrome/登入不足，顯示待連接，不能另開Chrome或將積欠多日任務一次暴增。
若已達N，後續刷新、重新載入頁面與重啟不得開新來源頁或再用N份額度。
來源429/403/challenge不連續重試、不規避，暫停並保留已完成資料。來源500可有短暫有界健康檢查，但不得盲目重播可能已扣額度的操作。
UI輪詢用runVersion避免舊回應覆蓋新公司/新條件；worker錯誤不能把所有按鈕永久disabled。

## 9. 篩選與每日分配

必要條件以工作相關欄位為主：目標職缺、地區/工作意願地點、期望職稱/職類、關鍵技能、相關經驗區間、薪資期望、最近活躍區間、到職時間（來源提供時）。
條件分mustHave / optional / exclude，AND/OR、空值與缺資料語意明確；不以姓名或受保護個人特徵當篩選條件。
每筆結果顯示命中、未命中與資料不足的具體理由。未知欄位不可當符合；重要條件缺資料列待確認。
預設使用可重現的規則篩選；AI關閉時完整主流程仍可用。未取得外部資料傳輸及provider配置時不傳履歷/電話給外部AI。
如保留既有AI設計，只輸入已允許的最小職務相關資料，電話不送；輸出只輔助解釋/排序，格式錯誤不阻塞規則主流程。

每日目標：
- workspaceDailyTarget=N，campaign可有target/min/max/priority/weight及是否啟用。
- 當日剩餘 = min(操作者當日目標, 方案每日準備上限) - 同workspace當日已commit的distinct電話數，最小為0。
- 採用後取消、DNC或撤權不退回歷史已採用名額；可交付數因此下降時顯示缺額原因，不能悄悄以另一人補進而超過當日採用上限。需要追加時由操作者調整日目標，仍受方案上限與來源額度控制。
- 多職缺共用總目標與來源額度；所有campaign合計不能超過workspace日上限。
- 單職缺優先依符合程度、資料新鮮、穩定ID排序，提供穩定tie-break，不用每次random導致不可重現。
- 多職缺先滿足有設定的可行minimum，再依priority與weight用可重現的weighted round robin分配，遵守各自max。minimum總和超預算時在開始前明確報配置衝突。
- 同候選人同時適合兩職缺，依既定排序僅commit一個campaign，另一個補下一位；同電話的兩個人同樣只占一個資格。
- 若某職缺無合格者，可在允許的campaign間重新分配剩餘；不能放寬mustHave、忽略DNC或重用舊電話來補。
- 名單生成模式預設「符合條件即納入今日名單」，使用者按開始前可看到此效果；可切換「先預覽再批次確認」。預覽草稿不加入H，正式確認必須走同一交易。
- 將系統自動納入與操作者批次確認視為同一commitUse業務動作，不能只有匯出才記採用。
- 電話已用、無權限、未達條件與額度不足分別計數；總數守恆，不能把多個原因重覆相加宣稱讀了更多人。

## 10. 持久資料模型與交易

使用新版本migration新增/調整模型，禁止改寫已套用的001。下列是必須覆蓋的邏輯實體；可合理合併物理表，但API/測試需驗證相同不變量。

|邏輯實體|主要欄位及約束|
|---|---|
|workspace / source_account / source_scope|穩定workspaceId、公司scope、來源帳號opaque ref、quotaScope策略；與session分離；不存登入秘密|
|quota_plan / quota_policy|版本、各quotaBucket上限、timezone、有效時間、方案名稱；同scope每時段僅一個有效policy|
|quota_day_balance|scope+bucket+day唯一；limit快照、confirmed/uncertain/reserved/external；非負與鎖定規則|
|source_operation / quota_event|operationId、logicalIdempotencyKey、action、cost上界/實際、day、scope、status、dispatchAt、sequence、observationWatermark、lease；追加事件不改舊證據|
|source_quota_observation|來源計數/remaining、來源重置視窗、observedAt、includedSequence、confidence|
|search_run / search_run_item|criteria/plan版本、companyScope、狀態、cursor、stopReason、lease、計數；item唯一run+sourceCandidateId|
|candidate / source_profile / contact|workspace scoped來源ID、allowlist個資/新鮮度、密文電話、權限證據、parserVersion；隔離既有FK|
|candidate_phone_identity|candidateId、keyVersion、phoneHmac、phone密文ref、primary、contactEvidence；多電話支援|
|candidate_use / suppression_key|workspace、candidateKey/phoneKey、首次採用時間、reason、useId；UNIQUE(workspace,keyType,keyVersion,keyValue)，永久跨campaign去重|
|daily_list / daily_list_item|業務日期、campaign、run、candidate/use/phone ref、排序、狀態；同workspace/day不重複，同歷史也受suppression唯一約束|
|delivery_batch / delivery_item / delivery_event|穩定batchId、manifestHash、formatVersion、內容版本、項目ref、channel、回報狀態、idempotencyKey；不把download等同撥打|
|outreach_receipt / audit_log / do_not_contact|verified事件ID、provider或human來源、去重事件、結果；DNC含candidate與電話層級，對全campaign有效|

必要交易：
A. reserveSourceCost：quota rows鎖定 → 檢查policy/day/remaining → operation+reservation入庫 → commit。
B. confirmSourceResult：驗operation/來源身份 → 同transaction寫實際計費狀態及解析結果；回傳遺失不導致二次來源操作。
C. commitUse：鎖daily target → 再檢查公司/條件/聯絡權限/DNC/候選人與電話鍵 → 寫use+所有suppression鍵+daily item+用量 → commit。
D. prepareDelivery：重新檢查所有項目權限/DNC/資格 → 固定manifest/version與batch → commit，才允許輸出。
E. delivery回報：冪等寫event；不能因回報重送多扣每日目標、再標採用或新建call task。

同號不同ID與同人多電話競態由DB unique與transaction處理；catch衝突回傳可理解原因，不能破壞整批。
對001遺留資料先盤點；若存在真實明文個資/空workspace，停止新增敏感資料並做可恢復的定向修正，不整庫重建。
遷移含checksum、事前專案DB備份、已套用版本檢查及二次啟動冪等測試；無DROP DATABASE/TRUNCATE/清空data。
SCRAM轉換必須先核實55432是本專案cluster、保存可恢復設定與備份、建立DPAPI保護的遷移/應用角色憑證、驗證可登入，再收斂本專案pg_hba的trust。migration用受控遷移角色，runtime app role不可superuser/CREATEROLE/DDL；密碼不在命令列、stdout或Git。失敗回復該專案設定，不修改其他cluster。啟動器先檢查安全provision與schema version，再依序遷移與啟動；不在每次啟動重建角色或另生dedupe key。
個資欄位的落地策略要列成明確schema：電話、email、原始履歷及姓名等可識別內容用密文欄位或加密profile payload；原表對應明文欄位停用並作定向遷移。只保留必要的範圍鍵、HMAC鍵與最少工作條件索引；所有raw HTML/完整履歷預設不保存，必要的去識別欄位才作fixture。禁止同時寫密文又漏寫一份完整明文。

## 11. 本機API契約與錯誤

以下路徑是本機產品API的設計，絕不是宣稱104提供這些API。實作前建立DTO表並在前後端、Zod、錯誤碼、測試中一致。

- GET /api/recruiting/overview：connection、plan、quota、todayCounts、currentRuns、shortfallSummary；不帶完整電話。
- GET/POST /api/recruiting/settings：方案/目標/排程設定，POST含expectedVersion；前端不可傳角色自我授權。
- GET/POST /api/recruiting/campaigns：從真實企業職缺建立條件，驗證當前scope和jobId；可分拆update路由。
- POST /api/recruiting/search/preview：純本機計畫與成本預估，不觸發付費來源操作。
- POST /api/recruiting/runs：criteriaVersion、campaignIds、runMode、idempotencyKey，回202和runId；不是長時間HTTP等待。
- GET /api/recruiting/runs/:id：版本、狀態、計數、stopReason、下一個可做動作；cursor不洩漏來源秘密。
- POST /api/recruiting/runs/:id/pause|resume|cancel：strict action payload、expectedVersion、idempotencyKey。
- GET /api/recruiting/candidates：受workspace與run範圍限制的分頁列表，遮罩電話，穩定排序。
- POST /api/recruiting/daily-lists/commit：批次草稿採用，與自動run共用commitUse；防任意ID採用與超額。
- GET /api/recruiting/daily-lists?date=YYYY-MM-DD：同workspace歷史/今日名單，電話預設遮罩。
- POST /api/recruiting/contacts/:id/reveal：只回已授權、已採用且仍有效的本機電話，no-store、稽核、不可隱式觸發來源付費讀取。
- POST /api/recruiting/deliveries：準備凍結批次，驗權限/DNC、格式、版本與冪等key。
- POST /api/recruiting/deliveries/:id/materialize：在重新檢查後回電話文字或CSV內容/安全下載句柄；不可用GET暗中採用/改帳本。
- POST /api/recruiting/deliveries/:id/events：記COPY_COMPLETED、DOWNLOAD_INITIATED、MANUALLY_CONFIRMED等明確事件，不能宣稱已匯入外部或已撥打。
- GET /api/recruiting/history：條件/run/use/delivery/outcome/額度事件，可按日期和職缺篩選，不回秘密。
- POST /api/recruiting/dnc：操作者禁止聯絡，原子阻擋待交付項；測試只用合成候選人。

新增業務錯誤至少覆蓋：
QUOTA_EXHAUSTED、QUOTA_COST_UNKNOWN、SOURCE_RESULT_UNCERTAIN、SOURCE_PERMISSION_REQUIRED、PHONE_UNAVAILABLE、PHONE_DUPLICATE、CANDIDATE_ALREADY_USED、DNC_BLOCKED、TARGET_REACHED、CONFIG_REQUIRED、VERSION_CONFLICT、CONNECTOR_UPGRADE_REQUIRED、SCOPE_CHANGED、ENCRYPTION_UNAVAILABLE。
與現有domain ERROR_CODES/HTTP統一擴充，不硬回INTERNAL_ERROR。
HTTP 202用於已接收工作、409用於版本/重複/狀態衝突、422用於業務配置、429用於本機/來源速率情境並附safe reason；不得只看200當成功。
所有修改/敏感讀取沿用Host/Origin/CSRF防護、schema、parameterized SQL、no-store及無第三方資源CSP。
完整電話不出現在URL、console log、exception、diagnostic、source discovery檔、截圖驗收或LLM payload。

## 12. 快速應徵使用方式與交付語意

「快速應徵」是使用者要拿名單去使用的目的地；不得自行理解成104邀請、寄信或已完成外撥。
本輪必須先交付兩種可直接使用的功能：
1. 「複製電話」：本機產生已核准名單的電話純文字，每行一筆；明確選擇TW格式或E.164，預設TW手機維持09開頭。保留原始字串，不經Number轉型。
2. 「下載名單」：UTF-8 BOM CSV與可選純電話TXT；欄位至少phone、name/displayName（可選關閉）、jobTitle、sourceCandidateId、businessDate、batchId。電話必須維持字串，逗號/換行/引號正確轉義。自由文字欄位做試算表公式注入處理；電話欄只允許已驗證的電話語法。

若可在使用者明確指出的快速應徵程式找到實際匯入畫面/範例/契約，可在該外部專案唯讀範圍核對欄位、電話格式、重複處理、編碼和batchId支援：
- 文件記錄真實完整路徑與契約版本。
- 在本專案實作format adapter，不直接寫外部DB/佇列、不擅自修改或部署接收端。
- 無法定位時，電話純文字功能照常完成，通用CSV標明「通用格式，相容性尚未核實」，不要聲稱已串接或已能直接匯入某特定程式。
- 可以把「一個去識別匯入範例或匯入畫面位置」列為最終唯一整合缺口，不能用它阻塞本機名單/電話/歷史/去重交付。
- 不為本輪重造SIP、ASR、TTS或AI電話系統。

三種重複要分開：
- 同一批次重新下載/複製：允許取回原manifest的仍可聯絡項，標「重新取得原名單」，不消耗新名單額度、不把同電話再分配為另一位。
- 另一批次/明天的「新名單」：永久H排除，不能再次拿走同電話。
- 外部快速應徵重複匯入：除非接收端支援並實測batchId/itemId idempotency，無法保證不重複撥打。提供stable ID與接收端建議，不謊稱剪貼簿/CSV有跨系統exactly-once。
重新取得原名單前仍查DNC/撤權；資格改變則產生帶parentBatchId的修訂manifest與變更說明，不能改寫歷史hash或回出失效電話。
clipboard失敗/下載取消是交付失敗或未知，不回滾已採用去重、也不當作撥打成功。
實際dialed/answered/no_answer/outcome只有真實adapter事件或使用者明確標記能更新；provider事件必須eventId去重。
如後續接收端提供dry-run/import-validation，不發起真人外撥即可驗證契約；真人聯絡仍由使用者在既有快速應徵系統執行。

## 13. 完整產品頁面

沿用現有靜態前端架構與品牌，完善可用性；不換React/Next、不新增設計雲服務、不放假候選人做主畫面。

導覽：「人才篩選」「今日名單」「歷史使用」「額度與方案」。進階診斷收合，主頁不展示Gate、Phase、P編號、11步驗證或工程error stack。

主頁：
- 頂部顯示Chrome連接與企業scope的友善識別、目前方案、今日可用查看額度、來源對帳狀態。
- 分開顯示今日查看次數、合格人數、有可用電話、已採用、已交付、重複排除、目標缺額。實際撥打未回報時顯示「未回報」，不可顯示0已完成暗示已整合。
- 左/上方為職缺選擇、工作相關條件、今日目標與預覽/自動採用設定；可儲存條件。
- 主按鈕「開始今日篩選」；運行時顯示即時計數、目前動作、「暫停」與「繼續」，不是要求點下一步。
- 結果表至少包含候選人顯示名、相關職能/地區、命中理由、條件缺口、電話狀態、採用狀態、排除原因；來源提供什麼就顯示什麼。
- 已完成名單清楚列出職缺歸屬、採用日與交付狀態；電話預設遮罩，按揭露或複製時才本機解密。
- 「複製電話」「下載名單」按鈕只對當前核准名單，呈現實際筆數與格式；操作失敗可重試同batch。
- 短缺顯示可理解原因，例如「查看額度已用完，目前取得18/30位可用電話」或「有12位需先取得來源聯絡權限」。
- 搜尋列表重複排除與不同ID同電話去重有不同原因，避免使用者以為漏資料。
- 歷史頁可按日期/職缺查已查看、已採用、已交付；展示重新取得原批次與新名單的差別。
- 額度設定可新增不同方案/上限、修改每日N、開關每日排程；顯示生效版本與當日已使用，不能一按切方案清零。
- 斷線、失效登入、舊插件、額度未知、加密不可用、沒有資料、仍在執行、部分完成都要有真實可恢復畫面。
- 來源連線失效仍可查看本機遮罩歷史；敏感揭露/交付依持久權限證據新鮮度與DNC決策，不以歷史PASS取代現時權限。
- 選取職缺與設定每日N屬正常業務操作；工程連線/驗證自動完成，只有真人登入/來源權限步驟才提示。
- 做1440px與1366px桌面、窄螢幕布局；可鍵盤操作、可見focus、正確label/status、長文換行、錯誤不只靠顏色辨識。

BAT：
F:\AI\_104Recruit\start-104recruit.bat 雙擊可從任意cwd啟動，開 /recruiting；同服務存在時避免開重複service。
辨識本專案43104 owner/build版本，若是舊版則提出/執行受控的本專案更新重啟；不因health同名就宣稱新版已載入。
保留未知port owner停止機制、Node版本檢查、DBready、build失敗可讀訊息；後台helper用隱藏視窗，避免額外登入Chrome。
服務重啟不得清除每日額度、採用排除、歷史名單或對未知來源動作重播。

## 14. 施工分解與允許範圍

所有變更在 F:\AI\_104Recruit；外部專案只可在有實際必要時唯讀觀察明確介面，不能改其他Repo、WSL、Docker、共享DB或Production。
本輪沒有自動commit/push產品程式授權；保留本機WIP供檢視。允許新增專案內必要檔案並修改下列現有檔案，按任務擴充owned tests；不能拿舊phase allowlist當停止理由。

|任務|必做內容|主要完整路徑|
|---|---|---|
|R0 接手與權威收斂|核對快照、保留WIP、局部修訂舊計畫與新工作狀態，不更改歷史驗收|F:\AI\_104Recruit\docs\planning\MASTER-PLAN.md；01-REQUIREMENTS.md；02-ARCHITECTURE.md；03-DATA-MODEL.md；05-DEBUG-ACCEPTANCE-PLAN.md；07-CALL-QUEUE-PLAN.md；09-SECURITY-COMPLIANCE.md；10-EXECUTION-PHASES.md；11-TEST-MATRIX.md；13-CHANGE-BOUNDARIES.md；14-DECISION-LOG.md；EXECUTION-STATE.md|
|R1 本機資料安全與遷移|DB備份、SCRAM/app role、DPAPI keys、GCM/HMAC、順序migrations與workspace隔離|F:\AI\_104Recruit\src\db.ts；src\config.ts；src\crypto.ts（新）；db\migrations\002-*.sql以後（新）；scripts\db.ps1；scripts\migrate.ts；scripts\secure-local.ps1（新）；.env.example|
|R2 額度與來源動作帳本|政策/來源計費發現、原子預留、未知成本、跨日/恢復與API|F:\AI\_104Recruit\src\quota.ts（新）；src\source-operations.ts（新）；src\domain.ts；src\server.ts；docs\source\quota-discovery.md（新）|
|R3 來源人才/聯絡讀取|小樣本觀測、固定命令、parser、protocol版本、合法電話/權限|F:\AI\_104Recruit\chrome-extension\manifest.json；console-bridge.js；background.js；src\chrome-bridge.ts；src\parsers\candidate.ts（新）；src\parsers\contact.ts（新）；docs\source\candidate-search-discovery.md；contact-discovery.md|
|R4 篩選/歷史去重/每日分配|cache、criteria、雙重去重、永久use ledger、每日target公平分配|F:\AI\_104Recruit\src\search.ts（新）；src\candidates.ts（新）；src\dedupe.ts（新）；src\daily-lists.ts（新）；src\domain.ts；src\server.ts|
|R5 自動工作與恢復|後端job/lease、按一次自動到結果、暫停續跑、排程、狀態持久化|F:\AI\_104Recruit\src\recruiting-jobs.ts（新）；src\scheduler.ts（新）；src\server.ts；src\chrome-bridge.ts；public\app.js|
|R6 交付格式與紀錄|文字複製、CSV/TXT、stable manifest、重新取得、DNC重驗、接收端契約盤點|F:\AI\_104Recruit\src\delivery.ts（新）；src\quick-apply.ts（新）；src\server.ts；docs\recruiting\QUICK-APPLY-CONTRACT.md（新）|
|R7 完整UI與啟動器|業務導覽、表單、工作進度、每日/歷史/設定頁、診斷收合、現有Chrome|F:\AI\_104Recruit\public\index.html；app.js；style.css；chrome-connector.html；src\server.ts；start-104recruit.bat；README.md|
|R8 驗收修復與交接|完整本機回歸、小量真實來源、跨日模擬、頁面與BAT、保存證據|F:\AI\_104Recruit\tests\unit\*.test.ts；tests\integration\*.test.ts；scripts\ui-smoke.ts；scripts\test-runner.mjs；docs\recruiting\BUILD-REPORT.md；FINAL-ACCEPTANCE.md（新）；data\acceptance\complete-delivery-*.json（私有）|

表中同列省略的相對檔名均相對於該列明示的專案根 F:\AI\_104Recruit；來源觀測文件一律在該根的docs\source。
若需要額外小模組，把實際完整路徑與理由追加owned-files紀錄後繼續，不需為每個新檔請示。
已有測試與其他WIP的合理修正可以做；不得以讓測試過關為由刪除來源拒絕、scope隔離、票券、secret與真實驗收檢查。
依相依順序執行R0→R1→R2；R3有成本的真人資料讀取必須在R1/R2完成後；R4/R5/R6核心可用去識別fixtures先完成；R7整合；R8驗證並修復到收斂。
每個任務完成更新狀態與測試證據後直接做下一個，不向使用者報Gate A/B/C/D或要求階段批准。

## 15. 必測矩陣

這些是實作後必須跑的有意義測試，不是本次寫計畫就能宣稱PASS。

|ID|情境|必須斷言|
|---|---|---|
|Q01|每日上限300，所有cache miss，模擬來源每detail扣1|第301個成本操作不會dispatch；confirmed+uncertain+reserved不超可用|
|Q02|只剩1份額度，兩個run/worker競用|最多一個來源操作取得reservation|
|Q03|重複開始/重送result/同logical request不同HTTP|同run、同operation、同帳本結果；不二次扣點/讀來源|
|Q04|已dispatch後timeout、parse failure或process crash|CHARGED_UNKNOWN或已確認成本保留，沒有自動refund/replay|
|Q05|RESERVED尚未dispatch就崩潰|復原後能證明未發出才釋放，不影響已派送操作|
|Q06|來源remaining比本機低、外部手動耗額度|採較低可用值，顯示差額；不double debit|
|Q07|同日改方案300→500→100|已用量不歸零，新增差額正確，降額remaining0|
|Q08|同帳號多職缺/多分頁/重連Chrome|共享額度key；binding/epoch改變不能新增300|
|Q09|23:59到00:01，pending與inflight跨日|pending重新預留；inflight不雙放行，歷史去重不清|
|Q10|source list也扣點/揭露另扣點|各cost/bucket正確；不把preview/search自動當免費|
|D01|第一日X=100、N=30，第二日混合舊30人|舊30人以ID預讀排除，明細開啟0；新名單人/電話交集0|
|D02|不同來源ID同電話09與+886格式|只允許一次採用；必要額外查看如實計費|
|D03|同人多電話、次日改電話、跨campaign|候選人與所有已知電話均排除；只產生一個use|
|D04|兩個worker同時採用同人/同電話|DB unique+transaction使僅一方commit，其餘可恢復|
|D05|只看未選的X-N人|命中cache不扣來源額度，仍可符合新職缺；歷史採用不回流|
|D06|已採用後取消、未接、刪profile或重啟|永久排除仍在；不自動重派|
|D07|HMAC加密key遺失/錯key/輪替中|停用敏感流程或雙版本查詢，不出現重複新名單|
|P01|VISIBLE/GRANTED有效電話|條件/DNC/歷史都通過才可採用|
|P02|INVITE_REQUIRED/PENDING/REVOKED/MASKED/UNAVAILABLE|不能輸出完整電話、不可假裝已採用|
|P03|明細人ID不符、換公司、結果票券replay|拒絕且不混資料；已發出成本仍按規則保留|
|P04|DNC在commit後/export前新增|交付排除；稽核及修訂manifest正確|
|P05|同名不同人與無穩定ID|不以姓名合併；無穩定ID不自動採用|
|S01|多職缺總目標與weight分配|總數不超全局N，每項max有效，同人同號只分一職缺|
|S02|來源不足/條件不足/額度不足|PARTIAL與實際缺額，不降低條件或用重複電話補|
|S03|目標達成後再按開始/重啟/每日排程|不額外讀取、不多產生一批N|
|S04|背景分頁、斷線、關Chrome、MV3重啟|可暫停續跑，不丟帳本、不重播不確定操作|
|S05|執行中改criteria/company，舊response較晚回|version/scope guards拒絕覆寫|
|E01|複製/下載一次、同batch重取|電話唯一、ID穩定、使用與每日用量不增加|
|E02|複製權限拒絕/下載取消/回報重送|交付狀態真實，不回滾使用排除、不標撥打|
|E03|CSV BOM、引號、換行、中文、09/+886|內容round trip字串完整，無自由文字公式注入|
|E04|unknown快速應徵格式|純電話功能可用，通用CSV不冒充相容已測|
|E05|接收端事件重送（若契約已知）|eventId去重；匯出不等於dialed|
|SEC01|密文/nonce/tag/AAD/不同workspace|不存明文聯絡資料；交換密文或錯key不能解密|
|SEC02|資料掃描/錯誤/來源文件/console/驗收截圖|沒有真實電話、完整履歷、登入秘密|
|SEC03|CSRF、wrong Origin/Host、任意ID、cross workspace|後端拒絕，不能靠前端遮罩當權限|
|DB01|新migrations跑兩次、既有001 checksum、失敗回滾|冪等/有序，舊資料和WIP不丟，未知schema不硬蓋|
|UI01|主頁完整流程與所有recover狀態|無Gate/Phase工程導覽、無手動下一步、無永久disabled|
|UI02|1440/1366與窄螢幕、鍵盤操作|無主要操作被遮、狀態可讀、資料可換行|
|BOOT01|任意cwd雙擊BAT、服務已啟動/舊版/未知port|開正確主頁，版本吻合、不殺未知process、資料保留|

數量守恆：
- 每個source operation只屬一個最終計費狀態；未知不當失敗退款。
- 每個run item有唯一主要結果分類，其他原因作tags；不能把同一人多算到總人數。
- 每日new use=本日commit成功的unique phone/candidate項；delivery retries不增加。
- 所有新電話與歷史H交集0；同日不同campaign間交集0。
- 真實來源小樣本的成本和採用排除不得為了測試清空；跨日測試使用fake clock與去識別DBfixture，不改電腦時間。

標準本機檢查依序：
npm run lint
npm run typecheck
npm run test:unit
npm run test:integration
npm run build
再執行專案內必要的UI smoke與BAT流程驗證。
新增nested tests時修改test-runner確實收集，不可新增檔案卻沒跑到。
所有測試使用本專案隔離DB/schema及去識別fixtures，不清真實資料、不碰共享5432、不測真人電話。
失敗修當前問題後重跑受影響檢查；最後完整標準檢查一次並記command、exit code與摘要。

真實來源驗收：
- 登入者目前正常Chrome、插件新版本協定與企業scope。
- 一個真實已選職缺條件，受控1–5人小樣本（不為驗收耗300）。
- 真實搜尋、ID解析、一次成本觀察、允許可見的電話完整取得、一次確定採用與後續排除。
- 同一人第二次嘗試在付費明細前被擋；同號不同ID用fixture證明並明列其可觀測性限制。
- 複製/下載可在本機驗證完整電話與唯一性；證據只留masked/HMAC/counters，不把電話貼在對話或截圖。
- 若帳號根本沒有可見電話，只能標「真實電話驗收未完成：來源無權限」，仍交付完整本機功能，不用測試電話補成真實PASS。

## 16. 完成定義與最後交付

必須完成：
1. 已安裝插件能在目前正常Chrome搜尋人才、讀必要明細及有權可見電話。
2. 操作者一次設定條件與每日目標後，只按一次即可自動得到每日名單，能暫停/續跑。
3. 方案查看額度可調；300只是預設；來源額度、未知成本、併發、重啟與跨日保護均通過。
4. 以「確定採用」為歷史去重起點，跨職缺、跨日、換方案/重連仍不重複人或電話。
5. 每日分配、名單複製與CSV/TXT下載、交付紀錄、DNC與歷史查詢在主頁完整可操作。
6. 主頁沒有工程Gate/逐步驗證要求；進階診斷仍可供開發者使用。
7. 本機DB安全、密文/排除鍵、migration、復原、權限與固定插件命令全部有必要測試。
8. 真實來源小樣本驗收與本機自動化測試結果分開，沒有用fake/stub當真實成功。
9. 快速應徵的電話純文字輸出可用；特定接收端匯入契約若未核實，要單獨寫明，不能把它冒充直接整合成功。
10. 外部真人撥打不屬於本輪測試完成定義；已撥打欄位只能接受真實事件或操作者明確紀錄。

交付檔案：
- F:\AI\_104Recruit\docs\recruiting\FINAL-ACCEPTANCE.md：完成/未完成能力、實際測試、來源限制、一次操作指南。
- F:\AI\_104Recruit\docs\recruiting\QUICK-APPLY-CONTRACT.md：電話複製/通用輸出，以及特定接收端契約是否已核實。
- F:\AI\_104Recruit\docs\recruiting\BUILD-REPORT.md：當前真實版本、變更檔案、測試、已知缺口；保留舊歷史。
- F:\AI\_104Recruit\docs\planning\EXECUTION-STATE.md：最後完成任務/正在等待的外部條件/可恢復下一步。
- F:\AI\_104Recruit\data\acceptance\complete-delivery-<timestamp>.json：去識別metadata證據，包含scope、版本、tests、quota/dedupe樣例結果與真實小樣本計數；不進Git公開。
- 正常執行的 http://127.0.0.1:43104/recruiting 與可雙擊的 F:\AI\_104Recruit\start-104recruit.bat。

最後對使用者用繁體中文，只報：
- 可點擊的主頁。
- 現在能完成的業務流程。
- 每日額度與跨日排除已驗證的結果。
- 真有外部障礙才集中列出必要動作；不要列內部Phase或要求使用者當測試員。
- 不宣稱尚未完成的真實電話/快速應徵直接匯入/外部實際撥打已驗收。

開始執行：先做R0接手，再依相依順序完成R1–R8，遇到可修復問題自行修復。不要在計畫整理、舊P12停止點、搜尋5人樣本、純測試PASS或單一頁面完成時結束。

**END OF EXECUTION PLAN — 104RECRUIT COMPLETE DELIVERY V2**

