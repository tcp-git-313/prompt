# EchoGuard v11.3 — TX -6 dB Sweet Spot Validation

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。
>
> 本輪不是重新做完整 acceptance，也不是再跑大量真人通話。v11.2 已經以 1 通真實 PSTN/HT813 校準證明 `RX=0 + TX=-9 dB` 能大幅降低 echo、消除 clipping，並恢復 caller observability。本輪只回答一個工程問題：**TX -6 dB 是否也足夠穩定，從而在保留更高客戶端聽感音量的前提下，維持 full-duplex caller observability？**

---

## 一、目前已知基線

### Production / safety

- production 9090 不可修改
- `[saas-dialogue]` 不可修改
- barge-in 必須維持 OFF
- HT813 不可修改
- 不碰 LEC / NEC / impedance
- 不 deploy production
- 不 push production changes

### 歷史 baseline

舊 PSTN / HT813：

- `VOLUME(RX)=8`
- clipping median 約 4.70%
- double-talk clipping 最高約 12.85%
- post-gain peak 常到 0.0 dBFS
- echo-only RMS 約 -9.2 dBFS
- caller-only RMS 約 -19.2 dBFS
- echo 約比 caller 高 10 dB
- caller transcript availability 約 2/10
- caller coverage 約 0.14
- CER 約 0.875

### v11.2 真實 TX -9 dB 校準

已完成 1 通，2 個 Normal interruptions。

設定：

```text
RX = 0
TX = -9 dB
```

結果：

- actual TX RMS（nodes 100/200/117）約：-23.85 / -28.14 / -23.43 dBFS
- TX peak 約：-9.87 / -13.49 / -9.85 dBFS
- echo-only RX RMS = -34.8 dBFS（RX=0）
- 換算同 RX=8 baseline 比較，echo 額外下降約 7.6 dB
- clipping = 0.0%
- peak = -9.89 dBFS
- caller availability = 2/2
- caller coverage = 0.778 / 1.000，median 0.889
- CER median = 0.111
- ASR：
  - A：`等一下啊我現在不`
  - B：`你哪裡打來的啊`

Verdict：

`LIVE_TX_REDUCTION_EFFECT_CONFIRMED`

---

## 二、本輪唯一目的

驗證：

```text
RX = 0
TX = -6 dB
```

是否仍能達到足夠好的 caller observability。

若 -6 dB 足夠穩定，優先候選 production gain staging 將是：

```text
RX = unity / 不再使用 VOLUME(RX)=8
TX = -6 dB
```

若 -6 dB 明顯比 -9 dB 差，則保留：

```text
RX = unity
TX = -9 dB
```

本輪**不測 -3 dB**。

只有當 -6 dB 已經明顯 PASS，報告中才可以「建議下一輪是否值得測 -3 dB」，但不得自動執行。

---

## 三、硬性禁止事項

禁止：

- 改 semantic threshold
- 改 persistence
- 改 semantic window
- 改 polling interval
- 換 ASR
- 改 caller classifier
- 改 v10/v10.1/v11/v11.2 evidence
- 改 HT813 LEC
- 改 HT813 NEC
- 改 impedance
- 上 AEC3
- 上 DTLN-AEC
- 開 production barge-in
- 修改 production 9090 behavior
- 修改 `[saas-dialogue]`
- deploy production
- force push
- `git reset --hard`
- 自動加打第 2 通
- 自動測 TX -3 dB

---

## 四、先做 preflight

在任何真人通話前先確認：

1. production 9090 正常 listening
2. barge-in = OFF
3. 目前 production `SAAS_PLAY_GAIN` 仍為 0 dB / 未設定
4. `[saas-dialogue]` hash 記錄
5. `extensions.conf` hash 記錄
6. test port 19091 沒有殘留 process
7. 沒有 active call
8. 上一輪 test context 已不存在

將 preflight 結果寫入：

```text
results/full_duplex_v11_3/preflight.json
```

若任何項目異常：

`PREFLIGHT_FAILED`

停止，不得硬做。

---

## 五、建立隔離 test context

