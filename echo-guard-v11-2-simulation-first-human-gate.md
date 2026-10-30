# EchoGuard v11.2 — Simulation-First / Human-Only-When-Required

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。
>
> 本輪原則非常明確：**優先榨乾既有真人資料與離線模擬，只有在某個關鍵結論無法由現有 evidence 證明，而且必須改變真實 media path 才能知道答案時，才停在 HUMAN GATE 叫真人測。**
>
> 不要因為「真人比較真」就預設重打電話。先證明真的非打不可。

---

## 一、目前已知狀態

v11.1 Phase 1 / simulation 已完成。

已知：

- production 9090 未改
- barge-in OFF
- `VOLUME(RX)=8` 是唯一軟體 RX gain
- `VOLUME(RX)=8` 位於 AudioSocket 前
- `RX8_CLIPPING_CONFIRMED`
- 每通 clipping 中位數約 4.70%
- double-talk clipping 最高約 12.85%
- post-gain peak 每通都到 0.0 dBFS
- echo-only RMS 約 -9.2 dBFS
- caller-only RMS 約 -19.2 dBFS
- echo 約比 caller 大 10 dB
- `PRE_GAIN_CAPTURE_UNAVAILABLE`

離線真實片段混音模擬：

| Cell | available | coverage median | CER median | echo-dominant |
|---|---:|---:|---:|---:|
| G0 RX8 / TX0 | 3/6 | 0.50 | 0.889 | 3 |
| G1 RX0 / TX0 | 3/6 | 0.50 | 0.889 | 3 |
| G2 RX0 / TX-6 | 6/6 | 0.833 | 0.222 | 0 |
| G3 RX0 / TX-9 | 6/6 | 1.000 | 0.167 | 0 |

目前 provisional evidence：

- `RX_GAIN_NOT_PRIMARY`（simulation only）
- `TX_LEVEL_MATERIAL_TO_ECHO`（simulation supported, not yet live-confirmed）

重要：

> 這些模擬證明「如果實際 echo 真能隨 TX 下降而下降，caller observability 會明顯恢復」。
>
> 但它還沒有證明真實 HT813/PSTN 路徑在 TX -6 / -9 dB 下會真的產生相同 echo reduction。

---

## 二、本輪最高原則

### 1. Simulation first

先使用：

- v10 recordings
- v10.1 recordings
- v11 PSTN baseline
- v11 digital control
- v11.1 audio metrics
- caller-only clips
- echo-only clips
- actual-sent TTS PCM
- node/playback timeline
- existing ASR replay cache

把能離線證明的全部做完。

### 2. 不得預設真人測試

除非符合後面的 HUMAN TEST NECESSITY GATE，否則：

- 不要求使用者打電話
- 不要求重跑 G1/G2/G3
- 不建立真人 acceptance 流程
- 不重做 v10.1 protocol

### 3. 真人測試必須是「最少量校準」

如果真的需要真人：

- 只測無法離線得到的 media-path response
- 優先 1 通或 2 通
- 不因為 sample 少就機械式要求 5～10 通
- 先看 effect size，再決定是否值得補樣本

---

## 三、本輪禁止事項

禁止：

- 改 semantic threshold
- 改 persistence
- 改 500/2000ms window
- 改 polling interval
- 換 ASR
- 改 caller classifier
- 改 production 9090 application behavior
- 開 production barge-in
- deploy production
- force push
- `git reset --hard`
- 為了方便驗證而覆蓋 v10/v10.1/v11/v11.1 evidence

本輪不是 tuning。

---

## 四、Phase A — 歷史真人資料完整再分析

先建立一份所有真人 PSTN evidence 的統一 manifest。

至少納入：

- v10
- v10.1
- v11 PSTN baseline
- v11.1 Phase 1 metrics

每通至少整理：

- call_id
- date/time
- RX gain
- TX gain
- HT813 known config
- caller event count
- echo-only segments
- caller-only/listening-gap segments
- double-talk segments
- clipping %
- peak dBFS
- echo RMS
- caller RMS
- caller/echo relative level
- caller transcript availability
- caller coverage
- CER
- TTS echo coverage
- ASR latency

輸出：

```text
results/full_duplex_v11_2/
  historical_real_manifest.csv
  historical_real_summary.json
```

目標：

> 先確認歷史真人資料是否已經包含足夠的自然變異，能推估 TX level 與 echo/caller observability 的關係。

---

## 五、Phase B — 找歷史資料中的「自然 TX 差異」

檢查既有所有 call 是否真的完全相同 TX level。

搜尋：

- 不同 `SAAS_PLAY_GAIN`
- 不同 WAV/TTS source amplitude
- 不同 fixed phrase loudness
- 不同 node actual-sent PCM RMS
- 不同 Asterisk TX gain
- 不同 HT813 TX gain（若 log 有）

將 actual sent PCM 的每個 playback node 實際量：

- RMS dBFS
- peak dBFS
- LUFS-like proxy（若合理）

