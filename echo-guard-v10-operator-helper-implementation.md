# EchoGuard v10 — Operator Helper 完整實作與真人測試入口

> 用途：交給 IDE / DeepSeek V4.1 Flash 執行。這一輪不是再研究 EchoGuard 演算法，而是把目前已完成的 v10 collection backend 包成真正可操作的真人測試工具，讓操作者不需要自己拼 command、找 evidence、手動追 node、自己整理資料。

---

# 一、目前狀態

EchoGuard v10 backend 已完成，但真人 Operator UX 尚未完成。

目前已有：

- `_workspace/acceptance/live/ht813/v10_collect.py`
- `--ingest <evidence_dir> --case T01`
- `--scan-onset T01`
- `--add-event ...`
- `--validate-only`
- full replay / acceptance
- v9 frozen replay logic
- per-node TTS reference
- GT schema
- `results/full_duplex_v10/`
- `docs/runbooks/echo-guard-v10-live-collection.md`
- `scripts/saas-shadow-v10.conf`
- dry-run self-test

目前 verdict：

`LIVE_COLLECTION_INSUFFICIENT`

原因只有一個：

> 還沒有 3～5 通真正的 live interruption calls。

但現在不能直接要求真人自行操作一長串 shell / Python command。

本輪任務是完成：

> **v10 Operator Helper**

讓操作者之後只需要執行少數固定命令，就可以完成：

```text
prepare
→ start-case T01
→ 真人接電話照畫面插話
→ finish-case T01
→ 確認 onset 候選
→ validate
→ T02～T05
→ run-acceptance
→ cleanup
```

---

# 二、這一輪禁止改演算法

嚴格禁止：

- 改 semantic threshold
- 改 persistence
- 改 sliding window
- 改 polling interval
- 改 v9 / v10 semantic decision logic
- 改 EchoGuard acoustic logic
- 換 ASR
- 把 SenseVoice 改成 TEA production
- 修改 production 9090 application code
- 真正開 barge-in
- deploy production
- force push
- `git reset --hard`

本輪只做：

- Operator UX
- command orchestration
- environment preflight
- evidence automation
- onset review UX
- GT helper
- acceptance wrapper
- cleanup / restore

如果為了做 Operator Helper 必須改演算法：停止並回報。

---

# 三、安全條件

真人測試全程必須：

```text
SAAS_BARGEIN=0
SAAS_BARGEIN_MODE=shadow
SAAS_STREAM_DEBUG=1
```

AI 播放：

- 照常播放
- 不真正被 caller cut 掉
- shadow judge 只記錄 would-barge

9090 可以因 zero-traffic 測試時段 restart `saas-streaming` 來套 shadow/debug env，但：

- 不得修改 9090 程式碼
- 不得 deploy
- 不得變更 production dialplan
- 測完必須完整 restore

如果當下有 active call，`prepare` 必須拒絕操作。

---

# 四、開始前先盤點實際 repo

先執行：

```bash
git status
git branch
git log --oneline -20
```

確認實際專案位置。

預期但不能硬假設：

```text
F:\00-Ticenpi-SaaS\Ticenpi-Voip
```

必須找到並確認：

- `_workspace/acceptance/live/ht813/v10_collect.py`
- v9 runner / replay implementation
- `scripts/voip-call-evidence.sh`
- `docs/runbooks/echo-guard-v10-live-collection.md`
- `scripts/saas-shadow-v10.conf`
- `results/full_duplex_v10/`
- call API / dial helper
- evidence directory convention
- `saas-streaming` systemd unit
- 9090
- 19090（若仍使用）
- 19110 TEA-ASR
- active-call status API / process / logs（如果存在）

不要憑記憶重寫。

先輸出：

| Item | Path / Endpoint | Status | Evidence |
|---|---|---|---|

---

# 五、本輪主要交付：Operator Helper

優先建立：

```text
_workspace/acceptance/live/ht813/v10_operator.py
```

