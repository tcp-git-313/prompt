# DM — STAGING SEAT, SHARED ENGINE, YUCT/SHADOW AND ROLLBACK CLOSEOUT

推薦模型：GPT-5.6 Luna — High

## 任務目標

按以下順序完成 **DM Staging closeout**，每一關以本次親查的證據判定，不沿用舊報告的 PASS：

```text
DM Central Seat gate
→ Staging DM entitlement
→ assigned / unassigned E2E
→ platform_admin
→ Shared Engine identity
→ YUCT UI + Shadow
→ Staging rollback / restore
→ DM READY（僅指 Staging closeout）
```

Production 已另行部署，但真人 canary／Production ACCEPTED 尚未證明。本任務不得部署、回滾、修改 Production 或 Production DB，也不得把 Staging READY 寫成 Production GO_LIVE。

## 2026-09-26 起始事實：先重驗，不能直接當結論

- DM Staging `/api/health` 現為 200，`environment=staging`，release `20260924-213157`，source commit `2dd02e2378600f02b9d14590391b9edf84bc14a2`；`runtime-config.js` 指向 Staging Supabase `jlsqjvehwblkeuycjoyj`。
- `F:\00-Ticenpi-SaaS\.release-evidence\dm\accepted-staging.json` 記錄同一 release 為 ACCEPTED，CI run `35968635281`，backend digest `sha256:7f5012ee4acad7267b164e1c3320df88e99bee6691e143303de942175a835468`，frontend digest `sha256:e731f573843874e9436e6654eabaf54f79f4d92c85636410667215fb6f41d33a`；CI run 是 success。執行前仍需核對 VPS active manifest、實跑容器 digest、runtime env。
- Staging config commit `0f146ca7a5275c2d45e0f5fc16e9ca511405b876` 的 compose 將 `DM_SEAT_POLICY` 設為 `require`。source commit 的 `require_dm_access()` 在 require 模式查 Central Seat；中央錯誤回 503，DENY 回 `403 CENTRAL_SEAT_REQUIRED`。
- `F:\00-Ticenpi-SaaS\artifacts\w5a-staging-seat-e2e-20260925\post-dm-seat-e2e.json` 記錄 2026-09-24 18:37Z 的真實 DM `/api/designs` API：固定 assigned 一般使用者 200，固定 unassigned 一般使用者 403 `CENTRAL_SEAT_REQUIRED`，壞／缺 token 401。這是歷史 E2E，不是本次即時重測。
- `F:\00-Ticenpi-SaaS\deploy\docs\system\STAGING_TEST_IDENTITY_CONTRACT.md` 記錄 2026-09-25 09:13Z 的 Staging 唯讀資料查驗：兩位固定使用者同屬 active business customer，DM entitlement active/manual、seat_limit=2；`job0975805890@gmail.com` 應 assigned，`tcp.ai313@gmail.com` 應 unassigned。舊的「DM entitlement 缺失」結論已過時。
- DM source 的 `shared_engine_lock.json` 預期乾淨 source commit `11762508332bb2508538761302a90f8dca5f2586`、wheel `0.1.0`、wheel SHA-256 `e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3`。`GET /api/admin/shared-engine/identity` 存在，但尚未找到正式 Staging platform_admin session 的即時 endpoint 證據。
- YUCT Legacy + Core Shadow 有本機整合報告 `F:\00-Ticenpi-SaaS\TicenpiDM\docs\DM-YUCT-SHADOW-INTEGRATION-REPORT.md`。該報告保留圖片 19 對 18 的 mismatch，並標示 Staging browser/admin E2E 未做；不可寫成 Staging PASS。已部署 source 的 Shadow 預設 disabled、sample rate 0；先查 active runtime 設定。
- `.release-evidence\dm\history` 未見 Staging rollback/restore 演練紀錄。舊 preflight 提及 rollback target `20260924-200109`，但必須在 VPS 重新核對 release 目錄、manifest snapshot、digest、可用性，不可僅憑文字使用。
- 主 checkout `F:\00-Ticenpi-SaaS\TicenpiDM` 在 `dm/integration-20260920`、HEAD `561bd8c...`，有大量未提交 UI/YUCT/OAuth WIP。`F:\00-Ticenpi-SaaS\.worktrees\dm-central-seat-staging-20260924` 的 compose 也有未提交修改。不要 reset、clean、stash、覆蓋或將這些工作樹當成正式 release。乾淨的 W3 config worktree 是 `F:\00-Ticenpi-SaaS\.worktrees\w3-dm-final-config`，但使用前仍要重查 status。
- DM Production 目前 `/api/health` 為 release `20260925-085643`、同 source commit，`.release-evidence\dm\history\20260925-085643.production.DEPLOYED.json` 的 `e2e_verified=false`；只作界線紀錄，不在本任務操作。

