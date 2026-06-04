# C06 — k-Space Parallel Imaging: SMASH & GRAPPA

> **Module 4 — Advanced Reconstruction · Day 26**
> **Source:** Computational MRI — *"Parallel Imaging II: k-space methods"* (FAU Erlangen / NYU)
> **Companion notebook:** `C06_notebook.ipynb`
> **Key papers:** Sodickson MRM 1997 (SMASH) · Griswold MRM 2002 (GRAPPA) · Pruessmann MRM 1999 (SENSE, prev. lecture)

---

## The one-paragraph picture

Parallel imaging speeds up MRI by **skipping phase-encode lines** in k-space. Skipping lines shrinks the field
of view, so the image folds onto itself (aliasing). A **multi-coil receiver array** rescues us: each coil sees
the object through its own smooth spatial sensitivity, so the coils together carry the spatial information the
skipped lines threw away. **SENSE** (last lecture) used that information in *image space* — IFFT first, then
unfold the aliased pixels. **This lecture** uses it in *k-space* — fill the missing lines first, then IFFT.
**SMASH** does this by synthesising spatial harmonics from coil sensitivities; **GRAPPA** generalises it to a
robust, coil-by-coil, data-driven k-space interpolation that needs no coil maps. GRAPPA is what scanners
actually use.

---

## 1 · Recap — undersampling causes aliasing

Keeping every $R$-th line along $k_y$ (spacing $R\,\Delta k_y$) reduces the FOV to $\text{FOV}/R$. In image
space the object wraps onto itself $R$ times. $R$ is the **acceleration / undersampling factor**, and it is the
*same thing* as the harmonic order $m$ in the equations below. With one coil this information is gone; with
$N_c$ coils it is recoverable, and $R \le N_c$ is the hard ceiling.

---

## 2 · The duality that powers everything

A coil measures the object **times** its sensitivity:

$$\text{coil image}_\ell(\mathbf r) = C_\ell(\mathbf r)\,\rho(\mathbf r) \;\;\xleftrightarrow{\;\text{FT}\;}\;\; \hat C_\ell(\mathbf k)\;\ast\;\hat\rho(\mathbf k).$$

> **Multiplication in image space ⇔ convolution in k-space.** This is the *core principle of GRAPPA.*

Because sensitivities are **smooth**, $\hat C_\ell$ is concentrated near the k-space centre (only low
frequencies). So the convolution is **local**: a coil's k-space point is a short weighted blend of *nearby*
object k-space points. GRAPPA just learns those local weights. (It also explains why the calibration data only
needs the *centre* of k-space.)

---

## 3 · SENSE vs GRAPPA — *when do you run the IFFT?*

This is the entire conceptual difference, and worth memorising:

| | **SENSE** (image space) | **GRAPPA** (k-space) |
|---|---|---|
| Step 1 | **IFFT** → get an aliased image | **Compute** → synthesise missing k-space lines |
| Step 2 | **Compute** → unfold pixels with coil maps | **IFFT** → get artefact-free coil images |
| Solves | a small linear system per pixel | a k-space interpolation (fitting) problem |
| Needs coil maps? | **Yes** (explicit) | **No** (weights learned from data) |

---

## 4 · SMASH — *Simultaneous Acquisition of Spatial Harmonics* (Sodickson 1997)

**Insight.** A missing line at $k_y + m\,\Delta k_y$ is the acquired line at $k_y$ **modulated** by a spatial
harmonic of order $m$. By the Fourier **shift theorem**, $F\{e^{i 2\pi k_0 r} s(r)\} = S(k - k_0)$ —
multiplying the image by a harmonic *shifts* its k-space, manufacturing the skipped line.

**Trick.** We cannot physically multiply by a harmonic, but we may try to *synthesise* one as a weighted sum of
the coil sensitivities:

$$\sum_{\ell=1}^{N_c} w_\ell^{(m)}\, C_\ell(y) \;\approx\; e^{-i\,m\,\Delta k_y\, y}.$$

