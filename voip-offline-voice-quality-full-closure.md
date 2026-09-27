# VOIP Offline Voice Quality Full Closure

## Role
VOIP REALTIME VOICE QUALITY / TURN-TAKING / STT / DIALOGUE OPTIMIZATION OWNER

## Task Type
This is an **autonomous offline closure task**.

Do not stop after every small finding.
Do not ask the user to approve each intermediate step.
Continue investigating, making the smallest justified code changes, and rerunning regression until the software-side voice pipeline is as strong as it can be **without requiring a real human phone call or production deployment**.

Only stop when:
1. all offline gates pass, or
2. the remaining uncertainty genuinely requires live HT813/PSTN/human testing, or
3. a decision would materially change product behavior and cannot be resolved from evidence.

## Primary Goal
Bring the current VOIP system as close as practical to a fast, interruption-friendly, natural realtime voice experience using the existing architecture, while preserving deterministic behavior and avoiding unnecessary complexity.

Optimize and close, in this order:

1. Barge-in accumulator / trigger latency / single-trigger guard
2. Endpoint / turn-end detection
3. STT preprocessing and weak-speech preservation
4. STT recognition quality and latency
5. Dialogue routing / deterministic intent selection
6. End-to-end offline voice pipeline
7. Optional RTX 3060 Ti acceleration evaluation
8. Final full regression

Do **not** require the user to perform another live call until all offline work is exhausted.

---

# CURRENT VERIFIED BASELINE

Previous diagnostics established:

## Audio / runtime
- Production path already uses Asterisk + AudioSocket.
- Streaming service exists.
- Silero VAD is used.
- Production barge-in remains disabled with `SAAS_BARGEIN=0`.
- Existing work has not been deployed yet.
- HT813/PSTN-specific attenuation remains only partially proven and must not be treated as the sole root cause.

## Confirmed software issues

### A. PLAYING barge-in accumulation
The old consecutive-only logic reset on every non-speech frame.

A rolling-window candidate was created and exact regression confirmed:

- `WIN_MS = 32`
- `BARGEIN_WINDOW_MS = 320`
- `window_frames = 10`
- `BARGEIN_REQUIRED_SPEECH_MS = 192`
- `required_frames = 6`

Exact regression corrected earlier reporting errors:
- old 240ms threshold requires 8 × 32ms windows, not 7
- a 15% positive-speech fixture cannot satisfy a 6/10 rolling threshold

Remaining concerns:
- current rolling candidate may wait until full 10-frame window before triggering
- one playback generation must emit at most one actionable interrupt

### B. Endpoint
Offline replay confirmed:
- 400ms internal pause is safe
- ~800ms pause terminates the utterance
- current endpoint silence around 700ms can cut a naturally paused Mandarin utterance too early

### C. STT preprocessing
Offline replay confirmed:
- weak caller speech can be removed by `vad_trim`
- threshold/min-speech preprocessing can produce empty STT before SenseVoice gets useful speech

### D. Dialogue routing
Offline replay confirmed:
- walker/intent/LLM routing can choose the wrong branch
- an ASK_WHY-like input was misclassified as ASK_COMPANY
- fixed-script/deterministic intents should not be unnecessarily delegated to LLM

### E. Hardware path
There is evidence of substantial level difference between some call phases, but:
- HT813/PSTN behavior is not fully proven offline
- do not change hardware gain yet
- do not assume half-duplex is the universal root cause

---

# AUTONOMOUS EXECUTION RULE

Work in repeated closure loops:

```
inspect current code
→ reproduce issue offline
→ make smallest justified change
→ run targeted regression
→ run impacted regression
→ compare before/after
→ continue to next software issue
```

Do not return to the user after each loop.

Keep a running internal change log and final evidence table.

If a change causes regression:
- revert or correct it
- continue testing
- do not leave the branch in a knowingly worse state

Do not optimize by guesswork.
Every parameter change must be justified by replay/regression evidence.

---

# HARD SAFETY BOUNDARIES

