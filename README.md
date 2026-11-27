# EchoLine — Stereo Delay Plugin (JUCE)

A clean, lightweight stereo delay audio plugin built with [JUCE](https://juce.com).
Builds as **VST3**, **AU** (macOS) and **Standalone**.

> Status: early development. Parameters and sound may change between versions.

---

## Features

- **Delay time**: 1 ms – 2000 ms, or **tempo-synced** to the host (1/32 to 1 bar, dotted and triplet)
- **Feedback**: 0 – 95 %, with a safety limiter in the feedback loop
- **Mix**: dry/wet blend
- **Ping-pong mode**: echoes bounce between the left and right channels
- **Feedback filters**: low-cut and high-cut inside the loop for darker or thinner repeats
- **Smoothed parameters**: changing the delay time or other controls doesn't cause clicks or zipper noise
- **Real-time safe**: no allocations, locks or file I/O on the audio thread
- Mono → stereo and stereo → stereo bus layouts
- State is saved with your DAW project (all parameters are automatable)

## Parameters

| Parameter     | Range                   | Default  |
|---------------|-------------------------|----------|
| Time          | 1 – 2000 ms             | 350 ms   |
| Sync          | Off / On                | Off      |
| Note Division | 1/32 … 1/1 (· and T)    | 1/8      |
| Feedback      | 0 – 95 %                | 35 %     |
| Mix           | 0 – 100 %               | 30 %     |
| Ping-Pong     | Off / On                | Off      |
| Low Cut       | 20 Hz – 2 kHz           | 80 Hz    |
| High Cut      | 1 kHz – 20 kHz          | 12 kHz   |
| Output        | -24 – +6 dB             | 0 dB     |

---

## Building from source

### Requirements

- **CMake** 3.22 or newer
- A C++17 compiler
  - Windows: Visual Studio 2022 (Desktop development with C++)
  - macOS: Xcode 14+
  - Linux: GCC 11+ or Clang 14+, plus the
    [JUCE Linux dependencies](https://github.com/juce-framework/JUCE/blob/master/docs/Linux%20Dependencies.md)
- **JUCE 8** (included as a git submodule)

### Clone

```bash
git clone --recurse-submodules https://github.com/<your-username>/EchoLine.git
cd EchoLine
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

### Configure and build

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

Built plugins are placed in:

```
build/EchoLine_artefacts/Release/VST3/EchoLine.vst3
build/EchoLine_artefacts/Release/AU/EchoLine.component        (macOS only)
build/EchoLine_artefacts/Release/Standalone/
```

To copy the plugin to your system plugin folder automatically after building,
set `COPY_PLUGIN_AFTER_BUILD TRUE` in `CMakeLists.txt`.

### Install manually

| OS      | VST3 folder                                   |
|---------|-----------------------------------------------|
| Windows | `C:\Program Files\Common Files\VST3\`         |
| macOS   | `~/Library/Audio/Plug-Ins/VST3/`              |
| Linux   | `~/.vst3/`                                    |

Rescan plugins in your DAW after copying.

---

## Project structure

```
EchoLine/
├── CMakeLists.txt
├── JUCE/                    # JUCE submodule
├── Source/
│   ├── PluginProcessor.h/.cpp   # audio processing, parameters, state
│   ├── PluginEditor.h/.cpp      # user interface
│   └── DSP/
│       ├── DelayLine.h          # circular buffer with fractional (interpolated) reads
│       └── FeedbackFilter.h     # low-cut / high-cut in the feedback loop
└── README.md
```

## How it works

1. `prepareToPlay()` allocates a circular buffer large enough for the maximum
   delay time at the current sample rate. This is the only place memory is allocated.
2. Each sample, the delay line is read at `time × sampleRate` samples behind the
   write position. Reads between samples use linear (or Lagrange) interpolation,
   so the delay time can change smoothly.
3. The delayed signal passes through the low-cut/high-cut filters and a soft
   limiter, is scaled by **Feedback**, and is written back into the buffer.
4. In ping-pong mode, the feedback from the left channel is written into the
   right channel's buffer and vice versa.
5. Dry and wet signals are blended by **Mix**, then **Output** gain is applied.

All parameters use `juce::AudioProcessorValueTreeState` and
`juce::SmoothedValue` to avoid clicks.

---

## Testing

- Load the VST3 in a host such as REAPER, Ableton Live, Bitwig, Cubase or FL Studio,
  or in JUCE's **AudioPluginHost** (`JUCE/extras/AudioPluginHost`).
- Validate with [pluginval](https://github.com/Tracktion/pluginval):

```bash
pluginval --strictness-level 10 --validate build/EchoLine_artefacts/Release/VST3/EchoLine.vst3
```

## Roadmap

- [ ] Modulation (wow/flutter) on the delay time
- [ ] Tape-style saturation in the feedback loop
- [ ] Preset browser
- [ ] CLAP format

## Contributing

Issues and pull requests are welcome. Please:

- Keep the audio thread real-time safe: no `new`/`delete`, locks or logging in `processBlock()`
- Run pluginval before opening a pull request
- Follow the existing code style (JUCE naming conventions)

## License

This project is licensed under the **GNU AGPLv3**. See [LICENSE](LICENSE).

JUCE 8 is dual-licensed under AGPLv3 and the commercial JUCE license. Because this
plugin uses the open-source (AGPLv3) JUCE license, the plugin must also be released under
AGPLv3 (or a compatible license). To release it under a closed-source or permissive
license, you need a commercial JUCE license.

VST is a registered trademark of Steinberg Media Technologies GmbH.