If those weights exist, the **same** weights applied to the coil *signals* synthesise the missing line:

$$S(k_y + m\,\Delta k_y) = \sum_{\ell=1}^{N_c} w_\ell^{(m)} \int C_\ell(y)\,\rho(y)\,e^{-i k_y y}\,dy.$$

**Why it's fragile.** Synthesising a clean order-$m$ harmonic from a handful of fixed coil shapes gets harder as
$m$ grows and depends entirely on array geometry — *"it's not easy to build a coil to make exactly these
signals."* SMASH also yields a single combined image (no proper sum-of-squares). GRAPPA removes both limitations.

---

## 5 · GRAPPA — *GeneRalized Autocalibrating Partially Parallel Acquisitions* (Griswold 2002)

**Idea (slides 29–34).** A missing k-space point in coil $j$ is a **linear combination of the acquired
neighbours from *all* coils**. Stacking many such relations:

$$\boxed{\,T = S\,w\,}\qquad\Rightarrow\qquad \boxed{\,w = (S^{H}S)^{-1}S^{H}T\,}$$

- **Source** $S$ — for each target, a $k_h\times k_w$ block of acquired points, over all $N_c$ coils
  → length $n_k n_c$ (with $n_k = k_h k_w$).
- **Target** $T$ — the missing point(s), one value per coil → length $n_c$.
- **Weights** $w$ — the GRAPPA kernel, shape $n_k n_c \times n_c$.

**Matrix dimensions** (example: $R=2$, kernel $2\times3$, $N_c=8$, $n_b$ calibration blocks):

| matrix | meaning | shape | example |
|---|---|---|---|
| $S$ | source points | $n_b \times n_k n_c$ | $n_b \times 48$ |
| $T$ | target points | $n_b \times n_c$   | $n_b \times 8$  |
| $w$ | GRAPPA weights | $n_k n_c \times n_c$ | $48 \times 8$ |

### The algorithm
1. **Calibrate.** From the fully-sampled centre of k-space — the **Auto-Calibration Signal (ACS)** — both sources
   and targets are known. Slide the kernel over the ACS to gather many $(S,T)$ pairs and solve for $w$.
2. **Synthesise.** Apply $w$ to the undersampled data to fill every missing line, **coil by coil**.
3. **IFFT** each coil; **combine** (sum-of-squares or matched filter). Zero-pad at the k-space border.

The ACS can be embedded in the acquisition (*autocalibration*) or measured separately.

---

## 6 · Parameters & trade-offs (the Lab-6 lessons)

- **Acceleration $R$** — higher $R$ ⇒ less data, more extrapolation, and stronger **noise amplification**
  captured by the **g-factor** (worst where coil sensitivities overlap). Quality falls monotonically; $R \le N_c$.
- **Kernel size** — a *bias–variance* knob. Too small underfits the k-space convolution; too large explodes the
  number of weights ($n_k n_c \times n_c$) and overfits the limited calibration data. Bigger kernels **demand**
  more ACS lines.
- **ACS lines** — more calibration ⇒ better-conditioned, more accurate weights (then plateaus). **But** ACS lines
  are acquired, so they reduce the *effective* acceleration. Pick the smallest ACS + kernel with acceptable
  artefacts.
- **Conditioning / "is it invertible?"** — with too few blocks, $S^{H}S$ ($n_k n_c \times n_k n_c$) is rank
  deficient and **singular**. Fix with more ACS lines and **Tikhonov regularisation** (the `lam` term). In the
  notebook, $\lambda = 0$ with a $6\times7$ kernel and only 24 ACS lines drops PSNR to ~12 dB; a small $\lambda$
  recovers it.

---

## 7 · GRAPPA vs SENSE — pros, cons, and the verdict

**GRAPPA advantages**
- No explicit **coil-sensitivity estimation** (a major real-world SENSE error source).
- The small kernel acts as **implicit regularisation**.
- **More robust** to inconsistencies between calibration and imaging data.

