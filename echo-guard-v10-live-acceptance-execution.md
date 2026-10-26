# EchoGuard v10 — 真人 Live Acceptance 執行任務

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。
>
> 本輪不是再建工具、不是再改演算法、不是再調參。`v10_operator.py` 已完成並 self-test PASS。現在唯一工作是：用既有 Operator Helper 帶真人完成 T01～T05 interruption acceptance，取得第一份有效的 Normal Caller recall 與 latency 結果。

---

## 一、目前狀態

已完成：

- `v10_collect.py`
- `v10_operator.py`
- v9 frozen replay logic
- per-node TTS reference
- evidence ingest / onset scan / GT / validation
- operator preflight / prepare / start-case / finish-case / review / validate / summary / run-acceptance / cleanup
- Operator self-test PASS
- 9090 PASS
- TEA PASS

目前 dataset verdict：

`LIVE_COLLECTION_INSUFFICIENT`

原因只有一個：還沒有 3～5 通真正的真人 interruption calls。

---

## 二、硬性規則

本輪禁止：

- 改 semantic threshold
- 改 persistence
- 改 sliding window / polling interval
- 改 ASR
- 改 EchoGuard algorithm
- 修改 production application code
- 打開正式 barge-in
- deploy
- force push
- `git reset --hard`

真人測試必須保持：

- barge-in OFF
- mode = shadow
- debug recording ON
- AI 播放照常繼續，不真正被 caller cut 掉

本輪是 frozen acceptance，不是 tuning。

---

## 三、你的角色

你是執行代理，不是規劃代理。

不要再產出新的規劃文件。

你要：

1. 進入實際 repo
2. 執行既有 `v10_operator.py preflight`
3. 確認 ready
4. 在真正需要真人允許環境切換時停在 HUMAN GATE
5. prepare 成功後帶真人逐通做 T01～T05
6. 每通真人完成後，自動執行 finish / review / validate
7. 所有資料夠後執行 run-acceptance
8. 最後執行 cleanup
9. 回報最終 verdict

所有不需要真人的步驟都由你自己完成。

---

## 四、Phase 0 — Preflight

先確認實際 repo 與 git 狀態。

然後執行既有 Operator Helper 的：

`preflight`

必須確認：

- 9090 PASS
- TEA PASS
- `saas-streaming` active
- active calls = 0
- Operator Helper ready
- v10 backend ready
- barge-in OFF

若 critical preflight FAIL：停止真人測試，只處理 blocker，不重構。

---

## 五、HUMAN GATE — Prepare

Preflight PASS 後，先停在 HUMAN GATE。

只需告訴真人：

- 現在沒有 active calls
- 9090 / TEA 都 PASS
- 下一步會由既有 Operator Helper 套用 shadow/debug 測試狀態並重新驗證 runtime

真人確認可進行後，才執行既有：

`prepare`

prepare 成功標準：

- barge-in OFF
- mode shadow
- debug ON
- 9090 PASS
- TEA PASS

如果 prepare FAIL，不開始通話。

---

## 六、真人測試目標

預設做完 T01～T05 五通。

目標至少：

- Normal Caller ≥ 6 events
- Short Caller ≥ 4 events
- 至少 3 通不同真電話
- 插話位置涵蓋不同 AI turn / 句首句中句尾

每個事件：

- caller 必須在 AI 正在說話時插話
- AI 不停是正常的，因為 barge-in 仍 OFF
- 不要因為 AI 沒停而重複講第二次

---

## 七、Frozen 真人腳本

### T01

A｜Normal

時機：AI 第一句開始約 1 秒、句中。

說：

`等一下，我現在不方便。`

B｜Short

下一個 AI turn 說：

`嗯`

---

### T02

A｜Normal

說：

`你哪裡打來的？`

B｜Short

說：

`喂`

---

### T03

A｜Normal

說：

`不用了，謝謝。`

B｜Short

說：

`不用`

---

### T04

A｜Normal

說：

`我現在在忙，晚點再說。`

B｜Short

說：

`欸`

---

### T05

A｜Normal

說：

`這個是什麼服務？`

B｜Normal

下一個不同 AI turn 說：

`你剛剛說什麼？`

---

## 八、每通固定流程

以 T01 為例。

先由你執行：

`start-case T01`

確認 Operator 顯示本通腳本後，停在 HUMAN GATE：

- 請真人用既有安全測試流程開始 V10-T01
- 接電話
- AI 正在說話時依畫面做 A / B 插話
- 完成後結束通話
- 真人只需要回覆「T01 完成」

你不得猜測任何私人測試目的地資訊，也不得自行建立新的外撥流程。

---

## 九、真人完成後由代理自動處理

