# C08 — Compressed Sensing

> **MRI: Zero to Hero · Module 4 (Advanced Reconstruction)**
> **Source:** Computational MRI (CMRI) Lecture 8 — *Compressed Sensing*, FAU Erlangen-Nürnberg / NYU
> **Foundational papers:** Candès, Romberg & Tao, *IEEE TIT* 2006; Donoho, *IEEE TIT* 2006; Lustig, Donoho, Santos & Pauly, *IEEE Sig. Proc. Mag.* 2008.

---

## The one-paragraph version

Nyquist sampling is wasteful: we acquire a full grid of samples and then immediately throw most of them away when we compress the image (JPEG-2000 keeps only a few wavelet coefficients). **Compressed sensing (CS) builds the compression into the acquisition** — *first compress, then reconstruct* instead of the other way around. It works whenever three things hold together: the image has a **sparse** representation in some transform domain, the undersampling produces **incoherent** (noise-like) aliasing, and we recover the image with a **non-linear** algorithm that promotes sparsity while staying consistent with the measured data. For MRI this is a natural fit, because MR images are compressible and we sample in k-space, where incoherent sampling is easy to arrange.

---

## 1 · Why Nyquist is inefficient

- **Conventional pipeline:** sample at Nyquist → reconstruct → *then* compress with a sparsifying transform, storing only the non-zero coefficients (e.g. 300 kB → 30 kB, a 10× compression).
- **The provocative question:** *why acquire all those samples if we are going to discard most of them?*
- **CS answer:** don't. Move the compression *into* the measurement. A sparse image — *"fewer coefficients than the original"* — should require **fewer samples**.

> 🧠 *Intuition (from the slide margins):* the right questions to keep in mind are *"how many samples do I actually need? what is my FOV, what is my resolution?"* CS reframes the sampling budget around **information rate**, not pixel count.

---

## 2 · Ingredient #1 — Sparsity

An image is **sparse** in a domain if a transform **T** maps it to a vector with only a few large coefficients. Medical images are *not* sparse in pixels, but they are sparse after an appropriate transform.

### Sparsifying transforms

| Transform | Definition | Note |
|---|---|---|
| **Finite differences** | $y(n) = x(n) - x(n-1)$, i.e. $\mathbf{y} = \mathbf{D}\mathbf{x}$ | $\mathbf D$ is bidiagonal $\begin{bmatrix}1&-1&&\\&1&-1&\\&&1&-1\end{bmatrix}$; sparse for piecewise-smooth images |
| **Total variation (TV)** | $TV(x) = \sum_n \lvert x(n)-x(n-1)\rvert = \lVert \mathbf{Dx}\rVert_1$ | Minimising TV **=** minimising the ℓ₁-norm of the finite differences |
| **Wavelets** | Multi-resolution decomposition (recursive wavelet + scaling functions, each scale ½ the resolution) | The JPEG-2000 transform; Daubechies-4 (db4) is the lab's default |

### The TV example (radial subsampling)

As you keep fewer radial spokes, undersampling streaks raise the total variation:

| Data set | Total Variation (a.u.) |
|---|---|
| Original (256×256) | 2801 |
| 128 spokes | 4493 |
| 64 spokes | 6274 |
| 32 spokes | 8281 |

So penalising TV during reconstruction *pushes the streaks back out*.

> 🧠 *Intuition:* compare the wavelet image **B** with the pixel image **A**: the **number of non-zero coefficients** satisfies $B \ll A$, while the **energy** (ℓ²-norm) is almost unchanged, $B \approx A$. That gap — much sparser, same information — is exactly what CS exploits. Always ask: *how sparse is it in the image domain vs. the wavelet domain?*

---

## 3 · Ingredient #2 — Incoherence

Undersampling always aliases. CS needs the aliasing to be **incoherent** — *"it has statistics like noise"* — so a denoiser can remove it. Structured (coherent) replicas cannot be removed.

- **Regular** undersampling → **coherent** aliasing: clean fold-over copies. Deterministic — *"but you can find its pattern,"* which is **bad** for CS because the artifact looks like real signal.
- **Random** undersampling → **incoherent**, noise-like aliasing. **Good** for CS.

### The Transform Point-Spread Function (TPSF)

A tool to *measure* incoherence in the sparse domain. With encoding model $\mathbf{s} = \mathbf{E}\mathbf{m}$ and representation model $\mathbf{p} = \mathbf{W}\mathbf{m}$ (with $\mathbf W$ orthogonal, so $\mathbf{s} = \mathbf{E}\mathbf{W}^H\mathbf{p}$):

