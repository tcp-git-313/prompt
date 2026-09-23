# CTHOUSE ACCESS DIAGNOSTIC OWNER — LUNA EXECUTION PLAN

## 任務目標

接續 CTHouse Reference Adapter 工作。

目前已知：

- CTHouse adapter 已存在並完成 features truncation 修復。
- CTHouse focused / image / YUCT / full ExtractionHub regression 已通過。
- CTHouse Golden fixture 已通過。
- 真實公開來源曾回 HTTP 200，之後同一類公開頁面開始回 HTTP 403 ACCESS_DENIED。
- 因 live Development、圖片 payload、fresh unseen 尚未完成，所以 CTHouse 維持 DRAFT / REFERENCE BLOCKED。

本任務唯一目標：

> 找出 CTHouse live access 從 HTTP 200 變成 HTTP 403 的可證明原因或最小可證明邊界，判斷目前應繼續 STANDARD_HTTP、需要正常 Browser transport、還是來源目前不可自動驗證。

本任務不是：
- 繞過 403
- 破解 CAPTCHA
- 模擬真人指紋
- 使用 stealth / anti-detection
- 取得未授權內容
- 重寫 CTHouse adapter
- 開始其他來源
- 部署 Staging / Production

若來源明確拒絕自動存取，必須停止並如實回報。

---

## 工作目錄

主要：
F:\00-Ticenpi-SaaS\ExtractionHub

參考，只讀：
F:\00-Ticenpi-SaaS\TicenpiDM
F:\00-Ticenpi-SaaS\TicenpiPost
F:\00-Ticenpi-SaaS\Ticenpi591

不要修改產品 repo。

---

# PHASE 0 — PREFLIGHT

先確認：

git branch
git HEAD
git status
git diff
git diff --cached
untracked files

保留現有 WIP。

禁止：

git reset --hard
git clean
git stash
rebase

確認是否有其他 active writer 修改：

- CTHouse adapter
- transport
- access policy
- diagnostics
- CTHouse tests

如果有真正 collision：

HARD STOP

輸出：

CTHOUSE ACCESS DIAGNOSTIC COLLISION

如果沒有：

CTHOUSE ACCESS DIAGNOSTIC PREFLIGHT PASS

---

# PHASE 1 — FREEZE APPLICATION LOGIC

本階段先不要修改 CTHouse parser / canonical mapping。

把目前 final candidate 視為：

DIAGNOSTIC BASELINE

記錄：

- branch
- HEAD
- relevant dirty files
- adapter version
- schema version
- transport implementation
- current access-policy behavior

如果需要加入診斷輸出，只允許：

- sanitized
- non-secret
- 不改變 request semantics

的 instrumentation。

不要為了取得 200 改 parser。

---

# PHASE 2 — REPRODUCE 403 SAFELY

使用上一輪已知的 CTHouse 公開 detail URLs / fixture source references。

如果原始 live URLs 已記錄在：

docs/CTHOUSE-CONTROLLED-VALIDATION-20260923.md

優先從該文件取得，不要自行猜 URL。

對每一個 live URL 做最小請求驗證。

記錄：

- timestamp
- input URL
- final URL
- status code
- redirect chain
- response content type
- response length
- selected safe response headers
- access-policy classification
- sanitized body fingerprint / title / reason snippet
- transport used
- elapsed time

不要保存：

- full cookies
- bearer tokens
- secrets
- sensitive headers

如果 body 是 CDN / WAF / challenge page：

只記安全 fingerprint，例如：

- title
- known challenge marker
- hash
- short sanitized indicator

不要嘗試解 challenge。

---

# PHASE 3 — CLASSIFY THE 403

將每一個 403 優先分類成：

A. ORIGIN_ACCESS_DENIED
來源站明確拒絕。

B. CDN_WAF_BLOCK
CDN / WAF / anti-automation response。

C. RATE_LIMIT_OR_TEMPORARY_BLOCK
有明確 Retry-After、429/403 policy、時間性跡象或同環境稍後恢復。

D. SESSION_OR_PUBLIC_COOKIE_REQUIRED
正常公開瀏覽流程會先設置公開 session/cookie，detail request 缺少該公開狀態。
只允許觀察正常公開流程，不得使用私人憑證或繞過存取限制。

