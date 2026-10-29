# EchoGuard v11 — Caller Observability / Media-Path Separation

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。
>
> 本輪不是再調 EchoGuard threshold、不是再調 semantic novelty、不是再換 polling/window，也不是直接做 production barge-in。v10.1 已用乾淨 protocol 證實 frozen semantic route 在目前 HT813/PSTN mixed RX path 下失敗。本輪唯一目標是定位：**caller 到底在哪一層失去可觀測性**。

---

## 一、已確認的前提

v10.1 已完成 Protocol-Corrected Live Acceptance，結果：

- Protocol：`PROTOCOL_PASS`
- Semantic：`SEMANTIC_NOT_SEPARABLE`
- 5 通真實電話
- Normal eligible = 9
- Short eligible = 1
- Normal recall = 0%（0/9）
- raw CALLER FP = 1 / 290 echo windows
- confirmed PURE_ECHO_FP = 1
- EARLY = 0
- turn-boundary carry-over = NOT_OBSERVED
- bridge contamination = 0
- cleanup / restore = PASS
- production barge-in 仍 OFF

v10 舊資料重新分類後：

- 舊 2 個 echo FP = 2/2 PURE_ECHO_FP

ASR latency 已知：

- P50 約 141ms
- P90 約 196ms
- P95 約 223ms

所以目前不能再把主因歸咎於：

- protocol 污染
- turn-boundary carry-over
- bridge 污染
- ASR 太慢
- threshold 沒調好

目前最重要觀察：

> 在 AI playback + caller double-talk 的 HT813/PSTN 真實路徑中，多數 ASR transcript 仍被 TTS/echo 主導，caller 語音無法可靠進到 transcript。

因此本輪改問：

> **在 ASR 前，能不能取得比現在 mixed RX 更乾淨、更能保留 caller 的 media signal？**

---

## 二、本輪禁止事項

嚴格禁止：

- 改 semantic threshold
- 改 novel threshold
- 改 persistence
- 改 500ms / 2000ms window
- 改 polling interval
- 加 phrase whitelist
- 加 case-specific workaround
- 改 v9/v10/v10.1 frozen semantic decision logic
- 把 v10.1 失敗結果重新調參到 PASS
- 直接打開 production barge-in
- deploy production
- 修改正式 9090 application behavior
- force push
- `git reset --hard`

本輪也不要先做大規模 neural AEC / audio enhancement sweep。

理由：

> 先搞清楚 media path 與 caller observability，再決定值得在哪一層處理。

---

## 三、本輪核心假說

至少驗證以下 3 個假說：

### H1 — HT813 / PSTN media path 是主要污染來源

如果同樣的：

- TTS
- ASR
- caller phrase
- playback timing
- semantic replay

在 SIP / digital control path 上 caller transcript 明顯恢復，而 HT813/PSTN 仍失敗，則主因高度集中在 analog/FXO/media-path contamination。

### H2 — 現在抓到的是不理想的 tap point

可能目前供 ASR 使用的 RX 已經是：

- caller
- far-end playback echo
- analog nonlinear residual
- AGC / gain / codec / channel mixing

的混合結果。

如果系統內其實還能取得更早、更乾淨的 RX leg、bridge leg、channel audio、RTP direction、externalMedia、MixMonitor raw leg 或其他 per-direction stream，應優先比較。

### H3 — 即使 digital path 也失敗，則問題才可能在 ASR / double-talk modeling 層

如果 SIP/digital control 仍無法保留 caller，才進下一階段：

- known-TTS-conditioned separation
- double-talk ASR
- speaker-conditioned / source-conditioned ASR
- neural separation

本輪不預設哪個假說成立。

---

## 四、第一階段：Media Path Inventory

先做純讀取盤點，不改 production。

必須把目前電話媒體路徑畫清楚。

至少回答：

