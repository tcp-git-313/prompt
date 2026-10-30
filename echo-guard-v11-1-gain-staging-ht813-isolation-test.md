# EchoGuard v11.1 — Gain Staging / HT813 Analog Path Isolation Test

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。
>
> 本輪不是再調 semantic threshold，也不是直接做 production barge-in。v11 已確認 PSTN/HT813 media path 會讓 caller observability 大幅下降，但 v11 PSTN baseline 同時經過 `VOLUME(RX)=8`，因此必須先把 gain staging、clipping、HT813 LEC/NEC/阻抗這些變數拆開，確認 analog path 到底有多少是「原生 hybrid echo」，多少是後端增益把問題放大。

---

## 一、目前已知基線

v11 結果：

- PSTN/HT813 caller transcript availability = 2/10
- PSTN coverage median = 0.125
- PSTN CER = 0.875
- digital control availability = 11/11
- digital coverage median = 1.00
- digital CER = 0.00
- echo-dominant windows = 3 → 0
- ASR latency 正常：P50 約 141ms / P90 196ms / P95 223ms
- production barge-in 仍 OFF

目前 ASR tap：

```text
HT813 / PSTN
→ SIP/RTP
→ Asterisk
→ VOLUME(RX)=8
→ AudioSocket :9090
→ ASR
```

因此這一輪核心問題是：

> `VOLUME(RX)=8`、TTS TX level、HT813 FXO gain、LEC/NEC、AC termination impedance，分別對 clipping、echo energy 與 caller ASR observability 造成多少影響？

---

## 二、本輪硬性規則

禁止：

- 改 semantic threshold
- 改 persistence
- 改 500/2000ms window
- 改 polling interval
- 換 ASR
- 改 caller classifier
- 改 production application code
- 直接開正式 barge-in
- deploy production
- force push
- `git reset --hard`

允許：

- 建立隔離 PoC / test dialplan / test context
- 暫時套用測試用 RX/TX gain
- 暫時改 HT813 測試設定
- 收集真人 double-talk 測試
- 離線分析既有 v10.1 / v11 recordings
- 測試結束後完整 restore

所有 production 9090 正式行為保持不變。

---

## 三、第一階段：先確認 `VOLUME(RX)=8` 到底做了什麼

先純讀取確認目前 Asterisk dialplan / channel 設定中：

- 哪裡設定 `VOLUME(RX)=8`
- 是否所有 PSTN calls 都經過
- 是否發生在 AudioSocket 之前
- 是否還有其他 RX gain
- 是否還有 TX gain
- 是否存在多重 gain 疊加

輸出：

```text
results/full_duplex_v11_1/GAIN_CHAIN_AUDIT.md
results/full_duplex_v11_1/gain_chain.json
```

必須畫出：

```text
HT813 RX
→ Asterisk RX gain
→ VOLUME(RX)
→ other gain/normalization
→ AudioSocket
→ app RX
→ ASR
```

以及 TX：

```text
TTS PCM
→ app TX gain
→ AudioSocket
→ Asterisk TX gain
→ HT813
→ PSTN
```

---

## 四、建立不改 production 的 Test Matrix

優先使用隔離 PoC / test context。

至少測以下 4 組：

| Case | Asterisk RX gain | TTS TX level | HT813 config |
|---|---:|---:|---|
| G0 | current (`RX=8`) | current | current |
| G1 | `RX=0` | current | current |
| G2 | `RX=0` | -6 dB | current |
| G3 | `RX=0` | -9 dB | current |

如果系統中 `RX=0` 的語義不是 unity gain，請先確認實際 behavior，不能靠猜。

每組使用相同：

- caller phrase
- AI TTS phrase
- ASR
- test timing
- playback node
- protocol

每組至少 3 個 Normal Caller double-talk events；最好共用同一組真人語料與相同播放內容。

Short Caller 本輪不是主指標。

---

## 五、每組必須量的 Audio 指標

對 RX gain 前後都記：

