# L07 — Gradients

> **Module 1 — MRI Physics Fundamentals · Day 9 of the roadmap**
> *Source: MRI1 Lecture, Chapter 8 "Gradients" (PDF labelled `L07.pdf`)*

---

## 🎯 Why this lecture matters

Without gradients, every proton in the magnet sees the same $B_0$, so every proton spins at the same Larmor frequency $\omega_0 = \gamma B_0$. The receive coil hears one big aggregated signal and has **no way to tell where in the body each contribution came from**. The image is gone before it can even exist.

The gradient field is the trick that fixes this. By making the total $B$-field — and therefore $\omega$ — depend on position $\boldsymbol r$, we turn *spatial position* into *frequency*. That single idea is the foundation of slice selection, frequency encoding, phase encoding, k-space — basically all of MRI imaging.

This lecture is about **what an ideal gradient field looks like, how the scanner actually makes one, and why the real thing isn't quite ideal (but is close enough)**.

---

## 1. Recap — the three magnetic fields

| Field | Symbol | Time-dependent? | Position-dependent? | Job |
|---|---|---|---|---|
| Main field | $\boldsymbol B_0$ | no | no | Generates the equilibrium magnetization |
| RF field | $\boldsymbol B_1^+(t)$ | yes (pulsed) | no (in our approximation) | Excites / inverts the magnetization |
| Gradient field | $\boldsymbol B_\mathrm{G}(\boldsymbol r, t)$ | yes (pulsed) | **yes** | Makes the Larmor frequency position-dependent |

The most fundamental equation throughout the lecture:

$$
\omega(\boldsymbol r, t) = \gamma\,\lvert\boldsymbol B(\boldsymbol r, t)\rvert
$$

The magnetization doesn't know about our mental decomposition into "main + RF + gradient". It only feels the **total** field — we invent the labels.

---

## 2. The ideal linear gradient field

### 2.1 What "ideal" means

We *want* the Larmor frequency to be a linear function of position:

$$
\omega(\boldsymbol r) = \gamma\bigl(B_0 + \boldsymbol G\cdot\boldsymbol r\bigr),
\qquad
\boldsymbol G = \begin{pmatrix} G_x \\ G_y \\ G_z \end{pmatrix}
$$

Two reasons:

1. **Linear is mathematically clean** — slice selection, k-space, FFT reconstruction all assume linearity.
2. **Linear is controllable** — knowing $G_x, G_y, G_z$ tells us *everything* about the spatial encoding.

### 2.2 Why the gradient field points along $\hat{\boldsymbol e}_z$

A subtle point that often trips people up. $\omega$ is set by the **magnitude** of the total $\boldsymbol B$, not by one of its components. So we need to design $\boldsymbol B_\mathrm{G}$ such that adding it to $\boldsymbol B_0$ gives a clean *linear* magnitude.

**The trick: point the gradient field along the same direction as $\boldsymbol B_0$ (i.e. $\hat{\boldsymbol e}_z$).** Then $\boldsymbol B_\mathrm{total} = (B_0 + \boldsymbol G\cdot\boldsymbol r)\,\hat{\boldsymbol e}_z$ and its magnitude is just $B_0 + \boldsymbol G\cdot\boldsymbol r$. Linear. ✓

If instead we had been "silly" and pointed the gradient along, say, $\hat{\boldsymbol e}_y$, the magnitude follows Pythagoras:

$$
B_\text{silly}(\boldsymbol r) = \sqrt{B_0^2 + (\boldsymbol G\cdot\boldsymbol r)^2}
$$

Non-linear. Ugly. Not what we want.

So by construction:

$$
\boxed{\,\boldsymbol B_\mathrm{G,ideal}(\boldsymbol r, t) = \bigl(\boldsymbol G(t)\cdot\boldsymbol r\bigr)\,\hat{\boldsymbol e}_z\,}
\qquad\Longleftrightarrow\qquad
\boldsymbol B_\mathrm{G,ideal}(\boldsymbol r, t)
= \begin{pmatrix} 0 \\ 0 \\ x G_x(t) + y G_y(t) + z G_z(t)\end{pmatrix}
$$

Only the z-component is non-zero, and it varies linearly with position.

### 2.3 Lines (planes) of constant frequency

Since $\omega_\mathrm{G}(\boldsymbol r) = \gamma\,\boldsymbol G\cdot\boldsymbol r$, points with the same frequency satisfy

$$
\boldsymbol G\cdot\boldsymbol r = \text{const.}
$$

