# CLAUDE.md

Guidance for working on this repository.

## Project shape

- The entire app is `index.html`: inline `<style>`, a small static DOM skeleton, and one `<script>` IIFE in strict mode. There is no build, no bundler, no dependencies, and no tests.
- The file is a fragment. It has no `<!doctype>`, `<html>`, `<head>` or `<body>` tags and starts directly with `<title>`. Keep it that way unless asked: it is likely also used as a hosted artifact page, where the host wraps it in a document skeleton.
- The only external resource is Google Fonts (Barlow, Barlow Condensed, IBM Plex Mono). Don't add CDNs or libraries without asking.
- To run it, open the file in a browser, or serve the folder (`python -m http.server`) when testing Web MIDI over http.

## Code style

- Plain ES2015+ written in a compact, dense style. Helpers are `$` (querySelector), `el` (create HTML element), `sv` (create SVG element) and `clamp`. Match the existing terse naming (`P`, `DEF`, `ctx`, `v.o1`, `v.g2n`) and the sparse, short `/* ... */` comments.
- The UI is built from data in JS, not written as HTML. The only static markup is the header, the scope band, the readouts and the keyboard dock.
- Use typographic characters in user-facing text, as the file already does: `−` (U+2212) for minus, `–` for ranges, `“ ”` for quotes, and `×`.

## Architecture (in file order)

1. **Parameters:** `DEFS` lists the continuous params (`id, name, min, max, def, step?, curve?: 'exp'|'sq', fmt, bi?`). `SEL` lists the discrete params (`opts`, `labels` or `kind:'wave'`, `def`). `P` is the single live state object; `DEFAULTS` is derived from both. `P.table` (16 floats) and `P.arpSteps` (16 values of 0/1/2) are array params.
2. **Value mapping:** `toNorm` and `fromNorm` convert between a param value and 0..1, respecting `curve` and `step`. Knobs, MIDI CCs and Randomize all go through these.
3. **`MODULES`:** the declarative UI layout. Each entry gives a title, a hue class (`src`/`flt`/`mod`/`fx`), `sel` pairs, `knobs` ids, and optional `viz`, `wide`, `arp`, `perf` or `lib` flags. `buildModules()` renders it.
4. **Audio graph:** `build()` creates the shared nodes once: noise buffers, drive, limiter, analyser, delay, reverb, the LFO and the bend `ConstantSource`. Each note creates a voice in `noteOn()` with its own oscillators, filter, and `amp → rel → kill → trem` gain chain. Separate gain stages handle release and voice stealing, so ramps never have to be cancelled.
5. **Parameter dispatch:** `onParam(id)` sends a change to the right `update*()` function or redraw. Any code that changes `P` must call `onParam(id)`, repaint the control (`knobs[id].paint()` or `refreshSeg(id)`), and call `userEdited()` to mark the preset as "Edited".
6. **Presets and patches:** `PRESETS` holds partial overrides on top of `DEFAULTS`. Keys in `ARP_KEYS` are preserved when a preset or patch doesn't specify them. All imported or stored data passes through `cleanParams()` and `cleanMap()`, which validate it.
7. **Input:** pointer, computer keyboard (`KEYMAP` uses physical `e.code`) and MIDI all go through `playNote()` and `stopNote()`, which hand notes to the arpeggiator when it is on.
8. **Arpeggiator:** look-ahead scheduling. A `setInterval` runs every 25 ms and schedules 120 ms ahead on `ctx.currentTime`. The playhead UI is driven from `arp.queue` in the rAF loop.
9. **Rendering loop:** `tick()` runs on rAF, drawing the scope and spectrum to canvas and updating readouts every 6th frame. Canvas colours are read from CSS custom properties through `tokens()`.

## Adding a parameter

1. Add an entry to `DEFS` (or `SEL`).
2. Put its id in a module in `MODULES` (or build it by hand, as `buildArp` does).
3. Handle it in `onParam()`, and apply it in `noteOn()` and/or `updateVoices()` or the relevant `update*()`.
4. If it shouldn't be randomized, exclude it in `randomize()`.
5. Patch export, import and MIDI learn pick it up automatically through `DEFS` and `SEL`. If it belongs with the arp or performance settings, add it to `ARP_KEYS`.

## Storage and compatibility

- `localStorage` keys are `wapoly.patches`, `wapoly.maps` and `wapoly.map.current`. Always use the `store` wrapper, which falls back to memory when storage is blocked.
- Patch JSON is `{ app: 'web-audio-polysynth', type: 'patch', v: 1, name, params }`; mappings use `type: 'mapping', map: { cc: paramId }`. Keep old patches loading: `cleanParams` already migrates legacy `arpDiv` strings and `arpBpm`.

## Theming and accessibility

- Colours are tokens on `:root`, with light-mode overrides under both `prefers-color-scheme: light` and `[data-theme="light"]`. Update both blocks together. Don't hard-code colours in JS; read them through `tokens()`.
- Knobs, the wheel and the table columns are `role="slider"` elements with keyboard support. Segmented switches are `radiogroup` elements. Keep ARIA values updated when changing these controls.
- The layout must work at phone width: there's a 16px gutter, and the keyboard scrolls horizontally inside `.keys-scroll`.