$$\mathrm{TPSF}(r) = \mathbf{W}\,\mathbf{E}^H\,\mathbf{E}\,\mathbf{W}^H(r)$$

**How to compute it:**
1. Inverse-FFT the sampling mask (1 = sampled, 0 = not).
2. Apply the sparsifying transform.
3. **Incoherence = ratio of the main peak to the standard deviation of the pseudo-noise floor.** A single sharp peak on a low, flat noise floor = incoherent.

### Two practical facts

- **k-space trajectory matters:** regular Cartesian lines → coherent; random / variable-density / spiral / radial → progressively *more incoherent and random*. Radial in particular is **robust** — it *"doesn't need a high-dimensional pattern to work."*
- **Higher dimensionality → more incoherence:** the same R=4 mask gives incoherence I≈71 at 64×64 but I≈140 at 128×128. *Dimensionality up ⇒ incoherence up.* This is why CS pairs so well with 3D and dynamic imaging.

> 🧠 *Practical pattern:* **variable-density random** sampling — dense centre (captures contrast/energy) + random periphery (incoherence) — is the everyday Cartesian CS choice.

---

## 4 · Ingredient #3 — Non-linear reconstruction

### The optimization

With $\mathbf{E}$ the undersampled Fourier operator, $\mathbf{d}$ the acquired data, and $\mathbf{T}$ the sparsifying transform:

**Constrained form (the ideal):**
$$\min_{\mathbf m}\ \lVert \mathbf{Tm}\rVert_1 \quad\text{subject to}\quad \lVert \mathbf{Em - d}\rVert_2 < \varepsilon$$

- $\lVert\mathbf{Tm}\rVert_1$ = **sparsity** (smallest number of non-zero coefficients in the wavelet representation — *lower ℓ₁ ⇒ better*).
- $\lVert\mathbf{Em-d}\rVert_2 < \varepsilon$ = **data consistency** (the image must explain the measurements).

**Unconstrained form (used in practice):**
$$\min_{\mathbf m}\ \lVert \mathbf{Em - d}\rVert_2^2 + \lambda \lVert \mathbf{Tm}\rVert_1$$

where $\lVert\mathbf x\rVert_1 = \sum_{n=1}^N \lvert x_i\rvert$.

### The regularization parameter λ

λ is the **weighting parameter that balances** data fidelity against artifact removal:

- **High λ** → strong artifact removal & denoising, *but* image corruption: **blurring, ringing, blocking**.
- **Low λ** → no corruption, *but* residual aliasing remains.

### Why not plain gradient descent?

The cost function $C(\mathbf m) = \lVert\mathbf{Em-d}\rVert_2^2 + \lambda\lVert\mathbf{Tm}\rVert_1$ has an ℓ₁ term that is **non-differentiable at 0** (since $\lVert x\rVert_1 = \sum |x|$ has a kink at the origin). Two fixes:

1. **Smoothed gradient descent** — approximate $\lvert x\rvert \approx \sqrt{x^*x + \mu}$, giving
   $$\nabla C(\mathbf m_n) = 2\mathbf{E}^H(\mathbf{Em}_n - \mathbf d) + \lambda\,\mathbf{T}^H\mathbf{M}^{-1}\mathbf{Tm}_n,\qquad M_{ii} = \sqrt{(\mathbf{Tm})_i^*(\mathbf{Tm})_i + \mu}$$
   then iterate $\mathbf m_{n+1} = \mathbf m_n - \alpha\nabla C(\mathbf m_n)$.

2. **Proximal gradient descent = iterative soft-thresholding (ISTA)** — the clean, standard approach:
   $$\mathbf m_{n+1} = \mathbf{T}^{-1}\Big[\,S\big(\mathbf{T}[\mathbf m_n - \mathbf{E}^H(\mathbf{Em}_n - \mathbf d)],\ \lambda\big)\Big]$$
   with the **soft-thresholding operator**
   $$S(x,\lambda) = \begin{cases} 0 & \lvert x\rvert \le \lambda \\[2pt] \dfrac{x}{\lvert x\rvert}\big(\lvert x\rvert - \lambda\big) & \lvert x\rvert > \lambda \end{cases}$$

