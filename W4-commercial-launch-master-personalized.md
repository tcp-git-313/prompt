# W4 — Ticenpi Commercial Launch Master（個人化版）

## OWNER
COMMERCIAL LAUNCH OWNER

## 推薦模型
GPT-5.6 Sol High

## 我的工作方式
你不是來重新規劃整個專案，而是接手目前已經做到最後階段的 Ticenpi 商用上線。

執行原則：
- 先查證，再修改；不要猜。
- 已 PASS 的項目不要重做大型 audit。
- 保留所有 unrelated dirty WIP；release 工作使用 isolated worktree。
- 不要把「額外證據」升級成新的 mandatory gate。
- 遇到一般工程問題，自己找 root cause → 最小修正 → tests → CI → 繼續。
- 沒有產品 logout 按鈕不是 blocker；用獨立 browser context/profile。
- 只有 OAuth 真人操作、不可逆 Production 操作、rollback failure、或真正 identity/schema 衝突才停下來找我。
- 回報要短，只維護進度表與 blocker，不要一直重述背景。

## 工作區
F:\00-Ticenpi-SaaS

Platform: F:\00-Ticenpi-SaaS\ticenpi-platform
Post: F:\00-Ticenpi-SaaS\TicenpiPost
DM: F:\00-Ticenpi-SaaS\TicenpiDM
ORC/Letter: F:\00-Ticenpi-SaaS\TicenpiLetter
Launcher: F:\00-Ticenpi-SaaS\Ticenpi-Launcher
Deploy: F:\00-Ticenpi-SaaS\deploy

## 現在已確認
Platform Staging = PASS
Post Staging = PASS
DM Staging = PASS
Launcher Workbench Staging contract E2E = PASS

DM current successful Staging:
- release 20260924-213157
- DM_SEAT_POLICY=require
- assigned protected API 200
- unassigned 403
- deploy-config 0f146ca7a5275c2d45e0f5fc16e9ca511405b876

ORC 是目前唯一 Staging blocker。
ORC commercial product_code = ocr。
Letter 是 ORC internal component，不建立獨立 entitlement/Seat。

已證明 ORC bug：
product_seat_status 呼叫漏掉 p_product_code='ocr'
→ PostgREST PGRST202/404
→ backend fail closed 503 CENTRAL_SEAT_UNAVAILABLE。

Customer / entitlement / Seat 本身已存在，不要先去重建 fixture。

## Phase 1 — ORC 最小修正
1. 建立 isolated worktree。
2. 再確認實際 call site 與 request body。
3. 最小修正：RPC body 傳入 p_product_code='ocr'。
4. 不改 Platform schema、ORC RLS、Letter boundary、product code。
5. focused tests 至少包含：
   - request body 有 p_product_code=ocr
   - assigned → allow
   - unassigned → 403
   - invalid JWT → 401
   - RPC unavailable/malformed → fail closed / 503
6. 跑 relevant regression + git diff --check。
7. 建立最小 commit，跑正常 CI。
8. 記錄 source SHA、build run、immutable digest。
9. pin exact digest，只部署 ORC Staging。
10. 驗 running digest/source/config/health/auth。

## Phase 2 — ORC Browser E2E
使用兩個獨立 browser context。

Assigned:
→ login
→ protected OCR API 200
→ Letter 可用

Unassigned:
→ login success
→ ORC commercial deny / 403
→ protected data/API 不載入
→ Letter 不可 bypass

證明兩者同 customer、同 ocr entitlement、同 ordinary class，唯一差異是 Seat。

完成後：
ORC_STAGING_E2E=PASS

## Phase 3 — STAGING COMMERCIAL FREEZE
只有 Platform/Post/DM/ORC/Launcher 全部 PASS 才 Freeze。

記錄每個產品：
APP_SOURCE_COMMIT
BACKEND_IMAGE_DIGEST
FRONTEND_IMAGE_DIGEST
DEPLOY_CONFIG_COMMIT
STAGING_RELEASE
ROLLBACK_RELEASE

然後：
STAGING_COMMERCIAL_FREEZE=YES

Freeze 後不要混入 UI、新功能、auth cleanup 或 extraction 改動。

## Phase 4 — Production Read-only Preflight
在任何 Production mutation 前，先自動完成 read-only preflight：
- 確認 Production Supabase/project identity
- schema drift comparison
- frozen artifact identities
- rollback targets
- migration plan
- Post/DM/ORC/Launcher cutover順序
- Facebook/Studio canary plan

若 preflight 有真正衝突，報 exact blocker。
若全部可安全執行，停在第一個不可逆 Production mutation 前，要求我一次批准進入 Production execution。

## Phase 5 — Production Execution（批准後）
順序固定：
1. Production Commercial Core
2. Post Production
3. DM Production
4. ORC/Letter Production
5. Launcher Production
6. Production Canary
7. GO LIVE

每一階段必須 PASS 才能進下一階段。
失敗只 rollback 當前產品，不要繼續往後。

標準 artifact proof：
source SHA
→ CI exact digest
→ deploy-config pin
→ running digest
→ runtime source identity

DM 必須保持 DM_SEAT_POLICY=require。
ORC 必須保持 product_code=ocr。
Launcher 不建立 Staging App。

Production Canary 最後至少：
- Post Facebook OAuth/callback/account binding/Studio reconnect/real smoke publish
- DM assigned-user protected API smoke
- ORC assigned-user OCR + Letter smoke
- Launcher platform_admin Customer/member/Seat smoke

## 回報格式
只維護：

ORC_STAGING_FIX =
ORC_STAGING_E2E =
STAGING_COMMERCIAL_FREEZE =
PRODUCTION_PREFLIGHT =
PRODUCTION_COMMERCIAL_CORE =
POST_PRODUCTION =
DM_PRODUCTION =
ORC_PRODUCTION =
LAUNCHER_PRODUCTION =
FACEBOOK_STUDIO_CANARY =
PRODUCTION_CANARY =
GO_LIVE =

如果只差真人操作：
MANUAL_ACTION_REQUIRED =
STATE_TO_RESUME_FROM =

如果是 blocker：
BLOCKER =
ROOT_CAUSE =
SMALLEST_NEXT_ACTION =

不要再自行拆成很多新的 Prompt。