1. PSTN caller audio 從 HT813 進來後依序經過哪些 component？
2. Asterisk 是否參與？若是，在哪個 channel/bridge？
3. AI TTS audio 從哪裡送入？
4. 現在 ASR 實際吃的是哪個 audio source？
5. `saas-dbg-*.raw` 是在哪個 tap point 寫出的？
6. `RX` / `TX` / actual-sent-PCM 各自代表什麼？
7. 是否存在 pre-mix caller leg？
8. 是否存在 per-channel recording？
9. 是否能取得 RTP inbound/outbound 分離資料？
10. 是否用了 Asterisk `MixMonitor` / `Monitor` / audiohook / externalMedia / snoop channel / bridge tap？
11. 是否有 codec transcode？
12. sample rate / bit depth / gain chain 為何？
13. HT813 是否做 LEC/NEC/VAD/AGC？
14. Asterisk 或 app 層是否還有 gain / volume / normalization？

輸出：

```text
results/full_duplex_v11/
  MEDIA_PATH_MAP.md
  media_path_inventory.json
```

並畫：

```text
PSTN caller
→ HT813 FXO
→ SIP/RTP
→ Asterisk/channel/bridge
→ app RX tap
→ ASR
```

以及反向：

```text
TTS
→ app TX
→ Asterisk/channel/bridge
→ SIP/RTP
→ HT813 FXO
→ PSTN
```

每個箭頭標：

- protocol
- sample rate
- gain
- codec
- mixing point
- 可否單獨 capture

---

## 五、第二階段：找可用的 Caller-Side Tap Points

不要直接假設現在 RX 就是最佳 source。

請盤點能實際取得的候選 tap point，例如但不限於：

- app current RX
- Asterisk inbound channel before bridge
- bridge input leg
- snoop channel
- externalMedia inbound stream
- MixMonitor separate receive leg
- RTP inbound capture
- pcap / RTP decode
- HT813 SIP-side inbound RTP
- 任何目前已存在但未餵 ASR 的 caller-only leg

對每個候選建立表：

| Tap | Caller present | TTS echo present | Pre/Post mix | Accessible | Realtime | Production impact |
|---|---|---|---|---|---|---|

目標不是找最多 tap，而是找：

> **最早、最乾淨、最接近 caller-only、又能 realtime 取得的點。**

如果候選 tap 需要改 production 才能測：

先設計隔離 PoC，不直接改 9090。

---

## 六、第三階段：建立 SIP / Digital Control

這是本輪最重要的控制實驗。

建立一條不經 HT813/analog PSTN 的 digital control path。

可以使用現有可行方式，例如：

- SIP softphone
- local SIP endpoint
- PJSIP test endpoint
- WebRTC/SIP endpoint
- local loopback digital call
- 其他 repo 已存在的 safe test path

不要為了這個任務重構整套電話系統。

要求：

- 盡量沿用相同 AI/TTS pipeline
- 沿用相同 TEA-ASR
- 沿用相同 test phrases
- 沿用相同 500/2000 interim replay measurement
- 不開 live cut
- shadow only

目的：

> 唯一改變主要 media path：HT813/PSTN vs SIP/digital。

---

## 七、A/B 對照矩陣

至少做：

### Path A — HT813 / PSTN

用現有 v10.1 recordings 優先，避免無謂重打。

若需要補錄，最多補少量，不能再做 5 通大規模 protocol acceptance。

### Path B — SIP / Digital Control

收集至少：

- 5 個 Normal Caller double-talk events
- 最好 8～10 個
- 每個 0.5～1.5s
- caller 在 AI playback 中插話
- post-onset ≥3s

使用固定 phrase，可沿用：

- `等一下，我現在不方便。`
- `你哪裡打來的？`
- `不用了，謝謝。`
- `我現在在忙，晚點再說。`
- `這個是什麼服務？`
- `你剛剛說什麼？`
- `我沒有要賣房子。`

Short Caller 本輪不是主驗收，不必浪費時間。

---

## 八、對每個 Path 都必須量的指標

### 1. Caller observability

每個 event：

- caller phrase
- caller onset
- caller duration
- ASR first non-empty
- ASR first caller-bearing text
- caller chars observed
- expected caller chars
- caller char coverage
- caller CER / normalized edit distance（若合理）
- TTS echo chars observed
- TTS echo coverage
- transcript attribution：caller / echo / mixed / empty

