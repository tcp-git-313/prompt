# W5 — Ticenpi Production Promotion + Release Evidence Master（個人化版）

## OWNER / MODEL
OWNER: RELEASE & PRODUCTION OWNER
推薦：OPUS 5.5（主控）
Executor：SONNET 5
Verifier：KIMI 3 / MiMo V2.6 Pro（只做 read-only audit / log / evidence cross-check，除非 OWNER 明確交付 mutation）

## 我的工作方式
不要重新規劃整套系統。接手目前已完成的 Staging 成果，目標是：
1. POST、DM 從已驗證 Staging 版本一路 Promotion 到 Production。
2. OCR 先修完目前 Staging blocker、完成 Staging E2E，再 Promotion 到 Production。
3. 建立一套通用 release evidence / promote 機制，讓之後產品操作能固定成：
   .\deploy.ps1 <product> staging
   .\release.ps1 <product> status
   .\promote.ps1 <product> production
4. 每一階段必須留下可機器讀取的 evidence 檔，不再靠人工記 commit/digest。
5. Production 不重新 build；只能 promote 已在 Staging 驗收過的 exact immutable digest。

## 工作區
F:\00-Ticenpi-SaaS
Platform: F:\00-Ticenpi-SaaS\ticenpi-platform
Post: F:\00-Ticenpi-SaaS\TicenpiPost
DM: F:\00-Ticenpi-SaaS\TicenpiDM
OCR/Letter: F:\00-Ticenpi-SaaS\TicenpiLetter
Launcher: F:\00-Ticenpi-SaaS\Ticenpi-Launcher
Deploy: F:\00-Ticenpi-SaaS\deploy

## 已知狀態
Platform Staging = PASS
Post Staging = PASS
DM Staging = PASS
Launcher Workbench Staging contract E2E = PASS

DM successful Staging:
- release 20260924-213157
- DM_SEAT_POLICY=require
- assigned 200
- unassigned 403
- deploy-config 0f146ca7a5275c2d45e0f5fc16e9ca511405b876

OCR 是唯一尚未 Freeze 的產品。
OCR commercial product_code = ocr。
Letter 是 OCR internal component。
已知 OCR blocker：product_seat_status 呼叫缺少 p_product_code='ocr'，造成 PGRST202/404 → 503 CENTRAL_SEAT_UNAVAILABLE。

## 全域規則
- 先查證再修改，不猜。
- preserve unrelated dirty WIP；release 工作用 isolated worktree。
- 已 PASS 的 broad audit 不重做。
- source commit / image digest / deploy-config commit / release ID 分開記錄。
- Staging/Production 只用 immutable sha256 digest。
- artifact chain：source SHA → CI PASS → exact digest → deploy pin → running digest → runtime source SHA。
- Production deploy 不平行；產品逐一 Promotion。
- 一個產品失敗只 rollback 該產品，不往下一個產品繼續。
- Central Commercial Core 是唯一商用授權 source of truth。
- Seat/entitlement mutation 走 canonical admin RPC。
- auth/commercial error fail closed。
- browser 沒 logout 不是 blocker；使用 isolated browser context。
- 只有 OAuth 真人操作、不可逆 Production approval、rollback failure、或重大 Production drift 才停下找我。

# PHASE A — 先建立 Release Evidence 標準

先 audit F:\00-Ticenpi-SaaS\deploy 現有 scripts / services manifest / release layout，不要先假設檔名或重寫整套 deployment。

在不破壞現有 deploy chain 的前提下，建立一個通用 evidence contract，至少支援：
post
dm
ocr
591
sign

但本任務實際 Production Promotion 只包含：
post
dm
ocr

Evidence 必須至少包含：
product
environment
status
source_commit
backend_digest
frontend_digest（若該服務沒有則 null）
deploy_config_commit
release_id
rollback_release_id
ci_run_id
accepted_at
accepted_by
runtime_identity_verified
health_verified
auth_verified
commercial_verified
e2e_verified

建議結構可依現有 deploy repo 慣例調整，但必須同時保留：
- immutable 歷史 release evidence
- 每產品目前 accepted Staging pointer/file

例如概念上：
release-evidence/<product>/staging/<release_id>.json
release-evidence/<product>/staging/accepted.json
release-evidence/<product>/production/<release_id>.json

若 repo 已有等價結構，沿用，不要重複發明。

規則：
- deploy staging 成功只能產生 DEPLOYED evidence。
- mandatory Staging E2E 全 PASS 後才能標 ACCEPTED。
- accepted.json 只能指向 / 複製一個已驗證 immutable release。
- promote production 只能讀 accepted Staging evidence。
- Production 不重新 build image。
- promoted Production digest 必須等於 accepted Staging digest。
- Production 完成後也留下 Production release evidence。
- rollback 後 evidence 要記錄 rolled_back_from / restored_release。

# PHASE B — 統一操作入口

目標操作：
.\deploy.ps1 post staging
.\deploy.ps1 dm staging
.\deploy.ps1 ocr staging

.\release.ps1 post status
.\release.ps1 dm status
.\release.ps1 ocr status

.\promote.ps1 post production
.\promote.ps1 dm production
.\promote.ps1 ocr production

