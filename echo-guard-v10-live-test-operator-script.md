# EchoGuard v10 — 真人 Live Interruption 測試操作腳本

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。目標不是再改演算法，而是把真人測試環境、操作順序、通話腳本、證據收集、GT 標註、驗收與收尾全部編排好，讓操作者只需要照表測試。

---

# 一、目前狀態

EchoGuard v10 已準備完成，目前判定：

`LIVE_COLLECTION_INSUFFICIENT`

已完成：

- `v10_collect.py` collection harness
- `--ingest`
- `--scan-onset`
- `--add-event`
- `--validate-only`
- existing v9 replay integration
- per-node TTS reference
- GT schema
- v10 dataset skeleton
- dry-run self-test
- operator runbook
- shadow overlay reference config

目前唯一缺少：

> **3～5 通真人 live interruption calls。**

本輪不得再改：

- Semantic threshold
- persistence
- polling interval
- ASR model
- EchoGuard algorithm
- production runtime

本輪唯一工作：

> **把真人測試流程排好，讓操作者照腳本完成 3～5 通電話，收集至少 5 個 Normal Caller + 3 個 Short Caller event，完成 GT，然後自動跑 v10 acceptance。**

---

# 二、硬性安全規則

必須遵守：

- `SAAS_BARGEIN=0`
- `SAAS_BARGEIN_MODE=shadow`
- 正式 barge-in 不得開啟
- AI 播放不得真的被 interruption cut 掉
- 9090 程式碼不得修改
- 不 deploy production
- 不改 ASR
- 不把 SenseVoice 換成 TEA production
- 不改 threshold
- 不改 persistence
- 不改 v9 semantic logic
- 不 force push
- 不 `git reset --hard`

如果任何步驟會碰正式 production 行為：停止。

本輪允許：

- zero-traffic 時段重新啟動 `saas-streaming`
- 只透過 systemd override 加 shadow/debug env
- 啟動既有 PoC / TEA service
- 真人撥打 / 接聽測試電話
- 收集 full-call evidence
- 建立 GT
- offline replay
- acceptance report

---

# 三、你的角色

你是執行代理。

你必須先自動完成所有不需要真人的準備工作，然後在真正需要真人拿電話測試時，**只輸出一份非常簡單的操作面板 / checklist**，不要丟大量 command 給操作者自行判斷。

你要做到：

1. 檢查 repo/runtime
2. 檢查 v10 harness
3. 檢查 TEA-ASR
4. 檢查 9090
5. 建立 shadow/debug override
6. 啟動 / 驗證測試環境
7. 建立 `T01～T05` 測試腳本
8. 建立每通通話 checklist
9. 每通完成後自動抓 evidence
10. 自動 ingest
11. 輔助 onset scan
12. 等真人確認 onset / offset / phrase
13. 自動 add-event
14. 自動 quality gate
15. 全部完成後自動 replay acceptance
16. 產生 summary
17. 自動還原 shadow override
18. 驗證 9090 正常

不要讓真人自己組 command。

---

# 四、Phase 0 — Preflight

先執行：

```bash
git status
git branch
git log --oneline -20
```

確認實際專案位置，預期但不要硬假設：

```text
F:\00-Ticenpi-SaaS\Ticenpi-Voip
```

檢查：

- `_workspace/acceptance/live/ht813/v10_collect.py`
- `results/full_duplex_v10/`
- `run_v9.py`
- `scripts/voip-call-evidence.sh`
- `docs/runbooks/echo-guard-v10-live-collection.md`
- `scripts/saas-shadow-v10.conf`
- TEA endpoint `:19110`
- `saas-streaming`
- 9090 listening
- 19090 listening（如果目前 PoC 仍使用）
- 19110 listening

輸出 preflight 表：

| Item | Expected | Actual | PASS/FAIL |

如果 v10 harness 缺失：停止。

如果 TEA 不健康：先修復 PoC/research service，但不得更換模型。

如果 9090 非目前既有狀態：停止。

---

# 五、建立自動化操作入口

請建立一個 **operator helper**，名稱可依 repo 現況，例如：

```text
_workspace/acceptance/live/ht813/v10_operator.py
```

或：

```text
scripts/v10-live-operator.ps1
```

