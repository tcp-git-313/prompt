# VOIP VAD Virtual Replay Diagnostic

## Role
VOIP AUDIO PIPELINE / VAD STATE-MACHINE DIAGNOSTIC OWNER

## Context
Live tests already produced inconclusive behavior:

- AI-only PLAYING segments did not falsely trigger VAD.
- Normal caller speech in LISTENING could reach VAD max ~0.9 and produce STT text.
- Caller interruption during PLAYING reached meaningful audio levels and VAD probability above 0.5, but `speech=true` / `speech_ms` did not accumulate as expected.
- Small caller speech could also produce empty STT.
- User also observed:
  - AI sometimes reacts before the user finishes speaking.
  - AI sometimes answers a different branch than expected.
  - AI sometimes says the user did not speak even when the user did.

The user is not available for more live phone testing now.

## Goal
Build a safe virtual/offline diagnostic path that can reproduce and isolate the software-side causes without requiring a real phone call.

The priority is to answer, with evidence:

1. Why can `vad_prob > 0.5` occur while `speech=true` and `speech_ms` remain zero during PLAYING?
2. Does PLAYING use different accumulation/reset/gating logic than LISTENING?
3. Can the current endpoint logic cut an utterance before the speaker has actually finished?
4. Can existing real call audio be replayed through the exact VAD/STT path to reproduce the problem?
5. When STT text is correct but the reply is wrong, which layer selected the wrong branch: normalization, intent, keyword, LLM, or fallback?

## Safety / Scope
Start with investigation and isolated tests.

Do not:
- enable production barge-in
- change `SAAS_BARGEIN`
- change production VAD threshold
- change HT813 settings
- change Asterisk gain
- change production STT/TTS/walker behavior
- deploy behavioral fixes
- restart Asterisk
- touch GPU/CUDA configuration

You may:
- read runtime/source
- create isolated test scripts/harnesses outside the production execution path
- reuse copies of existing WAV/PCM/log samples
- invoke the same VAD/STT/walker functions in isolation
- create synthetic 8 kHz mono PCM test fixtures
- add tests under an isolated test location
- run deterministic replay experiments

If any test would alter production runtime behavior, stop and report instead.

## Phase 1 — Trace the Exact State Logic
Read the current source of:

- `StreamVad`
- `reader_loop()`
- `listen_utterance()`
- PLAYING/LISTENING state transitions
- `speech`
- `speech_ms`
- any consecutive-frame counters
- any reset conditions
- barge-in gate conditions
- endpoint / silence handling

Produce the exact logic chain:

`Audio frame -> VAD probability -> bool -> speech counter -> state gate -> speech_ms -> endpoint/barge decision`

Specifically identify every place where:
- `speech` is reset
- `speech_ms` is reset
- PLAYING behaves differently from LISTENING
- VAD probability is observed but intentionally ignored
- state changes can happen between frames

Do not infer from comments; cite actual code paths.

## Phase 2 — Build an Offline VAD Replay Harness
Create an isolated replay harness that calls the same production VAD implementation without modifying production behavior.

Input formats to support:
- 8 kHz
- mono
- signed 16-bit PCM/WAV

Replay in frame sizes equivalent to production.

For each frame, capture:
- virtual timestamp
- state: PLAYING or LISTENING
- RMS
- peak
- VAD probability
- raw VAD bool
- speech bool
- speech_ms / consecutive speech duration
- any reset event and reason

The harness must allow the same audio to be replayed twice:

A. state = LISTENING
B. state = PLAYING

This is the primary comparison.

## Phase 3 — Use Existing Real Audio First
Find usable historical audio from the recent live tests if present.

Prefer, in order:
1. AudioSocket/raw diagnostic capture if one exists
2. per-turn caller-side recording if one exists
3. MixMonitor WAV only as a fallback/reference

Be explicit if MixMonitor is used, because it is not the same stream as AudioSocket inbound.

Take at least one segment containing clear caller speech and replay the identical audio under:

- LISTENING
- PLAYING

Compare whether the VAD probability is the same but `speech/speech_ms` behavior differs.

