# Beelzebub Technical Architecture

## 1. Status

This document proposes an architecture; it is not an inventory of existing code. The repository currently has no plugin source code, CMake configuration, or tests. All components below are planned. Choices should be checked with small prototypes before they become permanent design constraints.

## 2. Proposed high-level structure

The plugin is planned as a set of cooperating layers:

1. **JUCE/VST3 adapter:** receives audio, host timing, parameter changes, and state requests; provides the plugin editor and reports latency.
2. **Parameter and state model:** defines stable parameter identifiers, defaults, ranges, host automation behavior, and versioned saved state.
3. **Real-time DSP path:** processes audio using preallocated buffers and fixed-size state. It consumes a prepared parameter snapshot and makes no blocking calls.
4. **Analysis path:** estimates vocal events and their confidence. The real-time version is bounded in work per audio block; deeper analysis may run outside the audio callback under a separately designed workflow.
5. **Correction engine:** turns analysis and user controls into bounded spectral and envelope changes, with preservation safeguards.
6. **User interface:** renders Simple and Professional modes, visualization and event review, and sends parameter or non-real-time requests through safe communication paths.

The initial implementation should keep modules small and independently testable. Suggested logical modules are `PluginProcessor`, `PluginEditor`, `ParameterState`, `RealtimeAnalyzer`, `DeepAnalyzer`, `CorrectionEngine`, `EventStore`, and `ProfileStore`. These are proposed names only; no such source files exist yet.

## 3. Audio callback rules

The processor callback must be bounded and predictable. It must not:

- allocate or free memory;
- open, read, or write files;
- perform network access;
- wait on a mutex or other blocking lock;
- perform UI work, logging that can block, or unbounded analysis;
- depend on editor creation or visibility.

Buffers, delay lines, FFT work areas, smoothing state, and event queues should be allocated or sized during preparation, outside processing. Callback work should depend only on configured channel count, block size, and a bounded amount of DSP work. If the host supplies an unexpected configuration, the processor should fail safely rather than allocate in the callback.

Parameters should be read using JUCE's supported real-time-safe parameter mechanisms or copied into a coherent snapshot. UI-to-audio and audio-to-UI communication should use atomics or bounded single-producer/single-consumer queues where appropriate; queue overflow behavior must be defined. The audio thread must never wait for the UI.

For analysis reporting, the audio thread is the producer of compact, fixed-size frame/event messages and the analysis worker is the consumer. Queue overflow must drop reporting data rather than block; the associated pass is marked incomplete. The worker may merge adjacent detections and publish a read-only snapshot to the interface thread. The interface must not access mutable audio-thread state directly. Large snapshots are copied or swapped outside the callback.

## 4. Proposed signal and analysis flow

```text
Host audio
   -> input validation / channel handling
   -> bounded real-time feature analysis
   -> confidence and preservation decisions
   -> adaptive correction controls
   -> frequency-selective DSP and optional envelope shaping
   -> output / Delta tap / metering
```

The analyzer should separate observation from correction. The first detector estimates sibilance evidence and confidence from causal signal features. A preservation layer should limit or suppress correction when evidence is uncertain. The correction engine should receive bounded targets, not an opaque instruction to remove every detected feature.

Candidate techniques include filter banks or FFT-based spectral analysis, transient/envelope tracking, pitch and harmonic estimation, and time-varying filters or gains. The exact methods are open until listening tests establish their tradeoffs in CPU, latency, phase response, and sound quality. No particular machine-learning model is assumed by this specification.

The initial sibilance prototype should use combined left/right channel power for detection and one shared correction amount for both channels. Pool channel powers rather than summing waveforms, avoiding cancellation from opposite-phase content. This is a starting strategy, not a proven best approach; test centered, asymmetric, and stereo-effect material.

## 5. Real-Time Intelligence and Deep Analysis

**Real-Time Intelligence** is planned to analyze and process the stream with bounded per-block work. The initial target is causal processing with zero added sample latency. This is a target to validate, not a guarantee. A causal detector sees the sibilant only after its onset, so correction may miss part of the leading transient. Faster response can increase false triggers; smoothing to reduce false triggers can delay action. Measure event onset response, phase behavior, and artifacts. Any future lookahead must be fixed, reported to the host, and documented as latency.

### First version: playback event collection

The first Deep Analysis workflow is user-triggered. The user starts analysis, plays the intended range, then finishes the pass. During playback, the ordinary real-time detector emits compact timestamped results; the worker merges them into events. The plugin need not store the full audio. This pass improves coverage and event review, but does not use a more sophisticated detector than Real-Time Intelligence. Its results are for review only and must never trigger late corrections to audio that has already passed.

Each pass has a monotonically changing generation ID. A new pass supersedes the prior pass. Event records carry the generation ID so late worker results from an old pass can be discarded. Track explicit coverage start/end in project samples and states such as running, complete over the declared observed range, partial, interrupted, missing timing, or incomplete due to lost queue data.

