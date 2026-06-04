# ✨ Vesper Studies

[![License: MIT](https://img.shields.io/badge/License-MIT-fabd2f.svg?style=flat-square)](./LICENSE)
[![Live on GitHub Pages](https://img.shields.io/badge/live-github%20pages-fe8019?style=flat-square&logo=github)](https://ikelaiah.github.io/vesper-studies/)
[![No.01](https://img.shields.io/badge/N%C2%BA-01-d5c4a1?style=flat-square)](./no-01/)
[![No build step](https://img.shields.io/badge/build-none-8ec07c?style=flat-square)](#run-it-locally)
[![Made with Canvas](https://img.shields.io/badge/canvas-2D-665c54?style=flat-square)](./no-01/README.md#how-it-works)

A series of small interactive generative pieces, each self-contained, each numbered. No build step, no dependencies — every study is a single static HTML page that you can open in a browser or host on GitHub Pages.

## 🎨 Studies

| №                 | Title                                                                                         | Status      |
| ----------------- | --------------------------------------------------------------------------------------------- | ----------- |
| [No.01](./no-01/) | **Light Flow Study** — five chapters of flow-field motion with a charged-burst gesture        | Released    |
| [No.02](./no-02/) | **Pendulum Study** — five chapters of a phase-coupled pendulum field; pluck bobs and release  | Released    |
| [No.03](./no-03/) | **Sand Study** — five chapters of granular flow with a bend / sculpt / fountain gesture       | Released    |
| [No.04](./no-04/) | **Murmuration Study** — five chapters of a boid flock with circling hawks; shelter or hunt    | Released    |
| [No.05](./no-05/) | **Fluid Study** — five chapters of ink in water; paint, mix, settle; the canvas remembers     | Released    |

Each study lives in its own folder with its own `README.md` and `index.html`. The root [index.html](./index.html) acts as a series foyer linking to each piece.

## Run it locally

Open [index.html](./index.html) in a browser. To navigate between studies via relative links, you can serve the folder over HTTP:

```sh
python -m http.server 8000
# then visit http://localhost:8000
```

## 🌐 Live

Hosted on GitHub Pages — `https://ikelaiah.github.io/vesper-studies/`

## 📜 License

[MIT](./LICENSE). Studies are released for viewing, learning, and remixing.
