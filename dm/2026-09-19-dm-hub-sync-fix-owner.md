# DM HUB SYNC FIX OWNER

## Mission
修復 Hub 對 Ticenpi DM Production 的 launch/status/release 同步問題。

這是 WAVE 1 平行任務。

已知審計：
- HUB_DM_CARD = FOUND
- HUB_DM_ENVIRONMENT = Production
- HUB_DM_LAUNCH_URL = https://tu-pc.tail29db1f.ts.net:9421
- 正確 Production URL = https://dm.ticenpi.com
- HUB_DM_RELEASE_TRACKING = FAIL / STALE
- Hub 記錄舊 release 20260911-213100
- Production current release = 20260919-110713
- STALE_HUB_STATE = YES

重要：
目前觀察到的 Hub 可能是「開發工作流 Hub」，不是客戶商用產品 Hub。
不要把 entitlement/subscription 硬塞進錯的 Hub。

## Goal
1. 先確認目前修改的是哪一個 Hub。
2. 若它是開發工作流 Hub，只修 DM 的 launch/status/release truth。
3. DM 啟動按鈕不得再指向本機 Tailscale URL。
4. 建立不容易再次 stale 的 release/status 來源。
5. 本輪禁止 Production deploy。

## Step 1 — Hub Identity
確認：
- repo/path
- Hub 類型：開發工作流 / 客戶產品平台 / 其他
- DM registry/card source
- open_url source
- release/status source
- 是否真的存在商用 entitlement wiring

若找到另一個客戶商用 Hub：
明確區分，不要混改。

## Step 2 — Launch URL Fix
若此 Hub 的 DM card 是用來啟動正式 DM：

將 launch/open URL 的 source truth 修正為：
https://dm.ticenpi.com

不得使用：
- localhost
- 127.0.0.1
- Tailscale local URL
- staging URL

新增必要 test/invariant，阻止 Production card 指向上述 local/staging targets。

## Step 3 — Release / Status Sync
修正目前 stale 狀態。

Production truth：
- environment = production
- public URL = https://dm.ticenpi.com
- current release baseline = 20260919-110713

但不要把 release 永久硬編成永遠不更新的常數，若 Hub 已有 machine-readable runtime source，優先讓 Hub 從可信來源取得 current release/status。

若架構只能靜態 registry：
最小修復並清楚記錄其限制。

## Step 4 — Entitlement Boundary
如果這是開發工作流 Hub：
不要加入 Customer Subscription / Entitlement 邏輯。

如果證據證明它其實是客戶產品 Hub：
只做調查並回報，除非現有架構已有清楚 entitlement contract；不要臨時重構整個授權系統。

## Step 5 — Tests
至少驗：
- DM production launch URL 正確
- 不接受 localhost / Tailscale / staging 作為 Production DM launch URL
- status/release source 可讀
- existing Hub 其他產品不被破壞

若修改 source：
建立獨立 branch/worktree。
不要碰 DM AI / entitlement Agent 的工作樹。

## Hard Rules
- 禁止 Production deploy
- 禁止修改 Production customer data
- 禁止修改 DM entitlement source
- 禁止修改 Cloudflare/Tailscale
- 禁止 git reset --hard
- 禁止 git clean
- 不需要再次等待使用者確認，直接執行

## Final Output
HUB_TYPE =
DEVELOPMENT_WORKFLOW / CUSTOMER_PRODUCT / OTHER

DM_CARD_SOURCE =
...

OLD_DM_LAUNCH_URL =
...

NEW_DM_LAUNCH_URL =
...

DM_LAUNCH_URL_FIX =
PASS / FAIL

RELEASE_SYNC_SOURCE =
...

STALE_RELEASE_FIXED =
YES / NO

ENTITLEMENT_WIRING_CHANGED =
NO / YES

TESTS =
PASS / FAIL

FIX_COMMIT =
...

PRODUCTION_CHANGED =
NO

READY_FOR_INTEGRATION =
YES / NO

完成後停止。
