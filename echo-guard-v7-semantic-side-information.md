# EchoGuard v7 — Semantic Side Information 驗證（ASR-Agnostic）

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。請完整讀取本文件後再開始，不要自行改題。

## 一、目前結論

v6 已正式判定：

`AUDIO_ONLY_NOT_SEPARABLE`

以真電話語料做最後一次 Audio-only 可分離性驗證，caller onset 與 echo burst 的候選聲學特徵 AUC 全部約等於隨機：

- similarity：0.569
- excess energy：0.551
- band novelty（8 bands）：0.555
- mel residual：0.478
- d(sim)/dt：0.497

caller 約 1014 frames（9 通）、echo 約 5030 frames。

已確認：

- echo return excess 約 +11～34 dB
- caller excess 約 +20.7～29.5 dB
- temporal pattern 也高度重疊
- 繼續調 Audio-only DSP threshold 不再值得

因此本輪禁止繼續做：

- similarity threshold sweep
- excess-energy threshold sweep
- band novelty threshold sweep
- mel residual threshold sweep
- acoustic-only shallow classifier
- Neural AEC / DTLN / AEC3 重試

本輪真正新增以前沒有的 side information：

> **AI 已知自己正在播放哪一句 TTS 文字，並利用目前既有 ASR 的 partial / interim / incremental transcript，判斷 RX 裡的語音是不是 AI 自己的回音。**

---

# 二、最重要限制：禁止自行把 ASR 換成 Qwen

本任務必須 **ASR-agnostic**。

不要假設目前 ASR 是 Qwen、Whisper、SenseVoice、Paraformer 或任何指定模型。

開始前先盤點 repo 與 runtime，找出：

- 目前實際 production / PoC 正在使用哪個 ASR
- ASR adapter / service / process 在哪裡
- 是否已支援 streaming partial / interim / incremental result
- partial 的輸出格式
- partial latency
- final transcript latency
- 是否已有 timestamps / confidence / token timing

只有實際 repo / runtime 證據能決定。

禁止：

- 為了本任務把正式 ASR 換成 Qwen
- 因為某模型方便就繞過現有 ASR
- 修改正式 9090 ASR stack
- deploy production

如果目前 ASR 沒有 partial 能力：

先研究如何利用「同一個現有 ASR」取得 incremental / chunked / partial 結果。

如果現有 ASR 技術上完全不支援 partial，才可以建立 offline research adapter 比較替代方案，但不得默默更換正式 ASR。

---

# 三、本輪核心概念

現在 AI 播放：

```text
您好，這裡是 Ticenpi，請問現在方便講電話嗎？
```

這段文字系統本來就知道。

同時電話 RX 經過目前既有 ASR，可能產生 partial：

```text
您好這裡是...
```

如果 RX transcript 高度對應目前 TTS 文字：

→ 強烈傾向 `ECHO`

如果 RX 出現：

```text
等一下
你哪裡
不用
我現在不方便
```

且這些內容無法由目前 TTS transcript 解釋：

→ `CALLER_CANDIDATE`

這就是本輪的真正新資訊。

---

# 四、目標架構

```text
                     ┌─────────────────────────┐
                     │ Known TTS Transcript    │
                     │ + playback position     │
                     └────────────┬────────────┘
                                  │
RX Audio ──→ Existing ASR partial/interim ─────┤
     │                            │             │
     │                            ▼             │
     │                   Transcript Alignment  │
     │                                          ▼
     └────→ Existing Acoustic EchoGuard ─→ Semantic Fusion Judge
                                             │
                              ┌──────────────┼──────────────┐
                              ▼              ▼              ▼
                            ECHO          CALLER         UNCERTAIN
                              │              │              │
                           不切            可切            不切
```

`UNCERTAIN → 不切` 必須保留。

本輪不是讓 ASR 單獨決定，而是增加新的 semantic evidence。

---

# 五、Phase 0 — Repo / Runtime 盤點

開始前：

