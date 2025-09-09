# Partificial life metrics  

Listing metrics that practitioners of pattern-formation, complex-systems, and agent-based modelling commonly use. Below are concise, implementation-ready definitions of the **five metrics** you asked to keep, adjusted to match your simulation’s specifics (continuous screen, multicolour particles).  No code—just the mathematics, what each term means, and practical notes.

---

## 1. Colour-weighted spatial clustering

### Generalised Ripley $K_\mathrm{col}(r)$

$$
K_{\mathrm{col}}(r) = \frac{A}{N^{2}} 
      \sum_{i \ne j}
      w(c_i, c_j)\,
      \mathbf{1}[d_{ij} < r]
$$


| Symbol       | Meaning                           |
| ------------ | ----------------------------------- |
| $A$          | 2-D area of the visible domain.     |
| $N$          | Number of particles in the snapshot. |
| $\mathbf 1$  | Indicator (1 if the condition holds, 0 otherwise). |
| $d_{ij}$     | Euclidean distance between particles $i$ and $j$.  |
| $c_i$        | Colour (RGB or HSV vector, or discrete label) of particle $i$. |
| $w(c_i,c_j)$ | **Colour similarity weight**.  Two common choices:<br>• *Hard*: $w=1$ if the colours match a rule (e.g. same label or ΔE < ε), else $0$.<br>• *Soft*: $w = \exp\left[-\lVert c_i - c_j \rVert^2/(2\sigma_c^2)\right]$ so near hues still contribute. |

*Interpretation* – Compare $K_\mathrm{col}(r)$ to the theoretical value for a random, colour-independent distribution $K_\text{Poisson}(r)=\pi r^{2}$.  
 *  $K_\mathrm{col}(r) > \pi r^{2}$: like-coloured particles form clusters at scale $r$.
 *  $K_\mathrm{col}(r) < \pi r^{2}$: they repel or mix.

*Practical tips* – Evaluate $r$ over a logarithmic range (e.g. 5–50 px) and plot the normalised ratio $L_\mathrm{col}(r)=\sqrt{K_\mathrm{col}(r)/\pi}-r$ to make deviations from randomness visually flat when there is no clustering.

---

## 2. Shannon spatial entropy without a grid

### k-nearest-neighbour (Kozachenko–Leonenko) estimator

For a point cloud in 2-D, differential entropy $H$:

$$
H \approx \psi(N) - \psi(k) + \ln c_2 + \frac{2}{N}\sum_{i=1}^{N}\ln\varepsilon_i \quad\text{with}\quad c_2=\pi
$$

| Term              | Meaning                                                                      |
| ----------------- | ---------------------------------------------------------------------------- |
| $\psi(\cdot)$     | Digamma function.                                                            |
| $k$               | Chosen neighbour order (2–5 is robust).                                      |
| $\varepsilon_i$   | Distance from particle $i$ to its $k$-th nearest neighbour (exclude itself). |

*Why it fits your canvas* – No artificial bins: entropy comes purely from inter-particle spacings. A tightly packed crystal gives small $\varepsilon_i$ values → low $H$; a well-mixed gas spreads the $\varepsilon_i$ values → high $H$.

*Multicolour extension* – Compute $H$ separately for each colour class and average, or append chromatic dimensions to the position vector and use a weighted metric $\sqrt{d_{xy}^2+\beta d_{\text{color}}^2}$.

---

## 3. Mean frame-to-frame displacement

### Average particle speed

$$
\langle v\rangle(t) = \frac{1}{N \Delta t}
\sum_{i=1}^{N}
\lVert\mathbf x_i(t+\Delta t) - \mathbf x_i(t)\rVert
$$

*Use* – Tracks how “busy” the system is.  Plot alongside clustering to spot phases where motion slows but structure still coarsens, or vice-versa.

---

## 4. Structural similarity change rate

### SSIM-based frame difference

1. Render each frame to a luminance image $I(t)$ (e.g. ITU-R BT.601 $Y=0.299R+0.587G+0.114B$).
2. Compute the Structural Similarity Index ${SSIM}\bigl(I(t),I(t+\Delta t)\bigr)\in[-1,1]$.
3. Define the change metric

