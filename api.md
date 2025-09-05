## 🧪 API

The simulation exposes a small API on `window.simAPI` once `index.html` finishes initialising. You can call it from the browser console or from a parent page (see `test.html`).

### Surface

```js
// Version
simAPI.version // "1.0.0"

// Config I/O
simAPI.loadConfig(cfgObject)
simAPI.exportConfig()

// Lifecycle
simAPI.start()
simAPI.stop()
simAPI.reset()

// Metrics (latest values + step counter)
simAPI.getMetrics() // { cluster, entropy, speed, change, cvi, step }

// Await-until utility (Promise)
simAPI.waitUntil(predicateFn, { timeoutMs=60000, checkInterval=250 })
```

* `loadConfig` accepts an object with any of the config keys listed above. It updates sliders/matrix UI and can auto-start if `AUTO_START` is true.
* `exportConfig` returns the *current* in-app state (range, matrix, equalized `INITIAL_COUNT`).
* `waitUntil` resolves once your predicate sees desired metric conditions; it times out (rejects) by default after 60s.

### Debug helpers

```js
simAPI.debug.enable()
simAPI.debug.disable()
simAPI.debug.log('hello')
simAPI.debug.logMetrics(1000)   // periodic console metrics
simAPI.debug.stopLogMetrics()
simAPI.debug.watchEvents()      // wrap API methods with counters/logs
simAPI.debug.testAPI()          // runs a mini test sequence
```

These utilities only print when enabled (`debug.enabled`). The test suite exercises config load, reset, start/stop, waitUntil, and export.

### Example: programmatic scenario

```js
// 1) Load a custom interaction matrix and smaller range
simAPI.loadConfig({
  INTERACTION_RANGE_INIT: 25,
  INTERACTIONS: [
    [ 0.5, -0.2,  0.0,  0.1, -0.3],
    [-0.2,  0.5, -0.1,  0.0,  0.2],
    [ 0.0, -0.1,  0.5, -0.2,  0.0],
    [ 0.1,  0.0, -0.2,  0.5, -0.1],
    [-0.3,  0.2,  0.0, -0.1,  0.5]
  ]
});

// 2) Start the sim
simAPI.start();

// 3) Wait until we have at least 5 metric samples
await simAPI.waitUntil(m => m.step >= 5, { timeoutMs: 10000 });

// 4) Inspect metrics
console.log(simAPI.getMetrics());

// 5) Stop and export the configuration
simAPI.stop();
const cfg = simAPI.exportConfig();
console.log(cfg);
```

The same pattern is used by the included test harness.

---

## 🧪 Test harness (`test.html`)

Open `test.html` to drive the sim from a parent page: it iframes `index.html`, waits for `simAPI`, and exposes buttons: **Check API**, **Run All Tests**, **Test Config**, **Test Lifecycle**, **Test Metrics**, **Enable/Disable Debug**, **Clear Log**. Status messages stream into a monospace log panel with time stamps.

---