This is the equation of a **plane perpendicular to $\boldsymbol G$**.

| Gradient | Constant-frequency planes |
|---|---|
| $G_x$ only | $yz$-planes (perpendicular to $x$) |
| $G_z$ only | $xy$-planes (perpendicular to $z$) |
| Oblique $\boldsymbol G$ | Oblique planes perpendicular to $\boldsymbol G$ |

**Intuition.** Each plane is one "frequency band". When the coil hears frequency $\omega_1$, it's hearing the sum of contributions from one entire plane — not one point. To localize a point, you need gradients in two (for a line) or three directions (for a voxel). This is the seed of how 2D/3D imaging works.

### 2.4 Oblique gradients = free choice of slice orientation

Because constant-frequency planes are always perpendicular to $\boldsymbol G$, and $\boldsymbol G$ can point in *any* direction, we can pick *any* slice orientation we like just by choosing the right combination $(G_x, G_y, G_z)$. **No need to physically rotate the patient — rotate the gradient instead.** Big clinical win.

---

## 3. How gradient fields are actually built

### 3.1 Real-world properties

In a modern clinical whole-body scanner:

- **Max amplitude**: typically 40–100 mT/m per axis (compare to $B_0 \sim 1.5$–3 T — gradients are *small*).
- **Currents**: up to ~500 A.
- **Power**: tens of kW → water cooling required.
- Coils are usually **copper wire cast in epoxy**, sitting just outside the bore.

### 3.2 The z-gradient: easy (Helmholtz-style)

Two loop coils, one at each end of the bore, with **currents in opposite directions**.

- Left coil at $\boldsymbol r_1$: field points one way along $z$.
- Right coil at $\boldsymbol r_3$: field points the opposite way.
- Center $\boldsymbol r_2$: contributions cancel → zero added field.

Going $\boldsymbol r_1 \to \boldsymbol r_2 \to \boldsymbol r_3$, the added field varies smoothly. With careful design (more windings, optimized geometry) you get a **nearly linear** dependence on $z$. The real field looks very close to the ideal one. 😊

### 3.3 The x- and y-gradient: harder

For a transverse gradient (say $x$), you need **four** loop coils — two top, two bottom — with carefully chosen current directions. On the $z = 0$ line you get something close to the ideal: a $B_z$ that varies linearly with $x$.

**But:** off the $z = 0$ line, the real field also has *transverse* components ($B_x$, $B_y$). These are **not** present in the ideal field. This is where things get interesting.

---

## 4. Why the real gradient field can *never* match the ideal one

This is the most beautiful idea in the lecture.

### 4.1 Maxwell strikes back

Check the ideal field against Maxwell's "no magnetic monopoles" equation $\nabla\cdot\boldsymbol B = 0$:

$$
\nabla\cdot\boldsymbol B_\mathrm{G,ideal}
= \frac{\partial}{\partial x}(0) + \frac{\partial}{\partial y}(0) + \frac{\partial}{\partial z}\bigl(xG_x + yG_y + zG_z\bigr)
= G_z
$$

Whenever $G_z \neq 0$, this is non-zero. **The ideal gradient field violates Maxwell's equations.**

No matter how clever the engineers are, **no physical coil can produce the textbook "ideal" field** — it's physically impossible.

### 4.2 Why MRI works anyway

Because $B_0$ is enormous (~1.5 T) compared to $B_\mathrm{G}$ (~10 mT at the edge of the FOV). When you add a small transverse vector $\boldsymbol B_{\mathrm{G},\perp}$ to a huge longitudinal $\boldsymbol B_0$, the magnitude of the resulting vector is dominated by the longitudinal part. The tilt angle $\Theta_\mathrm{G}$ between $\boldsymbol B_0$ and the total field is essentially zero.

Mathematically (good approximation):

$$
\boldsymbol B_\mathrm{G,ideal}(\boldsymbol r, t)
\;\approx\;
\bigl(\boldsymbol B_\mathrm{G,real}(\boldsymbol r, t)\cdot\hat{\boldsymbol e}_z\bigr)\,\hat{\boldsymbol e}_z
\;+\;
B_\mathrm{CC}(\boldsymbol r, t)\,\hat{\boldsymbol e}_z
$$

In words: **just use the z-component of the real field, and forget about the transverse components — except for one small correction, the concomitant field.**

### 4.3 The concomitant (Maxwell) field — derivation sketch

