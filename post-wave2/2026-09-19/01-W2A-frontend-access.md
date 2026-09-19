# W2A — POST Frontend Web Access / SessionGate

- 抬頭：POST WAVE2 FRONTEND ACCESS OWNER
- 模型：MiMo-V2.5
- 預計時間：15–25 分鐘
- 任務類型：Implementation
- Authoritative Base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

---

你現在是：

POST WAVE2 FRONTEND ACCESS OWNER

這是正式 Implementation。

## BASE

Authoritative Wave2 base：

branch:
`repair/post-v2-wave2-base`

commit:
`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

建立：

branch:
`repair/post-frontend-access-state`

worktree:
`F:\00-Ticenpi-SaaS\TicenpiPost-w2-frontend`

必須直接從 exact commit：

`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

建立。

建立後 `git rev-parse HEAD` 必須等於 exact base；`git status --porcelain` 必須 empty，否則 STOP。

## TARGET

Web Product Access 不得再被 `studio_offline`、`account_mismatch` 整站阻擋。

這兩個狀態只代表 Desktop capability / Launcher 狀態。

Google OAuth / Supabase JWT、Membership、Entitlement 仍然是 Web access 正式 gate。

## LOCAL / MOBILE TARGET

LOCAL：
- Google OAuth 保留。
- Chrome Extension 只處理 Facebook capability。
- Launcher 不得成為 Web access prerequisite。

Mobile：
iPhone / Safari：Google OAuth → JWT → Membership → Entitlement → POST Web，不需要 Launcher。

## EXCLUSIVE OWNERSHIP

優先只修改：
- `frontend/src/components/SessionGate.tsx`
- 它自己的 CSS
- dedicated tests

如果 baseline 實際顯示 Access UI 邏輯散落於以下檔案，可在有證據時修改：
- `frontend/src/app/page.tsx`
- `frontend/src/components/AccountWarmBoard.tsx`
- `frontend/src/components/AutomationIndex.tsx`
- `frontend/src/lib/api-client.ts`

禁止修改：
- `frontend/src/components/AccountStatusList.tsx`

G4 已完成。

## FORBIDDEN

禁止修改：
- `backend/*`
- `extension/*`
- `docker-compose.yml`
- `deploy/*`
- `session.ts`
- `AccountStatusList.tsx`

禁止 Production、VPS、push、real Facebook action。

## STATE MATRIX

Hard block：
- signed_out
- no tenant / no membership
- not_entitled
- real authentication failure

Informational only：
- studio_offline
- account_mismatch

backend_unavailable 不得錯誤顯示成 signed_out。

## TESTS

至少驗：
- normal
- signed_out
- no_tenant
- not_entitled
- studio_offline
- account_mismatch
- backend_unavailable

要求：
- studio_offline → Web shell still usable
- account_mismatch → Web shell still usable
- not_entitled → blocked
- signed_out → blocked

執行 targeted frontend tests、`tsc --noEmit`、`next build`；如安全可行跑 full frontend tests。

## CROSS OWNERSHIP

若完成任務必須修改 backend schema、session.ts、AccountStatusList，STOP，回報 `CROSS_OWNERSHIP_REQUIRED`，不要越界。

## COMMIT

`fix(post): make studio state informational in web access`

不要 push。

## REQUIRED OUTPUT

```
TASK=W2A_FRONTEND_ACCESS

BASE_COMMIT=
BRANCH=
WORKTREE=
COMMIT=
FILES_CHANGED=
HARD_BLOCK_STATES=
INFORMATIONAL_STATES=
STUDIO_OFFLINE_WEB_ACCESS=PASS/FAIL
ACCOUNT_MISMATCH_WEB_ACCESS=PASS/FAIL
ENTITLEMENT_GATE=PASS/FAIL
SIGNED_OUT_GATE=PASS/FAIL
BACKEND_UNAVAILABLE_MAPPING=PASS/FAIL
MOBILE_WEB_ACCESS_SOURCE=PASS/FAIL
TESTS=
TYPECHECK=
BUILD=
CROSS_OWNERSHIP_REQUIRED=
OWNERSHIP_VIOLATION=YES/NO
READY_FOR_INTEGRATION=YES/NO
```
