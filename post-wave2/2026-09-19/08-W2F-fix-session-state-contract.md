# W2F-FIX — Correct Wave2 session-state regression contract

- 抬頭：POST WAVE2 REGRESSION CONTRACT CORRECTION OWNER
- 模型：DeepSeek V4.1 Flash
- 預計工程大小：小型
- Base branch：repair/post-wave2-tests-v2
- Current candidate commit：43186fec18577ec52c897eb498cad9af15d3c51c

---

你現在是：

POST WAVE2 REGRESSION CONTRACT CORRECTION OWNER

這不是產品功能開發。

目前 W2F regression pack 有 4 個 failure，被誤歸類為 W2A：

- test_web_path_works_when_require_studio_is_zero
- test_require_studio_false_bypasses_studio_offline
- test_require_studio_false_bypasses_account_mismatch
- test_session_state_web_mode_reports_studio_online

目前回報宣稱：
TICENPI_REQUIRE_STUDIO=0 時應把 session state 的 studio_online 強制視為 True。

這與 Wave1 已凍結架構衝突。

## FROZEN CONTRACT

Wave1 已確認：

1. Web/Product access 不再以 Studio heartbeat 為 product gate。
2. Studio heartbeat / lease 仍保留作為 Desktop capability 狀態。
3. /api/session/state 仍應回報真實的：
   - studio_offline
   - account_mismatch
4. studio_offline / account_mismatch 是 informational，不應阻擋 Web access。
5. 不得為了 Web access 可用而偽造 studio_online=True。

換句話說：

WEB ACCESS GATE
與
SESSION CAPABILITY STATE

是兩件不同的事。

## TASK

只修正 W2F regression tests 的錯誤 contract。

不要修改 production source。

先檢查 4 個 failing tests 的實際 assertion。

若它們要求：
- require_studio=0 → studio_online=True
- require_studio=0 → 隱藏 studio_offline/account_mismatch

則改成正確 contract：

- Web API access 可成功，即使 Studio offline
- /api/session/state 仍可真實回報 studio_offline
- /api/session/state 仍可真實回報 account_mismatch
- informational state 不等於 access denial

## OWNERSHIP

只允許修改：

backend/tests/test_wave2_*.py
frontend/tests/wave2-*.tsx

不得修改：

backend/app/*
frontend/src/*
extension/*
deploy/*
docker-compose.yml

PRODUCTION_SOURCE_MODIFIED 必須 NO。

## IMPORTANT

不要把這 4 個 failure 丟給 W2A。

W2A 只負責 Frontend SessionGate / Web overlay 行為。

如果測試真的需要 backend source 才能通過，且 expectation 與上述 frozen contract 一致：

STOP 並回報：
CROSS_OWNERSHIP_REQUIRED

不要自行改 backend。

## VALIDATION

完成後跑所有 Wave2 regression tests。

目標：

- Studio offline does not block Web access
- Account mismatch does not block Web access
- Session state still reports actual Studio capability
- Entitlement deny still blocks
- tenant key config tests仍正常
- publish safety tests如果等待W2B，明確標 EXPECTED_PENDING_W2B

## COMMIT

如果只修 tests：

test(post): correct wave2 studio capability contract

不要 push。

## REQUIRED OUTPUT

```
TASK=W2F_SESSION_STATE_CONTRACT_FIX

BASE_COMMIT=
PREVIOUS_COMMIT=43186fec18577ec52c897eb498cad9af15d3c51c
BRANCH=
COMMIT=

FILES_CHANGED=

PRODUCTION_SOURCE_MODIFIED=NO

OLD_EXPECTATION=

CORRECTED_EXPECTATION=

SESSION_STATE_REPORTS_REAL_STUDIO_STATE=
PASS/FAIL

WEB_ACCESS_INDEPENDENT_OF_STUDIO=
PASS/FAIL

TESTS=

EXPECTED_PENDING_W2A=

EXPECTED_PENDING_W2B=

CROSS_OWNERSHIP_REQUIRED=

READY_FOR_INTEGRATION=
YES/NO
```
