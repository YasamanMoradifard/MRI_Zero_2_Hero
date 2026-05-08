# L01 — Intro to MRI

> **Source:** MRI1 Lecture 1 (Chapters 1 & 2 of the FAU course notes)
> **Goal of this lecture:** Get a bird's-eye view of how an MRI scanner produces an image — from the strong magnetic field down to the encoded signal. This is the "trailer" for the whole movie. Don't expect every detail to click yet.

---

## What you should be able to do after today

- Explain in one paragraph why MRI exists and what makes it special among medical imaging modalities.
- State the **three core principles of MRI** and what each one contributes.
- Define **proton magnetization** and explain why it points along $\mathbf{B}_0$.
- Explain the resonance condition and write down the **Larmor frequency** equation.
- Sketch a T₂-weighted contrast curve and explain why fluids look bright at long TE.
- Describe in words how **slice selection** and **frequency encoding** turn signal into a spatially-localized image.

---

## 1. The big picture: why MRI?

MRI competes with **CT**, **PET**, **SPECT**, and **ultrasound**. Its standout features are:

- **No ionizing radiation.** CT, PET, and SPECT all expose patients to X-rays or gamma rays. MRI uses static magnetic fields and radio-frequency waves — both non-ionizing.
- **Unmatched soft-tissue contrast.** Grey vs white matter, tumor vs healthy tissue, fat vs muscle — MRI distinguishes these with a clarity ultrasound can't touch.
- **Many contrasts from one machine.** The same scanner can produce dramatically different images of the same anatomy by changing pulse-sequence parameters. This is *the* superpower of MRI.

The trade-offs:
- **Bones are invisible** (or nearly so). Hard tissues = X-ray's domain.
- **Slow.** A typical exam takes 20–40 minutes. Speeding it up is the focus of Module 4.
- **Expensive.** A 3T scanner costs ~$1–3M plus heavy infrastructure (cooling, shielding, cryogens).
- **Safety hazards from the magnet.** Ferromagnetic objects → "missile effect." Strict screening required.

> **Historical note:** Bloch & Purcell described nuclear magnetic resonance in 1945–46 (Nobel Prize 1952). It took until the **early 1970s** for Lauterbur & Mansfield to turn NMR into *imaging* (Nobel Prize 2003 in Physiology or Medicine). The gap reflects how non-obvious the spatial-encoding idea was.

---

## 2. The Three Principles

The course is built like a theoretical physics text: a few axioms, then derive everything. Here are the axioms.

### 🟦 Principle 1 — Proton Magnetization

> When protons sit in a magnetic field $\mathbf{B}_0$, a proton magnetization $\mathbf{M} \parallel \mathbf{B}_0$ arises.

**Setup:** A strong, static magnetic field $\mathbf{B}_0 = B_0 \mathbf{e}_z$ (typically 1.5T, 3T, or 7T clinically). For comparison, a junkyard car-lifting magnet is ~0.3T.

**What happens microscopically:** Each proton has a magnetic moment $\boldsymbol{\mu}$ — think "tiny bar magnet." With no field, the $\boldsymbol{\mu}$'s point in random directions and average to zero. With $\mathbf{B}_0 > 0$, they preferentially align *along* the field. The macroscopic magnetization is:

$$\mathbf{M} = \frac{1}{V}\sum \boldsymbol{\mu}$$

**Key intuition:** The polarization is *tiny* — about 1 in 100,000 protons aligns with the field at 3T and body temperature. But there are ~$10^{23}$ protons per cm³ of water, so even a $10^{-5}$ excess produces a measurable signal.

**Why it scales with $B_0$:** $M \propto B_0$. This is *the* reason MRI scanners use such enormous field strengths — bigger field, more polarization, more signal.

**Don't confuse:** Electron magnetization (diamagnetism, paramagnetism, ferromagnetism) is much stronger than proton magnetization, but it's electrons that give materials their bulk magnetic properties. MRI deliberately targets **protons** because their signal is sharp and well-localized.

### 🟦 Principle 2 — Magnetization Generates a Dipole Field

> $\mathbf{M}$ generates a magnetic dipole field, called the $\mathbf{B}_1^{-}(\mathbf{r})$ field.

