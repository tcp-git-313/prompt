# EchoGuard 全雙工優化交接文件
> 版本：2026-11-02  
> 任務代號：EchoGuard v11.4 — Full-Duplex Shadow Acceptance  
> 適用：Claude Code、Codex、Cursor、其他 IDE Agent  
> 專案：Ticenpi-Voip  
> 本文件用途：把目前已知的技術背景、歷史實驗、驗證結果、安全邊界與下一階段任務一次交接給新 IDE，避免重做無效實驗或誤動正式環境。

---

## 0. 新 IDE 開始前必讀

你接手的是一個已經完成多輪 PSTN／HT813／Asterisk 語音路徑實驗的專案。上一階段在 **RX 恢復 unity（0 dB 額外增益）＋TX 播放端 -6 dB** 後，已經重新觀察到來電者在 AI 播放期間的語音，且本次測試的 2 個事件都被辨識；但目前**尚未證明 EchoGuard 能跨通話穩定分離真人插話與 AI 回音，也尚未證明插話會即時中止 TTS**。

因此你的第一個目標不是直接開啟 barge-in，而是完成：

**v11.4：在 RX=unity、TX=-6 dB 固定條件下，做 Full-Duplex Shadow Acceptance（全雙工影子驗收）。**

Shadow 的意思是：系統照常播放 AI 語音，EchoGuard 在背景計算「如果正式啟用，這個時間點是否應該中止 TTS」，完整記錄判斷，但**絕對不能真的停止 TTS、不能改變正式通話流程**。

### 最重要的工作原則

1. 先讀現有程式、設定、測試與 v11.3 原始產物，確認目前實際狀態；本文是交接脈絡，不代表目前工作目錄一定與歷史紀錄完全相同。
2. 把「已經有證據支持」和「仍待驗證」分開，不可把單次通話的結果寫成已完成全雙工。
3. 優先重用歷史錄音與測試資料。不要為了補齊報表就要求使用者反覆打電話。
4. 不要先改分類門檻、VAD、ASR、增益或 HT813 參數。先在現況下取得可重現的 baseline。
5. 先完成只讀盤點與執行計畫，再進行限定在 PoC／Shadow 範圍內的工作。
6. 未經使用者明確授權，不得部署、推送、重啟正式服務、修改正式設定或開啟正式 barge-in。

---

## 1. 專案位置與系統定位

### 1.1 專案路徑

Windows 專案路徑：

`F:\00-Ticenpi-SaaS\Ticenpi-Voip`

歷史紀錄中明確提及的程式：

- `engine/saas_stt.py`：正式對話路徑使用的 STT 相關程式。
- `engine/tea_asr_server.py`：TEA-ASR PoC 服務。
- `results/full_duplex_v11_3/`：v11.3 TX -6 dB 實測產物目錄。

以上路徑必須在目前實際 checkout 中重新確認。若目錄、檔名或分支已變動，先記錄實際位置，不要自行猜測或建立同名替代檔。

### 1.2 硬體與語音路徑

目前架構概念：

1. 使用者透過一般電話線／PSTN 打入。
2. Grandstream HT813 的 FXO 端接收類比電話線，轉成 SIP/RTP。
3. Asterisk 負責通話路由與音訊橋接。
4. Asterisk 透過 AudioSocket 把音訊送到 Python 對話服務。
5. Python 服務執行接收音訊、VAD／ASR／語意判斷、LLM、TTS。
6. TTS 音訊再經 AudioSocket 回到 Asterisk，最後經 HT813 回到電話端。

簡化圖：

```text
PSTN caller
    |
Analog phone line
    |
HT813 FXO  <---- 類比兩線混合／回音耦合的重要位置
    |
SIP / RTP
    |
Asterisk
    |
AudioSocket RX (8 kHz slin16; app-side RX is already post-hybrid)
    |
Python voice application
    |-- VAD / ASR
    |-- EchoGuard / semantic interruption decision
    |-- LLM
    |-- TTS
    |
AudioSocket TX
    |
Asterisk -> SIP/RTP -> HT813 -> PSTN caller
```

### 1.3 核心聲學問題

HT813／PSTN 是類比兩線環境。電話端接收與發送聲音會在兩線混合器／hybrid 處產生耦合。因此：

