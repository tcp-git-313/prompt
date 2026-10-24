# EchoGuard v10 — Live Interruption Data Collection & Acceptance Dataset

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。請完整讀取本文件後再開始，不要自行改題。

## 一、目前狀態

v9 判定：

`CALLER_DATASET_INSUFFICIENT`

目前已確認：

- Audio-only EchoGuard 已正式判定 `AUDIO_ONLY_NOT_SEPARABLE`
- Semantic echo recoverability 成立
- sliding-window interim pipeline 已可運作
- Echo safety：0 confirmed CALLER FP / 420 echo windows，EARLY = 0
- TEA-ASR request latency 已量到約 P50 138ms / P90 166ms
- v8 的 3 個 `double_talk` FN 已證明是 fixture artifact，不是演算法漏判
- `b3bf3c3a` 原始 full call 實際有約 16s post-onset audio，但 caller 本身主要只有 back-channel，沒有足夠 Normal Caller speech
- genuine caller fragments 很短，例如 `喂 / 嗯 / 現在嗯`

所以目前唯一阻塞不是 threshold、ASR latency 或 runtime，而是：

> **缺少足夠的真實、完整、可標註的 intentional caller interruption 語料。**

本輪不是演算法優化，而是 **Live Data Collection + Ground Truth + Dataset Acceptance**。

---

# 二、本輪唯一目標

收集最小但有效的真電話 interruption corpus，讓下一輪可以第一次真正回答：

1. Semantic Guard 對 Normal Caller 的 recall 是多少？
2. caller onset → first usable semantic decision 真實延遲是多少？
3. Short Caller 是否能辨識？
4. 在真實 AI playback + echo + caller double-talk 下，Echo FP 是否仍為 0？

本輪只收資料、標 GT、建立可重播 dataset。

**不要改演算法。**

---

# 三、硬性安全規則

禁止：

- 修改正式 9090 runtime
- deploy production
- 打開正式 barge-in
- 真正 stop TTS
- 改 ASR 模型
- 把 SenseVoice 換成 TEA production
- 修改 v9 semantic threshold 來配合新資料
- 修改 EchoGuard decision rule
- 重新做 Audio-only DSP sweep
- force push
- `git reset --hard`

Live call 必須：

- barge-in OFF
- shadow only
- AI playback 繼續播放
- 只記錄「如果現在允許 barge-in，shadow judge 會怎麼判」

如果任何步驟可能影響正式 9090 或 production dialplan：立即停止。

---

# 四、開始前盤點

先執行：

```bash
git status
git branch
git log --oneline -20
```

確認實際 repo / runtime 狀態，尤其：

- `F:\00-Ticenpi-SaaS\Ticenpi-Voip` 是否仍是目前專案位置
- 正式 9090 是否未動
- shadow / PoC 測試入口
- RX/TX recording path
- TEA-ASR endpoint `:19110`
- current semantic interim harness
- v9 `run_v9.py`
- v9 replay cache
- current call/session UID 生成方式
- playback node / TTS transcript logging
- current shadow log schema

以 repo 現況為準，不要假設。

---

# 五、最小 Live Collection 規模

目標：

- **3～5 通真電話**
- 每通至少 **2 次 intentional interruption**
- 至少總共 **8～10 個有效 interruption events**

其中至少包含：

## Normal Caller

至少 5～6 個事件。

每次 caller speech 約：

`0.5 ～ 1.5 秒`

例句可以自然說：

- 等一下，我現在不方便
- 你哪裡打來的？
- 不用了，謝謝
- 你剛剛說什麼？
- 我現在在忙，晚點再說
- 這個是什麼服務？

不要每通都講完全相同句子。

## Short Caller

至少 3～4 個事件。

約：

`80 ～ 300ms`

例如：

- 喂
- 嗯
- 欸
- 好
- 不用
- 等一下

Short Caller 必須獨立 lane，不得混入 Normal Caller acceptance denominator。

---

# 六、Interruption 時機要有變化

不要每次都在同一個 playback 時點插話。

至少覆蓋：

- AI sentence 前段
- AI sentence 中段
- AI sentence 後段
- 不同 TTS node / phrase

每通最好至少：

1. 一次 Normal Caller
2. 一次 Short Caller 或另一個 Normal Caller

如果可以，第 3 次可做不同位置。

---

# 七、每次 interruption 的最低錄音條件

對每個 intentional interruption event：

- onset 前至少保留 2～3s context
- onset 後至少保留 2s audio
- 最好保留到 caller speech 結束後 1s
- 必須保留 AI playback / echo context
- 不要只切 caller 乾淨語音

