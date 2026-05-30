# Vesper · No.02 — Pendulum Study

An interactive field of pendulum bobs in five chapters. Hundreds of independent pendulums hang from pivots arranged across the canvas; each swings under its own gravity and string length, leaving glowing trails as it moves. Bobs are coloured by their length so the field visibly stratifies — short bobs at one end of the palette's hue range, long bobs at the other. Hold the left mouse button to gather nearby bobs toward your cursor, release to scatter them in a radial impulse, and watch gravity reel everything home.

Part of the [Vesper Studies](../README.md) series.

## Run it

It's a single static file with no dependencies or build step. Open [index.html](./index.html) in a browser.

Or serve the folder locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000/no-02/
```

## Chapters

1. **Slow Bloom** — Cosmic, long strings, soft gravity, lazy unison
2. **Lacework** — Ocean, short strings, fast swings, woven trails
3. **Drift** — Aurora, tilted gravity — the whole field swings off-axis and walks sideways
4. **Pendulum Wave** — Fire, lengths in arithmetic progression so the field aligns, scatters into a travelling wave, gathers, scatters again. The colour gradient moves with the wave.
5. **Settling** — Forest, high damping; each swing is smaller than the last until the field comes to rest

## Controls

- **Prev / Next** or **← / →** — change chapter
- **Palette button** — cycle Cosmic / Ocean / Aurora / Fire / Forest
- **Clear** — reset the canvas and re-seed the bob field
- **Pause** or **Space** — pause/resume animation
- **Speed** slider — internal time multiplier (0.2×–2.2×)
- **Damping** slider — multiplies each chapter's damping (0.2×–2.2×). Higher = bobs come to rest faster.
- **Tail** slider — trail length (0.3×–2.0×). Higher = longer persistence.

## On phones and tablets

The piece is designed for desktop, but it adapts:

- The secondary controls (palette, clear, pause, and the three sliders) collapse into a **Settings** tray that slides up from the bottom on small screens. Only chapter navigation and the toggle stay visible.
- The tray closes automatically when you change chapters.
- The interaction works the same with touch: press and hold, release.

## Mouse interaction: gather and scatter

The canvas is a two-phase instrument.

- **Press and hold LMB** — bobs near your cursor are tugged toward it. The field bunches up. A ring around the pointer grows as you hold, showing the charge building (capped at ~0.9s).
- **Release** — a radial impulse fires from the cursor. Bobs within the burst radius are kicked outward (closer = harder), swinging wildly. Over the next few seconds gravity reels them back to their pivots.
- **Drag while holding** — the gathering point follows the cursor, so you can sweep the field around before releasing.
- **Tap (very short hold)** — no burst fires; only releases longer than ~80ms count, so accidental clicks don't disturb the field.
- **Touch** — same gesture: press, hold, release.

## How it works

Each bob is a simple pendulum: a mass on a rigid string of length `L`, hanging from a fixed pivot `(px, py)`. Its angle `θ` from vertical evolves under

```text
θ̈ = -(g / L) · sin θ − damp · θ̇
```

integrated each frame with semi-implicit Euler (two substeps for stability under high speed). The bob's screen position is `(px + L · sin θ, py + L · cos θ)`, and a short segment is drawn from its previous position to the new one. A semi-transparent background fill each frame produces the trailing-streak look.

Pivots are laid out on a slightly-jittered grid sized to the canvas. Each chapter tunes the grid density, length distribution, gravity, damping, and palette. In "Pendulum Wave" the lengths are assigned in a smooth arithmetic progression across the grid index, which is why the field periodically aligns, scatters into a travelling wave, then re-coheres — the same illusion the classroom physics demo plays with a single row of pendulums, here spread across two dimensions. "Drift" adds a constant lateral component to the effective gravity, so every swing walks sideways.

### Length-keyed colour

Each bob's hue offset is derived from its position in the chapter's length range, not chosen at random — short bobs sit at one end of the palette's hue band, long bobs at the other, with small jitter so neighbours aren't identical. In "Pendulum Wave" this turns the travelling wave into a moving colour gradient. In uniform-length chapters it gives the field a quiet stratification.

### Over-spin brake and impulse cap

A strong burst could in principle flip a bob into permanent rotation. Two mechanisms prevent that:

1. **Over-spin brake**: when `|ω|` exceeds ~2.8× the bob's natural swing speed `√(g/L)`, an extra cubic damping term bleeds energy quickly. Below that threshold normal damping rules and the chapter character is preserved; above it, energy drains in well under a second.
2. **Capped release impulse**: each bob's burst kick is limited to 1.8× natural speed at full charge × point-blank range, comfortably below the brake threshold. A max-charge burst gives a violent swing (maybe a single loop) but never sustained spin.

Together: bursts feel powerful, the field always reels back into pendulum motion, and the chapter's distinct character reasserts within a couple of seconds.

### Two-canvas architecture

The page uses two stacked canvases:

- **Trail canvas** — accumulates pendulum trails frame after frame, fading slightly each pass to produce the streak look. Only the trail segments are drawn here.
- **Overlay canvas** — cleared every frame and redrawn from scratch. The strings (pivot-to-bob hairlines), bob heads, pointer glow, charge ring, and burst flash all live here.

This split exists because effects that draw at the same screen position every frame (strings anchored at fixed pivots; burst flashes centered on the click point) would otherwise accumulate into permanent triangular fills or halo rings in the trail buffer. Putting them on a cleared overlay keeps the trails clean and lets transient effects actually be transient.

### String rendering

Strings are drawn only from 40% of the way down to the bob (not from the pivot itself) and are suppressed entirely when the bob is near rest (`|θ| < 0.08 rad ≈ 4.6°`). This keeps the resting state quiet — no harsh radial lines anchoring every grid point — while making the field visibly *come alive* during interaction as bobs swing harder and their strings appear.

## Roadmap

Planned improvements — see [CHANGELOG.md](../CHANGELOG.md) for shipped changes.

### Experience

- Soft generative audio drone per chapter (Web Audio API, no assets)
- Honor `prefers-reduced-motion` (slower speed and damping defaults)

### Features

- Shareable state — encode chapter/palette/speed/damping in the URL hash
- Export current frame as PNG
