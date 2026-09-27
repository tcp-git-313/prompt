# TICENPIPOST — FEATURE GAP / WEB APP / LINE LIFF ROADMAP AUDIT

## ROLE
TicenpiPost Product Recovery + Architecture Auditor

## MODEL
LUNA High

## MODE
READ-ONLY INVESTIGATION FIRST — NO IMPLEMENTATION

Project:
`F:\00-Ticenpi-SaaS\TicenpiPost`

Primary design reference:
`https://raw.githubusercontent.com/tcp-git-313/prompt/main/fb-group-feature-design.md`

Mission:

這個專案已上線一段時間，但當初為了趕上線，只完成基礎功能。
現在要把「以前討論過、設計過、記錄過、但尚未完整實作」的功能重新盤點出來，
並用目前實際程式碼、測試、部署現況、歷史文件與官方 LINE / LIFF 文件做交叉驗證。

本任務不是直接改程式。

本任務要回答：

1. 現在 Production / Staging / Local 到底已經有哪些功能？
2. 以前討論過哪些功能，但目前還沒做、只做一半、或已被架構變更取代？
3. `fb-group-feature-design.md` 中每一項目前實作到哪裡？
4. TicenpiPost 現在到底算：
   - 一般 Web App
   - SPA / SSR Web
   - Mobile Web
   - PWA
   - Installable Web App
   - LIFF-compatible Web App
   哪一種？
5. 如果未來要從 LINE 官方帳號貼網址進入：
   - 直接開 Web URL 有什麼限制？
   - 使用 LIFF 有什麼要求？
   - Google / Supabase Auth 是否能正常工作？
   - Central Customer / Seat / Tenant 如何對應 LINE userId？
   - Launcher / Chrome Extension 在 LINE WebView / 手機上不能使用時，Facebook 流程如何處理？
6. 哪些功能可以直接在現有架構上補？
7. 哪些功能需要新增 backend / queue / database / LINE channel integration？
8. 哪些舊設計已不再適用，不能照舊做？
9. 最後整理成一份可執行的 Product Roadmap，但本次不要實作。

你是 Auditor / Planner，不是 Executor。
不得為了讓現況符合舊 MD 而直接修改程式。

==================================================
0. ABSOLUTE RULES
==================================================

本任務 READ-ONLY。

禁止：

- 修改 source
- 修改 DB
- 修改 Supabase
- 修改 Cloudflare / DNS
- 修改 LINE Developer Console
- 修改 Production / Staging
- commit
- push
- deploy
- 建 migration
- 建新 API
- 改 UI
- 刪除舊程式
- 直接把舊設計當成現行事實

允許：

- 讀 source
- 讀 tests
- 讀 migration
- 讀 repo docs
- 讀本機 memory / handoff
- 讀 Git history
- 讀部署設定
- 對 Local / Staging / Production 做非破壞性的 GET / health / UI 檢查
- 查官方 LINE / LIFF 文件
- 查官方 Supabase / browser / PWA 文件
- 建立差異報告

若任何資訊無法證明：

標示：
UNKNOWN / UNVERIFIED

不得猜。

==================================================
1. READ PRIOR RECORDS FIRST
==================================================

先讀：

`C:\Users\Tu\.claude\projects\F--\memory\MEMORY.md`

並搜尋 memory 目錄所有與以下關鍵字相關的文件：

- Post
- Facebook
- FB
- group
- GroupSelector
- LINE
- LIFF
- Launcher
- Extension
- mobile
- PWA
- Web App
- Facebook worker
- cookie
- queue
- scheduled post
- extractor
- URL
- listing

不要只讀檔名。
真正打開相關內容。

另外讀：

`F:\00-Ticenpi-SaaS\TicenpiPost\_charter\history.md`

`F:\00-Ticenpi-SaaS\TicenpiPost\_charter\adr\ADR-001-tenant-identity-entry-point.md`

`F:\00-Ticenpi-SaaS\TicenpiPost\_charter\adr\ADR-002-central-commercial-post-tenant-seat.md`

`F:\00-Ticenpi-SaaS\TicenpiPost\_charter\adr\ADR-003-device-token-entitlement-revalidation-gap.md`

以及 repo 中：

