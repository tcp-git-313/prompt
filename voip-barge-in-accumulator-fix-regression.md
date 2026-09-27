# VOIP Barge-in Accumulator Fix + Offline Regression

## Role
VOIP REALTIME AUDIO ENGINE FIX OWNER

## Task Type
This is a **code modification task**, not another broad investigation.

A full virtual diagnostic has already confirmed the software bug. Your job is to make the smallest safe change to the PLAYING-state barge-in speech accumulator and prove it with offline regression.

Do not expand scope into endpoint, STT, walker, HT813, GPU, or unrelated latency work.

## Confirmed Evidence
The completed virtual diagnostic established:

- Same Call4 caller audio:
  - LISTENING replay: `speech_ms=24864ms`
  - PLAYING replay: `speech_ms=32ms`
  - `bargein_count=0`
- Current PLAYING accumulator effectively does:
  `speech_ms = speech_ms + WIN_MS if sp else 0`
- `WIN_MS` is approximately 32ms.
- `BARGEIN_SPEECH_MS=240ms` therefore requires roughly 8 consecutive speech windows.
- One non-speech frame resets the full counter.
- Real caller interruptions are fragmented enough that VAD probability can exceed 0.5 while the 240ms consecutive gate never completes.
- AI-only playback baseline did not produce false `speech=true` events in prior live shadow data.

The user is currently unavailable for more live phone tests.

## Goal
Replace the fragile consecutive-only PLAYING barge-in accumulator with the smallest robust algorithm that tolerates brief VAD dropouts while preserving resistance to false triggers.

Preferred direction:
- short rolling window or hysteresis
- reuse existing VAD output
- no model change
- no VAD threshold change
- no production gain change

Do **not** simply change `BARGEIN_SPEECH_MS` from 240ms to 160ms unless evidence proves that is safer than fixing the accumulator design.

## Hard Scope
You may modify:
- the PLAYING-state barge-in accumulation logic
- isolated tests / replay harnesses needed to prove it
- comments directly related to this change

Do not modify:
- `SAAS_BARGEIN` production value
- VAD threshold/model
- LISTENING endpoint timing
- STT / `vad_trim`
- SenseVoice / Whisper
- walker / intent / Groq
- HT813 or Asterisk gain
- TTS
- GPU/CUDA
- unrelated runtime behavior

Do not deploy to production in this task.

## Step 1 — Re-read the Exact Current Logic
Open the current source actually used by the repo and identify:

- current PLAYING VAD accumulator
- `BARGEIN_SPEECH_MS`
- frame/window duration
- state transitions
- reset conditions
- how the barge-in event would be raised when enabled
- whether LISTENING uses a different counter/endpoint mechanism

Confirm the previous diagnosis from source before editing.

If the current source no longer matches the diagnostic evidence, STOP and report drift.

## Step 2 — Define the Smallest Robust Rule
Design one minimal replacement for the PLAYING accumulator.

Preferred example pattern:

- maintain a short rolling observation window, e.g. 320–400ms
- accumulate speech-positive duration or ratio within that window
- allow one or more short negative VAD gaps
- trigger only when enough speech evidence exists inside the window

Example concept only:

`recent_window_ms = 320`
`required_speech_ms = 200`

Do not copy these values blindly. Derive candidate values from:
- current 32ms frame size
- existing 240ms requirement
- known AI-only baseline
- known Call4 caller replay

Keep semantics easy to explain and test.

Alternative hysteresis is acceptable if it is clearly simpler and safer.

## Step 3 — Add/Reuse Offline Regression Tests
Use the existing virtual replay harness if available.

Required fixtures:

### A. AI-only / silence playback
Expected:
- no barge-in trigger
- no meaningful accumulation

### B. Clear caller speech
Expected:
- trigger within a reasonable short interval

### C. Fragmented caller speech
Create or reuse speech where VAD has brief negative frames between positive frames.

Expected:
- new algorithm still accumulates enough evidence
- old consecutive-only algorithm fails or takes materially longer

### D. Short noise burst
Expected:
- no trigger

### E. Two short speech fragments separated by a gap too long to count as one interruption
Expected:
- no false accumulation across an excessive gap

### F. Same Call4 replay used in the diagnostic
Run the identical sample in PLAYING mode.

Expected:
- the new algorithm produces a stable barge-in candidate
- record trigger latency and evidence window

## Step 4 — Compare Old vs New
For every fixture report:

| Fixture | Old trigger | Old latency | New trigger | New latency | False positive? |

Also capture:
- max VAD probability
- speech-positive frame count
- effective speech ms within the decision window
- reset / expiration behavior

The change is acceptable only if:
- Call4-style fragmented caller speech becomes detectable
- AI-only playback remains non-triggering
- short noise does not trigger
- behavior is deterministic across repeated replay

## Step 5 — Implementation Constraints
Keep the change minimal.

Preferred:
- one small helper/state structure
- no new heavy dependency
- no new thread/process
- no blocking I/O
- no change to audio frame flow
- no change to VAD inference
- no change to current production env defaults

If `SAAS_BARGEIN=0`, production behavior must remain disabled exactly as before.

The code path may be testable while disabled, but do not silently enable it.

## Step 6 — Tests
Run:
- targeted unit/replay tests for the accumulator
- existing VAD/streaming tests relevant to the changed module
- syntax/import checks

If there is an established test suite for `saas_streaming_server.py`, run the narrowest relevant set plus any directly impacted regression suite.

Do not fix unrelated pre-existing failures.

## Step 7 — Output
Return:

# VOIP Barge-in Accumulator Fix Report

## CURRENT
- source file
- current branch/commit
- current accumulator behavior
- production `SAAS_BARGEIN` value if visible

## CHANGE
- exact algorithm chosen
- why it is safer than consecutive-only 240ms
- files changed

## OLD VS NEW
Regression table for A–F.

## CALL4 REPLAY
- old result
- new result
- trigger latency
- false-positive observation

## SAFETY
- AI-only false trigger = YES/NO
- short-noise false trigger = YES/NO
- production barge-in enabled = NO
- deployed = NO

## TESTS
- commands
- results
- any pre-existing unrelated failures

## NEXT STEP
Only one next step:
either
- ready for one controlled live shadow/enable test
or
- additional offline correction still required

## Stop Condition
Stop after code change + offline proof.

Do not deploy.
Do not enable production barge-in.
Do not continue into endpoint/STT/walker fixes.
