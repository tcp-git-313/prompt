# VOIP Barge-in Accumulator Exact Regression

## Role
VOIP REALTIME AUDIO REGRESSION OWNER

## Task Type
This is a **targeted verification + minimal-correction task**.

Do not repeat the full VOIP architecture audit.
Do not run another live phone test.
Do not deploy.

The previous task already changed the PLAYING-state barge-in accumulator from a consecutive-only gate to a rolling-window design. Before any deployment, verify the exact arithmetic and resolve two contradictions in the previous report.

## Current Candidate Design
Candidate defaults reported:

- frame/window unit: ~32ms
- `SAAS_BARGEIN_WINDOW_MS=320`
- `SAAS_BARGEIN_REQUIRED_MS=192`
- therefore expected rolling window ≈ 10 frames
- expected required speech evidence ≈ 6 positive frames

Previous report also claimed:

1. "Fragmented interruption (15%, sporadic) → NEW trigger"
2. "Clear caller speech (60%) → OLD trigger @ 7 windows"

Those statements appear inconsistent with the reported algorithm:

- 15% positive speech cannot satisfy 6/10 positive frames if "15%" really means speech ratio.
- old 240ms consecutive logic with 32ms frames needs 8 positive frames (7×32=224ms, 8×32=256ms), not 7.

The purpose of this task is to prove exactly what the implementation does and correct any test/report/code mismatch.

## Hard Scope

Allowed:
- read current accumulator implementation
- read existing regression harnesses
- add or refine isolated deterministic accumulator tests
- correct test labels/descriptions
- make the smallest accumulator-code correction only if exact regression proves the current implementation is wrong
- update comments directly related to accumulator semantics

Not allowed:
- enable production barge-in
- change production env values
- deploy
- restart Asterisk
- change VAD threshold/model
- change endpoint timing
- change STT/vad_trim
- change walker/Groq
- change HT813/gain
- change GPU/CUDA
- start live phone testing

## Step 1 — Verify Current Source Exactly

Read the current candidate source and report:

- exact accumulator class/function
- exact `WIN_MS`
- exact `BARGEIN_WINDOW_MS`
- exact `BARGEIN_REQUIRED_SPEECH_MS`
- integer/frame conversion formulas
- effective `window_frames`
- effective `required_frames`
- trigger condition
- reset/expiration condition
- whether trigger is evaluated before or after old-frame eviction

Do not rely on the previous prose report.

If the current source differs materially from the reported candidate, state the drift first and continue from actual source.

## Step 2 — Build Exact Boolean-Sequence Regression

Create or reuse an isolated test harness that bypasses audio/VAD inference and feeds the accumulator deterministic speech booleans directly.

For every frame print or capture:

- frame index
- input speech bool
- current rolling window
- positive-frame count
- speech_ms_in_window
- trigger YES/NO
- any eviction/reset

At minimum test these sequences:

### S1 — all silence
`0000000000`

Expected:
- OLD no trigger
- NEW no trigger

### S2 — old consecutive success boundary
`11111111`

Verify exact OLD trigger frame.
With 32ms frames and 240ms requirement, expected boundary should be mathematically explained.

### S3 — fragmented speech that should pass rolling window
`1101101101`

This contains 7 positive frames in 10.
Expected:
- OLD should not pass the consecutive-only gate
- NEW should pass if effective requirement is 6/10

### S4 — insufficient evidence
`1110000000`

Expected NEW no trigger.

### S5 — separated fragments
`1100001100`

Expected NEW no trigger.

### S6 — exact new threshold boundary
Construct:
- exactly required_frames positives inside one window
- one fewer than required_frames

Prove:
- exact threshold triggers
- threshold minus one does not

### S7 — long stream with eviction
Construct >20 frames where old positive frames leave the rolling window.
Prove the positive count/speech_ms decrements correctly and cannot accumulate forever.