- peak dBFS
- RMS dBFS
- clipped sample count
- clipped sample %
- crest factor
- echo-only RMS
- caller-only RMS
- double-talk RMS
- caller-to-echo relative level proxy

特別要有：

```text
pre_gain_peak_dbfs
post_gain_peak_dbfs
pre_gain_clipping_pct
post_gain_clipping_pct
```

如果無法取得 pre-gain raw audio，必須明確寫：

`PRE_GAIN_CAPTURE_UNAVAILABLE`

不要用推測代替。

---

## 六、每組必須量的 ASR 指標

每個 Normal interruption：

- caller transcript available
- expected caller text
- observed caller text
- caller char coverage
- CER / normalized edit distance
- echo-dominant transcript yes/no
- first caller-bearing token latency
- first usable caller text latency
- ASR inference latency

每組最後至少列：

| Metric | G0 | G1 | G2 | G3 |
|---|---:|---:|---:|---:|
| caller availability | | | | |
| coverage median | | | | |
| CER median | | | | |
| clipped sample % | | | | |
| echo-only RMS | | | | |
| double-talk RMS | | | | |
| first caller token median | | | | |

---

## 七、第一個判斷 Gate：Gain Staging 是否是主要放大器

如果從 G0 → G1 就出現明顯改善，例如：

- clipping 大幅下降
- caller availability 顯著上升
- coverage 明顯上升
- CER 明顯下降

標記：

`RX_GAIN_IS_MAJOR_CONTAMINATION_AMPLIFIER`

如果 G1 幾乎沒有改善：

標記：

`RX_GAIN_NOT_PRIMARY`

如果 G2 / G3 明顯優於 G1：

標記：

`TX_LEVEL_MATERIAL_TO_ECHO`

不要先定義「顯著」為固定百分比；請把實際 delta 完整列出，再依 evidence 判斷。

---

## 八、第二階段：HT813 LEC / NEC 極端組合

只有完成 G0～G3 後才做。

固定使用目前表現最佳的 gain staging。

測：

| Case | LEC | NEC |
|---|---|---|
| E0 | current | current |
| E1 | ON | ON |
| E2 | OFF | ON |
| E3 | ON | OFF |
| E4 | OFF | OFF |

如果 current 已等於某組，可以避免重複。

每組至少 2～3 個 Normal Caller double-talk events。

不要預設 LEC ON 一定好，也不要預設 OFF 一定好。

唯一評分：

- caller transcript availability
- caller coverage
- CER
- echo level
- clipping
- double-talk caller preservation

---

## 九、第三階段：AC Termination Impedance

只有在前兩階段仍不足時才做。

固定：

- 最佳 gain staging
- 最佳 LEC/NEC 組合

比較：

- current impedance
- 600R
- 若 HT813 有可選 complex profile，加入最合理的 1～2 個候選

不要因為規範提到 600Ω 就假設一定最佳。

每個 impedance 至少測：

- echo-only RMS
- caller coverage
- CER
- caller transcript availability

目標：找現場這條實際 PSTN line 的最佳 hybrid balance。

---

## 十、真人測試 Protocol

沿用 v10.1 corrected protocol，不再使用模糊 A/B/C 流程。

每通固定：

```text
接通
→ 真人「喂，你好」
→ AI 開場
→ A Normal interruption
→ AI 播完
→ 固定 Bridge（非測試事件）
→ 下一個 AI turn
→ B Normal interruption
→ B 後保留至少 5 秒
→ 正常 closing
→ 掛斷
```

固定 Normal phrases 可從以下挑選：

- `等一下，我現在不方便。`
- `你哪裡打來的？`
- `不用了，謝謝。`
- `我現在在忙，晚點再說。`
- `這個是什麼服務？`
- `你剛剛說什麼？`
- `我沒有要賣房子。`

Bridge 固定：

`好，我想先了解一下，你們主要是在做什麼？`

Bridge 不進 denominator。

---

## 十一、避免過多真人通話

這輪不要一開始就重打大量電話。

