# Ticenpi-Voip：正常對話、短句保留與 Shadow 可驗收性建置

目標執行模型：DeepSeek V4.1-Flash，Thinking enabled，reasoning effort high。
任務基準日期：2026-11-09，Asia/Taipei。

你是接手 Windows 本機專案的工程 Agent。直接閱讀、重現、修復、執行回歸及交接，不要只產出計畫、提案、工具空殼或新的任務提示詞。每個結論必須來自你實際讀到的原始碼、音訊、設定與執行結果。

模型必須運行於具檔案與終端工具的 IDE/Agent；純聊天介面不能存取本機 F:。沒有本機工具或錄音無法讀取時，明說實際阻礙與 NOT_RUN，不得假裝已建置。工具、context7、codebase-memory 有提供才使用；依專案 AGENTS.md 使用結構索引，未索引或覆蓋不足時直接讀檔/rg，不宣稱查過不存在的索引。

## 1. 任務成果與禁止事項

本階段交付能重現的隔離修復與可信驗收證據，順序為：

1. S0：修正基準／測試工具本身的失真與可重現故障。
2. P0：正常收句 → 真實 STT → 狀態機 → 正確回應 PCM 的完整離線對話管線。
3. A：播放期間既有真人短句不因視窗／排程遺失。
4. B：每個 Shadow 判定的 reference 與 ground truth 可追溯，分類有可信分母。
5. C：保留既有 writer 停播／ledger 控制測試，不升格為真人電話 barge-in。

只做最小必要改動，優先使用既有實作、標準庫與已安裝依賴。不得新增模型、TTS/LLM 服務、資料庫、通用排程框架或無需求的架構。

嚴禁：

- 修改、部署、重啟或停止正式 9090、Asterisk、HT813、正式 ASR、正式增益、dialplan、DB、barge-in；既有 19095/19110 也不操作。
- 真人／批次外撥、AMI Originate、任何新增真人電話測試、打開自動聲學 barge-in。
- push、merge、rebase、force push、git reset、git clean、切換共享工作樹分支。
- 用延長 timeout、修改 VAD/分類閾值、改 gain、改模型／window/hop／tokens 偷換本輪 A/B 條件，或以更多無界 ASR 工作掩蓋處理容量不足。
- 把合成、剪輯、重播、人工注入 interrupt、固定 transcript 重分類稱為真實電話實測。
- 將錄音、電話號碼、secret、模型、原始 log、完整轉錄、DB、state、CSV ledger 或大量 results 提交 Git／公開。

允許啟動自己擁有的離線 TEA 子程序：先確認 loopback 19111 是否空閒、程序 owner、cwd、venv、模型與設定；被占用就不用它，不得停掉他人程序。只清理自己建立且驗證身分的程序。沿用現有模型快取，禁止下載／升級依賴。測試不得呼叫外部 LLM；空字串 LLM_INTENT 會讓既有 helper 讀本機設定，必須明確停用並限制非 loopback HTTP，不得列印 secret。

## 2. 唯一專案與 Git 基準

Windows repo：`F:\00-Ticenpi-SaaS\Ticenpi-Voip`
WSL repo：`/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip`
現有 DEV：`_workspace/acceptance/live/ht813/offline_repair_20261107/`
現有 Shadow：`_workspace/acceptance/live/ht813/v11_4j_dev/semantic_shadow.py`

開始先讀 AGENTS.md，再讀：

- `docs/VOIP_STATUS.md`
- `docs/VOIP_ACCEPTANCE.md`
- `docs/VOIP_RUNBOOK.md`
- `docs/VOIP_OFFLINE_REPAIR_HANDOFF.md`，包括 2026-11-07 與 11-08 更新，逐條核對矛盾。
- DEV/repair.diff、最近 commit 的實際 diff、DEV 的 replay/shadow_replay/audit_audio/verify_results/test_repair/run_offline，以及 Shadow 和測試的實際來源。

已核對的線性歷史如下；仍須在任務開始重驗，不可 checkout 舊 SHA 蓋掉 WIP：

