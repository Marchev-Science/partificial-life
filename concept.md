# Particle Life Concept

## 🧠 How it works (high level)

* **World & rendering.** A toroidal canvas (`CANVAS_SIZE×CANVAS_SIZE`), where particles wrap around edges; each particle is drawn as a filled circle (radius `RADIUS`).
* **Forces.** For each pair within the current *interaction range*, apply a force proportional to `strength * (1 - dist / range)`. Positive strengths attract; negatives repel. Velocities are damped by `DAMP` each tick, and positions integrate with wrapping.
* **Separation.** A minimum centre-to-centre spacing (`RADIUS × MIN_DIST_FACTOR`) resolves overlaps by pushing pairs apart.
* **Metrics.** Every `CHART_UPDATE_INTERVAL` ms, the app computes:

  * *Clustering* (same-colour neighbours within 20 px),
  * *Spatial entropy* via k-NN (`ENTROPY_K`),
  * *Mean speed*,
  * *Frame change* (avg per-particle displacement vs previous frame),
  * *CVI* = normalized RMS of per-metric deltas (across the four series above).

> Note: `CVI_WIN` exists in the default config but is not currently used in the calculation loop (reserved for potential rolling-window CVI).


## 📊 Metrics details

* **Clustering:** average count of same-colour neighbours within 20 px (higher → tighter clusters).
* **Spatial Entropy:** 2D k-NN estimator with `ENTROPY_K` (default `3`); higher suggests spatial dispersion.
* **Mean Speed:** average per-particle speed magnitude.
* **Frame Change:** average per-particle displacement vs previous frame (with torus-aware distance).
* **CVI:** normalized RMS of the per-metric *deltas* (cluster, entropy, speed, change) → a quick “volatility” gauge.

All series are trimmed to `METRICS_HISTORY_LENGTH` and redrawn every `CHART_UPDATE_INTERVAL` ms.

[More on metrics...](metrics.md)

---
