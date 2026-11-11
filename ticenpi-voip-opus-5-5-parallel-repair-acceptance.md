# Ticenpi-Voip：OPUS 5.5 主控平行修復與一次完整驗證

你是本次工程唯一主控，使用者指定的模型名稱為 OPUS 5.5。先確認執行端實際可用的 provider/model ID 與推理選項；名稱只是使用者指定，不代表已驗證平台提供此型號。不得冒充模型、默默換模型、建立付費服務或把所有工作意外路由到同一高成本模型。

## 0. 任務與完成定義

接續現有成果，盡可能在同一輪自主完成：現況確認 → 保全 → 平行診斷與小幅修復 → 整合 → 公平 A/B → 回歸 → 本機提交 → 完整交接。

不要只寫計畫、重建測試框架、每完成一步就詢問使用者，或把可修的缺陷直接全部改標缺資料。已授權的離線修復、隔離測試及本機提交直接執行；缺少真人證據、權限或安全條件時，繼續其他獨立工作，最後一次列出精確阻礙。

完成是把可執行的工程事項做到有證據的結果，不是保證全部 PASS。FAIL 必須留下根因、候選與回歸；INSUFFICIENT_DATA 必須說清缺什麼、已搜過哪裡；不得把產品未驗收寫成完成正常真人對話。

目前重點：
1. A：播放期短句遺失，以及固定音訊窗的長推論。
2. B：可信播放 reference、caller/echo/double-talk 的獨立驗收。
3. P0/C：自然收句、真 STT、正確路由、真 PCM writer、自動停播與下一輪恢復。
4. 安全：修正並驗證日誌隔離、保存 WIP、無正式服務干擾。
5. 整合：一次完成受影響測試、至少三組有效性能對照、本機提交及可恢復交接。

本提示詞只授權下述本機離線／封閉 DEV 工作，不授權正式部署、新真人外撥、正式全雙工或雲端同步目錄刪除。

## 1. 鎖定現場與先讀內容

唯一 Git repo：
F:\00-Ticenpi-SaaS\Ticenpi-Voip

禁止修復或自動退回：
D:\GOOGLE雲端\AI\Realty\SAAS_B\VOIP

BASE：
F:\00-Ticenpi-SaaS\Ticenpi-Voip\_workspace\acceptance\live\ht813\parallel_repair_20261109

DEV：
BASE\source\offline_repair_20261107

SDEV：
BASE\source\v11_4j_dev

上面 BASE／DEV／SDEV 是本文縮寫，執行時必須展開成完整路徑，不可當成既有環境變數。

開工記錄 pwd、Git root、branch、HEAD、status、staged diff、活躍 Agent／測試程序／GPU 使用者及端口擁有者。預期最新已知 HEAD：
c92dd055d9af26fca32f10152c76789e37ff18e3
程式修復：
67462a8e8ef2c97baff42fe7634d233339e4e2ec

若 HEAD 已往前，先看新 diff、祖先關係與現有工作，保留更新，不要求退回舊版本，也不硬套本提示詞的舊結果。真正 Git root 不正確時先切到 F 槽；無法訪問就明說，不建立替代 repo、不捏造結果。

先讀：
- F:\00-Ticenpi-SaaS\Ticenpi-Voip\AGENTS.md
- F:\00-Ticenpi-SaaS\Ticenpi-Voip\docs\VOIP_STATUS.md
- F:\00-Ticenpi-SaaS\Ticenpi-Voip\docs\VOIP_ACCEPTANCE.md
- F:\00-Ticenpi-SaaS\Ticenpi-Voip\docs\VOIP_RUNBOOK.md
- F:\00-Ticenpi-SaaS\Ticenpi-Voip\docs\VOIP_OFFLINE_REPAIR_HANDOFF.md
- BASE\RECOVERY_HANDOFF_20261111.md
- BASE\repair.diff
- 上述兩個 commit，以及當前恢復 source 與測試。

