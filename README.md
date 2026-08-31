# 汉字雨 · Hanzi Rain

Matrix-style digital rain of Chinese characters, in Texas Ringer navy and red,
with the Ringer broadhead mark standing in the middle of the field — drawn in
the same falling characters, not pasted over them. One static HTML file — no
build step, no dependencies, no network requests.

Open `index.html` in a browser, or serve the directory with anything.

## Controls

| Key | |
| --- | --- |
| `Space` or click | Pause / resume |
| `↑` `↓` | Speed up / slow down |
| `R` | Reseed the field |

The hint strip fades out after five seconds and returns on mouse move.

## How it works

Several streams share each column (`STREAMS_PER_COL`), each tracked as a
fractional row position so they fall at their own speed rather than the frame
rate's. Streams sharing a column spawn well above the top at staggered offsets,
so they don't march in lockstep. Every frame the
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

## The mark

The broadhead is a three-color mask (navy / red / white on transparency)
inlined as a ~2.5KB data URI. It is never drawn to the screen. At each resize it
is rendered to an offscreen canvas at **one pixel per character cell**, and each
cell is snapped to the nearest brand color — averaging instead would blend navy
into red at the edges and read as mud. Characters falling through those cells
take the mark's color, and a slice of the mark's cells is repainted every frame
so it outruns the trail decay and stays legible while the rain moves through it.

Rain is held out of a two-cell ring around the silhouette. Without that gap the
mark dissolves into the surrounding field no matter how bright it is.

The wordmark can't be used this way: at ~16px cells a screen is only ~80
characters wide, and the wordmark's letters, outlines and drop shadow merge into
one solid block at that resolution. The broadhead survives because it's a bold
silhouette.

To swap in a different mark, rebuild the `LOGO_MASK` data URI from any image with
a transparent background — flatten it to a few strong colors first.

## Tuning

Everything worth adjusting is near the top of the inline script:

- Colors: the CSS custom properties in `:root` (`--rain`, `--head`, `--void`),
  plus `LOGO_NAVY` / `LOGO_RED` / `LOGO_WHITE` for the mark.
- Mark size and clearance: the `scale` fit in `buildLogoMask()` and its `R` ring
  radius; how solid it holds is the `0.08` share refreshed in `shimmerLogo()`.
- Density: `STREAMS_PER_COL` (how many streams share a column) and `CELL` in
  `resize()` (smaller cells, more columns).
- Speed: the `speed` range in `seedColumn()` (cells per second).
- Tail length: the `0.99` decay base in `frame()` — closer to 1 is longer.
- Character set: the `GLYPHS` string.