只建立隔離測試路徑，例如：

```text
[saas-v113]
RX = 0 / unity
AudioSocket → 19091
PoC PLAY_GAIN = -6 dB
```

要求：

- 不修改 `[saas-dialogue]`
- 不改 production 9090
- 不改 production env
- 不改 HT813
- 測試完完整移除

test context 建立前後都要記：

- `extensions.conf` hash
- `[saas-dialogue]` hash

---

## 六、本輪只打一通

只允許：

```text
1 通真人電話
2 個 Normal interruption events
```

不得因為資料看起來不夠漂亮就自動再打一通。

除非這一通技術性無效，例如：

- caller onset 標記錯誤
- 錄音損壞
- ASR server down
- test context 未生效
- post-onset 長度不足

才可以標：

`TEST_INVALID_TECHNICAL`

並停下回報原因，**不要自行重打**。

---

## 七、真人 Protocol

沿用 v10.1 corrected protocol，但縮成最小校準。

流程：

```text
接通
→ caller：「喂，你好」
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

建議 A/B：

A：

`等一下，我現在不方便。`

B：

`你哪裡打來的？`

若使用者自然講成近似句，不必判 invalid，但 expected text 要記實際 intended phrase。

---

## 八、必須收集的 raw evidence

本通至少保存：

- raw RX PCM/WAV
- actual-sent TTS PCM
- node timeline
- playback start/end timestamps
- caller onset markers
- ASR transcript per window
- final ASR transcript
- test env snapshot
- restore snapshot

放：

```text
results/full_duplex_v11_3/live_tx6/
```

---

## 九、Audio 指標

本通必須計算：

### TX

每個 playback node：

- TX RMS dBFS
- TX peak dBFS
- duration

確認 `-6 dB` 真的套用到 actual sent PCM，而不是只改設定文字。

如果 actual PCM 與預期不符：

`TX_GAIN_NOT_ACTUALLY_APPLIED`

停止判讀。

### RX

計算：

- full-call peak dBFS
- full-call RMS dBFS
- clipped sample count
- clipped sample %
- echo-only RX RMS
- caller-only RX RMS（若有乾淨 listening-gap segment）
- double-talk RMS
- caller-to-echo relative level proxy

---

## 十、ASR / caller observability 指標

每個 A/B event：

- expected caller text
- observed ASR text
- caller transcript available yes/no
- caller char coverage
- CER / normalized edit distance
- echo-dominant yes/no
- first caller-bearing token latency
- first usable caller text latency
- ASR inference latency

本通彙總：

- availability x/2
- coverage median
- CER median
- echo-dominant count

---

## 十一、直接對比 TX -9 dB 基線

建立：

```text
results/full_duplex_v11_3/tx6_vs_tx9.csv
```

至少包含：

| Metric | TX -9 dB baseline | TX -6 dB new | Delta |
|---|---:|---:|---:|
| caller availability | 2/2 | | |
| coverage median | 0.889 | | |
| CER median | 0.111 | | |
| clipping % | 0.0% | | |
| echo-only RX RMS | -34.8 dBFS | | |
| RX peak | -9.89 dBFS | | |
| first usable latency median | baseline value if available | | |

注意：

不同 call 的 speaker delivery 不完全相同，所以不可把單通差異當作精密統計；本輪目標只是找 engineering sweet spot。

---

## 十二、PASS / FAIL 判斷

### `TX6_SWEET_SPOT_CONFIRMED`

優先條件：

- caller availability = 2/2
- coverage median >= 0.80
- CER median <= 0.20
- clipping 接近 0
- 無明顯 echo-dominant event
- echo 沒有回到舊 baseline 的壓倒性狀態

如果符合，判：

`TX6_SWEET_SPOT_CONFIRMED`

代表 -6 dB 已足以作為優先 gain-staging 候選。

### `TX6_USABLE_BUT_TX9_SAFER`

例如：

- 2/2 available
- coverage 0.65～0.80
- CER 有改善但明顯差於 -9
- echo 明顯比 -9 強，但 caller 仍可讀

則判：

`TX6_USABLE_BUT_TX9_SAFER`

### `TX6_NOT_SUFFICIENT`

若出現任何明顯退化，例如：

- availability <= 1/2
- coverage median < 0.65
- CER 明顯惡化
- echo-dominant 再出現
- clipping 明顯回來

判：

`TX6_NOT_SUFFICIENT`

此時優先保留 TX -9 dB。

---

## 十三、不要被單一指標誤導

不可只因為 echo RMS 比 -9 高 3 dB 就判 FAIL。

真正優先順序：

1. caller availability
2. caller coverage
3. CER
4. clipping
5. echo dominance
6. echo RMS

目標是「客戶能不能被可靠辨識」，不是追求最低 echo 數值。

---

## 十四、如果 -6 dB PASS

本輪到此停止。

不要自動測 -3 dB。

報告可以提出：

`OPTIONAL_NEXT: TX -3 dB single-call calibration`

但是否測 -3 要留給使用者決定。

也不要直接改 production。

下一輪應該規劃：

```text
Candidate production gain staging
RX = unity
TX = -6 dB
→ shadow-only acceptance
→ barge-in still OFF
```

---

## 十五、如果 -6 dB FAIL

本輪也到此停止。

不要自動改 HT813。

優先候選維持：

```text
RX = unity
TX = -9 dB
```

下一輪才考慮：

1. 以 RX0/TX-9 做 shadow acceptance
2. 若仍有 residual echo 問題，再考慮 LEC/NEC
3. impedance
4. 最後才 AEC3 / DTLN

---

## 十六、Restore

測試結束後必須：

1. 移除 `[saas-v113]`
2. 關閉 19091 PoC
3. 確認 production `SAAS_PLAY_GAIN` 未被改成 -6
4. 確認 production 9090 正常
5. 確認 barge-in OFF
6. 比對 `extensions.conf` hash
7. 比對 `[saas-dialogue]` hash

若任何 restore 失敗：

`RESTORE_FAILED`

並優先處理 restore，不進其他實驗。

---

## 十七、輸出

全部放：

```text
results/full_duplex_v11_3/
```

至少：

- `preflight.json`
- `live_tx6_metrics.json`
- `live_tx6_events.csv`
- `tx6_vs_tx9.csv`
- `restore_report.json`
- `FINAL_REPORT_V11_3.md`
- `live_tx6/`

---

## 十八、最終必答

最後只根據 evidence 回答：

1. TX -6 dB actual sent PCM 是否真的生效？
2. TX RMS / peak 實際是多少？
3. clipping 是多少？
4. echo-only RX RMS 是多少？
5. caller availability 是多少？
6. coverage A/B 與 median 是多少？
7. CER A/B 與 median 是多少？
8. ASR transcript A/B 是什麼？
9. 是否有 echo-dominant event？
10. TX -6 與 TX -9 哪個 caller observability 更好？
11. -6 dB 是否已達 production-candidate 級別？
12. 最終 verdict 是：
   - `TX6_SWEET_SPOT_CONFIRMED`
   - `TX6_USABLE_BUT_TX9_SAFER`
   - `TX6_NOT_SUFFICIENT`
   - `TEST_INVALID_TECHNICAL`
13. 是否值得下一輪再測 -3 dB？只能建議，不可自動執行。
14. production 9090 是否完全未改？
15. `[saas-dialogue]` 是否完全未改？
16. barge-in 是否仍 OFF？
17. restore 是否 PASS？

---

## 十九、本輪停止條件

本輪只有 1 通。

無論結果 PASS 或 FAIL：

**測完即停。**

不要自動：

- 打第二通
- 測 -3 dB
- 改 HT813
- 做 AEC3
- 做 DTLN
- 開 barge-in
- 改 production

---

## 二十、本輪核心問題

> **TX -9 dB 已被真實 PSTN/HT813 校準證明有效。現在只需要知道：TX -6 dB 是否也能保留同等級 caller observability，讓客戶端 AI 聲音比 -9 dB 更大 3 dB，同時避免 echo 再次淹沒 caller。**

開始執行。