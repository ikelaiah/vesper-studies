# Vesper · No.04 — Murmuration Study

An interactive flock of boids *with predators* in five chapters. Hundreds of birds drift across the sky, each one watching only its nearest neighbours; from those three simple rules — separation, alignment, cohesion — emerges the breathing shape of a flock. But the flock isn't alone. Hawks circle above, attracted to dense bird regions, diving on stragglers. The cursor is a shepherd — hover to coax, hold to shelter the flock from the hawks — except in one chapter where the cursor is the falconer's lure, and *you* drive the attack.

Part of the [Vesper Studies](../README.md) series.

## Run it

It's a single static file with no dependencies or build step. Open [index.html](./index.html) in a browser.

Or serve the folder locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000/no-04/
```

## Chapters

1. **Dusk Flight** — Cosmic, ~300 birds, 1 hawk. The canonical murmuration with one circling predator. Hold to shelter.
2. **The Tight Pack** — Ocean, ~260 birds, 2 hawks pressing the knot from opposite sides. Hold to shelter.
3. **The Wary Flock** — Aurora, ~290 birds, 3 hawks. Easily startled, slow to regather, every shadow is a threat.
4. **Falconer's Hand** — Fire, ~320 birds, 4 hawks. **Holding pulls the nearest hawk toward your cursor like a lure** — you drive the attack. Tap to drop bait the flock gathers around.
5. **Last Light** — Forest, ~120 birds, no hawks. The predators are gone too. A quiet ending.

## Controls

- **Prev / Next** or **← / →** — change chapter
- **Palette button** — cycle Cosmic / Ocean / Aurora / Fire / Forest
- **Clear** — clear the wake and reseed the flock (and hawks)
- **Pause** or **Space** — pause/resume animation
- **Speed** slider — time multiplier (0.2×–2.2×)
- **Cohesion** slider — multiplies the chapter's cohesion weight (0×–3×). High = the flock tightens; low = it drifts apart into wandering pairs.
- **Tail** slider — wake persistence (0.3×–2.0×).

## On phones and tablets

The piece is designed for desktop, but it adapts:

- The secondary controls collapse into a **Settings** tray that slides up from the bottom on small screens.
- The interaction works the same with touch: tap to plant a meeting point, hold-and-drag to shelter (or lure, in Chapter 3).

## Mouse interaction: shepherd, lure, meeting point

The cursor speaks to the flock and the hawks at the same time, in three modes:

- **Hover (no click)** — the flock is *curious*. Birds gently steer toward the cursor at any distance.
- **Hold LMB — shelter (protect mode, default)** — a dashed pale ring appears around the cursor. Hawks within ~180 px are pushed away strongly. Birds inside the ring panic far less from any nearby hawk, so the cursor reads as a shelter the flock fights to stay inside. Release and the hawks return.
- **Hold LMB — lure (hunt mode, "Falconer's Hand" chapter only)** — the dashed ring takes on the chapter palette colour. The nearest hawk is pulled toward your cursor like prey on a string. Drag it across the flock and you drive the attack.
- **Tap (quick click, no movement)** — a *meeting point* (a "seed") is dropped at the cursor. For about three seconds it strongly attracts the entire flock; then it fades. Use it as bait, or to gather the birds somewhere specific.

The canvas **wraps toroidally** — birds and hawks that fly off one edge appear on the opposite edge, so the composition stays continuous.

## How it works

### The flock (Reynolds 1986)

Each bird is a position, a velocity, and a per-bird hue offset stored in flat typed arrays (Float32Array) so the hot loops are cache-friendly.

For every bird, look at its neighbours within `perceptRadius`. Three steering forces emerge:

1. **Separation** — for any neighbour within `sepRadius`, push away inversely proportional to distance.
2. **Alignment** — average the velocities of all neighbours within `perceptRadius`, then steer toward that average heading.
3. **Cohesion** — average the *positions* of all neighbours, then steer toward that centroid.

Each rule produces a desired velocity; the steering force is `(desired − current_velocity)` capped by `maxForce`. The three forces sum into the bird's acceleration. Velocity is integrated, then clamped to `[minSpeed, maxSpeed]` so birds always look like they're flying.

### The hawks (the second species)

Hawks are a tiny second flock — between zero and four per chapter — with their own physics: faster max speed (~220 px/s vs ~130), much higher steering force, no schooling rules. Each hawk steers toward the nearest bird (found by sampling a 5×5 region of the same spatial hash grid the birds use). When a hawk gets within ~12 px of a bird, the bird is taken — removed from the flock with a brief expanding ring of "feathers" in the bird's last colour.

Birds add a fourth force: a strong outward panic from any hawk within ~130 px. Hawk-panic is ~2× stronger than cursor-panic — a hawk is the most dangerous thing in a bird's world.

To keep the canvas alive over long sessions, taken birds are gradually replenished: with a small per-frame probability, a new bird appears at a random edge of the canvas, headed inward. The flock equilibrium is replenishment-rate ≈ predation-rate, so the population stabilises rather than collapsing.

### Cursor influence

- **Hover** — steer birds toward cursor at `wAttract` weight
- **Hold (protect)** — birds within `panicRadius` of the cursor get ¼-strength hawk-fear (sheltered); hawks within ~180 px of the cursor are pushed away
- **Hold (hunt)** — the nearest hawk is pulled toward the cursor at high weight (a "lure")
- **Seed** — tap drops a temporary attractor; the flock gathers around it for ~3 s

### Spatial hash grid

Naive neighbour search at 300 birds is O(N²) ≈ 90 000 checks per frame. Instead we bin birds into a 2D grid where each cell is `perceptRadius` wide; per-bird search visits only the 3×3 cells around the bird, which contain all possible neighbours within range. Cost: O(N × k) where k is the average cell occupancy (typically 5–15), comfortably 60 fps at all chapters. The hawks reuse the same grid, sweeping a 5×5 cell region because they see farther.

Cells use an intrusive singly-linked list — `gridStart[cell]` points at the first bird, `gridNext[bird]` chains to the next. Rebuilt fresh each frame.

### Two-canvas architecture

Bird bodies, short per-bird wing-tip trails, hawks and their trails, the cursor glow, shelter/lure ring, and seed pulse all draw to the **overlay canvas** (cleared every frame). The **trail canvas** carries a single faint per-bird dot each frame (the "wake"), then a fade pass tuned to clear the wake within ~1.5 seconds.

Per-bird **wing-tip trails** (two trails per bird, last ~20 positions each, ring-buffered) are drawn on the overlay as fading polylines. The draws are batched into a small grid of (alpha-band × hue-bucket) Path2D objects so the whole flock strokes in ~40 calls rather than ~12 000.

## Roadmap

Planned improvements — see [CHANGELOG.md](../CHANGELOG.md) for shipped changes.

### Experience

- Soft generative audio drone per chapter (wind, distant wings, hawk cry)
- Honor `prefers-reduced-motion` (slower max-speed defaults, calmer panic)

### Features

- Shareable state — encode chapter/palette/cohesion in the URL hash
- Export current frame as PNG