如果實際 repo 更適合 PowerShell，也可以建立 wrapper，但 Python 應作為主要 orchestration，因為 `v10_collect.py` 已是 Python。

不要重寫 `v10_collect.py`。

Operator Helper 必須包裝既有 backend。

至少支援：

```bash
python v10_operator.py preflight
python v10_operator.py prepare
python v10_operator.py status
python v10_operator.py start-case T01
python v10_operator.py finish-case T01
python v10_operator.py review T01
python v10_operator.py validate T01
python v10_operator.py summary
python v10_operator.py run-acceptance
python v10_operator.py cleanup
```

如果合理，再提供：

```bash
python v10_operator.py next
```

自動告訴操作者下一步。

---

# 六、Preflight 行為

`preflight` 必須只讀，不得改環境。

至少檢查：

- repo path
- git status
- v10_collect.py exists
- v9 runner exists
- evidence script exists
- results directory writable
- 9090 listening
- TEA 19110 healthy
- PoC 19090（若需要）
- `saas-streaming` active
- current process env
- existing systemd drop-ins
- current BARGEIN value
- current BARGEIN_MODE
- current STREAM_DEBUG
- 是否有 active calls

輸出清楚：

```text
V10 PREFLIGHT

Repo             PASS
v10_collect       PASS
9090              PASS
TEA 19110         PASS
Active calls      0
Current barge-in  OFF

READY_FOR_PREPARE = YES
```

如果任何 critical item FAIL：

`READY_FOR_PREPARE = NO`

---

# 七、Prepare 必須真正自動化

`prepare` 必須：

1. 再次確認 active calls = 0
2. snapshot 現有 systemd drop-in 狀態
3. 保存 restore manifest
4. 建立或套用 v10 shadow override
5. 保證：

```ini
Environment=SAAS_STREAM_DEBUG=1
Environment=SAAS_BARGEIN=0
Environment=SAAS_BARGEIN_MODE=shadow
```

6. `daemon-reload`
7. restart `saas-streaming`
8. 等待 ready
9. 驗證 9090 listening
10. 回讀 MainPID process env
11. 驗證 TEA 19110
12. 驗證 evidence path
13. 建立 v10 operator session state

保存：

```text
results/full_duplex_v10/operator_state.json
```

至少包含：

- prepared_at
- repo commit
- original systemd override state
- current MainPID
- 9090 status
- 19110 status
- current env
- completed cases
- pending cases
- restore data

只有全部成功才輸出：

```text
V10 LIVE TEST READY

BARGEIN     OFF
MODE        shadow
DEBUG       ON
9090        PASS
TEA         PASS

下一步：
python v10_operator.py start-case T01
```

---

# 八、Prepare 必須可重入 / 可恢復

如果 prepare 重跑：

不得疊加多份 override。

必須先讀 operator state。

如果已 prepared 且 runtime 狀態一致：

輸出：

`ALREADY_PREPARED`

不要再次 restart。

如果狀態漂移：

輸出差異並停止。

---

# 九、固定真人測試腳本

建立資料結構而不是散落硬編碼，例如：

```python
TEST_CASES = {
  "T01": {...},
  "T02": {...}
}
```

腳本如下。

## T01

### T01-A — Normal

時機：AI 第一句開始約 1 秒，句中。

真人說：

`等一下，我現在不方便。`

預期 duration：0.8～1.3s

### T01-B — Short

下一個 AI turn 開始後約 0.5～1 秒。

真人說：

`嗯`

預期 duration：80～250ms

---

## T02

### T02-A — Normal

AI 句子前段。

真人說：

`你哪裡打來的？`

0.6～1.0s

### T02-B — Short

AI 句中。

真人說：

`喂`

100～250ms

---

## T03

### T03-A — Normal

AI 句中偏後。

真人說：

`不用了，謝謝。`

0.6～1.0s

### T03-B — Short

真人說：

`不用`

150～300ms

