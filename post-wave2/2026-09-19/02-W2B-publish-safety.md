# W2B — POST Publish Safety / Action Gates

- 抬頭：POST WAVE2 PUBLISH SAFETY OWNER
- 模型：DeepSeek V4.1 Flash
- 預計時間：20–35 分鐘
- 任務類型：Implementation
- Authoritative Base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

---

你現在是：POST WAVE2 PUBLISH SAFETY OWNER

這是正式 Implementation。

## BASE

Exact base：`b3cc93de970f4c7562eff3a4d5e17614462ae8bb`

建立：
- branch: `repair/post-publish-safety`
- worktree: `F:\00-Ticenpi-SaaS\TicenpiPost-w2-publish-safety`

直接從 exact base 建立，確認 HEAD 等於 exact base 且 worktree clean，否則 STOP。

## GOAL

單一 `TICENPI_AUTO_PUBLISH` 不能無差別同時 arm 所有外部 Facebook write action。

## FIRST STEP

先只讀定位目前真正控制 `gate_publish`、`TICENPI_AUTO_PUBLISH`、normal post、Marketplace、relist、delete、comment 的實際 source。不要根據舊報告猜檔名。

## EXCLUSIVE OWNERSHIP

只有真正 Publish Safety / Worker execution files 屬於你。

禁止修改：
- `backend/app/api/auth.py`
- `backend/app/core/identity.py`
- `backend/app/api/studio.py`
- `backend/app/worker/fb_group_list.py`
- `backend/config/selectors.yaml`
- `frontend/*`
- `extension/*`
- `docker-compose.yml`
- `deploy/*`

## TARGET DESIGN

保留 `TICENPI_AUTO_PUBLISH` 作為 master external-write gate，其下建立最小 action-level gating。

只對實際存在的 action 建 gate，可能包含 general post、marketplace、relist、delete、comment。不存在的 action 不要發明。

## FAIL CLOSED

- 預設 real external write = OFF
- master OFF → 所有 external writes blocked
- master ON + action OFF → 該 action blocked
- master ON + action ON → 該 action 才可進 executor

## IMPORTANT

不要修改 Production env、不要啟用 AUTO_PUBLISH、不要真實 Facebook publish、不要重構整套 worker architecture。只做最低必要安全切分。

## TESTS

至少驗：
- master OFF → all blocked
- master ON + specific OFF → blocked
- master ON + specific ON → allowed
- actions independent
- dry-run / validation path 不得 external write

## W2F CONTRACT

已有 candidate tests：`40ae824`。不要修改它。

如果你建立的 public interface 與 candidate test 預期不同，以實際安全 architecture 為準，但明確回報 `W2F_TEST_CONTRACT_MISMATCH`，不要為了舊 test 寫錯 source。

## COMMIT

`fix(post): separate publish action safety gates`

不要 push。

## REQUIRED OUTPUT

```
TASK=W2B_PUBLISH_SAFETY
BASE_COMMIT=
BRANCH=
WORKTREE=
COMMIT=
FILES_CHANGED=
MASTER_GATE=
ACTION_GATES=
DEFAULT_FAIL_CLOSED=PASS/FAIL
GENERAL_POST=
MARKETPLACE=
RELIST=
DELETE=
COMMENT=
TESTS=
REAL_FB_ACTION=NO
W2F_TEST_CONTRACT_MISMATCH=
CROSS_OWNERSHIP_REQUIRED=
OWNERSHIP_VIOLATION=YES/NO
READY_FOR_INTEGRATION=YES/NO
```