Start from Pythagoras for the magnitude of $\boldsymbol B_0 + \boldsymbol B_\mathrm{G}$:

$$
\lvert \boldsymbol B_0 + \boldsymbol B_\mathrm{G}\rvert
= \sqrt{(B_0 + B_{\mathrm{G},\parallel})^2 + B_{\mathrm{G},\perp}^2}
$$

Pull the dominant term out:

$$
= (B_0 + B_{\mathrm{G},\parallel})\sqrt{1 + \frac{B_{\mathrm{G},\perp}^2}{(B_0 + B_{\mathrm{G},\parallel})^2}}
$$

Use $\sqrt{1+x} \approx 1 + x/2$ for small $x$, and replace $(B_0 + B_{\mathrm{G},\parallel})^2 \to B_0^2$ in the denominator (the correction would be a correction-on-a-correction):

$$
\omega(\boldsymbol r)
\approx \gamma(B_0 + B_{\mathrm{G},\parallel})
\;+\; \underbrace{\frac{\gamma}{2}\frac{B_{\mathrm{G},\perp}^2}{B_0}}_{\Delta\omega_\mathrm{CC}}
$$

The correction term defines the **concomitant field** (also called the **Maxwell field**):

$$
\boxed{\,B_\mathrm{CC}(\boldsymbol r, t) = \frac{B_{\mathrm{G},\perp}^2(\boldsymbol r, t)}{2 B_0}\,}
$$

Key properties:

- **Always positive** ($B_{\mathrm{G},\perp}^2$ is a square).
- Scales as **$1/B_0$** — worse at low field, better at high field.
- Scales **quadratically with $G$** — so flipping the sign of a gradient does *not* flip the sign of the concomitant field. This **breaks the usual trick** where a $-G$ rewinds a $+G$.
- Usually **negligible** — but matters in phase contrast MRI, EPI, spiral imaging, low-field MRI.

### 4.4 Explicit formula

Using $\nabla\cdot\boldsymbol B = 0$ and $\nabla\times\boldsymbol B \approx 0$ (currents in tissue are tiny, gradient switching is slow), plus the symmetry of the gradient coils at the isocenter, one can derive a closed form:

$$
B_\mathrm{CC}(\boldsymbol r, t) = \frac{1}{2 B_0}\!\left[
\frac{G_z^2 (x^2 + y^2)}{4}
\;+\; z^2 (G_x^2 + G_y^2)
\;-\; x z\, G_z G_x
\;-\; y z\, G_z G_y
\right]
$$

You don't need to memorize this. Just notice: quadratic in gradients, quadratic in position, $1/(2B_0)$ overall — high field suffers less.

---

## 5. Key takeaways

1. A gradient makes $\omega$ position-dependent: $\omega(\boldsymbol r) = \gamma(B_0 + \boldsymbol G\cdot\boldsymbol r)$. **This is the seed of all MRI spatial encoding.**
2. The ideal gradient field points along $\hat{\boldsymbol e}_z$ so the magnitude is linear in $\boldsymbol r$.
3. Constant-frequency surfaces are **planes perpendicular to $\boldsymbol G$**. Any oblique slice can be selected by choosing the right $\boldsymbol G$.
4. z-gradients use 2 anti-parallel loop coils; x/y gradients use 4 loop coils. Modern coils are heavily optimized.
5. The textbook ideal gradient field **violates Maxwell's equations** ($\nabla\cdot\boldsymbol B = G_z$). No coil can produce it.
6. MRI works anyway because $B_0 \gg B_\mathrm{G}$: only the z-component of $\boldsymbol B_\mathrm{G}$ matters in good approximation.
7. The leftover error is the **concomitant (Maxwell) field**, $B_\mathrm{CC} = B_{\mathrm{G},\perp}^2 / (2 B_0)$. Small, always positive, scales as $G^2$, more important at low $B_0$ and large $r$.

---

## 📖 Glossary

| Term | Meaning |
|---|---|
| Gradient $\boldsymbol G$ | The vector $(G_x, G_y, G_z)$ describing the strength and direction of spatial variation of $B_z$. Units: T/m or mT/m. |
| Gradient field $\boldsymbol B_\mathrm{G}$ | The actual additional magnetic field produced by the gradient coils. |
| Isocenter | Geometric center of the scanner, where all gradients are designed to be zero. |
| Concomitant field $B_\mathrm{CC}$ | Small extra field that arises because the *real* gradient field must satisfy Maxwell's equations and therefore has unwanted transverse components. |
| Maxwell field | Synonym for concomitant field. |
| Oblique gradient | $\boldsymbol G$ that does not point along a single scanner axis — used to select tilted slices. |