---

## T04

### T04-A — Normal

AI 句中。

真人說：

`我現在在忙，晚點再說。`

1.0～1.5s

### T04-B — Short

真人說：

`欸`

80～200ms

---

## T05

### T05-A — Normal

真人說：

`這個是什麼服務？`

0.7～1.1s

### T05-B — Normal

另一個 AI turn。

真人說：

`你剛剛說什麼？`

0.7～1.1s

---

# 十、start-case UX 必須極簡單

執行：

```bash
python v10_operator.py start-case T01
```

Operator 必須先確認：

- prepared
- 9090 alive
- bargein=0
- shadow
- debug=1
- TEA alive
- T01 未完成
- recording/evidence 路徑 ready

然後只輸出真人需要看的資訊：

```text
==============================
CASE T01 READY
==============================

A｜Normal
AI 第一句開始約 1 秒後說：
「等一下，我現在不方便。」

B｜Short
下一個 AI turn 說：
「嗯」

注意：
- 請在 AI 正在說話時插話
- AI 不會真的停，這是正常的
- 不要因為 AI 沒停而重複台詞

完成通話後執行：
python v10_operator.py finish-case T01
```

不要在 start-case 輸出大量 debug command。

---

# 十一、Dial 行為

如果 repo 已存在：

```text
POST http://127.0.0.1:50900/api/calls/dial
```

Operator 可以：

- 準備 case name `V10-T01`
- 解析現有合法 dial workflow
- 顯示 READY TO DIAL

但不得：

- 猜電話號碼
- 從檔案輸出 private number 到 log
- 自行修改 call destination
- 在沒有使用者明確既有流程時偷偷外撥

如果 repo 已經有安全、既定、明確的測試目的地與一鍵 dial workflow，可沿用。

否則停在：

```text
READY TO DIAL T01
請使用現有測試撥號流程開始電話。
```

---

# 十二、finish-case 必須自動完成 Evidence 流程

執行：

```bash
python v10_operator.py finish-case T01
```

必須自動：

1. 找到 `V10-T01` 對應 call
2. 找到或產生 call evidence bundle
3. 如既有流程需要，呼叫：

```bash
sudo bash scripts/voip-call-evidence.sh V10-T01
```

4. 驗證 evidence bundle
5. 確認至少：

- full RX raw / wav
- debug raw（若應存在）
- streaming.log
- turns.csv
- summary.json
- version-and-params.txt
- shadow.csv（若可得）

6. 檔案不能 0 bytes
7. 自動呼叫：

```bash
python v10_collect.py --ingest <evidence_dir> --case T01
```

8. 自動呼叫：

```bash
python v10_collect.py --scan-onset T01
```

9. 建立 review bundle

如果 evidence 不完整：

`COLLECTION_INVALID`

不要把 T01 標完成。

---

# 十三、Review Bundle

每通 finish 後自動產生：

```text
results/full_duplex_v10/review/T01/
```

至少：

- onset_candidates.json
- onset_candidates.csv
- candidate WAV clips
- `README.txt` 或簡單 HTML/Markdown review page

每個 candidate clip：

- candidate 前約 1.0～1.5s
- candidate 後約 1.5～2.0s

命名：

```text
T01-candidate-01-008420ms.wav
```

讓真人不需要自己打開 full WAV 找位置。

---

# 十四、Review 命令

執行：

```bash
python v10_operator.py review T01
```

應輸出：

```text
T01 REVIEW

預期事件：
A「等一下，我現在不方便。」
B「嗯」

Onset candidates:
1. 00:08.420  clip: ...
2. 00:14.860  clip: ...
3. 00:21.120  clip: ...

請確認：
A = candidate ?
B = candidate ?
```

如果 terminal 支援互動，可以讓真人輸入：

```text
A=2
B=3
```

然後 Operator 自動計算 onset。

Offset 可以：

- 使用 energy / ASR heuristic 提議
- 但真人可以確認

