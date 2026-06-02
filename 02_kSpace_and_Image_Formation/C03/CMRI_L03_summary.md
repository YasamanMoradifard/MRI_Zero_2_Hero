# C03 — Partial Fourier Imaging

> **Module 2 · Day 16** · *CMRI Lecture 3 (Partial Fourier Imaging and constrained reconstruction)*
> Source: Computational Imaging Lab, FAU Erlangen-Nürnberg / NYU

---

## The one-sentence idea

k-space is *redundant* — for a real-valued object, one half is the complex conjugate of the other — so we can acquire a little over half of it and **reconstruct the rest**, cutting scan time roughly in half. The catch is that real objects are never truly real-valued, so the whole technique hinges on **estimating and correcting the image phase**.

---

## 1 · Why bother: scan time lives in the phase-encode lines

Every **phase-encode (PE) line** of k-space costs one repetition time (TR). A full acquisition that takes 10 minutes becomes ~5 minutes if you skip half the PE lines. Readout samples within a line are essentially free, so partial Fourier always under-samples the **PE direction** (and the analogous trick on the readout axis is called *fractional / asymmetric readout*).

The question this lecture answers: *given a half-filled k-space, how do we get a full-resolution image back?*

---

## 2 · Hermitian symmetry — the theoretical free lunch

The key Fourier fact. If the object $\rho(r)$ is **real-valued**, then its spectrum has **Hermitian (conjugate) symmetry**:

$$\boxed{\,S(-k) = S^{*}(k)\,}$$

The lower half of k-space carries no information that the upper half doesn't already contain. **In theory, you only need half.**

Supporting Fourier transform properties worth keeping in your pocket:

| Property | Relation |
|---|---|
| Linearity | $\mathcal{F}\{a\,s_1 + b\,s_2\} = a S_1(k) + b S_2(k)$ |
| Shifting | $\mathcal{F}\{s(r-r_0)\} = e^{-i 2\pi k r_0} S(k)$ |
| Modulation | $\mathcal{F}\{e^{\,i 2\pi k_0 r} s(r)\} = S(k - k_0)$ |
| Scaling | $\mathcal{F}\{s(ar)\} = \tfrac{1}{|a|} S(k/a)$ |
| **Conjugate symmetry** | $\rho \in \mathbb{R} \Rightarrow S(-k) = S^{*}(k)$ |

**Historical note.** This idea was worked out early by Margosian & Schmitt (Siemens, Erlangen, 1985), *Faster MR Imaging Methods*: "half-data" acquisition using the central half, the upper/lower half + phase correction (the "half Fourier" method, ~53% of the data at full resolution), and interleaved sampling.

> 💡 **Implementation gotcha.** For an even-sized array, the conjugate partner of index $a$ is at $(N-a)\bmod N$, **not** $N-1-a$. A plain `array[::-1]` flip is off by one pixel and will make a "verify the symmetry" check fail spuriously. (The notebook fixes this with a flip + roll-by-1.)

---

## 3 · Why the naive half-acquisition fails

The textbook fantasy: acquire the lower half, synthesize the upper half via $S(-k)=S^*(k)$, inverse-FFT, done. It works **perfectly for a real object** — and falls apart the instant the object carries phase:

- **$B_0$ inhomogeneity** — the field is never perfectly homogeneous; off-resonance accrues phase.
- **Object motion** — bulk and physiological motion stamp phase onto the signal.
- **Chemical shift, flow, susceptibility** — all phase sources.

When $\rho$ is complex, $S(-k) \neq S^{*}(k)$, the synthesized half is wrong, and the reconstruction is corrupted. So the assumption "k-space is symmetric" silently rests on "$B_0$ is homogeneous," which is false in practice.

**The practical fix (slide 8):** acquire **slightly more than half** of k-space — a small **symmetric band around the k-space center** plus the asymmetric high-frequency lines on one side. The symmetric central band gives a low-resolution image from which we **estimate the phase**, then we enforce that phase and reconstruct the **magnitude** image. This works because real-world phase varies *slowly* in space, so a low-resolution estimate captures it.

---

## 4 · The sampling layout

Define the **partial Fourier fraction** $PF$ (e.g. $9/16 = 0.5625$). With image size $N$:

- **Acquired**: a contiguous block of $N_{acq} = \mathrm{round}(PF \cdot N)$ PE lines, running from one edge through the center and a bit beyond.
- **Symmetric band**: the central lines whose conjugate partner is *also* acquired. → **phase estimation**.
- **Asymmetric region**: acquired lines with no acquired partner. → these carry the **high-resolution detail** that lifts you above a low-res image.

---

## 5 · Phase estimation

Take **only the symmetric central band**, taper it with a smooth window (Hamming) to suppress Gibbs ringing from the hard k-space cutoff, inverse-FFT, and keep the phase:

$$\varphi(r) = \angle\, I_c(r), \qquad I_c = \mathcal{F}^{-1}\{ S_0(k)\, W_c(k) \}$$

