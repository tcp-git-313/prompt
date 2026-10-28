# EchoGuard v10.1 — Protocol-Corrected Live Acceptance

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。
>
> 這一輪不是調參、不是換 ASR、不是重寫 EchoGuard。目標只有兩個：
>
> 1. 修正 v10 真人測試協議的結構性缺陷。
> 2. 在 frozen v9 semantic logic 下重新取得乾淨、可解讀的真人 double-talk acceptance 結果。

---

## 一、背景與目前結論

v10 真人測試已完成 5 通，得到：

- Normal Caller 7 個有效事件
- Short Caller 3 個有效事件
- Echo FP = 2 / 179 echo windows
- EARLY = 0
- Normal Caller recall = 14%（1/7）
- 唯一 TP first usable latency = 5641ms
- Short Caller recall = 0/3
- ASR P50 141ms / P90 196ms / P95 223ms
- 主要 latency bottleneck = audio evidence，而非 ASR
- cleanup PASS
- production barge-in 仍 OFF

v10 原始 verdict：

`SEMANTIC_NOT_SEPARABLE`

但真人操作回報顯示 v10 protocol 有結構性問題：

1. A 插話後到 B 插話前，沒有固定 bridge utterance。
2. 操作者只能自由接話，造成每通上下文不一致。
3. 上一輪正常 caller speech 可能殘留進下一個 2000ms semantic window。
4. 最後事件太靠近掛斷，可能沒有足夠 post-onset audio。
5. C 類事件在實務上常來不及完成。

因此：

> **v10 視為 FAIL evidence，但 `SEMANTIC_NOT_SEPARABLE` 暫不視為最終技術結論。**

本輪要做的是：

`v10.1 Protocol-Corrected Acceptance`

---

## 二、Frozen 規則

本輪嚴格禁止：

- 改 semantic threshold
- 改 novel threshold 0.5
- 改 persistence = 2
- 改 500ms / 2000ms window
- 改 polling interval
- 換 ASR
- 改 per-node reference logic
- 加 phrase whitelist
- 加 case-specific workaround
- 改 EchoGuard acoustic logic
- 開正式 barge-in
- deploy production
- 修改 production application code
- force push
- `git reset --hard`

本輪只允許：

- 修正真人測試 protocol
- 修正 Operator Helper 的測試腳本 / UX
- 增加 bridge 標記
- 增加 turn-boundary correctness audit
- 增加 post-onset 保留規則
- 重新收真人資料
- 用 frozen logic 重播

---

## 三、第一優先：先查 Turn-Boundary Carry-Over

在重新打電話前，先用 v10 現有 recordings + logs 做一次只讀 audit。

核心問題：

> 新 AI playback node 開始時，semantic 2000ms rolling window 是否可能包含上一輪 caller 正常回答？

請對 v10 五通至少各抽一個 node transition，重建時間線：

```text
previous caller speech end
→ previous turn end
→ next AI playback start
→ first semantic poll
→ semantic window start/end
```

對每個 semantic window 記：

- call_uid
- node_id
- playback_start_ms
- poll_time_ms
- window_start_ms
- window_end_ms
- previous_caller_end_ms
- overlap_ms_with_previous_caller
- whether previous caller text appears in interim ASR
- final decision

輸出：

`results/full_duplex_v10_1/turn_boundary_audit.csv`

以及：

`results/full_duplex_v10_1/TURN_BOUNDARY_AUDIT.md`

### 判定

如果下一個 playback node 的 semantic window 會吃到上一輪 caller speech：

標記：

`TURN_BOUNDARY_CARRYOVER_CONFIRMED`

但**不要因此直接修改 semantic algorithm**。

本輪 acceptance 要透過 corrected protocol 盡量避免 bridge 殘留污染，並在 report 中分開標示 carry-over risk。

如果沒有 carry-over：

標記：

`TURN_BOUNDARY_CARRYOVER_NOT_OBSERVED`

---

## 四、更新 Operator Protocol，不重寫 Backend

沿用：

- `v10_operator.py`
- `v10_collect.py`
- frozen `run_v9.py`

只調整真人測試腳本與 collection protocol。

建議版本標記：

`V10_1`

case name：

```text
V10R-T01
V10R-T02
V10R-T03
V10R-T04
V10R-T05
```

不要覆蓋 v10 原始 evidence。

新資料輸出：

```text
results/full_duplex_v10_1/
```

---

## 五、每通只保留 2 個正式 Interruption

取消 C。

每通固定：

- A = Normal Caller
- Bridge = 非測試、非 interruption、不得進 recall denominator
- B = Normal 或 Short Caller
- B 後保留至少 5 秒
- 最後再正常結束

任何 Bridge 都標：

`BRIDGE_NON_OVERLAP`

