# C05 — Image-Space Parallel Imaging (SENSE)

> **Module 4 · Day 24** — *Computational MRI, Lecture 5: Parallel Imaging I (Image-domain methods)*
> **Source:** CMRI Lecture 5 (FAU / NYU Computational Imaging Lab)
> **Prereqs:** k-space sampling & FOV (C02), the Nyquist relation $\Delta k = 1/\mathrm{FOV}$, basic linear algebra (matrix inverse, pseudo-inverse).

---

## 🎯 What you'll be able to do after this lecture

- Explain *why* MRI is slow and what physically limits how fast we can traverse k-space.
- Combine multi-coil images optimally (sum-of-squares, matched-filter / least-squares, with and without the noise covariance).
- Reconstruct a regularly-undersampled Cartesian acquisition with **SENSE** by unfolding aliased pixels using coil sensitivities.
- Compute and interpret the **g-factor** map, and predict the SNR cost of acceleration.

---

## 🧠 The big picture

Every MRI scan lives inside a **triangle of fate**: *scan time*, *spatial resolution*, and *SNR*. You can optimise any two, never all three — push for speed and resolution and you pay in SNR. Conventional acceleration means traversing k-space faster (bigger, faster-switching gradients), but that road is blocked by hardware power limits ($\propto R^3$), an SNR penalty ($\propto \sqrt{R}$), and peripheral nerve stimulation.

Parallel imaging takes a different road. Instead of acquiring faster, we **acquire less** — we skip phase-encode lines — and then *recover the missing information from data redundancy*. The redundancy comes from using an **array of receive coils**, each with its own spatial sensitivity pattern. Because each coil "sees" the body differently, the coils perform a kind of extra spatial encoding for free. SENSE turns that into a small linear system at every pixel and solves it.

---

## 1 · Why conventional MRI is slow

k-space is traversed by playing out gradients. To go faster by a factor $R$ you must raise gradient amplitude ($\times R$) and slew rate ($\times R^2$), and shorten the dwell time ($/R$). The consequences:

- **Gradient amplifier power** scales as $R^3$ — quickly impractical.
- **SNR** falls as $\sqrt{R}$ because you spend less time collecting signal.
- High slew rates cause **peripheral nerve stimulation** (a hard physiological limit).

So brute-force speed-up hits a wall. The alternative: **undersample k-space** and reconstruct cleverly.

## 2 · k-space undersampling → aliasing

Skipping every other phase-encode line doubles $\Delta k_y$. By the FOV relation $\mathrm{FOV}_y = 1/\Delta k_y$, doubling $\Delta k_y$ halves the field of view. Anything outside the reduced FOV **wraps around** (aliases) on top of the image. This is fast and changes nothing about gradient switching, but a plain inverse FFT now shows the classic fold-over artifact (e.g. two brains stacked on top of each other for $R=2$).

> Undersampling the **readout** direction is possible but pointless — it doesn't save time, because the whole readout line is sampled in one shot. Acceleration always targets the **phase-encode** direction(s).

The cure is to **exploit redundancy**. Two flavors:
- **Image compressibility/sparsity** (inherent redundancy) → partial Fourier, compressed sensing, ML (later modules).
- **Multiple-coil redundancy** (real, measured) → **parallel imaging** (this lecture).

## 3 · Multi-channel receive coils

A receive coil measures voltage by Faraday induction, $u_{\text{ind}} = -\,d\Phi/dt$. A single coil sees the whole body. An **array** of coils each sees a localized region: pixels near a coil are bright, pixels far are dark. Formally, the image seen by coil $i$ is the true image $f(\mathbf r)$ weighted by that coil's complex **sensitivity** $c_i(\mathbf r)$:

$$ m_i(\mathbf r) = c_i(\mathbf r)\, f(\mathbf r) + n_i(\mathbf r) $$

This is the **sensitivity-encoding equation** — the foundation of everything that follows.

### Optimal coil combination (full sampling)

When you simply want one nice image from $N_c$ coils, you combine them. Options, in increasing sophistication:

- **Sum-of-squares (SoS):** $\; f(\mathbf r) = \sqrt{\sum_i |m_i(\mathbf r)|^2}\,$. No sensitivities needed; about a **10% SNR penalty** versus optimal. The squaring is what makes it work — summing complex signals directly would let out-of-phase coils cancel; squaring adds magnitudes.
- **Matched-filter / least-squares (Roemer 1990):** uses the sensitivities,
$$ f = (\mathbf C^H \mathbf C)^{-1}\mathbf C^H \mathbf m, \qquad \mathbf C = \begin{pmatrix}c_1\\\vdots\\c_{N_c}\end{pmatrix},\;\; \mathbf m = \begin{pmatrix}m_1\\\vdots\\m_{N_c}\end{pmatrix}. $$
- **Noise-weighted (with covariance $\boldsymbol\Psi$):** coil noise is **correlated** across channels. The optimal estimator pre-whitens with the coil **noise covariance matrix** $\boldsymbol\Psi$:
$$ f = (\mathbf C^H \boldsymbol\Psi^{-1}\mathbf C)^{-1}\mathbf C^H \boldsymbol\Psi^{-1}\mathbf m. $$
Equivalently, **pre-whiten** the data and sensitivities, $\mathbf m_w=\boldsymbol\Psi^{-1/2}\mathbf m,\ \mathbf C_w=\boldsymbol\Psi^{-1/2}\mathbf C$, then use the plain least-squares formula on the "virtual coils" (which now have uncorrelated, unit-variance noise).

$\boldsymbol\Psi$ is estimated from a **noise-only scan** (RF off): $\hat{\boldsymbol\Psi}=\frac{1}{N_s}\,\mathbf n^H\mathbf n$.

## 4 · SENSE — unfolding the aliasing

Here is the key idea. Accelerate by $R$ along phase encoding. After the inverse FFT, each coil gives an **aliased** image in which $R$ true pixels — spaced $\mathrm{FOV}/R$ apart — sit on top of each other. For one aliased location, with true (unknown) pixel values $u_1,\dots,u_R$:

$$
\underbrace{\begin{pmatrix} I_1 \\ I_2 \\ \vdots \\ I_{N_c}\end{pmatrix}}_{\text{measured, } N_c\times 1}
=
\underbrace{\begin{pmatrix} c_{1}(u_1) & \cdots & c_{1}(u_R) \\ c_{2}(u_1) & \cdots & c_{2}(u_R) \\ \vdots & & \vdots \\ c_{N_c}(u_1) & \cdots & c_{N_c}(u_R)\end{pmatrix}}_{\mathbf C,\;\, N_c\times R}
\underbrace{\begin{pmatrix} u_1 \\ \vdots \\ u_R\end{pmatrix}}_{R\times 1}.
$$

That's a tiny linear system, **one per aliased pixel**. As long as you have **at least as many coils as the acceleration factor** ($N_c \ge R$), it is solvable. The least-squares solution unfolds the pixels:

$$ \mathbf u = (\mathbf C^H \boldsymbol\Psi^{-1}\mathbf C)^{-1}\mathbf C^H \boldsymbol\Psi^{-1}\mathbf I. $$

The number of overlapping pixels equals the acceleration factor; the number of equations equals the number of coils. Solve the system at every aliased location and you have un-aliased the whole image. **Coils whose sensitivities are too similar give a poorly-conditioned $\mathbf C$** — they hand you the same information twice, and the inverse amplifies noise.

> **Why is reconstruction worst near the image center?** That's where all coils overlap most, so their sensitivity vectors are most alike, $\mathbf C$ is least well-conditioned, and noise amplification peaks.

## 5 · The g-factor: the price of acceleration

Acceleration always costs SNR. Two parts:
$$ \mathrm{SNR}_{\text{acc}} = \frac{\mathrm{SNR}_{\text{full}}}{g\sqrt{R}}. $$

- The $\sqrt{R}$ is unavoidable — you collected $R\times$ less data.
- The **g-factor** $g \ge 1$ is *extra* noise amplification from inverting an ill-conditioned encoding matrix. It is **local** (varies pixel to pixel) and rises rapidly with $R$.