### 2. Echo dominance

每個 semantic window：

- similarity to active TTS
- caller novel chars
- TTS chars
- caller-to-TTS text ratio
- whether transcript is echo-dominant

### 3. Latency

- caller onset → first caller token
- caller onset → first usable caller text
- ASR inference latency
- audio evidence accumulation

### 4. Audio signal

至少比較：

- caller-only section RMS
- echo-only section RMS
- double-talk RMS
- caller/echo relative level
- clipping
- crest factor
- simple SNR-like proxy

不要再把這些 signal features 當 classifier threshold，只拿來描述 path quality。

---

## 九、最重要的 Comparison Table

最後必須有一張：

| Metric | HT813/PSTN | SIP/Digital | Delta |
|---|---:|---:|---:|
| Normal events | | | |
| caller transcript available | | | |
| caller coverage median | | | |
| caller CER median | | | |
| echo-dominant windows | | | |
| first caller token median | | | |
| first caller token P90 | | | |
| ASR P50 | | | |
| ASR P90 | | | |
| caller/echo level proxy | | | |

如果 SIP 能顯著恢復 caller transcript，這張表應該直接看得出來。

---

## 十、Path Attribution Verdict

本輪不要直接用 PASS/FAIL 模糊結論。

最後必須選最合理的一個：

### `ANALOG_MEDIA_PATH_CONFIRMED`

條件：

- SIP/digital caller observability 明顯好於 HT813/PSTN
- caller transcript availability / coverage 有明顯改善
- 同 ASR / 同 TTS pipeline 下只有 media path 改變

代表：

> 問題主要在 HT813/PSTN/analog/media path。

下一步才研究更乾淨的 tap、AEC/reference、media architecture。

### `CURRENT_TAP_POINT_CONFIRMED_BAD`

條件：

- 同一通/同一路徑中，某個 alternate inbound tap 明顯優於 current RX

代表：

> 不是 HT813 本身必然不可用，而是現在 ASR tap 點選錯或太晚。

下一步改 ASR input source。

### `DIGITAL_PATH_ALSO_FAILS`

條件：

- SIP/digital double-talk 下 caller observability 仍極低
- 沒有顯著優於 HT813/PSTN

代表：

> 不應再把問題歸咎於 analog path alone。

下一步才值得做 known-TTS-conditioned separation / double-talk ASR。

### `MEDIA_PATH_INCONCLUSIVE`

條件：

- control path 不等價
- sample 不足
- tap 無法對齊
- evidence 不完整

不要硬下結論。

---

## 十一、Alternate Tap PoC

如果 inventory 發現有明確更乾淨的 inbound tap：

允許做隔離 PoC。

但必須：

- 新 port / test path
- 不改 production 9090 behavior
- barge-in OFF
- 不 deploy
- 不替換正式 ASR input

用 3～5 個已有或新測 event 即可。

比較：

```text
current RX
vs
alternate caller-side tap
```

同一 event 盡量共用時間軸。

若 alternate tap caller transcript availability 明顯改善，可直接判：

`CURRENT_TAP_POINT_CONFIRMED_BAD`

---

## 十二、不要把 HT813 當預設罪魁禍首

重要：

HT813/audio transport 已證明 half-duplex 可工作。

本輪不能直接說：

`HT813 不支援 full duplex`

除非證據真的支持。

正確要區分：

- HT813 analog echo contamination
- Asterisk bridge mixing
- app tap point
- codec / gain
- PSTN hybrid nonlinear echo
- ASR robustness

要靠 A/B evidence 定位。

---

## 十三、Asterisk / SIP 證據

如果系統使用 Asterisk/PJSIP，請讀實際 config / runtime evidence，確認：

- endpoint/channel
- codec
- direct_media
- bridge technology
- RTP direction
- MixMonitor/audiohook
- jitterbuffer
- gain/volume

若可安全取得 pcap / RTP metadata，可用於 research evidence。

不要抓取或輸出不必要的敏感資料。

只保存測試所需 audio / timestamps / technical metadata。

