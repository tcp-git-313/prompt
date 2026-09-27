# TICENPIPOST — PRODUCTION BASELINE RECONCILIATION + RECOVERY PRIORITIZATION

## ROLE
TicenpiPost Production Baseline Reconciliation Owner

## RECOMMENDED MODEL
LUNA High

## MODE
READ-ONLY RECONCILIATION + PRIORITIZATION
NO IMPLEMENTATION
NO SOURCE MODIFICATION

Project:
F:\00-Ticenpi-SaaS\TicenpiPost

Primary historical audit:
ticenpipost-full-historical-feature-trace-audit.md

Known historical-audit facts to verify, not blindly trust:

CURRENT_AUDIT_HEAD =
7c90916aafb4209e4dcfd228f63dc2a54835b7c6

CURRENT_AUDIT_BRANCH =
release/post-central-seat-shadow-staging-20260923

PRODUCTION_RUNTIME_COMMIT =
27e4efedbd91

PRODUCTION_RELEASE =
20260925-035353

Historical audit found:
- 90 tracked feature rows
- 61 still present
- 6 evolved
- 2 partially lost
- 0 regressed/lost
- 6 removed intentionally
- 11 never implemented
- 4 unknown
- 47 commits not reachable from the audited HEAD
- Marketplace implemented but incomplete in several areas

Mission:

把「歷史功能總帳」對齊到真正的：

1. Current canonical source
2. Current Staging source/runtime
3. Current Production source/runtime

然後把所有功能重新分類成：

A. 已完成、現在不要動
B. 已做但尚未完整驗證
C. 已做但未啟用 / 被 flag 關閉
D. 已做但沒有 UI / hidden / orphaned
E. 曾設計但未完成
F. 已被正式替代，不應恢復
G. Unknown，需要補證
H. 值得恢復 / 補完的功能

最後產出真正可以拿來開發的 Recovery Prioritization。

這次仍然不改程式。

---

## 0. ABSOLUTE RULES

本任務只能 READ-ONLY。

禁止：

- 修改 source
- 修改 tests
- 修改 docs
- 修改 DB
- 修改 Supabase
- 修改 Cloudflare
- 修改 deploy config
- checkout branch
- reset / restore / stash / clean
- merge / rebase / cherry-pick
- commit
- push
- deploy
- 啟用 feature flag
- 改 Production / Staging
- 執行真實 Facebook 寫入操作

允許：

- git status
- git log
- git show
- git diff
- git branch -a
- git tag
- git rev-list
- git merge-base
- git ls-tree
- git cat-file
- git grep
- 讀 CI / release metadata
- 讀 current source
- 讀 historical source
- 唯讀 GET /api/health
- 唯讀 runtime identity
- 唯讀 deploy manifest
- 讀 previous audit reports

若需要看其他 ref：
使用 git show / git diff。
不要 checkout。

---

## 1. READ THE FULL HISTORICAL AUDIT FIRST

必讀：

最新的 Full Historical Feature Trace Audit 報告。

如果報告只存在於 session / pasted file：
使用該完整內容。

如果 repo 有存檔：
優先讀 repo 版本並核對 commit。

至少取得：

- 90 feature ledger
- Marketplace deep dive
- Lost / Removed / Never Implemented
- Orphaned
- Previous audit gaps
- Historical timeline
- Unknown evidence gaps

不要重做 166 commits 的全歷史考古，
除非本次 reconciliation 發現矛盾。

本次重點是：
「歷史總帳 vs 現在真正線上 baseline」。

---

## 2. IDENTIFY CURRENT CANONICAL SOURCE

不要假設 7c90916 是現在 canonical。

確認：

- current working directory branch
- current HEAD
- origin default / active release branch
- latest Post release branch
- latest merged integration / release branch
- Production promotion branch
- Staging branch if separate
- latest remote refs

記錄：

CURRENT_WORKTREE_BRANCH =
CURRENT_WORKTREE_HEAD =

CANONICAL_SOURCE_REF =
CANONICAL_SOURCE_COMMIT =

PRODUCTION_RELEASE_REF =
PRODUCTION_SOURCE_COMMIT =