If no valid caller-only sample exists, create a synthetic test fixture from a known speech sample converted to production format.

## Phase 4 — Synthetic Edge Cases
Run isolated virtual tests covering:

### Case A — silence
Expect no speech.

### Case B — clear continuous speech
At least 1–2 seconds.

### Case C — short phrase
Approximately 300–500 ms.

### Case D — speech with short natural pause
Example structure:
speech 500 ms
silence 300–500 ms
speech 500 ms

Use this to test whether endpointing cuts too early.

### Case E — speech with 700–900 ms pause
Use this to expose the practical effect of current endpoint silence settings.

### Case F — PLAYING-state caller interruption
Use the same speech fixture while state is PLAYING.

For every case, report exact frame-by-frame decision transitions.

## Phase 5 — Endpoint / "User Not Finished" Test
Trace the actual endpoint logic and reproduce it virtually.

Determine:
- how much silence ends an utterance
- whether the timer starts on first silent frame
- whether transient VAD drops reset or end speech
- whether denoise/resampling affects endpoint timing
- whether short pauses inside normal Mandarin speech can terminate the utterance

Create at least three virtual utterances with controlled internal pauses and report the exact point where `listen_utterance()` returns.

Do not tune the endpoint yet.

## Phase 6 — STT Replay
Using the same test audio, invoke the actual production STT path in isolation.

For each fixture capture:
- input duration
- RMS/peak
- VAD-trimmed duration if applicable
- STT text
- empty/non-empty
- timing for denoise / VAD trim / inference if available

Compare:
- louder normal speech
- attenuated speech
- short phrase
- speech with pause

Determine whether empty STT is caused before inference (trim/gate) or by the recognizer itself.

Do not switch models yet.

## Phase 7 — Dialogue / Wrong-Answer Replay
For recent known STT outputs and several hand-written examples, invoke the current dialogue-selection path in isolation.

Trace:

`raw STT -> normalization -> corrections -> detect_intents -> keyword match -> LLM branch (if called) -> chosen node`

For each input report:
- normalized text
- correction applied or not
- intent result
- keyword result
- whether Groq/LLM was called
- LLM output if available
- final selected branch/node
- why that branch won

Include at least:
- a clearly matching keyword case
- an ambiguous case
- a short phrase
- an empty STT case
- one recent real STT text from Call4 if available

Do not modify walker logic.

## Phase 8 — Classification
At the end classify the software-side problem using evidence only.

Possible findings:

A. PLAYING-state speech accumulation/reset bug
B. VAD threshold/model issue
C. endpoint / silence timing cuts speech too early
D. STT preprocessing/trimming removes weak speech
E. STT recognizer itself fails on weak speech
F. walker/intent/LLM branch-selection issue
G. cannot reproduce in software; likely hardware/FXO/AudioSocket-path specific
H. multiple independent issues

More than one may apply.

## Required Output

# VOIP Virtual Replay Diagnostic Report

## 1. Runtime / Source Identity
- exact source file(s)
- commit/SHA if available
- no production behavior changed = YES/NO

## 2. VAD State Logic
- exact speech/speech_ms/reset rules
- PLAYING vs LISTENING differences

## 3. Same-Audio Replay Result
Table:

| Fixture | State | VAD max | speech true count | speech_ms max | reset reason |
|---|---:|---:|---:|---:|---|

## 4. Endpoint Results
For each controlled-pause fixture:
- pause duration
- returned early YES/NO
- return timestamp
- reason

## 5. STT Results
Table:
| Fixture | Level | Trimmed duration | STT | Empty? | Timing |

## 6. Dialogue Replay
Table:
| Input | Intent | Keyword | LLM called | Final node | Why |

## 7. Findings
Use classifications A–H with evidence.

## 8. What Is Proven vs Still Needs Live Phone Test
Separate:
- software facts proven virtually
- hardware/PSTN/HT813 facts that still require live testing

## 9. Next Step
Recommend only the smallest next engineering action based on evidence.

Do not implement the fix in this task.

## Stop Condition
Stop after the diagnostic report.

Do not enable barge-in.
Do not tune production thresholds.
Do not deploy a fix.