- 使用者在電話端聽到 AI TTS。
- AI TTS 可能經過電話端／類比路徑再洩漏回 RX。
- Asterisk AudioSocket 收到的 RX 已經是 hybrid 後的訊號，可能同時包含真人聲音與 AI 回音。
- 目前沒有已證實可直接取得的 HT813 hybrid 前、乾淨 caller-only RX tap。
- MixMonitor 的 mixed audio 不能當成乾淨 caller reference。
- RTP inbound 也不能假設是 hybrid 前的乾淨聲音。

因此 EchoGuard 必須面對的不是單純「RX 有沒有聲音」，而是**RX 中的語音究竟是來電者插話，還是 AI 自己的聲音洩漏回來**。

---

## 2. 目前安全邊界（絕對不可違反）

### 2.1 正式環境

- 正式 Asterisk 應用埠：`9090`。
- 正式 dialplan context：`[saas-dialogue]`。
- 正式 barge-in：**OFF**。
- 正式環境 `SAAS_PLAY_GAIN` 維持未設定／0 dB，除非使用者另外明確批准。
- 不得修改正式 `9090`、`[saas-dialogue]`、正式 AudioSocket 連線路徑或其執行中的服務。
- 不得因測試方便而 reload／restart 正式 Asterisk、正式 Python service 或正式通話服務。
- 不得把 PoC 的設定複製到正式環境。

### 2.2 PoC／測試環境

歷史上使用過：

- `19090`
- `19091`

v11.3 收尾時曾驗證 19091 已關閉、9090 仍在 listening。這只代表當時的 restore 狀態；新 IDE 開始時仍需重新檢查目前 process、port owner、service 與設定。

任何測試都必須限制在明確識別的 PoC／Shadow 環境。若無法證明目前命令會打到測試服務而不是正式 9090，立即停止，不要執行。

### 2.3 其他禁止事項

在使用者沒有明確授權前，不得：

- 開啟正式 barge-in。
- 修改正式 Asterisk dialplan 或正式服務設定。
- 部署、推送、上線或重啟正式服務。
- 修改 HT813 的 LEC／NEC／阻抗／FXO／增益等硬體設定。
- 把 TX 改成 -3 dB 或其他新值。
- 切換正式 ASR 引擎。
- 引入 AEC3、DTLN、其他神經網路 AEC，或改走 Digital SIP。
- 把過去失敗的 AEC／阻抗路線重新當成預設下一步。
- 進行大量真人 PSTN 測試。先分析既有錄音；只有既有資料不足且能明確指出缺少哪一種測試樣本時，才提出最小化新增測試需求。

---

## 3. 目前已知的軟體與模型狀態

### 3.1 ASR 現況

歷史確認的狀態：

- 正式對話路徑目前使用 SenseVoice，相關程式為 `engine/saas_stt.py`。
- SenseVoice 是 final utterance 類型的辨識流程，並沒有原生 streaming partial transcript。
- TEA-ASR-1.1 是 PoC，歷史上由 `engine/tea_asr_server.py` 提供 WAV -> HTTP 辨識，測試埠曾使用 `19110`。
- TEA-ASR 也不是已證實的原生 streaming partial ASR；過去採用 sliding-window 方式做語意實驗。
- Qwen3-ASR-1.7B 曾有歷史 benchmark 成績（CER 0.091），但不能因此推論正式 runtime 目前已使用 Qwen。
- 不可把 benchmark 模型、PoC 模型與正式 runtime 模型混為一談。

新 IDE 必須直接檢查目前程式、設定與啟動參數，重新確認正式與 PoC 的實際 ASR engine。沒有證據時不得更換 ASR。

### 3.2 相關環境變數／控制項

歷史上出現過以下參數，請先搜尋程式與設定檔，確認命名、預設值、套用位置及實際生效時機：

- `SAAS_RX_GAIN`
- `SAAS_ASR_GAIN`
- `SAAS_DETECT_GAIN`
- `SAAS_PLAY_GAIN`
- `SAAS_BARGEIN_MODE`
- `SAAS_BARGEIN`
- `SAAS_PACED_PLAYBACK`
- `SAAS_PLAY_TEMPO`

特別注意：

- `VOLUME(RX)=8` 在 Asterisk 音量控制中曾造成約 x8 線性增益，約 +18.06 dB。
- 本任務的 `RX=unity`／`RX=0 dB` 是「不額外放大、維持 unity gain」，**不是把音訊振幅歸零**。
- `TX=-6 dB` 是播放端衰減 6 dB。v11.3 實測曾確認 PCM 數值確實有套用增益，但新 IDE 必須找到當時實際施加位置，確認它只作用於 PoC／測試路徑。
- 正式 `SAAS_PLAY_GAIN` 仍維持 unset／0 dB。不可因 v11.3 使用 TX -6 就改寫正式環境。

