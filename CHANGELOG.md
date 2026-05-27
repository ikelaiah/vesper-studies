# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.2.0] — Unreleased

### Added

- Gallery-style masthead in Cormorant Garamond serif: "Light Flow Study · No.01 — A Field Study", with italic gold-gradient "Flow" and underlined "o" in "No"
- Scramble transition on chapter title and kicker, cross-fade on description
- Tail slider for trail length (0.3×–2.0×); persists across chapters
- Charged-burst LMB interaction: hold to gather, release to fire an outward radial impulse scaled by hold time; charge ring grows under the cursor while holding; restrained palette-tinted flash on release; sub-80ms taps ignored
- Reservoir mechanic: particles inside the pointer radius have their aging and bounds-respawn frozen, so long holds genuinely accumulate particles
- Gruvbox Dark Medium colour scheme for buttons, sliders, dots, and panels
- Collapsible **Settings** tray for screens under 900px wide; chapter navigation stays visible, secondary controls slide up from the bottom on demand and auto-close on chapter change
- Open Graph and description meta tags
- Series-level landing page at root `index.html` listing studies
- `LICENSE` (MIT)

### Changed

- Repository restructured as a series: `vesper-studies` with per-study folders (`no-01/`)
- README rewritten as a series overview; per-study details moved into [no-01/README.md](./no-01/README.md)
- Chapter panel moved from top-left to bottom-left; smaller, single-line title
- Browser tab title changed to "Vesper · No.01 — Light Flow Study"

### Fixed

- Scramble transition cancelling itself mid-flight when multiple elements were scrambling concurrently; each target now tracks its own token via a `WeakMap`
- Long titles no longer wrap to two lines

## [0.1.0]

### Added

- Single-file canvas flow-field "book" in `index.html`
- Five chapters: First Light, The River, The Turning, Bright Weather, Soft Return
- Five palettes: Cosmic, Ocean, Aurora, Fire, Forest
- Controls: chapter nav (buttons, dots, arrow keys), palette cycle, clear, pause (Space), density slider, curlyness slider
- Pointer / touch interaction that pushes particles radially
- Initial `README.md` with run instructions, controls, chapters, and roadmap
- `CHANGELOG.md`
- `.gitignore`
