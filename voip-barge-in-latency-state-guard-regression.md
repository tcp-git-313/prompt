# VOIP Barge-in Latency + Single-Trigger State Guard Regression

## Role
VOIP REALTIME AUDIO ENGINE LATENCY / STATE-GUARD FIX OWNER

## Task Type
This is a **targeted code modification + offline regression task**.

The rolling-window accumulator has already passed exact regression. Do not reopen the full VAD design.

This task only addresses two remaining software-side concerns before live testing:

1. The current rolling implementation waits until the full 10-frame window is filled before it can trigger, creating a minimum trigger delay of about 320ms even when enough speech evidence is already available earlier.
2. Continuous caller speech may produce repeated accumulator triggers after reset; confirm that the PLAYING state can emit at most one real interrupt event per playback generation.

Do not deploy in this task.

## Confirmed Current Baseline

Current candidate semantics:

- WIN_MS = 32
- BARGEIN_WINDOW_MS = 320
- window_frames = 10
- BARGEIN_REQUIRED_SPEECH_MS = 192
- required_frames = 6
- current trigger condition requires:
  - len(recent_windows) >= 10
  - speech_ms_in_window >= 192
- current reset on trigger:
  - recent_windows = []
  - speech_ms_in_window = 0

Exact regression already proved:

- S1 silence: no trigger
- S2 8 consecutive speech: old triggers, current new does not because it waits for 10 frames
- S3 1101101101: current new triggers at frame 9
- S6 exact 6/10 threshold: triggers at frame 9
- S8 20×1: current new triggers at frames 9 and 19
- production SAAS_BARGEIN remains 0
- no deployment has occurred

## Goal

Produce the smallest safe change that:

1. Preserves the rolling-window/dropout tolerance.
2. Allows a trigger as soon as enough speech evidence exists, rather than forcing the buffer to fill to 10 frames first.
3. Prevents duplicate real interrupt events within the same PLAYING playback generation.
4. Preserves AI-only / short-noise safety.

Preferred conceptual behavior:

- Keep at most 10 recent frames.
- Allow evaluation before the window is full.
- Trigger once enough positive speech evidence exists inside the active rolling horizon.
- Keep old frames expiring normally.
- Ensure one PLAYING generation can produce at most one actionable barge-in interrupt.

Do not blindly implement this wording if source semantics indicate a safer simpler equivalent.

## Hard Scope

Allowed:
- accumulator trigger timing logic
- minimal playback-generation / interrupt state guard
- isolated tests and replay harnesses
- comments directly related to these semantics

Not allowed:
- change SAAS_BARGEIN production value
- deploy
- restart Asterisk
- change VAD threshold/model
- change endpoint silence timing
- change STT/vad_trim
- change walker/Groq
- change HT813/gain
- change TTS
- change GPU/CUDA
- broad refactor of saas_streaming_server.py

## Step 1 — Re-read Current Production Candidate

Inspect the actual current source and verify:

- BargeInAccumulator implementation
- current trigger condition
- current reset semantics
- reader_loop PLAYING branch
- where shared["bargein"] or equivalent is set
- how stream_out reacts to barge-in
- when state leaves PLAYING
- whether one playback has an existing generation/id/token/state that can be reused
- whether duplicate triggers can create duplicate side effects before state transition completes

Do not rely on prior prose if source differs.

If material drift exists, report it before editing.

## Step 2 — Optimize Earliest Safe Trigger

Current behavior waits for 10 frames even when 6 positive frames have already arrived.

Design the minimal change so that the accumulator can trigger once required speech evidence exists without requiring a full 10-frame history.

At minimum reason through:

- 6 consecutive positives
- 7–8 positives with dropout
- sparse positives spread too widely
- initial startup when fewer than 10 frames exist
- whether a minimum observation span is still needed for noise resistance

Candidate behavior to evaluate:

- maintain max rolling window of 10 frames
- evaluate after each frame
- trigger when positive-frame count reaches required_frames within the current retained horizon
- optionally require a small minimum observation count only if regression proves it necessary

Do not add arbitrary latency unless needed by evidence.

## Step 3 — Single-Trigger Guard

Prove whether the current runtime can emit more than one actionable barge-in event during the same PLAYING generation.