## 工作範圍與規則

主要：`F:\00-Ticenpi-SaaS\TicenpiDM`、DM Staging runtime、`F:\00-Ticenpi-SaaS\deploy` 釋出工具與證據；Central Commercial Core 只讀核對。不要修改 Post、591、Sign、ORC、ExtractionHub、Central Seat schema、Production 或 Production DB。

所有資料與執行輸出都不得包含 JWT、cookie、refresh token、Authorization、secret、完整個人識別資訊。使用已定義的固定 Staging 測試身分；不得把 platform_admin 或 `test_access=allow` 當成 assigned/unassigned Seat 夾具。不要建立新的授權或調換 Seat 來「做出 PASS」。

若需要改 DM 程式或 Staging config，先以乾淨的隔離 worktree 建立最小、可審查的候選；保持原始 dirty WIP 不變。source、CI image、Staging runtime 必須能按 SHA/digest 對上。不得為了本任務切 Core Primary 或刪 Legacy YUCT 擷取器。

## Gate 0 — Read-only preflight

1. 記錄目前各 worktree branch/HEAD/status、staged diff、相關 active writer；挑選乾淨且正確的 release source，不覆蓋現有 WIP。
2. 重查 Staging `/api/health`、`runtime-config.js`、Supabase ref、active manifest、後端/前端容器 digest、`DM_SEAT_POLICY` 值、CI run、accepted-staging pointer、前一個可回滾 release 的 manifest snapshot。
3. 若 release identity、環境、digest 或 manifest 不一致，**HARD STOP**；先報告漂移，不進入 auth、YUCT 或 rollback 操作。

## Gate 1 — Central Seat + entitlement

1. 只讀確認固定 customer 的 DM entitlement 仍 active、seat_limit=2，兩位一般使用者仍為 active member，且 Central `product_seat_status('dm')` 分別為 assigned／not_assigned。
2. 在同一已驗證 Staging release，經正式 App/API 流程測 `/api/designs`：assigned 200、unassigned 403 `CENTRAL_SEAT_REQUIRED`、invalid/no token 401。記錄 status、typed code、release、時間與遮罩後的身分；不輸出 token。
3. 用 repo 既有 Seat/entitlement 回歸測試覆蓋：active entitlement＋assigned、active＋unassigned、no entitlement、inactive、個人戶免 Seat、中央 RPC timeout/error→503、`shadow`／`require` 模式差異。若現有測試缺覆蓋才補最小測試。
4. 若資料夾具漂移、`require` 不生效或 typed denial 不符，停在此關；不要藉由改身分或直接改 DB 繼續。

## Gate 2 — platform_admin

1. 使用**真正的 Staging platform_admin** 工作階段，確認 `my_commercial_context()` 的 role、`product_seat_status('dm')` 的 exempt，以及 `/api/designs` 的實際結果。不要在一般使用者 profile 上臨時改 role，不要用 Local bypass 冒充 Staging。
2. 驗 `GET /api/admin/shared-engine/identity` 對 platform_admin 為 200、一般使用者 403、未登入 401；先確認 admin 仍通過 DM auth dependency。
3. 若沒有合法 admin session 或 MFA 尚未完成，標 `PLATFORM_ADMIN_E2E=BLOCKED`，保留此前 PASS 證據，不以測試 mock 補位。

## Gate 3 — Shared Engine identity

