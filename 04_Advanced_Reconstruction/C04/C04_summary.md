# 📕 C04 — Reconstruction of Non-Cartesian MRI Data

> **Module 4 · Day 23 of the ROADMAP** — *Computational MRI, Lecture 4 (FAU Erlangen-Nürnberg / NYU)*
>
> Pairs with `C04_notebook.ipynb`. Read this first (derivations and all), then run the notebook cell-by-cell.

---

## 1. The big picture

Cartesian MRI is easy to reconstruct: sample k-space on a rectangular grid, call `ifft2`, done. But many of the most useful sequences trace **curved or rotated paths** through k-space — radial, spiral, EPI. For those the inverse FFT *alone does not work*, because the DFT is only defined for **uniformly-spaced samples on a grid**.

This lecture builds the classical toolkit for reconstructing such data — **gridding** and its CT-flavoured cousin **filtered backprojection** — and then points at the modern, optimised version (**NUFFT**) used everywhere today.

The MR signal equation is unchanged; k-space is still the Fourier transform of the object:

$$ s(k_x,k_y) = \iint m(x,y)\, e^{-i 2\pi (k_x x + k_y y)}\, dx\, dy $$

What changes is only *where* in k-space we sample, and therefore how we invert.

---

## 2. Non-Cartesian trajectories

A **trajectory** is the path the acquisition takes through k-space as the readout gradients play out. The three classic non-Cartesian patterns:

| Trajectory | Path | Character |
|---|---|---|
| **Radial** | straight spokes through the centre | the original Lauterbur 1973 scheme; CT-like |
| **EPI** | a fast zig-zag raster | covers k-space in one or few shots |
| **Spiral** | an outward spiral | very efficient gradient usage |

**Why use them.** Non-Cartesian readouts are typically **fast**, **motion-robust**, and **self-navigating** (the centre of k-space is re-measured every shot, giving a free motion/contrast reference).

**The price.** They are **susceptible to hardware imperfections** — off-resonance, gradient non-linearity, eddy currents — and they are **numerically harder to reconstruct**. That reconstruction difficulty is the entire subject of this lecture.

---

## 3. Why the direct FFT fails, and the two ways out

Radial/spiral/EPI samples land *between* Cartesian grid points, so there is no rectangular array to feed to `ifft2`. Two general escape routes:

1. **DFT along the trajectory** — evaluate $m(x,y)=\sum_j s(k_j)\,e^{+i2\pi(k_{x,j}x+k_{y,j}y)}$ directly at every pixel. Exact, but $O(N^2\!\cdot\! N_\text{samp})$ — **slow**.
2. **Gridding** — interpolate the scattered samples onto a Cartesian grid with a small kernel, then use the **fast** FFT. This is what everyone actually does.

For the special case of **radial** data there is a third, historically-first route inherited from CT: **filtered backprojection**. It is worth understanding because it explains *why density compensation exists*.

---

## 4. The CT connection

Lauterbur's 1973 experiment used radial sampling and reconstructed exactly like a CT scanner. The vocabulary transfers directly.

### 4.1 Radon transform (projection)

A projection integrates the object along parallel lines at angle $\phi$:

$$ P(s,\phi)=\iint I(x,y)\,\delta(x\cos\phi + y\sin\phi - s)\,dx\,dy $$

Stacking projections for all angles gives the **sinogram** — a single point in the image traces a sine wave in $(s,\phi)$, hence the name.

### 4.2 Fourier-slice (central-slice) theorem

> The **1-D Fourier transform of the projection** at angle $\phi$ equals the **radial line of k-space** at that same angle $\phi$.

So a radial MRI spoke *is* the Fourier transform of a CT projection. You can hop freely between sinogram-space and radial k-space with a single 1-D FFT (Bracewell, 1956).

### 4.3 Backprojection — and why it blurs

To invert, smear each projection back across the image along its angle and sum:

$$ b(x,y)=\sum_{m=1}^{M} P\big(x\cos\phi_m + y\sin\phi_m,\ \phi_m\big) $$

Plain backprojection is badly **blurred**: low frequencies are massively over-emphasised and high-frequency detail is lost. The cause is purely geometric — every radial spoke passes through the **centre** of k-space, so the centre is sampled $\approx N$ times while the periphery is sparse. This is **variable-density sampling**.

