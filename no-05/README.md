# Vesper · No.05 — Fluid Study

An interactive ink-in-water piece in five chapters. Drag the cursor to paint glowing ink into the water; an invisible curl-noise current carries it into wisps and tendrils. Each new stroke takes the next colour in a slow rainbow, so a few minutes of painting layers blue, magenta, gold, and aqua over one another. Tap for a coloured puff.

This is the closing piece of the [Vesper Studies](../README.md) series.

## Run it

It's a single static file with no dependencies or build step. Open [index.html](./index.html) in a browser.

Or serve the folder locally:

```sh
python -m http.server 8000
# then visit http://localhost:8000/no-05/
```

## Chapters

1. **First Ink** — Cosmic, balanced curl-noise. The canonical chapter for learning the gesture.
2. **Bloom** — Ocean, each stroke spreads outward as it ages, like a flower opening in slow water.
3. **Vortex** — Aurora, tight high-frequency swirls. Strokes braid around invisible eddies.
4. **Plume** — Fire, buoyancy added. Whatever you paint rises in slow columns of warm colour.
5. **Dissolve** — Forest, short-lived particles, rapid fade. Wisps appear and vanish in the same breath.

## Controls

- **Prev / Next** or **← / →** — change chapter
- **Palette button** — cycle Cosmic / Ocean / Aurora / Fire / Forest
- **Clear** — wipe the canvas and remove all live ink
- **Pause** or **Space** — pause/resume
- **Speed** slider — time multiplier (0.2×–2.2×); affects how fast the curl-noise field evolves and how fast particles age
- **Flow** slider — multiplies how strongly the curl-noise field carries the ink (0×–3×). Low = strokes hold their shape longer; high = ink immediately swept into turbulence.
- **Glow** slider — multiplies particle brightness (0.2×–3×)

## On phones and tablets

The piece is designed for desktop, but it adapts:

- The secondary controls collapse into a **Settings** tray that slides up from the bottom on small screens.
- The interaction works the same with touch: drag a finger to paint, tap for a puff.

## Mouse interaction: paint, puff, watch

- **Hover (no click)** — a soft cursor glow follows your pointer in the current "next stroke" colour. No ink is added.
- **Hold and drag** — paints ink along the cursor path. Each stroke is colour-coherent (locked to the cycling hue at the moment you press); the next stroke will be a different colour.
- **Tap (quick click, no movement)** — emits a radial burst of ink particles outward at the cursor.

The colour cycle is per-chapter and tied to wall-clock time, so two strokes a few seconds apart will be different colours. Paint several quick strokes in succession and the canvas builds up a multi-coloured ink-cloud where the strokes mingle.

## How it works

### Particles in a curl-noise field

Up to 5000 particles live in parallel typed arrays (position, velocity, hue, saturation, lightness, lifetime). Each frame, every live particle:

1. Samples the **curl-noise field** at its current position — a divergence-free vector field derived from 2D gradient noise via `(∂N/∂y, −∂N/∂x)`. Divergence-free means particles never compress, which gives the visual feel of incompressible fluid (wisps, swirls, vortices) at a fraction of a real fluid solver's cost.
2. Blends the noise-field velocity into its own velocity, weighted by age. Fresh particles still carry the brush stroke's punch; older particles surrender to the field.
3. Integrates position via semi-implicit Euler.
4. Splats its colour additively into the canvas image buffer with a Gaussian sprite. Overlapping particles brighten naturally — additive blending of soft blobs *is* bloom, no separate post-processing needed.

The noise field itself slowly evolves over time (we offset the noise sample with `renderClock × noiseSpeed`), so even with no input the existing ink continues to drift and reshape.

### Rendering

The canvas is rendered each frame from a single `Uint8ClampedArray` (RGBA bytes, sized to the canvas). Each particle's splat writes additively (saturating) into this buffer, then a single `putImageData` puts the whole buffer on the canvas. No per-particle canvas calls — just memory writes — so 5000 particles render in well under a millisecond on commodity hardware.

A multiplicative fade is applied to the whole image buffer at the start of each frame (`buf[i] *= chapter.fadeAlpha`), producing the trail/decay look. This is one pass over the buffer, also fast.

### Stroke colour

A wall-clock-driven cycle continuously walks through the chapter's palette hue range over `colorCycleSec` seconds. When the user presses LMB, the current hue is *locked* so a single drag is colour-coherent even as the cycle continues. Release unlocks. The next press picks up wherever the cycle is now, so consecutive strokes are reliably different colours — what gives the piece its multi-coloured ink-cloud character.

### Two-canvas architecture

The main canvas carries the additive ink (re-rendered every frame from the particle pool, with the multiplicative fade for trails). The overlay canvas (cleared every frame) carries only the cursor glow in the current stroke colour, so it stays perfectly transient.

## Why not real Stable Fluids?

The "ink in water" reference look — high-frequency turbulent tendrils with rainbow colours bleeding into one another — is most commonly produced by WebGL fragment-shader fluid simulations (Jos Stam's *Stable Fluids* on the GPU). Implementing that on Canvas 2D would not reach the same visual quality: shader-based fluids run at full canvas resolution (millions of cells), while Canvas 2D taps out at maybe 10 000 cells before stuttering.

Curl-noise particles cheat to within a hair of the same look at a tenth of the cost. The eye can't easily distinguish "real divergence-free flow" from "noise field that happens to be divergence-free", and additive splats give the volumetric glow that bloom shaders would otherwise have to add as a post-pass. The tradeoff is that the simulation is not physically conservative — ink doesn't push other ink, it just rides the field — but for a piece you *paint with*, the difference is invisible.

## Roadmap

Planned improvements — see [CHANGELOG.md](../CHANGELOG.md) for shipped changes.

### Experience

- Soft generative audio drone per chapter
- Honor `prefers-reduced-motion` (smaller particle counts, slower noise)
- Multi-touch (each finger paints an independent stroke with its own colour)

### Features

- Shareable state — encode chapter/palette in the URL hash
- Export current frame as PNG
