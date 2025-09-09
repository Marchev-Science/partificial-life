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

## Guide to every config key in your schema  

---

### Core world & visuals

* **CANVAS\_SIZE** — world side length in pixels (square torus); also sets `<canvas>` size.
  *Default:* `1000`. *Try:* **400–2000** (match viewport; larger spreads interactions spatially).&#x20;
* **RADIUS** — particle draw radius (px). Affects the minimum separation via `MIN_DIST_FACTOR`.
  *Default:* `3`. *Try:* **1–6** (bigger looks chunkier; too large + high counts can clutter).&#x20;
* **MIN\_DIST\_FACTOR** — minimum centre distance multiplier: `MIN_DIST = RADIUS × MIN_DIST_FACTOR`. Prevents overlaps.
  *Default:* `2`. *Try:* **1.5–3** (higher = stronger “personal space”).&#x20;
* **COLORS** — array of 5 species `{ name, key, h }` (defines matrix row/col order and swatch hex).
  *Default:* 5 entries (Red…Pink). Keep 5 to match UI. Edit `h` for custom palettes.&#x20;

### Population

* **INITIAL\_COUNT** — starting population per colour; save/export assumes equal counts.
  *Default:* `100`. *Try:* **20–300** per colour (remember the physics is $O(N^2)$).
* **MAX\_COUNT** — per-colour slider max in the UI.
  *Default:* `300`. *Try:* **100–500** (raising it increases peak cost; total max ≈ `5 × MAX_COUNT`).&#x20;
* **AUTO\_START** — if `true`, sim starts running on load.
  *Default:* `false`. Use `true` for kiosk/demo pages.&#x20;

### Interaction radius (neighborhood)

* **INTERACTION\_RANGE\_INIT** — current cutoff distance $R$ for pair forces; linear falloff to zero at $R$.
  *Default:* `20`. *Try:* **10–100** (≈ 1–10% of `CANVAS_SIZE`; too large feels globally coupled).
* **INTERACTION\_RANGE\_MIN / MAX** — UI slider bounds for the above.
  *Defaults:* `5` / `200`. Adjust if you change `CANVAS_SIZE` drastically.&#x20;

### Dynamics (per-step)

* **SPEED** — global acceleration scale applied to the summed forces.
  *Default:* `0.05`. *Try:* **0.01–0.2** (higher = more lively but can overshoot; co-tune with `DAMP`).
* **DAMP** — velocity damping factor per step (multiplicative).
  *Default:* `0.95`. *Try:* **0.90–0.995** (must be $0<\text{DAMP}<1$; closer to 1 = floatier).&#x20;
* **INTERACTIONS** — 5×5 strength matrix $k_{ab}$ in **\[−1.0, +1.0]** (step 0.1 via the number-lock UI). Diagonal = same-colour cohesion; off-diagonals = cross-colour attraction/repulsion. Asymmetry $k_{ab}\ne k_{ba}$ allowed.
  *Default:* all zeros. *Try:* **−0.8…+0.8** for stability headroom.

### Metrics & charts

* **ENTROPY\_K** — $k$ for k-NN spatial entropy estimator.
  *Default:* `3`. *Try:* **3–10** (must be < population; larger smooths but blurs local structure).&#x20;
* **METRICS\_HISTORY\_LENGTH** — max stored points per series (older trimmed).
  *Default:* `3000`. *Try:* **200–5000** (higher = longer sparkline memory; small CPU/RAM impact).&#x20;
* **CHART\_UPDATE\_INTERVAL** — ms between metrics recompute + redraw.
  *Default:* `1000`. *Try:* **200–2000** (faster updates feel more responsive but cost more).
* **CVI\_WIN** — intended rolling-window length for the Composite Volatility Index.
  *Default:* `40`. *Note:* in the provided build CVI is computed “instantaneously” (no explicit window), so this value isn’t referenced by the metric code. You can leave it as is or wire it into a rolling CVI (then **10–200** works well).

---

### Quick tuning tips

* Start by setting **diagonal** `INTERACTIONS` to `+0.2…+0.6` for clustering; use small negative cross-terms for segregation.&#x20;
* If motion jitters: lower **SPEED** or raise **DAMP** slightly.&#x20;
* If everything “locks up”: reduce **MIN\_DIST\_FACTOR** a bit or decrease **INTERACTION\_RANGE\_INIT**.&#x20;

If you want, I can bundle these as inline comments in a sample `sim-config.json` so you’ve got a ready-to-edit preset.



## Read also:  
* [Overview](README.md)
* [General concept](docs/concept.md)
* [Detailed methododology on metrics](docs/metrics.md)
* [Working with configurations](docs/configs.md)
* [Working with the API](docs/api.md)