### 4.4 Filtered backprojection — the ramp filter

Fix it by weighting k-space with a **ramp ("rho") filter** $\rho(k)\propto|k|$, rising from $\approx 1/N$ at the centre to $1$ at the edge. This down-weights the over-sampled centre and boosts the under-sampled periphery, exactly compensating the radial density. The result is sharp.

**Algorithm:** k-space data → multiply by density-compensation (ramp) filter → 1-D IFFT → Radon-space data → backprojection → image.

> ⚠️ Filtered backprojection works **only** for radial ("radiate") data, because it is built on CT-style rotation. **Gridding works for any trajectory** — so that is the general tool.

---

## 5. Gridding

> **convolve → resample → inverse FFT**

Interpolate the scattered samples onto a Cartesian grid with a small kernel, then FFT.

### 5.1 The math

The non-Cartesian sampling pattern is a sum of deltas at the trajectory locations:

$$ S(k_x,k_y)=\sum_i \delta\!\left(k_x-k_{x,i},\,k_y-k_{y,i}\right) $$

The measured data is $M(k_x,k_y)\,S(k_x,k_y)$. Gridding convolves this with a kernel $C$ and resamples on the Cartesian comb $\mathrm{III}$:

$$ \hat M(k_x,k_y)=\Big[\big(M\,S\big)\ast C\Big]\times \mathrm{III}\!\left(\tfrac{k_x}{K_x},\tfrac{k_y}{K_y}\right) $$

By the convolution theorem, **convolution in k-space = multiplication in image space**. After the inverse FFT:

$$ \hat m(x,y)=\Big[\big(m\ast s\big)\,c(x,y)\Big]\ast\mathrm{III}\!\left(\tfrac{x}{FOV_x},\tfrac{y}{FOV_y}\right) $$

where $c(x,y)=\mathcal F^{-1}\{C\}$. Each operation leaves an image-space fingerprint:

| k-space operation | image-space effect | cure |
|---|---|---|
| non-uniform sampling density | blurring / shading | **density compensation** (§6) |
| convolution with a finite kernel | **apodization** (intensity roll-off) + side-lobes | **deapodization** (§8) + oversampling (§7) |
| resampling onto a finite grid | **replication** (aliasing) | oversample the grid (§7) |

### 5.2 The triangular kernel (`grid1`)

The simplest practical kernel is **triangular** (bilinear) of width 2: each sample is spread onto its four nearest grid points with weights $(1-|\Delta x|)(1-|\Delta y|)$. Fast, but crude — broad point-spread and noticeable side-lobes. This is the kernel used in the lab's `grid1`.

---

## 6. Density compensation

Non-Cartesian trajectories sample k-space with **variable density**. For radial, the central point is hit once per spoke — $N$ times in total — while the edge is sparse. Before gridding, **pre-compensate** each sample by the inverse of its local density $\rho$:

$$ \hat M = \left[\left(\tfrac{M}{\rho}\,S\right)\ast C\right]\times\mathrm{III} $$

- **Radial:** density is known analytically and falls as $1/|k|$, so the weight is the **ramp** $\rho^{-1}\propto|k|$ — exactly the CT ramp filter of §4.4.
- **Arbitrary trajectory:** compute the per-sample area numerically, e.g. a **Voronoi diagram**.

This is the single highest-impact correction — without it, *every* reconstruction (gridding or NUFFT) collapses to a bright central blob.

---

## 7. Oversampling the Cartesian grid

The finite kernel causes **aliasing/replication** (the $\mathrm{III}$ comb folds neighbouring image copies into the FOV) and **apodization** (intensity roll-off). The cheap brute-force fix for both: grid onto a **larger** grid (e.g. $2N\times 2N$), inverse-FFT, then **crop** the central $N\times N$ in the image domain.

Why it works: a finer image-domain sampling pushes the aliased copies *outside* the FOV, and the steep part of the apodization roll-off lands *outside* the cropped region — leaving only mild residual shading. (Removes aliasing; reduces apodization.)

---

## 8. Deapodization

