# W5-E — Launcher Source Reconciliation（唯讀）

ROLE: Launcher Source / Artifact Reconciliation Verifier

推薦模型：
MiMo V2.6 Pro
備選：KIMI 3
不需要 OPUS 5.5。

## 目的

目前 Production 上的 app.ticenpi.com 顯示的是舊版主控台 UI，但 Central Seat 完成後曾看過新版 Workbench。

已知 Production 報告宣稱：
- Launcher Production source = b7f85bf
- Cloudflare Pages deployment = 5cb1ad90
- 線上檔案與該 build 一致
- Production Supabase 設定正確

但使用者實際看到的 UI 被確認是「最舊版」。

本任務只回答一個核心問題：

**Production 技術上雖部署成功，但是否選錯 Launcher candidate / source？**

不要修改任何檔案。
不要 commit。
不要 push。
不要 deploy。
不要改 Cloudflare。
不要改 Supabase。
不要新增 Customer / Member / Seat。
不要碰 Post / DM / OCR。

---

## 工作區

F:\00-Ticenpi-SaaS

主要檢查：
F:\00-Ticenpi-SaaS\Ticenpi-Launcher

也要搜尋：
F:\00-Ticenpi-SaaS\.worktrees
F:\00-Ticenpi-SaaS\artifacts
F:\00-Ticenpi-SaaS\deploy

已知候選：
- b7f85bf
- w3-launcher-preview
- Ticenpi-Launcher-w2-f1

---

# 1. 找出「新版 Workbench」真正 source

搜尋所有 Launcher repo / worktree / artifacts。

找包含以下能力的版本：

- Customer 詳細頁
- member list
- stable user_id
- Entitlement 顯示
- Seat assign
- Seat release
- member suspend/reactivate
- customer suspend/reactivate
- product isolation（post / dm / ocr）
- Central Seat Workbench

不要只看 commit message。
直接檢查實際程式碼 / build artifact。

輸出：

NEW_WORKBENCH_CANDIDATES =
LATEST_WORKBENCH_SOURCE =
LATEST_WORKBENCH_COMMIT =
LATEST_WORKBENCH_PATH =

---

# 2. 檢查 b7f85bf 到底是不是舊 UI

checkout / inspect b7f85bf，但保持唯讀。

確認 b7f85bf 是否真的包含：

- Workbench JS
- Customer detail
- Members
- Entitlements
- Seats
- lifecycle actions

如果只包含舊 Customer 主控台，就明確寫：

B7F85BF_UI_CLASS = LEGACY

如果包含新版完整 Workbench：

B7F85BF_UI_CLASS = CENTRAL_SEAT_WORKBENCH

列出證據檔案與重要 symbol / route。

輸出：

B7F85BF_UI_CLASS =
B7F85BF_WORKBENCH_FEATURES =
B7F85BF_MISSING_FEATURES =

---

# 3. 對比「目前 Production UI」與 b7f85bf

唯讀檢查：
https://app.ticenpi.com

確認目前實際部署的 HTML / JS asset identity。

對比：
- Cloudflare Pages deployment 5cb1ad90
- b7f85bf build artifact
- 現在線上 JS/CSS
- Production Supabase config

不要只看 HTML 外觀。
比對 asset hash / JS content / build identity。

輸出：

PRODUCTION_DEPLOYMENT =
PRODUCTION_ASSET_MATCHES_B7F85BF =
PRODUCTION_UI_CLASS =

---

# 4. 找出 W3 Launcher E2E 真正測的是哪一份

搜尋 W3 artifacts / reports / logs / preview 目錄。

確認當時通過：

- Customer lifecycle
- member lifecycle
- Seat assign/release
- Customer B restore

這些 E2E 是跑在：

A. b7f85bf
B. w3-launcher-preview
C. 另一個 worktree / commit
D. build artifact 與 source commit 不一致

不要相信 summary。
對回實際 test target / artifact / hash / source path。

輸出：

