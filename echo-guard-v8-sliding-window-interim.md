# EchoGuard v8 — Sliding-Window Interim Semantic Barge-in 驗證

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。請完整讀取本文件後再開始，不要自行改題。

## 一、目前已確認狀態

v7 判定：

`CONDITIONAL_PASS`

原因不是 semantic 方向錯，而是目前既有 ASR 沒有 streaming partial / interim 輸出。

已完成 Gate A：

把真電話 echo RX 重新送進現有 ASR，能高比例辨回 AI 自己正在播放的台詞。

已知 4 通真電話例子：

- `82b0223c`：LCS 0.907
- `9aa10006`：LCS 0.889
- `9f7c1815`：LCS 0.907
- `48f15dca`：LCS 0.943

也就是 echo RX 與 Known TTS Transcript 的文字相似度約：

`0.89 ～ 0.94`

這遠高於 v6 Audio-only 聲學特徵約 0.5 的可分離度。

所以：

`SEMANTIC_ECHO_REFERENCE_NOT_RECOVERABLE`

已被否定。

Semantic side information 值得繼續。

---

# 二、目前 ASR 現況

正式 runtime：

- `SenseVoice`
- `engine/saas_stt.py`
- 整句 final transcript

PoC / research：

- `TEA-ASR-1.1`
- `engine/tea_asr_server.py`
- vLLM server `:19110`
- 整句 WAV → HTTP

目前兩者都沒有真正 streaming partial。

`transcribe_bytes()` 只會在收句後呼叫。

因此 v7 無法量：

- caller onset 後多久有第一個可用 partial
- semantic FP
- caller recall
- short interruption recall
- semantic fusion latency

禁止拿 final transcript 假裝即時 barge-in。

---

# 三、本輪任務

**不換 ASR。**

使用同一支目前 PoC 的 TEA-ASR-1.1，建立 sliding-window interim 模擬。

方法：

在 playback active / shadow research 模式下，對 RX rolling buffer 定期擷取最近一段音訊，送進既有 TEA HTTP ASR。

例如：

```text
每 200～500ms
取最近 1.0～2.0s RX window
→ TEA-ASR-1.1 HTTP
→ interim transcript
→ Known TTS Transcript alignment
→ SEM_ECHO / SEM_CALLER / SEM_UNKNOWN
```

這不是換 ASR，只是把既有單句 ASR 以 sliding-window 方式取得 interim evidence。

本輪仍然：

- 不修改正式 9090
- 不 deploy production
- 不正式開 barge-in
- 不 push production
- 先 offline / replay / local harness

---

# 四、最重要 KPI

本輪最重要指標不是單次 ASR HTTP latency。

真正 KPI 是：

`first_usable_semantic_decision_ms`

定義：

> 從 ground-truth caller onset 開始，到第一次產生「足以做正確 CALLER / ECHO 判斷」的 semantic decision 的時間。

例如：

```text
0ms      caller 開始說「等一下」
200ms    first poll
400ms    TEA interim =「等一下」
420ms    semantic judge = CALLER
```

則：

`first_usable_semantic_decision_ms = 420ms`

這才是 barge-in 真正體感 latency。

---

# 五、Phase 0 — Repo / Runtime 保護

開始前：

```bash
git status
git branch
git log --oneline -20
```

盤點：

- `engine/tea_asr_server.py`
- TEA-ASR HTTP endpoint
- `transcribe_bytes()`
- RX frame / rolling buffer
- EchoGuard shadow path
- Known TTS transcript source
- playback start / end timestamp
- current playback segment / text
- v7 dataset / report
- original real-call corpus

禁止：

- `git reset --hard`
- force checkout
- force push
- deploy
- 修改正式 9090
- 修改 production ASR
- 把正式 ASR 改成另一模型
- 真正 stop TTS

本輪只允許 shadow / replay / local。

---

# 六、先建立 Sliding Window Interim Harness

建立獨立 research runner，例如：

`semantic_interim_bench.py`

或依實際 repo 命名。

它必須能：

1. 讀入真電話 RX 錄音
2. 依 ground-truth playback / caller onset timeline 重播
3. 每 N ms 擷取最近 M ms / sec 的 RX window
4. 送進同一支 TEA-ASR-1.1 HTTP endpoint
5. 取得 interim transcript
6. 對 Known TTS active reference 做 semantic alignment
7. 輸出每次 interim decision 的完整 timeline

不得先改 runtime 才測。

---

# 七、Polling Interval Sweep

至少比較：

- 200ms
- 300ms
- 400ms
- 500ms

不要預設 300 或 500 最好。

---

# 八、Audio Window Sweep

至少比較：

- 1.0s
- 1.5s
- 2.0s

建議組合：

- 200ms poll / 1.0s window
- 300ms / 1.0s
- 300ms / 1.5s
- 400ms / 1.5s
- 500ms / 1.5s
- 500ms / 2.0s