---

## 十四、Human Gate

如果需要真人建立 SIP softphone endpoint、接聽或說測試台詞：

先把所有可自動準備的工作做完，再停 HUMAN GATE。

只告訴真人：

- 要用哪個 safe test endpoint
- 要說哪句
- 何時插話
- 測完回覆什麼

不要讓真人自己拼 SIP/Asterisk command。

如果需要帳密/QR/2FA，不自行猜測或讀出 secrets。

---

## 十五、建議測試腳本

Digital control 每通只做 2 個 Normal event 即可。

### D01

A：`等一下，我現在不方便。`

B：`你哪裡打來的？`

### D02

A：`不用了，謝謝。`

B：`我現在在忙，晚點再說。`

### D03

A：`這個是什麼服務？`

B：`你剛剛說什麼？`

### D04（若需要）

A：`我沒有要賣房子。`

B：`等一下，你先不要講。`

每個 event：

- AI playback 中插話
- 不 cut TTS
- post-onset ≥3s
- full recording 保存

---

## 十六、輸出目錄

建立：

```text
results/full_duplex_v11/
  MEDIA_PATH_MAP.md
  media_path_inventory.json
  tap_candidates.csv
  digital_control/
  pstn_baseline/
  alternate_tap/
  caller_observability.csv
  transcript_attribution.csv
  latency_breakdown.json
  signal_path_metrics.json
  path_comparison.csv
  PATH_ATTRIBUTION_REPORT.md
  FINAL_REPORT_V11.md
  failures/
```

不要覆蓋 v10 / v10.1 evidence。

---

## 十七、Final Report 必答

至少回答：

1. current ASR tap 在哪一層？
2. caller 與 TTS echo 在哪一層第一次混合？
3. 有沒有 caller-only / cleaner inbound tap？
4. HT813/PSTN caller transcript availability？
5. SIP/digital caller transcript availability？
6. SIP 是否顯著改善 caller coverage？
7. caller CER / text coverage 差多少？
8. echo dominance 差多少？
9. first caller token latency 差多少？
10. alternate tap 是否比 current RX 好？
11. ASR latency 是否仍正常？
12. problem attribution 是 analog path / tap point / digital-also-fails / inconclusive？
13. production 是否完全未開 barge-in？
14. 9090 是否未改 behavior？
15. 下一步唯一建議是什麼？

---

## 十八、下一步決策樹

### 若 `ANALOG_MEDIA_PATH_CONFIRMED`

下一輪：

- media architecture / echo reference
- inbound tap relocation
- Asterisk media separation
- HT813/PSTN-specific echo handling

不要回去調 semantic threshold。

### 若 `CURRENT_TAP_POINT_CONFIRMED_BAD`

下一輪：

- 將 cleaner tap 接到 PoC ASR
- 重跑 caller observability
- 成功後才重新測 semantic gate

### 若 `DIGITAL_PATH_ALSO_FAILS`

下一輪：

- known-TTS-conditioned speech separation
- double-talk ASR
- source-conditioned transcription

這時才進模型層研究。

### 若 `MEDIA_PATH_INCONCLUSIVE`

補最小必要 evidence，不要大改系統。

---

## 十九、成功條件不是讓 Semantic PASS

本輪成功不是：

`把 barge-in 做成功`

而是：

> **把 caller observability failure 精準定位到 media path 的哪一層。**

只要能用 evidence 清楚回答：

- HT813/PSTN 有問題，SIP 沒問題
- current tap 有問題，alternate tap 沒問題
- 或 SIP 也一樣失敗

本輪就是成功。

---

## 二十、最終原則

不要再在已失敗的 frozen semantic gate 上調 threshold。
不要再做 audio-only magic threshold。
不要因為 HT813 是 analog 就先入為主。
用 A/B control 把問題定位。

本輪唯一核心問題：

> **同一套 TTS + ASR + caller phrase，在 HT813/PSTN 與 SIP/digital path 下，caller 的可觀測性是否有本質差異？如果有，差異發生在哪個 media tap / mixing point？**

開始執行。