This is "nothing special" from electrodynamics — any magnetization produces a field. The naming convention is the trick:

| Field | Meaning |
|-------|---------|
| $\mathbf{B}_0$ | Static main field (the magnet) |
| $\mathbf{B}_1^{+}$ | RF field *into* the body (transmit) |
| $\mathbf{B}_1^{-}$ | Field coming *out* of the body (receive — what we measure) |

The "+" and "−" don't mean positive/negative; they mean "going in" vs "coming out." When $\mathbf{M}$ rotates, $\mathbf{B}_1^{-}(\mathbf{r})$ rotates with it.

**Why this matters:** MRI doesn't measure $\mathbf{M}$ directly. It measures $\mathbf{B}_1^{-}$ via electromagnetic induction in a coil. **No $\mathbf{B}_1^{-}$ → no induction → no signal → no MRI.**

### 🟦 Principle 3 — Resonance and Relaxation

This one has three parts (3a, 3b, 3c), and it's the heart of MRI.

**3a — Resonance:** If you irradiate the body with an oscillating $\mathbf{B}_1^{+}$ field of frequency $\nu$, *nothing happens* — except a little tissue heating — **unless** $\nu$ matches the magic Larmor frequency:

$$\boxed{\nu_0 = \frac{1}{2\pi}\gamma B_0}$$

where $\gamma = 2.675 \times 10^8$ rad/s/T is the **gyromagnetic ratio** for protons. At that magic frequency, the body radiates an oscillating $\mathbf{B}_1^{-}$ field of the same frequency.

| Field strength | Larmor frequency |
|----------------|------------------|
| 1.5 T | ~63 MHz (FM radio range) |
| 3.0 T | ~126 MHz |
| 7.0 T | ~298 MHz |

**Why this is wild:** $B_0$ is several tesla. A typical $B_1^{+}$ is a few microtesla — *six orders of magnitude weaker*. How does such a tiny field budge such a large $\mathbf{M}$? **Resonance.** Same reason a child's swing — pushed gently at the right rhythm — goes higher and higher. Off-rhythm pushes do nothing.

**3b — T₂ relaxation (transverse decay):** After excitation, $\mathbf{B}_1^{-}$ decays exponentially:

$$B_1^{-}(t) \propto \exp(-t/T_2)$$

$T_2$ is **tissue-specific**. Typical values:
- Liver: ~35 ms
- Fat: ~70 ms
- Cerebrospinal fluid (CSF): ~2200 ms

**3c — T₁ relaxation (longitudinal recovery):** After excitation, the longitudinal magnetization $M_\parallel$ recovers back toward equilibrium $M_0$:

$$M_\parallel(t) = M_0 - (M_0 - M_\parallel(t_0))\exp(-t/T_1)$$

Typical values:
- Liver: ~800 ms
- Fat: ~380 ms
- CSF: ~4500 ms

**Note:** $T_1 > T_2$ always. We'll explore why in Module 3.

---

## 3. Contrast generation: T₂-weighting in 60 seconds

The simplest signal is just *proton density* — count the water. But the killer feature of MRI is that you can **shape contrast** by choosing *when* to measure.

**T₂-weighting recipe:**
1. Excite the magnetization.
2. **Wait** for a duration called the **echo time (TE)**.
3. Measure.

During TE, all tissues lose signal — but at different rates because their $T_2$ differs. By choosing TE wisely, you turn what would be a flat proton-density image into a high-contrast diagnostic image.

**The "no free lunch" rule:** Every MRI gain comes with a cost.

| Gain | Cost |
|------|------|
| More T₂ contrast (longer TE) | Less total signal; longer scan time |
| Higher resolution | More noise per pixel; longer scan |
| Faster scan | Lower SNR or more artifacts |

**Visual hallmark of T₂-weighting:** **Fluids (CSF, edema) appear bright** because their long $T_2$ means they haven't decayed much by the time you measure.

---

## 4. Signal excitation and detection (the hardware view)

- **Transmit coil:** Copper coil driven by an oscillating current at $\nu_0$. Built into the scanner bore. Generates $\mathbf{B}_1^{+}$.
- **Receive coil:** Sits as close to the body as possible (head coils, body arrays) because the dipole field falls off as $1/r^3$ in the far field. Detects $\mathbf{B}_1^{-}$ via induction.