STAGING_RELEASE_REF =
STAGING_SOURCE_COMMIT =

如果 canonical 無法由 repo / deploy docs 證明：
標 UNKNOWN，不猜。

---

## 3. VERIFY PRODUCTION RUNTIME IDENTITY

唯讀查：

Production /api/health

記：

PRODUCTION_ENVIRONMENT =
PRODUCTION_RELEASE =
PRODUCTION_COMMIT =
PRODUCTION_IMAGE_DIGEST = 如果可得

將 Production commit 對回 Git。

證明：

- commit 是否存在
- 所屬 branch/ref
- 是否為 audited HEAD 的 descendant
- audited HEAD 是否為 Production commit ancestor
- Production 是否包含歷史 audit 後的額外功能 / 修復

輸出：

PRODUCTION_RELATION_TO_AUDIT_HEAD =
AHEAD_COMMITS =
BEHIND_COMMITS =
DIVERGED = YES/NO

---

## 4. VERIFY STAGING RUNTIME IDENTITY

唯讀查目前真正 Staging：

- /api/health
- environment
- release
- commit
- digest if available

不要把 local acceptance 19418 當 Staging。

輸出：

STAGING_ENVIRONMENT =
STAGING_RELEASE =
STAGING_COMMIT =
STAGING_IMAGE_DIGEST =

並對 Git refs。

---

## 5. COMPARE THREE BASELINES

建立：

A = HISTORICAL_AUDIT_HEAD
B = CURRENT_CANONICAL_SOURCE
C = PRODUCTION_RUNTIME_SOURCE
D = STAGING_RUNTIME_SOURCE

對：

A → B
A → C
A → D
B → C
B → D
C → D

做 read-only diff / commit comparison。

不要只列檔案數。

分類 changed files：

- frontend
- backend
- worker
- auth
- commercial
- seat
- tenant
- marketplace
- groups
- scheduling
- reporting
- extension
- launcher handoff
- deployment only
- tests only
- docs only

輸出：

BASELINE_DIFF_SUMMARY

---

## 6. RECONCILE THE 90-FEATURE LEDGER

這是本次核心。

把歷史 audit 的每一個 feature row
重新對到：

CURRENT_CANONICAL
STAGING
PRODUCTION

至少輸出：

| Feature | Historical Status | Canonical Source | Staging | Production | Final Classification | Evidence |

Final Classification 只能用：

COMPLETE_ACTIVE
COMPLETE_BUT_UNVERIFIED
IMPLEMENTED_FLAG_OFF
IMPLEMENTED_HIDDEN
IMPLEMENTED_ORPHANED
PARTIAL
DESIGN_ONLY
NEVER_IMPLEMENTED
REPLACED_INTENTIONALLY
UNKNOWN
PRODUCTION_ONLY
STAGING_ONLY
CANONICAL_ONLY

不要因 source 存在就寫 COMPLETE_ACTIVE。

---

## 7. MARKETPLACE — RECONCILE AGAINST PRODUCTION

Marketplace 必須獨立列。

至少核對：

- marketplace_publish action
- marketplace driver
- rental
- general listing
- address autocomplete
- publish gate
- queue
- tests
- UI entry
- /post-auction
- actual Production flag
- actual Production route availability
- actual Production runtime support

並重新判定：

MARKETPLACE_PUBLISH_RUNTIME =
MARKETPLACE_UI_ENTRY =
MARKETPLACE_RENTAL =
MARKETPLACE_GENERAL =
MARKETPLACE_ADDRESS_AUTOCOMPLETE =
MARKETPLACE_UNPUBLISH =
MARKETPLACE_ADDRESS_FAIL_CLOSED =
MARKETPLACE_GROUP_COMBINED =
MARKETPLACE_REAL_PUBLISH_VERIFIED =

注意：
「source 有」≠「Production 已啟用」。

若 VPS env 真值無法讀：
標 UNVERIFIED。

---

## 8. AUTOMATION / SCHEDULE RECOVERY

獨立核對：

- daily_comment
- relist
- cleanup
- first-publish state machine
- automations page
- run-center
- schedule
- calendar
- batch rule management
- hot-comment automation

區分：

