# W5-A — OPUS 5.5 Production Release Controller

ROLE: Production Release Controller
MODEL: OPUS 5.5

工作區：F:\00-Ticenpi-SaaS

目標：
接手已完成的 Staging 成果，整合 W5-B（SONNET 5 實作）與 W5-C（KIMI 3 / MiMo V2.6 Pro 驗證）結果，完成 Production promotion。

已知：
- Platform Staging PASS
- Post Staging PASS
- DM Staging PASS
- Launcher Workbench Staging contract E2E PASS
- DM successful Staging release 20260924-213157
- DM_SEAT_POLICY=require
- DM assigned=200 / unassigned=403
- DM deploy-config commit 0f146ca7a5275c2d45e0f5fc16e9ca511405b876
- OCR product_code=ocr
- Letter 是 OCR internal component
- OCR blocker：product_seat_status 漏傳 p_product_code='ocr'

規則：
1. Production 不重新 build。
2. 只能 promote Staging ACCEPTED 的 exact immutable digest。
3. source commit / image digest / deploy-config / release ID 分開驗證。
4. Production 依序：Commercial Core → Post → DM → OCR/Letter → Launcher → Canary。
5. 前一階段 FAIL 不得進下一階段。
6. 失敗只 rollback 當前產品。
7. preserve unrelated dirty WIP。
8. 標準 provenance：source SHA → CI PASS → exact digest → deploy pin → running digest → runtime source SHA。
9. 不新增額外 mandatory gate。
10. OAuth 真人操作、不可逆 Production approval、rollback failure、重大 schema/artifact drift 才停下。

先讀 W5-B / W5-C handoff：
- W5-B：OCR fix、OCR CI/Staging/E2E、release evidence tooling
- W5-C：Post/DM evidence、Production read-only preflight、tooling cross-check

若 handoff 未完成，先做 read-only preflight，不要與 W5-B 同時修改 OCR 或 deploy scripts。

Phase 1 — Production Commercial Core
- Production project identity
- fresh backup + readability
- schema drift
- apply required canonical Commercial Core/Central Seat/Admin migrations
- verify entitlement/Seat/admin/member/customer/RLS/security/concurrency
- rollback proof
PASS 才 PRODUCTION_COMMERCIAL_CORE=PASS

Phase 2 — Post Production
只用 Post accepted Staging exact digest。
驗 runtime identity、health/auth、assigned/unassigned、tenant isolation、rollback。
PASS：POST_PRODUCTION=PASS

Phase 3 — DM Production
只用 DM accepted Staging exact digest。
必須 DM_SEAT_POLICY=require。
驗 runtime identity、health/ready/deep health、RLS、assigned/unassigned、fail closed。
PASS：DM_PRODUCTION=PASS

Phase 4 — OCR/Letter Production
只有 OCR_STAGING_RELEASE_ACCEPTED=YES 才能 promote。
保持 product_code=ocr；Letter 不另建 entitlement/Seat。
驗 runtime identity、health/auth、assigned OCR API、unassigned deny、Letter internal、RLS。
PASS：OCR_PRODUCTION=PASS

Phase 5 — Launcher Production
Launcher 不建立 Staging App。
使用已驗證 candidate，確認 Production Supabase config、Workbench、updater/stable channel、installer/package、rollback。
PASS：LAUNCHER_PRODUCTION=PASS

Phase 6 — Canary
- Post Facebook OAuth/callback/binding/Studio reconnect/real smoke publish
- DM assigned-user smoke
- OCR assigned-user OCR + Letter smoke
- Launcher platform_admin Customer/member/Seat smoke
需要真人 OAuth 時停在該步。

最終只回：
PRODUCTION_COMMERCIAL_CORE =
POST_PRODUCTION =
DM_PRODUCTION =
OCR_PRODUCTION =
LAUNCHER_PRODUCTION =
PRODUCTION_CANARY =
GO_LIVE =

若需真人：
MANUAL_ACTION_REQUIRED =
STATE_TO_RESUME_FROM =

除非真正 HARD STOP，否則持續到 GO_LIVE。