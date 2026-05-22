# L10 — k-Space Encoding, Resolution, Gibbs Ringing & Aliasing

> **Source:** MRI1 Lecture 10 (Frederik Laun, FAU Erlangen-Nürnberg, WS 23/24)
> **Module:** 02 — k-Space & Image Formation
> **Day:** 12 of the ROADMAP

This is a meaty lecture. It picks up from the k-space *formalism* (lecture 9) and works out the **two practical consequences** of how we actually sample k-space in a real scanner:

1. We can only sample a **finite region** of k-space → that gives us a *finite resolution* and a phenomenon called **Gibbs ringing**.
2. We can only sample on **1-dimensional trajectories** (lines) with finite spacing Δk → that gives us a **field of view (FoV)** and a phenomenon called **aliasing** (wrap-around).

Both of these are described by the same beautifully clean formalism: a **transfer function** $\tilde{T}(\mathbf{k})$ in k-space, whose inverse Fourier transform is the **point spread function (PSF)** in image space. The PSF tells you everything about what your image will actually look like.

---

## 🧭 Where we are in the big picture

We've already established (lecture 9):

$$
\tilde{S}(\mathbf{k}) = \int d^3r \; M_+'(\mathbf{r},t) \cdot \exp(-i \, 2\pi \, \mathbf{k} \cdot \mathbf{r})
$$

The measured MR signal at a point in k-space is the spatial Fourier transform of the transverse magnetization. If we measured all of k-space perfectly, we'd just inverse-FT and be done.

But we don't. Two limitations:

| Limitation | What it means physically | What it does to the image |
|---|---|---|
| Limitation 1 | We sample on lines spaced by Δk (not continuously) | **Aliasing / wrap-around** — FoV = 1/Δk |
| Limitation 2 | We sample only out to ±k_max (not infinitely) | **Finite resolution** + **Gibbs ringing** |

Both are captured by writing

$$
\tilde{S}_\text{MRI}(\mathbf{k}) = \tilde{S}(\mathbf{k}) \cdot \tilde{T}(\mathbf{k})
$$

and Fourier transforming this gives the *measured* image as a **convolution** with the PSF:

$$
S_\text{MRI}(\mathbf{r}) = S(\mathbf{r}) \,*\, T(\mathbf{r}),
\quad \text{where} \quad T(\mathbf{r}) = \mathcal{F}^{-1}\{\tilde{T}(\mathbf{k})\}.
$$

This is the central equation of the lecture. Everything else is plugging in specific $\tilde{T}$'s and seeing what happens.

---

## 10.6 — Finite k-space sampling (Limitation 2)

### The setup

Cartesian sampling out to ±k_max in both directions ⇒ the transfer function is a 2D **boxcar**:

$$
\tilde{T}(\mathbf{k}) = \Pi(k_x, 0, 2k_\text{max}) \cdot \Pi(k_y, 0, 2k_\text{max})
$$

Take the inverse Fourier transform of a boxcar and you get a **sinc**:

$$
T(\mathbf{r}) = \mathcal{F}^{-1}\{\tilde{T}(\mathbf{k})\} = \mathrm{sinc}(2\pi x \, k_\text{max}) \cdot \mathrm{sinc}(2\pi y \, k_\text{max}) \cdot (2k_\text{max})^2
$$

So **the PSF for Cartesian MRI is a 2D sinc function**. This is the single most important fact in this lecture.

### Point source illustration

If you scan a single delta-function point at $(x_0, y_0)$:

$$
S(\mathbf{r}) = \delta_D(x-x_0)\,\delta_D(y-y_0)
$$

then what you actually image is

$$
S_\text{MRI}(\mathbf{r}) = \mathrm{sinc}(2\pi(x-x_0) k_\text{max}) \cdot \mathrm{sinc}(2\pi(y-y_0) k_\text{max}) \cdot (2k_\text{max})^2
$$

— a sinc centered at the true location, **not** a delta. The point is "spread out in space" — hence *point spread function*.

Three things change with $k_\text{max}$:

- **Larger $k_\text{max}$** ⇒ narrower central peak ⇒ better resolution.
- **Larger $k_\text{max}$** ⇒ faster oscillation of the side lobes.
- **But:** the side lobes still drop only as $1/x$ — even at very high $k_\text{max}$, the ringing is "far-ranging."

### Intuition: what k-space center vs outer regions encode

A summary slide from earlier in the course is worth memorizing:

| Region of k-space | Encodes |
|---|---|
| Center (low spatial frequencies) | **Contrast** — overall brightness, smooth variations |
| Outer regions (high spatial frequencies) | **Resolution** — edges, fine detail |

If you only kept the center, you'd get a blurry but correctly-contrasted image. If you only kept the outer parts, you'd get edges with no contrast.

---

## Resolution: three competing definitions

The lecture is honest about this: "resolution" doesn't have a unique mathematical definition. Three reasonable choices for the sinc PSF $T(x) = 2k_\text{max} \, \mathrm{sinc}(2\pi x \, k_\text{max})$:

### Definition 1: First zero crossing of the PSF

Solve $T(\Delta x) = 0$:

$$
\sin(2\pi \Delta x \, k_\text{max}) = 0
\;\Rightarrow\; 2\pi \Delta x \, k_\text{max} = \pi
\;\Rightarrow\; \boxed{\Delta x = \frac{1}{2 k_\text{max}}}
$$

This is the textbook formula for Cartesian MRI resolution.

### Definition 2: Integral / "height" of the PSF

$$
\Delta x = \frac{1}{T(0)} \int_{-\infty}^{\infty} T(x)\,dx
$$

Motivation: if the PSF is a perfect rectangle of width $\Delta x$ and height $1/\Delta x$, this formula correctly returns $\Delta x$. It generalizes to *any* PSF, which is useful for non-Cartesian sampling.

For the sinc PSF, using $\int_{-\infty}^{\infty}\mathrm{sinc}(x)\,dx = \pi$, this also gives $\Delta x = 1/(2k_\text{max})$.

### Definition 3: Full Width at Half Maximum (FWHM)

Solve $0.5 \cdot T(0) = T(\Delta x / 2)$:

$$
\frac{1}{2} = \mathrm{sinc}(2\pi \cdot \tfrac{\Delta x}{2} k_\text{max})
$$

Numerically, $\Delta x \approx \frac{1}{1.65 \, k_\text{max}}$. This is **larger** (worse) than $1/(2 k_\text{max})$.

### Which to use?

In Cartesian MRI, **Definition 1 wins** in textbooks: $\Delta x = 1/(2 k_\text{max})$. But when you change PSF shape (e.g. by applying a Hanning filter — see exercise 10.1), the three definitions diverge and Definition 2 is often the most honest.

The $k_\text{max}^{-1}$ scaling is the deep truth: **higher spatial frequencies ⇒ better resolution**.

---

## 11.7 — Gibbs ringing

What happens when the "true" image has a sharp edge?

Take a 1D Heaviside step $S(x) = S_0 \, \Theta(x)$. Convolve with the sinc PSF:

$$
S_\text{MRI}(x) = S(x) * T(x) = S_0 \cdot 2 k_\text{max} \int_0^{\infty} dx' \, \mathrm{sinc}(2\pi (x-x') k_\text{max})
$$

After a clean variable substitution this becomes a **sine integral**:

$$
S_\text{MRI}(x) = \frac{S_0}{\pi} \left( \frac{\pi}{2} + \mathrm{Si}(2\pi k_\text{max} x) \right)
$$

