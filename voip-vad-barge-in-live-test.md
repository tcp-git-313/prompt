# VOIP VAD / Barge-in Live Test — Phase 1 Completion

## Role
VOIP VAD / BARGE-IN DIAGNOSTIC OWNER

## Current verified state
The previous audit and deployment already confirmed:

- Production uses Asterisk + AudioSocket on port 9090.
- Runtime file: `/usr/local/lib/saas/saas_streaming_server.py`.
- Shadow diagnostic instrumentation is deployed.
- Current runtime SHA256 begins with `9238c8ea...`.
- `saas-streaming` restarted cleanly: PID 439 → 1837.
- Silero VAD loaded and port 9090 is listening.
- `SAAS_BARGEIN=0` remains unchanged.
- `SAAS_VAD_THRESH` is unset, therefore effective threshold = 0.5.
- Asterisk was not restarted.
- Production dialplan contains AudioSocket and MixMonitor.
- No historical data can answer the live VAD questions; a new human test is required.

Continue from STEP 4. Do not repeat the full architecture audit.

## Goal
Complete the live A/B/C diagnostic and determine whether caller speech is visible to AudioSocket/Silero VAD while AI audio is playing.

This phase is diagnostic only. Do not enable barge-in and do not tune production behavior.

## Allowed minimal correction
The previous shadow logger only emitted during `PLAYING`, but Test B requires a `LISTENING` baseline.

Change only the diagnostic condition from:

`state == PLAYING`

to the equivalent of:

`state == PLAYING OR state == LISTENING`

Keep the same shadow metrics:

- monotonic timestamp
- call/ASID
- state
- RMS dBFS
- peak dBFS
- Silero VAD probability
- speech true/false
- consecutive speech ms if already available

Sampling should remain approximately 100–200 ms.

This is observability only. Do not change the existing VAD decision path.

## Hard boundaries
Do not change:

- `SAAS_BARGEIN=0`
- VAD threshold
- VAD model
- endpoint timing
- HT813 settings
- Asterisk gain
- STT
- TTS
- dialogue/walker logic
- Groq
- GPU/CUDA configuration
- production dialogue behavior

Do not restart Asterisk.

If the minimal logger change requires restarting only `saas-streaming`, that is allowed after syntax validation. Record before/after PID and verify port 9090/service health.

## Human live test procedure
Prepare the logger and journal capture first. Do not originate until the user says they are ready.

Use one real call if possible.

### Test A — AI ONLY
During an AI playback segment, the user stays completely silent.

Capture the PLAYING interval and calculate:

- RMS median/max
- peak median/max
- VAD probability median/max
- any speech=true events

This is the playback-leakage baseline.

### Test B — CALLER ONLY
When the system is in LISTENING state and AI is not playing, the user says:

`你好，我現在有在講話`

Capture the LISTENING interval and calculate the same metrics.

This is the normal caller-speech baseline.

### Test C — REAL BARGE-IN SHADOW
During AI playback, the user deliberately speaks over the AI twice:

1. `等一下`
2. `我有問題`

`SAAS_BARGEIN` must remain 0, so the AI continuing to speak is expected.

For each interruption, record:

- approximate caller speech start
- RMS rise
- peak
- VAD probability rise/max
- whether speech=true occurred
- delay from caller speech onset to VAD rise if measurable

Use MixMonitor only as a timing/reference recording. Do not treat MixMonitor as the VAD input path.

## Analysis
Compare A / B / C and classify only from evidence:

- **A — VAD detects caller correctly; barge-in is only disabled**
- **B — playback leakage itself causes high/false VAD**
- **C — caller audio is attenuated on the AudioSocket path during playback**
- **D — audio level rises but Silero classification does not**
- **E — AudioSocket does not receive meaningful caller audio during playback**
- **F — inconclusive**

Multiple findings are allowed only when supported by measured data.

Do not assume HT813 half-duplex is the cause unless the live measurements support it.

## Final report
Return only:

### CURRENT
- SAAS_BARGEIN
- effective VAD threshold
- runtime SHA
- service/PID
- whether PLAYING and LISTENING shadow logging worked

### TEST A — AI ONLY
- RMS median/max
- peak median/max
- VAD median/max
- speech=true count

### TEST B — CALLER ONLY
- same metrics

### TEST C — BARGE-IN
For each phrase:
- phrase
- RMS/peak
- VAD max
- speech detected YES/NO
- approximate detection delay

### FINDING
- classification A/B/C/D/E/F
- concise evidence

### BLOCKER
- the single most important blocker, if proven

### NEXT STEP
- one smallest safe next experiment only

### RUNTIME CHANGES
- files changed
- restart details
- `SAAS_BARGEIN remains 0 = YES/NO`
- Asterisk restart = NO

Stop after the report. Do not implement the fix.