再看 caller observability 是否隨實際 TX level 有關。

建立：

```text
historical_tx_vs_echo.csv
historical_tx_vs_observability.csv
```

至少分析：

```text
actual TTS TX RMS
vs
echo-only RX RMS
```

以及：

```text
actual TTS TX RMS
vs
caller coverage / CER
```

如果歷史資料本身已有足夠 TX level spread，優先用它做 quasi-experimental evidence。

---

## 六、Phase C — 建立 Echo Transfer Function Empirical Model

不要只做單一 -6/-9 等比例縮放。

用所有已有真人 echo-only segment 建立 empirical mapping：

```text
TTS actual-sent level
→ observed PSTN echo level
```

至少嘗試：

1. linear level model
2. piecewise-linear model
3. robust regression
4. saturation/clipping-aware model

要回答：

- echo 是否真的近似跟 TX level 一起下降？
- 是否有 saturation region？
- 是否存在某個 TX level 以下 echo 明顯變線性？
- 現有 G0 是否可能位於 clipping / saturation 區？

輸出：

```text
echo_transfer_model.json
ECHO_TRANSFER_ANALYSIS.md
```

如果 evidence 不足，不要假裝 fit 成功。

標記：

`ECHO_TRANSFER_MODEL_INSUFFICIENT`

---

## 七、Phase D — Monte Carlo / Resampling 模擬

不要只做 6 個固定混音 case。

使用：

- 多個 caller-only 真人片段
- 多個 echo-only 真人片段
- 多個 TTS node
- 多個 caller phrase

建立 resampling simulation。

TX 條件至少：

```text
0 dB
-3 dB
-6 dB
-9 dB
-12 dB
```

每個 level 至少產生足夠組合，例如：

- 30+ mixed events（若素材足夠）

但不得假造新的真人語音，只能重新組合已有真人片段。

每個 level 報：

- availability
- coverage median
- coverage P25/P75
- CER median
- echo-dominant rate
- clipping risk proxy

輸出：

```text
simulation_sweep.csv
simulation_distribution.json
SIMULATION_REPORT.md
```

---

## 八、Phase E — 模擬可信度檢查

模擬不能自己證明自己。

所以要用歷史資料做 backtest：

1. 用某些 historical echo/caller clips 建模
2. 用其他未參與建模的 clips 驗證
3. 看模型是否能預測：
   - echo RMS
   - clipping
   - ASR coverage
   - CER trend

如果 backtest 差：

標：

`SIMULATION_NOT_PREDICTIVE_ENOUGH`

如果 backtest 合理：

標：

`SIMULATION_PREDICTIVE_FOR_LEVEL_EFFECTS`

注意：

> 即使 simulation predictive，也只能證明 level effect，不能直接證明 HT813 在實際 TX -9 dB 時其 echo path 一定維持相同行為。

---

## 九、HUMAN TEST NECESSITY GATE

只有以下條件全部成立，才允許叫真人測：

### 必須同時滿足

1. 離線 evidence 已清楚指出一個高價值候選，例如 TX -6 或 -9 dB。
2. 這個候選的關鍵未知數是「真實 HT813/PSTN 在該 TX level 下 echo 會不會真的下降」。
3. 這個未知數不能從既有歷史 recordings、metadata、natural TX variation 或 model/backtest 推導。
4. 不做真人測就無法決定下一個工程方向。
5. 測試可以用最少 1～2 通完成，而不是要求重新跑整個 acceptance。

若不滿足全部條件：

**不要叫真人。**

繼續離線分析或直接下 `INCONCLUSIVE_WITHOUT_NEW_MEDIA_PATH_SAMPLE`。

---

## 十、如果真的需要 HUMAN GATE

只在真正需要時停。

而且先回報：

```text
HUMAN TEST REQUIRED

Why:
<哪一個未知數無法由現有資料回答>

Why existing data is insufficient:
<明確證據>

Minimum new test:
<只需要幾通、幾個事件>

Decision unlocked by this test:
<測完會決定什麼>
```

不要只說「真人比較準」。

---

## 十一、真人測試優先級

如果最後真的必須真人測，優先順序：

### Test 1 — TX -9 dB only

因為 simulation G3 最強：

- 6/6 available
- coverage 1.0
- CER 0.167
- echo-dominant 0

所以先測：

```text
RX = 0
TX = -9 dB
```

只需：

- 1 通
- 2 個 Normal interruption

量：

- actual sent TX RMS
- observed echo-only RMS
- caller transcript availability
- coverage
- CER
- clipping

### Test 2 — TX -6 dB only if needed

只有以下情況才補：

- -9 dB 成功，但想找較高可聽音量的最低有效衰減
- -9 dB 結果與 simulation 差異很大，需要比較 effect curve

否則不要自動要求 G2。

### G1 RX0/TX0

預設不需要真人重打。

只有要分辨 RX gain clipping 因果時，而且既有 evidence 無法回答，才測。

