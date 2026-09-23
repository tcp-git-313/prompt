# ORC P2 — Source Fix + Local Docker Alignment + Staging Revalidation

ROLE
你是 Ticenpi ORC / Letter 修復 Owner。

Canonical source:
F:\00-Ticenpi-SaaS\TicenpiLetter

Central deploy:
F:\00-Ticenpi-SaaS\deploy

Target local Docker project:
ticenpiletter

KNOWN CONFIRMED FINDINGS
- Staging backend OCR / health / routing 已可工作。
- Browser E2E 失敗根因已確認：frontend persistence 仍使用 legacy records/projects/user_profiles，但 Staging DB 使用 v2_records/v2_projects/v2_user_profiles，造成 PGRST205。
- OCR POST 本身可成功，但 persistence 失敗導致結果不 render、history/init 異常。
- 現有 ticenpiletter 是 stale local build，來源 identity 不可證明。
- Production 可能仍需要 legacy table contract，因此禁止 blanket rename 全環境為 v2。
- orc-core 只驗 OCR engine，不代表 Browser E2E。

GOAL
修好並重新證明：
Canonical Source → Local DEV → Local Docker → CI → VPS Staging → Browser OCR E2E

PHASE 1 — REVALIDATE
從 canonical source 自行找出 supabase.js、runtime config、v2 schema、Docker scripts、compose、tests。快速複核上述根因；若不一致，停止回報。

PHASE 2 — CONTRACT
先核對 legacy/v2 tables、columns、RLS、quota RPC。若 source-only 修復不足、需要改 DB schema，停止回報，不自行改 DB。

PHASE 3 — MINIMUM SOURCE FIX
採 logical resource → runtime-selected table mapping：
- Staging = v2 table set
- Production/default = legacy table set
- DEV offline 行為不退化
- Local Docker Staging-like runtime = v2
集中移除未受控的 legacy table hardcode；quota 只有 contract 已確認才改。
補 regression tests：staging→v2、production/default→legacy、runtime config、schema contract。

PHASE 4 — LOCAL DEV
依 repo 官方啟動方式驗 health、direct/proxy OCR、unit/contract tests、frontend build。全 PASS 才繼續。

PHASE 5 — LOCAL DOCKER ALIGNMENT
先檢查 official local Docker script 與 compose。
使用者要求 project name = ticenpiletter。
如果目前官方流程名稱不同且無法證明安全統一，HARD STOP，不得默默保留兩套 project。
若安全，使用官方流程從 canonical source 重建，image 必須能證明 source SHA / fingerprint / staging environment。

PHASE 6 — LOCAL DOCKER TRUE E2E
真實驗證 login → entitlement → OCR upload → API → persistence → result render → history reload。
必要條件：
- no PGRST205
- OCR 200
- result render
- history reload
- badge 不卡住
FAIL 不得進 CI。

PHASE 7 — CI
只提交本任務必要 diff；不得 git add -A。
Push exact SHA，等待 exact commit CI completed + success。
取得 frontend/backend immutable digests。
任一 required job FAIL 就停止。

PHASE 8 — STAGING
依現有正式流程 pin exact CI digests，使用 clean source 執行既有 DryRun → Preflight → Staging deploy。
不得跳 gate，不得碰其他產品。

PHASE 9 — STAGING TRUE E2E
真 Browser 驗：
runtime identity、login、entitlement、OCR、persistence、render、history、reload、badge。
同步對照 browser Network/Console、frontend/backend logs。
只有 Browser click → request → nginx → backend → OCR → response → persistence → UI render 全部有證據才算 PASS。

HARD STOP
以下任一成立即停：
- v2 schema/RLS/RPC 與 frontend contract 不相容
- 需要改 Supabase schema
- Production legacy default 無法安全保留
- local Docker project-name contract 有架構衝突
- 需要改 shared auth/entitlement/deploy architecture
- canonical/CI/Staging identity 與已知證據不一致

FINAL REPORT
輸出：
1 ROOT CAUSE
2 FILES CHANGED
3 TABLE CONTRACT (DEV / Local Docker / Staging / Production default)
4 LOCAL DEV result
5 LOCAL DOCKER identity + E2E
6 CI exact SHA/run/digests
7 STAGING release/digests/E2E
8 ALIGNMENT matrix
9 Production changed = NO
10 Remaining risks
11 Commits
12 FINAL STATUS = PASS / BLOCKED / FAIL

FINAL PASS 僅能在 Local Docker E2E、exact CI、Staging Browser E2E、no PGRST205、render/history、identity alignment 全部成立時使用。
