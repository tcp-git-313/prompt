# EchoGuard v6 — 最後一次 Audio-only 可分離性驗證

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。請完整讀取本文件後再開始，不要自行改題。

## 任務背景

目前 v5 已依 Stop Condition 正確停止。

判定：

**Excess-energy hypothesis failed as primary discriminator**

不要再 sweep 單一 excess threshold。

v5 已確認：

- 舊 echo gain estimator 會被 caller 污染約 **+4 ～ +17.7 dB**。
- 修正 estimator 後，caller excess peak 約 **20.7 ～ 29.5 dB**。
- 但 echo-only burst 也可以到 **11 ～ 34 dB**。
- 因此 `positive excess` 無法單獨區分 caller 與遠端 echo suppression 結束後的 echo return。
- similarity 也已證明不能單獨當主要 discriminator。
- robust trust 可以消除 EARLY，但會犧牲 recall。

目前正式 9090 / production 未動、barge-in 未正式開啟、未 push。

本輪定義為：

> **最後一次純 Audio-only / TX-reference 路線的可分離性驗證。**

如果 v6 仍無法可靠區分 caller 與 echo-return，不要再無限調 DSP；必須明確轉向 side information，例如已知 TTS 內容、ASR partial、TTS voice fingerprint，或重新驗證 endpoint / SIP path。

---

# 一、不要直接實作對稱 Gain Clamp

原本建議：

`±1 dB / 100ms`

本輪不要直接照做。

原因：

當遠端 echo 被壓低後突然恢復：

```text
actual echo ↑↑
predicted echo 仍低
```

如果 upward gain 也只能慢慢增加，會讓 positive excess spike 維持更久，反而增加 false barge 風險。

因此 v6 改成：

> **Asymmetric Echo Tracking：下降慢、上升快，但只有確認是 echo-return 時才允許快速上修。**

---

# 二、v6 核心問題

現在真正要區分：

## A. Echo return

遠端 echo 被壓掉：

```text
正常 echo
↓
silence / weak echo
↓
原本 TX echo 突然恢復
```

## B. Caller onset

遠端 caller 真正開始說話。

兩者都可能同時造成：

- RX energy 突然增加
- excess 大幅增加
- similarity 暫時下降
- VAD positive

所以單幀 feature 已經不夠。

v6 改成：

**Temporal + Spectral + TX-reference evidence**

---

# 三、本輪只研究三類主要 discriminator

1. **Echo-return temporal detector**
2. **Band-wise / log-mel new-content detector**
3. **100～400ms temporal sequence decision**

只有上述仍不足時，才允許進入小型 shallow classifier。

不要重新大改整套 audio pipeline。

禁止回頭重新投入：

- Neural AEC
- DTLN
- 大型 speech enhancement model
- 無限 similarity threshold sweep
- 無限 global energy threshold sweep

---

# 四、Phase 0 — Repo / Baseline 保護

開始前先執行：

```bash
git status
git branch
git log --oneline -20
```

確認：

- current branch
- dirty files
- untracked files
- v5 Acceptance report
- Acceptance Harness 可正常執行

先保存 v5 baseline。

禁止：

- `git reset --hard`
- force checkout
- force push
- push production
- deploy
- 修改正式 9090
- 修改 production dialplan
- 真正打開 barge-in ON

所有工作都只能：

- offline
- replay
- harness
- local experiment

每一層修改完成後都必須執行：

```bash
python echo_guard_acceptance.py --suite all
```

不要一次把全部 v6 功能寫完才看結果。

---

# 五、Experiment A — Echo Return Detector

建立明確狀態：

```text
ECHO_ACTIVE
    ↓
ECHO_SUPPRESSED
    ↓
ECHO_RETURNING
    ↓
ECHO_ACTIVE
```

必要時保留：

- `NEAR_END_CANDIDATE`
- `DOUBLE_TALK`
- `UNCERTAIN`

---

# 六、如何判斷 ECHO_SUPPRESSED

候選 evidence：

TX 仍 active，而且原本 echo path 已建立，但：

- RX echo energy 顯著下降
- similarity / envelope evidence 下降
- predicted echo 明顯高於 actual RX
- delay path 原本已建立且仍可信

這時不要立即降低 long-term echo gain。

進入：

`ECHO_SUPPRESSED`

---

# 七、Suppressed State 必須保留 Pre-drop Baseline

保存至少：

- `pre_drop_echo_gain`
- `pre_drop_delay`
- `pre_drop_band_profile`
- `pre_drop_mel_profile`
- TX reference alignment
- suppression start timestamp

