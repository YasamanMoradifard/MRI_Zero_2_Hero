# CMRI L02 — Fourier Image Reconstruction Basics

> **Computational MRI** · based on a lecture by Ricardo Otazo
> *How do we get from the raw signal an MR scanner records to a picture of the brain?*

This lecture is the hinge of the whole course. Module 1 ended with the punchline: a **linear gradient makes the Larmor frequency depend on position**, so the detected signal is a sum of differently-phased contributions from every point in the object. This lecture cashes that in: it shows that the recorded signal *is the Fourier transform of the image*, names the space it lives in (**k-space**), and works through everything you need to turn k-space samples back into pixels — sampling, aliasing, the FFT, resolution, Gibbs ringing, windowing, and SNR.

If you remember one sentence: **the scanner samples k-space (the spatial-frequency domain); the image is its inverse Fourier transform.** Everything else is consequences of that.

---

## 1. Spatial encoding → the signal equation

From Module 1 (the gradient lecture), a gradient `G_y` along `y` adds a position-dependent term to the field:

```
B(y) = B0 + G_y · y      ⇒      ω(y) = ω0 + γ G_y · y
```

So a spin at position `y` precesses at a frequency that encodes where it is. Accumulating that frequency over time accumulates **phase**:

```
φ(x, t) = γ x ∫₀ᵗ G_x dτ  ≡  k_x · x
φ(y)    = γ y ∫₀^{T_y} G_y dτ  ≡  k_y · y
```

This is the key definition — **the k-space coordinate is the time-integral of the gradient**:

```
k_x = γ ∫ G_x dτ ,      k_y = γ ∫ G_y dτ
```

You move through k-space by playing gradients. The total signal is the sum (integral) over all spins, each weighted by the transverse magnetization `m(x,y)` and carrying its encoding phase:

```
s(k_x, k_y) = ∫∫ m(x,y) · e^{−i k_x x} · e^{−i k_y y} dx dy
```

**That is exactly a 2D Fourier transform of the image `m(x,y)`.** The signal you measure *is* k-space. The image is the inverse transform of the measured signal — `FT⁻¹`.

> **Convention warning (this trips everyone up).** Two conventions for `k` coexist:
> - **Angular** `k = γ∫G dτ` (units rad/m), with kernel `e^{−ik·r}`.
> - **Cyclic** `k = (γ/2π)∫G dτ` (units 1/m), with kernel `e^{−i2πk·r}`.
>
> The gradient slide uses the angular form `e^{−ikr}`; the FT-properties slides use the cyclic form `e^{−i2πkr}`. The clean sampling relations (`Δk = 1/W`) are written in the **cyclic** convention. Pick one and stay in it; the physics is identical. (The `1/2π` prefactor written on the FT slide is an artifact of mixing the two — in the pure cyclic convention the inverse has no prefactor.)

---

## 2. The Fourier transform (the one tool)

Cyclic convention, 1D:

```
S(k) = ∫ s(r) e^{−i2πkr} dr        (forward)
s(r) = ∫ S(k) e^{+i2πkr} dk        (inverse)
```

**Intuition.** `S(k)` tells you "how much of the sinusoid at spatial frequency `k`" is present in `s(r)`. The transform is a change of basis from *positions* to *frequencies*. `S(k)` is the **spectrum** of the signal `s(r)`.

**1D audio example (slide 5).**
- A pure 440 Hz tone (A4, a tuning fork) is a single sinusoid → its spectrum is **one sharp peak** at 440 Hz.
- A flute playing A4 is *periodic* but not sinusoidal → its spectrum is a **fundamental peak plus harmonics** at 880, 1320, … Hz. Same pitch, different *timbre* = different high-frequency content.

This is the whole game in miniature: shape of the spectrum ↔ shape of the signal.

---

## 3. Fourier transform properties (and why MRI cares)