$$ g_\rho = \sqrt{\big[(\mathbf C^H\boldsymbol\Psi^{-1}\mathbf C)^{-1}\big]_{\rho\rho}\;\big[\mathbf C^H\boldsymbol\Psi^{-1}\mathbf C\big]_{\rho\rho}} $$

evaluated for each unfolded pixel $\rho$ in the aliased set. Typical behavior: $g\approx 1.0\!-\!1.2$ at $R=2$, climbing steeply by $R=4$, with the worst values toward the center.

### How to reduce noise amplification
- **Use more coils** / **better array design** (more distinct sensitivities) — *hardware*.
- **Regularize** the inverse problem (e.g. Tikhonov: $\hat{\mathbf m}=\arg\min \|\mathbf E\mathbf m-\mathbf s\|_2^2 + \lambda\|\mathbf m\|_2^2$), trading a little bias for much less noise — *software*.
- **Spread the acceleration over two dimensions** in 3D imaging ($R = 2\times 2$ instead of $4\times 1$) — the g-factor is dramatically lower because the unfolding draws on coil variation in two directions.

## 6 · Where SENSE stops being easy

In **non-Cartesian** acquisitions (radial, spiral) the clean decoupling is lost — every pixel aliases with *all* others, so you can no longer solve a tidy little $N_c\times R$ system per location. You must invert the full encoding operator, which calls for an **iterative** algorithm using only matrix–vector products (no explicit inverse). That's the bridge to the next lecture.

---

## ✅ Key takeaways

1. MRI is fundamentally limited by how fast gradients can encode k-space — hardware power, SNR, and nerve stimulation cap the speed.
2. Undersampling phase encoding is free speed but causes aliasing; the fix is to exploit redundancy.
3. A receive-coil array adds *real* spatial-encoding redundancy via distinct complex sensitivities $c_i(\mathbf r)$.
4. SENSE unfolds aliasing by solving $\mathbf I = \mathbf C\mathbf u$ at every pixel; you need $N_c \ge R$.
5. The cost is SNR: $\mathrm{SNR}_{\text{acc}} = \mathrm{SNR}_{\text{full}} / (g\sqrt R)$, with the g-factor capturing local noise amplification from ill-conditioning.
6. More/better coils, regularization, and 2D acceleration all reduce the g-factor.

---

## 📝 Self-test

1. Why does undersampling the **readout** direction not buy you any scan-time savings?
2. You have a 4-channel array and want $R=5$. What goes wrong, and why?
3. Sum-of-squares carries an ~10% SNR penalty versus the matched filter — what does the matched filter know that SoS doesn't?
4. Sketch (in words) why the g-factor is highest at the center of the brain for a circular array.
5. Doubling $R$ from 2 to 4 does more than double the noise. Decompose the SNR loss into its two factors and explain which one is "extra".
6. Pre-whitening replaces your real coils with "virtual coils." What property do those virtual coils have that lets you drop $\boldsymbol\Psi$ from the formula afterwards?
7. Why does non-Cartesian SENSE require an iterative solver while Cartesian SENSE does not?

---

## 📚 References (as cited in the lecture)

- Roemer FB *et al.* "The NMR phased array." *Magn Reson Med* **1990**; 16(2):192–225. *(optimal coil combination)*
- Sodickson DK, Manning WJ. "SMASH." *Magn Reson Med* **1997**; 38:591–603. *(k-space parallel imaging)*
- Pruessmann KP *et al.* "SENSE: sensitivity encoding for fast MRI." *Magn Reson Med* **1999**; 42:952–962. *(the SENSE paper)*
- Lin FH *et al.* "Regularized SENSE." *Magn Reson Med* **2004**; 51:559–567. *(Tikhonov regularization)*
- Ohliger MA *et al.* *Magn Reson Med* **2003**; 50:1018–30. *(2D vs 1D acceleration, ultimate intrinsic SNR)*

---

*Next up — C06: k-space parallel imaging (SMASH/GRAPPA), the same redundancy exploited entirely in k-space.*