---

## 4. 歷史實驗與已知結論

以下是目前交接到的歷史脈絡。新 IDE 應在現有結果目錄找原始 JSON、CSV、WAV 與程式碼，重新驗證關鍵數字；若原始資料與本文摘要不一致，以原始資料為準並標示差異。

### 4.1 v6：Audio-only 分離路線失敗

- 曾測試只靠音訊特徵分辨真人聲音與 AI echo。
- 結果為 `AUDIO_ONLY_NOT_SEPARABLE`。
- AUC 約 0.5，接近隨機判斷。
- 結論：不要再只靠 RMS、能量或單一音訊門檻調參來解決 echo／caller 分離。

### 4.2 v7：語意輔助路線

- 發現 AI 自己播放的 TTS 內容，可能在 echo 回到 RX 後又被 ASR 辨識出來。
- 因此可以把「目前 TTS 正在說什麼」與「RX ASR 聽到了什麼」做語意比對，協助分辨回音。
- 但語意比對不代表已能穩定分離；真人可能說出與 TTS 相似的內容，必須避免誤殺真人插話。

### 4.3 v8：Sliding-window 實驗

- Sliding-window 語意判斷曾有 7/10 可用決策。
- 首個可用決策延遲中位數約 1500 ms。
- 曾有 3 個 double-talk false negative，但後續發現當時測試 fixture 在 caller onset 後只保留約 70 ms，樣本設計不足以驗證真實雙講。
- 結論：需要以完整、具 ground truth 的 caller fixture 重新評估，不能直接把那 3 個 FN 當成分類器實際失敗。

### 4.4 v9：資料集不足

- 歷史 caller clips 多為 backchannel，而不是持續的插話。
- 沒有足夠的 sustained interruption 樣本。
- EchoGuard 在 echo safety window 曾觀察到 0/420 false positive，但 caller 樣本不足，不能據此宣稱具備完整插話能力。
- 當時結論為 `CALLER_DATASET_INSUFFICIENT`。

### 4.5 v10：第一次 live acceptance 的協定污染

- 第一次 live acceptance 結果受到測試協定污染。
- 後續修正流程，不能把受污染的 v10 結果當成乾淨基準。

### 4.6 v10.1：修正協定後，舊增益狀態下的語意 gate 失敗

修正協定後的結果：

- Protocol pass。
- Semantic gate 在當時的舊增益條件下仍無法有效分離真人插話。
- Normal interruption recall：0/9。
- Short interruption recall：0/1。
- Raw caller false positive：1/290，其中有 1 個 pure-echo FP。
- 沒有 bridge／carryover 類型的誤觸發。
- 這是舊增益條件下的真實失敗，但**不能直接推論 RX unity + TX -6 的新增益條件也必然失敗**。新條件必須重新驗證。

v10.1 後修正的實測協定重點：

- Bridge 句子：`好，我想先了解一下，你們主要是在做什麼？`
- 只採用 A、B 兩個事件，不使用 C。
- Bridge 應在 AI 靜音時播放，之後至少等待 1.5 秒。
- B 事件之後保留至少 5 秒資料。
- 結尾再播放 closing，最後掛斷。
- A、B 的測試句子應依 normal／short interruption 分類並保留逐字稿 ground truth。
- 不可將 bridge 本身或 bridge 的回音當成 caller interruption。

### 4.7 v11.1：找到 RX 過度放大的根本混淆因子

關鍵發現：

- `VOLUME(RX)=8` 相當於 x8 線性增益，約 +18.06 dB。
- RX8 歷史 clipping median 約 4.7%，最高約 5.35%。
- Double-talk clipping 曾達約 12.85%。
- Post-gain peak 曾撞到 0 dBFS。
- Echo-only RX RMS 約 -9.2 dBFS，caller-only 約 -19.2 dBFS；當時 echo 比 caller 大約 10 dB。
- 對已經 clipping 的舊錄音做離線衰減，無法恢復被削平的波形，因此不能靠事後 normalize 還原 ASR。
- 當時缺少真正的 pre-gain capture。

結論：後續必須實際採用 RX unity／不額外放大，再做 TX 測試；不可再用 RX8 的舊錄音推論新音量條件。

### 4.8 v11.1：離線模擬的限制

曾做過離線 gain 模擬：