順序：

1. 先用既有 v10.1 / v11 evidence 做能做的離線 clipping / level audit。
2. 只在必須取得新 gain/LEC/impedance 組合時才真人重打。
3. 每個 matrix cell 先 2～3 個 Normal events。
4. 只有看到改善趨勢才補到 5～10 個 events。

---

## 十二、如果 G1/G2/G3 已經救回 caller observability

若某組達到例如：

- caller transcript availability ≥80%
- coverage median 明顯上升
- CER 明顯下降
- clipping 接近 0

不要立刻改 production。

先標：

`ANALOG_PATH_RECOVERABLE_BY_GAIN_STAGING`

然後再做一輪小型真人 validation。

正式上線仍需另外 approval。

---

## 十三、如果 Gain / LEC / Impedance 都無法救回

若最佳組合仍接近：

- availability ≈ 2/10
- coverage ≈ 0.125
- CER ≈ 0.875

則標：

`ANALOG_ECHO_PERSISTS_AFTER_FRONTEND_TUNING`

下一步才進：

1. Offline WebRTC AEC3
2. Offline DTLN-AEC
3. Live SIP Control

不要再回頭調 semantic。

---

## 十四、AEC3 / Neural AEC 本輪只做準備，不直接上 runtime

本輪可以盤點：

- actual-sent TTS PCM
- mixed RX PCM
- exact timestamps
- delay 410～423ms
- sample rate / frame size

並準備下一輪 offline AEC harness。

但除非 gain/HT813 tuning 已經確認不足，不要先把 AEC3 / DTLN 接進 9090。

---

## 十五、Safety / Restore

任何測試設定變更都必須：

- 先 snapshot
- 0 active calls
- 測試後 restore
- restore 後重新驗證 9090
- barge-in 仍 OFF

HT813 任何 LEC/NEC/impedance/gain 修改都要記錄：

- before
- test value
- after restore

如果 restore 失敗：

`RESTORE_FAILED`

停止其他實驗。

---

## 十六、輸出

統一放：

```text
results/full_duplex_v11_1/
```

至少包含：

- `GAIN_CHAIN_AUDIT.md`
- `gain_chain.json`
- `gain_matrix.csv`
- `audio_level_metrics.csv`
- `caller_observability.csv`
- `lec_nec_matrix.csv`
- `impedance_matrix.csv`
- `clipping_report.json`
- `FINAL_REPORT_V11_1.md`
- `restore_report.json`

---

## 十七、最終必答

最後一定回答：

1. `VOLUME(RX)=8` 是否造成 clipping / observability 惡化？
2. G0→G1 clipping 改善多少？
3. caller availability / coverage / CER 改善多少？
4. TX -6dB / -9dB 是否進一步改善？
5. 最佳 gain staging 是哪組？
6. LEC / NEC 哪組最好？
7. impedance 哪組最好？
8. 最佳組合下 caller availability 是多少？
9. 最佳組合下 coverage median / CER 是多少？
10. echo-only RMS 改善多少？
11. 是否仍有 PURE_ECHO dominance？
12. 是否可判 `ANALOG_PATH_RECOVERABLE_BY_GAIN_STAGING`？
13. 或必須判 `ANALOG_ECHO_PERSISTS_AFTER_FRONTEND_TUNING`？
14. production 9090 是否完全未改？
15. cleanup / restore 是否 PASS？
16. barge-in 是否仍 OFF？
17. 下一步應該是 Live SIP、AEC3、DTLN-AEC，還是先做小型 gain validation？

---

## 十八、本輪唯一核心問題

> **目前 PSTN caller observability 崩潰，有多少是 HT813/PSTN 類比 echo 本身，有多少是 `VOLUME(RX)=8`、TX 音量、LEC/NEC 與 impedance 把它進一步放大？**

不要調 semantic。
不要換 ASR。
不要碰 production barge-in。
先把 front-end gain / echo path 榨乾再決定要不要進 AEC3 / DTLN。

開始執行。