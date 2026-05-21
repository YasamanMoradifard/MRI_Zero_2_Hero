# L05 — Signal Detection: Faraday's Law, Coil Sensitivity Profiles, and the B₀² Scaling

> **Scope.** This lecture is **chapter 7** of the notes (§7.1–§7.6.2). It answers the question "how does the scanner actually *hear* the precessing magnetization?" and introduces the **coil sensitivity profile c(r)** — a quantity that will return constantly in later modules (parallel imaging, SENSE, GRAPPA, multi-coil reconstruction).

**Source:** MRI1 lecture notes, ch. 7.1 — 7.6.2.

---

## 🎯 What you'll be able to do after this lecture

- State Faraday's law of induction and use it to compute the EMF in a coil from a known time-varying B field.
- Explain why MRI signal scales as **B₀²** — and why image quality nonetheless does *not* scale as B₀⁴.
- Write the signal as $\text{signal} = -\dot{\Phi}(t)$ with $\Phi(t) = \int \mathbf{c}(\mathbf{r})\cdot\mathbf{M}(\mathbf{r},t)\,dV$, and explain in plain language what c(r) is.
- Derive c(r) from the magnetic vector potential using Stokes' theorem (the bonus derivation).
- Compute c(r) for a circular loop coil on its center axis, and confirm:
  - the **R⁻¹ near-field** behavior (small coils win close-in),
  - the **r⁻³ far-field** behavior (dipole decay),
  - that only the M-component **parallel to S** contributes to the signal.
- Recognize that a loop coil with $\mathbf{S} \parallel \mathbf{B}_0$ produces **no MRI signal** (only the constant $M_z$ would couple to it, and $\dot{M}_z \approx 0$).

---

## 1. The basic mechanism (§7.1)

The story, in five beats:

1. **Principle 1** — protons (or other nuclei) in B₀ produce a magnetization **M**(r, t).
2. **Principle 2** — that magnetization generates a magnetic dipole field $\mathbf{B}_\text{dipole}(\mathbf{r}, t)$.
3. The transversal part of **M** precesses at $\omega_0$, so $\mathbf{B}_\text{dipole}$ rotates with it.
4. The dipole fields from all positions sum to a global rotating field $\mathbf{B}_1^-(\mathbf{r}, t)$.
5. A copper coil **detects** that rotating field by **Faraday induction** — the rate of change of magnetic flux through the coil drives an EMF in the wires.

The "$1^-$" is conventional notation for the *received* RF field (distinct from $\mathbf{B}_1^+$, the *transmitted* RF that excites the magnetization in the first place).

> **Why this matters.** The old picture of "the coil is an antenna receiving radio waves" is a useful first cartoon but slightly misleading. Faraday induction is the right model: the coil is a transformer secondary, with the magnetized tissue acting as the primary.

---

## 2. Faraday's law — quick reminder (§7.2)

In words: **the EMF around a closed loop equals the negative time derivative of the magnetic flux through it.**

$$
\text{emf}(t) = -\dot{\Phi}(t), \qquad \Phi(t) = \int_\text{coil surface} \mathbf{B}(\mathbf{r}, t)\cdot d\mathbf{S}
$$

A few conventions to keep straight:

- **dS** is a tiny surface element perpendicular to the (planar) coil plane. Its orientation is your choice — flipping S → −S flips dl → −dl and emf → −emf, so the equation is invariant.
- **dl** is the line element along the coil wires. Its direction is fixed by dS via a right-hand rule: thumb along dS → fingers curl along dl.
- The integral around the loop is what gives you the EMF — this is the actual voltage the receiver measures.

> The emf is, up to amplification and digitization, what the scanner records.

---

## 3. The B₀² scaling (§7.3)

Consider a stripped-down example: a square loop coil of side $L$ in the yz-plane (so $\mathbf{S} = -L^2 \mathbf{e}_x$), placed in a *spatially uniform* rotating field

$$
\mathbf{B}_1^-(t) = B_1^- \begin{pmatrix}\cos\omega_0 t\\ \sin\omega_0 t\\ 0\end{pmatrix}
$$

The flux is $\Phi(t) = \mathbf{B}_1^-(t)\cdot\mathbf{S} = -L^2 B_1^- \cos\omega_0 t$. The signal is:

$$
\text{signal}(t) = -\dot{\Phi}(t) = -L^2 B_1^- \omega_0 \sin\omega_0 t
$$

Four dependencies fall out:

| Factor | Scaling | Why |
|--------|---------|-----|
| Coil surface | $\propto S = L^2$ | Larger coil catches more flux |
| Field amplitude | $\propto B_1^-$ | More magnetization → bigger field |
| Larmor frequency | $\propto \omega_0 \propto B_0$ | Faster oscillation → bigger $\dot{\Phi}$ |
| Available M₀ | $\propto B_0$ (so $B_1^- \propto B_0$) | More polarization at higher field |

The last two combine:

$$
\boxed{\text{signal} \propto \omega_0 B_1^- \propto B_0^2}
$$

**B₀ enters twice.** Once via the *amount* of magnetization (more spins line up with stronger B₀), and once via the *speed* of precession (faster oscillation → larger $\dot{\Phi}$).

> **The caveat.** You might be tempted to conclude "doubling B₀ improves images 4×." Not so. What matters in practice is **SNR**, signal-to-noise ratio, and the dominant body noise scales sub-quadratically with B₀ at clinical strengths. Empirically SNR scales roughly **linearly** with B₀ at typical scanner field strengths up to ~7T — discussed later in the course.

**Side observation (left as an exercise in the lecture):** if the same coil were oriented with $\mathbf{S} \parallel \mathbf{e}_z$, then $\mathbf{B}_1^-(t)\cdot\mathbf{S} = 0$ for all $t$ — the field rotates in the xy-plane and never threads a coil whose normal is along z. **A coil with its face pointing along $\mathbf{B}_0$ picks up nothing.**

---

## 4. The coil sensitivity profile c(r) (§7.4)

In the uniform-field example, $\mathbf{B}_1^-$ had no spatial dependence. In reality, the dipole field of a sample at position **r** falls off as $1/|\mathbf{r}|^3$ from that sample, so coils are most sensitive to samples *near* them.

### 4.1 The definition (Eq. 7.1)

We *define* the coil sensitivity profile $\mathbf{c}(\mathbf{r})$ such that the magnetic flux can be written as a direct integral over the magnetization:

$$
\boxed{\Phi(t) = \int d^3\mathbf{r}\;\mathbf{c}(\mathbf{r})\cdot\mathbf{M}(\mathbf{r}, t)}
$$

This is huge — it bypasses computing $\mathbf{B}_1^-$ altogether and connects **signal directly to magnetization** (which is what you actually want to image). The "c" can mnemonically stand for "coil" or for "carpet" (everything B-field-related has been swept under it).

### 4.2 c(r) is a vector field

$\mathbf{c}(\mathbf{r}) = c(\mathbf{r})\mathbf{e}_c(\mathbf{r})$ has both magnitude and direction at every point:

- **Magnitude $c(\mathbf{r})$** — large near the coil, drops cubically in the far field (mirror of dipole decay).
- **Direction $\mathbf{e}_c(\mathbf{r})$** — encodes how the field lines emerging from a sample at **r** thread the coil. The sign in particular depends on whether B-field lines point through the coil in the +S or −S direction.

The vector character is essential: it determines *which component* of M at each position the coil is sensitive to. A coil pointing along $-\mathbf{e}_y$ at the origin is, on the y-axis, sensitive only to $M_y(\mathbf{r})$; the lecture works through the sign-flip example carefully to show this.

### 4.3 Why local receive coils

Because $c(\mathbf{r})$ drops with distance, modern MRI uses **many small surface coils** ("channels") placed near the anatomy. Each one has a *localized* sensitivity profile, and the images from individual coils are combined into a single high-SNR image. Typical hardware:

- 64-channel head/neck coils (a helmet of small loops)
- 16-channel knee coils
- 4-channel breast coils
- 32-channel spine arrays
- micro surface coils (down to 500 µm diameter for single-cell imaging)

This multi-coil setup is also what makes **parallel imaging** possible (Module 4: SENSE, GRAPPA) — knowing the different $\mathbf{c}_i(\mathbf{r})$ for each channel lets you reconstruct images from intentionally undersampled k-space.

A practical wrinkle: nearby coils inductively couple to each other. Suppressing this coupling (by precise geometric overlap, decoupling networks, or low-impedance preamps) is a serious electrical-engineering challenge in coil design.

---

## 5. Deriving c(r) (§7.5)

The point of this derivation is to **prove** that an expression for c(r) actually exists, and to give us a usable formula.

### 5.1 Setup

A small sample of volume dV at the origin with magnetization **M** produces the dipole field:

$$
\mathbf{B}_\text{dipole}(\mathbf{r}) = \mathbf{b}_\text{dipole}(\mathbf{r})\,dV, \qquad \mathbf{b}_\text{dipole}(\mathbf{r}) = \frac{\mu_0}{4\pi}\left[\frac{3(\mathbf{r}\cdot\mathbf{M})\mathbf{r}}{r^5} - \frac{\mathbf{M}}{r^3}\right]
$$

Adding many sources gives, in the continuum limit:

$$
\mathbf{B}_1^-(\mathbf{r}, t) = \int d^3\mathbf{r}_1\;\mathbf{b}_\text{dipole}(\mathbf{r} - \mathbf{r}_1)
$$

### 5.2 The vector-potential trick (bonus)

From electrodynamics: there exists a vector potential **A** with $\nabla\times\mathbf{A} = \mathbf{B}$, and for a magnetic dipole:

$$
\mathbf{A}_\text{dipole}(\mathbf{r}) = \frac{\mu_0\,dV}{4\pi}\frac{\mathbf{M}\times\mathbf{r}}{r^3}
$$

Now apply **Stokes' theorem** to flip the flux integral from a surface integral into a line integral around the coil:

$$
\Phi(t) = \int_\text{surf}\mathbf{B}_1^-\cdot d\mathbf{S} = \int_\text{surf}(\nabla\times\mathbf{A}_1^-)\cdot d\mathbf{S} = \oint_\text{coil}\mathbf{A}_1^-\cdot d\mathbf{l}
$$

Plug in the integrated dipole vector potential:

$$
\Phi(t) = \oint d\mathbf{l}\cdot\int d^3\mathbf{r}_1\;\frac{\mu_0}{4\pi}\frac{\mathbf{M}(\mathbf{r}_1, t)\times(\mathbf{r}-\mathbf{r}_1)}{|\mathbf{r}-\mathbf{r}_1|^3}
$$

Use the **scalar-triple-product identity** $\mathbf{a}\cdot(\mathbf{b}\times\mathbf{c}) = -\mathbf{b}\cdot(\mathbf{a}\times\mathbf{c})$ to swap which vector sits in the cross product, pull the **r**₁ integral outside, and identify the kernel that multiplies $\mathbf{M}(\mathbf{r}_1)$:

$$
\boxed{\mathbf{c}(\mathbf{r}_1) = -\frac{\mu_0}{4\pi}\oint_\text{coil}d\mathbf{l}\times\frac{\mathbf{r}-\mathbf{r}_1}{|\mathbf{r}-\mathbf{r}_1|^3}}
$$

This is the explicit formula for the sensitivity profile (Eq. 7.2 in the notes). It looks like a **Biot–Savart law** — exactly the kind of integral you'd write down for the magnetic field of a current loop. In fact, this is no coincidence: by **electromagnetic reciprocity**, the receive sensitivity of a coil at point **r**₁ equals (up to factors) the magnetic field the coil would produce at **r**₁ if you ran unit current through it. Bracketed below for emphasis because we'll use it constantly:

> **Reciprocity principle:** to compute how sensitive a coil is at point **r**₁, just compute the B-field that coil would make at **r**₁ as a transmitter. Same integral.

---

## 6. Worked example 1 — circular loop, sample on axis (§7.6.1)

Take a circular loop of radius $R$ at the origin with $\mathbf{S} = -\pi R^2\,\mathbf{e}_y$ (loop in the xz-plane, normal pointing in $-y$), and a sample at $\mathbf{r}_1 = (0, y_1, 0)$ with $y_1 < 0$.