遠端 echo 暫時消失時，不能馬上忘記原本 echo path。

---

# 八、如何判斷 Echo Return

RX 突然重新變強時，比較它是不是：

> 「原本應該存在的 TX echo 回來了。」

至少比較：

## 1. TX-envelope alignment

RX burst 是否與 delay-shifted TX 的能量變化一致。

## 2. Mel / band trajectory

RX burst 的頻譜變化是否能被目前 TX reference 解釋。

## 3. Pre-drop echo profile

新的 burst 是否回到 dropout 前的：

- echo gain
- band pattern
- mel pattern
- delay pattern

## 4. Temporal derivative

觀察 burst 恢復前後 100～400ms 的時間軌跡，而不是只看單幀。

---

# 九、Echo Return 可以 Fast Reacquire

如果 `EchoReturnConfidence` 高：

允許 echo gain 快速上調回 pre-drop baseline。

不要慢慢 `+1 dB / 100ms`。

目的：

避免 one-frame / multi-frame lag 製造巨大假 excess。

---

# 十、Caller 不准快速更新 Gain

如果 burst 無法被 TX reference 解釋：

```text
gain freeze
↓
NEAR_END_CANDIDATE
```

Caller / Double-talk / Uncertain frame 不得污染 echo gain estimator。

---

# 十一、Experiment A2 — Asymmetric Echo Tracking

## Downward adaptation

要慢，因為 echo 突然下降很可能只是 remote AEC / gate。

至少比較：

- frozen during suppression
- `-0.25 dB / 100ms`
- `-0.5 dB / 100ms`
- `-1 dB / 100ms`

## Upward adaptation

只有 `EchoReturnConfidence` 高時允許快速恢復。

至少比較：

- immediate restore to pre-drop baseline
- fast EMA
- `+3 dB / frame`
- `+6 dB / frame`

實際最佳值由 benchmark 決定。

---

# 十二、重要：Current-frame Causal Ordering

禁止：

```text
看到 current RX 很大
→ 先把 gain 拉高
→ 再拿新的 gain 判 current frame 是 echo
```

必須：

1. previous state / previous gain
2. extract current evidence
3. classify current event
4. 若判定 high-confidence echo return
5. 更新「下一 frame」gain

保持完全 causal。

---

# 十三、Experiment B — Band-wise New Content

不要只看 total RMS。

依實際 sample rate，把電話有效語音頻帶合理拆成多個 bands。

建議比較：

- 6 bands
- 8 bands
- 10 bands
- 12 bands

實際數量由 benchmark 決定。

---

# 十四、核心概念

Echo return：

雖然總能量突然增加，但增加的 band pattern 應與目前 TX reference 有結構關係。

Caller：

會加入 TX reference 原本無法解釋的新 spectral structure。

因此建立：

`new_band_score`

或等效 feature family。

---

# 十五、New Band Features

至少測：

- positive residual per band
- normalized band-energy vector difference
- cosine distance
- number of unexplained bands
- largest unexplained band
- mel residual
- spectral flux residual
- temporal derivative of band residual

不要要求 caller 必須出現「完全新的頻帶」。

核心是：

> RX spectral pattern 有多少無法被 delayed TX reference + existing echo profile 解釋。

---

# 十六、Reference 不使用 Waveform Phase 作主要證據

因為已證明 waveform correlation 太差。

主要比較：

- log-mel
- band energy
- envelope trajectory
- temporal derivative
- delay-aligned TX structure

waveform correlation 可以保留 diagnostic，但不能當主要 discriminator。

---

# 十七、Experiment C — Temporal Sequence

不要再用單 frame：

```text
if score > threshold
```

建立約：

**100～400ms rolling feature sequence**

每 20ms frame 至少保存：

- similarity
- envelope similarity
- RX dB
- TX dB
- predicted echo dB
- excess dB
- band novelty
- mel residual
- delay
- delay confidence
- echo state
- VAD
- gain state
- derivative features
- final state
- would-barge

---

# 十八、比較兩類 Temporal Signature

## Echo dropout → return

可能形態：

```text
TX active
→ RX echo ↓
→ ECHO_SUPPRESSED
→ RX burst ↑
→ spectral / envelope pattern重新與 TX 對齊
→ ECHO_RETURNING
```

## Caller

可能形態：

```text
TX active
→ RX 出現獨立 onset
→ band novelty ↑
→ TX-reference explainability ↓
→ caller activity 持續
```