E. REQUEST_CONTRACT_MISMATCH
例如來源目前要求正常且公開的必要 header / redirect / referer flow，而現行 transport 沒遵守一般網站請求契約。
不得使用假冒身份、stealth 或指紋偽裝。

F. NETWORK_OR_ENVIRONMENT_SPECIFIC
同一公開 URL 在合法不同執行環境有不同結果，例如：
- 本機正常
- VPS 403
或反之。

G. CONTENT_REMOVED_OR_INVALID_URL
URL 本身失效 / 內容移除。

H. UNKNOWN

每個判定都必須附證據。

不能只因「403」就猜：

CAPTCHA
IP block
rate limit
browser required

---

# PHASE 4 — COMPARE WITH NORMAL PUBLIC BROWSER BEHAVIOR

目的：

確認「網站是否對一般公開瀏覽本身可用」。

允許：

- 使用普通瀏覽器 / Playwright 非 stealth 模式
- 正常導航到公開 URL
- 觀察 status / DOM 是否正常
- 觀察是否出現登入牆 / challenge / consent / redirect
- 觀察公開頁是否先建立必要的 non-sensitive public session state

禁止：

- stealth plugin
- fingerprint spoofing
- CAPTCHA solving
- challenge bypass
- private account login
- proxy rotation to evade policy
- header impersonation designed to bypass controls

如果普通、非 stealth Browser：

也被拒絕
→ Browser 不是解法。

如果普通 Browser 正常，但 STANDARD_HTTP 被拒絕：

只記錄差異，進 PHASE 5。

---

# PHASE 5 — REQUEST CONTRACT DIFF

若 Browser 正常而 STANDARD_HTTP 403：

比較一般公開 Browser 請求與現有 HTTP transport 的「正常協定差異」。

只分析：

- HTTP method
- URL / redirect sequence
- ordinary Accept headers
- ordinary Content-Type / Accept-Language if already part of public request
- referer flow where naturally required
- public session cookie presence/absence
- response compression / protocol behavior
- timing / redirect ordering

不要收集或複製：

- browser fingerprint
- device fingerprint
- challenge token
- anti-bot clearance token
- authentication token
- private session cookie

如果差異只是一般網站協定需求：

提出最小 transport correction。

如果差異依賴 anti-bot / clearance / challenge：

分類：

ACCESS_POLICY_BLOCKED

不要實作 bypass。

---

# PHASE 6 — ENVIRONMENT COMPARISON

只有在現有開發環境已合法具備多個執行位置時才比較。

例如：

- current local development host
- existing VPS / staging host

不要為本任務新買 proxy、新建跳板或輪替 IP。

對同一公開 URL 使用相同 STANDARD_HTTP request。

比較：

- DNS result（安全摘要）
- resolved host/CDN
- status
- redirect
- response fingerprint
- timing
- access classification

如果：

Local = 200
VPS = 403

或：

Local = 403
VPS = 200

標記：

ENVIRONMENT_SPECIFIC

但不要嘗試透過換 IP 繞過。

---

# PHASE 7 — TEMPORAL / RATE-LIMIT EVIDENCE

如果懷疑暫時性限制：

不要高頻重試。

使用低頻、有限次數的觀察。

最多建立足夠證據判斷：

- always denied
- transient
- retry-after driven
- unknown

不要做壓力測試。

不要為了取得 200 反覆重試。

如果來源回：

Retry-After

尊重該值。

---

# PHASE 8 — DECISION TREE

最後只能做以下其中一個結論。

## RESULT A — STANDARD_HTTP_VALID

有證據證明：

- 公開頁可正常 HTTP 取得
- 403 是 temporary / contract bug / implementation issue
- 修正不涉及繞過存取控制

則：

1. 提出最小修正。
2. 加 regression test。
3. 重跑 CTHouse live Development。
4. 若 live Development PASS，再恢復後續 fresh unseen gate。

## RESULT B — NORMAL_BROWSER_REQUIRED

只有在有證據證明：

- 一般非 stealth Browser 公開導航正常
- HTTP 無法取得必要內容
- Browser 不依賴 challenge bypass / private login

才能判：

NORMAL_BROWSER_REQUIRED

此時：

不要在本任務完整實作大型 Browser Runtime。

只輸出：

Browser transport requirement
minimum interface
resource estimate evidence
integration impact

並 HARD STOP，等待核准。