---

## 十二、真人 Protocol（若真的進 HUMAN GATE）

沿用 v10.1 corrected protocol，但縮短成最少校準。

每通：

```text
接通
→「喂，你好」
→ AI 開場
→ A Normal interruption
→ AI 播完
→ 固定 Bridge
→ 下一個 AI turn
→ B Normal interruption
→ B 後至少保留 5 秒
→ closing
→ 掛斷
```

固定 Bridge：

`好，我想先了解一下，你們主要是在做什麼？`

不得重新引入模糊 A/B/C protocol。

---

## 十三、真人測試後的停止規則

若 1 通 / 2 events 已經出現非常明顯 effect，例如：

```text
availability 2/2
coverage >0.8
CER 大幅下降
echo RMS 明顯下降
clipping 接近 0
```

先停止。

標：

`LIVE_TX_REDUCTION_EFFECT_CONFIRMED`

不要機械式繼續打 5 通。

只有需要 estimate reliability 才補樣本。

反之，如果 -9 dB 仍 0/2 且 echo RMS 幾乎不變：

標：

`LIVE_TX_REDUCTION_EFFECT_NOT_CONFIRMED`

也停止，不要馬上連打更多。

---

## 十四、HT813 LEC / NEC / Impedance 何時才值得碰

只有在真人或足夠 evidence 證明：

- TX level reduction 不足

才進下一層：

```text
LEC / NEC
→ impedance
```

而且必須一次只改一類變數。

不要在本輪預先要求使用者登入 HT813。

---

## 十五、AEC3 / DTLN 何時才值得進

只有以下任一成立：

1. TX -9 dB 實際無法把 caller observability 拉回來
2. 為了讓 echo 足夠低，TX 必須低到電話另一端不可接受
3. LEC/NEC/impedance 仍不足

才進：

- offline WebRTC AEC3
- offline DTLN-AEC

不要直接接 production runtime。

---

## 十六、最終 Verdict

本輪可用 verdict：

### `SIMULATION_SUFFICIENT_NO_HUMAN_TEST_NEEDED`

現有 evidence 已足以決定下一步，不需要真人。

### `HUMAN_CALIBRATION_REQUIRED`

只有一個 media-path unknown 無法離線回答，需要最小真人校準。

### `LIVE_TX_REDUCTION_EFFECT_CONFIRMED`

最小真人校準證實降低 TX 可實際降低 echo 並恢復 caller observability。

### `LIVE_TX_REDUCTION_EFFECT_NOT_CONFIRMED`

真實 HT813/PSTN 並未重現模擬改善。

### `SIMULATION_NOT_PREDICTIVE_ENOUGH`

離線模型不能可靠預測真實 media behavior。

### `INCONCLUSIVE_WITHOUT_NEW_MEDIA_PATH_SAMPLE`

現有 evidence 不足，但本輪尚未批准/執行新真人測試。

---

## 十七、輸出

放：

```text
results/full_duplex_v11_2/
```

至少包含：

- `historical_real_manifest.csv`
- `historical_real_summary.json`
- `historical_tx_vs_echo.csv`
- `historical_tx_vs_observability.csv`
- `echo_transfer_model.json`
- `ECHO_TRANSFER_ANALYSIS.md`
- `simulation_sweep.csv`
- `simulation_distribution.json`
- `SIMULATION_REPORT.md`
- `human_test_necessity.json`
- `FINAL_REPORT_V11_2.md`

若完全沒有真人測試：

明確寫：

`NO_NEW_HUMAN_CALLS_REQUIRED`

---

## 十八、最終必答

最後一定回答：

1. 現有真人資料總共有多少通、多少 usable events？
2. 歷史資料中是否存在自然 TX level variation？
3. TTS TX level 與 echo RMS 的 empirical 關係是什麼？
4. 是否看到 saturation / clipping region？
5. 模擬在 backtest 上是否 predictive？
6. -3 / -6 / -9 / -12 dB 的 simulated availability / coverage / CER 各是多少？
7. 哪個 TX level 是最佳候選？
8. 是否真的還需要真人？
9. 若需要，為什麼現有資料無法回答？
10. 最少需要幾通、幾個事件？
11. 是否可以先只測 TX -9 dB？
12. G1 是否真的還有必要真人重打？
13. production 9090 是否完全未改？
14. barge-in 是否仍 OFF？
15. 下一步應該是：
   - 不用真人直接下結論
   - 最小 TX -9 校準
   - LEC/NEC/impedance
   - AEC3/DTLN
   - Live SIP

---

## 十九、最重要規則

> **不要把「真人測試」當預設流程。先證明現有真人資料真的無法回答，再叫真人。**

> **新的真人通話只用來回答「改變真實 media path 後會發生什麼」這類離線資料不可能知道的問題。**

> **如果真的需要真人，先做最少量、最高資訊價值的那一組，預設從 TX -9 dB 的 1 通 / 2 events 開始。**

開始執行。