**GRAPPA disadvantages — less flexible**
- Sensitive to **ACS region size** and **kernel size**.
- **Sampling-geometry dependent:** trivial for 1-D acceleration (one weight set works everywhere), harder for 2-D
  (each geometry needs its own weights), and very cumbersome for irregular / non-Cartesian sampling (a different
  kernel per geometry).

**Lecturer's verdict:** *"If you can, use SENSE. But GRAPPA is widely used — particularly on Siemens scanners —
so it is important to understand it."* With perfect coil maps SENSE is near-optimal; GRAPPA wins when maps are
unreliable.

---

## 8 · Key takeaways
1. Parallel imaging = undersample k-space, use **multi-coil** spatial information to recover the rest.
2. SENSE works in **image space** (IFFT → unfold); GRAPPA works in **k-space** (fill → IFFT).
3. *Image-space multiplication = k-space convolution* — and because coil maps are smooth, that convolution is
   **local**: GRAPPA learns short k-space interpolation kernels.
4. GRAPPA: calibrate from the ACS ($T=S w \Rightarrow w=(S^HS)^{-1}S^HT$), synthesise missing lines coil-by-coil,
   then IFFT + combine. **No coil maps needed.**
5. Quality vs. $R$, kernel size, and ACS lines is a conditioning/regularisation trade-off; the **g-factor**
   captures acceleration's noise penalty.

---

## 9 · Self-test (with answers)

**Q1. State the SENSE/GRAPPA difference in terms of *when the IFFT happens*.**
SENSE does the **IFFT first** (producing an aliased image) then computes (unfolds with coil maps). GRAPPA
**computes first** (synthesises missing k-space lines) then does the IFFT.

**Q2. Multiplication in image space equals what in k-space, and why does that make GRAPPA local?**
A **convolution**. Coil sensitivities are smooth, so their k-space ($\hat C_\ell$) is concentrated at low
frequencies; convolving with a narrow kernel only mixes **nearby** k-space points, so a small local interpolation
kernel suffices.

**Q3. What is the ACS, and what are $S$ and $T$ for?**
The **Auto-Calibration Signal** is a fully-sampled central k-space block. Within it both **source** points $S$
(acquired neighbourhoods) and **target** points $T$ (the would-be-missing points) are known, so they provide the
training examples used to fit the GRAPPA weights $w$.

**Q4. GRAPPA weight equation and shapes for $R=2$, kernel $2\times3$, $N_c=8$.**
$w=(S^HS)^{-1}S^HT$. With $n_b$ calibration blocks: $S$ is $n_b\times48$, $T$ is $n_b\times8$, $w$ is
$48\times8$ (since $n_k n_c = 2\cdot3\cdot8 = 48$).

**Q5. Why does quality fall as $R$ grows? Name the noise quantity.**
Fewer measured lines mean more extrapolation, and the linear system becomes more ill-conditioned where coil
sensitivities overlap, amplifying noise. That amplification is the **g-factor**.

**Q6. One advantage and one disadvantage of GRAPPA vs SENSE.**
*Advantage:* needs no explicit coil-sensitivity maps (more robust to map errors). *Disadvantage:* less flexible —
strongly tied to sampling geometry, so 2-D/non-Cartesian acceleration is awkward.

**Q7. Why is GRAPPA awkward for non-Cartesian / 2-D acceleration?**
The learned weights are tied to a specific neighbour geometry. For 1-D Cartesian one weight set works everywhere,
but each different 2-D or irregular sampling pattern needs its **own** kernel, which is cumbersome.

---

## 10 · Further reading
- Sodickson & Manning, *SMASH*, **MRM 1997**.
- Griswold et al., *GRAPPA*, **MRM 2002**.
- Pruessmann et al., *SENSE*, **MRM 1999** (the image-space counterpart, C05).
- Deshmane et al., *Parallel MR imaging*, **JMRI 2012** (excellent review of both families).