- README
- docs
- _charter
- deploy docs
- product requirement / handoff / design / TODO
- relevant Git history

==================================================
2. REQUIRED PRIOR DESIGN REFERENCES
==================================================

必讀：

`fb-group-feature-design.md`

URL:
`https://raw.githubusercontent.com/tcp-git-313/prompt/main/fb-group-feature-design.md`

該文件已知設計方向包括：

- 已加入社團清單
- 真實 Facebook 社團搜尋
- Group Management
- GroupSelector 分離
- AI 分類 / relevance
- 加入社團 Queue
- join terminal state
- 入社問題 QUESTIONS_REQUIRED
- retry / review queue
- 帳號風險控制
- Account Lock
- 每日行為預算
- Checkpoint
- Circuit Breaker
- evidence

不要假設這些都已完成。

逐項查實。

同時搜尋 `tcp-git-313/prompt` 是否還有：

- LINE / LIFF
- Post mobile
- FB group
- Facebook execution
- Launcher
- Extension
- account binding
- URL extraction

相關設計文件。

如果存在：
列入 evidence。

==================================================
3. RECONSTRUCT PREVIOUSLY DISCUSSED PRODUCT VISION
==================================================

從 memory + docs + Git history 重建先前討論過的功能。

至少核對以下已知曾討論方向：

A. Post 核心流程

- Google / Supabase login
- Commercial Auth
- Central Customer / Membership / Entitlement / Seat
- Post Tenant
- Facebook account binding
- Launcher primary
- Extension fallback
- 貼網址 / Listing extraction
- Group selection
- Queue posting
- scheduled posting
- task/report/result

B. Facebook group extension

- 已加入社團同步
- 社團 URL
- GroupSelector
- Facebook 真實搜尋
- AI 分類
- 探索新社團
- 自動 / 半自動加入
- 入社問題
- 終態追蹤
- risk control

C. LINE / LIFF

已知曾討論方向：

```text
LINE Official Account
↓
LIFF Web App
↓
LINE userId ↔ Google User / Ticenpi User / Tenant
↓
貼物件 URL
↓
擷取資料
↓
選 Facebook 帳號
↓
選社團
↓
預覽
↓
送出 Post Job
↓
進度 / 結果
```

同時曾定義：

LIFF = 操作介面
不是 Facebook executor。

需實際確認目前 source 是否已存在任何 LINE / LIFF groundwork。

==================================================
4. CURRENT SOURCE INVENTORY
==================================================

實際盤點目前 source。

至少找出：

### Frontend

- framework / build system
- routes/pages
- mobile layout
- responsive breakpoints
- auth gate
- Post composer
- URL input
- Listing preview/edit
- Facebook account UI
- GroupSelector
- group management
- tasks
- reports
- settings
- Launcher detection
- Extension bridge
- PWA manifest
- service worker
- web app manifest
- install prompt
- standalone display config
- offline handling

### Backend

- auth
- commercial auth
- tenant
- seat
- binding
- listing/extraction
- account
- group APIs
- posting
- tasks
- queue
- scheduler
- worker
- Facebook driver
- group search driver
- group join driver
- retry
- evidence
- risk controls
- checkpoint
- circuit breaker
- budget

### DB

找出與：

- account
- group
- posting
- task
- schedule
- tenant
- bind/device
- listing
- group discovery
- group join

相關 tables / migrations。

==================================================
5. FEATURE STATUS MATRIX
==================================================

對所有找到的功能建立矩陣。

Status 只能用：

IMPLEMENTED
PARTIAL
DESIGN_ONLY
NOT_FOUND
REPLACED
DEPRECATED
UNKNOWN

每一列至少：

FEATURE
PRIOR_REFERENCE
CURRENT_SOURCE
CURRENT_TEST
RUNTIME_EVIDENCE
STATUS
MISSING_PART
NOTES

至少覆蓋：

- GroupSelector
- 已加入社團同步
- 社團名稱
- 社團 URL
- 社團快取
- 手動 URL fallback
- Group Management page
- 探索新社團
- Facebook group search
- AI group classification
- relevance score
- group join
- Queue join
- join status
- questions required
- question answer UI
- retry/review
- group risk budget
- Facebook checkpoint handling
- account locking
- posting queue
- scheduled posting
- result reporting
- LINE entry
- LIFF
- LINE account linking
- mobile composer
- PWA
- installable web app
- mobile upload
- mobile auth