用 Gate 2 的正式 Staging admin session 核對 endpoint 回應：`source_clean=true`、source commit `11762508332bb2508538761302a90f8dca5f2586`、wheel version `0.1.0`、wheel SHA-256 `e543ceab5077bcb46a110b3afe186276e9473534fc34a5cb967f182d93ca2bb3`、semantic/presentation/formatter contract version 均為 `1`。任何值不符須停；不要把 source lock 檔案或單元測試當 runtime identity。

## Gate 4 — YUCT UI + Shadow

1. 先只讀確認 active runtime 的 Shadow enabled、sample rate、source allowlist、provider、timeout/concurrency、持久化位置；確認受控 YUCT 範例 URL 仍可合法擷取。失效時記錄，不改用任意第三方房源資料。
2. 用有 DM Seat 的一般使用者做真實 Staging UI smoke：輸入受控 YUCT URL → `/api/scrape` → Legacy 輸出 → DM 表單/圖片可見與保存；記錄前端、API、release 和關鍵欄位。失敗分清 source 下架、網路、auth、parser、UI。
3. Shadow 只比較 Core 與 Legacy，不改使用者回應；確認相同請求的 Shadow record、platform_admin 報表、diagnostics、耗時與敏感資訊遮罩。既有 19/18 圖片 mismatch 必須保留並分類，不可改成 MATCH。
4. 若 Shadow 在 Staging 未啟用，先準備最小 Staging-only、版本化 config candidate，固定同一 CI image digest，列出啟用與回退差異後再部署驗證；不得直接改活容器環境變數或 Production。無法安全建立候選時標 `SHADOW_STAGING=BLOCKED`。

## Gate 5 — Staging rollback / restore

1. 在動作前記錄 current release `20260924-213157` 的 active manifest、兩個 digest、健康、Seat `require`、accepted pointer；逐一核對 rollback target 的實際 release 目錄與 manifest snapshot。歷史文字所列 `20260924-200109` 只是候選。
2. 僅對 `dmruntimestaging` 使用中央部署工具的正式 rollback 流程；先核實 CLI 語法及是否會夾帶 unrelated service。不得手改 symlink、直接 `docker compose` 或重 build。確認 rollback 目標健康和 identity，記錄其功能差異；若舊版 Seat policy 不同，不將它的授權結果誤寫成目前 candidate 的 PASS。
3. 再以原本 ACCEPTED 的**相同兩個 immutable digest**正式 restore `20260924-213157`；核對 release、digest、健康、runtime config、Seat assigned/unassigned、admin identity、YUCT 關鍵 smoke。不得僅有 deploy 成功訊息就算 restore。
4. 依中央 release evidence 制度記錄 rollback/restore；若前版 snapshot、環境或回復路徑不可證明，**HARD STOP**，不要做實際切換。

## 最終判定與交付

輸出簡明矩陣，每關分 `PROVEN LIVE`、`HISTORICAL`、`SOURCE/TEST ONLY`、`BLOCKED`；附 runtime 時間、commit、digest、typed response、證據路徑。明列已做及未做的資料/程式/部署變更，並確認 Production untouched。

只有 Gate 0–5 的必要證據都完成、最後 Staging 運行原 ACCEPTED artifact 且無意外回歸時，才輸出 `DM READY — STAGING CLOSEOUT`。任何關卡缺證據或失敗，輸出 `DM STAGING CLOSEOUT BLOCKED` 和唯一下一個可執行動作。Production 仍依獨立真人 canary／ACCEPTED 流程處理。

## Hard stops

- Supabase ref、release SHA、CI digest、active manifest 或 `DM_SEAT_POLICY=require` 不一致。
- Seat 夾具或 DM entitlement 與固定合約不一致；勿自行改測試資料。
- 無正式 Staging platform_admin session；勿改 role、test_access 或使用 Local bypass。
- YUCT 範例失效、Shadow 無法以版本化 Staging config 安全啟用。
- rollback target 無可驗 manifest snapshot、deploy 會波及 unrelated service，或 restore 不能回原 digest。
- 任何 Production / Production DB 寫入需求。