```text
20816cd8ae19664fa8144152d4007564c084df94  隔離 VAD/node/ledger/路由修復
  -> 1a80ef9a564a7151892c871360f3bc7b5ef84765  文件交接
  -> 5d97a8fe644f8177462a768e84d03055d70e6a4d  bounded pending queue + LCS guard
```

2026-11-09 查核分支為 `rescue/echoguard-v11-4-snapshot`，HEAD 是最後一個 SHA。記錄實際 HEAD、parent、ancestor、git status、載入模組路徑與檔案 SHA256；HEAD 若前進就依當下內容重評，不自行回退。

當時共有 5 個既有 tracked dirty 檔案，已包含兩個測試，不是「5+2=7」：

```text
_workspace/acceptance/live/ht813/offline_repair_20261107/test_repair.py
_workspace/acceptance/live/ht813/v11_4j_dev/tests/test_echo_guard.py
docs/VOIP_ACCEPTANCE.md
docs/VOIP_OFFLINE_REPAIR_HANDOFF.md
docs/VOIP_STATUS.md
```

另有 `.opencode/`、`data/_backup/` 未追蹤；不動它們或 IDE 模型路由設定。
先保存這 5 檔的本機 diff/hash 清單，保留原路徑內容。不得代為提交既有 WIP；本輪在新的有日期隔離目錄作必要修復，記錄來源 snapshot，讀取現有測試當回歸基準。本輪自有且驗證後的變更依 AGENTS.md 作本機 commit，只精確 stage 本輪路徑，不夾帶他人變更。不要因本機 commit 再停下來索取已由 AGENTS.md 提供的例行授權。

正式 engine 基準 SHA256：`39634da1f7f5059e7d7ee1999a1bb36342d791bc5bd696dad88ac8ffee4b8892`。本輪開始／結束均唯讀比對，保留保護範圍 diff，不操作正式服務。

## 3. 目前真實能力評估：不要重新宣告全通過

截至 2026-11-09，實際重跑現有測試為 Shadow 14/14、DEV test_repair 8/8 PASS；這兩組未開新 ASR 或電話，不代表產品驗收。

| 項目 | 已有證據 | 現在界線 |
|---|---|---|
| 正常 STT | 舊串流 Silero 每 32ms 重置狀態已在隔離副本修復；同一真人音訊可辨識並驅動路由 | 只證明有限單輪及片段組合 |
| P0 | listen_2、S05_1 有自然收句；組合路徑 [100,105,105,115] | listen_1 仍是 4 秒 bounded EOF；沒有連續真人多輪驗收 |
| A | v11.3 B 可保留；A 的歷史 1.5s oracle 可完整辨識 | 全長 fresh 的 A 有 5/9、7/9 波動，仍未完整保留；不是完整句保留率 |
| B | 固定 64 筆存檔 ASR，修 reference 後 ECHO 47/CALLER 7/UNKNOWN 1/FILLER 9，should 1、closing CALLER/should 0 | CALLER 減少不代表 FP 減少；歷史沒有 per-frame R0 TX，v11.3 fresh 缺可信 reference 而 UNKNOWN |
| C | 人工 interrupt 可讓 memory writer 停止，並驗證 ledger/hash | 新增測試使用真實 node-116 TTS PCM 的前 10 幀，非純合成波形；但程序、writer 與 trigger 仍是隔離控制，不是電話打斷 |

原 handoff「3 次 STT 空白」不精確：原始 log 是兩次 Dialogue 空白且 STT 0ms，另一次 vmcheck silent。先前瓶頸是送 ASR 前的收句判靜音，不可描述為 TEA 回傳三次空白。

目前 product_acceptance=NOT_ACCEPTED；正常連續電話、播放中短句保留、可靠回音隔離、電話端停播、自然多輪都不能升格完成。

## 4. S0：先修工具與基準的實際故障

以下是 2026-11-09 已讀到／重現的線索，不是讓你猜原因：