==================================================
6. ACTUAL RUNTIME CHECK
==================================================

以非破壞方式確認目前線上。

至少：

- `https://app.ticenpi.com`
- Production Post URL
- Staging Post URL

如果 repo / deploy config 可證明實際 URL，
使用實際值。

只做安全 GET / UI navigation。

不要登入不必要的真實帳號。
不要送 Facebook 任務。
不要建立 Production data。

核對：

- mobile viewport
- navigation
- login
- Post UI
- group UI
- mobile usability
- installability
- manifest
- service worker
- browser console critical errors
- redirect behavior

==================================================
7. DETERMINE WHAT "WEB APP" MEANS HERE
==================================================

使用實際 source 分類目前 Post。

分開判斷：

### Normal Web Application

是否為正常可在 browser 使用的 web frontend？

### Responsive Mobile Web

是否真的對手機 usable？

不是只有 CSS 有 media query 就算 PASS。
實際檢查：

- viewport
- controls
- modal
- drawer
- GroupSelector
- composer
- text editing
- file/image upload
- scrolling
- touch targets

### PWA

查：

- web manifest
- service worker
- HTTPS
- icons
- display mode
- start_url
- installability
- offline strategy

不要因為「是網頁」就叫 PWA。

### LIFF compatible

查目前 frontend 是否能安全被 LINE LIFF WebView 開啟。

==================================================
8. LINE OFFICIAL ACCOUNT ENTRY — ACTUAL QUESTION
==================================================

使用者未來希望：

LINE 官方帳號
貼上一個網址 / Rich Menu / 訊息入口
↓
使用者點
↓
進 Post 操作

請實際研究目前 LINE Developers 官方文件。

優先官方：

- LINE Developers
- LIFF
- LINE Login
- Messaging API

記錄：

DOC_URL
DOC_DATE / accessed date
RELEVANT_RULE

必須回答：

### Plain HTTPS URL

如果 LINE OA 只是貼：

`https://post.ticenpi.com/...`

會發生什麼？

- LINE in-app browser?
- external browser?
- login session?
- Google OAuth?
- Supabase callback?
- cookies/localStorage?
- deep links?

### LIFF URL

如果改成 LIFF：

`https://liff.line.me/{LIFF_ID}`

有哪些差異？

查：

- endpoint URL
- LIFF app registration
- LIFF SDK
- LINE Login session
- `liff.init()`
- `liff.getProfile()`
- `liff.getIDToken()`
- redirect behavior
- external browser support
- share target / OA entry compatibility
- mobile WebView restrictions

不要憑記憶回答。
以目前官方 docs 為準。

==================================================
9. LINE IDENTITY LINKING
==================================================

設計前先查現況。

目標不是讓 LINE userId 取代 Supabase user_id。

既有正式 identity：

Supabase Auth user_id
↓
Customer
↓
Membership
↓
Entitlement / Seat
↓
Post Tenant

LINE 應作為額外 channel identity。

分析最小安全 mapping：

```text
LINE userId
↔
Ticenpi/Supabase user_id
```

需要回答：

- 第一次怎麼綁？
- 是否要求 Google Login 一次？
- LIFF 是否可以取得穩定 LINE user identity？
- 後端如何驗 LIFF token？
- 綁定後是否可免每次 Google Login？
- account unlink 如何做？
- 一個 LINE user 能否綁多個 Ticenpi account？
- 一個 Ticenpi user 是否允許多個 LINE identities？
- Customer/Seat/Entitlement revoke 後 LINE access 如何立即失效？

不得用 email 當 identity key。

==================================================
10. LINE WEBVIEW VS LAUNCHER / EXTENSION
==================================================

這是本次最重要的架構題之一。

現況：

Desktop Web：
Launcher primary
Extension fallback

但在：

LINE App WebView / 手機

通常不存在 Desktop Launcher / Chrome Extension。

因此實際調查目前 Facebook execution / binding 流程，
並回答：

### Facebook Account Binding

如果手機第一次使用 LIFF，
而該 User 尚未綁 Facebook，
能不能完成？

如果不能：