Convolving with the kernel $C(k)$ multiplies the image by $c(x)=\mathcal F^{-1}\{C\}$ — the **apodization** function (bright centre, dim edges). Undo it by **dividing** the image by $c(x)$, with a small constant $a$ to avoid division by zero:

$$ \hat m(x,y)=\frac{1}{c(x,y)+a}\left\{\big[(m\ast s)\,c(x,y)\big]\ast\mathrm{III}\!\left(\tfrac{x}{FOV_x},\tfrac{y}{FOV_y}\right)\right\} $$

- For the **triangular** (width-2) kernel, $c(x)=\operatorname{sinc}^2(x/N)$ analytically.
- For a **general** kernel, obtain $c(x)$ numerically: **grid a single delta** at $k=0$, inverse-FFT, and read off the kernel's image-domain footprint.

With a narrow kernel + oversampling the correction is mild; with wide kernels it matters a lot.

---

## 9. Better kernels — Kaiser–Bessel

The **ideal** kernel is an infinite sinc (its transform is a perfect rect → no apodization, no aliasing) but is impractical. So we use a **windowed sinc**; the consensus best practical choice is the **Kaiser–Bessel** kernel:

$$ C(k)=\frac{1}{W}\,I_0\!\left(b\sqrt{1-\left(\tfrac{2k}{W}\right)^2}\,\right)\operatorname{rect}\!\left(\tfrac{2k}{W}\right), \qquad c(x)=\frac{\sin\!\sqrt{\pi^2 W^2 x^2 - b^2}}{\sqrt{\pi^2 W^2 x^2 - b^2}} $$

with $I_0$ the zeroth-order modified Bessel function of the first kind, $W$ the kernel width, and $b$ a shape parameter. It concentrates energy in the main lobe — and suppresses side-lobes — far better than a triangle of the same width, which means **less aliasing** and a cleaner image.

---

## 10. Non-Uniform FFT (NUFFT)

The NUFFT is a **generalised, highly-optimised gridding**: same idea (Kaiser–Bessel interpolation + oversampling + deapodization), packaged as fast **forward and adjoint** operators you can drop into iterative reconstructions (later in Module 4, and the deep-learning recon of Module 5). Widely-used implementations: `sigpy`, `torchkbnufft`, `BART`, `gpuNUFFT`, Fessler's IRT (Fessler & Sutton, IEEE TSP 2003).

---

## 11. Gridding reconstruction — the full recipe