1. backend implemented
2. frontend entry exists
3. nav entry exists
4. runtime enabled
5. verified E2E

避免把 hidden page 當未做。

---

## 9. GROUP FEATURES RECONCILIATION

核對：

- joined-group sync
- known_groups
- categories
- group join
- membership questions
- retry/review
- group search
- AI classification

特別確認：

前一輪 audit 說：
group_search = NEVER_IMPLEMENTED

本輪確認 Production / Staging 有沒有後續 commit 新增。

---

## 10. ACCOUNT / SURVIVAL FEATURES

核對：

- budget
- warmup
- checkpoint
- circuit breaker
- account lock
- device token
- Studio heartbeat
- Extension
- Launcher handoff

標記：

ACTIVE
PARTIAL
FLAG_OFF
KNOWN_GAP

ADR-003 device token gap 要列為 correctness/security gap，
不要把它當 feature request。

---

## 11. COMMERCIAL / TENANT / SEAT

核對：

- Commercial gate
- Central Seat
- Tenant bootstrap
- Seat shadow
- Seat authoritative
- flags
- Staging status
- Production status

歷史 audit 可能基於 shadow HEAD。

現在 Production 已經是 9/25 release，
必須重新確認：

CENTRAL_SEAT_CURRENT_STATE =
COMMERCIAL_GATE_CURRENT_STATE =
PRODUCTION_SEAT_POLICY =
STAGING_SEAT_POLICY =

如果 repo config 與 runtime env 不同：
兩者分開報。

---

## 12. HIDDEN / ORPHANED ROUTES

歷史 audit 找到：

- /automations
- /relist
- /591
- /schedule
- /post-group
- /multi-account-schedule
- /post-auction

逐一核對：

ROUTE_EXISTS =
NAV_VISIBLE =
LINKED_FROM_UI =
BACKEND_SUPPORTED =
PRODUCTION_REACHABLE =
INTENTIONAL_HIDDEN =
RECOVERY_VALUE =

不要直接建議全部加回 nav。

---

## 13. REMOVED FEATURES — DO NOT RESURRECT BLINDLY

核對：

- Workbench
- MobileComposer
- PreviewCard / PreviewDrawer
- old GroupSelector
- local-mode.ps1
- 591 extractor

如果已有替代：

標：

REPLACED_INTENTIONALLY

不要放進 recovery backlog。

---

## 14. NEVER IMPLEMENTED — SEPARATE FROM RECOVERY

以下不得混成「遺失功能」：

- LINE / LIFF
- PWA
- payment
- group search
- marketplace unpublish
- calendar
- batch rules
- hot comment automation
- address fail-closed

若 current source 仍沒有：
標 DESIGN_ONLY / NEVER_IMPLEMENTED。

---

## 15. BUILD THE ACTIONABLE 4-BUCKET SUMMARY

把所有功能壓成四大桶：

### BUCKET A — DONE / LEAVE ALONE

Production 已存在且沒有已知 correctness gap。

### BUCKET B — IMPLEMENTED BUT NEEDS VERIFICATION

例如：
Marketplace 真實 publish click
或 source 有但 E2E 沒驗。

### BUCKET C — IMPLEMENTED BUT NOT EXPOSED / DISABLED

例如：
hidden route
feature flag off
Staging-only
shadow-only

### BUCKET D — NOT IMPLEMENTED / NEW WORK

例如：
Marketplace unpublish
group search
LINE/LIFF
PWA
calendar

---

## 16. PRIORITIZATION RULES

本次可以排序，但不能開始實作。

排序基準：

1. correctness / security blocker
2. 已有 70%以上 source，可低成本完成
3. 使用者已明確需要
4. 會解除多個其他功能依賴
5. Production 可直接受益
6. 新功能才放後面

不要因為 LINE 是新方向就自動排第一。

不要因為 Marketplace 有 source 就自動判定已完成。

---

## 17. RECOVERY PRIORITY MATRIX

每個值得處理的功能輸出：

FEATURE =
CURRENT_STATE =
PRODUCTION_STATE =
WHY_NOW =
MISSING =
DEPENDENCIES =
RISK = LOW/MEDIUM/HIGH
SCOPE = S/M/L/XL
CAN_RUN_IN_PARALLEL = YES/NO
RECOMMENDED_MODEL =
RECOMMENDED_NEXT_ACTION = VERIFY / ENABLE / COMPLETE / REDESIGN / LEAVE_ALONE