Results for an older generation are stale and must not overwrite the current snapshot. The plugin cannot reliably detect all edits or clip moves in Cubase that could invalidate previously displayed timeline events. Provide explicit clear/rerun controls and identify results as belonging to a particular pass; do not claim automatic edit-aware invalidation.

Treat a pass as complete only over a user-declared range observed with valid timing and no unexplained discontinuities. Starting mid-track may yield a valid partial range, but not a claim of full-project coverage. Detect jumps and loop wraps by comparing each process block’s project position with the expected next position. If timing is missing, a gap is unexplained, or the host stops calling the plugin during a region, report uncertainty or incompleteness. The plugin cannot infer the user’s intended project range or guarantee whole-project coverage without evidence from the host.

### Future version: independent background engine

Keep the event schema and event store independent of the detector. A future worker may consume richer timestamped feature frames or audio blocks through a preallocated bounded handoff and run more expensive analysis. If it falls behind, drop or mark missing data rather than blocking the audio thread. The user interface and event review should consume the same event format regardless of detector. This should limit redesign of event review, though worker-side audio capture and scheduling remain additional work.

The initial event data is temporary in memory and is not saved in plugin state. Persisting events with the Cubase project is a future investigation because it adds state size and stale-result questions after edits or moved audio.

Ghost Automation displays timestamped, non-destructive events and does not write host automation. Use project sample positions as the primary time coordinate; musical-position display is optional and depends on valid host context. Event data should include generation ID, start/end project samples, classification, confidence, frequency bounds, suggested correction, and pass/source metadata. Cubase Direct Offline Processing is a future compatibility experiment, not a first-release dependency. VST3 offline processing can call the processor in offline mode, but does not by itself provide project-wide event discovery or persistence; test DOP behavior in the intended Cubase version.

## 6. Stereo, phase, and bypass

The initial stereo strategy is linked: combine left/right channel power for detection and apply one shared correction amount to both channels while processing the channel audio paths separately. This should reduce time-varying image movement compared with independent correction, though asymmetric material may produce a less sensitive or over-triggered decision. Evaluate it against strongest-channel and independent alternatives before changing the strategy. Any frequency-dependent or time-varying process should be measured for phase response and mono compatibility. Zero added sample latency does not mean zero phase shift.

Bypass and A/B switching should avoid clicks, use an intentional transition strategy, and remain reliable under host automation. Delta monitoring must use a clearly defined signal difference and must not accidentally alter the main output level or state.

## 7. Parameters, automation, and state

Parameters should have stable identifiers from the beginning so Cubase automation remains meaningful across releases. Exposed controls should be meaningful and documented. Internal detector values need not all become host parameters.

Saved state should be versioned and validated when loaded. State changes requested by the host must be transferred safely to the processing state, with no file access from the audio callback. Profiles and preferences should use a deliberate persistence mechanism outside the callback. The design must define what is included in a project-saved state versus user-level preferences.

## 8. JUCE, CMake, and VST3 build layout

The planned project will use CMake and JUCE to build a Windows 64-bit VST3. The exact JUCE acquisition method, version pinning, license review, compiler version, and Cubase compatibility range must be selected before the first build milestone. Builds should be reproducible and should not silently depend on locally installed, untracked components.

Potential layout once implementation begins:

```text
beelzebub-vst3/
  CMakeLists.txt
  README.md
  docs/
  Source/
  Tests/
```

This is a future layout suggestion. At documentation creation time, only the README and requested documentation are intended to exist.

## 9. Verification strategy

Verification is planned at several levels:

- **DSP unit checks:** detector feature behavior, parameter bounds, filters, smoothing, state conversion, and edge cases.
- **Real-time safety review:** inspect callback paths for allocation, file/network access, blocking locks, and unbounded work; use suitable instrumentation in development builds.
- **Audio measurements:** CPU, latency, frequency and phase response, stereo image, bypass transitions, and output levels.
- **Listening review:** normal and difficult vocals, intentional distortion, breaths, plosives, bright instruments, silence, and noisy material.
- **Host checks:** load, automation, save/restore, bypass, reconfiguration, and long playback in the supported Cubase configuration.
- **Sibilance prototype set:** at least 8 distinct voices, at least 100 adjudicated sibilant events, and at least 10 minutes of labeled non-sibilant vocal material. Freeze settings before evaluation; use one-to-one event matching with documented onset/overlap tolerances. Provisional goals are at least 80% event precision and recall and no more than one false event per minute on non-sibilant material. Report raw counts, onset/offset timing errors, and per-voice results.
- **Correction listening:** evaluate separately from detector scoring using blinded comparisons; record improvements, neutral cases, and articulation/lisp artifacts. Detector metrics do not prove correction quality.
- **Analysis pass:** verify generation supersession, coverage reporting, playback jumps, loops, stopped/early passes, queue overflow, missing timing, and temporary-result behavior in Cubase.
- **DOP exploration:** test as a later compatibility experiment only; it is not a first-release dependency.

These checks are planned and have not been run. Concrete numeric targets and test fixtures must be selected during implementation.