### 4.1 tail 入列仍使用舊型別，已有最小重現

5d97a8f 的主路徑把 `_pending` 改成 `deque(maxlen=2)`，但 `_create_tail_window` 仍有：

```python
if self._pending is not None:
    old = self._pending
    self._set_state(old["window_id"], ...)
self._pending = job
```

以 enabled=True、tail_ms=2000、FakeAsr 建立純記憶體 observer，on_playback_start → on_playback_end(reason="completed") → 75 個 320-byte 靜音 RX tail 幀，可重現：

```text
TypeError: sequence index must be integer, not 'str'
```

先寫會失敗的小測試，再修所有普通／tail 入列、worker 取出、close、stats 與 lifecycle 的型別契約。讀取所有 `_pending` 使用點，不可只修報錯行。不得讓 deque 自動淘汰而沒有 superseded 記錄；每個 job 必須有唯一可對帳的終態。只有確實重複的入列邏輯才考慮一個小共享函式，不能新增排程框架。

檢查回放是否真的走到 tail：現有 fresh 使用 recorded_boundary／recording_end，而 observer 對 reason="completed" 才開 tail。不能因 harness 根本未走 tail，就稱 tail 回歸已過。

### 4.2 baseline/fixed 目前不是排程演算法 A/B

`shadow_replay.py` 的 baseline/fixed 共用同一個已載入 SS 實作；主要差別是第三段 node 111/116 reference。這能比較 reference 修正，不能用來證明 deque 或 LCS guard 的因果改善。

建立最小可驗證載入方式，明確分開 algorithm snapshot、reference policy、輸入／模型／參數；檢查實際 import 路徑、hash、有效 policy/queue capacity。避免 sys.modules/sys.path 快取把兩個版本其實載成同一份。

### 4.3 LCS 測試尚未涵蓋新守衛分支

新增三句測試都預期 CALLER，但對該測試長 REF 的 coverage 約 0.0370/0.0185/0.0370，全部沒有跨過現存 `cov > 0.5` 守衛。因此 14/14 不足以證明該守衛不會壓掉真人。

核對 coverage 分母、novel ratio、共同片段守衛、stale/generation 與真正播放時段。補能確實到達守衛的邊界／相似內容／引用 TTS 再補充真人意圖之測試；合成文字契約測試要明標，不冒充真人錄音。共字比例只能是文字相似證據，不能單獨證明音源是回音。既有 `cov > 0.5` 本來就是閾值，不得稱它是「完全不涉及 threshold」。本輪不新增或調整閾值；若證明規則有害，只做有回歸證據的撤回／根因修復，不能單靠降低 CALLER/should 數過關。

### 4.4 檢查驗證器是否自我證明

現有 verify_results 主要檢查 oracle 片段、固定預期數量、局部 reference 改善，且把全長 retention 的 max coverage 作診斷。保留這些舊回歸，但新驗收必須看實際全長事件、有效分母與失敗清單，不能只印 ab_assertions=PASS。

## 5. 資料、來源與不可改造的歷史

優先使用現有原音及 metadata：

