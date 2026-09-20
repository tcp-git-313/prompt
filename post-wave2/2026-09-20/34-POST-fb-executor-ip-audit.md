# POST-FB-EXECUTOR-AUDIT — Verify Facebook Execution Location and Source IP

抬頭：
POST FB EXECUTOR / REAL-IP AUDIT OWNER

模型：
DeepSeek V4.1 Flash

預計工程大小：
中型

任務性質：
READ-ONLY / TRACE / ARCHITECTURE AUDIT

---

# 目的

確認目前 Ticenpi POST 的 Facebook 功能，真正對 Facebook 發出 HTTP / Browser Request 的機器到底是哪一台。

最重要的判斷不是「Cookies 存在哪」，而是：

```
Facebook 最終 Request 的 Executor 在哪裡？
```

預期原始設計應該是：

```
VPS Backend
→ 建立任務 / Queue
→ Launcher 收到工作
→ Launcher 使用使用者本機 Chrome / FB Session
→ 從使用者真實網路 IP 存取 Facebook
→ 結果回傳 VPS
```

Local 開發則應該是：

```
POST Local
→ Chrome Extension
→ 本機 Chrome / FB Session
→ 從開發者本機真實 IP 存取 Facebook
```

需要排除的錯誤架構是：

```
Launcher / Extension
→ 只把 FB Cookies 上傳 VPS
→ VPS Worker 讀 Cookies
→ Oracle VPS 直接存取 Facebook
```

這次只做唯讀排查。

不要修改任何程式、設定、資料庫、VPS、Launcher、Extension、CI 或部署內容。

不要執行任何真實 Facebook write action。

---

# 0. HARD RULES

禁止：

- 修改 source
- commit
- push
- cherry-pick
- rebase
- reset
- deploy
- restart Production
- 修改 env
- 修改 Supabase
- 修改 Launcher
- 修改 Extension
- 真實 FB Post
- 真實 Comment
- 真實 Marketplace
- 真實 Relist
- 真實 Delete

允許：

- git log / show / grep
- source trace
- read logs
- read compose/env key names
- read queue/job implementation
- read Launcher/Extension code
- read backend/worker code
- read Production/Staging logs，但不得輸出 cookie/token/secret
- local static analysis
- safe read-only endpoint inspection

---

# 1. 先確認所有相關 Repo / Worktree

至少檢查：

```
F:\00-Ticenpi-SaaS\TicenpiPost
```

並用：

```
git worktree list --porcelain
```

找出所有 POST 相關 worktree。

同時找 Launcher repo。

不要直接相信目前 checkout。

記錄：

```
POST_REPO=
POST_BRANCH=
POST_HEAD=

LAUNCHER_REPO=
LAUNCHER_BRANCH=
LAUNCHER_HEAD=

EXTENSION_PATH=
```

如果 Launcher 有多版本：

- published stable
- origin/main
- local dirty workspace

全部標明，不要混在一起。

---

# 2. 建立完整 Facebook Job Trace

從 Web/API 開始，沿著真實程式碼一路追：

```
Web
→ Backend API
→ DB / Queue
→ Worker
→ Launcher / Extension
→ Facebook
→ Result
```

每一段都要列：

- source file
- function/class
- endpoint
- queue/job name
- caller
- callee
- transport
- executor process

不要只看函式名稱判斷。

必須追到真正接觸：

```
facebook.com
m.facebook.com
www.facebook.com
graph.facebook.com
Facebook browser DOM
Chrome automation
fetch/axios/request/playwright/selenium/browser navigation
```

的程式碼位置。

---

# 3. 逐項確認所有 Facebook Action

至少檢查：

```
GROUP LIST / GROUP READ
POST
COMMENT
MARKETPLACE
RELIST
DELETE
```

每一項最後一定要回答：

```
EXECUTOR=
LAUNCHER / EXTENSION / VPS_WORKER / BACKEND / UNKNOWN

FACEBOOK_REQUEST_SOURCE=
USER_IP / VPS_IP / MIXED / UNKNOWN

CODE_PATH=
<exact file/function chain>

COOKIES_USED_BY=
LAUNCHER / EXTENSION / VPS / NONE / UNKNOWN

RESULT_RETURN_PATH=
```

如果不同 action 執行位置不同，必須分開寫。

不要用單一結論概括所有 action。

---

# 4. Launcher 專項排查

確認 Launcher 到底是：

A. 真正本機 Facebook Executor

還是：

B. 只負責取得/同步 Cookies，真正 FB request 在 VPS

至少追：

```
connect-facebook
import-cookies
syncCookies
deviceToken
heartbeat
setInterval recurring sync
browser/chrome integration
job poll / job receive
action dispatch
result upload
```

找出：

- Launcher 是否接收 FB Job
- 接收方式：poll / websocket / HTTP / queue bridge / other
- Launcher 是否直接控制本機 Chrome
- Launcher 是否直接向 facebook.com 發 request
- Launcher 是否執行 DOM 操作
- Launcher 是否回傳 action result
- Launcher 是否只送 cookie/session

輸出：

