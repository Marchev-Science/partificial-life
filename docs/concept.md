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

## 1) Species & couplings

Let there be five species $\mathcal{S} = \\{ 1, 2, 3, 4, 5 \\}$ (e.g., Red…Pink).  
Interactions are encoded in a $5\times5$ **coupling matrix** $K = [k_{ab}]$, where $k_{ab} \in [-1,1]$ is the strength exerted **by** species $b$ **on** species $a$.

* $k_{ab} > 0$: attraction of $a$ toward $b$.
* $k_{ab} < 0$: repulsion of $a$ from $b$.
* $k_{ab} = 0$: no force within range.

Asymmetry is allowed ($k_{ab} \ne k_{ba}$), which yields **non-reciprocal** interactions and generally non-conservative dynamics (useful to model active matter).

---

## 2) Toroidal displacement (periodic boundaries)

The domain is a square of side $L$ with **periodic boundary conditions** (a flat 2-torus $\mathbb{T}^2$).
For particles $i$ and $j$ at positions $\mathbf{x}_i, \mathbf{x}_j \in [0,L)^2$, define the **minimal-image** displacement:

$$
\Delta x = \bigl((x_j - x_i + \tfrac{L}{2}) \bmod L\bigr) - \tfrac{L}{2},\quad
\Delta y = \bigl((y_j - y_i + \tfrac{L}{2}) \bmod L\bigr) - \tfrac{L}{2}.
$$

Then

$$
\mathbf{r}_{ij} = \begin{bmatrix}\Delta x\\ \Delta y\end{bmatrix},\quad
r_{ij} = \|\mathbf{r}_{ij}\|_2,\quad
\hat{\mathbf{r}}_{ij} = \mathbf{r}_{ij}/(r_{ij}+\varepsilon),
$$

with a tiny $\varepsilon$ to avoid division by zero.

This yields the **toroidal distance** and direction, ensuring interactions “across edges” use the shortest wrap-around path.

---

## 3) Finite-range pairwise force

Let $R>0$ be the **interaction range**. Forces vanish beyond $R$.
For particle $i$ of species $a$ and particle $j$ of species $b$:

$$
\boxed{\;\mathbf{F}_{i\leftarrow j}
= k_{ab}\,\phi(r_{ij})\,\hat{\mathbf{r}}_{ij},\qquad
\phi(r) = \max\!\Bigl(0,\,1-\frac{r}{R}\Bigr)\;}
\tag{1}
$$

* Linear taper to zero at $r=R$ keeps the field short-ranged and Lipschitz.
* The magnitude at contact ($r\to 0$) tends to $|k_{ab}|$.

### Potential view (inside range)

Within $r \le R$, (1) is the negative gradient of a piecewise quadratic potential:

$$
U_{ab}(r) = -k_{ab}\Bigl(r - \frac{r^2}{2R}\Bigr),\quad\mathbf{F}_{i\leftarrow j} = -\frac{dU_{ab}}{dr}\,\hat{\mathbf{r}}_{ij}
\tag{2}
$$

Outside range, $U_{ab}$ is constant, so no force.

---

## 4) Short-range separation (“hard-core”)

To avoid overlap, impose a **minimum centre-to-centre distance** $r_{\min}>0$ (e.g., radius×factor).
A simple linear penalty adds a repulsive term when $r_{ij} < r_{\min}$:

$$
\mathbf{F}^{\text{sep}}_{i\leftarrow j} = \gamma\,\max(0,\,r_{\min} - r_{ij})\,\hat{\mathbf{r}}_{ij}
\tag{3}
$$

with $\gamma>0$ sufficiently large. (Many implementations apply this as an instantaneous positional correction; the linear spring form above is the continuous analogue.)

---

## 5) Total force and time stepping (with damping)

Let $m$ be particle mass (often set to $1$), $D\in(0,1]$ a **velocity damping** factor, and $\Delta t$ the time step.

Total force on particle $i$ (species $a$):

$$
\mathbf{F}_i = \sum_{j\ne i}\bigl(\mathbf{F}_{i\leftarrow j} + \mathbf{F}^{\text{sep}}_{i\leftarrow j}\bigr)
\tag{4}
$$

A common explicit, damped Euler update is:

$$
\mathbf{v}_i^{t+1} = D\,\mathbf{v}_i^{t} + \frac{\Delta t}{m}\,\mathbf{F}_i^{t},\qquad \mathbf{x}_i^{t+1} = \mathrm{wrap}\!\left(\mathbf{x}_i^{t} + \Delta t\,\mathbf{v}_i^{t+1}\right)
\tag{5}
$$

where $\mathrm{wrap}([x,y]) = ([x\bmod L],[y\bmod L])$ maps positions back to $[0,L)$ per component.
Because $D<1$ and $K$ may be asymmetric, momentum and mechanical energy are **not** conserved (this is typical for visually stable, interactive particle systems).

---

## 6) “Every pair” across five species

Let species indices $a,b\in\{1,\dots,5\}$. For **any** pair $(a,b)$, the pairwise law is the same as (1), parameterized by $k_{ab}$. Concretely:

$$
\mathbf{F}^{(a\leftarrow b)}(r) = k_{ab}\,\max\!\Bigl(0,\,1-\frac{r}{R}\Bigr)\,\hat{\mathbf{r}},\quad \text{for } (a,b)\in\{1,\dots,5\}^2
\tag{6}
$$

That yields 25 interaction channels:

* **Self-interactions**: $k_{11}, k_{22},\dots,k_{55}$ (control cohesion/dispersion of each colour).
* **Cross-interactions**: $k_{ab}$ for $a\ne b$ (control hetero-species attraction/repulsion).
  If $k_{ab}=k_{ba}$ for all $a,b$, the field is reciprocal; if not, directed couplings can produce rotation, chasing, or steady non-equilibrium flows.

---

## 7) Torus-world principle (why & how)

* **What it is.** The 2-torus $\mathbb{T}^2$ identifies opposite edges: exiting right re-enters left; top connects to bottom. Mathematically: positions live in $[0,L)^2/\sim$ where $(x,y)\sim(x\!\pm\!L,y)\sim(x,y\!\pm\!L)$.
* **Why it’s used.**

  1. Removes boundary artifacts (no walls).
  2. Preserves mean density.
  3. Keeps the simulation finite while approximating an infinite tiling.
* **Computational recipe.**

  * Compute forces with the **minimal image** displacement (Section 2).
  * Integrate positions and then apply the **wrap** map (5).
  * Distances and neighbourhood queries must always use the toroidal metric; Euclidean distance in $[0,L]^2$ without wrapping is incorrect near edges.

---

## 8) Quick pseudocode (for clarity)

```js
for each i:
  F = (0,0)
  for each j != i:
    // toroidal minimal-image displacement
    dx = ((x[j]-x[i] + L/2) % L) - L/2
    dy = ((y[j]-y[i] + L/2) % L) - L/2
    r = sqrt(dx*dx + dy*dy)
    if r > 0:
      ux = dx / r; uy = dy / r

      // finite-range pairwise force
      phi = Math.max(0, 1 - r/R)
      F.x += K[a_i][a_j] * phi * ux
      F.y += K[a_i][a_j] * phi * uy

      // short-range separation
      sep = Math.max(0, r_min - r)
      F.x += gamma * sep * ux
      F.y += gamma * sep * uy

  // damped Euler step
  v[i].x = D * v[i].x + dt * F.x
  v[i].y = D * v[i].y + dt * F.y
  x[i].x = (x[i].x + dt * v[i].x) % L
  x[i].y = (x[i].y + dt * v[i].y) % L
  if (x[i].x < 0) x[i].x += L
  if (x[i].y < 0) x[i].y += L
```

---

## 9) Practical notes

* **Tuning:** Increase $R$ for larger neighbourhoods; use $k_{aa}>0$ for cohesive same-colour clusters; set selected $k_{ab}<0$ to create inter-species segregation.
* **Stability:** Larger $|K|$ or $R$ may require stronger damping $D$ (or smaller $\Delta t$) to avoid overshoot.
* **Non-reciprocal design:** Choosing $k_{ab}\neq k_{ba}$ can generate pursuit/escape motifs and persistent swirl patterns that cannot be captured by symmetric potentials.

If you want, I can tailor this to specific numeric defaults (e.g., your $R$, $r_{\min}$, $D$, $\Delta t$) or derive a continuous-time limit.