- G0 RX8/TX0：availability 3/6、coverage 0.5、CER 0.889。
- G1 RX0/TX0（離線模擬）：結果相近。
- G2 RX0/TX-6：availability 6/6、coverage 0.833、CER 0.222。
- G3 RX0/TX-9：availability 6/6、coverage 1.0、CER 0.167。

後續發現這些模擬高估真實 ASR 效果，不能拿來取代實際電話通話。新 IDE 不得只靠離線 gain transformation 宣稱新的實際通話效果。

### 4.9 v11.2：歷史資料盤點與 TX -9 dB 實測

歷史資料盤點：

- v10 + v10.1 共約 10 通歷史通話，21 個 interruption events，其中 20 個可用。
- 這些舊通話的 `PLAY_GAIN=0`，沒有真正的 TX gain variance。
- TX 音量與 echo RMS 的歷史 regression：slope 約 -0.232、R² 約 0.019，無法作可靠預測。
- 約 58.1% echo segments 的 peak ≥ -0.5 dBFS。
- RX8 造成的飽和嚴重。
- 離線模擬曾預測 ASR availability 約 0.70，但實際舊條件只有約 0.143，顯示離線模擬不可靠。

實際 TX -9 dB live calibration：

- 約 49.7 秒通話。
- 只有 1 通新通話，內含 2 個 normal events。
- RX=unity／不額外放大，TX=-9 dB。
- TX node RMS 約 -23.85、-28.14、-23.43 dBFS（以原始 v11.2 指標為準）。
- Peak 約 -9.87、-13.49、-9.85 dBFS。
- Echo-only RX RMS 約 -34.8 dBFS。
- clipping 0%。
- caller availability 2/2。
- coverage：A 約 0.778、B 約 1.000、中位數約 0.889。
- CER 中位數約 0.111。
- ASR 轉錄片段包括：`等一下啊我現在不`、`你哪裡打來的啊`。
- 當時結論：`LIVE_TX_REDUCTION_EFFECT_CONFIRMED`。
- 這仍是一通電話的工程訊號，不是統計上穩健的通話間差異結論。

### 4.10 v11.3：TX -6 dB 實測（目前最新狀態）

使用者報告最新結果：

`TX -6 dB 驗證完成 — TX6_SWEET_SPOT_CONFIRMED`

測試規模：

- 新增 1 通真人電話。
- 通話約 75.5 秒。
- 2 個 Normal interruption events。
- 沒有 repeat。
- 沒有測試 -3 dB。
- 沒有更動其他參數。

實際音量與訊號：

| 指標 | v11.3 TX -6 dB |
|---|---:|
| node100 TX RMS | -20.85 dBFS |
| node100 相對 TX -9 baseline | 約 +3.0 dB |
| node101 TX RMS | -20.59 dBFS |
| node101 相對 TX -9 baseline | 約 +2.84 dB |
| TX peak | 約 -6.87 / -6.92 dBFS |
| Full-call peak | 約 -7.03 dBFS |
| Full-call RMS | 約 -33.8 dBFS |
| Clipping | 0% |
| Echo-only RX RMS | 約 -33.5 dBFS |
| Caller-only RX RMS | 約 -29.0 dBFS |
| Caller 相對 echo | 約 +4.5 dB |

語音辨識與事件：

| 指標 | 結果 |
|---|---:|
| Caller availability | 2/2 |
| Coverage A | 1.000 |
| Coverage B | 1.000 |
| Coverage median | 1.000 |
| CER A | 0.000 |
| CER B | 0.000 |
| CER median | 0.000 |
| Echo-dominant events | 0 |
| EchoGuard／事件觀測 | 2 個 Normal events 均有 caller 可用訊號 |

轉錄片段：

- A：`等一下我現在不方便`
- B：`你哪裡打來的`

與 TX -9 的解讀：

- v11.3 的 caller coverage／CER 在這一通測試裡至少沒有比 v11.2 差。
- TX -6 的 TX 音量比 -9 約大 3 dB。
- Echo-only RX RMS -33.5 dBFS，比 v11.2 TX -9 的 -34.8 dBFS 高約 1.3 dB。
- 在 v11.3 這通資料中，caller-only 約 -29.0 dBFS，比 echo 約大 4.5 dB。
- 這是值得保留的候選 sweet spot，但 v11.2 和 v11.3 是不同通話、每個條件都只有一通，不能宣稱已完成嚴格 controlled A/B 或跨使用者穩健性驗證。
- 不應為了追求更大音量而自行嘗試 -3 dB。