```bash
git status
git branch
git log --oneline -20
```

盤點：

- EchoGuard runtime
- TTS pipeline
- TTS text source
- playback session id
- playback start/end timestamp
- TTS sentence / segment queue
- ASR implementation
- ASR streaming capability
- ASR partial event
- ASR final event
- caller turn / sentence collection logic
- shadow logs
- Acceptance Harness v3～v6

禁止：

- reset hard
- force checkout
- force push
- production deploy
- 修改正式 9090
- 真正開 barge-in ON

所有實驗先 offline / replay / local harness。

---

# 六、先取得 Known TTS Timeline

不能只保存整句文字。

建立可以知道「目前播放到哪裡」的 TTS timeline。

至少保存：

- playback_session_id
- text_segment_id
- full_tts_text
- segment text
- playback_start_ts
- playback_end_ts
- current playback elapsed ms
- 已播放文字的大約範圍

如果 TTS engine 有 word / phoneme / token timing：

直接利用。

如果沒有：

先做 coarse alignment，例如依 audio duration + text/token length 建立粗略 progress estimate。

不要為了 timing 精準度重建整個 TTS engine。

---

# 七、Semantic Reference Window

比對 ASR partial 時，不要永遠拿整段 TTS 全文比較。

建立：

`active_tts_reference_window`

內容包含：

- 已經播放過的文字
- 現在附近正在播放的文字
- 少量 future context

原因：echo 延遲存在，RX 裡出現的通常是約幾百 ms 前的 TTS 內容。

需要考慮現有實測 echo delay 分布。

不要硬寫死單一 415ms；以現有資料建立 reasonable reference window。

---

# 八、ASR Partial Capture

先驗證目前 ASR 到底能不能產生 partial / interim。

每個 partial 至少記：

- timestamp
- text
- confidence（若有）
- stability（若有）
- token/word timing（若有）
- is_final
- source call uid
- playback active yes/no

如果現有 ASR 只有 final transcript：

先量 final latency。

若 final latency 已經明顯 >700～1000ms，不能直接拿 final transcript 作即時 barge-in 主判斷，必須研究 incremental chunking。

---

# 九、Transcript Normalization

電話 ASR 文字不能直接 raw string exact match。

建立繁中 normalization：

- punctuation removal
- full-width / half-width normalize
- whitespace normalize
- common filler normalize（視需要）
- Arabic / Chinese number normalize（若需要）
- 同音常見誤辨的有限 normalization（只能有證據後加入）

禁止建立大量人工 phrase whitelist 來作弊過 dataset。

---

# 十、Semantic Echo Features

至少測以下 feature family。

## A. Character / token overlap

- LCS ratio
- edit similarity
- normalized CER-like similarity
- token overlap
- n-gram overlap

## B. Position-aware overlap

ASR partial 是否對應到 active TTS reference window。

不要只問：

「這句是否出現在整篇 TTS 裡」

要問：

「這句是否出現在目前 echo delay 合理範圍內、剛剛播放過的 TTS 文字裡」

## C. Novel token ratio

RX partial 有多少內容無法被 active TTS reference 解釋。

例如：

```text
TTS: 您好請問現在方便講電話嗎
RX:  您好請問現在方便
```

novel ratio 很低 → echo evidence 高。

```text
TTS: 您好請問現在方便講電話嗎
RX:  等一下你哪裡
```

novel ratio 高 → caller evidence 高。

## D. Temporal transcript consistency

不要只看單次 partial。

觀察 partial sequence：

```text
您好
您好請問
您好請問現在
```

如果持續跟著目前 TTS transcript 演進：

→ 強 echo evidence。

若 partial 突然形成獨立新內容：

→ caller evidence。

---

# 十一、不要一開始做 Embedding / LLM Semantic Similarity

第一階段先用：

- char/token alignment
- LCS
- edit distance
- n-gram overlap
- novel token ratio
- position-aware transcript alignment

原因：

這些快、穩、可解釋、沒有 API 成本。

