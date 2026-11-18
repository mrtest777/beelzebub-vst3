# Beelzebub

**Listen. Understand. Correct.**

Beelzebub is a planned Windows 64-bit VST3 audio plugin, initially aimed at use in Cubase. It is intended to help identify and reduce unpleasant sibilance, harshness, and resonances while preserving the parts of a performance that already sound good.

> Preserve what sounds good. Identify what sounds unpleasant. Correct only what needs correcting.

## Project status

This repository currently contains project documentation only. No plugin source code, audio processing, build configuration, or automated tests have been created. Every capability described here is a **product goal or proposal**, not an implemented feature. A capability will be called implemented and tested only after its code exists and its relevant tests have passed.

## Intended platform and tools

- Windows 64-bit
- VST3, initially targeting Cubase
- C++ with JUCE
- CMake for project configuration and builds

These are project requirements; the development toolchain has not yet been installed or verified.

## Product direction

Beelzebub is planned to analyze vocal-like and other musical material for sibilance, high-frequency and upper-midrange harshness, and narrow or broad resonances. It should take phonetic and singing cues into account without transcribing speech, then make restrained, adaptive corrections. Its interface is planned to offer Simple and Professional modes, useful event review, A/B and Delta monitoring, profiles, and user learning/preferences.

The complete intended behavior and limitations are described in [the product specification](docs/PRODUCT_SPECIFICATION.md). The proposed software design is in [the technical architecture](docs/TECHNICAL_ARCHITECTURE.md), and staged delivery is in [the roadmap](docs/DEVELOPMENT_ROADMAP.md).

## Initial analysis design

Real-Time Intelligence is planned around causal processing, with zero added sample latency as an initial target to validate, not a guarantee. The first Deep Analysis workflow is planned as a user-triggered playback pass that collects events from the real-time detector. Those results are for review only and will not trigger late corrections to live audio. Event records will use project sample positions, generation IDs, and explicit coverage status. First-version results are temporary and are not planned to persist in the Cubase project state.

The first detector prototype is planned to focus on conservative sibilance detection. Its evaluation will use at least eight voices, 100 adjudicated sibilant events, and ten minutes of labeled non-sibilant vocal material. Prototype precision and recall targets are 80%, with no more than one false detection per minute of non-sibilant material. These are evaluation goals, not achieved results.

## Development approach

Work is planned in milestones 0.1 through 0.7. Early milestones establish a working plugin shell and safe fundamentals before adding analysis and correction. Later milestones add deeper singing-aware analysis, review tools, and finishing work. Each milestone must name what is planned, what has actually been implemented, and what has been tested; at this initial documentation stage, nothing is implemented or tested.

## Current repository contents

- `README.md`
- `docs/PRODUCT_SPECIFICATION.md`
- `docs/TECHNICAL_ARCHITECTURE.md`
- `docs/DEVELOPMENT_ROADMAP.md`