這是本輪真正要驗證的假設。

---

# 十九、先做 Rule-based Sequence

第一階段不要直接上 ML。

例如概念：

```text
ECHO_SUPPRESSED
+ RX burst
+ TX explainability快速恢復
→ ECHO_RETURN
```

否則：

```text
new-band evidence高
+ TX explainability持續低
+ near-end activity持續
→ NEAR_END / DOUBLE_TALK
```

實際條件必須由 harness 驗證，不要硬寫 magic threshold。

---

# 二十、如果 Rule-based 不夠，才允許 Small Classifier

可使用：

- logistic regression
- shallow decision tree
- small random forest
- small gradient boosting

不要使用大型 neural model。

---

# 二十一、ML 嚴格防 Data Leakage

如果使用 classifier：

split 必須按照：

`source_call_uid`

同一通電話及其所有 derived variants：

只能全部在 train，或全部在 validation。

禁止混用。

優先：

`leave-one-call-out validation`

不得把同一通 call 的 derived samples 分到 train/test 兩邊。

---

# 二十二、不要只看 Accuracy

每個 variant / classifier 必須報：

- Echo FP
- EARLY
- Recall
- short recall
- median latency
- P90
- per-call confusion

優先順序：

1. FP = 0
2. EARLY = 0
3. Recall
4. Short recall
5. Latency

---

# 二十三、Experiment 順序

## A0

保存 v5 baseline。

## A1

只新增 `ECHO_SUPPRESSED / ECHO_RETURN` state。

跑：

```bash
python echo_guard_acceptance.py --suite all
```

## A2

加入 asymmetric gain tracking。

再跑完整 harness。

## B

加入 band-wise new-content。

再跑完整 harness。

## C

加入 temporal sequence decision。

再跑完整 harness。

## D

如果仍分不開，才試 shallow classifier。

一次只改一層。

---

# 二十四、Echo-return Permanent Regression Set

把 v5 出現巨大 echo-only excess 的 cases 至少加入：

- `9f7c1815`
- `bdba041a`
- `9aa10006`
- `82b0223c`

Expected：

**NO BARGE**

並標：

`echo_return/burst`

另外永久保留：

- 1.04s false-cut regression
- 3.72s false-cut regression
- v4 EARLY regression

---

# 二十五、Caller Set

所有真正 caller / double-talk cases 獨立列出。

比較 caller onset 前後：

**-1000ms ～ +1500ms**

輸出至少：

- TX
- RX
- gain
- predicted echo
- excess
- band novelty
- mel residual
- explainability
- echo state
- trust state
- final classification
- would-barge

---

# 二十六、Short Barge

這一輪不要先把 short path 當第一目標。

先證明：

**normal caller vs echo-return**

真的能分。

只有：

- Original Real FP = 0
- EARLY = 0
- normal caller recall 明顯恢復

之後，才重新測 80～300ms short interruption。

短插話不准為了追求 100% recall 重新引入 FP。

---

# 二十七、Failure Timeline

每個：

- FP
- FN
- EARLY
- >700ms latency

都必須自動產生 failure bundle。

至少包含 -1000ms ～ +1500ms timeline：

| ms | TX | RX dB | Pred Echo dB | Excess | Sim | Band Novelty | Mel Residual | Delay | Echo State | Gain | Trust | Final State | Would Barge |

並保留：

- RX audio
- TX reference
- metadata
- timeline CSV
- summary

---

# 二十八、Acceptance Gate v6

Original Real 是最高優先。

## Echo-only FP

要求：

**0**

## EARLY

要求：

**0**

## Permanent regressions

全部 PASS。

## Caller / Normal Double-talk Recall

目標：

**≥ 90%**

## Median latency

目標：

**≤ 400ms**

## P90

目標：

**≤ 700ms**

---

# 二十九、Short Interruption Gate

80～300ms short interruption：

只有在 normal caller / echo-return 已可靠分開後才正式評估。

不要為了 short recall 犧牲：

- FP
- EARLY

如果最後產品只能明確限制：

> 播放開始最前 X ms 或極短 80～120ms utterance 不保證 barge-in

可以接受，但必須量化並寫進 operating envelope。

---

# 三十、REAL-DERIVED 的用途

REAL-DERIVED 只能作：

- stress test
- operating-envelope evidence

不要把 derived cases 當成新的真人通話數。

最終產出 operating envelope 至少包含：