## RESULT C — ACCESS_POLICY_BLOCKED

如果來源要求：

- anti-bot challenge
- CAPTCHA
- clearance token
- login / authorization
- 明確拒絕 automated access

輸出：

ACCESS_POLICY_BLOCKED

停止。

不要提供或實作繞過方法。

## RESULT D — ENVIRONMENT_SPECIFIC

若不同既有合法環境結果不同：

輸出：

ENVIRONMENT_SPECIFIC

列清楚：

where 200
where 403
evidence
what remains unknown

不要自動用「能成功的 IP」當規避方案。

## RESULT E — INCONCLUSIVE

如果證據不足：

輸出：

INCONCLUSIVE

不要猜。

---

# PHASE 9 — ONLY IF SAFE HTTP FIX IS PROVEN

只有 RESULT A 才能修改 transport / access logic。

修改必須：

- minimal
- generic
- standards-compliant
- no stealth
- no bypass
- no TLS disable
- no verify=False
- no ssl=False

修後跑：

1. CTHouse focused tests
2. transport tests
3. access-policy tests
4. YUCT focused tests
5. full ExtractionHub pytest
6. git diff --check

不能造成 YUCT regression。

---

# PHASE 10 — RESUME CTHOUSE LIVE VALIDATION ONLY WHEN ACCESS IS LEGITIMATELY RESTORED

如果合法、安全方式恢復 live access：

再跑最終 candidate：

Development Set
↓
Canonical coverage
↓
provenance
↓
image validation
↓
Fresh Unseen
↓
Golden
↓
Full regression

Fresh unseen 規則維持：

第一次執行前不能看來修 parser。

如果拿 unseen 修程式：

該批即 consumed，不再算 fresh unseen。

---

# FINAL REPORT FORMAT

# CTHOUSE ACCESS DIAGNOSTIC REPORT

## 1. Baseline

Branch:
HEAD:
Adapter version:
Schema version:
Collision:
PASS / BLOCKED

## 2. Reproduction

| URL | Transport | Status | Final URL | Classification | Evidence |

## 3. Public Browser Check

Browser:
NORMAL / DENIED / CHALLENGE / NOT_TESTED

Public page visible:
YES / NO / UNKNOWN

Login required:
YES / NO / UNKNOWN

Challenge required:
YES / NO / UNKNOWN

## 4. HTTP vs Browser Contract

Differences proven:

- ...

Sensitive / anti-bot material collected:
NO

## 5. Environment Comparison

Local:
VPS/Staging:
Other:

Result:

SAME / DIFFERENT / NOT_TESTED

## 6. Rate / Temporal Evidence

Result:

STABLE_DENY
TRANSIENT
RETRY_AFTER
UNKNOWN
NOT_TESTED

## 7. Root Cause Classification

只能選：

STANDARD_HTTP_VALID
NORMAL_BROWSER_REQUIRED
ACCESS_POLICY_BLOCKED
ENVIRONMENT_SPECIFIC
INCONCLUSIVE

Evidence:

## 8. Changes

如果沒有安全可證明的修正：

NONE

如果有：

Root cause:
Minimal fix:
Tests:

## 9. Regression

CTHouse focused:
Transport:
Access policy:
YUCT focused:
Full pytest:
git diff --check:

## 10. CTHouse Validation Status

Live Development:
PASS / BLOCKED / NOT_RUN

Image payload validation:
PASS / BLOCKED / NOT_RUN

Fresh unseen:
PASS / BLOCKED / NOT_RUN

## 11. Final Result

只能輸出其中之一：

CTHOUSE ACCESS RESTORED — READY TO RESUME VALIDATION

CTHOUSE NORMAL BROWSER TRANSPORT REVIEW REQUIRED

CTHOUSE ACCESS POLICY BLOCKED

CTHOUSE ENVIRONMENT-SPECIFIC ACCESS

CTHOUSE ACCESS DIAGNOSTIC INCONCLUSIVE

---

# HARD STOP

本任務不要：

- 開始住商
- 開始永慶
- 開始 591
- 修改 DM / Post / 591
- 部署 Staging / Production
- 做 Browser stealth / anti-bot bypass
- 解 CAPTCHA
- 使用 proxy rotation
- 使用私人帳號登入
- 把 CTHouse 標 Supported

完成 Access Diagnostic 後停止，等待使用者下一個指令。
