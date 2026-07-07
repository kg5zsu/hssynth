# hssynth — Rhythm → Pitch Demo

A single-file browser demo that makes one of psychoacoustics' most striking phenomena audible and visible: **rhythmic note repetition gradually becomes continuous pitch as the repetition rate increases**.

[Live Demo](https://kg5zsu.github.io/hssynth/) <!-- update if you deploy to Pages -->

---

## What It Demonstrates

When you play a musical note repeatedly and slowly, you hear a discrete rhythm — individual beats separated by silence. As you speed up the repetition rate, something remarkable happens around **20–60 Hz**: the brain stops counting individual events and begins perceiving them as a single, fused tone. The frequency of that tone equals the repetition rate.

This is not a trick or a metaphor. It is the same mechanism that underlies all pitched sound. A vibrating string at 440 Hz is simply an air pressure event repeating 440 times per second. This demo lets you hear the transition from one domain to the other in real time.

### The Three Regimes

| Repetition Rate | Perception | Label |
|---|---|---|
| < 20 Hz | Distinct beats, no fused pitch | ⏱ Rhythm |
| 20 – 60 Hz | Ambiguous, flutter-like | ↗ Transition |
| > 60 Hz | Smooth, fused tone | ♪ Pitch |

### The Math

- **Repetition Rate (Hz)** = BPM × note-value-multiplier / 60
  - e.g. at 120 BPM with 1/8 notes: `120 × 2 / 60 = 4 Hz` (still rhythm)
  - at 3 000 BPM with 1/8 notes: `3000 × 2 / 60 = 100 Hz` (clear pitch)
- **Slot Period** = 60 / BPM / note-value — time between note onsets
- **Note Duration** = Period × Articulation — the sounding portion within each slot
- **Max BPM** is bounded by the Nyquist limit: `rate ≤ sampleRate / 4`. At 44 100 Hz with 1/8 notes that is ~330 000 BPM. Exceeding this creates inharmonic aliasing artifacts rather than a clean pitch.

---

## How It Works: The Signal Model

This demo is a real-time **amplitude modulator (AM)** — the same mathematical structure used in AM radio broadcasting, just shifted from radio frequencies into the audio range.

### The AM Framework

| Component | Symbol | Example | Radio analogy |
|---|---|---|---|
| **Carrier** | `fₙ` | 440 Hz (A4) | The station frequency |
| **Modulator** | `fᵣ` | 2–2000+ Hz | The audio signal being broadcast |
| **Output** | `a(t) = c(t) × m(t)` | Gated sine wave | The transmitted RF waveform |

Where:
- `c(t)` = a continuous sine (or square/saw/triangle) at the note frequency `fₙ`
- `m(t)` = a square-wave envelope at the repetition rate `fᵣ`, switching between 1 (note on) and 0 (silence/rest) according to the articulation setting

This on-off gating is the most extreme form of AM — **100% modulation depth** — which produces the richest possible sideband structure.

### The Three Spectral Families

The spectrum of the output contains three distinct families of peaks:

#### ① Envelope Harmonics (the "rhythm" lines)

The gating envelope `m(t)` is a square wave at `fᵣ`. A square wave contains only odd-numbered harmonics:

```
fᵣ,  3fᵣ,  5fᵣ,  7fᵣ,  …
```

At low BPM these are in the **Rhythm regime** (< 20 Hz) and are heard as individual beats. As `fᵣ` crosses ~20 Hz the harmonics fuse into a buzzing sensation (Transition). Above ~60 Hz (Pitch regime) the **fundamental `fᵣ` itself becomes the perceived pitch**, indistinguishable from a tone generator at that frequency.

This is the core phenomenon the demo demonstrates: **the envelope's repetition rate becomes the perceived fundamental frequency once it exceeds the brain's temporal fusion threshold**.

#### ② Waveform Harmonics (timbre)

The carrier `c(t)` produces its own harmonic series anchored at `fₙ`, determined by the selected waveform:

| Waveform | Harmonics | Spectrum |
|---|---|---|
| **Sine** | Fundamental only | Single peak at `fₙ` |
| **Square** | Odd: 1×, 3×, 5×, 7×… | Peaks at `fₙ`, `3fₙ`, `5fₙ`… |
| **Saw** | All integers: 1×, 2×, 3×… | Peaks at `fₙ`, `2fₙ`, `3fₙ`… |
| **Triangle** | Odd, falling amplitude | Peaks at `fₙ`, `3fₙ`, `5fₙ`… |

These encode **timbre** — they're what distinguish a square-wave note from a sine-wave note at the same pitch.

**Important:** The waveform harmonics are *not* the origin of the rhythm→pitch transition. A sine wave (which has no harmonics) still produces the transition. The harmonics merely color the resulting tone.

#### ③ AM Sidebands (the bridge between rhythm and pitch)

This is where radio communications and psychoacoustics converge. Amplitude modulation of `c(t)` by `m(t)` produces **sum-and-difference sidebands** flanking every harmonic of the carrier:

```
For each waveform harmonic at  n·fₙ:

    fₛ = n·fₙ ± k·fᵣ      for k = 1, 2, 3, …
```

In AM radio terms:
- **Upper sideband (USB)**: `n·fₙ + k·fᵣ`
- **Lower sideband (LSB)**: `n·fₙ - k·fᵣ`

**At low rep rates** (Rhythm regime), `fᵣ` is a few Hz. The sidebands cluster tightly around each waveform harmonic — e.g. for a 440 Hz carrier with a 4 Hz rep rate:

```
Carrier:       440 Hz
USB (k=1):     444 Hz
LSB (k=1):     436 Hz
USB (k=2):     448 Hz
LSB (k=2):     432 Hz
…
```

These clusters are perceived as **roughness or buzz** around the note, not as separate pitches.

**As `fᵣ` rises**, the sidebands spread apart. Around 20–60 Hz (Transition) the sidebands are wide enough that the ear begins to resolve individual frequencies. This is the ambiguous flutter zone.

**Above ~60 Hz** (Pitch regime), the sidebands are so widely spaced that the envelope harmonics `fᵣ, 2fᵣ, 3fᵣ…` form a harmonic series that the brain interprets as a **pitched tone with fundamental `fᵣ`**. The original carrier `fₙ` becomes just another overtone.

### The Match Condition

When `fᵣ = fₙ` (press the **Match** button), every AM sideband lands exactly on a harmonic of the carrier:

```
For n=1, k=1:  fₙ + fᵣ = 2fₙ    ← aligns with 2nd harmonic of carrier
For n=2, k=1:  2fₙ + fᵣ = 3fₙ   ← aligns with 3rd harmonic
...and so on.
```

At this point the envelope harmonics, waveform harmonics, and AM sidebands **all coincide**. The spectrum collapses into a perfectly harmonic series, and the tone sounds maximally "pure." This is the point where the rhythmic structure is most tightly locked to the pitch structure.

### The Inharmonic Regime

When `fᵣ > fₙ`, the 1st upper sideband `fₙ + fᵣ` exceeds `2fₙ`. The spectral spacing is no longer integer — the peaks are **inharmonic**. This produces a clangorous, bell-like quality rather than a clean pitch. This regime is bounded by the Nyquist limit (`fᵣ < sampleRate / 4` to avoid aliasing).

---

## Spectrum & Waveform Visualizations

### Spectrum View

The spectrum pane shows a real-time FFT (8192-point) of the audio output. Annotations include:

- **H1–H5** — colored dashed cursors at the carrier waveform harmonics `n·fₙ`. The info bar shows each harmonic's magnitude as a percentage of full scale.
- **↻ cursor** — dashed line at the repetition rate `fᵣ` (only shown when it differs from `fₙ` by > 10 Hz).
- **▲ peak label** — the single strongest bin in the FFT, regardless of which family it belongs to.
- **Regime zones** — translucent background bands (green = Rhythm < 20 Hz, yellow = Transition 20–60 Hz, red = Pitch > 60 Hz).

The frequency axis spans 0–10 kHz with 1 kHz tick marks.

### Waveform View

The waveform pane shows ~4 cycles of the pre-generated audio buffer, locked (non-scrolling) for a stable reference. At low BPM you see discrete note bursts separated by silence (the articulation gate). At high BPM the bursts merge into a continuous wave, making the transition from amplitude modulation to continuous tone visible.

---

## Real-World Connections

This demo bridges several fields under one roof:

| Field | Connection |
|---|---|
| **Radio communications** | The AM sideband structure is identical to AM broadcasting. The "carrier" is the note; the "modulation" is the gating envelope; the sidebands are the sum/difference frequencies. |
| **Psychoacoustics** | The 20 Hz / 60 Hz regime boundaries map to the temporal resolution limits of the human auditory system — the point where the ear stops tracking individual events and integrates them into a steady-state percept. |
| **Synthesis** | The gating technique used here is identical to **ring modulation** (carrier × modulator) when the modulator is a square wave. It's also related to **pulse-width modulation** and **tremolo** effects. |
| **Physics** | The transition from discrete events to continuous wave mirrors the particle/wave duality concept. Individual air-pressure pulses (particles) at low rates become a sustained tone (wave) at high rates. |

## Interface

### Playback Buttons

| Button | Description |
|---|---|
| **▶ Sweep** | Plays a continuous acceleration from Start BPM to End BPM over the configured sweep duration, then holds at End BPM for 20 seconds. |
| **♩ Normal** | Plays at the Start BPM for 20 seconds — pure rhythm mode. |
| **♩ High Speed** | Plays at the End BPM for 20 seconds — pitch mode. |

Clicking any active button stops playback.

### Knobs (drag up/down; hold Shift for fine control)

| Knob | Default | Description |
|---|---|---|
| **Start BPM** | 60 | Initial BPM for the sweep, and the fixed BPM for Normal mode. |
| **End BPM** | 2 000 | Final BPM for the sweep, and the fixed BPM for High Speed mode. Automatically capped to the Nyquist limit for the selected note value. |
| **Note** | A4 (440 Hz) | The pitch used for every note event. Drag to select any MIDI note (C2–C8). |

### Controls

| Control | Description |
|---|---|
| **Sweep duration** | How long the BPM acceleration lasts (2–30 s). |
| **Note value** | Subdivides each beat. 1/4 = quarter note, 1/8 = eighth note, etc. Finer values raise the repetition rate at the same BPM, so the pitch threshold is reached sooner. |
| **Time signature** | Affects how many note slots are displayed on the staff visual. |
| **Articulation** | The fraction of each slot that is sounding note vs. silence. Legato 100% = continuous; Staccatissimo 25% = short stabs. Shorter articulation makes the pitch regime sound buzzier. |
| **Waveform** | Sine, Square, Saw, or Triangle. The timbre of each note event. |
| **Volume** | Output gain (0–1). |

### Match Button

Sets End BPM to the value that makes the **repetition rate exactly equal the note's pitch frequency** (e.g. 440 Hz for A4). At this BPM the rhythm is literally oscillating at the same frequency as the note being played, which produces a characteristic reinforcement effect.

---

## Visualizations

### Staff View

A treble-clef staff renders the current note pattern for one bar: filled noteheads for sounding events and rest symbols for silent slots. The display updates every ~120 ms during playback and shows:

- Current note name and frequency (top-left of the staff)
- Current time signature, note value, BPM, and sweep progress (top-right status line)

### Info Table

A two-row table (Start / End) updates in real time showing:

- BPM
- Frequency (Hz) — for the End row, this is the **repetition rate**, not the note pitch
- Period between note onsets
- Note duration (= Period × Articulation)
- Regime classification (Rhythm / Transition / Pitch)

### Spectrum View

A real-time 8192-point FFT analyzer (0–10 kHz) with:

- **H1–H5 cursors** — dashed colored lines at the carrier harmonics `n × fₙ`, with per‑harmonic magnitude readout as a percentage of full scale
- **USB/LSB sideband cursors** — blue dashed lines at `fₙ ± n·fᵣ` (n = 1–4), showing the AM sum and difference frequencies created by modulation
- **↻ rep‑rate cursor** — dashed line at `fᵣ` when it differs from `fₙ` by more than 10 Hz
- **▲ peak label** — the single strongest FFT bin, regardless of source
- **Regime zones** — translucent background bands: green (Rhythm < 20 Hz), yellow (Transition 20–60 Hz), red (Pitch > 60 Hz)
- Log‑spaced frequency tick marks at 1 kHz intervals

### Waveform View

A live real‑time oscilloscope showing the current audio output via the AnalyserNode. The display updates at ~20 fps. The sharp on/off edges of the articulation gate are clearly visible at low BPM; at high BPM the bursts merge into a continuous wave.

---

## Audio Engine

All audio is generated in real time using the **Web Audio API** — no samples, no external files. The process:

1. A `ScriptProcessorNode` (256 samples per callback, ~5.8 ms at 44.1 kHz) runs the synthesis algorithm incrementally. The chosen waveform function (sine / square / saw / triangle) is evaluated sample-by-sample in each callback.
2. Each sample belongs to a slot determined by the running beat count. If the slot is "on" (given the articulation setting), the oscillator phase advances and the sample is filled; otherwise it is zero.
3. The synthesizer maintains `virtualTime`, `phase`, and `prevSlot` state between callbacks so the output is sample-accurate across the entire sweep.
4. BPM sweeps linearly: at any time `t` in the sweep, `bpm(t) = startBPM + (endBPM - startBPM) × t / duration`. Beat accumulation uses the integral of this ramp so timing is exact.
5. The output is connected through a `GainNode` (volume control) and an `AnalyserNode` (FFT / waveform analysis) before reaching the destination.

Because audio is generated on‑the‑fly rather than pre‑rendered, **every parameter change takes effect on the very next callback** (~5.8 ms latency). There is no generation delay, no buffer pre‑render spinner, and controls remain live during playback.

---

## Running Locally

No build step required. Open `index.html` directly in any modern browser:

```bash
# Clone the repo
git clone git@github.com:kg5zsu/hssynth.git
cd hssynth

# Open directly — works in Chrome, Firefox, Safari, Edge
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

A local server is not needed because there are no external fetch requests and no ES modules that require CORS headers.

---

## Browser Compatibility

Requires Web Audio API support (all evergreen browsers). The `AudioContext` is created on first user interaction to comply with autoplay policies.

---

## License

MIT