- delay range
- SNR
- caller/echo ratio
- minimum caller duration
- early-onset limit
- noise limit
- suppression duration tolerance

---

# 三十一、Runtime Performance

每個有效 variant 都要量：

- avg ms/frame
- p95 ms/frame
- CPU
- RAM
- RTF

目前不接受為了 classifier 大幅增加 realtime cost。

---

# 三十二、禁止 UID-specific Overfit

不得寫：

- 某個 UID 特別 threshold
- 某支錄音特別 bypass
- 某個 regression 專用 hack

任何規則必須跨 call 通用。

---

# 三十三、Stop Condition — 最重要

如果完成以下全部：

1. Echo suppression / return state
2. Asymmetric tracking
3. Band-wise new-content
4. Temporal sequence
5. 必要時 small classifier + strict call-level split

仍出現以下任一情況：

- Echo-only FP 無法降到 0
- caller 與 echo-return feature distribution 高度重疊
- 要清 FP 就必須把 caller recall 壓到不可接受
- leave-one-call-out validation 無法泛化

立即停止 magic threshold 調參。

最終明確下結論：

`AUDIO_ONLY_NOT_SEPARABLE`

不要繼續無限 DSP sweep。

---

# 三十四、若 AUDIO_ONLY_NOT_SEPARABLE，下一步必須比較 Side Information

不要只寫「回半雙工」。

至少分析三條下一步：

## A. TTS Known-content Fingerprint

因為我方知道正在播放的實際 TTS / 固定語句內容，可以利用：

- rolling TTS fingerprint
- log-mel sequence matching
- token / phoneme timing

判斷 RX 是否仍主要是目前 TTS 的遠端殘餘回音。

## B. ASR Partial Semantic Side Information

如果 RX partial transcript 與目前 TTS 文本高度一致：

提高 echo probability。

如果出現新的獨立語意：

提高 caller probability。

只能當 side information，不能單獨決策。

## C. Endpoint / SIP Path

評估是否值得：

- 換 SIP trunk
- 換 endpoint
- 換手機模式
- 測 handset / speakerphone / headset

確認問題是否主要是 far-end acoustic path。

---

# 三十五、Final Report

輸出：

`ACCEPTANCE_REPORT_V6.md`

至少包含：

## 1. Executive Summary

判定只能：

- PASS
- CONDITIONAL PASS
- FAIL
- AUDIO_ONLY_NOT_SEPARABLE

## 2. v3 → v6 比較

列：

| Version | Echo FP | EARLY | Recall | Short Recall | Median | P90 |
|---|---:|---:|---:|---:|---:|---:|

## 3. Echo-return Analysis

量化 remote suppression / return 的 temporal signature。

## 4. Spectral Separability

回答：

band-wise / mel residual 是否真的能把 caller 與 echo-return 分開。

## 5. Temporal Separability

回答：

100～400ms sequence 是否存在穩定、跨 call 可泛化差異。

## 6. Classifier（若有）

必須列：

leave-one-call-out validation。

不准只列 train accuracy。

## 7. Failure Cases

逐一分析所有：

- FP
- FN
- EARLY
- slow latency

## 8. Operating Envelope

清楚說明：

哪些條件可用、哪些條件不可用。

## 9. Final Recommendation

只能選主要方向：

### A
Audio-only EchoGuard 可進 live acceptance。

### B
Audio-only 只能維持 shadow。

### C
Audio-only 不可可靠 barge-in，回到 half-duplex。

### D
必須增加 semantic / TTS fingerprint / endpoint / SIP side information 後再研究。

---

# 三十六、工作模式

自行長跑：

```text
盤點
→ baseline
→ A1
→ harness
→ A2
→ harness
→ B
→ harness
→ C
→ harness
→ 必要時 D
→ leave-one-call-out
→ final report
```

不要每一步停下來問。

只有以下情況才停：

- ground truth 不足
- 需要真人重新撥電話
- 可能碰 production
- 需要付費外部服務
- 需要實體硬體操作

其餘自行完成。

---

# 最終目標

本輪不是繼續找另一個 magic threshold。

真正問題是：

> **遠端 echo suppression → return 的時間／頻譜軌跡，是否與真正 caller onset 存在可泛化差異？**

如果有：建立可靠 Audio-only EchoGuard。

如果沒有：停止 Audio-only DSP 路線，明確標記 `AUDIO_ONLY_NOT_SEPARABLE`，轉向 TTS / ASR semantic side information 或 endpoint / SIP 路線。

開始執行。