| Property | Statement | Why it matters in MRI |
|---|---|---|
| **Linearity** | `F{a·s₁ + b·s₂} = a·S₁ + b·S₂` | Coils, contrasts, and channels add linearly in both domains. |
| **Shifting** | `F{s(r − r₀)} = e^{−i2πk r₀} S(k)` | An image shift = a **linear phase ramp** in k-space. **Motion / position change during the scan → phase errors in k-space → ghosting/artifacts.** |
| **Modulation** | `F{e^{i2πk₀r} s(r)} = S(k − k₀)` | Multiplying the signal by a phase ramp slides k-space; off-resonance/chemical shift = a frequency offset = a position shift. |
| **Scaling** | `F{s(ar)} = (1/\|a\|) S(k/a)` | Stretch the object → squeeze its spectrum (and vice-versa). Underlies the FOV ↔ Δk relation. |
| **Conjugate symmetry** | `s(r) real ⇒ S(−k) = S*(k)` | Real images have Hermitian k-space → you can in principle skip ~half of k-space (**partial-Fourier** acquisition). |
| **Parseval** | `∫ s₁ s₂ dr = ∫ S₁ S₂ dk`; for `s₁=s₂`, `∫\|s\|² dr = ∫\|S\|² dk` | **Energy is conserved** across domains → you can measure signal intensity / SNR directly in k-space. |
| **Convolution theorem** | `F{s₁ * s₂} = S₁·S₂`  and  `F{s₁·s₂} = S₁ * S₂` | **The single most important property for reconstruction.** Multiplying k-space by a window = convolving the image with a blur kernel. This *is* the mechanism behind truncation, Gibbs ringing, the PSF, and windowing. |

### Convolution, briefly
```
(f * g)(t) = ∫ f(τ) g(t − τ) dτ
```
Slide and slide: flip one function, slide it across the other, integrate the overlap at each shift. The convolution theorem turns this expensive operation into a cheap multiplication in the other domain.

---

## 4. The Fourier "alphabet" you must memorize

Each k-space sampling/windowing effect is one of these pairs in disguise.

| `s(r)` | `S(k)` |
|---|---|
| Impulse `δ(r)` | **Constant** (flat spectrum — a point contains all frequencies) |
| Constant | Impulse `δ(k)` |
| **Rectangle** (boxcar, width `d`) | **Sinc** `∝ sinc(kd)` (main lobe + decaying side lobes) |
| Sinc | Rectangle |
| Sine / Cosine | **Pair of peaks** (delta functions at `±k₀`) |
| **Comb** `Σ δ(r − nT)` | **Comb** with spacing `1/T` (sampling ↔ replication) |
| Triangle `= rect * rect` | **`sinc²`** (by the convolution theorem: `sinc · sinc`) |
| Gaussian | Gaussian |

> **Worked-out triangle (slide 10, "break it down to what you already know").** A triangle is the convolution of two identical rectangles. By the convolution theorem, `F{rect * rect} = sinc · sinc = sinc²`. You never have to integrate a triangle by hand.

**The comb pair is the foundation of sampling:** sampling a signal = multiplying it by a comb of spacing `T` → in the frequency domain this *convolves the spectrum with a comb of spacing `1/T`*, i.e. it **replicates** the spectrum. Replicas that overlap = aliasing (Section 6).

---

## 5. Multidimensional FT and k-space

The transform is **separable** — you do it one axis at a time:

```
S(kx, ky) = ∫∫ s(x,y) e^{−i2π(kx·x + ky·y)} dx dy
```

A 2D FFT is just FFTs of all rows followed by FFTs of all columns. This is why MRI reconstruction of Cartesian data is "just" `ifft2`.

**k-space (slide 12).** The measured data, displayed as `|S(kx,ky)|`, is a bright blob in the center fading to noise at the edges. Run `FT⁻¹` and you get the recognizable (e.g. T₂-weighted) image. k-space and image are the *same information* in two representations.

### Where the structure lives: low vs high spatial frequencies
- **Center of k-space (low frequencies):** overall **brightness, contrast, bulk shape**. Most of the signal energy is here.
- **Periphery (high frequencies):** **edges and fine detail**.