不要要求真人自己算毫秒。

---

# 十五、Add Event 必須自動填大部分欄位

Operator 應從：

- turns.csv
- streaming.log
- playback node timeline
- candidate timestamp
- script TEST_CASES

自動推導：

- phrase
- type normal/short
- onset ms
- offset ms
- node
- active TTS text
- call UID

再呼叫既有：

```bash
python v10_collect.py --add-event ...
```

真人不應該手動拼這整串：

```text
--event-id --type --phrase --onset --offset --node
```

只有偵測有歧義時才要求確認。

---

# 十六、GT 時間軸規則

所有 acceptance ground truth 必須使用：

`call-start-relative timeline`

不能再重演 v8 問題。

如果同時需要 playback-relative：

另外保存：

- caller_onset_call_ms
- caller_onset_playback_ms

但 acceptance 主欄位固定 call timeline。

---

# 十七、validate case

```bash
python v10_operator.py validate T01
```

必須：

- 呼叫 v10_collect quality gate
- 驗證 RX full-call
- 驗證 onset/offset
- post-onset ≥2s
- node reference exists
- TTS text exists
- semantic logs / replay inputs exist
- GT schema valid

成功：

```text
T01 VALID
Events: 2/2
Normal: 1
Short: 1
Ready for acceptance: YES
```

失敗：

清楚列出原因，不要把 case 納入 acceptance。

---

# 十八、status / summary

`status` 顯示 runtime。

`summary` 顯示 collection progress。

例如：

```text
V10 COLLECTION STATUS

T01  VALID   Normal 1  Short 1
T02  VALID   Normal 1  Short 1
T03  READY
T04  PENDING
T05  PENDING

Valid calls: 2
Normal events: 2 / target 6
Short events: 2 / target 4

Next:
python v10_operator.py start-case T03
```

---

# 十九、run-acceptance

只有至少符合最低資料條件時才允許正式跑：

最低：

- 3 valid live calls
- Normal ≥5
- Short ≥3

建議：

- 5 valid calls
- Normal ≥6
- Short ≥4

不足時輸出：

`LIVE_COLLECTION_INSUFFICIENT`

不要硬跑最終 verdict。

資料足夠後：

1. freeze collected data
2. 記錄 hash / manifest
3. 使用既有 v9/v10 frozen logic
4. 不調參
5. 跑 replay acceptance
6. 產出 `ACCEPTANCE_REPORT_V10.md`

---

# 二十、Acceptance 必須報的核心結果

Normal Caller：

- eligible count
- TP/FN
- recall
- first non-empty ASR
- first novel transcript
- first usable semantic decision
- first stable semantic decision
- median latency
- P90 latency

Short Caller 獨立報。

Echo Safety：

- echo windows
- CALLER FP
- EARLY

Runtime：

- ASR P50/P90/P95
- queue max
- stale responses
- dropped/coalesced windows

---

# 二十一、Acceptance Gates

## PASS

至少：

- valid live calls ≥3
- eligible Normal Caller ≥5
- Echo FP = 0
- EARLY = 0
- Normal Recall ≥90%
- median first usable ≤500ms
- P90 ≤800ms

## CONDITIONAL_PASS

Normal semantic separability 明確成立，但 sample 小或部分 latency 稍高。

## SEMANTIC_SEPARABLE_BUT_TOO_SLOW

分類成立，但 P90 >1000ms。

## SEMANTIC_NOT_SEPARABLE

完整 Normal Caller 語料下仍無法和 echo 分開。

## LIVE_COLLECTION_INSUFFICIENT

真人資料仍不足。

---

# 二十二、Cleanup 必須是第一級功能

不論 acceptance 是否成功，都必須能執行：

```bash
python v10_operator.py cleanup
```

它必須：

