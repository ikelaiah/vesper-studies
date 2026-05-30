# Vesper · No.03 — Sand Study

An interactive granular flow in five chapters. Sand falls from the sky; the cursor bends the stream where it falls; holding LMB sculpts an invisible ridge that catches grains; tapping fires a brief upward fountain. The pile that accumulates is the record of everything you have done — a digital zen garden.

Part of the [Vesper Studies](../README.md) series.

## Run it

It's a single static file with no dependencies or build step. Open [index.html](./index.html) in a browser.

Or serve the folder locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000/no-03/
```

## Chapters

1. **First Snow** — Cosmic, slow even rain across the whole canvas, soft gravity
2. **Hourglass** — Ocean, narrow centred stream, fast falling, piles climb steeply
3. **Drift Dunes** — Aurora, steady wind blows the sand sideways; dunes accumulate downwind
4. **Updraft** — Fire, gravity inverts in the upper half — sand thrown up rises, sand at the bottom falls, the boundary churns
5. **The Last Grain** — Forest, the flow slows to almost nothing; one grain at a time

## Controls

- **Prev / Next** or **← / →** — change chapter
- **Palette button** — cycle Cosmic / Ocean / Aurora / Fire / Forest
- **Clear** — empty the pile and reset
- **Pause** or **Space** — pause/resume animation
- **Speed** slider — time multiplier (0.2×–2.2×)
- **Wind** slider — extra horizontal force (−1.0 to +1.0), added to the chapter's baseline wind
- **Tail** slider — trail/pile persistence (0.3×–2.0×). Higher = piles stay visible longer.

## On phones and tablets

The piece is designed for desktop, but it adapts:

- The secondary controls (palette, clear, pause, and the three sliders) collapse into a **Settings** tray that slides up from the bottom on small screens.
- The interaction works the same with touch: tap to fountain, drag to sculpt.

## Mouse interaction: bend, sculpt, fountain

Three things happen depending on what your hand does:

- **Move (no click)** — the stream of falling sand bends toward your cursor like a finger held in pouring water. You can steer the flow to one side of the canvas, then back, then up to the top.
- **Hold and drag** — sculpts the pile. The cursor leaves a trail of sand wherever it goes; you can draw curves, ridges, mountains, letters. The pile remembers everything.
- **Tap (quick click, no movement)** — fires a small upward fountain of grains at the cursor. They rise, fan out, fall, and join the pile. Useful for placing a quick cluster without dragging.

## How it works

Up to 6000 sand grains live in a pre-allocated pool. Each grain has `x, y, vx, vy` and a hue offset. Live grains are integrated forward each frame (semi-implicit Euler) under gravity and wind; dead grains form a free list, allocated on demand by the spawner and recycled when a grain leaves the canvas.

Pile collision uses a 1D **heightmap** — one entry per ~4-pixel column — so each grain's "have I landed?" check is O(1): compare the grain's `y` to `canvas.height − heightMap[col]`. When a grain lands it increments its column's height and dies. An angle-of-repose pass spills excess height into neighbouring columns when the slope gets too steep, so piles round off into dunes instead of growing as vertical spikes.

The cursor's **funnel effect** is implemented at spawn time: each new grain at the top of the canvas gets a small horizontal velocity bias toward the cursor's x. The further off-axis, the stronger the pull (capped by the chapter's `funnelStrength`). The funnel works on hover alone — no click required — so you can steer the stream by waving the mouse around.

**Sculpting** writes directly into the heightmap: while LMB is held and the cursor has moved past the tap threshold, the surface at the cursor's column is raised to meet the cursor's y. A short-lived sand-coloured splat is drawn at the cursor for immediate feedback before the next falling grain reaches the new pile height. The angle-of-repose rule then spreads the ridge sideways over the next few frames.

The piece uses the same **two-canvas architecture** as No.02: trail canvas accumulates the falling-sand streaks and the pile, fading slowly so piles persist; overlay canvas (cleared every frame) carries the cursor glow, ridge marker, and fountain flash so those don't bake into the pile buffer.

## Roadmap

Planned improvements — see [CHANGELOG.md](../CHANGELOG.md) for shipped changes.

### Experience

- Soft generative audio drone per chapter (sand hiss, falling)
- Honor `prefers-reduced-motion` (slower spawn, gentler gravity defaults)
- "Erase" mode — hold a modifier key to lower the heightmap instead of raise it

### Features

- Shareable state — encode chapter/palette in the URL hash
- Export current frame as PNG
