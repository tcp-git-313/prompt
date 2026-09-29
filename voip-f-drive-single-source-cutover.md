# VOIP F-Drive Single Source Reconciliation + Cutover

## Role
You are the **VOIP Runtime / Database / Deployment Cutover Owner**.

This is a **real modification + cutover task**, not another broad audit.

Your job is to make:

`F:\00-Ticenpi-SaaS\Ticenpi-Voip`

the **single canonical source of truth** for the Ticenpi VOIP project, reconcile the current live WSL runtime safely into that F: project where necessary, migrate the **current production database** from WSL runtime into F:, eliminate all active D: dependencies, cut over the services, and verify the result end-to-end.

Work autonomously through investigation → reconciliation → offline verification → safe cutover → runtime verification.

Do not stop after every small finding.

Only stop for a HARD STOP condition defined below.

---

# USER REQUIREMENT — NON-NEGOTIABLE

The old D: project was already copied to F:.

The required final project root is:

`F:\00-Ticenpi-SaaS\Ticenpi-Voip`

WSL equivalent:

`/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip`

## D: policy

D: is **legacy only**.

You must NOT:

- copy D: project data into F:
- copy D: database into F:
- restore D: backups into F:
- symlink F: to D:
- use D: as rollback source
- use D: as deploy source
- use D: as sync source
- use D: as runtime source
- leave any active service reading/writing D:
- leave any active wrapper pointing to D:
- leave any backup task writing to D:

D: may be inspected read-only only if needed to prove that it is no longer used.

The final runtime must not depend on D: in any way.

Do not delete D: contents in this task unless the user explicitly asks later.
The goal is **zero dependency**, not destructive cleanup.

---

# IMPORTANT SYSTEM-MANAGED EXCEPTIONS

The user wants all **project-owned code, data, database, assets, scripts, runtime state and backups** under the F: project.

System-managed launcher/config locations may remain where Linux/Asterisk requires them, for example:

- `/etc/systemd/system/*.service`
- `/etc/asterisk/*`
- `/usr/local/bin/*`

But these may contain only:

- thin launch/config glue
- direct references to F:
- generated system configuration derived from F:

They must NOT contain independent/stale copies of project logic or project data.

Any compatibility path such as `/opt/voip/*` may remain only as a symlink/redirect whose `realpath` resolves to F:.

---

# VERIFIED AUDIT BASELINE

The previous read-only audit established the following.

## F: structure exists

The F project already contains:

- backend
- engine
- data
- frontend
- prod-audio
- scripts
- db
- Git repository

## Production database truth

The old F: and D: SQLite files are old snapshots.

At audit time:

- F: DB had 2 customers / 46 call_records
- D: DB was byte-identical to the old F: DB
- active runtime DB was:
  `/opt/voip/dbdata/saas_voip.db`
- active runtime DB had approximately:
  - 536 customers
  - 250 call_records
  - 721 record_details
  - 517 task_targets
- active runtime DB passed SQLite integrity check

Therefore:

`D_DB_REQUIRED_FOR_CUTOVER = NO`

and:

`RUNTIME_DB_MUST_BE_PRESERVED = YES`

Do not hardcode the old row counts as current truth.
Re-measure the active runtime DB immediately before cutover.

## Active D: dependencies found

At audit time:

- `/opt/voip/data` → D:
- `/opt/voip/engine` → D:
- `/opt/voip/frontend` → D:
- `/opt/voip/prod-audio` → D:
- `/opt/voip/scripts` → D:
- `/usr/local/bin/saas-voip-sync` hardcoded D:
- existing `cutover-single-source.sh` referenced D:
- existing rollback logic pointed back to D:

## WSL stale copies found

Independent copies existed under:

- `/usr/local/lib/saas/`
- `/opt/voip/backend/`
- `/usr/local/bin/` wrappers

Known divergent files included:

- `saas_streaming_server.py`
- `saas_stt.py`
- `saas_sttd.py`
- `walker.py`
- `backend/server.js`
- `stt_corrections.json`

The audit showed **code drift**, not a simple "runtime is newer" relationship.
F: contains newer authorized VOIP work, while runtime may contain older production hotfixes.

Do not overwrite F wholesale with runtime.
Do not overwrite runtime blindly with F before reconciliation.

## Existing cutover script is unsafe

The current `scripts/cutover-single-source.sh` must NOT be run as-is.

It previously:

- copied D DB backups into F
- defined D as a source
- used D symlinks for rollback

That violates the user requirement.

