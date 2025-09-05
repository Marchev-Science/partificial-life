# Configuration  

## 🔢 Default configuration

Defaults live in a top-level `DEFAULT_CONFIG` (merged with any `localStorage.simConfig` on load). Keys include: `COLORS`, `INITIAL_COUNT`, `MAX_COUNT`, `CVI_WIN`, `CANVAS_SIZE`, `RADIUS`, `MIN_DIST_FACTOR`, `INTERACTION_RANGE_INIT`, `INTERACTION_RANGE_MIN/MAX`, `SPEED`, `DAMP`, `ENTROPY_K`, `METRICS_HISTORY_LENGTH`, `CHART_UPDATE_INTERVAL`, `AUTO_START`, `INTERACTIONS` (5×5).

---

## 💾 Save / Load a JSON configuration

* Click **SaveCfg** to download `sim-config.json` **and** persist to `localStorage`.
* Click **LoadCfg** to select a JSON file; the app updates the UI (matrix spinners, sliders) and reloads to apply fully.

### File schema

When saving, the app exports (at minimum):

```json
{
  "COLORS": [
    { "name": "Red", "key": "red", "h": "#f33" },
    { "name": "Green", "key": "green", "h": "#3f3" },
    { "name": "Blue", "key": "blue", "h": "#39f" },
    { "name": "Yellow", "key": "yellow", "h": "#ff3" },
    { "name": "Pink", "key": "pink", "h": "#f6c" }
  ],
  "INITIAL_COUNT": 100,
  "MAX_COUNT": 300,
  "CVI_WIN": 40,
  "CANVAS_SIZE": 1000,
  "RADIUS": 3,
  "MIN_DIST_FACTOR": 2,
  "INTERACTION_RANGE_INIT": 20,
  "INTERACTION_RANGE_MIN": 5,
  "INTERACTION_RANGE_MAX": 200,
  "SPEED": 0.05,
  "DAMP": 0.95,
  "ENTROPY_K": 3,
  "METRICS_HISTORY_LENGTH": 3000,
  "CHART_UPDATE_INTERVAL": 1000,
  "AUTO_START": false,
  "INTERACTIONS": [
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0]
  ]
}
```

* The app assumes equal per-colour counts when saving (`INITIAL_COUNT` mirrors the first colour slider). You can still edit counts individually after load using the UI sliders.

### Edit tips

* **Interaction strengths:** real numbers `[-1.0, 1.0]`. The diagonal controls same-colour cohesion; off-diagonals control cross-colour interactions.
* **Range & speed:** larger `INTERACTION_RANGE_INIT` increases neighbourhood size; `SPEED` is a global acceleration scale; `DAMP` applies per-tick velocity damping.
* **Safety:** If you load a malformed file, the app alerts “Invalid config.”

---