W3_E2E_TESTED_SOURCE =
W3_E2E_TESTED_COMMIT =
W3_E2E_TESTED_ARTIFACT =
W3_E2E_MATCHES_B7F85BF = YES/NO

---

# 5. 找出是否存在「新版 UI 未提交 / 未推送」

搜尋：

- dirty worktree
- detached worktree
- untracked files
- artifacts
- preview build
- local-only commit
- branches not pushed

特別檢查：

workbench.js
index.html
launcher/hub source
Customer detail UI
Seat UI

輸出：

UNPUSHED_NEW_WORKBENCH_FOUND =
UNPUSHED_PATH =
UNPUSHED_COMMIT =
DIRTY_WORKTREE_FOUND =

---

# 6. 判斷根因

只能從以下分類選一個主要 root cause：

A. b7f85bf 本身就是舊 UI，Production 選錯 candidate
B. b7f85bf 是新版，但 Production asset 不是 b7f85bf
C. W3 E2E 測的是另一份新版 artifact，source reconciliation 錯誤
D. 新版 UI 只存在 local/worktree/artifact，從未正式提交
E. browser cache / stale Cloudflare asset
F. 其他有直接證據的原因

輸出：

ROOT_CAUSE_CLASS =
ROOT_CAUSE =

---

# 7. 判斷是否有「新舊系統交叉」

確認：

- Production Launcher 是否連 Production Supabase
- UI source 是否舊版
- Platform Commercial Core 是否新版
- Customer / Member / Entitlement / Seat RPC 是否新版

如果：

Backend = 新
Frontend = 舊

請標：

MIXED_VERSION_STATE = YES

並說明這是：
「舊 Launcher UI + 新 Central Seat backend」
而不是 Staging/Production DB 混線。

如果真的發現 Production Launcher 指向 Staging Supabase：

ENVIRONMENT_CROSSOVER = YES

這是 GO-LIVE blocker。

輸出：

MIXED_VERSION_STATE =
ENVIRONMENT_CROSSOVER =
PRODUCTION_SUPABASE_REF =

---

# 8. 最小修正建議

如果確認 Production 選錯 Launcher source：

不要直接 deploy。

只提出：

CORRECT_LAUNCHER_SOURCE =
CORRECT_LAUNCHER_COMMIT =
CORRECT_BUILD_PATH =
REDEPLOY_LAUNCHER_ONLY = YES

並確認：

- Platform 不需重做
- Post 不需重做
- DM 不需重做
- OCR 不需重做
- Commercial Core 不需重做

只需：
Launcher 正確 candidate
→ build Production config
→ tests
→ Cloudflare Pages deploy
→ Workbench canary

---

# 最終輸出

只回：

NEW_WORKBENCH_CANDIDATES =
LATEST_WORKBENCH_SOURCE =
LATEST_WORKBENCH_COMMIT =
LATEST_WORKBENCH_PATH =

B7F85BF_UI_CLASS =
B7F85BF_WORKBENCH_FEATURES =
B7F85BF_MISSING_FEATURES =

PRODUCTION_DEPLOYMENT =
PRODUCTION_ASSET_MATCHES_B7F85BF =
PRODUCTION_UI_CLASS =

W3_E2E_TESTED_SOURCE =
W3_E2E_TESTED_COMMIT =
W3_E2E_TESTED_ARTIFACT =
W3_E2E_MATCHES_B7F85BF =

UNPUSHED_NEW_WORKBENCH_FOUND =
UNPUSHED_PATH =
UNPUSHED_COMMIT =
DIRTY_WORKTREE_FOUND =

ROOT_CAUSE_CLASS =
ROOT_CAUSE =

MIXED_VERSION_STATE =
ENVIRONMENT_CROSSOVER =
PRODUCTION_SUPABASE_REF =

CORRECT_LAUNCHER_SOURCE =
CORRECT_LAUNCHER_COMMIT =
CORRECT_BUILD_PATH =
REDEPLOY_LAUNCHER_ONLY =

GO_LIVE_BLOCKED_BY_LAUNCHER =

不要修改任何東西。
這個任務只查清楚 Launcher 到底部署錯哪一版。