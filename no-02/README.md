# Vesper · No.02 — Pendulum Study

An interactive field of phase-coupled pendulum bobs in five chapters. Hundreds of independent pendulums hang from pivots arranged across the canvas; each one is weakly coupled to its grid neighbours so disturbances ripple outward and the idle field forms slow, breathing phase domains. Pluck a handful of bobs with the cursor, drag them aside, and release — a slow release lets them snap home, a flick throws them into swinging motion that propagates through the field via the coupling.

Part of the [Vesper Studies](../README.md) series.

## Run it

It's a single static file with no dependencies or build step. Open [index.html](./index.html) in a browser.

Or serve the folder locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000/no-02/
```

## Chapters

1. **Slow Bloom** — Cosmic, long strings, soft gravity, gentle coupling — disturbances breathe outward as broad phase domains
2. **Lacework** — Ocean, short strings, strong coupling — a single pluck races outward as a travelling phase front
3. **Drift** — Aurora, tilted gravity, faint coupling — each bob carves its own walking oval; domains form and dissolve
4. **Pendulum Wave** — Fire, lengths in arithmetic progression, **zero coupling** — the lengths alone do the work; the field re-coheres periodically
5. **Settling** — Forest, high damping, high coupling — as energy drains, the field locks into synchrony and dies together

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
- The interaction works the same with touch: press and hold to grab nearby bobs, drag, release.

## Mouse interaction: pluck and release

The canvas is a direct-manipulation instrument. You handle individual bobs.

- **Press LMB on the canvas** — the bobs nearest the cursor (up to ~14 of them, within a generous radius) are *grabbed*. A glowing filament connects the cursor to each held bob, and each held bob brightens.
- **Drag while holding** — held bobs follow the cursor's offset from their pivot directly. The string hangs from where you put it, clamped to about 72° of stretch so you can't push a bob past the pivot.
- **Release LMB** — the bobs return to physics. Their angular velocity at the moment of release equals the velocity you imparted with the cursor — a slow release lets them snap home; a quick flick throws them into a wild swing whose energy then propagates through the coupling to their neighbours.
- **Touch** — same gesture: press, drag, release.

Because each held bob carries the cursor's velocity into its swing, the gesture has an unusual range: gentle plucks make quiet, contained ripples; sudden flicks send wave-fronts across the entire field.

## How it works

Each bob is a simple pendulum: a mass on a rigid string of length `L`, hanging from a fixed pivot `(px, py)`. Its angle `θ` from vertical evolves under

```text
θ̈ = -(g / L) · sin θ − damp · θ̇ + K · Σⱼ sin(θⱼ − θ)
```

where the sum runs over the bob's four grid neighbours (up/down/left/right). The last term is the **Kuramoto-style phase coupling** that distinguishes this study from a field of independent pendulums — each bob feels a soft torque pulling its phase toward its neighbours' phases. Integrated each frame with semi-implicit Euler (two substeps for stability).

The bob's screen position is `(px + L · sin θ, py + L · cos θ)`, and a short segment is drawn from its previous position to the new one. A semi-transparent background fill each frame produces the trailing-streak look.

Pivots are laid out on a slightly-jittered grid sized to the canvas. The chapter's `coupling` parameter scales `K` — high in Lacework and Settling so ripples and synchrony are visible, zero in Pendulum Wave so the length progression isn't fought by sync forces. In "Drift" a constant lateral component is added to the effective gravity, so every swing walks sideways.

### Why grid neighbours and not radial neighbours

We couple to the four grid neighbours rather than the four nearest bobs because grid adjacency is fixed at layout time — a single integer-index lookup per bob per frame, no spatial query, no per-frame distance computation. With ~200–300 bobs and 4 neighbours each, the coupling work is ~1000 sin/cos evaluations per frame, well under the integration cost.

### Pluck-and-release physics

When the cursor grabs a bob, we override its `theta` to point at the cursor (clamped to a maximum stretch), and set `omega = d(theta)/dt`. This means the bob carries the cursor's instantaneous angular velocity into the moment of release — flick the cursor, the bob swings hard; ease off, it snaps gently home. Held bobs are skipped in the physics integration entirely; they're driven directly, so they can't fight you.

Each released bob still has its angular velocity capped against the over-spin threshold (~2.4× natural frequency) so a violent flick produces a memorable swing but never permanent rotation. Below the threshold normal damping applies and the coupling reels neighbouring regions into the disturbance.

### Length-keyed colour

Each bob's hue offset is derived from its position in the chapter's length range, not chosen at random — short bobs sit at one end of the palette's hue band, long bobs at the other, with small jitter so neighbours aren't identical. In "Pendulum Wave" this turns the travelling wave into a moving colour gradient. In uniform-length chapters, when coupling synchronises a region, you can see the colour locally lock in step.

### Two-canvas architecture

The page uses two stacked canvases:

- **Trail canvas** — accumulates pendulum trails frame after frame, fading slightly each pass to produce the streak look. Only the trail segments are drawn here.
- **Overlay canvas** — cleared every frame and redrawn from scratch. The strings (pivot-to-bob hairlines), bob heads, pointer glow, held-bob filaments, and release-flash brightening all live here.

This split exists because effects that draw at the same screen position every frame would otherwise accumulate into permanent fills in the trail buffer.

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