$$
R_\text{change}(t) = 1-{SSIM}\bigl(I(t),I(t+\Delta t)\bigr)\quad\in[0,2]
$$

*Interpretation* –

* **0** : two frames identical.
* **≈1** : perceptible but moderate alterations.
* **>1** : major overhaul (colour swap, large translation, etc.).

Because SSIM mixes luminance, contrast, and local correlations, $R_\text{change}$ captures both particle motion and appearance changes in a perceptually meaningful way.

---

### How they complement each other

| Snapshot metric     | What it tells you                                             | Dynamic metric     | What it adds                                                         |
| ------------------- | ------------------------------------------------------------- | ------------------ | -------------------------------------------------------------------- |
| $K_\mathrm{col}(r)$ | **Where & how strongly** like colours cluster at every scale. | $\langle v\rangle$ | **How fast** particles roam—raw kinetics.                            |
| $H$ (k-NN)          | Overall **positional diversity** regardless of grids.         | $R_\text{change}$  | **Perceptual turnover** between frames, including pattern reshaping. |

Tracking all four gives a dashboard that separates spatial order, heterogeneity, physical activity, and visual novelty—rich enough to diagnose self-organisation without yet touching implementation details.



### 5. Composite Volatility Index (CVI)

*A single number that tells you how “settled” or “nervous” the whole simulation is by pooling the four metric time‑series you already track.*

---

#### Step 0 — Normalise each metric

Convert every raw series to a comparable scale:

$$
z_k(t) = \frac{M_k(t)-\mu_k}{\sigma_k}\quad\text{or}\quad z_k(t) = \frac{M_k(t)-\min(M_k)}{\max(M_k)-\min(M_k)}
$$

where $k\in\{1:K_{\text{col}},2:H,3:\langle v\rangle,4:R_{\text{change}}\}$.

---

#### Step 1 — Form the change vector

$$
\Delta\mathbf z(t)=
\begin{bmatrix}
z_1(t)-z_1(t-\Delta t)\\
z_2(t)-z_2(t-\Delta t)\\
z_3(t)-z_3(t-\Delta t)\\
z_4(t)-z_4(t-\Delta t)
\end{bmatrix}
$$

---

#### Step 2 — Instantaneous composite volatility

Choose non‑negative weights $w_k$ that sum to 1 (default $w_k=\tfrac14$).
Define

$$
\boxed{ 
\text{CVI}(t) = 
\sqrt{ \sum_{k=1}^{4} w_k \bigl[\Delta z_k(t)\bigr]^{2}} }
$$

*Interpretation:*<br>• **0** perfect stability (all four metrics flat);<br>• **↑** a larger number means at least one metric—or several in concert—jumped markedly in that frame.  

---

#### Step 3 — Windowed stability score (optional)

To judge steadiness over a span $T_w$ frames, take the running root‑mean‑square:

$$
\sigma_{\text{CVI}}(t) = 
\sqrt{\frac{1}{T_w}\sum_{\tau=t-T_w+1}^{t}\bigl[\text{CVI}(\tau)\bigr]^{2}}
$$

Small $\sigma_{\text{CVI}}$ ⇒ system has settled into a quasi‑stationary regime;
large $\sigma_{\text{CVI}}$ ⇒ continuing reorganisation or intermittent bursts.

---

#### Why this works

* **Scale‑free** – Normalisation removes unit and magnitude biases between clustering, entropy, motion, and perceptual change.
* **Multivariate** – By taking a Euclidean (or Mahalanobis, if you replace weights by the inverse covariance) norm of the four‑dimension change vector, you capture *co‑fluctuations*, not just each metric in isolation.
* **Adjustable** – Weights let you down‑play a noisy metric (e.g., SSIM flicker) or emphasise a key one (e.g., entropy).

Use CVI alongside the four base metrics: the latter tell you *what* is happening; CVI tells you *how restless* the system is as a whole.



## Read also:  
* [Overview](README.md)
* [General concept](docs/concept.md)
* [Detailed methododology on metrics](docs/metrics.md)
* [Working with configurations](docs/configs.md)
* [Working with the API](docs/api.md)