如果有必要可補其他組合。

---

# 九、禁止 Queue Backlog

這是本輪重要安全條件。

如果 TEA inference 比 polling interval 慢：

不得：

```text
request 1
request 2
request 3
request 4
→ 全部排隊
```

這會造成 latency 越跑越高。

必須實作 / 模擬：

`latest-window-wins`

或 equivalent coalescing。

規則：

- 上一個 interim request 尚未完成時，不排多個 stale request
- 最多保留最新一個 pending window
- 舊 pending window 可丟棄
- 不允許 unbounded queue

必須量：

- queue depth
- dropped stale windows
- request processing time
- effective polling interval
- end-to-end decision latency

---

# 十、ASR Interim Timeline

每次 interim request 至少記錄：

- call_uid
- request_id
- poll_interval_ms
- window_ms
- window_start_ts
- window_end_ts
- caller_onset_relative_ms
- playback_active
- TEA request start
- TEA response end
- ASR processing ms
- transcript
- normalized transcript
- LCS / edit / ngram / novel-token features
- semantic state
- decision confidence
- first usable yes/no

---

# 十一、Known TTS Reference Window

不能每次都拿整篇 AI 台詞比。

沿用 v7 概念，建立 active reference window。

至少包含：

- 最近已播放內容
- 當前附近內容
- 少量 future context

要考慮實際 echo delay。

不要硬寫死 415ms。

從現有真電話資料估合理 delay range。

---

# 十二、Semantic Features

先沿用可解釋 deterministic feature：

- LCS ratio
- edit similarity
- character/token overlap
- n-gram overlap
- novel token ratio
- active-window containment
- partial sequence stability

不要一開始上 embedding / LLM。

如果文字相似度因同音誤辨不足，再加：

- pinyin sequence similarity
- tone-agnostic phonetic edit distance

但只能在有證據時加入。

---

# 十三、Semantic State

每次 interim 只能判：

- `SEM_ECHO`
- `SEM_CALLER`
- `SEM_MIXED`
- `SEM_UNKNOWN`

`SEM_UNKNOWN`：

- 不切
- 不算 caller TP
- 保持 fail-safe

`SEM_MIXED`：

獨立統計，不要硬塞 ECHO 或 CALLER。

---

# 十四、Ground Truth

Original Real 優先。

每通要有：

- playback intervals
- echo-only intervals
- caller onset
- caller offset
- true double-talk windows
- expected no-barge windows

如果某通 caller onset 不可信：

標 `UNKNOWN_GT`

不得硬算 latency。

---

# 十五、第一階段先只跑 Original Real

不要先拿大量 derived dataset 調到漂亮。

先用現有 11 段 original。

其中：

- echo-only / quiet calls
- true caller interruption calls
- 原本 EARLY regression
- 1.04s / 3.72s false-cut regressions

全部重跑 sliding-window interim。

---

# 十六、Echo-only Gate

對 echo-only windows：

要求：

- semantic false caller decision = 0
- semantic EARLY = 0
- echo partial 應大多被判 `SEM_ECHO` 或 `SEM_UNKNOWN`

任何 echo-only 被判 CALLER：

建立 failure bundle。

---

# 十七、Caller Gate

對真正 caller onset：

至少量：

- recall
- first usable decision latency
- first CALLER decision latency
- first stable CALLER decision latency
- false UNKNOWN duration
- mixed duration

不要只報「最後有辨到」。

---

# 十八、Latency Gate

候選目標：

- median `first_usable_semantic_decision_ms` ≤ 500ms
- P90 ≤ 800ms

如果：

`P90 > 1000ms`

標記：

`SEMANTIC_SEPARABLE_BUT_TOO_SLOW`

如果 classification 很好但速度不夠，也不能進 live barge-in。

---

# 十九、Short Interruption

特別測：

80～300ms 的短插話。

不要要求一開始 100%。

至少逐 case 報：

- 是否產生可用 interim
- 第一次 transcript 幾個字
- 是否 SEM_CALLER
- latency
- 為什麼 FN

尤其：

- 喂
- 嗯
- 欸
- 好
- 不是
- 等一下

但禁止對這些詞建立硬編碼 whitelist 作弊。

---

# 二十、Polling / Window Pareto

最後輸出表：

| Poll | Window | Echo FP | Caller Recall | Median Decision | P90 | ASR ms | Drop % | Queue Max |

排序優先：

1. Echo FP = 0
2. EARLY = 0
3. Recall
4. P90 latency
5. resource cost

不要只選最快。

---

# 二十一、Resource Benchmark

必須量：

- TEA ASR avg request ms
- P50/P90/P95 request ms
- CPU
- GPU utilization（若可取得）
- VRAM
- concurrent request behavior
- queue / coalescing behavior

最重要：

