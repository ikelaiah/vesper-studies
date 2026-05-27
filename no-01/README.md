# Vesper · No.01 — Light Flow Study

An interactive flow-field particle study in five chapters. Each chapter sets its own palette, density, curl, and vector field, so navigating reshapes the motion on screen.

Part of the [Vesper Studies](../README.md) series.

## Run it

It's a single static file with no dependencies or build step. Open [index.html](./index.html) in a browser.

Or serve the folder locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000/no-01/
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
- **Tail** slider — trail length (0.3×–2.0×). Higher = longer persistence per particle. Persists across chapters; the other two reset to chapter defaults.

## On phones and tablets

The piece is designed for desktop, but it adapts:

- The secondary controls (palette, clear, pause, and the three sliders) collapse into a **Settings** tray that slides up from the bottom on small screens. Only chapter navigation and the toggle stay visible.
- The tray closes automatically when you change chapters, so it never obscures the new scene.
- The charged-burst gesture works the same with touch: press and hold a finger, release to fire. The charge ring grows under your fingertip while you hold.

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

While the LMB is held, particles inside the pointer's reservoir zone have their aging frozen and bounds-respawn disabled, so a long hold genuinely accumulates particles instead of churning through them.

## Roadmap

Planned improvements — see [CHANGELOG.md](../CHANGELOG.md) for shipped changes.

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