v11.3 restore／安全結果：

- extensions.conf before/after byte-identical。
- restore PASS。
- extensions.conf SHA256：`4b9b7628…9fa6`。
- `[saas-dialogue]` 未變更，SHA256：`6595f292…18ab4`。
- 19091 已關閉。
- 9090 仍在 listening。
- 正式 barge-in 保持 OFF。
- HT813 未更動。

歷史產物目錄：

`results/full_duplex_v11_3/`

紀錄中提及：

- `preflight.json`
- `live_tx6_metrics.json`
- `live_tx6_events.csv`
- `live_tx6` 原始音檔與 marks
- `tx6_vs_tx9.csv`
- `restore_report.json`
- `restore/`

新 IDE 必須確認以上檔案是否存在、內容是否完整，並讀取原始資料。不得只根據此摘要複製數字。

---

## 5. 現階段技術結論：哪些已經成立、哪些還沒有

### 5.1 已有證據支持

1. 過去 `VOLUME(RX)=8` 的 +18.06 dB 額外增益會造成嚴重 clipping，是舊測試的重要混淆因子。
2. RX 恢復 unity／0 dB 額外增益後，訊號不再出現 v11.1 那種 clipping。
3. 在 RX unity + TX -9 的單次 live test 中，2/2 caller events 可用。
4. 在 RX unity + TX -6 的單次 live test 中，2/2 caller events 可用，coverage 1.0、CER 0.0、clipping 0%。
5. v11.3 這一通錄音中，caller-only RMS 約比 echo-only 高 4.5 dB。
6. v11.3 restore 成功，正式 9090 未被改動，barge-in 仍關閉。
7. RX unity + TX -6 是目前合理的 Shadow 驗收候選設定。

### 5.2 尚未證明

以下全部仍是待驗證，不得對外寫成已完成：

1. EchoGuard 能在不同通話、不同來電者音量、不同說話距離下穩定分離 caller 與 echo。
2. Semantic classifier 在 RX unity + TX -6 下已經不會誤把 echo 判斷成真人插話。
3. Echo-only、無人說話、TTS 內容重複、double-talk 等情境不會造成錯誤中斷。
4. 插話判斷足夠即時。過去 sliding-window 的 first usable latency 中位數約 1500 ms，仍需在新設定重新測量。
5. 真正停止 TTS 的動作能正確執行且不會造成通話狀態錯亂。
6. 完整自然的 live full-duplex 對話已經完成。
7. TX -6 對所有使用者與所有電話線環境都是最佳值。

**請始終分清楚以下五個里程碑：**

- P1：TTS 播放期間，RX 能觀察到 caller 語音。
- P2：ASR 能辨識這些 caller 語音。
- P3：EchoGuard 能可靠分辨 caller interruption 與 AI echo。
- P4：分類器觸發後，TTS 能正確且及時停止。
- P5：整通電話能自然完成真正的全雙工互動。

目前 P1、P2 有初步 live evidence；P3、P4、P5 尚未正式驗收。

---

## 6. 下一階段任務：EchoGuard v11.4 — Full-Duplex Shadow Acceptance

### 6.1 核心目標

固定使用目前候選增益：

- RX：unity／0 dB 額外增益。
- TX：-6 dB。
- 現行 semantic classifier：先凍結，不可先調 threshold。
- barge-in：維持 OFF。
- 只產生 shadow decisions，不允許實際停止 TTS。

目的是驗證在新 gain staging 下，舊 semantic gate 是否能重新分辨真人插話與 AI echo，並測出可用決策延遲和誤判風險。

### 6.2 Phase 0：只讀環境盤點

先完成以下檢查並輸出報告：

1. 確認目前 checkout 的專案路徑、Git branch、HEAD commit、working tree 是否乾淨、有沒有其他 IDE 正在修改的檔案。
2. 掃描正式與 PoC 啟動方式、port、process owner、Asterisk context、AudioSocket endpoints。
3. 確認 9090、19090、19091、19110 目前分別由什麼服務監聽；不得只依賴歷史紀錄。
4. 確認正式服務目前的 barge-in 狀態、`[saas-dialogue]` hash、`SAAS_PLAY_GAIN` 實際值與來源。
5. 定位 v10、v10.1、v11.1、v11.2、v11.3 原始產物。
6. 確認 v11.3 metrics、events、marks、錄音和 restore report 是否一致。
7. 讀取實際程式，找到：
   - RX 音訊 gain 的套用點。
   - TX 音訊 gain 的套用點。
   - AudioSocket RX／TX 的 PCM format。
   - TTS 播放期間 RX capture 的生命週期。
   - VAD、ASR、sliding window、語意 gate、EchoGuard decision 的資料流。
   - 目前如何記錄 TTS text／播放時間戳／RX transcript。
   - interruption decision 是否已能在 shadow mode 下記錄而不觸發 stop。
