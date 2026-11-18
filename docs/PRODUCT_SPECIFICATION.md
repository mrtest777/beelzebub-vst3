# Beelzebub Product Specification

**Name:** Beelzebub
**Tagline:** Listen. Understand. Correct.
**Initial target:** Windows 64-bit VST3 in Cubase
**Proposed framework:** JUCE with CMake; C++

## 1. Status and intent

This is a specification of intended behavior, not a description of an existing plugin. At the time this document was written, there is no plugin implementation. All capabilities below are planned and must be validated through implementation, listening, and tests before being described as available.

## 2. Product principle

Beelzebub should preserve what sounds good, identify what sounds unpleasant, and correct only what needs correcting. The default behavior should be restrained: analysis is evidence for a correction, not a reason to alter every detected feature. Natural brightness, articulation, expression, and musical content must be treated as valuable unless there is a clear reason to change them.

## 3. Intended users and use

The initial use case is a producer or audio engineer working with vocals in Cubase. The user inserts the plugin on a track, chooses a simple starting point or adjusts detailed controls, listens to the result, and can compare the unprocessed and processed sound. The design may later be useful on other sources, but vocal work is the initial priority.

## 4. Functional goals

All sections in this list are **planned**. They are not currently implemented or tested.

### 4.1 Detection and classification

- Detect sibilance, high-frequency harshness, upper-midrange harshness, and narrow or broad resonances.
- Classify vocal phonetic regions such as sibilants, plosives, vowels, breath, and consonant transitions without requiring speech transcription.
- Estimate singing-related cues: pitch, harmonic structure, formants, vibrato, registers, breathiness, rasp, growl, and intentional distortion.
- Work with changing pitch and timbre, and avoid assuming that every high-frequency event is a defect.

Detection outputs should expose confidence or strength internally so that uncertain evidence can lead to gentler action or no action.

The first detector prototype is planned to be a conservative sibilance detector. It is not intended to represent the full phonetic or singing analysis goals listed above. Prototype evaluation should use a labeled set with at least 8 distinct voices, at least 100 adjudicated sibilant events, and at least 10 minutes of labeled non-sibilant vocal material. Provisional goals are at least 80% event precision and 80% event recall, and no more than one false detection per minute on the labeled non-sibilant material. Report onset and offset timing errors and per-voice results. These are test targets, not proven performance or release claims.

### 4.2 Adaptive correction and preservation

- Apply frequency-selective correction whose amount and shape can vary with detected content.
- Shape consonant attack, body, and tail with small envelope changes when this improves the result.
- Avoid flattening articulation or removing useful breath, brightness, harmonic detail, vibrato, or intentional vocal character.
- Keep processing stable when the signal changes quickly and avoid audible pumping, lisping, dullness, or tonal wobble.
- Provide an optional Smart Air Recovery goal to restore perceived openness after correction. It must not simply boost a fixed high-frequency shelf regardless of the input.
- Provide tonal character controls and optional harmonic enhancement, with predictable bypass and comparison behavior.

The chosen correction method and strength should be inspectable and adjustable. Automatic processing should not hide an uncontrolled chain of changes from the user.

### 4.3 Analysis modes and event review

- **Real-Time Intelligence:** planned causal analysis and correction while audio passes through the plugin. Zero added sample latency is the initial target to validate, not a guarantee. Causal detection cannot anticipate a sibilant, so its response at the event onset must be measured and may trade speed for false triggers or miss some of the initial transient.
- **Deep Analysis, first version:** a user-triggered playback pass that collects event results from the real-time detector. The user starts and finishes the pass, and the plugin records the observed project-sample range. Results are initially for review only; late results must not alter audio that has already passed through the plugin.
- **Future Deep Analysis:** a more sophisticated worker-based engine may consume audio blocks or richer feature data. Its event output format should remain independent of whichever detector produced it. This is a future extension, not a first-version capability.
- **Ghost Automation:** planned display of timestamped, non-destructive events using project sample positions. Events are internal review data, not host automation. Each pass has a generation ID and explicit coverage. Discontinuities, missing timing, or queue loss must produce incomplete or uncertain status. Results are initially temporary and are not saved in plugin state.