不要重做 `v10_collect.py`。

operator helper 只包裝既有流程，讓真人不用記 command。

至少支援：

```text
prepare
status
start-case T01
finish-case T01
scan T01
add-event T01 ...
validate T01
run-acceptance
cleanup
```

如果 repo 已有等價工具，沿用，不要重複造輪子。

---

# 六、Prepare 必須做什麼

`prepare` 必須自動：

1. 確認零通話 / 無 active call（如果系統有辦法查）
2. 備份現有 systemd override 狀態
3. 建立：

```ini
[Service]
Environment=SAAS_STREAM_DEBUG=1
Environment=SAAS_BARGEIN=0
Environment=SAAS_BARGEIN_MODE=shadow
```

4. `systemctl daemon-reload`
5. restart `saas-streaming`
6. 等待 service ready
7. 驗證 9090 listening
8. 回讀 process env
9. 驗證：

```text
SAAS_STREAM_DEBUG=1
SAAS_BARGEIN=0
SAAS_BARGEIN_MODE=shadow
```

10. 驗證 TEA :19110 healthy
11. 驗證 v10 results directory writable

只有全部 PASS 才能進真人測試。

如果任何一項 FAIL：停止並說明。

---

# 七、真人測試整體規模

預設執行 **5 通**，如果前 3 通已取得足夠資料，仍建議跑完 5 通，因為目前 sample 太少。

目標至少：

- 5 通真人電話
- Normal Caller ≥ 6 events
- Short Caller ≥ 4 events
- 每通 ≥ 2 interruption
- 不同 playback node / 不同句首中尾位置

如果實際只能做 3 通，最低：

- Normal Caller ≥ 5
- Short Caller ≥ 3

---

# 八、測試腳本總表

請建立並輸出以下真人腳本，讓操作者照著做。

## T01 — 基準 Normal + Short

### Event T01-A — Normal

時機：

> AI 第一句開始後約 1 秒，在句中插話。

真人說：

```text
等一下，我現在不方便。
```

目標 duration：

`0.8～1.3s`

### Event T01-B — Short

時機：

> 下一個 AI turn 開始後約 0.5～1 秒。

真人說：

```text
嗯
```

目標 duration：

`80～250ms`

---

## T02 — 問句型 Normal + Short

### Event T02-A — Normal

時機：AI 句子前段。

真人說：

```text
你哪裡打來的？
```

目標 duration：

`0.6～1.0s`

### Event T02-B — Short

時機：AI 句中。

真人說：

```text
喂
```

目標 duration：

`100～250ms`

---

## T03 — 拒絕型 Normal + Short

### Event T03-A — Normal

時機：AI 句中偏後。

真人說：

```text
不用了，謝謝。
```

目標 duration：

`0.6～1.0s`

### Event T03-B — Short

真人說：

```text
不用
```

目標 duration：

`150～300ms`

---

## T04 — 長一點 Normal + Short

### Event T04-A — Normal

時機：AI 句中。

真人說：

```text
我現在在忙，晚點再說。
```

目標 duration：

`1.0～1.5s`

### Event T04-B — Short

真人說：

```text
欸
```

目標 duration：

`80～200ms`

---

## T05 — 資訊詢問型 + 額外 Normal

### Event T05-A — Normal

真人說：

```text
這個是什麼服務？
```

目標 duration：

`0.7～1.1s`

### Event T05-B — Normal

真人說：

```text
你剛剛說什麼？
```

目標 duration：

`0.7～1.1s`

---

# 九、真人操作規則

操作者每通只需要做：

1. 按 / 執行 `start-case T0X`
2. 接聽測試電話
3. 照腳本在 AI 說話期間插話
4. **不要因為 AI 沒停而重講**
5. AI 要照常播完，因為 barge-in 還是 OFF
6. 通話結束後執行 `finish-case T0X`
7. 等系統自動收 evidence / ingest
8. 聽 onset 候選，確認真正開始時間
9. 確認 offset / phrase / node
10. 系統自動 add-event + validate

不要讓操作者自己找檔案或手動複製 evidence。

---

# 十、每通開始前自動檢查

`start-case T01` 必須確認：