1. 讀 restore manifest
2. 移除 v10 shadow override
3. 還原原有 drop-ins
4. daemon-reload
5. restart `saas-streaming`
6. 驗證 9090 listening
7. 回讀 production process env
8. 確認 v10 debug/shadow override 已消失
9. 保留結果檔，不刪 dataset
10. 更新 operator_state = cleaned

如果原本 production 本來就有某些 env，必須還原「原狀」，不是硬寫預設值。

---

# 二十三、Cleanup Failure 安全

如果 restore 失敗：

輸出：

`RESTORE_FAILED`

並清楚列：

- 哪個檔案
- 哪個 systemd state
- 哪個 env 不一致
- 9090 是否仍 listening

不要假裝成功。

---

# 二十四、Ctrl+C / Exception 保護

Operator Helper 在任何會修改 runtime 的操作中都必須：

- try/finally
- 保存 state
- 不因 exception 丟失 restore manifest

但不要在 `start-case` 後自動 cleanup，因為真人可能正在跑多通測試。

應提供明確：

```text
TEST SESSION ACTIVE
```

直到 `cleanup`。

---

# 二十五、不要依賴人工輸入機密

不得：

- 把電話號碼寫入 repo
- 把 SIP 密碼寫入 log
- 把 API secret 寫入結果

如果 dial flow 需要 private destination，沿用現有安全 config / account reference。

---

# 二十六、Operator Helper 自測

在真人測試前，必須做 fake/dry-run self-test。

例如：

```bash
python v10_operator.py self-test
```

要驗證：

- CLI parsing
- state save/load
- TEST_CASES
- review bundle generation
- v10_collect invocation wrapper
- invalid case handling
- repeated prepare protection
- cleanup dry-run

不得透過 self-test 真正外撥電話。

---

# 二十七、真人 Gate 前最終輸出

完成 Operator Helper 後，先不要直接要求真人測 5 通。

先輸出：

```text
V10 OPERATOR READY

Preflight: PASS
Self-test: PASS
9090: PASS
TEA: PASS
Operator helper: PASS
Runtime changed: NO（如果尚未 prepare）

真人開始時只需：

1. python v10_operator.py prepare
2. python v10_operator.py start-case T01
3. 接電話照畫面說台詞
4. python v10_operator.py finish-case T01
5. python v10_operator.py review T01

其餘由系統自動完成。
```

到這裡才算本輪完成。

---

# 二十八、不要提早收工

以下情況都不算完成：

- 只有 runbook
- 只有 shell command 範例
- 只有 `v10_collect.py`
- 只說需要真人測試
- 要真人自己下 `--ingest`
- 要真人自己下 `--add-event`
- 要真人自己找 node
- 要真人自己算 onset 毫秒
- 沒有 cleanup
- 沒有 restore verification

本輪完成條件：

> **真人之後能用少量固定 command 走完整 v10 live collection，而 evidence / ingest / onset helper / GT / validation / acceptance / restore 都由 Operator Helper 統一管理。**

---

# 二十九、交付

至少交付：

```text
_workspace/acceptance/live/ht813/v10_operator.py
```

以及必要的：

- operator state schema
- review helper
- dry-run/self-test evidence
- operator runbook 更新
- `OPERATOR_IMPLEMENTATION_REPORT.md`

報告回答：

1. operator path
2. 支援哪些 commands
3. preflight 結果
4. self-test 結果
5. prepare 是否會修改什麼
6. cleanup 如何 restore
7. 真人真正需要做幾步
8. 哪些事情已自動化
9. 哪些事情仍一定需要真人
10. 9090 是否被改 code
11. barge-in 是否仍 OFF
12. 是否有 deploy / push

---

# 三十、最終限制

- 不 push，除非原 repo 規則明確要求且已授權
- 不 deploy
- 不改 production code
- 不開 barge-in
- 不調 semantic algorithm
- 不換 ASR
- 不收假資料

本輪唯一目標：

> **把 v10 從「backend 已準備、但真人還要自己拼 command」提升成「真正可操作的一鍵真人 Live 測試流程」。**

開始執行。