### S8 — trigger-reset behavior
After a valid trigger, continue feeding speech and silence.
Prove reset semantics and whether duplicate triggers can occur immediately.

## Step 3 — Resolve the "15%" Claim

Find the exact fixture/test that produced the previous statement:

`Fragmented interruption (15%, sporadic) → NEW trigger`

Determine what "15%" actually represented:

- speech-positive ratio?
- dropout ratio?
- amplitude?
- injected probability?
- fixture label only?
- report-writing error?

Calculate the true ratio directly from the fixture.

Output:

`PREVIOUS_15_PERCENT_LABEL = CORRECT / INCORRECT`

and the actual value/meaning.

If the label is wrong, correct the test/report label.
Do not alter algorithm behavior merely to preserve the old wording.

## Step 4 — Resolve the "OLD @ 7 windows" Claim

Find the exact old-logic test that reported:

`OLD trigger @ 7 windows`

Reproduce the old logic exactly.

Show:
- frame duration
- cumulative speech_ms after each positive frame
- exact frame where the old condition becomes true

Output:

`OLD_7_WINDOWS_CLAIM = CORRECT / INCORRECT`

If incorrect, explain whether it was:
- zero-based index confusion
- display/reporting error
- different effective WIN_MS
- different condition
- test harness mismatch

Correct the test/report, not production semantics, unless the implementation itself is wrong.

## Step 5 — Cross-check Production Candidate vs Test Harness

Verify that isolated tests and production candidate use the same:

- frame duration
- window length
- required evidence
- trigger comparison (`>=` vs `>`)
- eviction order
- reset behavior

No duplicated "test-only approximation" is allowed to silently differ.

If possible, import/use the real accumulator class in the test rather than reimplementing it.

## Step 6 — Re-run Relevant Regression

After resolving contradictions, run:

- exact boolean-sequence regression
- existing barge-in unit tests
- existing tiered/probability tests
- module import/syntax check
- any directly related streaming tests already present

Do not run unrelated giant suites unless existing project convention requires them.

Do not fix unrelated pre-existing failures.

## Acceptance Criteria

The task is complete only when all are true:

1. Exact effective rolling semantics are proven.
2. The "15%" contradiction is resolved.
3. The "7 windows" contradiction is resolved.
4. Test harness semantics match the real accumulator.
5. Deterministic sequences S1–S8 behave as mathematically expected.
6. AI-only/noise safety tests still pass.
7. Production `SAAS_BARGEIN` remains disabled.
8. No deployment occurred.

## Final Output

# VOIP Barge-in Exact Regression Report

## CURRENT IMPLEMENTATION
- source path
- effective WIN_MS
- window_frames
- required_frames
- trigger/reset semantics

## CONTRADICTION 1 — "15%"
- original fixture
- actual ratio/meaning
- verdict
- correction made

## CONTRADICTION 2 — "OLD @ 7"
- exact cumulative frame table
- real trigger frame
- verdict
- correction made

## EXACT REGRESSION
Table:

| Sequence | Old Trigger | Old Frame | New Trigger | New Frame | Expected | Result |
|---|---:|---:|---:|---:|---|---|

Include S1–S8.

## HARNESS PARITY
Confirm whether tests use the actual production accumulator class.

## FILES CHANGED
List exact files and concise diff summary.

## TESTS
Commands + results.

## SAFETY
- Production SAAS_BARGEIN remains 0 = YES/NO
- Deployed = NO
- Asterisk restarted = NO
- Production behavior changed = NO

## DECISION
Choose one:

A. Candidate accumulator is internally correct and ready for the next controlled stage.
B. Candidate required a minimal correction; corrected version now passes exact regression.
C. Candidate still has unresolved logic risk; do not proceed.

## NEXT STEP
Only one smallest next step.

Do not start it automatically.

## Stop Condition
Stop after exact verification/correction and report.
Do not deploy.
Do not start live testing.