---

# CURRENT AUTHORIZED VOIP WORK MUST BE PRESERVED

The F: repo may contain uncommitted authorized work from recent VOIP optimization, including but not limited to:

- AudioSocket streaming work
- VAD / barge-in shadow diagnostics
- rolling barge-in accumulator
- exact accumulator regression
- endpoint/STT/dialogue routing improvements
- deterministic-first walker routing
- voice-quality offline test harnesses
- F-drive path corrections

Do not discard, reset, checkout-over, clean, or overwrite this work.

Forbidden Git operations unless strictly read-only:

- `git reset --hard`
- `git clean -fd`
- destructive checkout
- rebase that can lose WIP

Before changing anything, capture the exact current F: Git status and diff.

---

# TARGET FINAL ARCHITECTURE

The desired end state is:

```
F:\00-Ticenpi-SaaS\Ticenpi-Voip
│
├─ backend
├─ engine
├─ data
├─ frontend
├─ prod-audio
├─ scripts
├─ db
│  ├─ saas_voip.db        ← current production DB
│  └─ backup              ← future DB backups
└─ project-owned runtime/config/state
        │
        ▼
/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip
        │
        ├─ systemd services execute F code
        ├─ Asterisk config derives from F
        ├─ wrappers execute F code
        ├─ /opt/voip/* compatibility paths resolve to F
        └─ database realpath resolves to F
```

No active D dependency.

No independent app-code copy under `/usr/local/lib/saas`.

No independent backend copy under `/opt/voip/backend`.

No production DB remaining as an independent canonical copy under `/opt/voip/dbdata`.

---

# EXECUTION MODE

Proceed autonomously through all phases.

You may:

- edit F: project files
- edit safe systemd units after preflight
- edit thin wrappers
- edit Asterisk include/symlink configuration when needed
- rewrite the cutover script
- migrate the live runtime DB to F safely
- update `/opt/voip` compatibility symlinks
- restart VOIP services after all cutover prerequisites pass
- reload Asterisk if required by the verified configuration change
- run tests and healthchecks
- commit the final reconciled source after successful verification

Do not push unless the existing project workflow explicitly requires it and credentials/branch policy are already known.

---

# HARD STOP CONDITIONS

Stop and report instead of guessing if ANY of these occur:

1. An active phone call exists when service stop/cutover is about to begin.
2. The active production DB fails `PRAGMA integrity_check`.
3. The F: filesystem cannot safely open/lock SQLite using a reversible transaction test.
4. Reconciliation finds a runtime-only change whose intent cannot be determined and overwriting it could cause data loss or call-flow regression.
5. A required secret/config exists only in D: and cannot be recovered from current runtime/systemd/environment without copying from D.
6. Database migration verification does not exactly match the immediate pre-cutover production snapshot.
7. Service restart fails and rollback using F/WSL backups cannot restore a healthy state.
8. Any step would require deleting user data.
9. Any step would require using D: as source or rollback.
10. There is evidence that moving the active SQLite DB to `/mnt/f` is unsafe in this WSL environment and cannot pass the explicit SQLite lock/write test below.

Do not ask for approval for routine reversible steps.
Only ask when one of these hard stops is reached.

---

# PHASE 0 — PRE-CUTOVER SAFETY SNAPSHOT

Before editing runtime:

## 0.1 Confirm no active call

Use the real Asterisk CLI with sufficient permission.

Check active channels/calls.

If any active call exists:

HARD STOP.

## 0.2 Record current service/runtime state

Capture:

- `systemctl status/show/cat` for:
  - saas-voip
  - saas-streaming
  - saas-sttd
  - saas-db-backup service/timer if present
  - asterisk
- process command lines
- listening ports
- current environment values relevant to runtime paths
- `SAAS_BARGEIN`
- current Asterisk includes/contexts
- current `/opt/voip` symlinks
- current `/usr/local/lib/saas`
- current `/usr/local/bin/saas-*` wrappers

Do not print secrets.

## 0.3 Protect existing F WIP

Record:

- `git status --short`
- current branch
- current HEAD
- `git diff`
- untracked file list
- SHA256 manifest of critical F files

Create a cutover safety directory **inside F**, for example:

`F:\00-Ticenpi-SaaS\Ticenpi-Voip\_workspace\f-cutover-backup-<timestamp>`

Store:

- Git diff patch
- Git status
- SHA manifest
- copies of the systemd units being changed
- copies of current `/usr/local/lib/saas` runtime source
- copies of current wrappers
- copies of current Asterisk VOIP config
- active runtime DB backup later in the DB phase

