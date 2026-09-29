# VOIP F-Drive Single Source Audit

## Role
VOIP RUNTIME / FILESYSTEM / DATABASE SOURCE-OF-TRUTH AUDITOR

## Objective
The user has already copied the old D: project data to:

`F:\00-Ticenpi-SaaS\Ticenpi-Voip`

This F: project must become the **only canonical source**.

Important rule:

- D: is legacy only.
- Do **not** copy, migrate, restore, sync, or symlink any old D: database/data back into F:.
- D: may be read **only for comparison/audit evidence**.
- Do not treat D: as a migration source.
- Do not modify WSL/systemd/Asterisk yet.
- Do not execute the existing cutover script yet.

The purpose of this task is to determine exactly what still points to D:, what runtime data currently exists outside F:, and whether the F: copy already contains the complete/latest canonical data required for cutover.

## Canonical Target

Everything that belongs to this VOIP project should ultimately resolve to:

`F:\00-Ticenpi-SaaS\Ticenpi-Voip`

WSL equivalent:

`/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip`

No active VOIP component should read/write:

- `D:\...`
- `/mnt/d/...`
- stale project copies under old Google Drive paths
- stale Python runtime copies under `/usr/local/lib/saas` unless they are only compatibility wrappers and do not contain independent source/state
- any stale `/opt/voip` copy that is not clearly mapped to F:

## Mode

READ-ONLY AUDIT FIRST.

Allowed:
- read files
- grep/search
- git status/log/rev-parse
- readlink/realpath
- ls/stat/sha256sum
- systemctl cat/show/status
- ps/lsof/fuser if read-only
- sqlite3 read-only queries
- compare hashes/sizes/mtime
- inspect symlinks
- inspect scripts/services/configs
- inspect backup jobs/timers

Not allowed:
- copy D → F
- move D → F
- delete D
- modify database
- modify symlinks
- restart services
- reload Asterisk
- deploy
- execute cutover-single-source.sh
- rewrite systemd units
- alter backup jobs
- commit

## Step 1 — Inventory F: Canonical Copy

Inspect:

`F:\00-Ticenpi-SaaS\Ticenpi-Voip`

Map at minimum:

- backend
- engine
- data
- frontend
- prod-audio
- scripts
- deploy
- database/dbdata if present
- logs if project-owned
- backup directories if project-owned
- configuration/state files
- stt_corrections.json
- grade_rules.json
- dialogue templates
- generated dialogue config source
- any .env or runtime config files (report existence only; do not print secrets)

For each major item report:
- exists YES/NO
- real path
- size/count
- last modified
- git tracked/untracked where applicable

## Step 2 — Find Every D: Reference

Search the entire F: project for:

- `D:\`
- `/mnt/d/`
- old Google Drive VOIP paths
- old SAAS_B/VOIP paths
- old backup paths

Search:
- source code
- shell scripts
- PowerShell
- batch files
- service unit templates
- deploy scripts
- config files
- README/docs only as a separate category

Classify each match:

A. ACTIVE_RUNTIME_REFERENCE
B. DEPLOY_REFERENCE
C. BACKUP_REFERENCE
D. TEST_ONLY
E. DOC_ONLY
F. DEAD/UNUSED

Do not count documentation-only references as runtime blockers, but list them separately.

## Step 3 — Inspect Actual WSL Runtime Resolution

Identify exactly where the running system currently reads from.

Check:

- `/opt/voip`
- `/usr/local/lib/saas`
- systemd units:
  - saas-voip
  - saas-streaming
  - saas-sttd
  - any backup timer/service
- wrapper commands:
  - saas-stt
  - saas-answerprobe
  - saas-voip-sync
- Asterisk config/dialplan source paths

For every component show:

`component → configured path → realpath → drive/source`

Example:

`saas-streaming → /mnt/f/.../engine/saas_streaming_server.py → F:`

or

`saas-sttd → /usr/local/lib/saas/saas_sttd.py → independent stale copy`

Mark each:

- F_CANONICAL
- D_LEGACY
- WSL_STALE_COPY
- UNKNOWN

## Step 4 — Database Truth Audit

This is critical.

Find every VOIP SQLite/database file on:

- F project
- D legacy project
- `/opt/voip`
- `/usr/local/lib/saas` if applicable
- backup locations

Do NOT copy anything.

For each DB report:

- exact path
- realpath
- file size
- mtime
- SHA256
- SQLite integrity_check result (read-only)
- key table row counts
- newest relevant record timestamp
- newest call-record timestamp/ID if schema supports it

The goal is to answer:

1. Which DB is the one currently used by the running service?
2. Does F already contain the complete/latest canonical DB?
3. Is the F DB newer/equal/older than current runtime DB?
4. Is any active service still using a D DB?
5. Is the existing cutover script planning to copy/restore D DB into F? If yes, flag it as WRONG FOR USER REQUIREMENT.

Important:
D DB is comparison-only.
Never propose "copy D DB to F" as the default remedy.

## Step 5 — Compare F vs Current Runtime, Not D as Source

For key executable/runtime files compare F against the actual currently-running runtime copy:

- saas_streaming_server.py
- saas_stt.py
- saas_sttd.py
- saas_turn.py
- walker.py
- saas-answerprobe
- saas-stt
- backend/server.js
- dialogue templates/config
- prod-audio indexes/manifests if any

Use hash/diff.

If current runtime contains a hotfix that F lacks:
- report the exact diff
- mark `RUNTIME_NEWER_THAN_F`
- do NOT copy it automatically
- identify what must be reconciled into F before cutover

The source of comparison is the current runtime, not D.

## Step 6 — Audit Existing Cutover Script

Read:

`scripts/cutover-single-source.sh`

Check whether it does any of the following:

- copies D files into F
- copies D database into F
- restores D backup into F
- symlinks F to D
- leaves any /mnt/d dependency
- rewrites backup jobs to use old data as source
- preserves stale `/usr/local/lib/saas` executable copies
- changes `/opt/voip` to anything other than F canonical paths

Flag each violating action.

Do not execute the script.

## Step 7 — Desired Final Architecture

Produce the exact intended end state, based on user requirement:

```
F:\00-Ticenpi-SaaS\Ticenpi-Voip
        ↓
/mnt/f/00-Ticenpi-SaaS/Ticenpi-Voip
        ↓
backend
engine
data
frontend
prod-audio
scripts
database/runtime-state (where applicable)
        ↓
WSL services / Asterisk wrappers
```

No runtime dependency on D.

If some state must remain Linux-native for SQLite correctness/performance, do not silently violate the user's requirement. Report it as a design exception requiring explicit approval.

## Final Report

Return:

# VOIP F-Drive Single Source Audit

## 1. RESULT
Choose:
- READY_FOR_F_CUTOVER
- F_COPY_INCOMPLETE
- RUNTIME_NEWER_THAN_F
- DATABASE_CONFLICT
- D_REFERENCES_REMAIN
- MULTIPLE_BLOCKERS

## 2. F CANONICAL INVENTORY
Concise table.

## 3. ACTIVE RUNTIME MAP
Table:
| Component | Configured Path | Real Path | Source | Status |

## 4. DATABASE TRUTH
Table:
| DB | Path | SHA256 | Size | Latest Record | Used By Runtime? |

Explicitly state:
`D_DB_REQUIRED_FOR_CUTOVER = YES/NO`

Expected user intent is NO unless hard evidence proves otherwise.

## 5. D REFERENCES
Table:
| File/Unit | Reference | Classification | Must Remove? |

## 6. F vs CURRENT RUNTIME DRIFT
List only meaningful differences/hotfixes.

## 7. CUTOVER SCRIPT PROBLEMS
List every action that would incorrectly use D as source.

## 8. REQUIRED FIXES BEFORE CUTOVER
Smallest ordered list.

## 9. SAFE CUTOVER PLAN
Plan only. Do not execute.

Must target F as source-of-truth and must not import D data.

## 10. STOP
Do not modify runtime.
Do not run cutover.
Do not copy D → F.
Wait for user approval after this audit.