where $\mathrm{Si}(y) = \int_0^y \mathrm{sinc}(y') dy'$.

### The big takeaway

The result oscillates near the edge with amplitude **~9% overshoot** that **does not go away** as $k_\text{max}$ increases. Only the *spatial extent* of the ringing shrinks. This is the **Gibbs phenomenon** familiar from Fourier series — every truncated Fourier expansion of a discontinuous function does this.

**Clinical concern:** a small lesion near a high-contrast boundary can be obscured (or mimicked!) by Gibbs ringing.

### Mitigation: filters

Multiply $\tilde{S}_\text{MRI}(\mathbf{k})$ by a smooth filter $\tilde{F}(\mathbf{k})$ (e.g. the Hanning filter) that gently tapers k-space data toward zero at the edges:

$$
\tilde{F}_\text{Hanning}(k) = 0.5 \left( 1 + \cos\frac{\pi k}{k_\text{max}} \right) = \cos^2 \frac{\pi k}{2 k_\text{max}}
$$

This reduces ringing at the cost of resolution (the convolution of two smoothing functions is wider than either). Exercise 10.1 walks through this trade-off quantitatively.

---

## 11.8 — Aliasing artifacts (Limitation 1)

Now consider the *other* limitation: we sample k-space on lines spaced by $\Delta k$ in the phase direction.

### The Dirac comb appears

If we model the sampling as a 1D Dirac comb in $k_\text{phase}$ with spacing $\Delta k$:

$$
\tilde{T}_\text{phase}(k) = \mathrm{const} \cdot \sum_{n=-\infty}^{\infty} \delta_D(k - n \Delta k)
$$

Fixing the constant by requiring **unit density** (so that locally $\tilde{T}$ behaves like 1 in any $\Delta k$-wide region) gives:

$$
\tilde{T}(k) = \Delta k \cdot \sum_{n=-\infty}^{\infty} \delta_D(k - n \Delta k)
$$

### Fourier transform of a Dirac comb is a Dirac comb

The key result the lecture derives carefully (using the periodicity of the comb and a Fourier-series argument):

$$
\mathcal{F}\left\{ \Delta k \sum_n \delta_D(k - n\Delta k) \right\} = \sum_{m=-\infty}^{\infty} \delta_D\!\left( x - \frac{m}{\Delta k} \right)
$$

**The Fourier transform of a Dirac comb is another Dirac comb**, with spacing $1/\Delta k$.

### Consequence: replicated images

Convolving the true image with a Dirac comb produces *infinite copies* of the image:

$$
S_\text{MRI}(x) = \sum_{m=-\infty}^{\infty} S\!\left( x - \frac{m}{\Delta k} \right)
$$

The spacing of the copies is:

$$
\boxed{\mathrm{FoV} = \frac{1}{\Delta k}}
$$

This is the **field of view**. As long as the true object fits inside one FoV, the copies sit nicely outside the displayed image and you don't see them. **If the object is larger than the FoV, the copies overlap into the visible region** — this is the **wrapping / aliasing artifact**.

---

## 11.9 — Final remarks on k-space imaging

### Wrapping is a *phase*-direction problem

In Cartesian MRI:
- The **read** direction is sampled densely on each line — $\Delta k_\text{read}$ is essentially zero. You can choose FoV_read arbitrarily in reconstruction.
- The **phase** direction is where the $\Delta k$ that causes aliasing lives.

So **wrap-around only happens along the phase direction**. Exercise 10.2 makes this concrete: when you shrink FoV_phase below the object size, copies pile in; when you shrink FoV_read, you just see a clipped image with no aliasing.

### Phase direction is "the bad direction"

Because aliasing is a phase-direction issue and so are motion artifacts (e.g. head motion → ghosts in phase direction) and pulsation artifacts from arteries, radiologists pick the phase direction strategically — e.g. they put the phase direction perpendicular to the spine to push pulsation ghosts off the anatomy of interest.

### 2D vs 3D sequences

- **2D sequence:** slice-select to excite one thin slice + 2D k-space encoding (one read, one phase).
- **3D sequence:** excite a thick slab (or globally) + 3D Cartesian k-space encoding (one read, **two** phase). Both encoded directions can show aliasing.

### Caveat: non-Cartesian sampling doesn't have unit density

Radial sampling, for example, has spoke density that falls as $1/k$ — the spokes are dense at the center and sparse at the edges. This is *not* unit density, and standard FFT recon won't work directly. Reconstruction algorithms have to compensate for this (density compensation), and that's the topic of CMRI lectures 4 and 7.

---

## 🎯 Self-Test Questions

1. **Conceptual:** A point source produces a sinc-shaped image. Why don't we see this sinc pattern around every bright pixel of a real brain scan?
2. **Computation:** If your scanner samples up to $k_\text{max} = 2.5 \, \text{mm}^{-1}$, what is the Cartesian resolution by Definition 1? By Definition 3 (FWHM)?
3. **Computation:** If your phase-direction k-space step is $\Delta k = 1 \, \text{cm}^{-1}$, what is the FoV in that direction? If the patient is 30 cm wide there, what do you see?
4. **Trade-off:** A Hanning filter reduces Gibbs ringing. Why does it also reduce resolution? Answer using the convolution theorem.
5. **Why isn't the read direction aliased?** In your own words.
6. **Phase = bad direction:** Give two reasons (other than aliasing) that radiologists carefully choose the phase direction.
7. **Gibbs amplitude:** Why is the ~9% Gibbs overshoot a *constant*, independent of $k_\text{max}$? (Hint: think about what the sine integral approaches.)
8. **Dirac comb FT:** State, without computing it, why the FT of a comb-spaced-by-$\Delta k$ must again be a comb. (Hint: periodicity and discreteness are dual under FT.)
9. **3D sequence aliasing:** A 3D Cartesian sequence has two phase-encode directions. Can aliasing happen in both? Can it happen in the read direction?
10. **Application:** You see a faint structure at the edge of a high-contrast region in a brain scan. Three possibilities — real lesion, Gibbs ringing, partial-volume effect. How might you tell?

---

## 📌 Key Takeaways (one-line each)

- Real MRI samples a **finite, gridded region** of k-space.
- Finite extent ($\pm k_\text{max}$) ⇒ **sinc PSF** ⇒ resolution $\sim 1/(2 k_\text{max})$ + **Gibbs ringing** near edges (~9% constant overshoot).
- Discrete sampling ($\Delta k$) ⇒ **Dirac comb PSF** ⇒ image replication with period $\mathrm{FoV} = 1/\Delta k$ ⇒ **aliasing** if object exceeds FoV.
- Aliasing happens **only along the phase direction** in Cartesian MRI.
- The PSF formalism $S_\text{MRI} = S * T$ unifies both effects.
- Filters trade ringing for resolution. There is no free lunch.

---

## 📚 Further reading

- Bernstein, King, Zhou — *Handbook of MRI Pulse Sequences*, Chapter 13 ("Common Image Reconstruction Techniques") for Gibbs and partial-Fourier mitigation.
- Liang & Lauterbur — *Principles of MRI: A Signal Processing Perspective*, Chapter 5 for the rigorous PSF/FoV treatment.
- Nishimura — *Principles of Magnetic Resonance Imaging*, Sections 6.5–6.7.
- Veraart et al. (2016) NeuroImage — "Gibbs ringing in diffusion MRI" — a modern paper showing why ringing still matters in 2020s neuroimaging.