若現有 deploy.ps1 介面不同：
優先做相容 wrapper，不要破壞現有已驗證腳本。

release status 至少顯示：
STAGING_RELEASE_ACCEPTED
SOURCE_COMMIT
BACKEND_DIGEST
FRONTEND_DIGEST
DEPLOY_CONFIG_COMMIT
STAGING_RELEASE_ID
ROLLBACK_RELEASE_ID

promote 必須在 mutation 前自動檢查：
- accepted evidence exists
- status=ACCEPTED
- exact digest available
- Production target identity correct
- rollback target exists
- required secrets/config present
- schema/preflight PASS

# PHASE C — OCR Staging 收尾

1. isolated worktree。
2. 再證明 product_seat_status request 缺 p_product_code。
3. 最小修正：送 p_product_code='ocr'。
4. 不改 Platform schema / OCR RLS / Letter boundary / product_code。
5. focused tests：
   - request body 正確
   - assigned allow
   - unassigned 403
   - invalid JWT 401
   - RPC error/malformed fail closed
6. relevant regression + CI。
7. 取得新 immutable digest。
8. .\deploy.ps1 ocr staging
9. 驗 runtime identity / health / auth。
10. isolated browser E2E：
    assigned → OCR API 200 + Letter works
    unassigned → auth success + 403
11. PASS 後寫入 OCR Staging ACCEPTED evidence。

# PHASE D — POST / DM Product-Level Freeze

不要等 OCR 才開始 Production read-only work。

立即從現有 Post / DM Staging evidence 或 runtime/CI 證據重建並驗證：
POST_STAGING_RELEASE_ACCEPTED=YES
DM_STAGING_RELEASE_ACCEPTED=YES

只能在證據完整時建立 accepted evidence；不得猜 digest。

DM 必須確認：
DM_SEAT_POLICY=require

# PHASE E — Production Read-only Preflight（可與 OCR 平行）

對 Platform/Post/DM/OCR 做 read-only：
- Production Supabase/project identity
- schema drift
- migrations required
- Production DNS/service identity
- secrets/config presence
- frozen/accepted artifacts
- rollback targets
- deploy manifest collision/drift
- Facebook/Studio canary prerequisites

不要在 read-only preflight 階段 mutate Production。

# PHASE F — Production Execution

Production mutation 必須 serialized。

若 Production Commercial Core 尚未具備 frozen Staging contract：
先 backup → migration/preflight → apply canonical Commercial Core/Central Seat/Admin migrations → regression → rollback proof。

然後依序：

1. POST
.\promote.ps1 post production
驗 runtime identity / health / JWT / assigned / unassigned / tenant isolation。
PASS 後寫 Production evidence。

2. DM
.\promote.ps1 dm production
保持 DM_SEAT_POLICY=require。
驗 runtime identity / health / ready / RLS / assigned / unassigned。
PASS 後寫 Production evidence。

3. OCR
只有 OCR_STAGING_RELEASE_ACCEPTED=YES 才能執行：
.\promote.ps1 ocr production
驗 product_code=ocr、OCR API、Letter internal、RLS、assigned/unassigned。
PASS 後寫 Production evidence。

任何一項 FAIL：
rollback 當前產品
記錄 rollback evidence
停止後續 Production products。

# PHASE G — Production Canary

POST：
Facebook OAuth → callback → binding → Studio reconnect → real smoke publish。

DM：
assigned-user protected API smoke。

OCR：
assigned-user OCR + Letter smoke。

需要真人 OAuth 時只停在那一步，完成後同 session 繼續。

# PHASE H — 最後驗證「以後可以直接 Staging / Production」

對 post / dm / ocr 實際證明：
1. deploy.ps1 <product> staging 能完成 deploy chain。
2. PASS 後會產生 immutable Staging evidence。
3. acceptance 後 accepted evidence 正確。
4. release.ps1 <product> status 能正確讀取。
5. promote.ps1 <product> production 只使用 accepted exact digest。
6. Production 完成後會產生 Production evidence。
7. rollback 會留下 evidence 並恢復正確 release。

對 591 / sign：
本任務不做 Production deployment。
只 audit 是否可接入相同 evidence/promote contract，列出缺口，不要擴大修改。

## 最終輸出

POST_STAGING_RELEASE_ACCEPTED =
DM_STAGING_RELEASE_ACCEPTED =
OCR_STAGING_RELEASE_ACCEPTED =

POST_PRODUCTION =
DM_PRODUCTION =
OCR_PRODUCTION =

POST_EVIDENCE_CHAIN =
DM_EVIDENCE_CHAIN =
OCR_EVIDENCE_CHAIN =

DEPLOY_COMMAND_STANDARDIZED =
STATUS_COMMAND_STANDARDIZED =
PROMOTE_COMMAND_STANDARDIZED =

591_COMPATIBILITY =
SIGN_COMPATIBILITY =

PRODUCTION_CANARY =
GO_LIVE =

最後另外輸出三個實際操作範例：
.\deploy.ps1 dm staging
.\release.ps1 dm status
.\promote.ps1 dm production

不要再把正常工程問題拆成很多新 Prompt；除非是真正 HARD STOP，否則持續執行到本任務結束。