```
LAUNCHER_IS_FB_EXECUTOR=
YES/NO/PARTIAL

LAUNCHER_RECEIVES_JOBS=
YES/NO

LAUNCHER_CONTROLS_LOCAL_BROWSER=
YES/NO

LAUNCHER_CALLS_FACEBOOK_DIRECTLY=
YES/NO

LAUNCHER_RETURNS_FB_RESULTS=
YES/NO
```

---

# 5. Extension 專項排查

確認 Local Extension 實際行為：

- cookie acquisition
- session/account detection
- bind
- recurring sync
- browser/DOM execution
- Facebook request execution

輸出：

```
EXTENSION_IS_FB_EXECUTOR=
YES/NO/PARTIAL

EXTENSION_COOKIE_CAPTURE=
YES/NO

EXTENSION_CALLS_FACEBOOK_DIRECTLY=
YES/NO

EXTENSION_USES_LOCAL_CHROME_SESSION=
YES/NO

EXTENSION_FB_SOURCE_IP=
LOCAL_USER_IP / UNKNOWN
```

並比較 Launcher 與 Extension：

```
COOKIE_CAPTURE_METHOD
SESSION SOURCE
FB REQUEST EXECUTION
BROWSER PROFILE
PAYLOAD TO BACKEND
RESULT FROM FACEBOOK
```

---

# 6. VPS Backend / Worker 專項排查

這是本次最重要的檢查。

搜尋所有：

- FB cookie storage/read
- session storage/read
- FB account token
- browser automation
- HTTP client to Facebook
- worker code
- queue consumer
- group list fetch
- post/comment/marketplace/relist/delete implementation

明確回答：

```
VPS_READS_FB_COOKIES=
YES/NO/PARTIAL

VPS_CALLS_FACEBOOK_DIRECTLY=
YES/NO/PARTIAL

VPS_LAUNCHES_BROWSER_FOR_FACEBOOK=
YES/NO

VPS_ACTIONS_EXECUTED=
<list>

VPS_FB_SOURCE_IP=
ORACLE_VPS_IP / NONE / UNKNOWN
```

如果 Worker 只是：

```
建立任務
→ 等 Launcher
→ 收結果
```

要明確標：

```
VPS_WORKER_ROLE=DISPATCH_ONLY
```

如果 Worker 自己拿 cookies 打 Facebook：

```
VPS_WORKER_ROLE=FB_EXECUTOR
```

---

# 7. import-cookies / syncCookies 真正用途

目前已知 Launcher 有：

```
import-cookies
syncCookies
deviceToken
setInterval
```

但「Cookies 上傳 VPS」本身不能證明 VPS 用它打 Facebook。

必須沿資料流追：

```
Launcher cookie payload
→ API endpoint
→ validation
→ storage
→ consumer
→ actual downstream use
```

最後分類：

```
COOKIE_SYNC_PURPOSE=
ACCOUNT_MATCH_ONLY
SESSION_METADATA_ONLY
RECOVERY
SERVER_SIDE_FB_EXECUTION
MIXED
UNKNOWN
```

特別追目前觀察到的：

```
import-cookies → HTTP 422
```

只讀查清楚：

- request 是否真的到 backend
- 哪個 validation rule 拒絕
- payload 缺什麼
- account mismatch?
- tenant mismatch?
- schema mismatch?
- cookie field mismatch?
- device token mismatch?
- origin/deep-link mismatch?

禁止輸出：

- cookie values
- access tokens
- session secrets
- device token raw values

輸出：

```
IMPORT_COOKIES_422_REACHED_BACKEND=
YES/NO

IMPORT_COOKIES_422_VALIDATION_RULE=

IMPORT_COOKIES_422_ROOT_CAUSE=
```

---

# 8. 真實來源 IP 判定

不要靠猜。

對每個 Action 根據 Executor 判定來源 IP。

規則：

如果：

```
Launcher 本機 process / local Chrome
→ facebook.com
```

則：

```
SOURCE_IP = USER_REAL_IP
```

如果：

```
VPS worker/container
→ facebook.com
```

則：

```
SOURCE_IP = VPS / ORACLE IP
```

如果兩邊都會：

```
SOURCE_IP = MIXED
```

如果 source trace 不足：

```
SOURCE_IP = UNKNOWN
```

不要憑 cookie 存放位置推論來源 IP。

---

# 9. 檢查是否符合原始產品目的

原始商用目的：

```
VPS
= Control Plane / Queue / Auth / State / Result

Launcher
= Customer-side Facebook Executor

Facebook
= 從客戶自己的真實 IP 存取
```

Local：

```
Extension
= Local Facebook Executor
```

最終要判斷：

```
ARCHITECTURE_MATCHES_ORIGINAL_DESIGN=
YES / NO / PARTIAL
```

如果 NO/PARTIAL：

列出偏離項：

```
WRONG_PATHS=
- action
- current executor
- expected executor
- risk
```

這次不要修。

---

# 10. 額外檢查：Commercial fallback

目前曾調查到：

```
Commercial
→ Launcher
→ Launcher offline 時 fallback Extension
```

請確認這是真的 source behavior 還是誤判。