已有有效結果：
BASE\results\resume-20261111-XSee2dZJ
補充診斷：
BASE\results\resume-20261111-ubIDOnhF

先查 acceptance.json、stage-metrics.json、failure-events.json、source/audio manifest、loaded-inputs.json、metadata-post-audit.json、p0-writer-integration.json、逐輪 JSON/JSONL 及 TEA trace。不要只信 handoff 數字。

既有快照：
F:\Backups\Ticenpi-Voip\codex_resume_20261111_160534
結案快照：
F:\Backups\Ticenpi-Voip\codex_resume_20261111_160534\closure-c92dd055

歷史 commit 先前已核對：
20816cd8ae19664fa8144152d4007564c084df94
→ 1a80ef9a564a7151892c871360f3bc7b5ef84765
→ 5d97a8fe644f8177462a768e84d03055d70e6a4d
→ 67462a8e8ef2c97baff42fe7634d233339e4e2ec
→ c92dd055d9af26fca32f10152c76789e37ff18e3
仍須以當前 Git 驗證，不把原始修復 commit 當最新完整基準。

## 2. 已知事實與不能重犯的問題

以下是先前結果摘要，需對本機證據核對，不是新測試結果：
- Shadow 29、P0/Writer/舊回歸/證據 25，共 54 CPU 測試通過。
- 全長真音回放，兩錄音 × pending容量1/2 × 三組，GPU序列。
- v11.3 兩個有歷史支援區間的完整目標：容量1合計2/6，容量2合計3/6；各有3次 timeout。不是舊9個目標的coverage，不能混用分母。
- 容量2三組 v11.3 與 HT813 的決策P90均較差；FIFO=2沒有被驗收為性能改善。
- 六次 v11.3 timeout 同為 fresh-72、來源49.88–51.38秒；第一輪HTTP約2003.3ms失敗，GPU實際約11066.6ms，清理約9073.8ms。
- 短句A相關固定grid窗窮盡12個，完整句0/12、error0；歷史oracle診斷可辨識。不能人工挑oracle當即時成功。
- 真TEA＋真TTS的受控整合有兩段真人片段，狀態路徑100→111→115；941幀與941筆ledger一致，手動cancel後舊世代0幀。是COMPOSED_REAL，尚不是自動聲學打斷或真連續電話。
- B的可信歷史per-frame TX加獨立分類真值分母仍不足，不能用CALLER數量下降當改善。
- 先前已有新播放清ring、立即切generation、close終態不被晚回應覆寫等修復，不得回退。
- 非連續LCS cov>0.5即判ECHO的候選已撤回。原substring guard仍須檢查錯殺真人措辭的風險。
- 曾因timing預設路徑追加4筆離線紀錄至共用log，已修正為每run專用timing。不要刪改共用log或隱藏此事件。
- 有效舊A/B未於開始保存marks metadata hash；事後證明實際窗與PCM一致。新runner已補metadata hash，不得回填假造舊manifest。
- 測量後只補日誌／metadata，最終程式SHA不等於舊測量harness逐位元相同。這次有演算法變更必須用完整指紋的新對照。
- 舊v11.3最佳備份部分ACL拒讀；另有15個可讀歷史檔保全。不得說整個最佳runtime已全部重驗。
- 前次OmO的S/P成果已恢復，E原來只有調查；原Go背景任務因額度停止。不要重做session救援，也不要把過期bg ID當仍在工作。

本輪固定ASR基準先核實：
TEA-ASR-1.1、transformers、快取revision bda08df76d4fd6b487b4a1dd7f0bddf8541696f8；
先前套件 qwen-asr0.0.6、transformers4.57.6、torch2.9.1、soxr1.1.0；
max tokens96、timeout2秒、window1500ms、hop250ms、tail2000ms、1x回放。
不要猜9090與歷史19095用了同一模型；查實際啟動設定與來源證據。正式環境只允許必要唯讀核對。

## 3. 先保全，再凍結公平基準