Parametrize the loop:
- $\mathbf{r}(\theta) = R(\cos\theta, 0, \sin\theta)^T$
- $d\mathbf{l} = R(-\sin\theta, 0, \cos\theta)^T\,d\theta$
- $|\mathbf{r}-\mathbf{r}_1| = \sqrt{R^2 + y_1^2}$ (constant in θ — that's why this case is tractable)
- $d\mathbf{l}\times(\mathbf{r}-\mathbf{r}_1) = R\,d\theta\,(y_1\cos\theta, R, y_1\sin\theta)^T$

Integrating $\theta \in [0, 2\pi]$ kills the $\cos$ and $\sin$ terms, leaving:

$$
\boxed{\mathbf{c}(\mathbf{r}_1) = \frac{\mu_0}{2}\frac{R^2}{(R^2 + y_1^2)^{3/2}}\begin{pmatrix}0\\ -1\\ 0\end{pmatrix}}
$$

### 6.1 Reading the result

**Only $M_y$ contributes to the signal.** From $\Phi \approx \mathbf{c}(\mathbf{r}_1)\cdot\mathbf{M}(\mathbf{r}_1)\,dV$ and the $(0, -1, 0)^T$ direction of $\mathbf{c}$, we get $\Phi \approx -c(\mathbf{r}_1)M_y(\mathbf{r}_1)\,dV$. The $M_x$ and $M_z$ components are perpendicular to **c** and contribute nothing.

But wait — what *physically* drives the signal? Only the transverse magnetization that's **precessing**. So as the magnetization rotates between pointing along $+\mathbf{e}_x$ and $+\mathbf{e}_y$, the flux modulates as $M_y(t) = |M_\perp|\sin\omega_0 t$, and $\dot{\Phi}$ is non-zero.

**Two limiting regimes:**

- **Near field** ($y_1 \to 0$): $\mathbf{c}(\mathbf{r}_1) \to \frac{\mu_0}{2R}(0, -1, 0)^T$. The magnitude is $\propto R^{-1}$ — **smaller coils win** for close-in samples. This is why micro-coils can image single cells.
- **Far field** ($|y_1| \gg R$): $c(\mathbf{r}_1) \to \frac{\mu_0}{2}\frac{R^2}{|y_1|^3}$ — **cubic decay**, mirroring the dipole field of the coil itself.

### 6.2 If $\mathbf{S} \parallel \mathbf{e}_z$ — no signal!

Redo the same integral with $\mathbf{S} = -S\,\mathbf{e}_z$ and you get $\mathbf{c}(\mathbf{r}_1) = c(\mathbf{r}_1)(0, 0, -1)^T$ on the y-axis. Now **c** is along z, so only $M_z$ couples. But $M_z$ doesn't precess — it's effectively constant (the only thing that changes it is the slow T₁ recovery, on millisecond–second timescales). So $\dot{\Phi} \approx 0$ and the signal vanishes.

> **Rule of thumb:** for a coil to detect MRI signal, its normal **must have a component in the transverse plane** (perpendicular to B₀). A coil whose face points along $\mathbf{B}_0$ is blind.

---

## 7. Worked example 2 — far-field approximation (§7.6.2)

Now the sample is off-axis at $\mathbf{r}_1 = (x_1, y_1, z_1)^T$, in the **far field** ($|\mathbf{r}_1| \gg R$) so the coil looks small from where the sample sits. Don't integrate around the loop — instead use the dipole-field formula directly, treating the coil's dipole moment as $\mathbf{S}$ scaled appropriately.

For $\mathbf{S} = -S\,\mathbf{e}_x$ (coil normal along $-x$):

$$
\mathbf{c}(\mathbf{r}_1) \approx -\frac{\mu_0 S}{4\pi r_1^5}\begin{pmatrix}3x_1^2 - r_1^2\\ 3 x_1 y_1\\ 3 x_1 z_1\end{pmatrix}
$$

For $\mathbf{S} = -S\,\mathbf{e}_z$ (coil normal along $-z$):

$$
\mathbf{c}(\mathbf{r}_1) \approx -\frac{\mu_0 S}{4\pi r_1^5}\begin{pmatrix}3 x_1 z_1\\ 3 y_1 z_1\\ 3 z_1^2 - r_1^2\end{pmatrix}
$$

> **Heads up on a typo.** The original lecture writes the y-component of the second case as `3 z_1` which is dimensionally wrong — by the structure of the dipole formula it should be $3 y_1 z_1$. I've corrected that here.

The key takeaways:

- **All three components of c can be non-zero** when the sample is off-axis — both $M_x$ and $M_y$ contribute to the signal (the transverse parts both modulate Φ).
- **$M_z$ still doesn't contribute to the time-varying signal** even when $c_z \neq 0$, because $\dot{M}_z \approx 0$.
- On the **z-axis center line** of an $\mathbf{S} \parallel \mathbf{e}_z$ coil, all transverse $c$-components vanish — that's the "blind spot" of a coil oriented along B₀.

---

## 🧠 Self-test questions

1. **Faraday quick check.** A circular coil of area 10 cm² lies in the xz-plane. A uniform field $\mathbf{B}(t) = B_0\cos(\omega t)\mathbf{e}_y$ threads it. What is $\Phi(t)$? What is the EMF? At what times is the EMF maximal in magnitude?
2. **B₀² intuition.** Explain in two sentences why MRI signal scales as $B_0^2$ rather than $B_0^1$. What happens to that argument if you use a *hyperpolarized* tracer (where M₀ doesn't depend on B₀)?
3. **Coil orientation.** A patient lies supine in the bore (z = head–foot axis). Where should you put a loop coil to image their left kidney? Should its face point up, sideways, or along z? Why?
4. **The $\mathbf{S} \parallel \mathbf{e}_z$ paradox.** A loop coil's normal is along B₀. Its sensitivity profile $\mathbf{c}(\mathbf{r})$ is non-zero almost everywhere. Why does it still pick up no signal from a precessing transverse magnetization?
5. **Reciprocity.** Without computing the line integral, sketch the magnetic field lines a circular loop coil would produce if you ran 1 A through it. Where is the field strongest? Where does it point? What does that tell you about the coil's sensitivity profile?
6. **R⁻¹ vs r⁻³.** A loop coil of radius $R = 5$ cm sits on a patient's skin. Compute the ratio $c(\text{skin})/c(\text{depth } 10\text{ cm})$. Repeat for $R = 1$ cm. Which coil gives more SNR contrast between surface and deep tissue?
7. **Multi-coil combination.** If you have two coils with profiles $\mathbf{c}_1, \mathbf{c}_2$ producing signals $s_1, s_2$, and you combine them as $s_\text{comb} = w_1 s_1 + w_2 s_2$, what choice of weights $w_i$ maximizes SNR at point **r**? (This is the "sum-of-squares" / "matched filter" combination — you'll meet it again in parallel imaging.)
8. **No DC signal.** Why does $M_z(\mathbf{r}, t)$ never directly produce signal in a standard MR experiment, even when $\mathbf{c}(\mathbf{r})$ has a z-component? What experimental trick could you use to make $M_z$ visible?
9. **Order-of-magnitude check.** A typical proton M₀ in tissue at 3 T is about $10^{-2}$ A/m. A typical receive coil has radius 5 cm, sample at depth 5 cm. Estimate the EMF at $\omega_0 = 2\pi \cdot 128$ MHz. Compare to the thermal noise voltage $V_n = \sqrt{4 k_B T R \Delta f}$ for a 50 Ω coil at body temperature and 100 kHz bandwidth.
10. **The vector potential argument.** Why did we have to detour through **A** to derive c(r)? What goes wrong if you try to integrate **B**_dipole directly through the coil surface?

---

## 🔗 Connections to other lectures

- **L02 / chapter 4** introduced Principles 1 and 2. Principle 2 — "magnetization generates a dipole field" — is what makes Faraday detection possible at all.
- **L02 / chapter 5** showed that transverse M precesses at $\omega_0$. That precession is the *only* thing that gives $\dot{\Phi} \neq 0$ here.
- **L03–L04 / chapter 6** showed how to *create* transverse magnetization via RF excitation. Without that, there's nothing for the coil to hear.
- **Slice selection (L11):** the same c(r) determines which slice contributes to a given coil's signal.
- **Module 4 — parallel imaging (SENSE, GRAPPA):** these methods are *literally* algorithms that invert the multi-coil sensitivity matrix $\{\mathbf{c}_i(\mathbf{r})\}$ to recover an image from undersampled k-space. The c(r) functions you compute today are the inputs to those methods.
- **Module 5 — deep-learning reconstruction:** modern unrolled networks for MR recon (MoDL, Variational Networks) still embed the multi-coil forward model — you can't avoid c(r).

---

## 📚 Suggested reading

- **Hoult, D.I., and Richards, R.E.** (1976). "The signal-to-noise ratio of the nuclear magnetic resonance experiment." *J. Magn. Reson.* 24:71–85. The classic SNR-scaling paper.
- **Roemer, P.B. et al.** (1990). "The NMR phased array." *Magn. Reson. Med.* 16:192–225. The paper that introduced multi-coil arrays — required reading before Module 4.
- **Pruessmann, K.P. et al.** (1999). "SENSE: Sensitivity encoding for fast MRI." *Magn. Reson. Med.* 42:952–962. The paper that turned c(r) into a reconstruction tool.
- **Jackson, J.D.** *Classical Electrodynamics* — chapters 5 (magnetostatics) and 6 (Maxwell's equations) for the Faraday/vector-potential background.

---

*Next up: how do gradients turn a single Larmor frequency into a *spatially varying* one — and unlock spatial encoding?*
