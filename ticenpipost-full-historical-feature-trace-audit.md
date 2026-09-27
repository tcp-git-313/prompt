# TICENPIPOST — FULL HISTORICAL FEATURE TRACE AUDIT

## ROLE
TicenpiPost Historical Product Forensics Auditor

## RECOMMENDED MODEL
LUNA High

## MODE
READ-ONLY FORENSIC AUDIT — NO IMPLEMENTATION

Project:
F:\00-Ticenpi-SaaS\TicenpiPost

Primary goal:

把 TicenpiPost 從「最早討論 / 設計 / 實作 / 上線前趕工 / 上線後修補」一路到現在的所有功能脈絡重新追出來。

這次不是從「現在程式有哪些」往外猜，而是要從：
- Git history
- Git branches / tags / old commits
- 專案內 MD / ADR / handoff / README / TODO
- Claude project memory
- GitHub prompt repo
- deploy handoff / release docs
- 歷史測試
- migration
- dead code / hidden routes / feature flags

反向追查：

1. 以前明確討論過哪些功能？
2. 哪些曾經做過？
3. 哪些只做到一半？
4. 哪些上線前被砍掉、隱藏、停用、或沒有接 UI？
5. 哪些舊功能現在已被新架構取代？
6. 哪些功能其實存在 backend，但 UI 沒入口？
7. 哪些 UI 有入口，但 backend / worker 沒完成？
8. 哪些曾在舊 commit / branch 存在，但目前 HEAD 已消失？
9. Marketplace / 拍賣是否曾有完整設計、部分實作或歷史 commit？
10. 還有哪些我們現在已經忘記，但有實際歷史證據的功能？

最後要產出一份「完整歷史功能總帳」，不是產品 wishlist。

---

## 0. ABSOLUTE RULES

本任務只能 READ-ONLY。

禁止：
- 修改 source
- 修改 tests
- 修改 MD
- 修改 DB
- 修改 Supabase
- 修改 Cloudflare
- commit / push
- checkout 到其他 branch
- reset / restore / clean / stash / rebase / merge / cherry-pick
- deploy
- 建 migration
- 執行會產生真實 Facebook 寫入的測試

允許：
- git status
- git log
- git show
- git diff
- git branch -a
- git tag
- git reflog --all
- git grep
- git rev-list
- git ls-tree
- git cat-file
- git blame
- git log -S
- git log -G
- 讀 source / tests / docs / memory
- 讀 GitHub prompt repo
- 唯讀 health / runtime GET
- 讀 CI / release evidence
- 讀 deploy handoff

如果需要看其他 branch 的檔案：
不要 checkout。
使用 git show <ref>:<path> 或其他 read-only Git 方法。

---

## 1. EVIDENCE STANDARD — 最重要

每一個「歷史功能」結論都必須有實際 evidence。

不能只寫：
「以前好像討論過」
「記憶中有這功能」

至少要有以下一種 PRIMARY EVIDENCE：

### Git evidence
例如：
- commit SHA
- branch/ref
- tag
- historical file path
- deleted file path
- commit diff
- git log -S / -G hit
- old test
- old migration

每項至少記：
GIT_EVIDENCE:
commit=<sha>
path=<path>
symbol_or_lines=<function/component/keyword>
what_it_proves=<...>

### Document evidence
例如：
- project MD
- ADR
- charter history
- handoff
- spec
- roadmap
- prompt
- memory file

每項至少記：
DOC_EVIDENCE:
path_or_url=<...>
heading=<...>
what_it_proves=<...>

### Runtime / test evidence
例如：
- current route
- current test
- feature flag
- health
- UI route
- worker symbol

---

## 2. EVIDENCE CONFIDENCE

每個 feature 必須標記：

PROVEN
STRONG
WEAK
UNVERIFIED

定義：

PROVEN =
有 source / commit / test / migration 等直接 evidence。

STRONG =
有多份 MD / prompt / handoff 一致記錄，但沒有找到實作。

WEAK =
只有單一歷史文字提及。

UNVERIFIED =
使用者或 memory 提過，但本次沒有找到實際檔案 / Git evidence。

不得把 WEAK / UNVERIFIED 寫成「已做過」。

---

## 3. CURRENT GROUND TRUTH

先記錄：