**The signal equation (first version):**

$$\text{signal} \propto \int M(\mathbf{r}) \, c(\mathbf{r}) \, d^3\mathbf{r}$$

where $c(\mathbf{r})$ is the receive-coil sensitivity profile.

**The problem:** The signal is an *integral*. It's a single oscillating voltage with two degrees of freedom (amplitude + phase). How do you extract a 256×256 image (65,536 pixels) from that?

---

## 5. Image encoding (the Lauterbur–Mansfield trick)

**Add gradient coils.** These produce small, position-dependent additions to $B_0$:

$$B(x) = B_0 + G \cdot x$$

So the Larmor frequency becomes position-dependent:

$$\nu(x) = \frac{\gamma}{2\pi}(B_0 + G\cdot x)$$

This *single idea* unlocks two encoding techniques (and more, in later lectures):

### Slice selection
- Switch on a gradient *during* the RF pulse.
- Irradiate at frequency $\nu_2$ → only protons at position $x_2$ where $\nu(x_2) = \nu_2$ get excited.
- Use a *band* of frequencies → excite a *slice* of finite thickness.

### Frequency encoding
- Switch on a gradient *during* signal readout.
- Each position emits at a different frequency.
- Take the Fourier transform of the recorded signal → recover the 1D distribution of magnetization along the gradient direction.

> The two techniques have **opposite timing**: slice selection uses the gradient *with* the RF pulse; frequency encoding uses the gradient *with* the readout. This is the first "timing diagram" of MRI — and timing is *everything* from here on.

**A noisy side-effect:** Switching gradients on/off creates Lorentz forces that vibrate the gradient coils. This is why MRI scanners are *loud*.

---

## 6. Why MRI is hard to learn (and how to deal with it)

The lecture explicitly warns: many things won't click on the first pass. Don't panic — that's the point of the rest of the course. Open questions you should *expect* to have right now:

- How exactly does $\mathbf{M}$ "rotate"? What's the geometry?
- If $\mathbf{M}$ is parallel to $\mathbf{B}_0$, how can it produce a *time-varying* field that induces a current?
- What does "the magnetization oscillates" really mean physically?
- How do we encode the third dimension (we only saw 2)?

These are all answered in Lectures 2–11. **Patience, young MRI novice.** 🧙

---

## 🧪 Self-test questions

Try answering these without looking back. Bring up anything you can't answer in our next chat.

1. Why is a strong $B_0$ field beneficial for image quality? Give a quantitative argument.
2. What is the Larmor frequency at 1.5T? At 3T? At 7T?
3. Compute $\nu_0$ for a proton at $B_0 = 0.5$T. (Use $\gamma = 2.675 \times 10^8$ rad/s/T.)
4. Why does a long TE produce *more* T₂ contrast but *less* total signal?
5. In a T₂-weighted brain image, why does the CSF appear bright?
6. The transmit field $B_1^{+}$ is ~6 orders of magnitude weaker than $B_0$. Why can it still affect $\mathbf{M}$?
7. What is the role of the gradient coil in (a) slice selection, (b) frequency encoding? When is it switched on in each case?
8. The signal equation says signal $\propto \int M(\mathbf{r})c(\mathbf{r})d^3\mathbf{r}$. Why is this a problem if our goal is to determine $M(\mathbf{r})$? What did Lauterbur & Mansfield add to fix it?

---

## 🔗 Further reading

- **Schild, "MRI Made Easy"** (Bayer, free): great gentle intro that complements this chapter.
- **Lauterbur (1973),** *Image formation by induced local interactions*, Nature 242:190–191 — the Nobel-winning paper. Short and historic.
- **Plewes & Kucharczyk (2012),** *Physics of MRI: a primer*, JMRI 35(5):1038–54. A more rigorous overview at slightly higher math level than today's lecture.

---

## 💭 One sentence summary

> **MRI works by tipping a tiny bulk magnetization with a resonant RF pulse, listening to the dipole field it radiates back, and using gradient fields to make that field's frequency depend on position — which lets a Fourier transform turn the raw signal into an image.**

That sentence won't fully make sense yet. Re-read it after Lecture 11 — it should.
