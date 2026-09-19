# W2A-FIX — Verify entitlement remains a hard gate

- 抬頭：POST WAVE2 FRONTEND ACCESS CORRECTION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：小型
- Base：repair/post-frontend-access-state（目前 W2A implementation branch）

---

你現在是：

POST WAVE2 FRONTEND ACCESS CORRECTION OWNER

這不是重新設計 W2A。

目前 W2A summary 顯示：

- studio_offline → informational
- account_mismatch → informational
- signed_out → blocking
- no_tenant → blocking

但原 frozen contract 明確要求：

- not_entitled 必須仍然是 hard block
- real authentication failure 必須仍然是 hard block
- backend_unavailable 不得被誤判為 signed_out

因此本輪只做 targeted verification + 必要最小修正。

## HARD RULES

禁止修改 backend、extension、deploy、docker、AccountStatusList、session.ts。

只允許修改 W2A ownership 內的 frontend access files + dedicated tests。

不要重構整個 SessionGate。

## REQUIRED CHECKS

1. 找出 SessionGate 對以下 reason 的實際行為：
   - signed_out
   - no_tenant
   - not_entitled
   - studio_offline
   - account_mismatch
   - backend_unavailable
   - unknown auth failure（若 source 有）

2. 必須符合：
   - signed_out → BLOCK
   - no_tenant → BLOCK
   - not_entitled → BLOCK
   - studio_offline → ALLOW WEB SHELL
   - account_mismatch → ALLOW WEB SHELL
   - backend_unavailable → 不得 masquerade 成 signed_out；依既有 UI contract 顯示 service unavailable / retry 類狀態
   - real authentication failure → BLOCK

3. 若目前 not_entitled 已被錯誤放行：
   - 做最小修正
   - 補 dedicated test

4. 若目前 source 其實已正確 block not_entitled，只是 summary 寫錯：
   - 不要改 production source
   - 只補/跑足夠 tests 證明

## TESTS

至少新增或確認：
- not_entitled blocked
- signed_out blocked
- no_tenant blocked
- studio_offline not blocked
- account_mismatch not blocked
- backend_unavailable not mapped to signed_out

執行：
- targeted test
- full frontend test
- tsc --noEmit
- next build

## COMMIT

若需要 source/test 修正：
fix(post): preserve entitlement gate in web access

若不需要 source 修改，只需 test：
test(post): verify frontend access gate matrix

不要 push。

## REQUIRED OUTPUT

```
TASK=W2A_ENTITLEMENT_VERIFICATION

CURRENT_BRANCH=
CURRENT_HEAD=

FILES_CHANGED=

SIGNED_OUT_GATE=
PASS/FAIL

NO_TENANT_GATE=
PASS/FAIL

NOT_ENTITLED_GATE=
PASS/FAIL

STUDIO_OFFLINE_INFORMATIONAL=
PASS/FAIL

ACCOUNT_MISMATCH_INFORMATIONAL=
PASS/FAIL

BACKEND_UNAVAILABLE_MAPPING=
PASS/FAIL

REAL_AUTH_FAILURE_GATE=
PASS/FAIL/NOT_PRESENT

SOURCE_FIX_REQUIRED=
YES/NO

TESTS=
TYPECHECK=
BUILD=

COMMIT=

READY_FOR_INTEGRATION=
YES/NO
```