不得算 TP / FN / FP。

---

## 六、每通固定結構

所有 T01～T05 都遵守：

```text
接通
↓
真人：「喂，你好」
↓
AI 開場
↓
A 正式 interruption
↓
AI 繼續播完
↓
等待 AI 停止後至少 1.5 秒
↓
Bridge 正常回答（非 interruption）
↓
等待下一個 AI turn
↓
B 正式 interruption
↓
AI 繼續播
↓
B onset 後至少保留 5 秒完整錄音
↓
真人：「好，謝謝，我先了解一下。」
↓
再等待約 1 秒
↓
掛斷
```

### 非常重要

Bridge 必須在 AI 沒有播放時說。

如果 Bridge 和 AI 重疊：

標記：

`BRIDGE_CONTAMINATED`

該下一事件必須在 report 中標示可能受污染。

---

## 七、固定 Bridge

每通 Bridge 都使用同一句，避免自由發揮：

`好，我想先了解一下，你們主要是在做什麼？`

這句不是 interruption。

不進：

- Normal denominator
- Short denominator
- echo FP denominator

但要記錄開始 / 結束時間，供 turn-boundary audit。

---

## 八、五通真人腳本

### T01

A｜Normal

AI 第一句開始約 1 秒時插話：

`等一下，我現在不方便。`

Bridge：

`好，我想先了解一下，你們主要是在做什麼？`

B｜Short

下一個 AI turn 開始約 0.8～1.2 秒時插話：

`嗯`

B 後保留至少 5 秒，不要立刻掛斷。

---

### T02

A｜Normal

AI 句子前段插話：

`你哪裡打來的？`

Bridge：

`好，我想先了解一下，你們主要是在做什麼？`

B｜Normal

下一個 AI turn 插話：

`你剛剛說什麼？`

B 後保留至少 5 秒。

---

### T03

A｜Normal

AI 句中插話：

`不用了，謝謝。`

Bridge：

`好，我想先了解一下，你們主要是在做什麼？`

B｜Short

下一個 AI turn 插話：

`不用`

B 後保留至少 5 秒。

---

### T04

A｜Normal

AI 句中偏後插話：

`我現在在忙，晚點再說。`

Bridge：

`好，我想先了解一下，你們主要是在做什麼？`

B｜Normal

下一個 AI turn 插話：

`這個是什麼服務？`

B 後保留至少 5 秒。

---

### T05

A｜Normal

AI 第一句約 1 秒插話：

`等一下，你先不要講。`

Bridge：

`好，我想先了解一下，你們主要是在做什麼？`

B｜Normal

下一個 AI turn 插話：

`我沒有要賣房子。`

B 後保留至少 5 秒。

---

## 九、真人操作 UX

Operator 每通只顯示這種簡單卡片：

```text
CASE V10R-T01 READY

1. 接起先說：「喂，你好」

2. AI 第一句開始約 1 秒：
   A「等一下，我現在不方便。」

3. 等 AI 完整講完，而且停止至少 1.5 秒後：
   Bridge「好，我想先了解一下，你們主要是在做什麼？」

4. 等下一個 AI turn 開始約 1 秒：
   B「嗯」

5. B 說完後不要掛斷。
   至少再保持通話 5 秒。

6. 最後說：
   「好，謝謝，我先了解一下。」

7. 再等約 1 秒後掛斷。
```

不要再使用 A/B/C 模糊節奏。

---

## 十、每個事件 GT 必須增加的欄位

除了既有欄位，再加入：

- `protocol_version = v10.1`
- `event_role = interruption | bridge | closing`
- `bridge_contaminated = true|false`
- `previous_caller_end_ms`
- `current_playback_start_ms`
- `carryover_overlap_ms`
- `post_onset_available_ms`

A / B interruption eligibility：

- onset/offset 可定位
- caller 確實和 AI playback 重疊
- post-onset audio ≥ 3000ms
- 建議 ≥ 5000ms
- source full-call 可追溯

Bridge：

- 不進 recall denominator
- 不進 echo FP denominator

---

## 十一、Echo FP 要重新分類

v10 的 2 個 echo FP 不可直接丟掉。

重新分析並分類：

```text
PURE_ECHO_FP
PREVIOUS_CALLER_CARRYOVER
BRIDGE_CONTAMINATION
UNKNOWN
```

v10.1 新資料也用同樣分類。

真正 Echo Safety 指標要額外報：

- raw CALLER FP
- confirmed PURE_ECHO_FP
- carry-over related FP
- bridge-related FP
- EARLY

Acceptance 主 Echo FP 要使用：

`confirmed PURE_ECHO_FP`

但 raw FP 也必須完整報告，不可隱藏。

---

## 十二、Short Caller 仍獨立 Lane

不要改 frozen gate。

