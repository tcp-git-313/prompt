# VOIP VAD／Barge-in 真人測試 — Phase 1 完成

## 角色

VOIP VAD／BARGE-IN 診斷負責人

## 目前已知狀態

上一輪稽核與部署報告記載：

- Production 使用 Asterisk + AudioSocket，AudioSocket 埠為 9090。
- Runtime 檔案為 `/usr/local/lib/saas/saas_streaming_server.py`。
- Shadow diagnostic instrumentation 已部署。
- Silero VAD 已載入，9090 應在 listening。
- `SAAS_BARGEIN=0` 必須維持不變。
- 若 `SAAS_VAD_THRESH` 未設定，有效 threshold 應為 0.5。
- Asterisk 不得因本任務重啟。
- Production dialplan 應包含 AudioSocket 與 MixMonitor。
- 歷史資料無法回答本次真人 VAD 問題，必須取得新的真人測試資料。

開始執行前，先以即時證據重新核對上述狀態；不要把歷史報告當成目前 runtime 證據。不要重做完整架構稽核。

## 目標

完成真人 A／B／C 診斷，判定在 AI 音訊播放期間，caller speech 是否能被 AudioSocket／Silero VAD 看見。

本階段只做診斷。不得啟用 barge-in，也不得調整 production 行為。

## 允許的最小修正

如果目前 shadow logger 只在 `PLAYING` 狀態輸出，而 Test B 需要 `LISTENING` baseline，只能把診斷條件從：

`state == PLAYING`

改為等價的：

`state == PLAYING OR state == LISTENING`

只保留既有 shadow metrics：

- monotonic timestamp
- call／ASID
- state
- RMS dBFS
- peak dBFS
- Silero VAD probability
- speech true／false
- 若原本已有，保留 consecutive speech ms

取樣頻率維持約 100–200 ms。本 logger 只能提供 observability，不得改變既有 VAD decision path。

若已經同時記錄 `PLAYING` 與 `LISTENING`，不要再次修改程式。

## 硬性邊界

不得變更：

- `SAAS_BARGEIN=0`
- VAD threshold
- VAD model
- endpoint timing
- HT813 settings
- Asterisk gain
- STT
- TTS
- dialogue／walker logic
- Groq
- GPU／CUDA configuration
- production dialogue behavior

不得重啟 Asterisk。

如果最小 logger 修正確實需要只重啟 `saas-streaming`，則必須先通過 syntax validation，且記錄修改前後 PID，並驗證 9090 與 service health。除此之外不得重啟服務。

## 執行順序

先準備 logger 與 journal capture。準備完成後，必須等待使用者明確說「可以開始」或同等意思，才可以 originate／撥出真人測試電話。使用者尚未明確允許前，不得撥號。

盡量使用同一通真人電話完成 A、B、C。

### Test A — 只有 AI 播放

在 AI playback segment 期間，使用者完全保持安靜。

擷取 `PLAYING` interval 並計算：

- RMS median／max
- peak median／max
- VAD probability median／max
- `speech=true` 事件數

這是 playback leakage baseline。

### Test B — 只有 caller 說話

當系統處於 `LISTENING` 且 AI 沒有播放時，使用者說：

`你好，我現在有在講話`

擷取 `LISTENING` interval，計算與 Test A 相同的 metrics。

這是正常 caller-speech baseline。

### Test C — Barge-in shadow

在 AI playback 期間，使用者刻意壓過 AI 說兩次：

1. `等一下`
2. `我有問題`

`SAAS_BARGEIN` 必須維持 0，因此 AI 繼續說話是預期結果。

每次 interruption 記錄：

- caller speech approximate start
- RMS rise
- peak
- VAD probability rise／max
- 是否出現 `speech=true`
- 若可量測，caller speech onset 到 VAD rise 的 delay

MixMonitor 只能作為 timing／reference recording，不得把 MixMonitor 當成 VAD input path。

## 分析規則

只能依據實測證據分類：

- **A — VAD 正確偵測 caller；只是 barge-in 被關閉**
- **B — playback leakage 本身造成高音量／false VAD**
- **C — 播放期間 caller audio 在 AudioSocket path 被衰減**
- **D — audio level 上升，但 Silero classification 沒有上升**
- **E — 播放期間 AudioSocket 沒收到有意義的 caller audio**
- **F — 證據不足，無法判定**

只有在 measured data 支持時，才允許同時提出多個 findings。除非 live measurements 支持，不得假設 HT813 half-duplex 是原因。

## 最終回報格式

只回報以下內容，不要實作修正：

### CURRENT

- `SAAS_BARGEIN`
- effective VAD threshold
- runtime SHA
- service／PID
- `PLAYING` 與 `LISTENING` shadow logging 是否正常運作

### TEST A — AI ONLY

- RMS median／max
- peak median／max
- VAD median／max
- `speech=true` count

### TEST B — CALLER ONLY

- 相同 metrics

### TEST C — BARGE-IN

每個 phrase 都列出：

- phrase
- RMS／peak
- VAD max
- speech detected YES／NO
- approximate detection delay

### FINDING

- classification A／B／C／D／E／F
- concise evidence

### BLOCKER

- 若已證實，列出單一最重要 blocker

### NEXT STEP

- 只提出一個最小且安全的下一個實驗

### RUNTIME CHANGES

- files changed
- restart details
- `SAAS_BARGEIN remains 0 = YES／NO`
- Asterisk restart = NO

回報完成後停止。不要實作 fix。