8. 先建立一份「文件摘要 vs 實際 source」差異表。

**Phase 0 不修改任何檔案。** 發現未提交變更時，不要覆蓋、stash、checkout、reset 或清理其他 IDE 的成果。

### 6.3 Phase 1：重建可重現 baseline

優先使用歷史資料，不新增電話：

1. 匯入並盤點 v10／v10.1／v11.2／v11.3 的可用 recording、event marks、ground truth、metrics。
2. 每個 event 必須能對應：
   - call ID。
   - event ID。
   - event 類別（normal、short、echo-only、silence、double-talk 等）。
   - caller onset／offset。
   - TTS onset／offset。
   - ground-truth phrase。
   - ASR transcript。
   - Shadow decision。
   - decision timestamp。
3. 先弄清楚既有 `availability`、`coverage`、`CER` 的計算定義，避免不同版本指標名稱相同、實際計算不同。
4. 分開計算：
   - Caller-only。
   - Echo-only。
   - Double-talk。
   - TTS playback 後的 caller speech。
   - Silence／no caller speech。
5. 不能把 v10.1 舊 gain staging 的 semantic failure 混在 v11.3 新增益資料中，合併統計時必須標示 gain profile。
6. 不能只報 aggregate accuracy；必須保留 event-level 結果與 confusion matrix。

### 6.4 Phase 2：凍結條件的 Shadow Replay

在不觸發真實 TTS stop 的前提下，對歷史 recordings 重播目前 classifier：

- RX unity。
- TX -6 的對應真實錄音優先；舊錄音不可以透過離線 gain 假裝成新環境實測。
- classifier／threshold 使用當前實際版本。
- 不修改分類門檻，不先調參。
- 對每個 frame/window 輸出 shadow decision。

每筆 shadow decision 至少記錄：

- call_id、event_id、timestamp。
- TTS state（播放中／靜音／closing／其他）。
- RX window start/end。
- RX RMS／peak／clipping。
- ASR text、ASR availability、ASR confidence（若系統實際提供）。
- 正在播放的 TTS text 或對應 TTS segment ID。
- caller／echo semantic match features（若系統有）。
- classifier score、threshold、decision reason。
- `would_interrupt=true/false`。
- first usable decision latency。
- ground-truth label。
- 任何 stale window、carryover、bridge、closing contamination 標記。

Shadow mode 必須是純觀察，不得呼叫 stop、cancel、flush 或其他會改變 TTS 生命週期的函式。若目前沒有真正隔離的 shadow path，先設計最小侵入的 PoC 實作，不得為此碰正式服務。

### 6.5 Phase 3：測試矩陣

至少檢查下列情境，並確認每種情境有足夠且可追溯的 ground truth：

| 類別 | 情境 | 主要風險 |
|---|---|---|
| Normal interruption | caller 在 AI 說話中正常插話 | 漏掉真正插話 |
| Short interruption | caller 短句／短促插話 | 首個可用 window 太晚 |
| Echo-only | AI TTS 回音，caller 沒有說話 | 把 AI 自己的聲音當成 caller |
| Silence | 沒有 caller 語音、只有底噪 | noise false trigger |
| Double-talk | caller 與 TTS 同時說話 | caller 被 echo 淹沒或被語意 gate 排除 |
| After-playback caller | TTS 剛停止後 caller 才說話 | stale TTS text／window carryover |
| Similar wording | caller 說出與 TTS 相似的字詞 | 語意比對誤殺真人 |
| Bridge／closing | bridge 或 closing 的播放與回音 | protocol contamination |

每一類樣本不足時，請明確列出缺少的樣本種類，不得以重複同一段錄音冒充多個獨立通話。

### 6.6 Phase 4：必要時才安排最小化真人測試

歷史資料分析完成後，如果仍有明確缺口，才提出新測試需求。

建議先以 3–5 通為工程驗收樣本範圍，而不是無限制增加電話；這只是小型工程樣本，不代表統計上已足夠代表所有使用者。優先讓每通測試能同時覆蓋多個事件類別。

使用 v10.1 修正後的 A/B protocol：

