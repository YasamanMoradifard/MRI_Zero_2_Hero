# L09 — k-Space Encoding

**Source:** MRI1 WS23-24, Lecture 9 (Frederik Laun, FAU Erlangen)
**Chapter:** 10 (k-Space Encoding) — covers sections 10.1–10.5
**Module:** 2 — k-Space & Image Formation

---

## 🎯 Why this lecture is a big deal

This is **the** lecture of MRI1. Everything before it (B₀, RF excitation, gradients, Faraday induction) was setup; everything after it (sequences, contrast, advanced recon) is consequence. The big idea is shockingly simple once it clicks:

> **The MR signal you measure is the Fourier transform of the image you want.**
> The gradients let you choose which point in Fourier space (k-space) you're sampling, and the image is recovered by a single FFT.

This single sentence makes MRI image reconstruction look almost trivial. The rest of the lecture is unpacking what that sentence really means and what its physical limitations are.

---

## 1. The setup: extending frequency encoding to 2D and 3D

The "gentle introduction" earlier in the course used 1D frequency encoding: apply a gradient, the Larmor frequency becomes position-dependent, do a frequency analysis on the detected signal, and you get the 1D distribution of magnetization. Lovely, but how do you extend this to 2D or 3D?

The k-space formalism is the answer. It's not a new physical principle — it's a **reframing** of the same idea that generalizes cleanly to arbitrary dimensions and arbitrary gradient waveforms.

### The total magnetic field with gradients

The ideal gradient field everywhere points along **z** (this is the linearized "ideal gradient" assumption — real gradient fields have small components in x and y too, but we ignore those):

$$\mathbf{B}_\text{G}(\mathbf{r}, t) = (\mathbf{G}(t) \cdot \mathbf{r})\, \mathbf{e}_z$$

So the total field is:

$$\mathbf{B}(\mathbf{r}, t) = \mathbf{B}_0 + \mathbf{B}_\text{G}(\mathbf{r}, t) = \begin{pmatrix} 0 \\ 0 \\ B_0 + xG_x(t) + yG_y(t) + zG_z(t)\end{pmatrix}$$

The whole vector field still points along z — only its magnitude varies with position and time. This is why the magnetization keeps precessing around z (just at a position-dependent rate).

### The position-dependent Larmor frequency

$$\omega(\mathbf{r}, t) = \gamma B(\mathbf{r}, t) = \omega_0 + \omega_\text{G}(\mathbf{r}, t)$$

with $\omega_\text{G}(\mathbf{r}, t) := \gamma\, \mathbf{G}(t) \cdot \mathbf{r}$.

### The phase accumulated by gradients

In the rotating reference frame (RRF) rotating at $\omega_0$, the $B_0$ contribution vanishes and only the gradient-induced rotation matters. The phase accumulated between $t=0$ (just after excitation) and time $t$ is:

$$\varphi_\text{G}(\mathbf{r}, t) = \int_0^t \omega_\text{G}(\mathbf{r}, t')\, dt' = \gamma \int_0^t \mathbf{G}(t') \cdot \mathbf{r}\, dt'$$

So the transverse magnetization in the RRF, at position $\mathbf{r}$ and time $t$, is:

$$M'_+(\mathbf{r}, t) = M_\perp(\mathbf{r}, 0)\, e^{i\varphi_\text{G}(\mathbf{r}, t)}$$

(Relaxation is neglected throughout this lecture — it's tracked as a separate concern.)

### The signal equation (simplified)

The full signal equation from the Faraday-induction chapter, after dropping the coil sensitivity profile and the $\omega_0$ prefactor (we don't care about physical units here), is:

$$S(t) = \int d\mathbf{r}^3\, M'_+(\mathbf{r}, t)$$

**Physical interpretation:** the signal is the sum (integral) over all the little transverse magnetization arrows in the sample. Each arrow has been rotated by the gradient phase $\varphi_\text{G}$. The receiver coil adds them all up coherently.

---

## 2. The k-vector: where the magic happens

Here's the clever substitution. Define the **k-vector** as:

$$\boxed{\,\mathbf{k}(t) := \frac{\gamma}{2\pi} \int_0^t \mathbf{G}(t')\, dt'\,}$$

Units: $[\mathbf{k}] = \text{m}^{-1}$ — it's a **wave vector**, just like in optics or solid-state physics.

With this definition, the gradient phase becomes:

$$\varphi_\text{G}(\mathbf{r}, t) = 2\pi\, \mathbf{k}(t) \cdot \mathbf{r}$$

Plugging this into the signal equation:

$$S(t) = \int d\mathbf{r}^3\, M_\perp(\mathbf{r}, 0)\, e^{i 2\pi \mathbf{k}(t) \cdot \mathbf{r}}$$

After relabeling $\tilde{S}(-\mathbf{k}(t)) := S(t)$ (a sign convention to align with standard Fourier conventions — see the "sign convention" box below), we get **the** key equation:

$$\boxed{\,\tilde{S}(\mathbf{k}) = \int d\mathbf{r}^3\, M'_+(\mathbf{r}, 0)\, e^{-i 2\pi \mathbf{k} \cdot \mathbf{r}}\,}$$

**This is just the Fourier transform of the transverse magnetization.** The signal we measure as we move through k-space *is* the Fourier transform of the image we want.

### Recipe for MR image formation

1. Excite the spins with an RF pulse.
2. Apply gradients $\mathbf{G}(t)$ → this defines a **trajectory** $\mathbf{k}(t)$ through k-space.
3. While the trajectory is being traversed, sample the signal $\tilde{S}(\mathbf{k}(t))$.
4. Inverse Fourier transform → image $M'_+(\mathbf{r}, 0)$.

### k-space velocity

A nice intuition: the gradient *is* the velocity through k-space.

$$\mathbf{v}_k(t) = \dot{\mathbf{k}}(t) = \frac{\gamma}{2\pi} \mathbf{G}(t)$$

So when you turn on a positive $G_x$, you start moving to the right in k-space. When you turn it off, you stop where you are. Negative $G_x$ moves you left. This is the mental model that pulse-sequence designers use to navigate k-space.

### Sign convention sidebar

The minus sign in $\tilde{S}(-\mathbf{k}) := S(t)$ is an artifact of having defined the Bloch equation with $\dot{\mathbf{M}} = \gamma \mathbf{B} \times \mathbf{M}$ (left-handed-looking) instead of the textbook $\dot{\mathbf{M}} = \gamma \mathbf{M} \times \mathbf{B}$. Different textbooks make different choices. Some also use $\mathbf{k}(t) := \gamma \int \mathbf{G}\, dt'$ without the $2\pi$, which leads to $\tilde{S}(\mathbf{k}) = \int M\, e^{-i \mathbf{k} \cdot \mathbf{r}}\, d\mathbf{r}^3$. Always check which convention a paper or codebase is using. **This course uses the $2\pi$ convention throughout.**

---

## 3. Two unavoidable limitations

The equation $\tilde{S}(\mathbf{k}) = \mathcal{F}\{M'_+(\mathbf{r}, 0)\}$ is beautiful, but in practice we can't measure $\tilde{S}$ at every point in k-space:

### Limitation 1 — Sampling is along a 1D trajectory

Within a single "shot" (between one excitation and the next), the gradient waveform defines a single continuous curve through k-space. You can never "teleport" — you have to move along a path. Multi-shot acquisitions sample multiple paths, but each one is still 1D.

### Limitation 2 — Only a finite region of k-space is sampled

You start at $\mathbf{k}=0$ (no gradient has been applied yet). To reach high $|\mathbf{k}|$ takes time, and during that time the transverse magnetization is decaying due to T2/T2*. So in practice you can only sample some bounded region.

These two limitations are the source of basically every k-space-related artifact in MRI:
- **Limited region** → finite resolution + Gibbs ringing
- **Sparse trajectory** → aliasing, streaking, blurring

We'll quantify both via the **point spread function** (Section 5).

---

## 4. Building intuition for k-space

This is the conceptual heart of the lecture. The math is just notation; the *intuition* is what matters. Several complementary views:

### View A — k-space as "phase waves on the magnetization"

When you're at point $\mathbf{k}$ in k-space (i.e. when gradients have integrated to that value), every spin in the sample has been wound up by the phase

$$\varphi_\text{G}(\mathbf{r}) = 2\pi\, \mathbf{k} \cdot \mathbf{r}$$

This is a **plane wave in position space**:
- The wavelength is $\lambda_k = 1/|\mathbf{k}|$.
- The wave propagates along the direction of $\mathbf{k}$.

If $\mathbf{k} = 0$, the wave has infinite wavelength — every spin has the same phase ("inphase"). The signal is the sum of all arrows pointing the same direction → **maximum signal**. This is why **k-space center = average image intensity = contrast**.

If $|\mathbf{k}|$ is large, the wavelength is short and the arrows wind around many times across the sample. They cancel each other almost completely → **small signal**. But this small signal encodes the **high-frequency detail** of the image — i.e. **k-space edge = resolution**.

### View B — Worked 1D example

Take 9 spins evenly spaced from $x = 0$ to $x = 8$ mm, all with equal magnetization magnitude 1.

| Case | Wave vector $k$ | Wavelength $\lambda_k$ | Phase at $x = 8$ mm | Signal |
|------|---|---|---|---|
| No gradient | $0$ | $\infty$ | $0$ | $9$ (max) |
| $k = 1/16$ mm⁻¹ | small | $16$ mm | $\pi$ | $\approx i \cdot 5.03$ (imaginary, partial cancellation) |
| $k = 1/8$ mm⁻¹ | medium | $8$ mm | $2\pi$ | $1$ (almost full cancellation; only last unpaired spin survives) |

The notebook reproduces all three cases.

### View C — Point source in k-space ↔ plane wave in image space

A delta function at $\mathbf{k}_1$ in k-space inverse-Fourier-transforms to $e^{i 2\pi \mathbf{k}_1 \cdot \mathbf{r}}$ — a plane wave in image space, oriented along $\mathbf{k}_1$, with wavelength $1/|\mathbf{k}_1|$.

This is **the** symmetry of Fourier analysis: localized points in one domain ↔ extended waves in the other.

### View D — Center vs. edge of k-space

| Region | Role |
|---|---|
| k-space center ($\mathbf{k} \approx 0$) | **Contrast and brightness.** Recall $\tilde{S}(0) = \int M(\mathbf{r})\, d\mathbf{r}^3$ — literally the mean image intensity. |
| k-space periphery (large $\|\mathbf{k}\|$) | **Resolution and edges.** Encodes high spatial frequencies. |

You can confirm this experimentally: keep only the center 1/4 of k-space, you get a blurry-but-correctly-bright image. Keep only the outer part (notch out the center), you get an edge-detection image with completely wrong brightness.

---

## 5. The point spread function (PSF) — quantifying the limitations

The two limitations of section 3 motivate a clean formalism:

- The "true" k-space signal would be $\tilde{S}(\mathbf{k})$ on **all** of k-space.
- What we actually measure is $\tilde{S}_\text{MRI}(\mathbf{k}) = \tilde{S}(\mathbf{k}) \cdot \tilde{T}(\mathbf{k})$, where $\tilde{T}(\mathbf{k})$ is the **sampling function** — 1 where we sampled, 0 elsewhere.

By the convolution theorem:

$$\boxed{\,S_\text{MRI}(\mathbf{r}) = S(\mathbf{r}) \ast T(\mathbf{r})\,}$$

where $T(\mathbf{r}) = \mathcal{F}^{-1}\{\tilde{T}(\mathbf{k})\}$ is the **point spread function**.

**The PSF tells you how a point source in the true image gets smeared in the measured image.** Every artifact in MRI image acquisition (not reconstruction errors, but acquisition artifacts) can be traced to a non-ideal PSF.

### Convolution refresher

The convolution of an image $f$ with a kernel $g$ is:

$$f_\text{blur}(x) = (f \ast g)(x) = \int_{-\infty}^\infty f(x')\, g(x - x')\, dx'$$

If $g$ is a small boxcar function, convolution is blurring. If $f$ is a single point source, the convolution literally reproduces $g$ at the point's location — hence the name "point spread function" for $g = T(\mathbf{r})$.

### What does $T(\mathbf{r})$ look like for typical samplings?

- **Cartesian, fully sampled, rectangular box in k-space:** $T(\mathbf{r})$ is a 2D sinc — the source of **Gibbs ringing** (covered in detail next lecture).
- **Cartesian, undersampled (every other line):** $T(\mathbf{r})$ has aliasing replicas → ghosting.
- **Radial:** $T(\mathbf{r})$ has streaking artifacts in image space.

This formalism is the bridge to Module 4 (Advanced Reconstruction).

---

## 6. Sampling patterns: Cartesian and radial

### Building blocks: gradients move you in k-space

Recall: $\mathbf{k}(t) = (\gamma/2\pi) \int_0^t \mathbf{G}(t')\, dt'$. So:

- A positive $G_x$ pulse of duration $t_G$ moves you from $(0, 0)$ to $(k_\max, 0)$ where $k_\max = (\gamma/2\pi) G_1 t_G$.
- A negative $G_x$ pulse of the same duration moves you to $(-k_\max, 0)$.
- A positive $G_y$ pulse moves you to $(0, k_\max)$.
- Simultaneous $G_x$ and $G_y$ at equal amplitude moves you diagonally to $(k_\max, k_\max)$.
- $G_x$ at amplitude $G_1$ and $G_y$ at amplitude $0.5 G_1$ moves you to $(k_\max, 0.5 k_\max)$.

That's the whole vocabulary. Any k-space trajectory is built from these moves.

### Radial sampling

After each excitation, sample one radial spoke (a line through the origin) by turning on a gradient along that direction. Wait, excite again, sample the next spoke at a different angle. Repeat until you've covered all angles densely enough.

```
[excite] [spoke 1] [wait] [excite] [spoke 2] [wait] [excite] [spoke 3] ...
```

Pros: oversamples k-space center (good for contrast and motion robustness). Naturally circular k-space coverage.

Cons: reconstruction requires gridding/NUFFT (covered in Module 4), undersampling causes streak artifacts.

### Cartesian sampling

After each excitation, sample one horizontal line of k-space — a line **parallel** to the read direction at some fixed phase-encoding value. **Always start in the k-space center** (this matters for contrast — the center is acquired at a known echo time).

Typical sequence per excitation:
1. **Excite** (RF pulse with optional slice-selective gradient).
2. **Prephasing**: a negative $G_x$ + a positive $G_y$ to navigate from origin to $(-k_\max, k_y)$.
3. **Readout**: a positive $G_x$ while sampling continuously → walk along $k_x$ from $-k_\max$ to $+k_\max$ at fixed $k_y$.
4. **Wait, excite again, change $k_y$, repeat** until all phase-encoding lines are filled.

Pros: reconstruction is just an FFT — simple, fast, well-understood.

Cons: every line requires a full TR (repetition time). Slower than radial for many applications. Doesn't oversample the center.

### Read / phase / slice coordinate system

In Cartesian sampling there's a useful naming convention:
- **Read direction**: parallel to the lines you're sampling (the direction of $G_x$ during readout). Also called "frequency-encoding direction."
- **Phase direction**: perpendicular to the read direction in the imaging plane.
- **Slice direction**: perpendicular to the imaging plane (set by slice-selection gradient).

These don't have to align with the scanner's x/y/z — they're set by which physical gradients (or combinations) act as read/phase/slice. A "localizer" scan is acquired at the start to let the operator orient the imaging plane along anatomically meaningful directions.

---

## 7. Equations to remember

The five core equations of k-space encoding:

$$\mathbf{B}_\text{G}(\mathbf{r}, t) = (\mathbf{G}(t) \cdot \mathbf{r})\, \mathbf{e}_z \quad\text{(ideal gradient field)}$$

$$\mathbf{k}(t) := \frac{\gamma}{2\pi} \int_0^t \mathbf{G}(t')\, dt' \quad\text{(k-vector definition)}$$

$$\mathbf{v}_k(t) = \dot{\mathbf{k}}(t) = \frac{\gamma}{2\pi} \mathbf{G}(t) \quad\text{(k-space velocity = gradient)}$$

$$\varphi_\text{G}(\mathbf{r}, t) = 2\pi\, \mathbf{k}(t) \cdot \mathbf{r} \quad\text{(phase wave from gradient)}$$

$$\tilde{S}(\mathbf{k}) = \int d\mathbf{r}^3\, M'_+(\mathbf{r}, t)\, e^{-i 2\pi \mathbf{k} \cdot \mathbf{r}} \quad\text{(signal = Fourier transform of image)}$$

---

## 🧠 Self-test questions

1. The k-vector has units of $\text{m}^{-1}$. Why is that the right unit for what we're calling a "wave vector"? What is the corresponding wavelength?

2. You apply $G_x = 10$ mT/m for 1 ms (proton imaging, $\gamma/2\pi = 42.58$ MHz/T). Where in k-space are you?

3. Why does the signal you measure right at the k-space center ($t = 0$ if no prephasing) equal the integral of the magnetization over the whole sample?

4. You restrict your k-space sampling to a 64×64 grid centered on the origin, but your true image has features finer than the grid resolution. What artifact do you expect in the reconstructed image? What if instead you sampled a 256×256 grid but punched a hole in the center?

5. The PSF for fully sampling a rectangular region of k-space is a 2D sinc. **Why a sinc?** (Hint: what is the Fourier transform of a boxcar function?)

6. Explain in one sentence (no math) why the k-space center carries contrast information and the periphery carries resolution information.

7. You're designing a sequence that needs to acquire one phase-encoding line per TR. After the RF excitation, what gradient waveform takes you from the origin to the start of phase-encode line $k_y = k_y^{(n)}$ and then reads out a full $k_x$ line at that $k_y$?

8. Sign-convention check: this lecture defines $\tilde{S}(\mathbf{k}) = \int M\, e^{-i 2\pi \mathbf{k} \cdot \mathbf{r}}\, d\mathbf{r}^3$. Another textbook uses $\tilde{S}(\mathbf{k}) = \int M\, e^{-i \mathbf{k} \cdot \mathbf{r}}\, d\mathbf{r}^3$ (no $2\pi$). How does the relationship between the gradient waveform and the k-space trajectory change?

9. **Bonus / conceptual:** the equation $\tilde{S}(\mathbf{k}) = \mathcal{F}\{M'_+(\mathbf{r})\}$ doesn't mention time, gradients, or any of the physics. Why not? Where did the physics go?

---

## 📌 What to take into next lecture

L10 (Lecture 10) was already partially covered today (sections 10.2–10.5 of the script). The next FAU lecture (L10 in the slide deck numbering) will deepen the practical sampling discussion and dive into **Gibbs ringing and slice selection** — both direct consequences of the PSF idea introduced today.

For the upcoming days in the roadmap:
- **Day 12 (today):** k-space encoding — the math and the intuition. ✅
- **Day 13:** Gibbs ringing + slice selection (specific PSF consequences and pulse design).
- **Day 14:** CMRI Lecture 2 — Fourier image reconstruction basics (where we actually code up the inverse FFT pipeline on real data).

You've now got the conceptual machinery. The next two days are about applying it.
