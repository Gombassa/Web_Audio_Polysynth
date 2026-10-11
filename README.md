# Web Audio Polysynth

A 10-voice polyphonic synthesizer in a single HTML file, built on the Web Audio API with Web MIDI input. There's no build step and nothing to install. You open the file and play.

```
OSC 1 + OSC 2 + SUB + NOISE → FILTER → AMP → DRIVE → DELAY → REVERB → LIMITER
```

## Running it

Open `index.html` in a recent Chromium, Firefox or Safari browser, then click **Start audio**. Browsers only allow audio to start after a user gesture.

Web MIDI needs a secure context, so it works on `file://` and `localhost` in Chromium, but on other hosts you have to serve the page over HTTPS. To serve it locally:

```sh
python -m http.server 8000
# then open http://localhost:8000
```

## Features

**Sound sources**
- **Osc 1:** sine, saw, square, triangle, or a user-drawn **wavetable**, with octave, fine and level controls.
- **Osc 2:** sine, saw, square, triangle, or **noise** (white or pink), with semitone, fine and level controls. For noise, Semi/Fine change its colour.
- **Sub oscillator:** a sine one octave below.
- **Wavetable editor:** draw 16 steps to make one cycle. It has smooth (cubic) or stepped interpolation, shape presets, Invert, live waveform and harmonic previews, and you can paste any list of numbers (resampled to 16).

**Shaping**
- **Filter:** 12 dB/oct biquad with LP, HP and BP modes, plus cutoff, resonance, envelope amount and key tracking. A live frequency-response plot shows both the base cutoff and the envelope peak.
- **Amp and filter envelopes:** exponential ADSR, each with a shape preview.
- **LFO:** four shapes routed to pitch, filter or amp. The rate runs free or synced to tempo.

**Effects and output**
- Drive: tanh waveshaper with 4× oversampling, using a parallel dry/wet blend.
- Delay: up to 8 s of filtered feedback, free or tempo-synced.
- Reverb: convolution with a synthetic impulse response, plus size and damping controls.
- Master volume into a limiter.

**Performance**
- **Arpeggiator:** 16-step pattern where each step is off, on or accent. Modes are up, down, up/down, random and played order. It spans 1–4 octaves and has rate, gate, swing, step count and latch. Notes are scheduled on the audio clock.
- **Glide:** a continuous slide or a semitone-stepped glissando, applied always or only when legato.
- **Pitch wheel:** on-screen, spring-loaded, with a bend range of ±1–12 semitones. It also responds to MIDI pitch bend.
- **Global tempo:** 10–350 bpm, with tap tempo. It drives the LFO sync, delay sync and the arpeggiator.

**Visuals:** oscilloscope with auto gain, spectrum analyser (20 Hz–20 kHz), and peak, voice-count, sample-rate, latency and MIDI readouts.

**Library**
- 8 factory presets: Init Saw, Sub Foundation, Glass Pluck, Slow Pad, Detuned Lead, Wobble Bass, Drawn Table, Wind Sweep.
- Save and load your own patches and MIDI mappings in browser storage.
- Export and import patches or mappings as JSON through copy and paste.
- **Randomize** builds a new, musically constrained patch.

## Playing

| Input | Action |
|---|---|
| On-screen keys | Click or touch. Where you press on the key sets velocity (lower on the key is louder). Drag to glide across keys. |
| `A W S E D F T G Y H U J K O L P ;` | Play notes by physical key position: the home row gives white keys and the row above gives black keys. |
| `Z` / `X` | Octave down / up |
| MIDI keyboard | Note on/off with velocity, pitch bend, CC 123 (all notes off) |

**Knobs:** drag vertically (hold Shift for fine control), use the mouse wheel while the knob is focused, or use the arrow, PageUp/PageDown and Home/End keys. Double-click a knob to reset it.

## MIDI learn

1. Click **MIDI learn**.
2. Click a knob or switch.
3. Move a control on your controller.

The control's CC number then shows as a badge on the knob. CCs 0–119 can be mapped to any knob or switch; a switch steps through its options across the CC range. The current mapping is saved automatically, and you can save named mappings in the Library.

## Storage

Patches and mappings are kept in `localStorage` under keys prefixed `wapoly.`, so they exist only in that one browser. To keep a copy elsewhere, use **Export** in the Library. If storage is blocked, saves last until you reload.

## Browser support

The synth needs a browser with the Web Audio API, including `ConstantSourceNode` and `PeriodicWave`. MIDI needs Web MIDI, which Chromium-based browsers and Firefox support; Safari doesn't. The rest of the synth works without MIDI.