Important distinction:

- accumulator may mathematically become true multiple times
- actual interrupt side effect should occur at most once for one playback generation

Implement the smallest guard if needed.

Preferred properties:

- guard resets only when a new playback generation starts
- not just a global sticky flag across the whole call
- no race-prone duplicate interrupt
- no new thread
- no blocking I/O
- deterministic behavior under repeated VAD positives

If existing PLAYING state transition already guarantees one side effect, prove it and avoid unnecessary code.

## Step 4 — Deterministic Exact Regression

Use the actual production accumulator class and actual state-guard path where practical.

Run at least:

### L1 — clear speech
Sequence:
111111

Expected:
- new trigger as early as mathematically allowed
- report exact frame and latency

### L2 — clear speech + continued speech
Sequence:
20×1

Expected:
- accumulator may internally reset if designed that way
- actionable interrupt count for one PLAYING generation = exactly 1

### L3 — fragmented but sufficient
Sequence:
11011011

Expected:
- rolling logic can trigger without requiring 10 total frames if enough evidence exists
- report exact frame

### L4 — insufficient short noise
Sequence:
111000

Expected:
- no trigger

### L5 — sparse noise
Sequence:
1010001010

Expected:
- no trigger

### L6 — separated speech fragments
Sequence:
1100001100

Expected:
- no trigger

### L7 — exact threshold minus one
Construct exactly required_frames - 1 positives.

Expected:
- no trigger

### L8 — exact threshold
Construct exactly required_frames positives as early as possible.

Expected:
- trigger

### L9 — playback generation reset
Simulate:
- PLAYING generation A triggers once
- transition out
- new PLAYING generation B begins
- valid speech again

Expected:
- generation A actionable count = 1
- generation B actionable count = 1

### L10 — AI-only baseline
Use known AI-only probability/boolean pattern or a conservative synthetic equivalent.

Expected:
- no trigger

## Step 5 — Compare Latency

Report old vs current-candidate vs improved behavior.

At minimum:

| Pattern | Old consecutive | Current rolling | Improved rolling |
|---|---:|---:|---:|
| Clear speech | ~256ms | ~320ms | ? |
| Fragmented sufficient | often no trigger | ~320ms | ? |
| Short noise | no trigger | no trigger | no trigger |

Use frame-accurate values from tests.

## Step 6 — Regression Safety

Run:

- exact regression
- previous accumulator regression S1–S8
- any bargein unit/tiered/probability tests
- module import/syntax check
- directly relevant streaming tests

Do not fix unrelated failures.

## Acceptance Criteria

Complete only if all are true:

1. Enough valid speech can trigger earlier than 320ms when evidence is already sufficient.
2. Fragmented speech remains detectable.
3. Short noise and AI-only baselines remain non-triggering.
4. One PLAYING generation produces at most one actionable interrupt.
5. A new PLAYING generation can trigger independently.
6. Existing exact regression still passes or is intentionally updated with mathematically justified expectations.
7. SAAS_BARGEIN production remains 0.
8. No deployment occurred.

## Final Output

# VOIP Barge-in Latency + State Guard Report

## CURRENT
- source path
- current accumulator semantics
- current playback/barge-in side-effect path

## CHANGE
- exact latency optimization
- exact single-trigger guard behavior
- files changed

## LATENCY
Table:

| Pattern | Old | Previous Rolling | New |
|---|---:|---:|---:|

Include frame index and ms.

## SINGLE-TRIGGER PROOF
- same PLAYING generation repeated speech
- actionable interrupt count
- new generation reset behavior

## REGRESSION
Table for L1–L10.

## SAFETY
- AI-only false trigger = YES/NO
- short-noise false trigger = YES/NO
- production SAAS_BARGEIN remains 0 = YES/NO
- deployed = NO
- Asterisk restarted = NO

## TESTS
Commands + results.

## DECISION
Choose one:

A. Ready for dormant deployment / next controlled stage.
B. Minimal correction made; now ready.
C. Still unsafe; do not proceed.

## NEXT STEP
Only one smallest next step.

Do not start it automatically.

## Stop Condition
Stop after code change + offline regression report.
Do not deploy.
Do not start live phone testing.
Do not continue into endpoint/STT/walker work.