---

## 🧪 Exercises

Try these before peeking at the solutions. They build directly on the material above.

> Useful: $\gamma/(2\pi) = 42.58$ MHz/T for $^1$H, so $\gamma = 2.675\times 10^8$ rad/(s·T).

### Exercise 1 — Frequency offset from a gradient
A 3 T scanner applies $G_x = 20$ mT/m along $x$. A proton sits at $x = 10$ cm from the isocenter ($y = z = 0$).
**(a)** What is the Larmor frequency at this position (in MHz)?
**(b)** By how much (in kHz) does it differ from a proton at the isocenter?

### Exercise 2 — Constant-frequency planes
A gradient is applied with $\boldsymbol G = (G, G, 0)^T$, $G > 0$. Describe geometrically the set of points where $\omega_\mathrm{G}(\boldsymbol r) = 0$.

### Exercise 3 — Why z?
Someone proposes a gradient coil whose ideal field points along $\hat{\boldsymbol e}_x$ instead of $\hat{\boldsymbol e}_z$: $\boldsymbol B_\mathrm{wrong}(\boldsymbol r, t) = (G_x(t)\,x)\,\hat{\boldsymbol e}_x$. Write down $\lvert \boldsymbol B_0 + \boldsymbol B_\mathrm{wrong}\rvert$ and explain in one sentence why this leads to a non-linear $\omega(x)$.

### Exercise 4 — Maxwell catches the textbook gradient
Verify by direct computation that the ideal gradient field $\boldsymbol B_\mathrm{G,ideal} = (xG_x + yG_y + zG_z)\hat{\boldsymbol e}_z$ has divergence $G_z$, and therefore violates $\nabla\cdot\boldsymbol B = 0$ whenever a z-gradient is on.

### Exercise 5 — Concomitant field magnitude
Use the explicit formula
$$
B_\mathrm{CC}(\boldsymbol r) = \frac{1}{2 B_0}\!\left[\frac{G_z^2 (x^2+y^2)}{4} + z^2 (G_x^2 + G_y^2) - x z\, G_z G_x - y z\, G_z G_y\right]
$$
to compute $B_\mathrm{CC}$ at $\boldsymbol r = (10, 0, 0)^T$ cm and at $\boldsymbol r = (0, 0, 10)^T$ cm, for a 1.5 T scanner with $G_x = 30$ mT/m and $G_y = G_z = 0$. Explain the difference.

### Exercise 6 — Sign trick
A common MRI trick is to rewind the phase accumulated under $+G$ by applying $-G$ for the same duration. Explain in one sentence why this trick **does not** undo the phase accumulated due to the concomitant field.

### Exercise 7 — Field-strength scaling
Two scanners are identical except for $B_0$ — one is 0.5 T, the other is 3 T. Both apply the same gradient. By what factor does $B_\mathrm{CC}$ differ? Which suffers more?

### Exercise 8 — Oblique slice
You want to image a slice with normal $\hat{\boldsymbol n} = (1,1,1)^T/\sqrt{3}$. Using only $G_x$, $G_y$, $G_z$, give one valid choice of $\boldsymbol G$ that produces this slice geometry.

### Exercise 9 — Frequency range across a field of view
For a $G_x = 40$ mT/m gradient and a 25 cm field of view in $x$ centered on the isocenter, what is the **total spread** of Larmor frequencies (peak to peak, in kHz) across the FOV? This is roughly the receive bandwidth needed.

### Exercise 10 — Order of magnitude
At $B_0 = 1.5$ T with $G_x = 30$ mT/m only, how large is $B_\mathrm{CC}$ at $z = 20$ cm, expressed as a fraction of $B_0$? Comment on whether this matters for typical sequences.

---

## ✅ Solutions

### Solution 1
At $x = 0.10$ m: $B = B_0 + G_x x = 3 + 0.020\cdot 0.10 = 3.002$ T.
**(a)** $f = (\gamma/2\pi) B = 42.58 \cdot 3.002 \approx 127.83$ MHz.
**(b)** $\Delta f = (\gamma/2\pi)\, G_x\, x = 42.58\times 10^6 \cdot 0.020 \cdot 0.10 \approx 85.2$ kHz.
This is the kind of frequency spread you encode across a FOV — it must fit inside your receiver bandwidth.