Short Caller 仍可能因 novel chars 不足而 0 recall。

本輪目的不是救 Short Caller。

Short lane 單獨報：

- transcript availability
- novel chars
- recall
- first usable latency
- fail reason

如果 frozen rule 本質上要求 ≥4 novel chars，清楚寫出：

`SHORT_CALLER_INCOMPATIBLE_WITH_CURRENT_FROZEN_SEMANTIC_GATE`

但不要因此修改 gate。

---

## 十三、Acceptance 主體看 Normal Caller

至少應有：

- T01-A
- T02-A
- T02-B
- T03-A
- T04-A
- T04-B
- T05-A
- T05-B

預期 8 個左右 Normal Caller events。

主指標：

- TP
- FN
- recall
- first non-empty transcript
- first novel transcript
- first usable semantic decision
- first stable semantic decision
- median
- P90

目標不變：

- Recall ≥ 90%
- Median ≤ 500ms
- P90 ≤ 800ms

---

## 十四、Latency Breakdown

每個 Normal event 拆：

```text
caller onset
→ next poll
→ audio evidence accumulation
→ queue/coalescing
→ ASR
→ semantic compute
→ first usable decision
```

至少輸出：

- poll_wait_ms
- post_onset_audio_needed_ms
- queue_wait_ms
- asr_ms
- semantic_compute_ms
- total_ms

如果仍然是 audio evidence 佔 >80%，明確指出。

---

## 十五、Protocol Correctness Gate

在算法 verdict 前，先給 protocol verdict。

### PROTOCOL_PASS

條件：

- ≥5 通完成
- 每通 A/B 時序清楚
- Bridge 都能識別
- ≥90% interruption events post-onset ≥3s
- 不再有 C 來不及問題
- 沒有自由發揮 bridge

### PROTOCOL_FAIL

如果仍出現：

- 大量 bridge overlap
- 最後事件太短
- event timing 不可定位
- evidence 缺失

則本輪停止算法結論。

---

## 十六、最終 Verdict 規則

### PASS

Protocol PASS，且：

- confirmed PURE_ECHO_FP = 0
- EARLY = 0
- Normal Recall ≥ 90%
- Median ≤ 500ms
- P90 ≤ 800ms

### CONDITIONAL_PASS

Protocol PASS，Normal clearly separable，但 sample / latency 尚有不足。

### SEMANTIC_SEPARABLE_BUT_TOO_SLOW

Protocol PASS，caller/echo 可分，但 P90 > 1000ms 或即時性不可接受。

### SEMANTIC_NOT_SEPARABLE

只有在 **Protocol PASS** 後才能給。

條件：

- Normal Caller 語料充足
- Bridge contamination 已排除
- turn-boundary carry-over 已分析
- post-onset audio 足夠
- 仍然 recall 很低，或 confirmed PURE_ECHO_FP > 0，且無法可靠分離

### PROTOCOL_INVALID

真人資料流程本身仍有污染，不允許下算法結論。

---

## 十七、最終輸出

建立：

```text
results/full_duplex_v10_1/
  ACCEPTANCE_REPORT_V10_1.md
  PROTOCOL_REPORT.md
  TURN_BOUNDARY_AUDIT.md
  turn_boundary_audit.csv
  interruption_ground_truth.json
  bridge_timeline.csv
  caller_timeline.csv
  echo_safety.json
  latency_breakdown.json
  protocol_quality.json
  freeze_manifest.json
  failures/
```

---

## 十八、Cleanup

真人測試結束後一定要執行既有 cleanup。

確認：

- 9090 PASS
- service active
- env restore
- barge-in OFF
- dataset preserved

如果 restore FAIL：

先處理 restore，不做下一步研究。

---

## 十九、最後報告必答

1. v10.1 完成幾通
2. Protocol PASS / FAIL
3. Bridge contamination 幾次
4. Turn-boundary carry-over 是否確認
5. Normal eligible events
6. Short eligible events
7. raw CALLER FP
8. confirmed PURE_ECHO_FP
9. carry-over / bridge-related FP
10. EARLY
11. Normal recall
12. Median / P90 first usable latency
13. Short Caller 表現
14. latency 最大瓶頸
15. 最終 verdict
16. cleanup 是否 PASS
17. production barge-in 是否仍 OFF

---

## 二十、本輪唯一核心問題

> 在修正真人測試 protocol、固定 bridge、保留足夠 post-onset audio、並釐清 turn-boundary carry-over 後，frozen v9 Semantic Guard 在真實 HT813 double-talk 下，究竟是真的 `SEMANTIC_NOT_SEPARABLE`，還是 v10 結果受到 protocol contamination 影響？

不要調參。
不要換 ASR。
不要救結果。
先把 protocol 做乾淨，再重新驗收。

開始執行。