Do not:
- enable production `SAAS_BARGEIN=1`
- deploy to production
- restart Asterisk
- modify HT813 settings
- modify production Asterisk gain
- modify production secrets
- modify unrelated services
- silently change dialogue content
- replace the architecture with a new framework
- rewrite the whole voice engine
- introduce large dependencies without strong evidence
- convert this into a generic "GPT Live clone" rewrite

You may:
- modify VOIP source code
- modify/add offline tests and replay harnesses
- add small helpers
- add configuration parameters where justified
- create isolated local benchmark environments
- create an isolated GPU test venv if required
- run local/offline benchmarks
- use existing recorded samples
- generate synthetic PCM/WAV fixtures
- compare candidate algorithms
- clean up test-only code if no longer needed

Production behavior must remain unchanged unless/until a later live/deploy task explicitly authorizes it.

---

# PHASE 1 — FINISH BARGE-IN SOFTWARE CLOSURE

Start from the current rolling accumulator candidate.

## 1A. Earliest-safe trigger
Verify whether the current condition waits for:
`len(recent_windows) >= window_frames`

If yes, determine whether this needlessly delays clear speech.

Optimize so valid speech can trigger as soon as enough evidence exists, while still respecting the maximum rolling horizon.

Target:
- clear speech should not have to wait ~320ms if sufficient evidence exists earlier
- fragmented but valid caller speech must remain detectable
- sparse noise must not trigger

Prefer simple, explainable semantics.

Candidate concept:
- retain at most 10 recent frames
- evaluate every frame
- trigger when at least required speech evidence exists inside the active horizon
- add only the minimum observation guard proven necessary by tests

Do not reduce safety by arbitrary threshold lowering.

## 1B. Single-trigger guard
Prove whether continuous caller speech can create more than one real interrupt side effect within one PLAYING generation.

Requirement:
- one playback generation → at most one actionable interrupt
- new playback generation → can trigger again

Reuse existing state/generation identity if possible.
Do not add race-prone global sticky flags.

## 1C. Required regression
At minimum test:

- silence
- AI-only baseline
- short noise burst
- 6 consecutive positive frames
- 8 consecutive positive frames
- fragmented sufficient speech
- sparse insufficient speech
- separated fragments with long gap
- continuous 20-frame speech
- two different playback generations

Report old vs candidate vs final trigger latency.

Preferred target:
- clear speech actionable candidate around ~192–250ms if safe
- no AI-only/noise trigger

Do not force this numeric target if evidence shows a safer alternative.

---

# PHASE 2 — ENDPOINT / TURN-END OPTIMIZATION

Goal:
Stop cutting the user off too early without making every response feel slow.

Current problem:
~700ms silence can terminate an utterance, and ~800ms natural pause reproduced premature endpointing.

Build deterministic synthetic and real-audio replay fixtures for Mandarin-style pauses.

Test candidate endpoint settings/logic across at least:

- 400ms pause
- 600ms pause
- 700ms pause
- 800ms pause
- 900ms pause
- 1000ms pause
- 1200ms pause
- true end-of-utterance silence

Do not just pick a larger constant blindly.

Evaluate:
- false early termination
- added response latency after real utterance end
- interaction with VAD state drops
- short filler/pause patterns

Prefer:
1. smallest safe static improvement, or
2. simple hysteresis/adaptive endpoint only if clearly superior and still easy to reason about

Avoid overengineering.

Acceptance goal:
- natural short pauses do not prematurely terminate speech
- real end-of-turn does not add excessive delay
- regression is deterministic

---

# PHASE 3 — STT PREPROCESSING / WEAK SPEECH

Goal:
Prevent valid weak speech from being removed before recognition.

Trace exact path:

`PCM → denoise/norm → VAD trim → SenseVoice → post-processing`

Use:
- recent Call3-like weak speech sample if available
- normal caller sample
- short phrase sample
- low-level synthetic speech
- silence/noise controls

Measure:
- input duration
- input RMS/peak
- trimmed duration
- retained speech ratio
- STT output
- empty/non-empty
- processing latency

Investigate:
- trim threshold
- min speech duration
- padding
- whether double-VAD is unnecessarily aggressive
- whether endpoint already supplied a speech segment such that STT-side trimming can be relaxed or bypassed safely

Prefer preserving weak speech without admitting long noise segments.