where $W_c(k)$ is the windowed low-pass filter over the symmetric band. Because the band is symmetric and low-resolution, $\varphi(r)$ is a smooth, reliable estimate of the object's true phase — exactly what the correction methods below consume.

> The Hamming window's job is **multiplication in k-space → smoothing in image space**, which decreases Gibbs ringing (we "get rid of sharp edges").

---

## 6 · Reconstruction method A — Margosian / homodyne (one-step)

A single, non-iterative pass. Phase-corrected image:

$$\boxed{\,I(r) = \mathrm{Re}\!\left\{\, I_0(r)\, e^{-i\,\angle I_c(r)} \right\}\,}$$

with

$$I_0(r) = \mathcal{F}^{-1}\{ S_0(k)\, W(k) \}.$$

**Steps:**

1. Weight the acquired k-space with a **homodyne ramp filter** $W(k)$: value **2** in the single-sided (asymmetric) region — standing in for the missing conjugate partner — ramping **2 → 0** across the symmetric band, so the doubly-sampled center is weighted **~1** (no double counting). Zero in the unacquired region.
2. Inverse-FFT → $I_0(r)$.
3. **Rotate away the estimated phase** ($\times\, e^{-i\varphi}$) and take the **real part**. The phase rotation lines the true signal up with the real axis; whatever leaks into the imaginary axis was phase error, and `Re{·}` discards it.

The decomposition the lecture draws: $I = \underbrace{|I_0|}_{B}\,\underbrace{e^{+i\theta}}_{}\,\underbrace{e^{-i\theta}}_{A}$ → the $A$ term (phase correction) and $B$ term (magnitude) combine to give a real $C$.

**Ramp vs Hamming weighting:** the ramp version gives **sharper** results; the Hamming-windowed version is smoother but **blurrier**. More acquired lines → better resolution either way (compare $PF=9/16$ at 288 PE lines vs $PF=5/8$ at 320 PE lines for a 512² image).

---

## 7 · Reconstruction method B — POCS (iterative)

Frame the reconstruction as a **constrained optimization** solved by **Projection Onto Convex Sets**. POCS finds a point in the intersection of convex constraint sets by alternately projecting onto each — and converges in a few iterations when the constraints are well-defined.

> A set $\Omega$ is convex if for $x_1, x_2 \in \Omega$, $\lambda x_1 + (1-\lambda)x_2 \in \Omega$ for all $\lambda \in [0,1]$ (the line between any two members stays inside).

**The two constraints:**

- $\Omega_1$ — **Phase**: $\varphi(r) = \angle I_c(r)$ (the image must have the estimated phase). We don't need to estimate the *entire* k-space — only a subset (the phase) is unknown.
- $\Omega_2$ — **Data consistency**: $\mathcal{F}\{I(r)\} = S_0(k)$ wherever we actually sampled.

**Algorithm:**

$$
\begin{aligned}
&\textbf{Initialize:} && I_0(r) = \mathcal{F}^{-1}\{S_0(k)\} \quad \text{(zero-filled)} \\
&\textbf{Iterate:} \\
&\quad\text{Phase:} && I_{n+1}(r) = |I_n(r)|\, e^{\,i\,\angle I_c(r)} \\
&\quad\text{Consistency:} && S_{n+1}(k) = \mathcal{F}\{I_{n+1}(r)\}, \quad S_{n+1}(k) = S_0(k)\ \text{if } k \text{ was sampled}
\end{aligned}
$$

Each iteration imposes the estimated phase (keep current magnitude, replace phase), transforms to k-space, then overwrites the sampled locations with the **measured** data. The final magnitude image is the answer.

---

## 8 · Margosian vs POCS — when each wins

| | Margosian / homodyne | POCS |
|---|---|---|
| Type | one-step | iterative |
| Speed | fast | a few iterations |
| Easy data (smooth phase) | ✅ essentially identical to POCS | ✅ |
| Hard data (rapid/high-order phase) | ❌ artifacts remain | ✅ refines to optimum |

> Lecture's own caption: *"In easy cases, no phase fluctuation, the result is the same. But in complicated data, Margosian fails, while POCS is iterating to reach the optimal minimum."*

**Rule of thumb:** homodyne when phase is benign and you want speed; POCS when phase is ugly and you want robustness.

---

## 9 · The cost: SNR

Fewer samples → less signal averaging → lower SNR:

$$\boxed{\,SNR \propto \frac{1}{\sqrt{R}}\,}, \qquad R = \frac{N}{N_h}$$

where $N$ is the reconstructed image size and $N_h$ the number of acquired k-space samples. Since $SNR \propto \tfrac{\text{Signal}}{\text{Noise}}$ and the reduction factor for partial Fourier is $R = 1/PF$, the relative SNR is simply $\sqrt{PF}$.

**Worked example ($PF = 9/16$):** $R = \dfrac{N}{N_h} = \dfrac{16}{9}$, so

$$\frac{1}{\sqrt{R}} = \sqrt{\frac{9}{16}} = 0.75 \quad\text{(a 25\% SNR hit)}.$$

