# C07 — Non-Cartesian Parallel Imaging & Iterative Reconstruction

**Module 4 · Day 27 · CMRI Lecture 7 (Parallel Imaging III)**
**Source:** Pruessmann et al., *Advances in Sensitivity Encoding With Arbitrary k-Space Trajectories*, MRM 46:638–651 (2001) · Maier et al., *CG-SENSE revisited*, MRM 2021

---

## The one-paragraph version

Cartesian SENSE undoes aliasing pixel-by-pixel because skipping k-space lines produces clean, predictable fold-over. Radial and spiral trajectories don't: their undersampling produces incoherent streaks and swirls with no neat per-pixel structure, so image-domain unfolding breaks down. The fix is to stop "undoing aliasing" and instead pose reconstruction as a **linear inverse problem** — find the image `u` whose simulated multi-coil k-space best matches the measured data, i.e. minimise `‖Ku − g‖²`. The encoding operator `K = F_Ω·C` is far too large to invert directly (≈139 GB for a realistic radial scan), so we solve it **iteratively** using only applications of `K` and its adjoint `Kᴴ`. Gradient descent works but crawls; **conjugate gradient** (CG-SENSE) converges in ~20–30 iterations and is the workhorse non-Cartesian parallel-imaging method.

---

## 1 · Why non-Cartesian breaks Cartesian SENSE

Recall Cartesian SENSE (C05): undersampling by R folds R equally-spaced pixels onto each location in the reduced-FOV image. Each coil sees a different weighted sum (via its sensitivity), giving a small `nc × R` system you solve **independently at every pixel**. The point-spread function is a handful of sharp, equidistant peaks — well-behaved.

With **arbitrary trajectories** (radial, spiral, stochastic), reducing sampling density still lets you cover the same k-space area, but the PSF becomes a continuous, ring-shaped structure. Aliasing is spread everywhere and is not the superposition of a few discrete pixels. There is no localised system to solve per pixel — you must reconstruct **the whole image at once**.

**The visual punchline (lecture slides 4–10):**

| Sampling | Aliasing appearance |
|---|---|
| Cartesian R=2 | coherent fold-over (anatomy copied, shifted by FOV/2) |
| Radial | incoherent **streaks** |
| Spiral | structured swirls — "the value of pixels & intensity is not obvious" |

Incoherent aliasing is actually *good news* for later (compressed sensing, C08, exploits it), but it kills the simple unfolding trick.

---

## 2 · The encoding operator

Discretising the multi-coil signal equation (slides 11–12):

```
g_k(kx,ky) = Σ_x Σ_y  c_k(x,y) · e^(−i(kx·x + ky·y)) · u(x,y)
```

- `c_k` — sensitivity of coil *k*
- `e^(−i…)` — the spatial-encoding Fourier kernel evaluated on the trajectory Ω
- `u` — the unknown image

Stacking all coils and all sampled k-space points gives the compact form:

```
g = F_Ω · C · u  =  K · u
```

where `C` applies the coil sensitivities (image → per-coil images) and `F_Ω` is the (non-uniform) Fourier transform onto the sampled trajectory. **`K` is the forward/encoding operator.** Reconstruction = solving `Ku = g` for `u`.

In code (this notebook), `K` and `Kᴴ` are just a few NUFFTs plus pointwise multiplies — see `forward()` / `adjoint()`. We never build the matrix.

---

## 3 · Reconstruction as an inverse problem — and why direct inversion fails

The direct solution is the pseudoinverse:

```
u = (Kᴴ K)⁻¹ Kᴴ g
```

But `K` is gigantic. The lecture's worked example: **N=256, 8 coils, 256 radial spokes, 256 samples/spoke → K is 528,384 × 65,536 ≈ 139 GB.** You cannot store it, never mind invert it.

**Key insight:** iterative solvers only need to *apply* `K` and `Kᴴ` to a vector — they never need the explicit matrix. Each application is cheap (NUFFTs). That is what makes large-scale MRI reconstruction tractable.

---

## 4 · Gradient descent

Minimise the data-consistency cost `f(u) = ‖Ku − g‖²`. The gradient (Exercise 1, slide 21):

```
∂/∂u ‖Ku − g‖²  =  2 Kᵀ(Ku − g)
```