Dataset fixture 建議：

```text
[pre-onset AI playback / echo context 2～3s]
→ caller onset
→ [caller / double-talk 0.5～1.5s]
→ [post-onset context ≥2s]
```

---

# 八、每通電話必須保存 Full Call

本輪不要再只依賴 PLAYING slice。

每通真電話必須保存完整：

- RX full-call WAV
- TX / actual-sent-PCM full-call WAV（若目前可得）
- call metadata
- playback node timeline
- known TTS text timeline
- ASR interim / final logs
- semantic shadow logs

任何 fixture 都必須可以追溯回 full call source。

---

# 九、Ground Truth 標註

每個 intentional interruption event 必須記：

- `call_uid`
- `event_id`
- `caller_type = normal | short`
- exact or approximate caller phrase
- `caller_onset_ms`
- `caller_offset_ms`
- caller duration
- playback node id
- active TTS text
- playback position
- whether AI still playing
- expected result = CALLER
- GT confidence

Ground truth onset 必須相對 **call start**。

不要再使用容易混淆的「相對第一段 playback start」作唯一時間基準。

如果內部仍需要相對 playback time，可另外保存：

- `caller_onset_call_ms`
- `caller_onset_playback_ms`

但 acceptance 一律以 call timeline 為主。

---

# 十、必須保存 Playback Node Reference

延續 v9 的 per-node reference。

每個 playback node 至少保存：

- node id
- text
- playback start ms
- playback end ms
- audio duration
- phrase/session id

Semantic comparison 只和當下合理的 active / recent TTS node 做比對。

禁止回退成一條 global TTS string。

---

# 十一、Live Call 執行原則

本輪需要真人配合打電話 / 接電話。

IDE/Agent 可以：

- 啟動 shadow test service
- 啟動 recording
- 準備 live test script
- 自動記錄 logs
- 收集檔案
- 產生 GT template

但不得替真人：

- 撥真電話（若需人工操作）
- 輸入帳密 / 2FA
- 操作實體電話設備
- 假造真人 interruption

如果需要真人實體操作，清楚列出最少步驟後停在可執行狀態。

---

# 十二、Live Test Script

為避免資料品質不一致，建立簡單測試腳本 / checklist。

每通電話建議：

## Event A — Normal Caller

AI 開始講固定台詞後 0.5～2s 內，真人插話約 0.5～1.5s。

例：

`等一下，我現在不方便。`

## Event B — Short Caller

下一個 AI turn 中插：

`喂` / `嗯` / `不用`

約 80～300ms。

## Optional Event C

另一個不同 playback node，再做一次 Normal Caller。

每個事件中：

- AI 照常播放
- 不真正 cut TTS
- shadow judge 照常記錄

---

# 十三、Live Collection Quality Gate

每個 event 收完後立即檢查：

- RX 是否有真 caller speech
- caller onset 是否可定位
- onset 後 audio 是否 ≥2s
- playback node 是否有記錄
- TTS reference 是否存在
- ASR / semantic log 是否存在
- sample rate / duration 是否正確
- file 是否非空 / 非截斷

不合格 event 立即標記：

`COLLECTION_INVALID`

不要等全部做完才發現錄音壞掉。

---

# 十四、建立 v10 Dataset

建議輸出：

```text
results/full_duplex_v10/
  full_recordings/
  fixtures/
  metadata/
  gt/
  interim/
  failures/
  ACCEPTANCE_REPORT_V10.md
  live_collection_manifest.json
  interruption_ground_truth.json
  caller_fixture_audit.json
  caller_timeline.csv
  echo_safety.json
  latency_breakdown.json
  pareto.json
```

實際名稱可依 repo 現況調整，但結構必須清楚。

---

# 十五、Original Real 與 Derived 必須分開

新收集的真電話：

`ORIGINAL_REAL`

任何：

- gain augmentation
- noise injection
- delayed mixing
- synthetic double-talk

只能標：

`REAL_DERIVED`

Acceptance 的主結果只能用 ORIGINAL_REAL。

---

# 十六、收集完後才跑 Existing v9 Harness

資料收集完成後：

不要改 semantic logic。

使用現有 v9 harness / replay runner 對新 ORIGINAL_REAL 跑：

- sliding-window interim
- per-node TTS reference
- current persistence rule
- current thresholds

目的：

驗證目前方法本身，而不是在新 dataset 上現場調參。

---

# 十七、Normal Caller Acceptance

只計 eligible Normal Caller events。

至少輸出：

- caller recall
- first usable semantic decision
- first stable semantic decision
- transcript evolution
- first non-empty transcript
- first novel transcript
- semantic state evolution

