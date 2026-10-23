# EchoGuard v9 — Caller Fixtures 重建與 Semantic Latency Breakdown

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。請完整讀取本文件後再開始，不要自行改題。

## 一、目前狀態

v8 benchmark 已完成，但不能把結果直接解讀成「Semantic 只有 70% 成功率」。

v8 結果：

- 10 cases 中 7 cases 有 usable semantic decision
- 這 7 cases 全部是 `echo_only`
- median first usable semantic decision 約 1500ms
- 3 個 `double_talk` cases 全部沒有 usable decision

關鍵原因：

`b3bf3c3a` 的 caller onset 約在 11.57s，但 v8 fixture 在 onset 後只保留約 70ms 音訊。

因此這 3 個 `double_talk` cases 根本不足以回答：

- caller 是否可被 semantic guard 分離
- caller recall
- caller first usable decision latency
- caller stable decision latency
- short interruption capability

所以 v8 的 7/10 不是有效的 caller acceptance rate。

目前更準確狀態：

`CONDITIONAL_PASS / CALLER_DATA_BLOCKED`

已確認：

- semantic echo recoverability 成立
- sliding-window pipeline 能工作
- echo-only 約 1.0～1.8s 可得到 usable semantic evidence
- caller gate 尚未真正測到
- 目前 latency 仍有偏慢警訊

本輪 v9 的目的不是再調 LCS threshold，而是先把資料與 latency measurement 做對。

---

# 二、本輪核心問題

v9 必須回答兩件事：

1. **有足夠 post-onset audio 的真 caller interruption，Semantic Guard 到底能不能辨出 caller？**
2. **如果能，真正延遲到底卡在哪一段？**

最終要能把：

`caller onset → first usable semantic decision`

拆成可量化的 latency components。

---

# 三、禁止事項

本輪禁止：

- 把 70ms post-onset fixture 算成 caller FN
- 用 final transcript 冒充 realtime partial
- 為了過 gate 調 magic LCS threshold
- 換正式 ASR
- 把 SenseVoice 改成 TEA production
- 修改正式 9090
- deploy production
- 真正開 barge-in
- force push
- `git reset --hard`
- 用大量 derived samples 冒充 Original Real acceptance

本輪只允許：

- offline
- replay
- fixture rebuild
- local harness
- shadow research

---

# 四、Phase 0 — Repo / Dataset 盤點

開始前：

```bash
git status
git branch
git log --oneline -20
```

盤點：

- v8 harness / runner
- v8 `window_features.csv`
- v8 `separability.json`
- v8 `ACCEPTANCE_REPORT_V8.md`
- original full call recordings
- sliced fixture generation logic
- caller onset ground truth
- playback intervals
- echo-only intervals
- double-talk intervals
- TEA-ASR HTTP endpoint
- sliding-window polling logic
- latest-window-wins / coalescing logic

不要假設 fixture 就等於 full call source。

必須追到每個 fixture 的來源錄音與 source_call_uid。

---

# 五、先修正 Dataset Semantics

建立或更新每個 case metadata，至少包含：

- `case_id`
- `source_call_uid`
- `label`
- `source_audio_path`
- `fixture_audio_path`
- `fixture_start_ms`
- `fixture_end_ms`
- `caller_onset_ms`
- `caller_offset_ms`（若有）
- `post_onset_available_ms`
- `playback_active_intervals`
- `echo_only_intervals`
- `double_talk_intervals`
- `gt_confidence`
- `eligible_for_caller_latency`

新增明確 eligibility rule。

例如：

```text
post_onset_available_ms < 1000ms
→ DATASET_INSUFFICIENT_FOR_CALLER_LATENCY
```

不要把這種 case 算 FN。

如果需要 1.5～2.0s context 才能公平測 sliding-window semantic，則 gate eligibility 可以更嚴格。

請根據實際 window sweep 最大值設定。

---

# 六、重建 Caller Fixtures

對 `b3bf3c3a` 先回查完整原始真電話錄音。

目標 fixture：

- caller onset 前至少 2～3s
- caller onset 後至少 1.5～2.0s
- 保留 playback / echo context
- 不要只切 caller 本身

推薦形式：

```text
[pre-onset echo context 2~3s]
→ caller onset
→ [post-onset caller/double-talk 1.5~2.0s+]
```

如果原始完整錄音其實本身就只剩 70ms post-onset：

- 不准人工補造 Original Real
- 標記 `ORIGINAL_REAL_INSUFFICIENT`
- 從現有其他真電話錄音找完整 interruption
- 如果仍不足，才允許下一階段要求少量 3～5 通新 live interruption calls

Derived 可以先研究方法，但不得當 Original Real acceptance。

---

# 七、建立 Caller Fixture Audit

輸出：

`caller_fixture_audit.json`

至少列：

| case | source uid | label | pre-onset ms | post-onset ms | eligible | reason |

