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

### Custom Pattern

Check **Custom pattern** and enter a comma-separated sequence of `1` (note on) and `0` (rest), e.g. `1,0,1,1,0,1`. The pattern loops across all slots. This lets you hear how rhythmic accents are preserved — or smeared — as the BPM climbs.

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

---

## Audio Engine

All audio is generated entirely in JavaScript using the **Web Audio API** — no samples, no external files. The process:

1. `genBuf()` pre-renders the entire sweep into a `Float32Array` at the browser's native sample rate (typically 44 100 Hz).
2. The chosen waveform function (sine / square / saw / triangle) is evaluated sample-by-sample.
3. Each sample belongs to a slot determined by the running beat count. If the slot is "on" (given the pattern and articulation), the oscillator phase advances and the sample is filled; otherwise it is zero.
4. BPM sweeps linearly: at any time `t` in the sweep, `bpm(t) = startBPM + (endBPM - startBPM) × t / duration`. Beat accumulation uses the integral of this ramp so timing is exact.
5. The buffer is loaded into a `BufferSource` node and played back.

Rendering happens asynchronously (behind a "Generating audio buffer…" overlay) so the UI stays responsive.

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