由主控先做，不得讓子Agent先改演算法：
1. 建立新的具時間戳快照，保存當前status、binary diff、staged diff、未追蹤必要成果、逐檔SHA256、Git bundle並驗證可讀回。不得覆蓋舊備份。
2. 起始已知dirty包含.omo/omo.jsonc、原offline_repair的test_repair.py、原v11_4j_dev/tests/test_echo_guard.py；另有.opencode/與data/_backup/。現場可能更多，全部先分類A既有WIP/B上輪成果/C本次新改。
3. 禁止reset --hard、clean、整樹覆蓋checkout、git add -A、git stash亂藏他人工作；禁止把Git archive當完整WIP備份。
4. 明確凍結可執行baseline source與實際載入依賴，在F槽repo內隔離保存，含當前未提交相關修復。私人素材只唯讀共用，不複製進Git。
5. 固定原音manifest、marks metadata、reference catalogue、state tree、corrector、套件／模型revision及模型參數的hash；確認source loader真的載入baseline或candidate，不能靠錯誤全域路徑碰巧通過。
6. E先執行一次必要的當前基準／代表性失敗對帳；可沿用逐位元相同且可信的既有CPU證據，但修後harness需要的新基準不能用舊表冒充。
7. 對日誌、state、cache、臨時檔、trace、TX ledger指定隔離路徑；確認所有網路請求僅自有loopback，禁止模型下載、雲端ASR與靜默fallback。

## 4. 平行派工與檔案所有權

最多同時三個worker加一個主控；若平台槽數較少就分批。CPU程式與唯讀調查可平行，所有GPU/ASR驗證只有E可執行，其他人只交請求。

先查實際可用且已授權的模型與剩餘額度。主控OPUS負責根因、取捨與整合；範圍清楚的閱讀、測試、對帳可用現有較省token模型。不得新增供應商或付費API；不自動重演所有worker實際都跑昂貴主模型的情況。記錄每工作實際model ID。未提供子Agent功能時，在同輪按相同責任順序執行，不虛稱平行。

每個worker只收到：本任務摘要、自己的檔案、共享manifest位置、輸入／輸出契約、禁止事項及驗收門檻。不要複製整段對話與所有大log。每人回傳根因、改檔、測試／artifact路徑、待驗證風險；主控按需讀原證據，不只信摘要。

### S：短句與Shadow演算法

唯一寫入權：SDEV\semantic_shadow.py及SDEV\tests下相關測試。
不得改DEV runner、engine_patched、正式engine或模型參數。

- 逐窗重建created→pending→superseded／started→ASR結束／timeout→decision→terminal；對照真音source sample區間、generation、VAD／episode邊界與每層hash。
- 分清短句被VAD排除、窗切斷、queue丟棄、解碼截斷、服務長推論或其他原因；固定grid0/12完整不能再只靠增加pending修。
- 檢查現有EpisodeTracker是否能重用來做因果自然收句／完整episode保留或避免重複重疊工作；不得另造大型框架。
- 「因果」表示當時尚未到達的音訊、事後轉錄、人工標記與oracle不能參與切窗或排程。標註僅供盲評。
- 優先最小一個有根因證據的候選；需要時最多三個實質不同候選，不以換threshold湊候選。
- 模型、gain、VAD threshold、timeout、token上限固定。若比較不同切窗演算法，輸出每個窗長與邊界，把它明確列為受評演算法差異，不能聲稱仍是全部相同1500ms窗；禁止人工裁最佳窗。
- 所有pending、inflight、episode PCM buffer均有明確上限與丟棄／終態紀錄；不可把ASR工作轉存到無界背景thread逃避容量限制。
- 保留已修ring/generation/close行為，測快切播放、尾段、close timeout、晚到結果與跨輪。
- 若B分類也需改semantic_shadow，由P/E提出具體證據給S；只有S寫核心檔，不得多Agent同檔。

### P：P0、Writer與reference傳遞