並在 report 中明確分成：

- Eligible Original Real
- Ineligible Original Real
- Derived Research Only

禁止再用總 case 數直接算 recall。

Caller recall denominator 只能使用 eligible caller cases。

---

# 八、Latency Breakdown

對每個 eligible caller case，把 latency 拆成至少以下 components：

## A. Poll wait

`caller onset → next eligible polling instant`

例如：

- poll interval 300ms
- caller 在兩次 poll 中間開始
- 真正等待 0～300ms

記：

`poll_wait_ms`

## B. Audio evidence accumulation

從 caller onset 到音窗內累積足夠 caller speech，讓 ASR 能產出可用文字。

記：

`audio_evidence_ms`

這不是 TEA inference latency。

## C. ASR request processing

從 request send 到 TEA response。

記：

- `asr_request_ms`
- P50/P90/P95

## D. Semantic compute

Transcript normalize + alignment + semantic judge。

記：

`semantic_compute_ms`

## E. Scheduling / queue / coalescing delay

記：

- `queue_wait_ms`
- `coalescing_delay_ms`
- `stale_drop_count`

## F. Total

`caller_onset → first usable correct semantic decision`

記：

`first_usable_semantic_decision_ms`

另記：

`first_stable_semantic_decision_ms`

---

# 九、定義 First Usable Decision

不能只要 ASR 回任何字就算 usable。

必須同時滿足：

- transcript 非空
- semantic state 可決策
- decision 與 ground truth 一致
- 不是 stale response
- 不是只有無意義 filler / hallucination

Caller case：

第一次正確出現 `SEM_CALLER` 或明確可接受的 caller-positive 狀態才算 usable。

Echo-only case：

第一次正確出現 `SEM_ECHO` 可算 usable，但 echo latency 與 caller latency必須分開報。

不要拿 echo median 1500ms 直接代表 caller latency。

---

# 十、重新跑 Window / Poll Sweep

沿用 v8，但只對 eligible data 計算 caller metrics。

至少比較：

- 200ms poll / 1.0s window
- 300ms / 1.0s
- 300ms / 1.5s
- 400ms / 1.5s
- 500ms / 1.5s
- 500ms / 2.0s

若實測顯示某些 window 根本沒有足夠 post-onset audio，該 case/variant 標 invalid，不得算 FN。

---

# 十一、重要：不要把 Window 長度誤認成必要等待時間

Sliding window 是 rolling buffer。

例如 window=1.5s 並不代表 caller onset 後一定要再等 1.5s。

Window 內可以包含：

- caller onset 前 echo
- caller onset 後新 speech

真正需要量的是：

> onset 後累積多少 caller speech，ASR 就開始產生可用新 transcript。

所以每個 decision 都要記：

`post_onset_audio_in_window_ms`

再分析：

- 100ms caller speech
- 200ms
- 300ms
- 500ms
- 800ms
- 1000ms

何時開始有可用 caller semantic evidence。

---

# 十二、Caller Transcript Evolution

對每個 caller case輸出 partial evolution timeline，例如：

```text
+200ms  ""
+400ms  "等"
+600ms  "等一下"
+800ms  "等一下你"
```

同時列：

- active TTS reference
- LCS
- edit similarity
- novel ratio
- semantic state
- confidence

目的：

找出到底是：

- caller speech 太短
- poll 太慢
- TEA 太慢
- ASR 需要更多音訊
- semantic rule 太保守

不要只報最終 latency。

---

# 十三、Echo-only 仍保留 Safety Gate

不能因為這輪重點是 caller，就放鬆 echo FP。

Echo-only 要求：

- `SEM_CALLER` FP = 0
- EARLY = 0

如果為了加快 caller decision 造成 echo FP：

該 variant 不可接受。

優先順序仍是：

1. Echo FP = 0
2. EARLY = 0
3. Caller recall
4. Caller latency

---

# 十四、Caller Recall 計算修正

Caller recall：

```text
TP eligible caller cases
/
all eligible caller cases
```

以下不得進 denominator：

- post-onset audio 不足
- caller onset ground truth 不可信
- fixture 截斷
- source audio 遺失

這些必須獨立標：

`DATASET_INSUFFICIENT`

---

# 十五、Short Interruption 另列

80～300ms interruption 不能混在 normal caller gate。

分成：

## Normal Caller

有足夠 speech duration，可以合理期待 ASR 輸出文字。

## Short Caller

80～300ms。

分開報：

- recall
- transcript availability
- latency
- fail reason

不要讓 short barge 把 normal caller acceptance 污染掉。

---

# 十六、ASR Latency Root Cause

若 caller decision仍 >1s，必須量清楚原因比例。

每個 case輸出：

```text
Total latency = poll_wait
              + audio_evidence_accumulation
              + queue/coalescing
              + ASR processing
              + semantic compute
```

報告中至少做 aggregate：