Results from a superseded generation are stale and must not replace the current pass. The plugin cannot reliably detect every Cubase edit that could make an event list outdated; users must be able to clear results and run the analysis again after editing the audio or timeline.

Analysis is complete only over a user-declared range that the plugin actually observed with valid timing and no unexplained gaps or jumps. The plugin must not imply that it analyzed an entire Cubase project unless it can demonstrate coverage of that entire intended range. Starting mid-track can produce a valid partial-range analysis, but not a claim of full-project coverage.

Cubase Direct Offline Processing (DOP) is a future compatibility experiment. It is not required for the first release. Any support must be tested in the intended Cubase version and edition.

The product must clearly communicate when an analysis mode adds latency, requires a pass, or has reduced confidence.

### 4.4 User interface and workflow

- Simple mode should provide a small set of meaningful controls and clear feedback.
- Professional mode should expose deeper detection, timing, spectral, and correction controls without making the default workflow intimidating.
- Use a premium dark visual design with readable text, clear state changes, and accessible control sizes.
- Every interactive control must affect audio, analysis, view state, or a documented workflow. No decorative control should imply functionality it does not have.
- Provide A/B comparison, Delta monitoring of the removed/changed component, profiles, and user preferences.
- Planned learning should be explicit and user-controlled. Any learned preference must be inspectable, persisted intentionally, and resettable.

### 4.5 Host and plugin behavior

- Stereo support is required for the initial plugin.
- Expose meaningful automatable parameters to the host.
- Save and restore plugin state reliably, including relevant mode and parameter settings.
- Provide stable VST3 operation in Cubase on Windows 64-bit.
- Report processing latency accurately to the host whenever the selected mode introduces latency.
- Begin Real-Time Intelligence with causal processing and target zero added sample latency; validate this target in implementation and report any later mode that adds latency.
- Begin stereo detection by combining left/right channel power and apply one shared correction amount to both channels. This is intended to reduce stereo image movement; it must be evaluated against asymmetric stereo material.
- Respect host transport, sample-rate changes, and lifecycle callbacks.

## 5. Quality requirements

- **Real-time safety:** the audio callback must not allocate memory, access files or the network, or wait on blocking locks.
- **Audio quality:** corrections should be judged on representative voices and musical material, including edge cases such as breathy, raspy, distorted, and high-register singing.
- **CPU:** measure representative worst-case CPU use at supported sample rates and block sizes; set concrete budgets before release.
- **Latency:** measure and report latency; establish acceptable mode-specific limits before release.
- **Stability:** test repeated enable/bypass, preset and state changes, sample-rate and block-size changes, and long playback sessions in supported hosts.
- **Phase and stereo behavior:** test phase response, stereo image, channel linking, and mono compatibility for each processing mode.
- **Detector evaluation:** use at least 8 distinct voices, 100 adjudicated sibilant events, and at least 10 minutes of labeled non-sibilant vocal material. Provisional prototype goals are at least 80% event precision and recall and no more than one false detection per minute on non-sibilant material. Report onset/offset timing errors and per-voice results. Evaluate correction quality separately through blinded listening; detector scores alone do not establish sound quality.
- **Usability:** ensure the user can hear what changed, compare it, and reduce or disable processing easily.

Numeric release targets for CPU, latency, detection accuracy, and supported host versions remain to be established from prototypes and listening tests. They must not be invented as achieved results.

## 6. Scope boundaries and open decisions

- Speech transcription is not required and is outside the intended analysis approach.
- The plugin should not automatically write or modify host automation without a clearly defined, user-approved workflow.
- Exact detector models, correction topology, parameter ranges, latency strategy, and profile format remain design decisions.
- Event results are initially temporary; saving them in plugin state and keeping them valid after project edits are future investigations.
- No full-project coverage is assumed from ordinary plugin playback. The first analysis workflow reports only the user-declared range that was observed.
- The first release target is Windows 64-bit VST3; other operating systems, formats, and hosts are not promised.
- “Learning” must not imply network use or collection of user audio. The default design should be local and transparent; any future data handling requires a separate explicit product decision.

## 7. Acceptance principle

A feature is accepted only when it has an implementation, documented controls and behavior, relevant automated or manual verification, and listening review. Documentation of a goal alone does not make it implemented.
