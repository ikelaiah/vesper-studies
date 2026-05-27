# Vesper Flow Book

An interactive flow-field particle "book" rendered on an HTML canvas. Each chapter sets its own palette, density, curl, and vector field, so navigating the book reshapes the motion on screen.

## Run it

It's a single static file with no dependencies or build step. Just open [index.html](index.html) in a browser.

Or serve it locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000
```

## Chapters

1. **First Light** — Cosmic, gentle drift
2. **The River** — Ocean, horizontal ribbons
3. **The Turning** — Aurora, orbital motion
4. **Bright Weather** — Fire, turbulent storm
5. **Soft Return** — Forest, settling inward

## Controls

- **Prev / Next** or **← / →** — change chapter
- **Palette button** — cycle Cosmic / Ocean / Aurora / Fire / Forest
- **Clear** — reset the canvas
- **Pause** or **Space** — pause/resume animation
- **Density** slider — particle count (100–1200)
- **Curlyness** slider — flow-field intensity multiplier

## Mouse interaction: charged burst

The canvas is a two-phase instrument. Pressing the left mouse button **gathers** particles toward the cursor; releasing it **fires a burst** outward from the release point.

- **Press and hold LMB** — particles accelerate toward the cursor. A ring around the pointer grows as you hold, showing the charge building (capped at ~1.6s).
- **Release** — particles within the release radius receive a radial outward impulse, and a brief flash blooms at the release point. Strength and radius scale with how long you held.
- **Drag while holding** — the release point follows your cursor, so you can aim the burst somewhere other than where you started pressing.
- **Tap (very short hold)** — no burst fires; only releases longer than ~80ms count, so accidental clicks don't snap.
- **Touch** — same gesture: press, hold, release.

Think of it like drawing back a bow: the longer the draw, the bigger the snap.

## How it works

Each chapter defines a `flow(x, y, t)` function returning an angle for the vector field at that point and time. Particles accumulate velocity along that angle, with damping, optional center-pull, and a radial push from the pointer. A semi-transparent background fill each frame produces the trailing-streak look.

## Roadmap

Planned improvements — see [CHANGELOG.md](CHANGELOG.md) for shipped changes.

### Experience

- Autoplay between chapters with a slow crossfade, with a pause toggle
- Smooth chapter transitions (interpolate density / curliness / palette over ~1.5s instead of snapping)
- Soft generative audio drone per chapter (Web Audio API, no assets)
- Honor `prefers-reduced-motion` (lower density/force or pause by default)

### Performance

- Replace per-particle `ctx.arc + fill` with an offscreen sprite via `drawImage`, or batch into an `ImageData` buffer
- Explicit 60fps cap on high-refresh displays to save battery

### Code structure

- Split the single-file IIFE into `flow.js` + `chapters.js` + `index.html`
- Move chapters and palettes into a JSON file so non-coders can author chapters

### Features

- Shareable state — encode chapter/palette/density in the URL hash
- Deep-link chapters (`#chapter-2`)
- Export current frame as PNG