唯一寫入權：DEV\engine_patched\saas_streaming_server.py、walker.py、test_p0_dialogue.py、test_p0_writer.py、test_repair.py。
不得改S核心或E runner；接口修改先給主控與E。

- 驗證真人RX→VAD自然收句→真STT→正確state transition→真回應PCM→LISTENING／合理結束。
- 分開ht813_listen_2、S05_1與bounded EOF的ht813_listen_1；不得補靜音把EOF偽裝自然收句。
- 在封閉DEV把實際caller判定接入真正的writer取消路徑；保持正式barge-in關閉。只有離線memory／自有loopback sink，不能接Asterisk或電話設備。
- 分開證明：人工cancel契約、文字／合成控制訊號、自有隔離聲學判斷→cancel、真人片段組合回放。不得用手動設bargein旗標取代聲學鏈路證據。
- 每個frame記generation、node、reference文本版本或hash、PCM hash、sequence、write/drain時間、cancel發出／接收時間、最後實送frame與terminal。ledger必須是實際writer產物。
- 驗證cancel後停止舊世代實送，沒有重送殘留、音訊queue污染、無限等待或錯誤mode；新真人STT與下一輪回答仍可完成。
- 同時涵蓋echo-only、silence、caller-only、double-talk、重複／引用TTS措辭、ASR error／timeout、write/drain failure、closing node等可取得情境。
- 若來源是拼接真人＋TTS或模擬回音，標COMPOSED_REAL或SYNTHETIC。電話線真正停播與自然真人連續多輪仍須獨立驗收。

### E：證據、reference稽核與唯一GPU執行者

唯一寫入權：DEV內replay.py、shadow_replay.py、tea_trace_server.py、recovery_acceptance.py、report_recovery.py、diagnose_windows.py、run_offline.sh、test_recovery_evidence.py，以及本輪新的ignored結果。
不得改S/P擁有的檔案或正式engine。

- 重用既有runner；補缺漏，不另建並行評測平台。
- GPU加跨程序單一租約／鎖，記擁有者PID與run ID；鎖已占用不得搶占、kill或猜測stale。只清理自己啟動的服務。
- baseline與candidate各有完整source/input/model/metadata指紋；每輪前後核對。metadata與模型revision缺失的run不得稱完整公平新基準。
- 對固定長推論窗，逐層核對PCM字節、WAV header、sample rate／duration、聲道、gain／clipping、prep樣本、輸入hash、server lock、infer/decode時間、輸出／錯誤。可增加觀測，不可偷偷調decode參數或提高deadline。
- 對2秒HTTP失敗與GPU後續執行分別記時；timeout必須算失敗，排空等候不能作延長timeout的成功。若不能安全釋放自有GPU工作，停GPU lane，其他CPU工作繼續。
- 追查歷史node/TTS reference來源、實際TX WAV與時間映射；能查實的才提升可信度，只有文字目錄或推測node不算實際播放PCM。
- 不因沒有ledger就完全略過現存TX；檢查原始TX/混音/metadata能否提供可驗證的時間與來源約束。若只能部分對齊，分層報partial，不假裝逐幀R0。
- 新DEV可錄得真正writer ledger與已知產生的TX reference，但必須標DEV_CAPTURED；不得補造歷史ledger或把合成回音當歷史真人回音。
- 分類標記需獨立於待測ASR／EchoGuard輸出。不得用模型自己的CALLER/ECHO結果當ground truth。
- 可信reference與獨立真值同時符合的事件才計caller recall、echo-only FP、silence FP、double-talk FN。其他保留UNKNOWN，逐事件說原因；分母0則null，不報0%。
- 有足夠合法本地資料才擴大案例；不能只挑容易句子稀釋舊失敗。新增案例與原核心固定集分開報告。

### 主控：安全、整合與交付