1. Compute the non-Cartesian k-space **sampling pattern** (trajectory).
2. Choose the **gridding kernel** (e.g. Kaiser–Bessel).
3. **Density pre-compensation** (analytic for radial; Voronoi otherwise).
4. **Convolve** the pre-compensated data with the kernel and **resample** onto an **oversampled** Cartesian grid.
5. **Inverse FFT.**
6. **Deapodize** (divide by the kernel's image-domain transform).
7. **Crop** to remove the oversampling.

---

## 12. Key takeaways

- The DFT needs a uniform grid; non-Cartesian data must be **gridded** (or reconstructed by DFT / backprojection).
- A radial spoke is a CT projection in disguise — **Fourier-slice theorem**.
- Radial sampling over-weights the k-space centre → **density compensation (a $|k|$ ramp) is essential**, not optional.
- Gridding's three fingerprints — blur/shading, side-lobes, replication — map to density, kernel, and grid respectively, with matching cures.
- **Oversampling** kills aliasing and softens apodization; **deapodization** finishes the job.
- **Kaiser–Bessel ≫ triangular** for the same width; the **NUFFT** packages the whole pipeline and gives reusable forward/adjoint operators.

---

## 13. Lab 4 in one glance (worked in the notebook)

1. **Radial sampling pattern** — build the golden-angle trajectory; Nyquist needs $N_\text{spokes}=\tfrac{\pi}{2}N$ (≈ **604** spokes for a 384² matrix).
2. **Basic gridding** — `grid1` triangular kernel; without compensation the recon is a bright central blob.
3. **Density compensation** — the $|k|$ ramp brings the image into focus; oversampling/deapodization are second-order for this narrow kernel.
4. **Oversampling** — grid at 2×, IFFT, `decimate2d` (crop) back to the FOV.
5. **Deapodization** — get $c(x)$ by gridding a delta (or analytically $\operatorname{sinc}^2$); divide it out.
6. **NUFFT** — reconstruct with `sigpy` ± density compensation; its Kaiser–Bessel kernel beats the triangular gridding markedly.

---

## 🧠 Self-test questions

1. Why can't you reconstruct radial or spiral data by calling `ifft2` directly?
2. State the Fourier-slice theorem in one sentence and link a radial spoke to a CT projection.
3. Plain backprojection is blurry. In k-space terms, *why* — and what filter fixes it?
4. What is the radial density-compensation weight as a function of $|k|$, and where does the $\propto|k|$ come from geometrically?
5. Match each gridding fingerprint (blur/shading, side-lobes, replication) to its k-space cause and its cure.
6. Why does oversampling the Cartesian grid reduce *both* aliasing and apodization?
7. What is deapodization, and how do you obtain the deapodization function by "gridding a delta"?
8. Why is the Kaiser–Bessel kernel preferred over a triangular one of the same width?
9. For a 256² matrix, roughly how many radial spokes give Nyquist sampling? Show the formula.
10. Name two things a NUFFT operator gives you that a one-shot gridding recon does not.

<details>
<summary><b>Answer key</b> (try first, then peek)</summary>

1. The DFT/`ifft2` is defined only for samples on a uniform rectangular grid; non-Cartesian samples sit between grid points, so there is no array to invert. You must grid, DFT along the trajectory, or backproject.
2. *The 1-D Fourier transform of a projection at angle $\phi$ equals the radial k-space line at angle $\phi$.* A radial MRI spoke is therefore the Fourier transform of a CT projection.
3. Radial spokes all cross the k-space centre, so low frequencies are vastly over-sampled relative to the edge → low-frequency over-emphasis = blur. A ramp filter $\rho(k)\propto|k|$ re-balances the density.
4. Weight $\propto|k|$ (rising from $\approx1/N$ at the centre to $1$ at the edge). Geometrically, the sample density of spokes falls as $1/|k|$ (samples crowd at the centre), so the compensating weight is its inverse, $|k|$.
5. Non-uniform **density** → blur/shading → density compensation. Finite **kernel** → apodization + side-lobes → deapodization (+ better kernel). Finite **grid** → replication/aliasing → oversampling.
6. A larger grid means finer image-domain sampling: aliased copies are pushed outside the FOV, and the steep apodization roll-off falls outside the cropped region — so both effects shrink within the kept FOV.
7. Dividing the reconstructed image by $c(x)=\mathcal F^{-1}\{\text{kernel}\}$ to cancel the kernel-induced intensity roll-off. Numerically: grid a unit sample at $k=0$, inverse-FFT — the result *is* $c(x)$.
8. For the same width it concentrates far more energy in the main lobe and has much lower side-lobes, so it causes less aliasing and a cleaner image (the windowed-sinc that best approximates the impractical infinite sinc).
9. $N_\text{spokes}=\tfrac{\pi}{2}N \approx 1.57N$. For $N=256$: $\tfrac{\pi}{2}\times256 \approx \mathbf{402}$ spokes.
10. (i) Fast **forward and adjoint** operators reusable inside iterative/deep-learning reconstructions; (ii) built-in good kernel + oversampling + deapodization (better image quality), plus differentiability for learned recon.

</details>

---

## 📚 Further reading

- **Lauterbur P.** *Image Formation by Induced Local Interactions: Examples Employing Nuclear Magnetic Resonance.* Nature **242**, 1973. (the original radial/backprojection MRI experiment)
- **Bracewell R.** Fourier-slice theorem, 1956.
- **Jackson JI et al.** *Selection of a convolution function for Fourier inversion using gridding.* IEEE TMI, 1991.
- **Fessler JA, Sutton BP.** *Nonuniform Fast Fourier Transforms Using Min-Max Interpolation.* IEEE TSP **51**(2), 2003.
- **Kak & Slaney.** *Principles of Computerized Tomographic Imaging*, 1988. (ramp filter, FBP)
- NUFFT toolboxes: `sigpy`, `torchkbnufft`, `BART`, `gpuNUFFT`, Fessler IRT.

---

*Next up — Day 24: Image-Space Parallel Imaging (SENSE). From one coil to many.*