---

## 18. IDENTIFY FAST WINS

找出：

FAST_WINS

條件：

- source 已存在
- 不需架構重做
- 不碰核心 auth
- 不需 DB migration 或只有小 migration
- 可快速驗證 / 啟用

例如：
hidden route
existing feature flag
missing nav
existing driver not verified

但必須有 evidence。

---

## 19. IDENTIFY HIGH-RISK ITEMS

列：

HIGH_RISK_ITEMS

例如：

- mobile FB binding
- auth identity changes
- device-token revocation
- Marketplace real publish
- Central Seat Production enablement
- LINE identity linking

說明為什麼高風險。

---

## 20. SAFE PARALLELIZATION PLAN

最後給出下一輪可以安全平行的 task groups。

規則：

同一組不能：

- 同時改同一核心檔案
- 同時改 auth/identity
- 同時改同一 DB schema
- 同時動同一 deploy manifest
- 同時 deploy Production

至少列：

PARALLEL_GROUP_A =
PARALLEL_GROUP_B =
SERIAL_CHAIN =

每個 group 明確列：
FILES / AREAS
WHY_SAFE
MERGE_ORDER

---

## 21. OUTPUT — FINAL PRIORITIZED BACKLOG

只列 10～15 個真正值得做的項目。

不要把 90 個全塞進 backlog。

每項：

PRIORITY =
TASK =
CATEGORY =
CURRENT =
TARGET =
WHY =
DEPENDENCY =
RISK =
SCOPE =
MODEL =
PARALLEL_GROUP =

Priority：

P0
P1
P2
P3

P0 只放 correctness/security/release blocker。

---

## 22. OUTPUT — TOP 5 NEXT EXECUTION TASKS

最後只挑 5 個下一步。

格式：

1.
TASK_NAME =
WHY_NOW =
CURRENT_EVIDENCE =
EXPECTED_RESULT =
DEPENDENCIES =
MODEL =
THINKING =
CAN_RUN_PARALLEL =
RISK =

不要寫 Prompt。
只列任務。

下一輪再由使用者指定哪些要開 Prompt。

---

## 23. REPORT FORMAT

PHASE = TICENPIPOST PRODUCTION BASELINE RECONCILIATION
RESULT = PASS / PARTIAL / HARD_STOP

A. BASELINES
AUDIT_HEAD =
CANONICAL_SOURCE =
STAGING_SOURCE =
PRODUCTION_SOURCE =
PRODUCTION_RUNTIME =
STAGING_RUNTIME =

B. RELATIONSHIPS
AUDIT_TO_CANONICAL =
AUDIT_TO_PRODUCTION =
CANONICAL_TO_PRODUCTION =
STAGING_TO_PRODUCTION =

C. FEATURE RECONCILIATION
TOTAL_FEATURES =
COMPLETE_ACTIVE =
COMPLETE_BUT_UNVERIFIED =
IMPLEMENTED_FLAG_OFF =
IMPLEMENTED_HIDDEN =
IMPLEMENTED_ORPHANED =
PARTIAL =
DESIGN_ONLY =
NEVER_IMPLEMENTED =
REPLACED_INTENTIONALLY =
UNKNOWN =

D. MARKETPLACE
MARKETPLACE_PRODUCTION_STATUS =
MARKETPLACE_GAPS =

E. COMMERCIAL
COMMERCIAL_GATE =
CENTRAL_SEAT =
TENANT =
DEVICE_TOKEN_GAP =

F. RECOVERY
FAST_WINS =
HIGH_RISK_ITEMS =
PRIORITIZED_BACKLOG =

G. PARALLELIZATION
PARALLEL_GROUP_A =
PARALLEL_GROUP_B =
SERIAL_CHAIN =

H. TOP_5_NEXT_TASKS
1.
2.
3.
4.
5.

I. FINAL
READY_FOR_EXECUTION_PROMPTS = YES/NO

No source changes.
No commit.
No push.
No deploy.