只有 deterministic text alignment 無法泛化時，才研究：

- small local sentence embedding
- phonetic / pinyin similarity

禁止一開始把每個 partial 丟給大型 LLM 判斷。

---

# 十二、Phonetic Matching（第二層候選）

ASR 可能把 echo 文字辨成同音不同字。

如果純文字 alignment 不足，可以建立：

- Mandarin pinyin sequence
- tone-agnostic pinyin sequence
- phonetic edit distance

比較：

Known TTS text → pinyin

vs

ASR partial → pinyin

這可以處理：

字錯但聲音內容其實是同一句。

但必須 benchmark 後才加入，不要先過度複雜化。

---

# 十三、Semantic States

建立 semantic judge 狀態：

- `SEM_ECHO`
- `SEM_CALLER`
- `SEM_MIXED`
- `SEM_UNKNOWN`

## SEM_ECHO

partial 高度可由 active TTS reference 解釋。

## SEM_CALLER

partial 有穩定的新內容，且與 active TTS reference 低重疊。

## SEM_MIXED

一部分像 TTS，一部分有新 caller 內容。

這可能是真正 double-talk 的重要訊號。

## SEM_UNKNOWN

文字太短、ASR 不穩、沒有 partial、confidence 太低。

`SEM_UNKNOWN` 不得直接 trigger barge-in。

---

# 十四、短插話特別處理

短插話例如：

- 喂
- 嗯
- 欸
- 好
- 不是
- 等一下

可能只有 1～4 個中文字。

因此長字串 similarity 對短插話不公平。

建立獨立 short-partial feature：

- exact active-TTS containment
- novel short token
- repeated stability across partial frames
- acoustic onset evidence

禁止建立只對目前 4 個測試詞有效的 hardcoded whitelist。

可以有「短 token 規則」，但不能依特定 phrase UID 過擬合。

---

# 十五、Acoustic + Semantic Fusion

v6 已證明 acoustic-only 不可分。

但 acoustic evidence 仍可當 supporting feature。

建立 fusion，不要完全刪除 EchoGuard。

候選邏輯：

## Case A

Acoustic uncertain
+
Semantic = ECHO

→ NO BARGE

## Case B

Acoustic caller-like
+
Semantic = ECHO

→ NO BARGE

## Case C

Acoustic caller-like / onset
+
Semantic = CALLER

→ CALLER CANDIDATE

## Case D

Acoustic uncertain
+
Semantic = CALLER 且 partial 持續穩定

→ CALLER CANDIDATE

## Case E

Semantic UNKNOWN

→ fail-safe，不切

實際條件由 benchmark 決定。

---

# 十六、不要讓 ASR Echo 自己觸發循環

如果 caller 尚未存在，AI echo 被 ASR partial 辨出來：

Semantic Guard 必須壓掉 barge-in。

如果 caller 和 echo 同時存在：

ASR partial 可能混在一起。

所以 `SEM_MIXED` 必須獨立統計。

不要強迫所有 partial 只能二分類 echo/caller。

---

# 十七、Dataset 建設

沿用既有 Original Real / Derived dataset。

新增 semantic dataset metadata：

- known_tts_text
- playback segment
- playback timeline
- rx audio
- ground-truth caller onset
- expected echo-only windows
- expected caller windows
- ASR partial sequence
- ASR final transcript
- semantic label

原始真電話仍最高優先。

---

# 十八、先離線重跑現有真電話

不需要先重新打電話。

把現有真電話錄音重新送進：

目前實際使用的 ASR

得到：

- partial sequence（若可）
- final transcript
- timestamps

再跟該時間點 known TTS transcript 做 alignment。

先回答：

> echo-only RX 經現有 ASR 後，究竟有多少比例仍能辨回 AI 自己正在講的內容？

這是 v7 第一個關鍵 gate。

---

# 十九、Experiment A — Echo ASR Recoverability

對所有 echo-only windows 統計：