| Component | Median | P90 | % of total |

只有這樣才能決定下一步該優化 polling、window、ASR serving，還是根本 ASR 需要太多 speech context。

---

# 十七、Backlog / Latest-window-wins 再確認

延續 v8 safety：

- max one active request
- max one latest pending window
- stale pending 可以丟
- stale response 不得覆蓋新 state

記錄：

- queue max
- dropped stale windows
- request overlap
- stale response count

如果 queue/backlog 導致 latency，必須從 total latency 中分離。

---

# 十八、Resource Metrics

對最佳 2～3 個候選設定量：

- ASR P50/P90/P95
- CPU
- GPU util
- VRAM
- request throughput
- effective interim rate

如果 poll=200ms 但 ASR P90=700ms，必須說明 coalescing 後實際有效更新率，而不是宣稱 5Hz realtime。

---

# 十九、如果 Original Real Caller 仍不足

若盤點後真正 eligible Original Real caller cases < 3：

不要強行下 PASS/FAIL。

判定：

`CALLER_DATASET_INSUFFICIENT`

並產出一個最小 live data collection plan，但不要自行撥電話。

建議只需：

- 3～5 通真電話
- 每通至少 2 次 intentional interruption
- interruption onset 後保留 ≥2s
- 一部分短插話
- 一部分正常 0.5～1.5s caller speech

此時停止，等待真人取得資料。

---

# 二十、Acceptance Gate v9

只對 eligible Original Real 計算。

## Echo Safety

- Echo FP = 0
- EARLY = 0

## Normal Caller

目標：

- Recall ≥ 90%
- Median first usable semantic decision ≤ 500ms
- P90 ≤ 800ms

## Speed Classification

如果 semantic classification 可分但：

- P90 > 1000ms

則：

`SEMANTIC_SEPARABLE_BUT_TOO_SLOW`

如果 caller data 不足：

`CALLER_DATASET_INSUFFICIENT`

如果 caller/echo 仍大量混淆：

`SEMANTIC_NOT_SEPARABLE`

---

# 二十一、Final Verdict

最終只能選：

- `PASS`
- `CONDITIONAL_PASS`
- `CALLER_DATASET_INSUFFICIENT`
- `SEMANTIC_SEPARABLE_BUT_TOO_SLOW`
- `SEMANTIC_NOT_SEPARABLE`
- `FAIL`

不得把 dataset truncation 算 algorithm failure。

---

# 二十二、輸出檔案

建立：

`results/full_duplex_v9/`

至少包含：

- `ACCEPTANCE_REPORT_V9.md`
- `caller_fixture_audit.json`
- `latency_breakdown.json`
- `caller_timeline.csv`
- `echo_safety.json`
- `pareto.json`

若有 failure：

`failures/<case_id>/`

保存：

- audio reference
- transcript timeline
- feature timeline
- decision timeline
- fail reason

---

# 二十三、Final Report 必須回答

`ACCEPTANCE_REPORT_V9.md` 至少回答：

1. v8 的 3 個 double-talk 為何不能算 FN？
2. 找到的 Original Real caller 完整錄音有幾通？
3. 真正 eligible caller cases 有幾個？
4. caller recall 是多少？
5. caller median / P90 decision latency 是多少？
6. latency 最大瓶頸是 poll、audio evidence、ASR、queue 還是 semantic compute？
7. echo FP 是否仍為 0？
8. TEA sliding-window 是分類可行但太慢，還是實際可進 shadow runtime？
9. 是否需要補 3～5 通 live caller interruption 資料？
10. 下一步只能選一個主要方向。

---

# 二十四、最重要判斷原則

不要再用：

`7/10 = 70%` 

這種方式評估混有截斷 fixture 的 dataset。

本輪必須先確定：

`資料有沒有足夠訊號讓演算法有公平機會判斷？`

再談 recall。

同時不要因為 echo-only 可以在 1.5s 辨認，就直接假設 caller 也一定要 1.5s。

必須實測 caller onset 後最少需要多少 speech evidence。

---

# 二十五、執行順序

自行長跑：

盤點 → fixture audit → 找 full recordings → 重建 caller fixtures → eligibility → latency instrumentation → original real replay → poll/window sweep → echo safety → caller recall → latency breakdown → resource benchmark → final report。

不要每一步停下來問。

只有以下情況才停止：

- 找不到完整 caller source audio
- eligible Original Real caller < 3
- 需要重新真人撥電話
- 可能碰 production
- 需要付費外部資源

否則自行完成。

本輪不要修改正式 9090、不要 deploy、不要正式開 barge-in、不要自行換 ASR。

本輪唯一核心問題：

> **在資料完整且 ground truth 合格的真 caller interruption 上，Semantic Guard 到底能不能可靠辨出 caller？如果能，caller onset 到第一次可用 decision 的真實延遲到底花在哪裡？**

開始執行。