Do NOT copy anything from D:.

---

# PHASE 1 — RECONCILE F: AGAINST CURRENT LIVE RUNTIME

The F: project is the target source of truth.

The current live runtime is the only valid comparison source for runtime-only hotfixes.

D: is not a source.

## 1.1 Compare critical files

Compare F: with current live runtime for at least:

- `engine/saas_streaming_server.py`
- `engine/saas_stt.py`
- `engine/saas_sttd.py`
- `engine/saas_turn.py`
- `engine/walker.py`
- `engine/saas_answerprobe.py`
- `scripts/saas-stt`
- `scripts/saas-answerprobe`
- `scripts/saas-asprobe`
- `scripts/saas-healthcheck`
- `scripts/saas-lineprobe`
- `backend/server.js`
- `backend/db.js`
- `backend/ami.js`
- `engine/stt_corrections.json`
- `engine/grade_rules.json` if it exists
- dialogue templates/config
- backup scripts
- service templates
- deploy scripts

For each diff classify:

- F_AUTHORIZED_NEWER_WORK
- RUNTIME_ONLY_HOTFIX
- SAME
- UNKNOWN

## 1.2 Reconcile into F

Rules:

- preserve all authorized newer F work
- port only necessary runtime-only hotfixes into F
- do not copy entire runtime files over F blindly
- do not import D content
- preserve recent barge-in/STT/walker work
- keep deterministic-first routing if that is the latest authorized F behavior
- preserve timing/observability hotfixes if runtime has them and F lacks them

Every meaningful runtime-only line that is retained must end up in F.

At the end:

`F = authoritative reconciled source`

## 1.3 Offline regression before cutover

Run the relevant existing project tests, including where present:

- syntax/import checks
- walker tests
- dialogue logic tests
- barge-in exact regression
- barge-in unit/tiered/probability tests
- endpoint/STT/voice-quality offline tests
- backend tests
- any F-drive path tests

Do not fix unrelated historical failures unless this task caused them.

If reconciliation breaks authorized VOIP tests:

fix before proceeding.

---

# PHASE 2 — PURGE D: FROM F PROJECT LOGIC

Search the entire F project for:

- `D:\`
- `/mnt/d/`
- old Google Drive VOIP paths
- old `SAAS_B/VOIP` paths

For operational files:

ZERO active D references are allowed.

Fix at minimum:

- scripts
- PowerShell
- BAT
- systemd templates
- deploy scripts
- backup scripts
- runtime config
- validation logic
- wrappers
- operational README/handoff instructions that could be followed as current procedure

Historical archive documents may retain a legacy path only if:
- clearly marked historical/legacy
- not used by any executable/config generator
- not likely to be interpreted as current deployment instructions

Prefer zero executable/config D references.

Rewrite `scripts/cutover-single-source.sh` so it contains **no D variable and no D rollback logic**.

---

# PHASE 3 — DESIGN F-ONLY RUNTIME MAPPING

Before changing services, establish the exact mapping.

Required realpaths after cutover:

- backend → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/backend`
- engine → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/engine`
- data → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/data`
- frontend → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/frontend`
- prod-audio → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/prod-audio`
- scripts → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/scripts`
- DB → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/db/saas_voip.db`
- DB backups → `/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/db/backup`

## /opt/voip compatibility

If old code/tools still expect `/opt/voip/*`, convert them into F-backed symlinks.

Preferred:

```
/opt/voip/backend    -> /mnt/f/.../backend
/opt/voip/engine     -> /mnt/f/.../engine
/opt/voip/data       -> /mnt/f/.../data
/opt/voip/frontend   -> /mnt/f/.../frontend
/opt/voip/prod-audio -> /mnt/f/.../prod-audio
/opt/voip/scripts    -> /mnt/f/.../scripts
/opt/voip/dbdata     -> /mnt/f/.../db
```

After cutover, `realpath` must resolve to F.

Do not keep independent project copies under `/opt/voip`.

---

# PHASE 4 — SQLITE SAFETY TEST ON F BEFORE MIGRATION

The active production DB must eventually live under F:.

Before moving production data, prove SQLite can safely operate on the F-mounted filesystem in this environment.

## 4.1 Use a COPY of a DB already on F

Create a temporary test DB under the F workspace, never on D.

Run:

- `PRAGMA integrity_check`
- open read
- `BEGIN IMMEDIATE;`
- perform a temporary reversible transaction on a test table/copy
- `ROLLBACK;`
- WAL mode test if production uses WAL
- close/reopen
- integrity_check again

The purpose is to prove:

- file locking works
- transactions work
- rollback works
- WAL behavior is acceptable

Do not test by mutating the production DB.

If this fails:

HARD STOP.

Do not silently keep production DB Linux-local while claiming F-only completion.

---

# PHASE 5 — PRODUCTION DB SNAPSHOT AND MIGRATION

This is the highest-risk step.

## 5.1 Re-measure active runtime DB immediately before stop

From:

`/opt/voip/dbdata/saas_voip.db`

Capture:

- path/realpath
- file size
- SHA256 where meaningful
- journal mode
- integrity_check
- table counts
- max/latest IDs/timestamps for:
  - customers
  - tasks
  - task_targets
  - call_records
  - record_details
  - settings
  - whisperings/nodes/branches

Store this as `PRE_CUTOVER_DB_MANIFEST`.

## 5.2 Confirm no active call again

Immediately before service stop:

- verify zero active calls/channels

If not zero:

HARD STOP.

## 5.3 Stop only VOIP writers

Stop the minimum services that can write/read the VOIP DB, expected:

- saas-voip
- saas-streaming
- saas-sttd

Do not stop Asterisk unless necessary.

Confirm writer processes are gone.

## 5.4 Checkpoint safely

If DB uses WAL:

- perform a safe checkpoint after writers stop
- ensure the resulting DB is consistent
- preserve WAL/SHM state in the safety snapshot if present

## 5.5 Preserve the current production DB in F backup

Create a timestamped backup under:

