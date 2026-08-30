# 汉字雨 · Hanzi Rain

Matrix-style digital rain of Chinese characters, in crimson. One static HTML
file — no build step, no dependencies, no network requests.

Open `index.html` in a browser, or serve the directory with anything.

## Controls

| Key | |
| --- | --- |
| `Space` or click | Pause / resume |
| `↑` `↓` | Speed up / slow down |
| `R` | Reseed the field |

The hint strip fades out after five seconds and returns on mouse move.

## How it works

Each column owns one falling stream, tracked as a fractional row position so
streams fall at their own speed rather than the frame rate's. Every frame the
whole canvas is washed with a low-alpha rectangle of the background color; that
decay *is* the trail. The leading glyph is drawn near-white with a shadow glow,
and the previous head is repainted as ordinary rain the moment the stream
advances.

Three details that are easy to get wrong:

- **Trails are frame-rate independent.** The wash alpha is derived from elapsed
  time (`1 - 0.985^(dt*60)`), so a 120Hz display doesn't get half-length trails.
- **The wash is applied in steps of at least 5% alpha.** Below that, subtracting
  from an 8-bit channel rounds to zero and the dimmest glyphs stay on screen
  forever as a red haze.
- **Trail length is speed x fade time.** Changing one without the other either
  stubs the streams or stretches them off the screen.

`prefers-reduced-motion: reduce` gets a single static field instead of the
animation.

## Tuning

Everything worth adjusting is near the top of the inline script:

- Colors: the CSS custom properties in `:root` (`--rain`, `--head`, `--void`).
- Density: `CELL` in `resize()` — smaller cells, more columns.
- Speed: the `speed` range in `seedColumn()` (cells per second).
- Tail length: the `0.985` decay base in `frame()` — closer to 1 is longer.
- Character set: the `GLYPHS` string.