- 是否應導回 Desktop 做首次 binding？
- 是否有 mobile-safe binding path？
- 哪些能力在 LINE 中應 disabled？

### Posting execution

查目前真正 Facebook operation 是：

- VPS/backend worker
- Launcher
- Extension
- combination

不要猜。

如果 LINE 只是提交 Job，
而 Facebook execution 仍在 server/worker，
則 LIFF 可以只作 UI。

若實際 execution 依賴 Launcher online，
必須明確指出。

==================================================
11. PASTE URL FLOW FOR LINE
==================================================

評估未來最核心的 LINE 使用流程：

```text
LINE 官方帳號
↓
開 LIFF
↓
貼 591 / YUCT / 物件網址
↓
Extraction
↓
預覽物件
↓
選 FB 帳號
↓
選已加入社團
↓
發文
↓
Job Queue
↓
查看進度
```

逐步標出：

READY_NOW
NEEDS_UI
NEEDS_API
NEEDS_AUTH_LINK
NEEDS_LINE_CONFIG
BLOCKED_BY_FB_BINDING
BLOCKED_BY_EXECUTION_MODEL

==================================================
12. NOTIFICATION DESIGN
==================================================

研究 LINE Messaging API 官方限制。

判斷哪些通知需要：

- Reply Message
- Push Message
- LIFF UI polling
- WebSocket/SSE
- Post completion notification

避免設計成每一個 job state 都用付費 / 額度型 Push。

比較：

A.
LIFF 內直接看進度

B.
完成後 LINE Push 一次

C.
只 Reply 不主動 Push

說明各自技術需求。

不要直接改。

==================================================
13. FB GROUP DESIGN RECONCILIATION
==================================================

對 `fb-group-feature-design.md`
逐章 reconcile。

輸出：

SECTION
ORIGINAL_PLAN
CURRENT_REALITY
STATUS
STILL_VALID
CHANGE_REQUIRED

尤其：

- Browser Worker 現在是否仍在 VPS？
- Launcher architecture 是否已取代部分 Browser Worker 假設？
- Group search 應該在哪裡執行？
- Group join 應該在哪裡執行？
- Risk control 是否已有現成 infrastructure？
- 是否已有 ExtractionHub / shared worker 可重用？

不要因舊文件寫 T2 / Browser Worker
就假設今天仍應照舊。

==================================================
14. FIND ALL PARTIAL / DEAD / HIDDEN FEATURES
==================================================

搜尋：

- TODO
- FIXME
- disabled feature flags
- dead routes
- hidden nav entries
- unfinished components
- commented APIs
- unreferenced services
- old migrations
- tests with skipped markers
- mocks not wired to runtime
- env flags default off

找出：

「其實程式已做一半，但 UI 沒入口」
與
「UI 有入口，但 backend 未完成」

這類最值得優先恢復的功能。

==================================================
15. DO NOT CREATE A WISHLIST
==================================================

Roadmap 只能包含：

A. 有先前討論/設計證據
B. 現有 source 已有基礎
C. 為 LINE / mobile / current architecture 必要
D. 明確可提升目前產品完整性

不要隨意加一堆 unrelated SaaS feature。

每一項都要有：

WHY
EVIDENCE
DEPENDENCY
ESTIMATED_SCOPE

Scope 只能用：

S
M
L
XL

不要估工時天數。

==================================================
16. ROADMAP GROUPING
==================================================

最後分成：

### P0 — Architecture / correctness blockers

例如：
- mobile auth blocker
- LIFF identity
- Facebook binding incompatibility
- missing runtime contract

### P1 — Recover previously designed features

例如：
- Group Management
- FB search
- AI group classification
- Join workflow

### P2 — LINE entry MVP

只做到：
- LINE entry
- account link
- paste URL
- select account/group
- submit
- progress/result

### P3 — PWA / mobile polish

只有實際有必要才列。

### P4 — Advanced automation

例如：
- group join questions
- AI assistance
- advanced recommendation

但排序必須由依賴關係與 current gaps 支持。

==================================================
17. OUTPUT — EXECUTIVE SUMMARY
==================================================

先給白話總結：

CURRENT_PRODUCT_STATE =

例如：