The 64/128/256 demo (slides 13–20) makes this concrete:
- **Keep only a central box** (low-pass) → image is **blurry**; bigger box → sharper. Crucially, **contrast (dark/bright) is unchanged** — you've only thrown away detail, not brightness. Bigger window → higher resolution; smaller window → lower resolution.
- **Keep only the periphery** (high-pass, center zeroed) → you see **edges only**; the bulk is gone. The smaller the removed center / the more you keep, the more edge structure remains.

This is the intuition behind every k-space trick that follows.

---

## 6. Sampling and aliasing (Nyquist/Shannon)

**Theorem.** A signal of bandwidth `B` can be perfectly reconstructed from regular samples taken with period

```
Δt ≤ 1/(2B)        (Nyquist rate)
```

- `Δt ≤ 1/2B` ⇒ spectral replicas don't overlap ⇒ **no aliasing**.
- `Δt > 1/2B` ⇒ replicas overlap ⇒ **aliasing** (high frequencies masquerade as low ones; fold-over).

**Mapping to MRI (slide 23) — memorize this dictionary:**

| Sampling-theorem concept | MRI counterpart |
|---|---|
| **Bandwidth** of the signal | **Spatial extent of the object / FOV** (image domain) |
| **Sampling rate** | **Density of k-space samples** (k-space domain) |

So in MRI you sample **k-space**, and the "bandwidth" you must respect is set by the **size of the object / field of view**.

---

## 7. Sampling k-space: FOV, resolution, trajectories

### Cartesian sampling (slide 24)
Nyquist spacing:
```
Δk_x = 1/W_x ,      Δk_y = 1/W_y
```
where `W_x, W_y` are the FOV dimensions. Two independent knobs:

- **Δk sets the FOV.** `FOV = 1/Δk`. Halving `Δk` (sampling twice as densely) **doubles the FOV** — and does **nothing to resolution**. If you undersample (`Δk` too large) the FOV shrinks below the object and you get **wrap-around aliasing**.
- **k_max sets the resolution.** `Δr = 1/(2·k_max) = FOV/N`. To improve resolution you must **extend k_max** (acquire higher spatial frequencies), which costs acquisition time. Sampling more densely at fixed `k_max` only enlarges the FOV — it does **not** sharpen the image.

> Slide-margin summary: "to change FOV → change `Δk`; to change resolution → change `k_max`. How densely you sample doesn't change resolution — it only changes the field of view."

### Radial sampling (slide 25)
Spokes through the center; spacing `Δk` along a spoke and angular spacing `Δφ`. Approximate full-sampling requirement:
```
N_radial ≈ (π/2) · N_Cartesian
```
Radial **oversamples the center** (good for motion robustness and self-navigation) and aliases *gracefully* — its undersampling artifact is incoherent **streaking** rather than coherent fold-over.

### Aliasing examples (slide 26)
Increasing the sampling spacing (undersampling) increases aliasing artifacts:
- **Cartesian** `Δk_y = 2/W_y, 4/W_y`: discrete **fold-over copies** of the object stacked on top of each other (FOV too small).
- **Radial** `Δφ = 2×, 4× Δφ_Nyquist`: diffuse **streak** artifacts; the object stays recognizable far longer than in the Cartesian case. This benign failure mode is a major reason radial (and non-Cartesian) trajectories are popular for accelerated/compressed-sensing MRI.

---

## 8. Discrete reconstruction: the DFT/FFT

Real data is discrete, so we use the **Discrete Fourier Transform**, computed fast as the **FFT** (`O(N log N)`):

```
S(k) = Σ_{n=0}^{N−1} s(n) e^{−i2πnk/N}          (forward)
s(n) = (1/N) Σ_{k=0}^{N−1} S(k) e^{+i2πnk/N}      (inverse)
```
NumPy: `S = np.fft.fft(s)`, `s = np.fft.ifft(S)` (and `fft2`/`ifft2` in 2D).

### Reconstructing Cartesian k-space (slides 28–29) — the "fftshift dance"
k-space data is **centered** (DC in the middle), but `np.fft` assumes the origin sits at index 0. If you call `ifft2` directly you get the image split into the four corners (a checkerboard of quadrants). The fix:

```python
# image  ← k-space
s = np.sqrt(S.size) * np.fft.fftshift(np.fft.ifft2(np.fft.ifftshift(S)))

# k-space ← image
S = (1/np.sqrt(s.size)) * np.fft.fftshift(np.fft.fft2(np.fft.ifftshift(s)))
```
Reading it inside-out:
1. **`ifftshift`** — move the centered DC component to index 0 (where the FFT expects it).
2. **`ifft2` / `fft2`** — the transform.
3. **`fftshift`** — move the origin back to the center for display.
4. **`sqrt(N)` factor** — makes the transform **unitary** (Parseval-preserving) so signal energy / SNR is comparable across domains.

> From the exercise notes: "fft assumes your signal starts at zero, so we first shift it to zero, then transform, then shift back to center." Radiologists want the **magnitude** of the reconstructed image (k-space and the recon are complex; `|·|` discards phase).

---

## 9. Resolution effects: zero-padding, Gibbs ringing, windowing, PSF

These are all the **convolution theorem in action**: whatever you multiply k-space by, you convolve the image with its transform.

### Zero-padding = Fourier interpolation (slides 30–31)
Pad measured k-space with zeros (e.g. 128×128 → 256×256). Result:
```
p_x = W_x / N_{x,padded}     (pixel size shrinks)
```
**The pixel size decreases but the resolution does NOT improve** — you added no new high-frequency data, only interpolated. Two facts to keep separate:
- **Pixel size** = how the image is displayed (number of pixels).
- **Resolution** = the true blur set by `k_max` / acquisition.

They are not the same thing. Zero-padding is a smooth (sinc) interpolation onto a finer grid. Side effect: it *reveals/enhances* **Gibbs ringing**, because you are still implicitly multiplying k-space by a boxcar (truncation) — its sinc ripples are now displayed on a finer grid.

### Gibbs ringing (slide 32)
- **What:** spurious ripples ("ringing") parallel to **sharp edges**.
- **Cause:** **k-space truncation**. Cutting k-space at `k_max` = multiplying by a **boxcar** ⇒ convolving the image with a **sinc** (`F{rect} = sinc`). The sinc's side lobes are the ringing.
- **Trend:** *stronger for smaller N* (smaller k-space window → wider sinc → more visible ringing). `N↓ ⇒ Gibbs↑`.
- **Clinical note:** ringing can mimic real anatomy (e.g. a bright/dark line that looks like a lesion or a syrinx), so it matters diagnostically.

### k-space windowing / filtering (slide 33)
Multiply k-space by a smooth taper `W(k)`:
```
S_W(k) = S(k) · W(k)
```
**Hamming window:**
```
W(k) = 0.54 + 0.46·cos(2πn/N)
```
Comparing point-spread functions (PSFs):
- **Boxcar (no window):** narrow main lobe (good resolution) but **large side lobes** (strong ringing).
- **Hamming:** **tiny side lobes** (ringing suppressed) but a **wider main lobe** (lower resolution → blurrier).

**You always pay something.** Hamming trades resolution for reduced ringing. At high N the Hamming image is still slightly blurrier than boxcar, "but that's good enough," and avoiding Gibbs (which can masquerade as anatomy) is often worth the small blur.

### Point spread function (PSF) — the unifying idea (exercise notes)
The **PSF is the image of a single point** — equivalently, the inverse FT of your k-space sampling/window function. The measured image is the true object **convolved with the PSF**:
```
image = object * PSF
```
- **Narrow PSF → sharp image; wide PSF → blurry image.** Quantify with the **FWHM** (full-width at half-maximum) of the PSF.
- Boxcar truncation → sinc PSF (ringy). Hamming → broad, low-side-lobe PSF (smooth but blurry).
- This is why "object (sharp) ⊛ PSF = blurry image": multiplication in k-space ⇔ convolution in the image domain.

### Oversampling in the readout direction (exercise notes)
Data is cheaply **oversampled along readout** (`Δk` smaller / more samples per line, "free" because it only extends the readout):
- Enlarges the **readout FOV** ⇒ **prevents fold-over aliasing** in that direction.
- **More samples ⇒ smaller PSF side lobes ⇒ less ringing** ("fewer samples = more side lobes").
- The **decimated image = the central part of the oversampled image** (you reconstruct the big FOV, then crop back to the FOV you wanted).

