# POCKET CHOP

> A professional sample slicer and step sequencer for iPad, built entirely in Swift Playgrounds.

![Platform](https://img.shields.io/badge/platform-iPadOS%2018.1+-black?style=flat-square)
![Swift](https://img.shields.io/badge/swift-5.9-orange?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

| | |
|---|---|
| ![Chop selection](Screenshots/chopSelection.PNG) | ![Chop playing](Screenshots/chopPlaying.PNG) |
| ![Patterns & recordings](Screenshots/patternRecording.png) | ![Piano roll](Screenshots/pianoRoll.PNG) |

---

## What is it?

PocketChop lets you import any audio file, slice it into up to 16 named regions (chops), assign each chop to a sample pad, and build musical patterns in a 64-step sequencer — all without leaving Swift Playgrounds. It is designed for iPad and works entirely offline.

---

## Features

### Sample Engine
- Import MP3, WAV, AIF or M4A files via the system file picker
- Visual waveform display with pinch-to-zoom
- Up to 16 chops per sample — draw them by hand or auto-slice into 2 / 4 / 8 / 16 equal regions
- Left handle drags the whole chop; right handle resizes it — frame-accurate, no gesture conflicts

### 16 Sample Pads
- Zero-latency pad triggering via UIKit touch events
- Each pad shows a mini waveform of its assigned chop
- Pads can play simultaneously (polyphonic)
- Physical iPad keyboard support: `1–8` and `Q–I` map to pads 1–16 with full press/release gate

### FX & ADSR per Pad
- Delay mix, reverb mix, pitch (±24 semitones), time-stretch rate
- Full ADSR amplitude envelope baked into a cached buffer — no real-time allocation on the trigger path
- Target a single pad or all 16 at once

### 64-Step Sequencer
- Adjustable BPM (40–200), scrubbed by dragging
- Live pattern recording with automatic note-length capture
- Quarter-note metronome click
- Step indicator with auto-scroll during playback

### Piano Roll
- Per-pattern note editor for all 16 pads across 64 steps
- Tap to select, drag to move (time and pitch), double-tap to delete
- Resize notes by dragging the right edge
- Conflict-free gesture system — no accidental deletions

### Audio Recording
- Captures the full engine output (all pads + FX) to a CAF file
- Preview recordings in-app with a play/stop toggle
- Share directly via the iOS share sheet

### Onboarding
- Step-by-step first-launch guide covering every feature
- Shown once, skippable, never repeated

---

## Requirements

| | |
|---|---|
| **Device** | iPad (any model supporting iPadOS 18.1) |
| **Software** | Swift Playgrounds 4.x or Xcode 16+ |
| **Orientation** | Portrait and landscape |
| **Keyboard** | Optional — any connected iPad keyboard |

---

## Getting Started

1. Clone or download this repository.
2. Open `PocketChop.swiftpm` in Swift Playgrounds or Xcode.
3. Place your default audio file (MP3/WAV/AIF/M4A) **at the root level of the package** — not inside any subfolder — and update the file name in `SamplerEngine.swift`:

```swift
// SamplerEngine.swift → loadDefaultSample()
Bundle.main.url(forResource: "YourFileName", withExtension: "mp3")
```

4. Build and run on a connected iPad or the Playgrounds simulator.

> **Note:** Swift Playgrounds flattens the bundle at build time — all resources must be at the package root, not inside subfolders.

---

## Keyboard Shortcuts

When a physical keyboard is connected:

| Keys | Pads |
|---|---|
| `1` `2` `3` `4` `5` `6` `7` `8` | Pads 1 – 8 |
| `Q` `W` `E` `R` `T` `Y` `U` `I` | Pads 9 – 16 |

Hold a key to sustain a pad; release to trigger the ADSR release tail — identical behaviour to the on-screen pads.

---

## Project Structure

```
PocketChop.swiftpm
├── Engine/
│   ├── AudioEngine.swift       — AVAudioEngine graph, 16 signal chains, metronome
│   ├── SamplerEngine.swift     — Main ObservableObject, coordinates all subsystems
│   ├── Sequencer.swift         — Background-thread 64-step timer
│   └── SliceCache.swift        — Pre-baked per-pad buffer cache
├── Models/
│   └── Models.swift            — Data types, color palette, helpers
├── Utilities/
│   └── WaveformBuilder.swift   — Off-thread waveform downsampling
├── Views/
│   ├── ContentView.swift       — Root layout
│   ├── TopBar.swift            — Branding, file info, import button
│   ├── OnboardingView.swift    — First-launch step-by-step guide
│   ├── Controls/
│   │   ├── SharedComponents.swift  — DTSmallButton, HorizontalKnob, InstantTouchView
│   │   └── KnobRow.swift           — FX / ADSR knob strip
│   ├── Waveform/
│   │   └── WaveformPanel.swift     — Waveform display, chop editing
│   ├── Pads/
│   │   └── PadsPanel.swift         — 8×2 pad grid
│   ├── Pattern/
│   │   ├── PatternManager.swift    — Pattern library
│   │   ├── PianoRollView.swift     — Piano roll editor
│   │   └── PatternBar.swift        — Transport bar
│   └── Recordings/
│       └── RecordingsPanel.swift   — Full-mix recording list
└── KeyboardPadInput.swift      — Physical keyboard → pad mapping
```

---

## Architecture Notes

**No third-party dependencies.** Everything is built on AVFoundation, SwiftUI, and UIKit interop.

**Audio thread safety.** The sequencer runs on a dedicated `DispatchQueue` with `.strict` timer flags to prevent timing drift from UI work. All pad triggers and state updates are dispatched to `@MainActor`.

**Gesture design.** All drag gestures use a latch-once `@GestureState` snapshot pattern — the starting value is captured exactly once when the finger lands and never mutated during the gesture. This prevents the delta-compounding bug that causes controls to jump or become inaccessible.

**SliceCache.** Each pad's audio buffer is pre-baked (sliced + ADSR envelope applied) and cached. The hot trigger path performs zero allocation — it only schedules the cached buffer on the player node.

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

## Acknowledgements

Design language inspired by [teenage engineering EP–133 K.O. II](https://teenage.engineering/products/ep-133) and [OP–1 field](https://teenage.engineering/products/op-1).  
Built as a personal project for iPad music production.