真人回覆 T01 完成後，你自己繼續執行既有 Operator Helper：

- `finish-case T01`
- `review T01`

finish-case 應自動：

- 找 evidence
- ingest
- scan onset
- 建 review clips
- 建 onset candidates

如果 candidate 對應很清楚，再使用既有 review mapping 功能建立 GT。

如果 ambiguous：

停在 HUMAN GATE，只告訴真人需要確認哪幾個候選 clip 對應 A / B。

不要叫真人自己找 full-call WAV。

---

## 十、Validate

GT 完成後執行：

`validate T01`

T01 必須得到：

- VALID
- 或 COLLECTION_INVALID + reason

如果 invalid，不要假裝完成。

能保留的有效事件照 helper schema 保留，不能人工修改 denominator。

---

## 十一、T02～T05

依相同節奏：

`start-case → HUMAN GATE → finish-case → review → 必要時 HUMAN GATE → validate`

每完成一通都執行：

`summary`

確認目前：

- completed calls
- valid Normal events
- valid Short events
- invalid events
- 是否達 acceptance minimum

若前 3 通已達最低門檻，仍優先完成 5 通，除非真人明確停止。

---

## 十二、禁止邊測邊調參

看到新資料後禁止：

- 改 LCS threshold
- 改 persistence
- 改 window
- 改 per-node reference logic
- 加 phrase whitelist
- 加特定 case workaround
- 換 ASR

本輪先讓 frozen acceptance 得出真實結果。

如果失敗，就完整 FAIL；不要現場修到 PASS。

---

## 十三、Run Acceptance

T01～T05 完成後先執行：

`summary`

若 minimum-data gate 足夠，再執行：

`run-acceptance`

不得繞過 helper 的 minimum-data refusal。

Acceptance 主結果只使用 eligible ORIGINAL_REAL。

---

## 十四、必須回報的核心數字

### Echo Safety

- echo-only windows
- confirmed CALLER FP
- EARLY

目標：

- Echo FP = 0
- EARLY = 0

### Normal Caller

- eligible events
- TP
- FN
- recall

目標：

- Recall ≥ 90%

### Latency

- first non-empty transcript
- first novel transcript
- first usable semantic decision
- first stable decision
- median
- P90

目標：

- Median ≤ 500ms
- P90 ≤ 800ms

### Short Caller

獨立報：

- transcript availability
- recall
- latency

Short Caller 不得混入 Normal Caller 主 acceptance denominator。

---

## 十五、Latency Breakdown

如果 Normal Caller latency 偏慢，拆成：

- poll wait
- post-onset audio accumulation
- queue/coalescing
- TEA inference
- semantic compute
- total first usable decision

不要只說「慢」。

---

## 十六、最終 Verdict

依 evidence 選最合理的 verdict：

### PASS

- Echo FP = 0
- EARLY = 0
- Normal Caller Recall ≥ 90%
- Median ≤ 500ms
- P90 ≤ 800ms

### CONDITIONAL_PASS

- Normal Caller 明顯可分
- Echo safety 保持
- 但 sample / latency / short lane 尚有不足

### SEMANTIC_SEPARABLE_BUT_TOO_SLOW

- caller / echo 可分
- 但 P90 > 1000ms 或 realtime 體感不可接受

### SEMANTIC_NOT_SEPARABLE

- 有完整 Normal Caller 語料後，仍無法可靠區分

### LIVE_COLLECTION_INSUFFICIENT

- 真人資料仍未達 minimum

不要為了繼續專案而美化 verdict。

---

## 十七、Cleanup 必做

真人測試結束後，無論 PASS / FAIL，都執行既有：

`cleanup`

確認：

- 9090 PASS
- service active
- 原始環境狀態已 restore
- barge-in 仍 OFF
- dataset / evidence 保留

如果 helper 回報 `RESTORE_FAILED`，先處理 restore，停止其他研究。

---

## 十八、最終回報

最後只需要清楚回答：

1. 真人完成幾通
2. 有效 Normal / Short event 各幾個
3. Echo FP / EARLY
4. Normal Caller recall
5. Median / P90 first usable decision
6. Short Caller 表現
7. latency 最大瓶頸
8. 最終 verdict
9. cleanup / restore 是否 PASS
10. production 是否仍未開 barge-in
11. 主要輸出檔案路徑

---

## 十九、本輪唯一核心問題

> 在真實 HT813 + 電話回音 + AI playback + 真人 Normal Caller double-talk 場景下，既有 frozen Semantic Guard 能否在 Echo FP=0、EARLY=0 的前提下，以可接受延遲可靠辨出 caller？

不要再建工具。
不要再調參。
直接執行真人 acceptance。

開始執行。