1. 確認測試服務是 PoC，且不會連到正式 9090。
2. RX unity、TX -6 固定。
3. classifier／threshold 凍結。
4. bridge 使用：`好，我想先了解一下，你們主要是在做什麼？`
5. bridge 在 AI 靜音時播放，之後等待至少 1.5 秒。
6. 只做 A、B 事件，不使用 C。
7. 每一事件保留足夠的前後錄音與明確 marks；B 事件後至少保留 5 秒。
8. 完成 closing 後掛斷。
9. 不做 -3 dB、不改 HT813、不動正式設定。
10. 如果要真人實測，先把測試話術、預期事件、錄音標記方式與 PoC 端口交代清楚；不要要求使用者重做已經有乾淨紀錄的事件。

### 6.7 指標與驗收報告

至少輸出：

- Caller event recall：normal、short 分開計算。
- Echo-only false positive：事件數與每 window 數都要報。
- Silence false positive。
- Double-talk false negative。
- Caller availability。
- Coverage。
- CER。
- First usable decision latency：median、P90、每 event。
- Caller onset 到 first shadow decision 的延遲。
- Echo-only 與 caller-only 的 RMS／peak 分布。
- Clipping rate。
- Event-level confusion matrix。
- 每個誤判事件的時間軸、RX window、ASR text、TTS text、decision reason。
- 不同通話之間的差異，不要只提供 pooled aggregate。

**安全驗收底線：**

1. Echo-only 與 silence 情境不得有 shadow interrupt decision；有任何一次就必須逐筆分析原因，不能用整體平均稀釋。
2. 不得有 stale window／bridge／closing carryover 造成的觸發。
3. 每個 normal／short caller event 都要能追溯是否被辨識，不能只提供總 accuracy。
4. 遲延必須有實際分布，不得只說「即時」。
5. 如果樣本不足、ground truth 不完整或事件數太少，結論必須是「尚不足以驗收」，不能硬判 PASS。

Sensitivity 的最低目標與 latency 目標，先根據現有有效樣本數、產品可接受延遲及誤判成本提出建議值，再清楚列出依據；不要在看完結果後才回頭調門檻讓結果變漂亮。

### 6.8 Phase 5：是否進入下一個階段

只有 v11.4 Shadow Acceptance 有清楚的 event-level evidence，才能提出下一階段方案。

若 v11.4 通過：

- 提出一個獨立的 v11.5 PoC interruption action test。
- 只在測試埠／隔離測試通話驗證 stop TTS。
- 仍不開正式 barge-in。
- 必須驗證 stop latency、重複 stop、音訊 buffer flush、狀態回復、caller 後續語音辨識及掛斷清理。
- 正式上線需另行提出風險與授權。

若 v11.4 失敗：

先根據錯誤分類決定原因：

- Caller ASR availability／coverage 不足。
- Streaming／sliding-window latency 不足。
- Echo transcript 與 TTS text 重合。
- Semantic classifier decision logic 不足。
- VAD／音訊 frame／time alignment 問題。
- TTS lifecycle／state machine 問題。
- Analog hybrid echo 仍然限制 caller／echo 可分性。

不要一失敗就同時換 ASR、改 gain、調 threshold、加 AEC，否則無法歸因。

---

## 7. 歷史上不值得優先重做的路線

除非 v11.4 的新資料明確證明需要，否則暫時不要投入以下方向：

1. 只靠 RMS／能量／音訊特徵做 caller-vs-echo 分離：v6 AUC 約 0.5，已經顯示分離能力不足。
2. 舊 RX8 錄音事後 normalize：clipping 資訊已永久遺失。
3. 只用離線 gain transformation 推估真實電話表現：歷史模擬曾明顯高估實際 ASR。
4. Linear AEC 512-tap LS／NLMS：歷史最大 suppression 約 1.6 dB／0.7 dB，delay 約 410–423 ms，coherence max 約 0.44，沒有足夠效果。
5. AECM／先前 neural AEC 嘗試：歷史上未達需求。
6. 未先驗證新 gain staging 就重調舊 semantic thresholds。
7. 直接改走 Digital SIP 或新增大量 DSP 系統，卻沒有先判斷目前主要瓶頸。

如果之後真的需要重新開啟 LEC／NEC／impedance、AEC3／DTLN 或 Digital SIP 方案，必須先提供新的證據、成本／風險、預期改善與隔離測試方法；不可直接動 HT813 或正式服務。

---

