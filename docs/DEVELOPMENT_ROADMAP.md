# Beelzebub Development Roadmap

## Status convention

This roadmap describes planned work. At the time of writing, the repository contains documentation only: no milestone feature is implemented or tested. Future updates should keep **Planned**, **Implemented**, and **Tested** separate. Move an item from Planned only when the code exists; mark it Tested only after the named verification has actually passed.

## Milestone 0.1 — Project foundation

**Planned**

- Select and document the JUCE version, toolchain, license approach, and supported Cubase test setup.
- Create the CMake/JUCE project and a minimal Windows 64-bit VST3 plugin shell.
- Establish stable parameter identifiers, a versioned state format, and a basic stereo signal path.
- Create the first build and developer workflow documentation.

**Implemented:** None.
**Tested:** None.
**Exit checks planned:** reproducible build; plugin loads in the chosen Cubase version; state save/restore and host lifecycle work; callback inspected for forbidden operations.

## Milestone 0.2 — Safe audio and control foundation

**Planned**

- Implement clean bypass, output handling, smoothing, and meaningful initial controls.
- Add A/B comparison and a first definition of Delta monitoring.
- Verify stereo routing, host automation, and state persistence.
- Establish baseline CPU, latency, phase, and bypass-transition measurements.

**Implemented:** None.
**Tested:** None.
**Exit checks planned:** automated parameter/state checks; measured latency and CPU baseline; Cubase automation and project reload review; listening review for clicks and level changes.

## Milestone 0.3 — First detection and restrained correction

**Planned**

- Prototype conservative sibilance detection exclusively, with confidence values.
- Add conservative, adaptive sibilance correction with user-adjustable strength.
- Target causal processing with zero added sample latency; validate onset response, phase behavior, and audio quality rather than treating zero latency as guaranteed.
- Begin with combined stereo channel power for detection and one shared correction amount across channels.
- Provide clear metering so users can understand when and why correction acts.
- Begin building representative vocal test material and repeatable listening comparisons.
- Evaluate the sibilance prototype with at least 8 distinct voices, 100 adjudicated events, and 10 minutes of labeled non-sibilant vocal material. Provisional goals: at least 80% precision and recall, no more than one false event per minute of non-sibilant material, with onset/offset errors and per-voice results reported. Evaluate correction quality separately through blinded listening.

**Implemented:** None.
**Tested:** None.
**Exit checks planned:** detector behavior on voiced, unvoiced, breathy, and bright material; audio quality review for lisping, dullness, pumping, and missed events; real-time safety and CPU review.

## Milestone 0.4 — Phonetic and temporal awareness

**Planned**

- Distinguish likely sibilants, plosives, vowels, breath, and consonant transitions without transcription.
- Add attack/body/tail-aware micro-envelope shaping where listening tests support it.
- Improve detection for narrow and broad resonances and upper-midrange harshness.
- Add first-useful Simple and Professional interface modes; remove or defer any control without working behavior.

**Implemented:** None.
**Tested:** None.
**Exit checks planned:** classification review on varied speech and singing without transcription; attack/tail artifact review; every visible control mapped to real behavior.

## Milestone 0.5 — Singing-aware preservation and tonal tools

**Planned**

- Improve estimates of pitch, harmonics, formants, vibrato, and register.
- Account for breathiness, rasp, growl, and intentional distortion in preservation decisions.
- Prototype Smart Air Recovery, tonal character controls, and optional harmonic enhancement.
- Improve stereo linking and verify phase and mono compatibility.

**Implemented:** None.
**Tested:** None.
**Exit checks planned:** preservation listening review across vocal styles; comparison against unprocessed and deliberately harsh cases; phase, stereo, CPU, and latency measurements.

## Milestone 0.6 — Deep Analysis and event review

**Planned**

- Implement the first user-triggered playback pass, collecting events from the real-time detector without storing the complete audio.
- Store the pass results temporarily in memory and display timestamped, non-destructive Ghost Automation events using project sample positions.
- Track generation IDs, user-declared coverage, and pass status; detect discontinuities and report partial, interrupted, missing-timing, stale-generation, or queue-overflow results honestly. Provide clear/rerun controls because project edits cannot be detected reliably.
- Keep results for review only; do not let late Deep Analysis results drive real-time correction.
- Keep the event format independent of the detector so a future background analysis engine can consume richer features or audio blocks.
- Investigate project-state persistence and Cubase Direct Offline Processing only as future compatibility work; neither is a first-release dependency.
- Add profiles and explicit, resettable user preferences or learning behavior.
- Refine cancellation, state ownership, event confidence, and long-session memory limits.

**Implemented:** None.
**Tested:** None.
**Exit checks planned:** worker/audio-thread isolation review; event timing and stale-generation tests; coverage checks for mid-track starts, early stops, jumps, loops, missing timing, and queue loss; temporary-result behavior; host workflow validation in Cubase. DOP and persisted event state remain experiments outside the first-release gate.

## Milestone 0.7 — Release candidate and quality pass

**Planned**

- Complete the intended Simple and Professional workflows and visual polish.
- Finalize supported Windows, VST3, and Cubase compatibility statements.
- Set and meet documented CPU, latency, stability, and audio-quality acceptance targets.
- Run regression, state migration, host automation, long-session, and listening test passes.
- Prepare user documentation and release packaging only after results support the claims.

**Implemented:** None.
**Tested:** None.
**Exit checks planned:** all release-blocking tests pass; no decorative controls; no known critical stability or audio-quality defects; measured results are documented; project state and installation instructions are reviewed.

## Cross-milestone practices

- Keep the audio callback bounded, allocation-free, file-free, network-free, and free from blocking locks.
- Treat detection as uncertain evidence; preserve content when confidence is weak.
- Maintain a regression set containing varied voices and intentionally challenging material.
- Record test conditions and results instead of making unverified performance claims.
- Do not advance a feature's status based on a roadmap entry, mock-up, or control that has no working behavior.