輸出：

```
COMMERCIAL_EXTENSION_FALLBACK_EXISTS=
YES/NO

FALLBACK_CODE_PATH=

FALLBACK_ACTIONS=

FALLBACK_EFFECT=
```

但這次不要移除。

先確認它是否會造成：

- Production 使用 Extension
- FB request 從使用者 IP 執行
- 或只是 bind fallback

不要先假設。

---

# 11. 必須輸出的最終報告

```
TASK=POST_FB_EXECUTOR_AUDIT

=== SOURCE ===

POST_REPO=
POST_BRANCH=
POST_HEAD=

LAUNCHER_REPO=
LAUNCHER_BRANCH=
LAUNCHER_HEAD=

EXTENSION_PATH=

=== EXPECTED ARCHITECTURE ===

EXPECTED=
VPS dispatch → Launcher local execution → Facebook → user real IP

LOCAL_EXPECTED=
Extension local execution → Facebook → local real IP

=== GROUP LIST ===

EXECUTOR=
FACEBOOK_REQUEST_SOURCE=
COOKIES_USED_BY=
CODE_PATH=
RESULT_RETURN_PATH=

=== POST ===

EXECUTOR=
FACEBOOK_REQUEST_SOURCE=
COOKIES_USED_BY=
CODE_PATH=
RESULT_RETURN_PATH=

=== COMMENT ===

EXECUTOR=
FACEBOOK_REQUEST_SOURCE=
COOKIES_USED_BY=
CODE_PATH=
RESULT_RETURN_PATH=

=== MARKETPLACE ===

EXECUTOR=
FACEBOOK_REQUEST_SOURCE=
COOKIES_USED_BY=
CODE_PATH=
RESULT_RETURN_PATH=

=== RELIST ===

EXECUTOR=
FACEBOOK_REQUEST_SOURCE=
COOKIES_USED_BY=
CODE_PATH=
RESULT_RETURN_PATH=

=== DELETE ===

EXECUTOR=
FACEBOOK_REQUEST_SOURCE=
COOKIES_USED_BY=
CODE_PATH=
RESULT_RETURN_PATH=

=== EXTENSION ===

EXTENSION_IS_FB_EXECUTOR=
EXTENSION_COOKIE_CAPTURE=
EXTENSION_CALLS_FACEBOOK_DIRECTLY=
EXTENSION_USES_LOCAL_CHROME_SESSION=
EXTENSION_FB_SOURCE_IP=

=== LAUNCHER ===

LAUNCHER_IS_FB_EXECUTOR=
LAUNCHER_RECEIVES_JOBS=
LAUNCHER_CONTROLS_LOCAL_BROWSER=
LAUNCHER_CALLS_FACEBOOK_DIRECTLY=
LAUNCHER_RETURNS_FB_RESULTS=

COOKIE_SYNC_PURPOSE=

=== VPS WORKER ===

VPS_READS_FB_COOKIES=
VPS_CALLS_FACEBOOK_DIRECTLY=
VPS_LAUNCHES_BROWSER_FOR_FACEBOOK=
VPS_ACTIONS_EXECUTED=
VPS_FB_SOURCE_IP=
VPS_WORKER_ROLE=

=== IMPORT-COOKIES 422 ===

IMPORT_COOKIES_422_REACHED_BACKEND=
IMPORT_COOKIES_422_VALIDATION_RULE=
IMPORT_COOKIES_422_ROOT_CAUSE=

=== COMMERCIAL FALLBACK ===

COMMERCIAL_EXTENSION_FALLBACK_EXISTS=
FALLBACK_CODE_PATH=
FALLBACK_ACTIONS=
FALLBACK_EFFECT=

=== FINAL ===

ARCHITECTURE_MATCHES_ORIGINAL_DESIGN=
YES/NO/PARTIAL

FACEBOOK_REQUESTS_USE_USER_REAL_IP=
YES/NO/PARTIAL/UNKNOWN

VPS_DIRECT_FB_ACCESS=
YES/NO/PARTIAL

WRONG_PATHS=
MISSING_PATHS=

REQUIRED_FIXES=
<只列建議，不修改>

READY_TO_DESIGN_FIX=
YES/NO
```

---

# 12. SUCCESS STANDARD

完成這次任務後，必須能明確回答：

1. Group List 到底在哪裡執行？
2. Post 到底在哪裡執行？
3. Comment 到底在哪裡執行？
4. Marketplace 到底在哪裡執行？
5. Relist 到底在哪裡執行？
6. Delete 到底在哪裡執行？
7. Launcher 是否真的使用客戶本機 IP 存取 Facebook？
8. VPS 是否直接拿 FB Cookies 存取 Facebook？
9. import-cookies/syncCookies 的真正用途是什麼？
10. 422 的實際驗證失敗原因是什麼？
11. Extension 和 Launcher 的 FB Executor 行為是否一致？
12. 現在架構是否仍符合「Launcher = 客戶端 Facebook Executor」的原始設計？

如果任何一項沒有 source/runtime evidence：

標 UNKNOWN。

禁止用推論冒充 PASS。