`F:\00-Ticenpi-SaaS\Ticenpi-Voip\_workspace\f-cutover-backup-<timestamp>\db\`

This backup must come from the current WSL runtime DB.

Not from D.

## 5.6 Preserve the old F snapshot separately

The old F `db/saas_voip.db` must not overwrite production data.

Move/copy it into an F-only pre-cutover archive location such as:

`F:\...\_workspace\f-cutover-backup-<timestamp>\old-f-db\`

Do not delete it until final verification is complete.

## 5.7 Install production DB as F canonical DB

Place the current production database at:

`/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/db/saas_voip.db`

Use a safe SQLite backup/copy procedure after writers are stopped.

Do not merge with old F DB.
Do not import D DB.

## 5.8 Verify exact parity

Against `PRE_CUTOVER_DB_MANIFEST`, verify:

- integrity_check = ok
- row counts match
- latest IDs/timestamps match
- settings match
- important schema objects match

If any mismatch:

HARD STOP and restore from the F cutover backup.

---

# PHASE 6 — SYSTEMD / WRAPPER CUTOVER TO F

Update the actual units so they directly execute F canonical code.

At minimum verify/change:

## saas-voip
- WorkingDirectory points to F/backend
- ExecStart uses F/backend source
- runtime DB path resolves to F/db
- `RequiresMountsFor=/mnt/f` or equivalent safe mount dependency

## saas-streaming
- ExecStart points to F/engine/saas_streaming_server.py
- no `/usr/local/lib/saas` source dependency
- `RequiresMountsFor=/mnt/f`
- preserve current environment
- preserve `SAAS_BARGEIN=0`

## saas-sttd
- ExecStart points to F/engine/saas_sttd.py
- imports F/engine modules
- no hardcoded `/usr/local/lib/saas`
- `RequiresMountsFor=/mnt/f`

## DB backup service/timer
- source script from F
- destination F/db/backup
- no D path
- no Linux-local canonical backup path

Run `systemctl daemon-reload` after edits.

---

# PHASE 7 — REMOVE STALE APP CODE OUTSIDE F

After F code has passed offline tests and units are prepared:

## /usr/local/lib/saas

Do not leave independent production source here.

After backing it up under F cutover backup:

- remove/rename stale project .py copies from active use
- no service/wrapper may import from this directory

If a minimal package marker is required, keep only what is technically necessary and document it.

## /usr/local/bin wrappers

Wrappers may remain but must be thin.

Each wrapper must:

- execute/import F source
- contain no independent business logic if avoidable
- contain no D path
- contain no stale `/usr/local/lib/saas` source path

Audit at least:

- saas-stt
- saas-answerprobe
- saas-turn
- saas-asprobe
- saas-healthcheck
- saas-lineprobe
- saas-voip-sync

For `saas-voip-sync`:
- either convert it to an F-safe compatibility helper
- or explicitly deprecate it if backend now executes F directly

It must never copy from D.

---

# PHASE 8 — /opt/voip COMPATIBILITY CUTOVER

Replace current D-backed/independent paths.

Ensure:

```
realpath /opt/voip/backend    → /mnt/f/.../backend
realpath /opt/voip/engine     → /mnt/f/.../engine
realpath /opt/voip/data       → /mnt/f/.../data
realpath /opt/voip/frontend   → /mnt/f/.../frontend
realpath /opt/voip/prod-audio → /mnt/f/.../prod-audio
realpath /opt/voip/scripts    → /mnt/f/.../scripts
realpath /opt/voip/dbdata     → /mnt/f/.../db
```

Do not create a second backend copy.

Do not create a second DB copy.

---

# PHASE 9 — ASTERISK CONFIG SOURCE-OF-TRUTH

Audit:

- `/etc/asterisk/extensions.conf`
- `/etc/asterisk/saas-dialogue.conf`
- `/etc/asterisk/saas-dialogue-as.conf`
- generation scripts/templates

Required outcome:

- project-owned dialogue source/template lives in F
- active Asterisk include derives from F
- no D path
- stale `saas-dialogue-as` is removed/disabled if confirmed unused
- do not keep competing active dialogue copies

Prefer a safe F-backed symlink for the generated dialogue file if Asterisk handles it reliably.
Otherwise, keep only the minimal generated system copy under `/etc/asterisk` and prove its source/generator is F.

Do not alter call-flow semantics in this migration task.

Reload Asterisk only after config validation succeeds.

---

# PHASE 10 — F-ONLY CUTOVER SCRIPT

Rewrite:

`scripts/cutover-single-source.sh`

It must:

- contain no D variable
- contain no D copy
- contain no D rollback
- check zero active calls
- create F-only safety backup
- verify SQLite integrity
- stop required services
- migrate current runtime DB to F if not already migrated
- set F-backed compatibility symlinks
- install/update F-backed systemd units/wrappers
- reload daemon
- validate Asterisk config
- restart required services
- run verification
- provide an F-only rollback path

Rollback must use:

- F cutover backup
- current WSL system config backup stored under F

Never D.

The script should be idempotent where practical:
running it after a successful cutover should not recreate D dependencies or duplicate data.

---

# PHASE 11 — START SERVICES

Start in dependency order:

1. saas-sttd
2. saas-streaming
3. saas-voip

Keep Asterisk running unless it required a validated reload.

Verify immediately:

- service active
- no traceback
- no import error
- no permission error
- no DB open/locking error
- port 9090 listening
- backend port 50900 listening
- health endpoint OK

If any service fails:

attempt F-only rollback from the saved snapshot.

Do not use D.

---

# PHASE 12 — POST-CUTOVER DATABASE VERIFICATION

Confirm the running backend now opens the F database.

Use:

- process open-file inspection if available
- configured DB path
- `realpath`
- SQLite queries

Required:

`ACTIVE_DB_REALPATH = /mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip/db/saas_voip.db`

Re-run:

- integrity_check
- row counts
- latest IDs/timestamps
- schema check

Compare to `PRE_CUTOVER_DB_MANIFEST`.

No data loss is allowed.

Use a reversible SQLite lock test if needed.
Do not insert fake production customer/call records.

---

# PHASE 13 — BACKUP VERIFICATION

Verify the DB backup service/timer:

- reads F DB
- writes to F/db/backup
- no D
- no stale `/opt/voip/dbdata` independent destination

If safe, run one manual backup cycle.

Verify:

- backup created under F
- integrity_check on backup = ok
- expected row counts are present

Do not delete prior backups.

---

# PHASE 14 — ZERO-D VERIFICATION

This is mandatory.

Search active runtime and operational project files for:

- `D:\`
- `/mnt/d/`
- old Google Drive VOIP paths

Check:

- systemd units
- environment files
- wrappers
- scripts
- backend
- engine
- deploy scripts
- backup jobs
- `/opt/voip` symlinks
- Asterisk config
- process command lines
- open file handles if possible

Required:

`ACTIVE_D_DEPENDENCY_COUNT = 0`

Documentation-only historical references must be listed separately.
They must not be executable or operational instructions.

---

# PHASE 15 — NO-STALE-COPY VERIFICATION

Required assertions:

- no service executes `/usr/local/lib/saas/*.py`
- no service executes an independent `/opt/voip/backend` copy
- `/opt/voip/*` project paths realpath to F
- DB realpath to F
- wrappers point to F
- systemd points to F
- backup writes to F
- Asterisk project config source derives from F

Generate a final runtime map:

| Component | Configured Path | Real Path | Source |
|---|---|---|---|

Every project-owned source/data row must be F_CANONICAL.

---

# PHASE 16 — VOIP BEHAVIOR REGRESSION

Before declaring success, run the safe non-live regression suite available in F.

At minimum, where present:

- Python syntax/import
- walker tests
- dialogue logic
- barge-in exact regression
- barge-in state/latency tests
- endpoint/STT offline regression
- backend tests
- healthcheck
- AudioSocket probe
- Asterisk dialplan validation

Confirm:

`SAAS_BARGEIN=0`

Do not enable live barge-in in this task.

No real phone call is required unless a regression cannot be proven offline.

---

# PHASE 17 — GIT / SOURCE STATE

After all runtime verification passes:

- re-run `git status`
- ensure no generated secret/runtime junk is accidentally staged
- preserve intended code/test/script changes
- do not stage DB, secrets, huge backups, WAL/SHM, logs, node_modules
- do not discard pre-existing authorized WIP

Create a clean commit for the F single-source reconciliation/cutover source changes if repository policy permits.

Suggested commit message:

`voip: make F drive canonical runtime source`

Report exact commit SHA.

Do not push unless the existing workflow explicitly authorizes it.

---

# SUCCESS CRITERIA

Do not declare success unless ALL are true:

## Source
- F is the authoritative project source
- no active D dependency
- no stale executable project copy in WSL

## Database
- current production DB preserved
- F DB matches the immediate pre-cutover runtime manifest
- active DB realpath is F
- integrity_check = ok
- backup path is F

## Runtime
- systemd executes F code
- wrappers execute F code
- /opt compatibility paths resolve to F
- Asterisk project config derives from F

## Services
- saas-voip active
- saas-streaming active
- saas-sttd active
- Asterisk healthy
- 50900 listening
- 9090 listening
- healthcheck passes

## Safety
- SAAS_BARGEIN remains 0
- no active call interrupted
- D contents not deleted
- no data loss
- rollback bundle exists on F

---

# FINAL OUTPUT

Return one report only:

# VOIP F-Drive Single Source Cutover Report

## RESULT
Choose exactly one:

- CUTOVER_SUCCESS
- CUTOVER_SUCCESS_WITH_NONBLOCKING_WARNINGS
- ROLLED_BACK
- HARD_STOP

## F CANONICAL ROOT
Show Windows and WSL paths.

## SOURCE RECONCILIATION
Table:
| File | F Before | Runtime | Final Decision | Final F Hash |

Explain only meaningful runtime hotfixes merged into F.

## DATABASE
Show:

- pre-cutover active runtime path
- pre-cutover manifest
- final F DB path
- post-cutover manifest
- parity result
- integrity result
- active DB realpath

Explicitly state:

`D_DB_USED = NO`

## RUNTIME MAP
Table:
| Component | Configured Path | Real Path | Status |

## D DEPENDENCY AUDIT
- active D refs before
- active D refs after
- `ACTIVE_D_DEPENDENCY_COUNT`

## SYSTEMD / WRAPPERS / ASTERISK
Concise changes and validation.

## BACKUP
- destination
- manual backup result if run
- integrity result

## TESTS
Commands + PASS/FAIL.

## SERVICES
- saas-voip
- saas-streaming
- saas-sttd
- asterisk
- ports
- health

## GIT
- branch
- HEAD before
- final commit if created
- remaining dirty files and why

## ROLLBACK
State the F-only rollback location and whether it was needed.

## REMAINING ISSUES
Only genuine remaining issues.

## FINAL ASSERTIONS

`F_IS_SINGLE_SOURCE_OF_TRUTH = YES/NO`

`ACTIVE_D_DEPENDENCY_COUNT = <number>`

`ACTIVE_DB_IS_F = YES/NO`

`RUNTIME_CODE_IS_F = YES/NO`

`SAAS_BARGEIN_REMAINS_0 = YES/NO`

`DATA_LOSS = NO/YES`

If any required assertion is NO, do not report CUTOVER_SUCCESS.

---

# FINAL RULE

Do not treat "services started" as sufficient.

The task is complete only when runtime paths, database, wrappers, systemd, backups and Asterisk integration all prove that F is canonical and D is unused.

Do not use D as source, migration input, rollback source or backup source at any point.