主控唯一寫管理文件與最終交接；審查S/P/E接口、移除無效候選、負責commit。
其中一個worker完成後，重用該worker做另一模組唯讀審查；審查者不得自審後直接宣布PASS，不得另開GPU。
根因跨檔時主控明確轉交所有權或由原owner修改，禁止兩個人同時改同一檔。
主控須追蹤原任務完成程度，不得因某worker「完成」就忽略整合FAIL。

## 5. 公平A/B與採納規則

先凍結評分契約，不可在看到候選結果後改分母／門檻：
- 主核心集保留v11.3全長與HT813全長；已有S05_1等自然收句素材按原來源納入。
- 原兩個支援區間目標保留，其他歷史九目標只有來源與文字真值可證明時才加入獨立表；不能把2/2寫9/9。
- 固定模型、revision、推理參數、timeout、gain、回放速率、暖機、資源條件、評分方法與原音。
- 相同算法參數全部固定；被刻意改的切窗／排程演算法單獨列差異。基準與候選都不得使用oracle排程。
- 有意義的候選至少三組有效成對對照，交替順序AB/BA/AB，單GPU序列；失效run列原因、不得挑三次最好結果。
- 同一批必須涵蓋完整音檔、尾段flush、下一輪、回音與失敗窗；新修影響項目重新測，未改且指紋相同的舊CPU證據可沿用。
- 同時輸出完整句n/N、漏句列表、字詞／語意正確性（有獨立真值才評）、ASR_ERROR、timeout、superseded、stale、unresolved、queue/PCM buffer峰值。
- 延遲分開報收句等待、排程等待、HTTP、server lock、infer/decode、timeout後清理、window-ready→decision的median/P90；真人onset未知不可杜撰端到端反應延遲。
- 提供每個失敗事件的來源sample／hash、job ID、reference可信度、完整timeline、實際輸出與根因。私人轉錄與錄音只留本地。
- A採納門檻：核心完整目標不可減少、原主要漏句須實際改善、不得新增timeout／ASR_ERROR／未終結job；三組D90不可較同對基準退化。若聲稱A全部通過，所有有真值核心目標須每組完整保留且無原故障殘留；部分改善只能PARTIAL。
- B採納門檻：對可評分固定集不得新增錯殺真人，回音誤觸須可對帳下降或維持原本已零，所有子類分開報；資料不足不得驗收整體B。
- C採納門檻：實際writer frame與ledger一致，cancel後舊世代無越界送幀（明列協議／已在途frame邊界），下一輪真STT及正確回答通過；聲學自動路徑與人工控制路徑分表。
- 候選有正確性改善但延遲退化，不可宣稱全部解決或自動採納；保留候選與精確失敗證據，使用最後安全版本。
- 每次先重現失敗→最小修復→同素材回歸；無效只撤回本候選的owned hunks，不破壞A/B既有修改。
- 同一根因最多三個有證據的實質候選無改善時，停止盲調，做診斷與獨立審查，保留FAIL並處理其他工作。不得把多次重跑相同失敗當新候選。

## 6. 隔離安全與D槽處置

不得：
- 修改、重啟、部署正式9090／19095／19110、Asterisk、HT813、正式gain／ASR／barge-in。
- 真人外撥、接入真人新測試、使用AMI撥號或批次電話。
- push、merge、force push、rebase，或公開私人錄音、轉錄、電話號碼、金鑰。
- 新增付費API／外部服務、繞過ACL、下載模型、修改全域IDE模型路由。
- 任意kill服務或重用未知owner的端口／GPU。
- 刪改共用timing log以掩蓋前次追加。

每輪前後核對正式source hash、必要服務PID／啟動時間、端口owner；只做讀取，不改正式設定。共享日誌可能被正式服務自然追加，不能只憑整檔hash變化就斷言本輪寫入；同時查離線UID與實際logger目的地，記錄無法歸因處。

D槽只做必要唯讀差異／備份可還原核對。現有完整ZIP並不證明雲端刪除安全，獨有DB/WAL/SHM不能當垃圾。本輪未授權雲端同步刪除；保留D原樹及舊信任記錄，確保本輪派工root防呆即可。不得為了「全部完成」擴大破壞性範圍。