```text
目前不是「沒有 Web App」，
而是：
有 Web App
但不是 PWA
手機版 usability 部分不足
尚未 LIFF 化
```

或者依實際結果填寫。

必須以 source/runtime 證據為準。

==================================================
18. OUTPUT — FEATURE RECOVERY MATRIX
==================================================

表格：

| Feature | Previously discussed | Current code | Runtime | Status | Missing | Recommended next |
|---|---|---|---|---|---|---|

至少 30 項。

不要省略。

==================================================
19. OUTPUT — LINE / LIFF FEASIBILITY
==================================================

固定輸出：

LINE_PLAIN_URL_ENTRY =
LIFF_REQUIRED = YES/NO/PREFERRED
CURRENT_FRONTEND_LIFF_COMPATIBLE =
GOOGLE_AUTH_IN_LIFF =
SUPABASE_AUTH_IN_LIFF =
LINE_TO_TICENPI_LINK_REQUIRED =
FB_BINDING_FROM_MOBILE =
FB_EXECUTION_FROM_MOBILE =
MAIN_BLOCKERS =

並解釋。

==================================================
20. OUTPUT — CURRENT WEB APP CLASSIFICATION
==================================================

固定：

NORMAL_WEB_APP =
RESPONSIVE_MOBILE_WEB =
PWA =
INSTALLABLE =
SERVICE_WORKER =
WEB_MANIFEST =
LIFF_READY =

每項：

YES / PARTIAL / NO / UNKNOWN

附 evidence。

==================================================
21. OUTPUT — ROADMAP
==================================================

最後給：

ROADMAP_PHASE_0 =
ROADMAP_PHASE_1 =
ROADMAP_PHASE_2 =
ROADMAP_PHASE_3 =
ROADMAP_PHASE_4 =

每個項目：

- 功能
- 現況
- 依賴
- Source owner
- Scope S/M/L/XL
- Acceptance criteria

==================================================
22. OUTPUT — WHAT TO DO NEXT
==================================================

最後只選出下一輪真正值得執行的 3～5 個 implementation tasks。

不要直接執行。

每個 Task 給：

TASK_NAME
WHY_NOW
DEPENDENCIES
FILES / AREAS
MODEL_RECOMMENDATION
CAN_RUN_IN_PARALLEL = YES/NO
RISK = LOW/MEDIUM/HIGH

並明確指出：

哪些可以平行，
哪些必須串行。

==================================================
23. FINAL REPORT FORMAT
==================================================

PHASE = TICENPIPOST FEATURE / WEB / LIFF AUDIT
RESULT = PASS / PARTIAL / HARD_STOP

A. SOURCES
MEMORY_FILES_READ =
PROJECT_DOCS_READ =
PRIOR_PROMPTS_READ =
GIT_HISTORY_CHECKED =
RUNTIME_CHECKED =
OFFICIAL_LINE_DOCS_CHECKED =

B. CURRENT PRODUCT
NORMAL_WEB_APP =
RESPONSIVE_MOBILE_WEB =
PWA =
INSTALLABLE =
LIFF_READY =

C. FB GROUP
JOINED_GROUP_SYNC =
GROUP_SELECTOR =
GROUP_MANAGEMENT =
GROUP_SEARCH =
AI_CLASSIFICATION =
GROUP_JOIN =
JOIN_QUESTIONS =
RISK_CONTROL =

D. LINE
PLAIN_URL_ENTRY =
LIFF_ENTRY =
LINE_ID_LINK =
MOBILE_POST_FLOW =
FB_BINDING_LIMITATION =
FB_EXECUTION_LIMITATION =

E. RECOVERY
IMPLEMENTED_COUNT =
PARTIAL_COUNT =
DESIGN_ONLY_COUNT =
NOT_FOUND_COUNT =
REPLACED_COUNT =

F. ROADMAP
P0 =
P1 =
P2 =
P3 =
P4 =

G. NEXT IMPLEMENTATION TASKS
1.
2.
3.
4.
5.

H. PARALLELIZATION
SAFE_PARALLEL_TASKS =
SERIAL_DEPENDENCIES =

I. FINAL
READY_FOR_IMPLEMENTATION_PLANNING = YES/NO

Do not modify source.
Do not commit.
Do not push.
Do not deploy.