## 8. 新 IDE 的執行方式與交付要求

### 8.1 執行方式

請以資深 VoIP／即時音訊／Python 工程師的方式工作：

- 先盤點、再實驗、最後才修改。
- 所有推論要附上檔案路徑、event ID、metrics 或原始 log。
- 任何指標都要說明計算定義。
- 小樣本要明確標示，不可誇大。
- 對其他 IDE 的未提交變更採取保護措施。
- 優先採取可回滾、最小差異、PoC-only 的修改。
- 不可默默改 threshold 或把 debug mode 變成正式行為。
- 如果現有程式已經能 shadow replay，不要為了重構而大改架構。
- 若有修改，必須提供 diff、測試命令、測試結果及 rollback 步驟。
- 不要在沒有使用者要求時 push、deploy 或更新正式環境。

### 8.2 必須建立的交付物

在專案既有文件慣例下建立或更新以下內容；先檢查是否已有 canonical 文件，避免重複建立一堆 MD：

1. **接手盤點報告**
   - 實際 branch／commit／working tree。
   - 正式與 PoC 端口、process、設定。
   - source-of-truth 路徑。
   - 文件摘要與實際 source 的差異。
2. **v11.4 Shadow Acceptance 計畫**
   - 測試資料集。
   - 測試矩陣。
   - 凍結的參數。
   - 驗收條件。
3. **Event-level 結果**
   - CSV 或專案現有格式。
   - 每一事件的 ground truth、ASR、shadow decision、latency、FP/FN。
4. **Metrics JSON**
   - 可重現的指標。
   - 清楚列出 sample count 與計算方式。
5. **正式安全檢查／Restore report**
   - 正式 9090 未變更。
   - `[saas-dialogue]` 未變更。
   - 正式 barge-in 仍 OFF。
   - 測試 port／process 已正確收尾。
6. **結論報告**
   - PASS／FAIL／INSUFFICIENT DATA。
   - 哪些證據支持結論。
   - 未解風險。
   - 下一個最小實驗建議。
   - 是否需要使用者進行實際電話測試。

### 8.3 最終回報格式

完成後請用以下格式回報：

```text
任務：EchoGuard v11.4 — Full-Duplex Shadow Acceptance

1. 接手盤點
- 實際專案路徑：
- branch / commit：
- working tree：
- 正式 9090 狀態：
- PoC ports：
- 正式 barge-in：
- v11.3 原始產物完整性：

2. 已完成的分析與實驗
- 歷史資料重建：
- Shadow replay：
- 使用的通話數：
- 使用的事件數：
- Normal / Short / Echo-only / Silence / Double-talk：

3. 核心結果
- Caller recall：
- Echo-only FP：
- Silence FP：
- Double-talk FN：
- Caller availability：
- Coverage / CER：
- First usable latency median / P90：
- Clipping：
- Event-level 問題：

4. 結論
- PASS / FAIL / INSUFFICIENT DATA：
- 已證實：
- 尚未證實：
- 最大風險：

5. 修改紀錄
- 修改檔案：
- 修改原因：
- Diff：
- 測試命令與結果：
- Rollback：

6. 正式環境保護
- 9090 是否保持未修改：
- [saas-dialogue] 是否保持未修改：
- 正式 barge-in 是否保持 OFF：
- HT813 是否保持未修改：
- 是否有部署／push（預期：沒有）：

7. 下一步
- 最小下一個實驗：
- 是否需要使用者打電話：
- 需要使用者做什麼（只有確實需要時才列）：
```

---

## 9. 最後提醒：目前正確的技術判斷

目前最值得做的不是繼續盲目調大／調小音量，也不是馬上換 AEC 或改硬體，而是：

1. 保留 RX unity + TX -6 作為目前 Shadow 候選。
2. 把 v11.3 原始資料與舊資料重新整理成可追溯的 event-level dataset。
3. 在不停止 TTS 的 shadow mode 下，驗證 semantic EchoGuard 是否真的能分離 caller 和 echo。
4. 先取得足夠的 false-positive、false-negative 和 latency 證據。
5. 通過後才進入隔離 PoC 的實際 stop-TTS 測試。
6. 正式 9090、正式 dialplan、正式 barge-in、HT813 設定在整個 v11.4 階段都維持不動。

**一句話交接：v11.3 已經讓真人聲音在 AI 播放期間重新可觀察、可辨識；v11.4 的責任是證明 EchoGuard 能安全判斷「何時真的該停」，而不是直接把正式全雙工打開。**