- ASR 有輸出的比例
- partial latency
- transcript overlap with TTS
- pinyin overlap（若測）
- 完全亂碼 / unrelated transcript 比例

如果 echo 經遠端 AEC 後 ASR 大部分根本辨不出 TTS 內容：

Known-TTS semantic route 可能不可行，要及早停止。

不要硬做後面 classifier。

---

# 二十、Experiment B — Caller Novelty

對 caller / double-talk windows 統計：

- ASR partial latency
- novel token ratio
- overlap with active TTS
- mixed transcript ratio
- short caller recoverability

比較 echo-only vs caller 的 semantic feature AUC / PR-AUC。

v6 acoustic AUC 約 0.5。

本輪希望 semantic feature 明顯高於 acoustic baseline。

---

# 二十一、Experiment C — Position-aware Alignment

比較：

1. 對整句 TTS 全文 match
2. 對 active playback window match
3. 對 delay-aware playback window match

看哪個能降低 false semantic echo / false caller。

---

# 二十二、Experiment D — Partial Sequence

比較單一 partial vs rolling partial sequence：

例如 100 / 200 / 300 / 500ms evidence window。

量：

- FP
- EARLY
- Recall
- median decision latency
- P90 latency

不要只看 transcript accuracy。

產品真正關心的是「多久能安全決定 caller」。

---

# 二十三、Experiment E — Phonetic Matching（必要時）

只有字面 alignment 明顯被同音誤字拖累時才加入。

比較：

- char similarity
- pinyin similarity
- combined score

如果沒有改善就移除，不要增加 runtime complexity。

---

# 二十四、Optional：TTS Speaker Identity（只做研究，不是主線）

如果 semantic transcript 有部分 case 仍難分，可以研究固定 TTS voice speaker embedding。

目標：

RX 片段是否仍像已知 TTS speaker。

但這只是 optional supporting evidence，因為 PSTN / codec / remote AEC 可能嚴重改變 speaker embedding。

本輪主線仍是 transcript side information。

---

# 二十五、主動 Watermark 暫不做

active acoustic watermark / pilot fingerprint 是另一條真正新路線，但需要改播放音訊。

本輪先不要做。

只有 semantic-assisted route 失敗後，再另開 v8 研究。

---

# 二十六、Acceptance Harness v7

建立或擴充：

`echo_guard_semantic_acceptance.py`

或整合既有 acceptance harness。

要求可一鍵跑：

```bash
python echo_guard_semantic_acceptance.py --suite original
python echo_guard_semantic_acceptance.py --suite all
```

輸出：

- ASR partial logs
- semantic alignment rows
- per-call metrics
- failure bundles
- final report

---

# 二十七、核心指標

至少：

## Echo-only

- semantic FP
- final fusion FP

## Caller

- semantic recall
- fusion recall

## Double-talk

- mixed detection recall
- caller detection recall

## Latency

從 ground-truth caller onset 到：

- first usable partial
- first SEM_CALLER
- final would-barge

統計：

- median
- P75
- P90
- P95
- max

---

# 二十八、Failure Bundle

每個：

- semantic FP
- FN
- EARLY
- >700ms slow decision

都輸出：

- RX audio
- known TTS text
- playback timeline
- ASR partial sequence
- normalized partial
- text overlap scores
- pinyin score（若有）
- acoustic features
- final fusion state
- ground truth

---

# 二十九、Acceptance Gate v7

Original Real 優先。

理想 gate：

- Echo-only FP = 0
- EARLY = 0
- Caller Recall ≥90%
- Normal caller median decision latency ≤500ms
- P90 ≤800ms

Short caller 要獨立報告，不准為追 short recall 引入 echo FP。

如果 partial ASR 本身就需要 1～2 秒才穩定：

要誠實報告，不准把 final transcript latency 當即時 barge-in 成功。

---

# 三十、Stop Conditions

## Stop A — Echo ASR 不可恢復

如果 echo-only RX 經目前 ASR 後，大多數 transcript 無法與 Known TTS 建立穩定關係：