*(derivation: chain rule on `f = Σ rᵢ²` with `r = Ku − g`, giving `Σ 2rᵢ Kᵢᵀ = 2Kᵀr`; full proof in the notebook. For complex data, `Kᵀ → Kᴴ`.)*

Update rule (slide 16), `u₀ = 0`, step size `t > 0`:

```
u_{n+1} = u_n − t · 2Kᵀ(K u_n − g)
        = u_n − t · 2 (Kᴴ K u_n − Kᴴ g)
```

**Step size matters.** Convergence requires roughly `t ≤ 1/L` with `L` = largest eigenvalue of `KᴴK` (found via a few power iterations). The lab's sweep makes this concrete:

- too small (`t=0.005`) → underfit, blurry, hasn't reached the minimum;
- well-chosen (`t≈0.01`) → steady convergence;
- too large (`t=0.1`) → **diverges** ("couldn't make the image at all").

**Is GD stuck in local minima?** No — `‖Ku − g‖²` is a convex quadratic, so there's a single global minimum. The annotation on slide 15 is right: "Not here, it's a convex problem."

**GD's weakness:** consecutive steepest-descent directions are orthogonal (`p_kᵀ p_{k+1} = 0`), so in a narrow, elongated cost valley it zig-zags and converges slowly.

---

## 5 · Conjugate gradient (CG-SENSE)

CG fixes the zig-zag by choosing search directions that are **K-orthogonal (conjugate)** rather than orthogonal:

```
p_kᵀ K p_{k+1} = 0
```

Properties (slides 20–22):
- Converges to the **exact** solution in `n` steps (`n` = system size) → interpretable as a direct method.
- In practice used **iteratively**: required tolerance is reached after `it ≪ n` (typically 20–30 for CG-SENSE).
- **Unstable to perturbations** (e.g. noise) — important caveat, see §7.
- ~20 lines of code; available as `scipy.sparse.linalg.cg`, in MATLAB, etc.

The CG recurrences (slide 21):

```
r_k   = g − K u_k                                  # residual
p_k   = r_k − Σ_{i<k} (p_iᵀ K r_k / p_iᵀ K p_i) p_i # conjugate search direction
u_{k+1} = u_k + α_k p_k                             # update
α_k   = (p_kᵀ r_{k−1}) / (p_kᵀ K p_k)              # step length
```

**CG needs a symmetric positive-definite matrix.** `K` itself isn't, so we run CG on the **normal equations** (slide 23):

```
Kᴴ K u = Kᴴ g
```

Now `A = KᴴK` is Hermitian PSD, the right-hand side `b = Kᴴg` is just the gridding image, and `A` is applied via `normal_op` (forward then adjoint). This *is* CG-SENSE (Pruessmann 2001): the algorithm's inner loop alternates FT-to-image (gridding) and FT-to-k-space (de-gridding), wrapped by the central CG process across all coils.

**Preconditioning** (`MKu = Mg`) can speed convergence; the "perfect" preconditioner `M = K⁻¹` is useless because we don't have it — the whole point was that `K` is uninvertible. Practical preconditioners (e.g. density-compensation/intensity correction) approximate it.

---

## 6 · Results & convergence behaviour