若資料足夠就完成離線自動聲學PoC；若必須新真人連續電話才能證明最後一關，明列LIVE_ACCEPTANCE_PENDING，不請使用者立即外撥，也不自行解除正式barge-in限制。

## 7. 執行節奏、停止條件與token控制

- 主控先凍結基準／所有權，再同時派S/P/E。等待GPU時做CPU修復、證據稽核及文件草稿；不要平行跑ASR。
- 短進度只報新根因、驗證結果、阻礙與接續動作，不貼大log、不反覆向使用者索取現存資料。
- 每個worker輸出精簡結果與artifact索引；大JSON／音訊不塞進主控context。共用一次manifest，不讓每人全庫掃描或重讀全部session。
- 每個新根因給最小必要回歸；只對通過診斷的候選做完整三組，避免每個微小改動重跑全GPU矩陣。
- 子Agent額度或工具失效時保存當前patch與checkpoint，由主控接手剩餘可做工作，不停止於「worker失敗」。
- 如果context／執行時間可能中斷，先更新既有工作紀錄，寫當前commit、dirty檔、run ID、是否仍有自有GPU工作、恢復指令與未完成項；不能將WIP寫PASS。
- 只有可做的離線工程完成、同根因候選上限加獨立審查後仍FAIL、缺必需外部證據，或明確安全阻礙時才結束。不得只交下一輪提示詞替代本輪執行。
- 本輪只交一份完整結果；不要每個小步驟收工要求再交辦。部署／真人證據門檻獨立保留，不因此停止其他修復。

## 8. 最終交付：一次自足、可重現、可恢復

主控審查整合後：
1. 重跑所有受修改影響的CPU／接口／隔離測試，取得可採納候選的三組公平A/B與所有失敗事件。
2. 唯讀獨立審查：caller錯殺、race／late result、queue上限、真值洩漏、假PASS、錯模組、共用log、GPU並行、敏感資料與正式副作用；有問題先修再驗。
3. 僅更新既有VOIP_STATUS、VOIP_ACCEPTANCE、VOIP_RUNBOOK及既有完整交接，不增生多份互相衝突的總表。
4. 明列目前程式變更來自A/B/C哪一類、baseline與final實際source SHA／模型／參數、採納或撤回理由、各三組原始數據、絕對路徑測試指令、修復diff及rollback。
5. 正式未部署就說沒有部署可回滾；程式rollback只針對本輪commit／owned hunks，先保全WIP，禁止reset/clean。
6. explicit path／hunk staging、本機commit、讀回commit內容、檢查殘留dirty與私有資料未被提交。若最終harness與測量版本不同，說明差異、補必要驗證，不硬把舊run綁成最新SHA。
7. 完整交接存檔後重新讀回，核對首尾、非空、行數與內容，IDE最終回覆直接貼全文，不只給路徑。
8. 最終總表逐項列：S、A、B、P0、C人工／聲學自動／電話線、日誌隔離、歷史保全、D槽、安全狀態、本機commit與剩餘WIP。
9. 區分OFFLINE_PASS、PARTIAL、FAIL、INSUFFICIENT_DATA、LIVE_ACCEPTANCE_PENDING；真人連續電話、可信歷史reference、聲學自動打斷任一產品關卡未證實時，product_acceptance維持NOT_ACCEPTED。
10. 尚需外部證據者一次列精確缺口及最小未來驗收程序，不宣稱本輪已完成該驗收、不再要求使用者重述背景。

成功交付不是一張全綠表，而是最大範圍的真實修復、可核對結果、清楚的未驗收界線，以及下一位可直接接續的完整工程狀態。

現在開始：先確認F槽現場與既有成果，保全並凍結基準，再派S/P/E平行工作；持續執行到上述停止條件，最後一次完整交付。