→ 標記 `SEMANTIC_ECHO_REFERENCE_NOT_RECOVERABLE`

停止大量 threshold sweep。

## Stop B — Partial 太慢

如果 semantic separation 很好，但 ASR partial latency 導致實際 decision P90 明顯 >1000ms：

→ 標記 `SEMANTIC_SEPARABLE_BUT_TOO_SLOW`

不要假裝產品已可用。

## Stop C — Semantic 仍重疊

如果 echo 與 caller semantic features 仍高度重疊：

→ `SEMANTIC_NOT_SEPARABLE`

下一步才考慮：

- TTS speaker identity
- active watermark
- endpoint / SIP 路徑

---

# 三十一、不要過擬合

禁止：

- 依 call UID 特判
- 依目前幾個 caller phrase hardcode
- 建立大量特殊字詞 whitelist 只為過 dataset
- 只報 train result

如果有 learned classifier：

按 `source_call_uid` group split。

同一通電話所有 derived variant 不得跨 train/test。

---

# 三十二、Runtime 整合策略

只有 offline acceptance 明顯有效後，才規劃 runtime。

Runtime 建議模式：

- `off`
- `shadow`
- `semantic-shadow`
- `on`（本輪禁止正式啟用）

`semantic-shadow`：

- 接收現有 ASR partial
- 做 semantic judge
- 記錄 would've-barge
- 永遠不停止播放

正式 9090 不動。

---

# 三十三、最終輸出

建立：

`full_duplex_v7/`

至少包含：

- `asr_inventory.json`
- `tts_timeline_samples.json`
- `echo_asr_recoverability.csv`
- `caller_semantic_features.csv`
- `partial_latency.csv`
- `semantic_alignment_sweep.csv`
- `fusion_results.csv`
- `failure_bundles/`
- `best_config.json`
- `ACCEPTANCE_REPORT_V7.md`

---

# 三十四、ACCEPTANCE_REPORT_V7.md 必須回答

## Q1

目前實際使用的 ASR 是什麼？

是否支援 partial / interim？

禁止猜。

## Q2

Echo-only RX 經目前 ASR 後，有多少比例仍能辨回正在播放的 Known TTS？

## Q3

文字 alignment / novel token / pinyin 是否比 v6 acoustic feature 有更高可分離性？

## Q4

Caller onset 後多久能取得第一個可用 partial？

## Q5

Semantic-only FP / Recall / latency？

## Q6

Acoustic + Semantic fusion 是否真的優於 semantic-only？

## Q7

短插話是否可用？

## Q8

是否值得進 5～10 通 live semantic-shadow acceptance？

---

# 三十五、最終判定只允許以下幾種

- `PASS_TO_LIVE_SEMANTIC_SHADOW`
- `CONDITIONAL_PASS`
- `SEMANTIC_ECHO_REFERENCE_NOT_RECOVERABLE`
- `SEMANTIC_SEPARABLE_BUT_TOO_SLOW`
- `SEMANTIC_NOT_SEPARABLE`
- `FAIL`

---

# 三十六、工作模式

自行長跑：

盤點目前 ASR → 建 Known-TTS timeline → replay 現有真電話 → 收 partial → semantic feature → alignment benchmark → fusion → acceptance → final report。

不要每一步停下來問。

只有以下情況才停：

- 現有錄音完全缺 Known TTS 對應文字
- ASR 需要人工 credential / 付費服務
- 需要真人重新撥電話
- 可能碰 production
- 需要實體硬體操作

否則自行完成。

---

# 最重要原則

本輪不是「換 ASR」。

本輪是：

> **保持目前實際使用的 ASR 不動，額外利用它的 partial / incremental transcript，配合系統本來就知道的 TTS 文字，增加以前 v3～v6 完全沒有的 semantic side information。**

如果目前 ASR 沒 partial，先研究同一 ASR 的 incremental 方法；只有完全不可行時才提出替代方案，不准自行切換正式 ASR。

開始執行。