# Vesper · No.04 — Murmuration Study

An interactive flock of boids in five chapters. Hundreds of birds drift across the sky together, each one watching only its nearest neighbours; from those three simple rules — separation, alignment, cohesion — emerges the breathing shape of a flock. The cursor is a soft attractor when you hover, a predator when you hold, and a meeting place when you tap.

Part of the [Vesper Studies](../README.md) series.

## Run it

It's a single static file with no dependencies or build step. Open [index.html](./index.html) in a browser.

Or serve the folder locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000/no-04/
```

## Chapters

1. **Dusk Flight** — Cosmic, ~300 birds, balanced rules. The canonical murmuration.
2. **The Tight Pack** — Ocean, ~260 birds, high cohesion / low separation. The flock clumps into a moving knot.
3. **The Wary Flock** — Aurora, ~290 birds, high panic radius, jumpy separation. Easily startled, slow to regather.
4. **Predator Garden** — Fire, ~320 birds, strong hover attraction *and* strong hold-repulsion. Birds always swirl around the cursor; holding scatters them violently.
5. **Last Light** — Forest, ~120 birds, slow and sparse. A quiet ending.

## Controls

- **Prev / Next** or **← / →** — change chapter
- **Palette button** — cycle Cosmic / Ocean / Aurora / Fire / Forest
- **Clear** — clear the wake and reseed the flock
- **Pause** or **Space** — pause/resume animation
- **Speed** slider — time multiplier (0.2×–2.2×)
- **Cohesion** slider — multiplies the chapter's cohesion weight (0×–3×). High = the flock tightens; low = it drifts apart into wandering pairs.
- **Tail** slider — wake persistence (0.3×–2.0×). Controls how long the faint shimmer of where the flock has been lingers behind it. The bird bodies and short trails always render fresh each frame; this slider affects only the soft long-memory wake layer.

## On phones and tablets

The piece is designed for desktop, but it adapts:

- The secondary controls (palette, clear, pause, and the three sliders) collapse into a **Settings** tray that slides up from the bottom on small screens.
- The interaction works the same with touch: tap to plant a meeting point, hold-and-drag to scare the flock.

## Mouse interaction: curious, scared, drawn

The cursor speaks to the flock in three voices:

- **Hover (no click)** — the flock is *curious*. Birds gently steer toward the cursor at any distance. Move the cursor around and watch the flock follow at a leisurely pace.
- **Hold LMB** — the flock is *scared*. Within the panic radius birds flee outward with strong force, closer birds harder; release and they relax back into curiosity within a second or two.
- **Tap (quick click, no movement)** — a *meeting point* (a "seed") is dropped at the cursor. For about three seconds it strongly attracts the entire flock; then it fades and dissipates. Use it to gather the birds somewhere specific without dragging.

The canvas **wraps toroidally** — birds that fly off one edge appear on the opposite edge, so the flock never collides with walls and the composition stays continuous.

## How it works

Each bird is a position, a velocity, and a per-bird hue offset stored in flat typed arrays (Float32Array) so the hot loops are cache-friendly.

### The three rules (Reynolds 1986)

For every bird, look at its neighbours within `perceptRadius`. Three steering forces emerge:

1. **Separation** — for any neighbour within `sepRadius` (a smaller radius), push away inversely proportional to distance. Closer = stronger.
2. **Alignment** — average the velocities of all neighbours within `perceptRadius`, then steer toward that average heading.
3. **Cohesion** — average the *positions* of all neighbours, then steer toward that centroid.

Each rule produces a desired velocity; the steering force is `(desired − current_velocity)` capped by `maxForce`. The three forces (weighted by chapter parameters) sum into the bird's acceleration. Velocity is integrated, then clamped to `[minSpeed, maxSpeed]` so birds always look like they're flying — no stationary frame.

### Cursor influence

The cursor adds a fourth force:

- **Hover** — steer toward cursor at `wAttract` weight, scaled by current speed
- **Hold** — within `panicRadius`, steer *away* with `panicStrength × maxSpeed × falloff` where falloff = `1 − d/panicR`
- **Seed** — when a tap is active, steer toward the seed at `(1 − age/ttl) × 2.0`, fading out smoothly over the seed's lifetime

### Spatial hash grid

Naive neighbour search at 300 birds is O(N²) ≈ 90 000 checks per frame. Instead we bin birds into a 2D grid where each cell is `perceptRadius` wide; per-bird search visits only the 3×3 cells around the bird, which contain all possible neighbours within range. This brings the cost to O(N × k) where k is the average occupancy of a cell (typically 5–15), running comfortably at 60 fps with all chapters.

Cells use an intrusive singly-linked list — `gridStart[cell]` points at the first bird, `gridNext[bird]` chains to the next. Rebuilt fresh from current positions every frame; no per-frame allocations.

### Two-canvas architecture

Bird bodies, short per-bird trails, the cursor glow, and the seed pulse all draw to the **overlay canvas** (cleared every frame) — so those elements stay perfectly transient with no smearing. The **trail canvas** carries a single faint per-bird dot each frame (the "wake"), then a fade pass that's tuned to clear the wake within ~1.5 seconds after the flock leaves an area. Steady-state alpha at any pixel ≈ paint / fade ≈ 0.20, so the wake reads as a soft shimmer rather than permanent paint.

Per-bird **short trails** (last 7 positions, ring-buffered) are drawn on the overlay as fading polylines. Because the overlay clears each frame the trail length is fixed by `TRAIL_LEN` rather than by a fade rate — no smearing, no accumulation. Segments that would span a toroidal wrap are skipped so wrapped birds don't draw a line across the whole canvas.

## Roadmap

Planned improvements — see [CHANGELOG.md](../CHANGELOG.md) for shipped changes.

### Experience

- Soft generative audio drone per chapter (wind, distant wings)
- Honor `prefers-reduced-motion` (slower max-speed defaults, calmer panic)
- Predator mode toggle: the cursor *is* a hawk and just being near it scares the flock

### Features

- Shareable state — encode chapter/palette/cohesion in the URL hash
- Export current frame as PNG