REPO =
CURRENT_BRANCH =
CURRENT_HEAD =
WORKTREE_STATUS =
REMOTE_REFS =
TAGS =

不要改工作樹。

如果工作樹 dirty：
記錄即可。
不要因此停止整個 historical audit，除非 dirty changes 真的讓 current source 無法判讀。

對 current source 建 baseline inventory：

- frontend routes
- frontend components
- backend routers
- services
- workers
- drivers
- models
- migrations
- tests
- feature flags
- env switches
- deploy config

這只是「現在」，不是歷史結論。

---

## 4. PROJECT INTERNAL HISTORY

遞迴搜尋並讀取 TicenpiPost repo 中所有可能保存產品歷史的文件：

README*
docs/**
_charter/**
adr/**
history*
handoff*
roadmap*
spec*
design*
todo*
notes*
release*
deploy*
*.md
*.txt

不要只列檔名，要真正讀內容。

搜尋關鍵字至少：

feature
功能
拍賣
Marketplace
marketplace
商品
刊登
房屋
出租
出售
vehicle
car
group
社團
join
search
留言
comment
互動
engagement
養號
warmup
刪文
delete
schedule
排程
report
dashboard
文案
copywriting
AI
template
account
帳號
listing
extract
URL
591
YUCT
Facebook
Launcher
Extension
LINE
LIFF
mobile
PWA
通知
notification
retry
review
checkpoint
budget
circuit
queue
worker
bulk
batch
location
address
autocomplete
resolver

---

## 5. CLAUDE PROJECT MEMORY

Root:
C:\Users\Tu\.claude\projects\F--\memory\

必讀：
MEMORY.md

然後遞迴搜尋所有與 Post / Ticenpi / Facebook 相關記錄。

尤其：
- project_post_*
- reference_post_*
- Facebook / Marketplace / auction / group
- Launcher
- Extension
- listing
- 591 / YUCT
- mobile
- LINE / LIFF
- posting
- task
- schedule
- old architecture

每個 relevant memory 記：

MEMORY_FILE =
TOPICS =
HISTORICAL_CLAIMS =
CAN_BE_CORROBORATED_BY_GIT = YES/NO

Memory 只能當線索。
最終功能狀態不能只靠 memory。

---

## 6. GITHUB PROMPT REPOSITORY

Repository:
tcp-git-313/prompt

全面搜尋與 TicenpiPost 有關的 prompt / spec / audit / design。

至少搜尋：

Post
TicenpiPost
Facebook
Marketplace
拍賣
marketplace
社團
group
LINE
LIFF
591
YUCT
listing
posting
schedule
comment
養號
報表
dashboard
account
mobile
PWA
Launcher
Extension

已知必讀：
fb-group-feature-design.md

但本次不能只依賴它。

每個 prompt 記：
- file
- commit if available
- original task goal
- whether implemented
- corresponding current source
- historical source evidence

---

## 7. GIT HISTORY FORENSICS

這是本次核心。

不要只看最近 20 commits。

### 7.1 Full commit subjects

掃完整 Git history：
git log --all --date=iso --decorate --stat

建立 major feature timeline。

### 7.2 Keyword archaeology

用 git log -S 與 git log -G 搜尋功能詞。

至少做以下 archaeology。

#### Marketplace / 拍賣
Marketplace
marketplace
拍賣
auction
listing_type
marketplace_type
housing
property
rent
sale
vehicle
car
truck
motorcycle
address
autocomplete
location
resolver
ADDRESS_MISMATCH
fb_address
marketplace/address

#### Facebook groups
group
groups
GroupSelector
group_join
join_group
group_search
category
known_groups
membership_questions

#### Engagement / account automation
comment
reply
like
engagement
warmup
養號
feed
delete_post
刪文

#### Scheduling / batch
schedule
batch
bulk
queue
repeat
cron
calendar

#### Reporting / operations
report
analytics
dashboard
tracking
result
history
audit
review
monitor

#### Content / extraction
copywriting
template
prompt
AI
listing
extract
591
YUCT
URL
OCR
image

#### Mobile / LINE
mobile
MobileComposer
PWA
manifest
service worker
LINE
LIFF

記錄每個 significant hit。

---

## 8. SEARCH ALL REFS, NOT JUST CURRENT BRANCH

列出：

git branch -a
git tag

檢查 remote / historical refs。

對有產品功能意味的 branch：
不要 checkout。

用：
git log <ref>
git diff <current>...<ref>
git ls-tree -r <ref>

找出：
- current HEAD 不存在的功能檔案
- old experimental implementations
- abandoned pages
- feature branches never merged
- partially merged work

如果存在已刪 branch 的 local reflog evidence：
記錄，但不要 mutation。

---

## 9. LOST / REGRESSED FEATURE DETECTION

對歷史 evidence 中「曾有 source」的功能，檢查 current HEAD。

分類：

STILL_PRESENT
= 歷史功能目前還在。

EVOLVED
= 舊功能仍存在，但已換新架構 / 新名稱。

PARTIALLY_LOST
= 部分核心仍在，但 UI / route / worker / test 消失。

REMOVED_INTENTIONALLY
= 有明確 ADR / commit message / replacement evidence 證明刻意移除。

REGRESSED_OR_LOST
= 歷史上有明確實作，current HEAD 已無，且找不到正式替代。

NEVER_IMPLEMENTED
= 只有設計 / prompt，Git 從未出現實作 evidence。

不得把 NEVER_IMPLEMENTED 跟 LOST 混在一起。

---

## 10. MARKETPLACE / 拍賣 — MANDATORY DEEP DIVE

這次必須獨立做完整章節。

不要因 current source 沒看到 Marketplace 就結束。

追查：

- 以前是否設計 Marketplace 發布？
- 是否有普通商品 / 房屋出售 / 房屋出租 / 車輛等 publication type？
- 是否有 Marketplace + Group 同步發布設計？
- 是否有地址輸入 / FB autocomplete / address resolver？
- 是否有 Facebook 真實 suggestion 解析？
- 是否有 address mismatch fail-closed？
- 是否有 marketplace driver？
- 是否有 marketplace task type？
- 是否有 marketplace UI？
- 是否有 tests？
- 是否有 old branch / deleted source？
- 是否有 prompt / memory / MD？
- 是否有 feature flag 關閉？
- 是否曾因趕上線而暫時拿掉？

強制搜尋：

Marketplace
marketplace
拍賣
刊登
market listing
address resolver
autocomplete
ADDRESS_MISMATCH
fb_address
source_address
fb_address_query
fb_address_label
fb_address_rank
fb_address_resolved_at

如果找到：
建立 MARKETPLACE_TIMELINE：

DATE
COMMIT / DOC
FEATURE
STATUS_AT_THAT_TIME
CURRENT_STATUS

如果完全找不到 Git evidence：
明確寫：

MARKETPLACE_GIT_EVIDENCE = NOT_FOUND

並列出實際搜尋範圍。
不能說「以前沒做過」。

---

## 11. OTHER MANDATORY HISTORICAL FEATURE FAMILIES

除了 Marketplace，至少逐一追查：

### Posting
- 一般發文
- Marketplace
- 社團發文
- 多社團
- 多帳號
- 批次
- 排程
- 草稿
- preview
- retry
- delete post

### Facebook Account Operations
- account binding
- cookie import
- account verification
- warmup / 養號
- comment
- like / reaction
- reply
- page/profile actions
- checkpoint recovery
- account rotation

### Groups
- joined group sync
- category
- GroupSelector
- group search
- AI group classification
- join
- questions
- review/retry

### Content / AI
- copywriting
- templates
- AI rewrite
- images
- listing extraction
- URL paste
- multi-listing
- OCR / image extraction references if any

### Scheduling / Queue
- queue
- schedule
- recurring
- task priority
- concurrency
- account lock
- budget
- circuit breaker

### Reporting
- post result
- post URL
- failure reason
- dashboard
- performance
- history
- audit
- reports

### Mobile
- mobile composer
- mobile selection
- mobile upload
- mobile account management
- PWA

### External Channels
- Launcher
- Extension
- LINE
- LIFF
- notification

---

## 12. SOURCE EXISTS BUT NOT WIRED

搜尋高價值情況：

### Orphan frontend
Component 存在，但沒有 route/import/navigation entry。

### Orphan backend
Router/service 存在，但沒有 include_router 或 call site。

### Feature flag off
功能完整但預設永遠 false。

### Hidden nav
route 存在但 UI 不顯示。

### Mock-only
tests / mocks 有功能，runtime 不存在。

### Partial worker
job enqueue 有，worker terminal state 不完整。

### Migration-only
DB schema 有欄位，應用層沒使用。

建立：
ORPHANED_FEATURES

清單。

---

## 13. TEST HISTORY

Tests 往往保存被忘記的功能。

搜尋 current + historical tests。

找：
- deleted tests
- skipped tests
- xfail
- TODO tests
- fixtures for features current UI 不存在
- test names that imply forgotten capability

每個 historical test 要對 current runtime 查證。

---

## 14. MIGRATION HISTORY

掃全部 migrations。

找出曾規劃的 model / capability。

例如：
- marketplace columns
- group discovery
- comment tasks
- account state
- report fields
- schedules
- templates
- notification channels
- mobile/line identity

如果 migration schema 還存在但 source 沒用：
標：
SCHEMA_ORPHAN

---

## 15. RELEASE / DEPLOY HISTORY

讀：
- deploy docs
- PRODUCT_TO_STAGING
- handoff
- release notes
- image identity
- rollback evidence

找出曾因：
- 上線時程
- production risk
- unfinished UI
- unstable Facebook DOM
- auth blocker
- migration blocker

而延後的功能。

只有有 evidence 才可以寫「因趕上線被延後」。

---

## 16. MASTER HISTORICAL FEATURE LEDGER

最終至少 50 個 feature rows。

表格：

| ID | Feature | Family | Earliest Evidence | Strongest Evidence | Historical State | Current State | Classification | Confidence | Missing / Replacement | Next Action |

Historical State：
DESIGNED
PARTIAL_IMPLEMENTATION
IMPLEMENTED
SHIPPED
UNKNOWN

Current State：
IMPLEMENTED
PARTIAL
HIDDEN
ORPHANED
REMOVED
REPLACED
NOT_FOUND

Classification：
STILL_PRESENT
EVOLVED
PARTIALLY_LOST
REMOVED_INTENTIONALLY
REGRESSED_OR_LOST
NEVER_IMPLEMENTED
UNKNOWN

---

## 17. FEATURE TIMELINE

建立時間軸：

DATE
↓
feature decisions
↓
commit(s)
↓
what changed
↓
what remains today

至少列重大事件：
- earliest Post architecture
- FB account binding evolution
- group system
- Marketplace
- queue / scheduler
- extraction
- Launcher
- Extension
- commercial auth / tenant
- Central Seat
- mobile
- LINE / LIFF

---

## 18. POSSIBLY CUT / DEFERRED FOR LAUNCH

獨立輸出：

POSSIBLY_CUT_FOR_LAUNCH

只允許列有 evidence 的項目。

每項：

FEATURE =
EVIDENCE =
WAS_IMPLEMENTED = YES/NO/PARTIAL
WHY_BELIEVED_CUT_OR_DEFERRED =
CURRENT_RECOVERY_COST = S/M/L/XL

如果沒有證據證明是「趕上線砍掉」：

不要這樣寫。

改寫：

DEFERRED_OR_UNFINISHED_REASON = UNKNOWN

---

## 19. FORGOTTEN BUT VALUABLE

列出：

FORGOTTEN_FEATURE_CANDIDATES

條件：
- 有歷史 evidence
- 不在目前主流程
- current audit / roadmap 沒列
- 對產品仍有合理價值

優先列：
Marketplace / 拍賣
以及所有其他被前一份 Feature Gap audit 漏掉的項目。

---

## 20. RECONCILE WITH PREVIOUS FEATURE GAP AUDIT

讀上一份：

ticenpipost-feature-gap-web-pwa-liff-roadmap-audit.md

以及它的報告（如果已保存）。

建立：

PREVIOUS_AUDIT_MISSED
= 前一份漏掉哪些歷史功能？

PREVIOUS_AUDIT_INCORRECT
= 前一份哪些 status 被本次 Git evidence 推翻？

PREVIOUS_AUDIT_CONFIRMED
= 哪些仍成立？

Marketplace 必須在這裡明確處理。

---

## 21. DO NOT PLAN IMPLEMENTATION YET

這一輪只做 historical recovery。

不要直接決定：
- 要全部恢復
- Marketplace 一定要做
- LINE 要先做
- Group Search 要先做

先把歷史真相整理完整。

Implementation prioritization 是下一輪。

---

## 22. FINAL REPORT STRUCTURE

固定輸出：

### A. AUDIT COVERAGE

CURRENT_HEAD =
BRANCHES_CHECKED =
TAGS_CHECKED =
GIT_HISTORY_RANGE =
MEMORY_FILES_READ =
PROJECT_DOCS_READ =
PROMPT_FILES_READ =
MIGRATIONS_CHECKED =
TEST_HISTORY_CHECKED =
DEPLOY_HISTORY_CHECKED =

### B. MARKETPLACE DEEP DIVE

MARKETPLACE_DISCUSSION_EVIDENCE =
MARKETPLACE_GIT_EVIDENCE =
MARKETPLACE_OLD_SOURCE =
MARKETPLACE_OLD_TESTS =
MARKETPLACE_CURRENT_SOURCE =
MARKETPLACE_CURRENT_STATUS =
MARKETPLACE_TIMELINE =

### C. MASTER FEATURE LEDGER

至少 50 rows。

### D. LOST / REGRESSED

REGRESSED_OR_LOST_COUNT =
PARTIALLY_LOST_COUNT =
REMOVED_INTENTIONALLY_COUNT =

逐項 evidence。

### E. NEVER IMPLEMENTED

只設計但從未落地的功能。

### F. ORPHANED

ORPHANED_FRONTEND =
ORPHANED_BACKEND =
SCHEMA_ORPHAN =
FLAG_DISABLED =

### G. POSSIBLY CUT / DEFERRED FOR LAUNCH

只列有 evidence 者。

### H. PREVIOUS AUDIT GAPS

PREVIOUS_AUDIT_MISSED =
PREVIOUS_AUDIT_INCORRECT =
PREVIOUS_AUDIT_CONFIRMED =

### I. FORGOTTEN FEATURE CANDIDATES

依 evidence strength 排序，不是依主觀喜好排序。

### J. HISTORICAL TIMELINE

完整重大功能演變。

### K. UNKNOWN / EVIDENCE GAPS

哪些 memory / discussion 有提，但 Git / MD 沒找到？
明確保留 UNKNOWN。

---

## 23. MACHINE-READABLE SUMMARY

最後一定輸出：

PHASE = TICENPIPOST FULL HISTORICAL FEATURE TRACE
RESULT = PASS / PARTIAL / HARD_STOP

CURRENT_HEAD =
GIT_COMMITS_REVIEWED =
BRANCH_REFS_REVIEWED =
DOCS_REVIEWED =
MEMORY_FILES_REVIEWED =
PROMPTS_REVIEWED =

FEATURES_TOTAL =
FEATURES_STILL_PRESENT =
FEATURES_EVOLVED =
FEATURES_PARTIALLY_LOST =
FEATURES_REGRESSED_OR_LOST =
FEATURES_REMOVED_INTENTIONALLY =
FEATURES_NEVER_IMPLEMENTED =
FEATURES_UNKNOWN =

MARKETPLACE_FOUND_IN_DOCS = YES/NO
MARKETPLACE_FOUND_IN_GIT = YES/NO
MARKETPLACE_CURRENT_STATUS =

PREVIOUS_AUDIT_MISSED_COUNT =

READY_FOR_RECOVERY_PRIORITIZATION = YES/NO

---

## 24. STOP CONDITION

只在以下情況 HARD STOP：

- Git repo 無法讀
- history 被 shallow clone 截斷且無法取得遠端 refs
- 必須 mutation 才能取得關鍵 evidence
- 主要歷史資料來源不可用

不要因：
- worktree dirty
- stale ADR
- 某一 feature 找不到
- current source 沒有 Marketplace

就停止整個 audit。

找不到就標 NOT_FOUND / UNKNOWN，繼續追其他歷史功能。

---

## 25. FINAL PRINCIPLE

本次最重要原則：

不是問：
「現在還能加什麼功能？」

而是問：
「我們以前到底規劃過、做過、刪過、做一半過什麼？
有什麼可以被 Git / MD / Prompt / Test / Migration 真正證明？」

任何沒有 evidence 的功能，
都不能寫成歷史事實。

任何在 Git 歷史中曾存在、
但現在消失的功能，
都必須被列出並解釋。

尤其 Marketplace / 拍賣，
本次必須做獨立深挖。