Each iteration alternates: **(1)** a data-consistency step (*go from k-space to the image, keep measured lines*), **(2)** transform to the sparse domain and **soft-threshold** (*remove the noise-like aliasing*), **(3)** transform back. The picture is the loop *zero-filled image → sparse representation → soft-threshold → k-space → data consistency → repeat*. On sparse signals it cleanly *"gets rid of the noise"* within ~15–30 iterations.

---

## 5 · Image quality in CS

- **SNR is not a good metric.** Soft-thresholding discards small sparse-domain coefficients, which means **loss of contrast, blurring, blockiness, ringing**, and an over-smoothed *"synthetic"* look. The SNR number can look great while diagnostically important detail is gone — *we need something else to evaluate performance* (e.g. task/lesion conspicuity).
- Effects worsen with acceleration (compare R=2 vs R=4).

---

## 6 · Combining CS with parallel imaging (CS & PI)

**Why combine them?**
- Image **sparsity** and **coil-sensitivity** encoding are *complementary* sources of information.
- CS can **regularize** the (often ill-posed) parallel-imaging inverse problem.
- PI can **reduce** the incoherent aliasing artifacts CS leaves behind.

**The tension:** CS wants **irregular** k-space sampling; PI (SENSE/GRAPPA) wants **regular** sampling. The usual reconciliation is **Poisson-disc** sampling (regular-ish spacing + randomness). Note the practical ceiling: the **highest useful acceleration ≈ the number of coils**.

**Approaches:**
- **CS + SENSE** — put coil sensitivities in the data-consistency term: $\min_{\mathbf m}\lVert\mathbf{Em-d}\rVert_2^2 + \lambda\lVert\mathbf{Tm}\rVert_1$ with $\mathbf E$ including coil maps (multicoil variational regularization).
- **CS + GRAPPA** — e.g. **ℓ₁-SPIRiT**.

At 4× acceleration on a 12-channel array, **joint CS & PI** visibly beats both GRAPPA alone (noisy) and coil-by-coil CS.

---

## Key equations at a glance

| Concept | Equation |
|---|---|
| Acquisition model | $\mathbf{d} = \mathbf{E}\mathbf{m}$ |
| Total variation | $TV(x) = \lVert \mathbf{Dx}\rVert_1$ |
| CS (constrained) | $\min \lVert\mathbf{Tm}\rVert_1$ s.t. $\lVert\mathbf{Em-d}\rVert_2 < \varepsilon$ |
| CS (unconstrained) | $\min \lVert\mathbf{Em-d}\rVert_2^2 + \lambda\lVert\mathbf{Tm}\rVert_1$ |
| Soft-thresholding | $S(x,\lambda) = \tfrac{x}{\lvert x\rvert}\max(\lvert x\rvert-\lambda,\,0)$ |
| ISTA update | $\mathbf m_{n+1} = \mathbf{T}^{-1}[\,S(\mathbf{T}[\mathbf m_n - \mathbf{E}^H(\mathbf{Em}_n-\mathbf d)],\lambda)]$ |

---

## Key takeaways

1. **CS is a new sampling theorem** based on **information rate**, not pixel rate.
2. The three ingredients are **sparsity + incoherence + non-linear reconstruction** — all three are required.
3. **MR images are naturally compressible**, and **k-space acquisition makes incoherence easy** → MRI is an ideal CS application.
4. **Random / variable-density** sampling beats regular sampling because its aliasing is **noise-like**.
5. **λ** trades artifact removal against image corruption; tune it (a good rule: threshold ≈ **1 % of the max coefficient** of the initial solution).
6. Judge CS images by **task performance, not SNR**.
7. CS **combines with parallel imaging** (Poisson-disc sampling; CS-SENSE, ℓ₁-SPIRiT) for the highest acceleration.

---

## Self-test

1. Why is acquiring at the Nyquist rate "wasteful" for a compressible image?
2. Name the three ingredients of CS and what each contributes.
3. Why does **random** undersampling beat **regular** undersampling?
4. What does **total variation** minimisation correspond to in terms of the ℓ₁-norm, and why does TV rise as you take fewer spokes?
5. Why can't we use plain gradient descent on the ℓ₁ term, and what replaces it?
6. What happens as **λ → large**? As **λ → 0**?
7. Why is **SNR** a poor quality metric for CS reconstructions?
8. Why do CS and parallel imaging have *conflicting* sampling requirements, and what pattern reconciles them?

*(Work through the companion notebook `C08_notebook.ipynb` — the answers are demonstrated in code, and Lab 8 is solved at the end.)*