判斷目前硬體是否能長時間每 200～500ms 做 interim。

---

# 二十二、Single-Call Long-run Stress

至少模擬長通話：

- 3 分鐘
- 5 分鐘

確認：

- queue 不累積
- memory 不持續成長
- request latency 不漂移
- stale interim 不會晚到後覆蓋新 decision

---

# 二十三、Stale Response Protection

如果舊 request 比新 request 晚回：

禁止舊 transcript 覆蓋目前 semantic state。

每個 interim 必須帶：

- request sequence id
- audio window end timestamp

Semantic Judge 只接受仍然有效的最新 response。

---

# 二十四、Decision Stability

不要單一 partial 一出現就立刻算穩定。

比較：

- first usable decision
- first stable decision

可以測：

- 1 次即可
- 2 次 consecutive
- 3 次 consecutive

但不要為了 stability 把 latency拖到 >1s。

---

# 二十五、Acoustic Fusion 暫時只做 supporting evidence

v6 已證明 acoustic-only 不可分。

本輪 primary 研究先看 semantic interim 本身能否做到：

- echo FP=0
- caller 可分
- latency可接受

只有 semantic 單獨有希望後，才加入 acoustic supporting feature。

不要讓 v6 acoustic rule 重新阻擋 semantic caller。

---

# 二十六、Acceptance Gate v8

Original Real：

## Echo

- Echo semantic FP = 0
- EARLY = 0

## Caller

- Recall ≥ 90% 目標
- 至少明確列每通 caller case

## Latency

- median ≤ 500ms 目標
- P90 ≤ 800ms 目標

## Runtime

- queue max ≤ 1 pending latest window
- no unbounded backlog
- no crash
- no stale response override

---

# 二十七、判定

最終只能選：

## PASS

Semantic interim 分得開，且 latency / runtime 都可進 shadow runtime。

## CONDITIONAL_PASS

方向可行，但 sample 數或 latency 還需改善。

## SEMANTIC_SEPARABLE_BUT_TOO_SLOW

分類成立，但 P90 > 1000ms 或 resource/backlog 不可接受。

## FAIL

Echo/caller semantic interim 仍不可可靠分。

---

# 二十八、如果 PASS / CONDITIONAL_PASS

下一步只能：

`shadow runtime integration`

不能直接開 barge-in ON。

Shadow 要記：

- actual playback
- all interim transcripts
- semantic decisions
- would-barge timestamp
- first usable semantic decision
- actual caller onset（可人工標）

至少收 5～10 通 live shadow 後再談 ON。

---

# 二十九、如果 TOO_SLOW

不要立刻換 ASR。

先分析 latency 組成：

```text
poll wait
+ window readiness
+ HTTP overhead
+ ASR inference
+ semantic judge
```

量化哪一段最慢。

如果主要是 polling：

調 poll/window。

如果主要是 TEA inference：

才提出是否需要：

- 更小 window
- batching / no batching
- persistent connection
- local in-process adapter
- 更快同系列模型

但不得自行換 production ASR。

---

# 三十、如果 FAIL

如果 Echo transcript 仍高相似，但 caller interim 無法在合理 latency內穩定產生：

清楚分辨：

- ASR recognition failure
- partial timing failure
- short-token instability
- echo+caller mixture failure

不要模糊寫「semantic 不行」。

---

# 三十一、產出

建立：

`results/full_duplex_v8/`

至少：

- `interim_config_sweep.csv`
- `original_interim_results.csv`
- `short_barge_results.csv`
- `latency_distribution.csv`
- `resource_benchmark.json`
- `queue_behavior.json`
- `stale_response_tests.json`
- `best_config.json`
- `failure_cases.json`
- `ACCEPTANCE_REPORT_V8.md`

如果修改 research code：

可 commit 本機，但：

- 不 push
- 不 deploy
- 不碰 9090

---

# 三十二、最終報告必答

1. 最佳 poll / window 組合是什麼？
2. Echo semantic FP 是否為 0？
3. Caller recall 是多少？
4. `first_usable_semantic_decision_ms` median / P90 / P95 是多少？
5. 短插話 80～300ms 成功率多少？
6. TEA 每次 interim inference P50/P90/P95 是多少？
7. 有沒有 queue backlog？
8. `latest-window-wins` 是否有效？
9. 長通話 3～5 分鐘是否 latency 漂移？
10. 是否可進 `shadow runtime integration`？

---

# 三十三、最重要原則

這輪不是重新研究 ASR 準確率。

真正問題只有：

> **同一支既有 TEA-ASR，透過 sliding-window interim，能不能在 caller onset 後約 400～800ms 內，可靠產生足夠的 semantic evidence，同時保持 echo FP=0？**

不要換模型，不要碰 production，不要用 final transcript 假裝 realtime，不要讓 queue backlog 污染 latency。

開始執行。