Do not switch STT models until preprocessing has been proven not to be the primary cause.

---

# PHASE 4 — STT QUALITY / LATENCY

After preprocessing is fixed, benchmark current SenseVoice behavior.

Measure:
- accuracy on available real/synthetic fixtures
- empty result rate
- latency
- short-phrase reliability
- weak-speech reliability

If current STT becomes adequate after preprocessing:
- keep it

If still materially weak:
- test alternative local STT options in isolation
- do not replace production STT merely because another model is newer

Candidate alternatives may include:
- faster-whisper
- whisper.cpp
- another local model already compatible with the project

Only compare on actual fixtures and latency.

---

# PHASE 5 — DIALOGUE / WRONG-ANSWER CLOSURE

Goal:
A correct STT transcript should map to the correct scripted response as deterministically as possible.

Trace:

`raw STT → normalization → correction → explicit intent → keyword → LLM (if needed) → branch → node`

Build a regression corpus containing:

- ASK_WHY
- ASK_COMPANY
- ASK_LOCATION
- positive/negative responses
- known keywords
- short ambiguous statements
- empty STT
- recent real Call4 STT examples if available
- typo/correction examples

Policy:
- explicit deterministic intent/keyword rules should win when they are unambiguous
- LLM should handle genuinely ambiguous text
- LLM must not override an already reliable deterministic match unless current product rules explicitly require it

Specifically prevent known ASK_WHY → ASK_COMPANY misrouting.

Do not rewrite the whole walker.
Make the smallest ordering/guard changes needed.

Record for every corpus item:
- normalized text
- correction
- detected intent
- keyword match
- LLM called YES/NO
- selected node
- expected node
- result

---

# PHASE 6 — FULL OFFLINE E2E REPLAY

Build/extend an offline pipeline replay that exercises as much real production code as possible:

`audio fixture
→ VAD / endpoint
→ STT preprocessing
→ STT
→ normalization
→ intent / keyword / LLM gate
→ walker
→ response node`

Use real production functions/classes rather than duplicated test-only approximations whenever possible.

Required fixture classes:
- silence
- AI-only/noise
- clear caller speech
- weak caller speech
- short phrase
- fragmented caller interruption
- speech with 400–800ms pause
- speech with >1s end silence
- known intent phrases
- ambiguous phrases

Produce an E2E matrix showing:
- endpoint time
- STT
- selected node
- total pipeline latency
- pass/fail

---

# PHASE 7 — RTX 3060 Ti EVALUATION

The RTX 3060 Ti **must be considered**, but it must **not be integrated merely because it exists**.

Current known context:
- GPU is present
- current VOIP stack historically used CPU for Silero/SenseVoice
- VAD itself is lightweight and does not need GPU
- prerecorded/runtime audio playback does not benefit meaningfully from GPU
- cloud Groq latency is not fixed by local GPU

Therefore evaluate GPU only where it may materially improve:
- STT latency/accuracy
- optional local STT replacement
- optional local LLM only if it can reduce routing latency without harming correctness
- optional local TTS only if future runtime TTS is actually needed

## GPU evaluation method

First benchmark the optimized CPU pipeline.

Then, only if STT remains a meaningful latency/accuracy bottleneck, create an **isolated** compatible environment for one GPU candidate.

Do not disturb the production Python environment.

Possible isolated approach:
- Python 3.11/3.12 venv if required by CUDA wheels
- faster-whisper CUDA or whisper.cpp CUDA

Compare on the same fixture corpus:

| Metric | CPU Current | GPU Candidate |
|---|---:|---:|
| STT latency | | |
| short phrase accuracy | | |
| weak speech accuracy | | |
| empty rate | | |
| VRAM use | | |
| startup overhead | | |
| implementation complexity | | |

## GPU decision gate

Only recommend integrating 3060 Ti if:
- measurable latency or accuracy improvement is significant
- deployment complexity is acceptable
- it does not destabilize the current voice pipeline

If CPU pipeline already meets the practical latency/accuracy target:
- explicitly conclude GPU is optional/not needed for this stage

Do not force local LLM/TTS migration.

---

# PHASE 8 — PERFORMANCE / LATENCY CLOSURE

