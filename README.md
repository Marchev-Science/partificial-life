# Partificial Life
## Particle Simulation Studying Self-organization (v2.7)

A zero-dependency, browser-only particle system with live controls, a 5×5 interaction matrix, JSON save/load, a realtime metrics dashboard, and a small JavaScript API you can call from the console or an embedding page.

---

## Current implementations

For educational and research purposes https://basaga.org/basaga_files/partificial-life/index.html   

---

## ✨ Features

* **Five “species” of particles** (Red, Green, Blue, Yellow, Pink) you can spawn, remove, and color-code.
* **5×5 interaction matrix** with a slick “number-lock” spinner per cell (−1.0 … +1.0 in 0.1 steps). Scroll, tap, or use arrow keys.
* **Sidebar controls:** Start / Stop / Reset / Randomize, plus sliders for interaction range and per-colour counts.
* **Save / Load configuration as JSON** and auto-persist in `localStorage` under `simConfig`.
* **Metrics dashboard (5 charts):** Clustering, Spatial Entropy, Mean Speed, Frame Change, Composite Volatility Index (CVI).
* **Public API (`window.simAPI`)** for lifecycle, config I/O, metrics, and async “wait until” utilities—plus a built-in debug helper.
* **Test harness (`test.html`)** that embeds the sim in an `<iframe>` and provides one-click API/metrics tests and debug toggles.

---

## 🏁 Quick start

1. Clone or download this repo.
2. Open `index.html` in a modern browser—no build, no server required.
3. (Optional) Open `test.html` for the API test harness.

---

## 🧰 UI & Controls

* **Sliders**

  * **Interaction range:** min `5`, max `200`, default `20`.
  * **Population per colour:** `0 … MAX_COUNT` (default `INITIAL_COUNT` per colour). Adjusting spawns/removes particles immediately.
* **Matrix knobs**

  * Per-cell “number-lock” spinners from `−1.0` to `+1.0` in `0.1` steps; wheel, touch, and keyboard accessible (`role="spinbutton"`).
* **Buttons**

  * `Start`, `Stop`, `Reset`, `Randomize`, `SaveCfg`, `LoadCfg`. `Reset` respawns based on current desired counts and clears all chart series.

---

## 🔌 Embedding tips

* You can embed `index.html` in an `<iframe>` and call `contentWindow.simAPI` from the host (same-origin). This is exactly what `test.html` does.
* Prefer `simAPI.waitUntil(...)` over arbitrarily timed `setTimeout` when you need metrics-based synchronization.

---

## ⚙️ Troubleshooting

* **“simAPI not found”** (in `test.html`): the harness retries until the iframe initializes; give it a second or click **Check API** again.
* **Loaded config doesn’t apply fully:** the file loader updates UI controls, stores JSON to `localStorage`, then reloads the page to ensure a clean boot with the merged config.
* **Charts flatline:** metrics update every `CHART_UPDATE_INTERVAL` and only while **Start** is active. Click **Start** to resume.

---

## Additional documentation:  
* [Overview](README.md)
* [General concept](docs/concept.md)
* [Detailed methododology on metrics](docs/metrics.md)
* [Working with configurations](docs/configs.md)
* [Working with the API](docs/api.md)


---

## 📂 Project structure

```
index.html   # Main simulation app, UI, metrics, and simAPI
test.html    # API test harness (embeds index.html via <iframe>)
sim-config.json   # Example of configuration file   
```

Both files are pure HTML+JS and can be opened directly in a browser.

---

## ✍️ Contributing

PRs that add new metrics, improve the CVI (e.g., make `CVI_WIN` operational), or enhance accessibility are welcome. (This README only documents the current behavior; feel free to extend.)

---

## 📜 [MIT License](LICENSE)