- 9090 正常
- barge-in OFF
- mode shadow
- debug ON
- recording path ready
- case ID 尚未使用
- results path writable
- TEA healthy

然後輸出一張超簡單卡片：

```text
CASE T01 READY

這通請做：
A. AI 第一句講約 1 秒時說：
   「等一下，我現在不方便。」

B. 下一個 AI turn 說：
   「嗯」

注意：AI 不會真的停，這是正常的。
完成後結束通話，再執行 finish-case T01。
```

T02～T05 同理。

---

# 十一、如果系統可以自動撥測試電話

如果 repo / API 已經有：

```text
POST http://127.0.0.1:50900/api/calls/dial
```

operator helper 可以幫忙準備 dial request，case name：

```text
V10-T01
V10-T02
V10-T03
V10-T04
V10-T05
```

但：

- 不要猜電話號碼
- 不要從 repo 洩漏 private number
- 如果需要真人確認號碼，停在「READY TO DIAL」
- 不要在沒有真人明確操作 / 現有安全流程的情況自行外撥

---

# 十二、每通結束後自動 Evidence Collection

`finish-case T01` 必須：

1. 找到對應 call evidence
2. 執行既有：

```bash
sudo bash scripts/voip-call-evidence.sh V10-T01
```

或實際 repo 等價命令。

3. 確認至少存在：

- RX full-call raw / wav
- streaming.log
- turns.csv
- summary.json
- version-and-params.txt
- shadow.csv（若 runtime 有）
- debug raw

4. 驗證檔案非空
5. 把 evidence path 傳給：

```bash
python v10_collect.py --ingest <evidence_dir> --case T01
```

6. 執行：

```bash
python v10_collect.py --scan-onset T01
```

---

# 十三、Onset 標註 UX

不要只輸出一串毫秒讓真人猜。

如果可行，建立簡單 helper：

- 列出 RMS-rise candidate
- 每個 candidate 顯示 timestamp
- 切出 candidate 前後各 1～2 秒的小 WAV
- 讓真人逐個聽

例如：

```text
T01 onset candidates

1. 00:08.420
2. 00:14.860
3. 00:21.120

A event 預期：「等一下，我現在不方便」
B event 預期：「嗯」
```

如果可以自動用現有 ASR 輔助轉錄，可以顯示，但不得用 ASR 自動 GT 取代真人確認。

---

# 十四、Ground Truth 寫入

真人確認後由 helper 執行既有：

```bash
python v10_collect.py --add-event T01 \
  --event-id T01-A \
  --type normal \
  --phrase "等一下，我現在不方便" \
  --onset <CALL_TIMELINE_MS> \
  --offset <CALL_TIMELINE_MS> \
  --node <NODE_ID>
```

Short 同理。

所有 onset / offset 必須：

> **相對 call start。**

node 必須從：

- `turns.csv`
- `streaming.log`
- playback timeline

取得。

禁止憑空猜 node。

---

# 十五、每通立即 Quality Gate

每通做完就驗，不要 5 通結束才發現壞掉。

執行：

```bash
python v10_collect.py --validate-only
```

或 case-scoped equivalent。

每個 event 至少確認：

- full RX exists
- caller phrase 聽得到
- onset 可定位
- offset 可定位
- post-onset audio ≥ 2s
- active node reference 存在
- AI playback 當時 active
- TEA replay 可執行
- metadata 完整

如果不合格：

標：

`COLLECTION_INVALID`

然後在下一通補一個同類型 event。

---

# 十六、測試過程不得做的事情

禁止在 T01～T05 中途看到結果不好就：

- 改 LCS threshold
- 改 persistence 2→1
- 改 500x2000 window
- 改 polling
- 加 phrase whitelist
- 加特殊 case hack
- 改 TEA prompt
- 換 ASR

我們就是要驗 **v9 frozen logic**。

---

# 十七、收集完成條件

只有以下條件至少達成才可跑正式 v10 acceptance：

- 真人 calls ≥ 3
- eligible Normal Caller ≥ 5
- eligible Short Caller ≥ 3
- 每個 event post-onset ≥ 2s
- GT onset/offset confirmed
- active node reference exists
- full call source traceable

建議目標：

- 5 calls
- 6 Normal
- 4 Short

---

# 十八、Acceptance 執行

