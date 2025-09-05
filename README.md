# Partificial Life
## Particle Simulation — 5-Colour · Metrics Dashboard (CVI v2.7)

A zero-dependency, browser-only particle system with live controls, a 5×5 interaction matrix, JSON save/load, a realtime metrics dashboard, and a small JavaScript API you can call from the console or an embedding page.&#x20;

---

## ✨ Features

* **Five “species” of particles** (Red, Green, Blue, Yellow, Pink) you can spawn, remove, and color-code.&#x20;
* **5×5 interaction matrix** with a slick “number-lock” spinner per cell (−1.0 … +1.0 in 0.1 steps). Scroll, tap, or use arrow keys.&#x20;
* **Sidebar controls:** Start / Stop / Reset / Randomize, plus sliders for interaction range and per-colour counts.&#x20;
* **Save / Load configuration as JSON** and auto-persist in `localStorage` under `simConfig`.&#x20;
* **Metrics dashboard (5 charts):** Clustering, Spatial Entropy, Mean Speed, Frame Change, Composite Volatility Index (CVI).&#x20;
* **Public API (`window.simAPI`)** for lifecycle, config I/O, metrics, and async “wait until” utilities—plus a built-in debug helper.&#x20;
* **Test harness (`test.html`)** that embeds the sim in an `<iframe>` and provides one-click API/metrics tests and debug toggles.&#x20;

---

## 🏁 Quick start

1. Clone or download this repo.
2. Open `index.html` in a modern browser—no build, no server required.&#x20;
3. (Optional) Open `test.html` for the API test harness.&#x20;

---

## 🧠 How it works (high level)

* **World & rendering.** A toroidal canvas (`CANVAS_SIZE×CANVAS_SIZE`), where particles wrap around edges; each particle is drawn as a filled circle (radius `RADIUS`).&#x20;
* **Forces.** For each pair within the current *interaction range*, apply a force proportional to `strength * (1 - dist / range)`. Positive strengths attract; negatives repel. Velocities are damped by `DAMP` each tick, and positions integrate with wrapping.&#x20;
* **Separation.** A minimum centre-to-centre spacing (`RADIUS × MIN_DIST_FACTOR`) resolves overlaps by pushing pairs apart.&#x20;
* **Metrics.** Every `CHART_UPDATE_INTERVAL` ms, the app computes:

  * *Clustering* (same-colour neighbours within 20 px),
  * *Spatial entropy* via k-NN (`ENTROPY_K`),
  * *Mean speed*,
  * *Frame change* (avg per-particle displacement vs previous frame),
  * *CVI* = normalized RMS of per-metric deltas (across the four series above).&#x20;

> Note: `CVI_WIN` exists in the default config but is not currently used in the calculation loop (reserved for potential rolling-window CVI).&#x20;

---

## 🧰 UI & Controls

* **Sliders**

  * **Interaction range:** min `5`, max `200`, default `20`.&#x20;
  * **Population per colour:** `0 … MAX_COUNT` (default `INITIAL_COUNT` per colour). Adjusting spawns/removes particles immediately.&#x20;
* **Matrix knobs**

  * Per-cell “number-lock” spinners from `−1.0` to `+1.0` in `0.1` steps; wheel, touch, and keyboard accessible (`role="spinbutton"`).&#x20;
* **Buttons**

  * `Start`, `Stop`, `Reset`, `Randomize`, `SaveCfg`, `LoadCfg`. `Reset` respawns based on current desired counts and clears all chart series.&#x20;

---

## 📂 Project structure

```
index.html   # Main simulation app, UI, metrics, and simAPI
test.html    # API test harness (embeds index.html via <iframe>)
```

Both files are pure HTML+JS and can be opened directly in a browser.

---

## ✍️ Contributing

PRs that add new metrics, improve the CVI (e.g., make `CVI_WIN` operational), or enhance accessibility are welcome. (This README only documents the current behavior; feel free to extend.)&#x20;

---

## 📜 MIT License