The summary's phrase *"SNR loss close to $\sqrt{2}$"* refers to the **half-acquisition limit** $PF \to 1/2$, where $1/\sqrt{R} = 1/\sqrt{2} \approx 0.707$ — the worst case, since you can't go below ~half and still recover the phase.

---

## 10 · Key takeaways

- **Hermitian symmetry** $S(-k)=S^*(k)$ makes half of k-space redundant — *but only for real-valued objects*.
- Real MRI objects carry **phase** ($B_0$ inhomogeneity, motion, flow), so naive symmetry-fill fails.
- Fix: acquire a **symmetric central band** to **estimate the phase**, enforce it, reconstruct the **magnitude**.
- **Margosian / homodyne**: ramp-weight → phase-correct → `Re{·}`. One-step, fast, great for smooth phase.
- **POCS**: alternate **phase** ($\Omega_1$) and **data-consistency** ($\Omega_2$) projections. Iterative, robust for rough phase.
- **Cost**: $SNR \propto \sqrt{PF}$ — about 0.75 at $9/16$, approaching $1/\sqrt2$ at the half-acquisition limit.
- **Applications**: reduce phase-encoding steps (faster scans) and reduce readout duration (fractional/asymmetric readout, shorter TE).

---

## 🧪 Self-test (try before reading the answers)

1. State the Hermitian symmetry relation and the single property of the object it requires.
2. Real MRI objects are never truly real-valued. Name three physical sources of image phase.
3. Why do we acquire *slightly more* than half of k-space rather than exactly half?
4. What is the symmetric central band used for, and why is it windowed (e.g. Hamming) before use?
5. In the homodyne filter, why is the asymmetric region weighted by **2** while the center is weighted by **~1**?
6. Write the Margosian phase-corrected image equation and explain what `Re{·}` throws away.
7. Name the two convex constraints in POCS and the projection performed for each.
8. Give one regime where Margosian and POCS agree and one where they diverge. Why?
9. For $PF = 5/8$, compute the reduction factor $R$ and the relative SNR.
10. The summary says "SNR loss close to $\sqrt2$." Which limiting $PF$ does that correspond to, and why is $\sqrt2$ the worst case for partial Fourier?

---

<details>
<summary><b>Answers (no peeking until you've tried)</b></summary>

1. $S(-k) = S^{*}(k)$; it requires the object $\rho(r)$ to be **real-valued**.
2. Any three of: $B_0$ inhomogeneity / off-resonance, object motion, flow, chemical shift, susceptibility effects.
3. The exact-half assumption needs a real object. Since real objects have phase, we acquire a **symmetric central band** on top of the half so we can *estimate* that phase and correct for it. The extra lines pay for phase knowledge.
4. To produce a **low-resolution phase estimate** $\varphi(r) = \angle I_c(r)$. It's windowed (Hamming) because a hard k-space cutoff causes Gibbs ringing; the smooth window suppresses ringing (smoothing in image space).
5. The asymmetric region only has **one** of each conjugate pair $\{k,-k\}$; weighting by 2 compensates for the missing partner. The center is doubly-sampled (both partners present), so weighting it ~1 avoids double-counting — hence the ramp from 2 down to 0 through the symmetric band.
6. $I(r) = \mathrm{Re}\{ I_0(r)\, e^{-i\angle I_c(r)} \}$ with $I_0 = \mathcal{F}^{-1}\{S_0 W\}$. The phase rotation aligns true signal with the real axis; `Re{·}` discards the **imaginary component**, which is residual phase error.
7. $\Omega_1$ **phase**: set the image phase to $\angle I_c(r)$ (keep magnitude, replace phase). $\Omega_2$ **data consistency**: overwrite sampled k-space locations with the measured $S_0(k)$.
8. **Agree** for smooth/slowly-varying phase (no phase fluctuation). **Diverge** for rapid, high-order phase: Margosian's single weighting can't undo it and leaves artifacts, while POCS keeps enforcing data consistency and converges to the optimum.
9. $R = N/N_h = 1/(5/8) = 8/5 = 1.6$; relative SNR $= 1/\sqrt{R} = \sqrt{5/8} \approx 0.79$.
10. $PF \to 1/2$, giving $1/\sqrt{2} \approx 0.707$. It's the worst case because you cannot acquire fewer than ~half of k-space and still have a symmetric band to estimate the phase — half is the floor.

</details>

---

## 📚 Going deeper

- Margosian P., Schmitt F. (1985), *Faster MR Imaging Methods*, SPIE Vol. 593.
- Noll, Nishimura, Macovski (1991), *Homodyne detection in magnetic resonance imaging* (IEEE TMI) — the homodyne method.
- Haacke, Lindskog, Lin (1991), *A fast, iterative, partial-Fourier technique capable of local phase recovery* — POCS-style.
- Cuppen, van Est (1987) — POCS for partial Fourier.
- Bernstein, King, Zhou, *Handbook of MRI Pulse Sequences* — partial Fourier chapter.

**Next (Day 17):** Module 2 review — image → k-space → undersample → reconstruct three ways → compare. You now have zero-fill, Margosian, and POCS reconstructors in hand.