---

## 10. Signal-to-noise ratio (SNR)

Simplified scaling law (slides 34–35):
```
SNR ∝ V · √T
```
- **V** = voxel volume. **T** = cumulated readout/acquisition time.
- **Higher resolution → smaller V → lower SNR.** This is the fundamental tension: resolution, SNR, and scan time trade against each other. You can buy SNR back by averaging longer (`√T`), at the cost of time.

A simple **measurement** (slide 35):
```
SNR = S / N = (mean pixel signal in tissue) / (std of the background)
```
**Caveats (Dietrich, JMRI 2007):**
- It assumes the noise std measured in the air/background equals the noise inside the tissue — which often **isn't true** (parallel imaging makes noise spatially varying; magnitude background noise is **Rayleigh**-distributed, not Gaussian, when `signal→0`).
- Better alternatives: **Monte-Carlo** estimation, or the **two-image difference / "image − noise"** method (acquire twice, subtract to isolate noise) — valid when relaxation/`TR` effects can be neglected between the two acquisitions.

---

## Key takeaways

1. **The MR signal is the Fourier transform of the image; k-space is the spatial-frequency domain.** Reconstruction (Cartesian) = `ifft2` + the fftshift dance.
2. `k = γ∫G dτ` — you traverse k-space by playing gradients.
3. **Δk sets the FOV (`FOV = 1/Δk`); k_max sets the resolution (`Δr = 1/2k_max`).** Two independent knobs.
4. **Low spatial frequencies (k-space center) = contrast/bulk; high frequencies (periphery) = edges/detail.**
5. **Undersampling k-space (Δk too big) → aliasing.** Cartesian aliases as coherent fold-over; radial aliases as benign streaks.
6. **Multiplying k-space by anything = convolving the image with its transform** (convolution theorem). This single fact explains truncation, Gibbs ringing, the PSF, and windowing.
7. **Truncation (boxcar) → sinc PSF → Gibbs ringing**, worse for small N. **Windowing (Hamming) suppresses ringing at the cost of resolution.** You always pay something.
8. **Zero-padding shrinks pixels but does not add resolution** — it's Fourier interpolation.
9. **SNR ∝ V·√T** — resolution, SNR, and time are a three-way trade-off.

---

## Self-test (no peeking)

1. Starting from `B(y) = B0 + G_y y`, derive the position-dependent precession frequency and explain why the recorded signal equals the 2D Fourier transform of `m(x,y)`. What is the definition of the k-space coordinate?
2. The two `k` conventions (`γ∫G` vs `(γ/2π)∫G`) differ by a factor of `2π`. In which one is `Δk = 1/W` written, and what are the units of `k` in each?
3. A pure tone and a flute play the same note. Their spectra differ — how, and what musical property does the difference correspond to?
4. State the convolution theorem in both directions. Use it to explain *in one sentence* why truncating k-space causes Gibbs ringing.
5. Without integrating, give the Fourier transform of a triangle function and justify your answer using a known pair plus a property.
6. You scan an object with `Δk_y` twice the Nyquist spacing. Describe the resulting artifact for (a) a Cartesian trajectory and (b) a radial trajectory, and explain why they differ.
7. You want to double the spatial resolution of a Cartesian acquisition. Which k-space quantity must you change, what happens to scan time, and what happens to SNR (give the scaling)?
8. Write the NumPy one-liner that reconstructs a centered NxN k-space array into a unitary, centered image. Explain the role of each of `ifftshift`, `ifft2`, `fftshift`, and the `sqrt(N)` factor.
9. You zero-pad 128×128 k-space to 256×256. A colleague says "now we have twice the resolution." Are they right? Distinguish pixel size from resolution in your answer.
10. Define the point spread function. Sketch (qualitatively) the PSF of a boxcar vs a Hamming window and state the resolution/ringing trade-off each makes. Why might a radiologist *prefer* a slightly blurrier Hamming reconstruction?