完成收集後執行：

```bash
python v10_collect.py
```

必須沿用：

- v9 frozen semantic logic
- 500x2000（如實際 v9 是此設定）
- threshold 0.5（如實際 v9 是此設定）
- persistence 2（如實際 v9 是此設定）
- adaptive per-node reference

執行前先從 `run_v9.py` 回讀實際值，不要只相信本文件。

如果實際值不同，以 frozen v9 實際值為準，並在報告寫清楚。

---

# 十九、Acceptance 必須輸出

## A. Normal Caller

逐 event：

- phrase
- onset
- first non-empty ASR
- first novel transcript
- first `SEM_CALLER`
- first stable `SEM_CALLER`
- recall
- latency

Aggregate：

- Recall
- Median first usable
- P90

目標：

- Recall ≥ 90%
- Median ≤ 500ms
- P90 ≤ 800ms

---

## B. Short Caller

逐 event：

- phrase
- duration
- ASR transcript availability
- semantic state
- first usable latency
- fail reason

Short lane 不得污染 Normal Caller denominator。

---

## C. Echo Safety

必須報：

- echo windows
- confirmed caller FP
- EARLY

要求：

- FP = 0
- EARLY = 0

---

## D. Latency Breakdown

每個 Normal event：

```text
poll_wait_ms
audio_evidence_ms
queue_wait_ms
asr_request_ms
semantic_compute_ms
first_usable_semantic_decision_ms
first_stable_semantic_decision_ms
```

Aggregate：

| Component | Median | P90 | % Total |

---

# 二十、最終判定

只能從以下選：

## PASS

至少：

- 3 live calls
- 5 eligible Normal events
- Echo FP=0
- EARLY=0
- Recall ≥90%
- Median ≤500ms
- P90 ≤800ms

→ 可以進下一階段 `shadow runtime integration / live no-cut acceptance`。

## CONDITIONAL_PASS

Semantic caller 明顯可分，但 sample / latency / short lane 還需補。

## SEMANTIC_SEPARABLE_BUT_TOO_SLOW

Recall 可接受，但 P90 >1000ms。

## SEMANTIC_NOT_SEPARABLE

在完整 Normal Caller 語料下仍大量與 echo 混淆。

## LIVE_COLLECTION_INSUFFICIENT

真人資料仍不足。

---

# 二十一、Cleanup 必須自動執行

無論 PASS / FAIL，最後執行：

1. 移除 / 還原 `zz-v10-shadow.conf`
2. `systemctl daemon-reload`
3. restart `saas-streaming`
4. 驗證 9090 listening
5. 回讀 process env
6. 確認：

```text
SAAS_BARGEIN=0
```

或回到原本 production env。

7. 確認沒有留下測試 override
8. 不 deploy
9. 不 push

如果 cleanup 失敗，報：

`CLEANUP_FAILED`

不得宣告測試完成。

---

# 二十二、最後回報格式

完成後只需要給操作者一份簡潔摘要：

```text
EchoGuard v10 Live Test

Calls: X
Normal events: X
Short events: X
Invalid events: X

Echo FP: X
EARLY: X
Normal Recall: X%
Median decision: X ms
P90 decision: X ms
Short recall: X%

Verdict: XXXXX

Report:
<path>

9090 restored: YES/NO
Barge-in production enabled: NO
```

並附：

- `ACCEPTANCE_REPORT_V10.md`
- `live_collection_manifest.json`
- `interruption_ground_truth.json`
- `caller_timeline.csv`
- `echo_safety.json`
- `latency_breakdown.json`

---

# 二十三、最重要原則

這輪不要再做研究型調參。

這輪就是一次 **真人資料驗收**。

操作者應該只需要：

1. `prepare`
2. T01～T05 照台詞插話
3. 每通 `finish-case`
4. 聽 candidate 確認 onset/offset
5. 最後 `run-acceptance`
6. `cleanup`

其他能自動做的全部自動做。

本輪唯一真正要回答的問題：

> **在真實 HT813 + 電話回音 + AI playback + 真人正常插話的 double-talk 場景下，既有 frozen Semantic Guard 能不能在 Echo FP=0 的前提下，以可接受延遲可靠辨出 Normal Caller？**

開始執行。