Measure the final offline pipeline by stage where possible:

- VAD detection delay
- barge-in candidate delay
- endpoint delay
- STT preprocessing
- STT inference
- dialogue routing
- LLM call if used
- response-node selection

Goal:
Identify the dominant remaining software latency.

Prefer optimizing the largest measured bottleneck rather than micro-optimizing cheap stages.

If a deterministic rule can avoid an unnecessary LLM call, measure the saved latency.

---

# PHASE 9 — FULL REGRESSION

Before declaring offline closure:

Run:
- exact barge-in accumulator regression
- state-guard regression
- endpoint regression
- STT weak-speech regression
- STT normal speech regression
- walker/intent regression
- E2E fixture replay
- import/syntax checks
- directly impacted existing project tests

Do not fix unrelated pre-existing failures unless they are caused by this task.

If any task-introduced regression occurs:
- fix it before closure

---

# OFFLINE CLOSURE GATES

Do not declare PASS unless all software-side gates are satisfied.

## BARGE-IN
- fragmented caller speech detectable
- clear speech trigger latency materially below old full-window delay if safe
- AI-only/noise does not trigger
- one playback generation emits at most one actionable interrupt

## ENDPOINT
- common natural pause fixtures do not prematurely terminate
- true end-of-turn does not become excessively slow

## STT
- valid normal speech does not become empty
- weak speech is not unnecessarily removed by preprocessing
- short phrases are usable
- no material false acceptance of silence/noise

## DIALOGUE
- known deterministic intents route correctly
- ASK_WHY no longer becomes ASK_COMPANY
- LLM is used only where justified by current routing policy

## E2E
- representative fixture → expected response node
- stage timings recorded
- no task-introduced regression

## GPU
- explicit evidence-based decision:
  - INTEGRATE_LATER
  - OPTIONAL
  - NOT_NEEDED
  - BLOCKED_BY_ENV
- do not leave this unanswered

---

# WHEN TO STOP AND ASK FOR LIVE TESTING

Stop only when remaining unknowns are genuinely physical/live-path dependent, for example:

- HT813/PSTN caller attenuation
- actual echo/half-duplex behavior
- real AudioSocket inbound during simultaneous talk
- real subjective "feels natural" barge-in experience
- real phone-network jitter/codec behavior

At that point do not keep changing software to guess around hardware.

---

# FINAL OUTPUT

Return one report only:

# VOIP Offline Voice Quality Full Closure Report

## RESULT
`OFFLINE_CLOSURE = PASS / BLOCKED`

## FINAL CHANGES
Table:
| Area | Problem | Change | Evidence |

## FINAL PARAMETERS
List only parameters actually changed or recommended.

## BARGE-IN
- final algorithm
- clear speech latency
- fragmented speech latency
- false trigger result
- single-trigger proof

## ENDPOINT
- final logic/value
- pause test matrix
- end-of-turn latency impact

## STT
- preprocessing change
- weak speech result
- short phrase result
- normal speech result
- latency

## DIALOGUE
- routing change
- deterministic coverage
- LLM usage before/after
- known misroute regression

## GPU / RTX 3060 Ti
- benchmark performed YES/NO
- candidate tested
- CPU vs GPU table
- final GPU decision

## E2E
Full fixture/result table.

## TESTS
Commands + results.

## FILES CHANGED
Exact files and concise summary.

## PRODUCTION SAFETY
- SAAS_BARGEIN still 0 = YES/NO
- deployed = NO
- Asterisk restarted = NO
- HT813 changed = NO
- production behavior changed = NO

## REMAINING LIVE-ONLY UNKNOWN
List only items that cannot be proven offline.

## REQUIRED LIVE TEST
Provide the **smallest possible** human test plan needed to validate the final candidate.

Do not start deployment.
Do not enable production barge-in.
Stop here.

## Success Philosophy
Do not optimize for "more changes".
Optimize for:
- fewer false decisions
- faster safe interruption
- no premature cut-off
- stronger STT preservation
- deterministic scripted routing
- measurable latency
- minimal complexity

Continue offline until there is no useful software-side work left that can be proven without a real phone call.