### Solution 2
Need $\boldsymbol G \cdot \boldsymbol r = G(x + y) = 0 \Rightarrow y = -x$. In 3D this is the plane containing the $z$-axis and the line $y = -x$ — the anti-diagonal plane. It is perpendicular to $\boldsymbol G$, as expected.

### Solution 3
$\boldsymbol B_0 \perp \boldsymbol B_\mathrm{wrong}$, so by Pythagoras $\lvert\boldsymbol B_0 + \boldsymbol B_\mathrm{wrong}\rvert = \sqrt{B_0^2 + (G_x x)^2}$. This is a square root of a quadratic in $x$ — *not* linear — so $\omega(x) = \gamma\sqrt{B_0^2 + (G_x x)^2}$ is non-linear and image encoding would be a mess.

### Solution 4
$$
\nabla\cdot\boldsymbol B_\mathrm{G,ideal}
= 0 + 0 + \frac{\partial}{\partial z}(xG_x + yG_y + zG_z) = G_z.
$$
Non-zero whenever $G_z \neq 0$, violating Maxwell. ∎

### Solution 5
With $G_y = G_z = 0$, the formula reduces to $B_\mathrm{CC} = z^2 G_x^2 / (2 B_0)$.

- At $\boldsymbol r = (0.10, 0, 0)^T$ m: $z = 0$, so $B_\mathrm{CC} = 0$.
- At $\boldsymbol r = (0, 0, 0.10)^T$ m: $B_\mathrm{CC} = (0.10)^2 (0.030)^2 / (2\cdot 1.5) = 3\times 10^{-6}$ T $= 3\ \mu$T.

**Interpretation:** with only $G_x$ active, the unwanted transverse $B$-components live *off* the gradient's own axis (specifically, where $z \neq 0$). Sitting on the x-axis you feel no concomitant field; sitting on the z-axis at the same distance, you feel $\sim 3\ \mu$T.

### Solution 6
Because $B_\mathrm{CC}$ is **quadratic** in the gradient: $B_\mathrm{CC} \propto G^2$. So $+G$ and $-G$ produce the *same* $B_\mathrm{CC}$, and the second pulse adds to (rather than cancels) the phase already accumulated.

### Solution 7
$B_\mathrm{CC} \propto 1/B_0$, so the 0.5 T scanner has $3/0.5 = 6\times$ larger concomitant field than the 3 T scanner. **Low-field MRI suffers more** — one of the reasons CC corrections matter most for low-field systems.

### Solution 8
Any $\boldsymbol G$ parallel to $\hat{\boldsymbol n}$. For example, $\boldsymbol G = G_0\,(1,1,1)^T/\sqrt{3}$ for some chosen amplitude $G_0$. A simpler (un-normalized) choice: $\boldsymbol G = (G, G, G)^T$ — same direction, just larger magnitude.

### Solution 9
Frequency offset from $G_x x$ at the edges: $\Delta f_\text{edge} = (\gamma/2\pi)\,G_x\,(\text{FOV}/2) = 42.58 \times 10^6 \cdot 0.040 \cdot 0.125 \approx 213$ kHz on each side.
**Total peak-to-peak spread:** $\approx 426$ kHz.

### Solution 10
$B_\mathrm{CC} = z^2 G_x^2/(2 B_0) = (0.20)^2 (0.030)^2 / 3 = 1.2\times 10^{-5}$ T.
As a fraction of $B_0$: $1.2\times 10^{-5} / 1.5 \approx 8\times 10^{-6}$, i.e. **8 ppm**.
For comparison, typical magnetic field inhomogeneity tolerances in MRI are ~1 ppm over the imaging volume. 8 ppm is not huge but is *not* negligible — it can cause noticeable phase errors in sensitive sequences (phase contrast, EPI off-resonance), which is exactly why CC corrections exist.

---

## 📚 Suggested reading

- **Bernstein, King, Zhou**, *Handbook of MRI Pulse Sequences*, Elsevier 2004 — the bible.
- **Bernstein et al.**, *Concomitant gradient terms in phase contrast MR: analysis and correction*, MRM 1998;39(2):300 — the original CC field paper.
- **Webb**, *New Developments in NMR — Magnetic Resonance Technology*, RSC 2016 — Ch. 5 on gradient hardware.

---

*Next up: L08 — Fourier Series and Fourier Transform. Once you understand gradients, the next question is: how does a position-encoded time signal become an image? Fourier is the answer.*
