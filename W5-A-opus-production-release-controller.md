# W5-A — OPUS 5.5 Production Release Controller

ROLE: Ticenpi Release Controller / Production Owner
MODEL: OPUS 5.5

## 任務
你負責總控，不要重做所有程式實作。你的工作是讀取目前 repo / deploy / release evidence / CI / runtime 狀態，整合 W5-B 與 W5-C 的結果，完成安全的 Production promotion。

工作區：
F:\00-Ticenpi-SaaS

主要 repo：
- Platform: F:\00-Ticenpi-SaaS\ticenpi-platform
- Post: F:\00-Ticenpi-SaaS\TicenpiPost
- DM: F:\00-Ticenpi-SaaS\TicenpiDM
- OCR/Letter: F:\00-Ticenpi-SaaS\TicenpiLetter
- Launcher: F:\00-Ticenpi-SaaS\Ticenpi-Launcher
- Deploy: F:\00-Ticenpi-SaaS\deploy

## 已知狀態
- Platform Staging = PASS
- Post Staging = PASS
- DM Staging = PASS
- Launcher Workbench Staging contract E2E = PASS
- DM successful Staging release = 20260924-213157
- DM_SEAT_POLICY=require
- DM assigned protected API=200 / unassigned=403
- DM deploy-config commit = 0f146ca7a5275c2d45e0f5fc16e9ca511405b876
- OCR commercial product_code = ocr
- Letter 是 OCR internal component
- OCR 目前 blocker：product_seat_status 漏傳 p_product_code='ocr'，造成 PGRST202/404 → 503 CENTRAL_SEAT_UNAVAILABLE

## 你要等待 / 讀取的兩個子任務結果
W5-B（SONNET 5）：
- OCR minimal fix
- OCR CI / Staging deploy / E2E
- release evidence / deploy-status-promote tooling implementation

W5-C（KIMI 3 或 MiMo V2.6 Pro）：
- read-only audit
- Post/DM frozen Staging evidence cross-check
- Production read-only preflight
- 591/Sign compatibility audit
- evidence/promote contract verification

如果兩份 handoff 尚未完成：
- 先做你自己的 read-only Production preflight
- 不要與 W5-B 同時修改 OCR/deploy scripts
- 不要搶同一 branch/worktree

## 核心原則
1. Production 不重新 build。
2. 只能 promote Staging 已 ACCEPTED 的 exact immutable digest。
3. source commit / image digest / deploy-config commit / release ID 分開驗證。
4. Production product deploy 必須 serialized：
   Platform Commercial Core → Post → DM → OCR/Letter → Launcher → Canary。
5. 前一產品 FAIL 不得進下一產品。
6. 失敗只 rollback 當前產品。
7. preserve unrelated dirty WIP。
8. 不新增額外 release gate；標準 provenance：
   source SHA → CI PASS → exact digest → deploy pin → running digest → runtime source SHA。
9. Seat/entitlement mutation 只走 canonical admin RPC。
10. OAuth 真人操作、不可逆 Production approval、rollback failure、重大 schema/artifact drift 才停下找使用者。

## Phase 1 — 收斂子任務
確認 W5-B 已交付：
- OCR_STAGING_RELEASE_ACCEPTED=YES
- OCR source/digest/deploy-config/release/rollback evidence
- deploy.ps1 / release.ps1 / promote.ps1 的實際可用狀態

確認 W5-C 已交付：
- POST_STAGING_RELEASE_ACCEPTED
- DM_STAGING_RELEASE_ACCEPTED
- Production preflight
- evidence chain cross-check
- blocker list（若有）

若證據不足，不猜，補最小 read-only 查證。

## Phase 2 — Production Commercial Core
任何產品 deploy 前：
- 確認 Production Supabase/project identity
- fresh backup
- backup readability
- schema drift preflight
- 套用必要 canonical Commercial Core / Central Seat / Admin migrations
- 驗證 entitlement resolver / product_seat_status / assign/release / admin listing / member/customer lifecycle / RLS/security/search_path / seat-limit concurrency
- rollback proof

PASS 才：
PRODUCTION_COMMERCIAL_CORE=PASS

## Phase 3 — Post Production
只使用 Post accepted Staging exact digest。
執行 promote。
驗：
- runtime identity
- health/auth
- assigned allow
- unassigned deny
- tenant isolation
- rollback target

PASS：
POST_PRODUCTION=PASS

## Phase 4 — DM Production
只使用 DM accepted Staging exact digest。
必須 DM_SEAT_POLICY=require。
驗：
- runtime source/digest/config identity
- health / ready / deep health
- RLS
- assigned allow
- unassigned deny
- fail closed

PASS：
DM_PRODUCTION=PASS

## Phase 5 — OCR/Letter Production
只有 OCR_STAGING_RELEASE_ACCEPTED=YES 才能 promote。
保持 product_code=ocr。
Letter 不建立獨立 entitlement/Seat。
驗：
- runtime identity
- health/auth
- assigned OCR API allow
- unassigned deny
- Letter internal
- user-owned RLS
- product_seat_status request 正確帶 p_product_code='ocr'

PASS：
OCR_PRODUCTION=PASS

## Phase 6 — Launcher Production
Launcher 不建立 Staging App。
使用已驗證 canonical candidate。
確認：
- Production Supabase config
- Workbench
- updater/stable channel
- installer/package identity
- rollback/update fallback

PASS：
LAUNCHER_PRODUCTION=PASS

## Phase 7 — Production Canary
需要真人 OAuth 時停在該步。
- Post Facebook OAuth → callback → binding → Studio reconnect → real smoke publish
- DM assigned-user protected API smoke
- OCR assigned-user OCR + Letter smoke
- Launcher platform_admin Customer/member/Seat smoke

PASS：
PRODUCTION_CANARY=PASS

## 最終回報
只輸出：
PRODUCTION_COMMERCIAL_CORE =
POST_PRODUCTION =
DM_PRODUCTION =
OCR_PRODUCTION =
LAUNCHER_PRODUCTION =
PRODUCTION_CANARY =
GO_LIVE =

以及每個產品：
SOURCE_COMMIT =
BACKEND_DIGEST =
FRONTEND_DIGEST =
DEPLOY_CONFIG_COMMIT =
PRODUCTION_RELEASE_ID =
ROLLBACK_RELEASE_ID =

若需要真人：
MANUAL_ACTION_REQUIRED =
STATE_TO_RESUME_FROM =

除非真正 HARD STOP，否則持續到 GO_LIVE。