- **GD vs CG, same data:** CG reaches a low error in a handful of iterations; GD plods (notebook §6 reproduces the lab's Figure 7 — CG curve drops far steeper and lower).
- **Acceleration:** as R increases (fewer spokes / interleaves), `KᴴK` becomes more ill-conditioned, recon quality degrades and noise rises. Radial holds up better than spiral at high R; at R≈8 spiral, "acceleration is too high."
- **Trajectory comparison:** radial/spiral undersampling spreads residual artefact incoherently, unlike Cartesian's coherent fold-over.

---

## 7 · Noise propagation (Exercise 3 / lab 3.4)

Because CG solves the *exact* system, it faithfully fits noise too. Two observations:

1. Higher acceleration ⇒ more amplified noise (ill-conditioning).
2. **Reconstruct pure noise** (random data instead of real k-space) and run many iterations: the error does *not* shrink. Instead the result, viewed in k-space, lights up exactly along the **sampled trajectory** — the radial spokes become visible. CG has amplified noise along precisely the directions the trajectory constrains.

**Consequence:** CG-SENSE is run for a *limited* number of iterations — early stopping acts as implicit regularisation. Production pipelines add an explicit regulariser:

```
û = argmin_u ‖Ku − g‖² + λ R(u)
```

with `R` = Tikhonov (L2), total variation, wavelet sparsity (→ compressed sensing, C08), or a **learned prior** (→ unrolled networks, Module 5). This `data-consistency + regulariser` template is the bridge to everything that follows.

---

## 8 · Reproducibility footnote

CG-SENSE was the subject of the first **ISMRM Reproducible Research Study Group** challenge (2019): 13 groups re-implemented Pruessmann 2001 from the paper alone, on shared radial brain & cardiac data. Results were collated in **Maier et al., MRM 2021** — including consolidated reference Python and MATLAB implementations. The challenge data (`rawdata_brain_radial_*.h5`, BART array conventions) is what the optional §7 of the notebook loads.

---

## Key takeaways

1. **Non-Cartesian ⇒ no per-pixel unfolding.** Reconstruct the whole image at once as an inverse problem.
2. **`g = Ku`, `K = F_Ω C`.** Never build `K`; only apply `K` and `Kᴴ`.
3. **Direct inversion is impossible** (139 GB) → iterative solvers.
4. **GD:** simple, convex, but slow (orthogonal zig-zag); step size `t ≤ 1/L` or it diverges.
5. **CG-SENSE:** conjugate directions, ~20–30 iterations, run on the normal equations `KᴴKu = Kᴴg`.
6. **CG is noise-unstable** → early stopping / explicit regularisation; this motivates Modules 5 (learned recon) and C08 (compressed sensing).

---

## Self-test — answers

1. **Why no per-pixel unfolding for radial/spiral?**
   Their undersampling PSF is a continuous ring-shaped structure, not a few discrete equidistant peaks, so aliasing isn't a sum of a small known set of pixels per location. There's no small per-pixel system to invert — you must solve for the full image jointly.

2. **What lets iterative solvers avoid the 139 GB matrix?**
   They only require matrix–vector *products* `Ku` and `Kᴴg`, each computable as a few NUFFTs + pointwise multiplies. The explicit matrix is never formed or stored.

3. **GD update + too-large step?**
   `u_{n+1} = u_n − t·2Kᴴ(Ku_n − g)`. If `t` exceeds ~`2/L`, the update overshoots the quadratic's curvature and the iterates diverge — exactly the lab's `t=0.1` row, where the image "blows up" into a useless bright blob.

4. **Conjugate directions vs orthogonal?**
   Conjugate means `p_kᵀ K p_{k+1} = 0` (orthogonal in the metric defined by `K`). Unlike steepest descent's plain-orthogonal directions (which re-undo progress and zig-zag in elongated valleys), conjugate directions never interfere, so each step makes irreversible progress and the minimum is reached in at most `n` steps.

5. **Why the normal equations?**
   CG requires a symmetric (Hermitian) positive-definite matrix. `K` is rectangular and not PSD; `KᴴK` is Hermitian PSD, so CG applies. The RHS `Kᴴg` is the gridding image.

6. **Exact in `n` steps but we stop at 20–30 — why?**
   (i) The required image-quality tolerance is reached long before iteration `n` (`it ≪ n`). (ii) CG is unstable to noise — running to "convergence" amplifies noise — so early stopping is a deliberate regulariser. (Floating-point round-off also erodes the exact-in-`n`-steps guarantee.)

7. **CG on pure noise for 500 iterations — what appears in k-space, and what does it mean?**
   The radial **sampling trajectory** emerges (k-space lights up along the spokes). It shows CG amplifies noise along the sampled directions rather than reducing error — concrete evidence of CG's perturbation instability and the reason for early stopping / regularisation.

---

## Recommended reading
- Pruessmann, Weiger, Börnert, Boesiger. *Advances in Sensitivity Encoding With Arbitrary k-Space Trajectories.* MRM 46:638–651 (2001). — the CG-SENSE paper.
- Maier et al. *CG-SENSE revisited: results from the first ISMRM reproducibility challenge.* MRM (2021). — modern reference implementations.
- Hestenes & Stiefel (1952) — the original conjugate-gradient method.
- Shewchuk, *An Introduction to the Conjugate Gradient Method Without the Agonizing Pain* — the friendliest CG tutorial there is.