目標：

- Recall ≥ 90%
- Median first usable ≤ 500ms
- P90 ≤ 800ms

如果 sample 太少，不要假裝統計穩定；報 raw per-event data。

---

# 十八、Short Caller Acceptance

Short Caller 獨立報：

- transcript availability
- recall
- first usable latency
- fail reason

不得因 Short Caller 失敗直接否決 Normal Caller 路線。

但要清楚回答：

- 80～150ms 能不能辨
- 150～300ms 能不能辨
- 哪些 short utterance ASR 根本沒有文字輸出

---

# 十九、Echo Safety 不能退步

新 live calls 也要計：

- echo-only windows
- confirmed CALLER FP
- EARLY

要求：

- confirmed Echo FP = 0
- EARLY = 0

如果 Normal Caller recall 提升但 echo FP > 0：

不得判 PASS。

---

# 二十、Latency Breakdown

對每個 Normal Caller event 拆：

```text
caller onset
→ poll wait
→ audio evidence accumulation
→ queue/coalescing
→ TEA ASR processing
→ semantic compute
→ first usable decision
```

至少報：

- `poll_wait_ms`
- `post_onset_audio_needed_ms`
- `asr_request_ms`
- `queue_wait_ms`
- `semantic_compute_ms`
- `first_usable_semantic_decision_ms`
- `first_stable_semantic_decision_ms`

Aggregate：

| Component | Median | P90 | % Total |

---

# 二十一、ASR / Runtime Resource

沿用 v9 設定，確認新 live data 下：

- ASR P50/P90/P95
- no unbounded queue
- latest-window-wins 正常
- stale response 不覆蓋最新 state
- CPU/GPU/VRAM 不異常

這輪不是 performance optimization，只驗證 runtime 仍穩定。

---

# 二十二、判定規則

最後只能選以下之一：

## PASS

至少 3 通真電話、至少 5 個 eligible Normal Caller events，且：

- Echo FP = 0
- EARLY = 0
- Normal Caller Recall ≥ 90%
- Median first usable ≤ 500ms
- P90 ≤ 800ms

→ 可進下一階段 shadow runtime integration / live no-cut acceptance。

## CONDITIONAL_PASS

方向成立，但：

- event 數仍偏少
- latency 部分超標
- Short Caller 表現不足

但 Normal Caller 已顯示明確可分。

## SEMANTIC_SEPARABLE_BUT_TOO_SLOW

Normal Caller 可分，但：

- P90 > 1000ms

或 runtime scheduling 造成不可接受延遲。

## SEMANTIC_NOT_SEPARABLE

有足夠 Normal Caller data，但仍無法穩定和 echo 區分。

## LIVE_COLLECTION_INSUFFICIENT

真人資料仍不足、錄音壞掉、GT 不可信，無法下結論。

---

# 二十三、Final Report 必須回答

`ACCEPTANCE_REPORT_V10.md` 至少回答：

1. 實際收集幾通真電話？
2. Normal Caller events 幾個？
3. Short Caller events 幾個？
4. 每個 caller event 的實際 phrase / duration / onset？
5. Echo FP / EARLY 是多少？
6. Normal Caller Recall 是多少？
7. Short Caller Recall 是多少？
8. first usable latency per event？
9. Median / P90？
10. 哪個 latency component 是主瓶頸？
11. TEA ASR P50/P90/P95？
12. 是否有 queue/backlog/stale response？
13. 是否已足以進下一階段 shadow runtime integration？
14. 如果不能，唯一下一步是什麼？

---

# 二十四、最重要停止規則

這一輪不要因為收集到新資料就順手修改演算法。

如果結果不好：

先如實報告。

不要在同一輪：

- 調 threshold
- 加 whitelist
- 改 persistence
- 換 ASR
- 改 window rule

資料收集與演算法優化必須分開，避免 acceptance leakage。

---

# 二十五、執行方式

先：

盤點 → 建立 live collection harness / checklist → 準備 shadow recording → 等真人完成 3～5 通電話 → 每通即時 quality check → 建 GT → 建 fixtures → 用既有 v9 logic replay → 產 v10 report。

如果需要真人撥電話或操作 HT813/實體電話，請明確顯示「現在需要真人做哪些步驟」，不要自行假裝已完成。

全程保持：

- production 9090 untouched
- barge-in OFF
- shadow only
- no deploy
- no push production

本輪唯一目標：

> **取得足夠、可重播、可驗收的真實 caller interruption dataset，並在不改演算法的前提下第一次真正量出 Normal Caller semantic recall 與 latency。**

開始執行。