- v11.3 原始 debug 75.5 秒：`_workspace/acceptance/live/ht813/results/full_duplex_v11_3/live_tx6/`。
- 19095 歷史 RX/TX/mixed 各 52.7 秒及 AudioSocket debug 52.54 秒：`results/full_duplex_v11_4j/shadow_test/`，相對根為 `_workspace/acceptance/live/ht813/`。
- S05_1 原始 M4A、caller_8k 衍生 WAV、caller_metadata；onset 未確認不可補寫為已確認。
- DEV/results/run-KnOsVh7H、run-Ld5rU0Cu、queue_trial、run-fifo-QcmhL77H、run-pending4-HQarC1NS；先看各自 source hash、policy、完整與否，不混用不同 run 的數字。
- `F:\Backups\Ticenpi-Voip\v11_3_best_20261107\` 及保全說明。2026-11-09 可確認路徑存在，但讀 PRESERVATION.md 遭 AccessDenied，故本輪提示詞整理沒有重驗 320 檔或 bundle。接手時如仍不可讀，明說不可驗證；不要改 ACL、take ownership 或繞過限制，使用可讀 repo 原音繼續工作。

每筆來源記錄 SHA256、格式、取樣率、聲道、位元寬、samples、來源種類與衍生方式。保留歷史，不覆寫 results。

19095 debug 已有 6.09% 削波：RX 位移 +1760 samples 後乘 8 再截頂，可與 debug 419840 samples 完全對上；末尾 480 samples 無對應 RX。精確重演舊管線使用當時實際 debug bytes，不把乾淨 RX 當成原 STT 輸入。不同取音支路或乾淨 RX 的診斷須另列，不能混入演算法 A/B。不改正式增益或假裝能恢復已削波音訊。

現行正式與歷史隔離的模型／VAD 條件不同。核對實際 env、模組與 health，不能把設定預設當成已發生推論；歷史未保存的 backend/revision 保持未知。本輪 TEA 的 model revision、backend、timeout、tokens、前處理、gain、window/hop 全部凍結並記錄。

## 6. P0：建立真正完整的離線正常對話驗收

使用整段可讀真人原音按原始 frame 時序送入實際收句、STT、Call/walker 與 response PCM 路徑。驗證 VAD state、取樣率、端點原因、送入 ASR 的精確 bytes/hash、HTTP 結果、文字守門、node/path、response PCM hash 及完成原因。

不能以 EOF/None、追加人工靜音、剪出較佳片段、跳過 VAD 或直接塞 transcript 達成「自然收句」。EOF、remote_hangup、silent_timeout、natural_endpoint 必須分開。listener 被截斷而沒有自然尾端時，該事件是 INSUFFICIENT DATA，保留已可證明的部分並繼續其他事件，不把所有工作改成等 20 筆新錄音。

以實際 ASR 輸出驅動既有狀態機，驗證來處詢問、聽不清重播一次、拒絕結束、語音信箱、連續空白及正常多輪狀態不錯接；不得硬編碼真人句子去繞過 STT。構造的多輪 fixture 可以測狀態機，但標示「片段組合／非真實連續電話」。真人可懂度與自然電話回應仍需相應的人類／電話證據，不能從 CER 或單元測試推定。

## 7. A：播放期短句逐窗追查與公平排程比較

### 7.1 先產出完整時間軸，再選候選

每個 window 至少可查：case/run/version、generation/turn/node、原始檔 sample_start/end、observer sample_start/end、window_id、建立／入列／被取代／ASR start/end／error／decision 時間、queue depth、timeout、request ID、PCM/WAV hash、transcript、狀態、superseded_by、stale、終態原因。

注意 observer sample clock 可能只累計 PLAYING/tail，不能直接當整通電話的 sample clock。建立來源映射，沒有可信映射就不報真人 onset 延遲。歷史原始 log 不能倒填不存在的時間戳；新回放可記錄新時鐘並明標 replay。

對 v11.3 A/B 的每個缺字或漏失事件，逐層核對：RX 有沒有內容、播放期是否經過 VAD、切窗是否截字、窗是否建立、是否被 supersede、是否 timeout、模型是否回錯、分類是否覆蓋真人、是否 stale／episode reset。沒有進該路徑的 VAD 不可背鍋。

### 7.2 演算法與 reference 分開比較

先比較單 pending latest 基準與修正所有入列契約後的 bounded FIFO 候選；兩者保持相同 reference、模型及其他參數。再獨立比較 reference 修復／分類規則，避免同時改三件事。

先審閱既有 first-pending、FIFO、pending4 結果與版本，不能重複宣告舊失敗方案為新成果。只在已證明瓶頸下新增最小候選；不得因 pending 滿了就無限制加容量或並行 GPU。優先保持 1 inflight、最多 2 pending 的有限上限；若主張其他有限容量，必須事前寫出證據、上限、memory/age 代價與拒絕條件，不能事後選漂亮數字。

冷／暖機分開，A/B 相同暖機條件，一次只跑一組 ASR 負載。固定播放 1x、相同完整來源、同模型 revision、backend、timeout、tokens、window/hop、sample rate、前處理與 gain；至少 3 對完整比較，交替執行順序。GPU 其他負載若變動，記錄該 run 不可直接比較，不停他人服務。

HTTP timeout 不等於 server 推論已取消。追查逾時後殘留的 GPU 工作／排隊、跨 run 汙染與過期回應；只能清理自己擁有的離線程序，不能靠延長 timeout 消除錯誤。不得把已完成 callback 改掛到新 generation。

### 7.3 指標與失敗事件

先固定 normalization 與真值事件清單，再跑，不准看到結果後挑窗口或改判準。

- 完整句保留率：固定真人目標事件中，實際全長串流輸出完整保留指定句子的事件數／有效事件總數；報每句、每 run 的 n/N 與 CER。A/B 是兩個語音事件，不能拿字元 coverage 7/9 稱為 7/9 事件成功率。
- 單窗 max LCS coverage／歷史 oracle 片段只作診斷，不作成功條件。若需要拼接 partial，只能用事前定義的同 episode／同 generation 規則，記錄原片段，不能事後手工串成答案。
- ASR_ERROR、HTTP/模型錯誤、timeout、superseded、unresolved、stale、完整／部分／完全漏失各自列出數量與事件 ID。
- queue_wait、ASR inference、window_end→decision 的 median/P90、有效 n、最大 queue depth／最老工作年齡、實際送出與完成 request 數；記錄原始時間戳和計算公式。
- 只有可信 onset/end 才報 onset/end→decision 延遲。window_end→decision 不可包裝成真人反應延遲；無法量測時寫 NOT_EVALUABLE。
- 校驗每個已建立 job 的唯一終態及全體守恆，不能以丟棄錯誤或 UNKNOWN 降低延遲／錯誤分母。

候選必須沒有不可解釋錯誤增加、真人保留退化或不受控排隊，且提升在重複 run 可重現，才採納。若現有模型處理容量無法達到既定目標，交付容量證據與 FAIL，不靠改 timeout 或假 PASS 結案。

## 8. B：Reference 與 ground truth 各自驗證

追查 node_reference_text 的產生方式：來自原始話術、當時 TTS snapshot、目前檔案，還是 ASR/人工推論。當下 node WAV 存在，不等於歷史通話就是播放該版本；不得用待評估的 ASR 文字當自身 ground truth。

每個 window 建立最小 reference manifest：node/generation、文字與來源 artifact/hash、PCM 原始及播放轉換 hash、播放／write／drain 時間與時鐘、對齊方法與不確定度、來源可信等級、缺失原因。沿用專案既有 R0 等級定義；若缺定義，清楚分成逐幀 writer 證據、部分歷史播放證據、僅 catalogue/推論、UNKNOWN，不混成同一成功分母。

歷史未保存的 per-frame TX ledger 永遠不補造。離線新跑 writer 可產生自己的新 ledger，但只能證明 replay writer 交付與 drain，不能證明 Asterisk／電話喇叭曾收到或播放。原樣保留 node-116 reference 修復的有限結論。

reference 可信度與 caller/echo 的真值標註是兩回事。RX/TX mixed 有內容不等於有乾淨 caller-only；沒有獨立真值，不得把所有 CALLER 當誤報或所有 ECHO 當正確。無法確認的歷史 reference 保留 UNKNOWN，另列數量，不列入分類成功率。

在同時具可信 reference 與相應 ground truth 的資料中，分開列：caller recall、echo-only FP、silence FP、double-talk FN、決策延遲與有效 n/N。沒有該類樣本就 INSUFFICIENT DATA，不能填 0% 錯誤或 100% 召回。用事件級與 window 級兩種單位分開列，定義兩者的分母；不事後刪除失敗窗口。

允許用既有真人人聲加已知 TTS 的衍生混音、靜音、純 TX 測隔離契約，來源必須標示 DERIVED/SYNTHETIC，SNR/混音方法凍結；這些控制不能取代歷史真實電話的 double-talk 驗收。

## 9. 最小回歸、採納與回滾

先列修改要到達的 callers、tail、fixtures、tests、stats、工具與文件，再編輯。沿用既有測試，至少補：

- 普通和 tail 的 pending 空／滿／worker in-flight，以及 completed／hangup／close 的一致型別與唯一終態。
- superseded／timeout／ASR error／generation 變更後舊結果不錯接、新舊模組載入隔離、實際 max queue depth 與 policy 報告一致。
- 能真正跨進 LCS 守衛分支的控制，及真人拒絕／短句不被共字誤殺；未知 reference 不變成成功 ECHO。
- 真實 TTS PCM 經 memory writer 正常完成／人工中斷、write/drain 失敗、ledger node/hash/最後幀一致；不開聲學 barge-in。
- P0 的自然 endpoint、EOF、silent timeout、正常意圖／拒絕優先／重播上限與 PCM 回應。

每輪：原基準 → 第一個實際失敗點 → 最小修復 → 同音訊／同參數 A/B → 採納或撤回。失敗候選只保留在本機獨立結果與去識別化比較表，回復本輪自有副本，不動起始 WIP、正式檔或舊資料。不因難度改成只寫文件，也不為湊 PASS 擴架構。

## 10. 交付與最終判定

資料整理、log、manifest 與指標優先共用既有工具，不建立十幾份重複報告。交付：

1. 一個可直接在 Windows PowerShell 呼叫的離線重現入口，內部使用既有 WSL venv，建立獨立 run 目錄、記錄完整命令／環境／版本，結束清理自己的程序。
2. 完整逐窗 timeline、每個失敗事件、受控 A/B 表及可重算 median/P90；原始結果只放本機 ignored 目錄。
3. S0/P0/A/B/C 的 PASS / FAIL / INSUFFICIENT DATA（未執行用 NOT_RUN），分開寫單元、離線真音、衍生控制、真電話與正式部署。
4. S0 只有型別／基準／時鐘／守恆等契約真修好才 PASS；P0 缺自然完整輪、A 缺完整句、B 缺可信 reference 或真值時，不因測試數字全綠而升格產品驗收。
5. 修改 diff、來源 snapshot/hash、採納／撤回理由、精確 rollback 方法、本機 commit SHA、起始 5 檔 WIP 仍保留的狀態。
6. 按 AGENTS.md 更新受影響管理文件；先 diff 並保留既有未提交內容。未經同意不把起始 5 檔一起 stage。需要整合文件但不能安全區分 hunk 時，將本輪交接寫入獨立文件並列清需合併的位置，不擅自覆寫。
7. 保存一份完整交接 Markdown，最終 IDE 對話直接貼出全文，不只提供路徑或說「請看報告」。交接開頭直接回答：現在是否能正常對話、證明到哪一層、哪一項仍失敗、下一步最小必要動作。

結案前確認正式 engine hash／保護範圍未變、既有服務未被操作、自己的離線程序已停止、沒有私密資料被 stage。若仍缺真人實測，明寫真電話／全雙工未驗收；也要指出仍存在的離線缺口，不能把所有未完成事項都推成「只差一通真人電話」。

不要回覆「等你決定下一步」而漏掉已授權的本輪工作；完成能獨立處理的部分。真正無法存取來源或缺少不可替代證據時，保留精確阻礙與已完成結果，不能補造證據。

## 執行模型設定來源

- 官方模型名稱與 API alias：<https://www.deepseek.com/en/news/deepseek-v4-1-flash/>（`deepseek-flash`）。
- 官方 Thinking 設定：<https://api-docs.deepseek.com/guides/thinking_mode/>（Thinking enabled、reasoning_effort high；使用既有 IDE 支援的相應設定，不為本任務改全域模型路由）。

任務結束標記：TICENPI_VOIP_DIALOGUE_SHADOW_BUILD_PROMPT_END
