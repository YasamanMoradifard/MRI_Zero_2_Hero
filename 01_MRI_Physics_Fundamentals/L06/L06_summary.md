# L06 — From Faraday to the Complex Signal Equation

> **Scope.** This PDF is **chapter 7 continued** (§7.8–§7.15). It picks up where L05 left off — having defined Faraday's law and the coil sensitivity profile $\mathbf{c}(\mathbf{r})$ — and ends with **the signal equation that the rest of the course uses**:
>
> $$\text{signal}_\text{complex}(t_m) = \omega_0 \int d^3\mathbf{r}_1\,M_+'(\mathbf{r}_1, t_m)\,c_+^*(\mathbf{r}_1)$$
>
> If you only remember one equation from Module 1, make it this one. **k-space, image reconstruction, parallel imaging, all of Module 4** — they're all manipulations of this single expression.

**Source:** MRI1 lecture notes, ch. 7.8 — 7.14. (Sections 7.12 "Visualization" and 7.15 "Further reading" are marked *missing* in the PDF; we'll work without them.)

---

## 🎯 What you'll be able to do after this lecture

- Recognize that $M_z$ never contributes to the measured signal — only the transverse parts of $\mathbf{M}$ and $\mathbf{c}$ matter.
- Solve the Bloch equation in **complex-plane notation**: $\dot M_+(t) = i\omega(t)\,M_+(t)$ with solution $M_+(t) = M_+(0)e^{i\beta(t)}$.
- Decompose the magnetization phase into the fast Larmor part ($\omega_0 t$) and the slow off-resonance part ($\int\Delta\omega\,dt'$), and switch fluently to the rotating frame.
- Geometrically interpret the emf: the **transverse magnetic flux $\Phi_\perp$** is maximal when $\mathbf{M}_\perp \parallel \mathbf{c}_\perp$, while the **emf** is maximal when $\mathbf{M}_\perp \perp \mathbf{c}_\perp$ (90° out of phase — Faraday's $-\dot\Phi$ in action).
- Walk through **quadrature demodulation**: split the coil voltage, multiply with sin and cos references, low-pass filter, and digitize the real and imaginary signals.
- State and use the **complex signal equation**: digitized signal = integrated RRF magnetization weighted by the complex-conjugate of the transverse coil sensitivity.
- Recognize the **reciprocity principle** in its Biot–Savart form, and know that it breaks down at very high fields ($B_0 \gtrsim 7$ T in humans).

---

## 1. Only the transverse parts matter (§7.8)

The signal is $\text{emf} = -\dot\Phi(t)$. Pull out the z-component of the integrand:

$$
\text{emf} = -\frac{d}{dt}\int d^3\mathbf{r}_1\,\mathbf{M}(\mathbf{r}_1, t)\cdot\mathbf{c}(\mathbf{r}_1)
$$

Now $M_x, M_y$ oscillate at the Larmor frequency but $M_z$ is essentially constant (it only changes on the slow T₁ timescale). So $\dot M_z \approx 0$ and the z-component of M drops out:

$$
\boxed{\text{emf} = -\frac{d}{dt}\int d^3\mathbf{r}_1\,\mathbf{M}_\perp(\mathbf{r}_1, t)\cdot\mathbf{c}_\perp(\mathbf{r}_1)}
$$

Call the corresponding flux $\Phi_\perp(t) := \int d^3\mathbf{r}_1\,\mathbf{M}_\perp\cdot\mathbf{c}_\perp$ — the **transverse magnetic flux**. Everything that follows can be done in 2D (the transverse plane), which is exactly what makes complex-plane notation natural here.

> **Why $M_z$ doesn't show up:** the receive coil really *does* couple to $M_z$ if its sensitivity has a z-component. But coils detect $-\dot\Phi$, not $\Phi$. Static fluxes are invisible. The same coil that's blind to $M_z$ would happily measure a slowly-changing magnetic moment, e.g. via T₁ recovery — but on those timescales the signal is far weaker than the Larmor oscillation.

---

## 2. The Bloch equation in complex notation (§7.9)

### 2.1 The setup

Consider a B-field aligned with z but with a small position- and time-dependent perturbation:

$$
\mathbf{B}(\mathbf{r}, t) = (B_0 + \Delta B(\mathbf{r}, t))\,\mathbf{e}_z, \qquad \Delta B \ll B_0
$$

This is more general than it looks. $\Delta B$ can be:
- a gradient field (next lecture)
- field inhomogeneity from air-tissue interfaces
- chemical-shift offsets (fat vs water)

Multiply by $\gamma$ to switch to angular frequencies:

$$
\omega(\mathbf{r}, t) = \gamma B(\mathbf{r}, t) = \omega_0 + \Delta\omega(\mathbf{r}, t)
$$

> **Note from §7.9:** The lecture mentions in passing that B-field components along x or y are "mostly irrelevant" for MRI besides the oscillating $\mathbf{B}_1^+$ field. This is at first counterintuitive but turns out to be true for gradient fields — only the z-component matters because the perpendicular components are too small to compete with B₀ for the direction of precession. We'll see this in the next lecture.

### 2.2 The derivation

Starting from $\dot{\mathbf{M}} = \gamma\mathbf{B}\times\mathbf{M}$ with $\mathbf{B} \parallel \mathbf{e}_z$:

$$
\dot{\mathbf{M}}(t) = \omega(t)\,\mathbf{e}_z\times\mathbf{M}_\perp(t) = \omega(t)\begin{pmatrix}-M_y\\ M_x\\ 0\end{pmatrix}
$$

In components: $\dot M_x = -\omega M_y$, $\dot M_y = \omega M_x$, $\dot M_z = 0$. The first two are the real and imaginary parts of a single complex equation. Define $M_+ := M_x + i M_y$:

$$
\dot M_+ = \dot M_x + i\dot M_y = -\omega M_y + i\omega M_x = i\omega(M_x + i M_y) = i\omega(t)\,M_+
$$

$$
\boxed{\dot M_+(t) = i\omega(t)\,M_+(t)}
$$

### 2.3 The solution

This is a 1D linear ODE with time-varying coefficient. The solution:

$$
\boxed{M_+(t) = M_+(0)\,e^{i\beta(t)}, \qquad \beta(t) = \int_0^t \omega(t')\,dt'}
$$

Verify by differentiating: $\dot M_+(t) = M_+(0)\,e^{i\beta(t)}\,i\dot\beta(t) = i\omega(t)\,M_+(t)$. ✓

### 2.4 Three useful phases

The phase $\beta(t)$ is the **accumulated** angle in time $t$. The total phase of $M_+(t)$ depends on where it started:

$$
M_+(t) = |M_\perp(0)|\,e^{i\phi(t)}, \qquad \phi(t) = \beta(t) + \phi_0
$$

Splitting $\omega = \omega_0 + \Delta\omega$:

$$
\phi(t) = \underbrace{\omega_0 t}_\text{fast Larmor} + \underbrace{\int_0^t\Delta\omega(t')\,dt'}_\text{slow off-resonance} + \phi_0
$$

In the rotating frame at $\Omega = \omega_0$, the fast part disappears, leaving the **RRF phase**:

$$
\boxed{\varphi(t) = \int_0^t\Delta\omega(t')\,dt' + \phi_0}
$$

And the RRF magnetization is:

$$
M_+'(t) := M_\perp\,e^{i\varphi(t)}
$$

The lecture's Fig. 7.9.ii visualizes this neatly: lab-frame $M_+$ spins around at $\omega_0$ + slow drift; in the RRF, only the slow drift $\Delta\omega$ matters.

> **Why this matters:** $\varphi(t)$ is the *encoding phase*. When we add gradients next lecture, $\Delta\omega$ becomes spatially varying ($\Delta\omega(\mathbf{r}) = \gamma\mathbf{G}\cdot\mathbf{r}$), and the integrated phase $\varphi$ becomes position-dependent. *That's* what writes spatial information into the signal.

---

## 3. The geometric interpretation (§7.10–§7.11)

### 3.1 Putting things into complex form

Write both M⊥ and c⊥ as complex numbers with magnitude and phase:

$$
M_+(\mathbf{r}_1, t) = M_\perp(\mathbf{r}_1)\,e^{i\phi(\mathbf{r}_1, t)}, \qquad c_+(\mathbf{r}_1) = c_\perp(\mathbf{r}_1)\,e^{i\theta_c(\mathbf{r}_1)}
$$

The dot product is just $\mathbf{M}_\perp\cdot\mathbf{c}_\perp = M_\perp c_\perp \cos(\phi - \theta_c)$. So:

$$
\Phi_\perp(t) = \int d^3\mathbf{r}_1\,M_\perp(\mathbf{r}_1)\,c_\perp(\mathbf{r}_1)\,\cos(\phi(\mathbf{r}_1,t) - \theta_c(\mathbf{r}_1))
$$

### 3.2 Taking $-\dot\Phi$

The only time-dependent thing is $\phi(\mathbf{r}_1, t)$. Differentiating the cosine introduces a $-\sin$ and a factor of $\dot\phi \approx \omega_0$ (since $\Delta\omega \ll \omega_0$):

$$
\boxed{\text{emf} \approx \omega_0\int d^3\mathbf{r}_1\,M_\perp(\mathbf{r}_1)\,c_\perp(\mathbf{r}_1)\sin(\phi(\mathbf{r}_1,t) - \theta_c(\mathbf{r}_1))}
$$

### 3.3 Two geometric pictures (lecture's Figs. 7.11.i, iv, v)

**For Φ⊥:** max when $\mathbf{M}_\perp \parallel \mathbf{c}_\perp$ (aligned or anti-aligned), zero when perpendicular. This makes physical sense from the field-line picture: a magnetization pointing toward (or away from) the coil produces field lines threading the coil in maximum density.

**For the emf:** max when $\mathbf{M}_\perp \perp \mathbf{c}_\perp$, zero when parallel. This is **Faraday's −dΦ/dt at work**: when Φ is at its peak ($\phi - \theta_c = 0$ or $\pi$), it's momentarily not changing — so the emf is zero. When Φ passes through zero ($\phi - \theta_c = \pm\pi/2$), it's changing fastest — so the emf peaks.

The picture (Fig. 7.11.vi in the lecture): the emf contribution from a point $\mathbf{r}_1$ is the **projection of $\mathbf{M}_\perp$ onto a 90°-rotated $\mathbf{c}_\perp$**. Visually: pick up $\mathbf{c}_\perp$, rotate it a quarter turn, and ask how much of $\mathbf{M}_\perp$ points that way.

> **Why a coil isn't blind even when $\phi = \theta_c$ at some instant:** the magnetization is precessing fast, so a moment later $\phi$ has moved on. Across the duration of an MRI experiment (milliseconds), every coil sees every magnetization direction many times. The "blind angles" are instantaneous, not structural.

### 3.4 The §7.12 gap

Section 7.12 ("Visualization of the signal equation") is *missing* in the PDF. We move directly to §7.13.

---

## 4. Quadrature demodulation (§7.13)

The emf oscillates at ~128 MHz at 3 T — far too fast to digitize directly with an ADC. The trick is to **shift it down to baseband** by multiplying it with a reference oscillator at $\omega_0$, then low-pass filtering.

### 4.1 The trig identities

$$
2\sin a\sin b = \cos(a-b) - \cos(a+b)
$$
$$
2\sin b\cos a = \sin(b-a) + \sin(a+b)
$$

If $a, b$ both contain a fast piece $\omega_0 t$ plus slow corrections, then **$a - b$** contains only the slow corrections, while **$a + b$** is super-fast (~$2\omega_0$). The fast piece gets killed by a low-pass filter; the slow piece is what we keep.

### 4.2 The "I" channel (real part)

Take the emf and multiply by a sine reference at $\omega_0$:

$$
\sin(\phi_0 + \omega_0 t + \Delta\omega t - \theta_c)\cdot\sin(\omega_0 t) = \tfrac{1}{2}\cos(\phi_0 + \Delta\omega t - \theta_c) - \tfrac{1}{2}\cos(2\omega_0 t + \cdots)
$$

The first term is slow (~kHz scale, set by gradients and tissue inhomogeneity). The second is fast (~256 MHz). After a low-pass filter, only the first remains:

$$
\text{signal}_\text{re}(t_m) = \omega_0\sum_n M_\perp(\mathbf{r}_n)\,c_\perp(\mathbf{r}_n)\cos(\phi_0 + \Delta\omega(\mathbf{r}_n)\,t_m - \theta_c(\mathbf{r}_n))
$$

**Notice the sine became a cosine.** This is just trig — but it'll matter in a moment.

### 4.3 The "Q" channel (imaginary part)

Repeat with a cosine reference. By the second trig identity:

$$
\text{signal}_\text{im}(t_m) = \omega_0\sum_n M_\perp(\mathbf{r}_n)\,c_\perp(\mathbf{r}_n)\sin(\phi_0 + \Delta\omega(\mathbf{r}_n)\,t_m - \theta_c(\mathbf{r}_n))
$$

In hardware: the coil voltage is split, one branch multiplied by sin reference (producing the I/real channel), the other by cos reference (producing the Q/imaginary channel). Each is low-pass-filtered and digitized.

### 4.4 The complex signal — the punchline

Combine them as the real and imaginary parts of a complex number:

$$
\text{signal}_\text{complex}(t_m) = \text{signal}_\text{re}(t_m) + i\,\text{signal}_\text{im}(t_m)
$$

Use $\cos x + i\sin x = e^{ix}$:

$$
\text{signal}_\text{complex}(t_m) = \omega_0\sum_n M_\perp(\mathbf{r}_n)\,c_\perp(\mathbf{r}_n)\,e^{i\phi_0 + i\Delta\omega(\mathbf{r}_n)\,t_m - i\theta_c(\mathbf{r}_n)}
$$

Now recognize the structure. We had $M_+'(\mathbf{r}_n, t) = M_\perp(\mathbf{r}_n)\,e^{i(\phi_0 + \Delta\omega t_m)}$ (the RRF magnetization) and $c_+(\mathbf{r}_n) = c_\perp(\mathbf{r}_n)\,e^{i\theta_c(\mathbf{r}_n)}$ (the complex sensitivity). The combination factors:

$$
\boxed{\text{signal}_\text{complex}(t_m) = \omega_0\int d^3\mathbf{r}_1\,M_+'(\mathbf{r}_1, t_m)\,c_+^*(\mathbf{r}_1)}
$$

**This is the signal equation used in the rest of the course.**

What we've done:
- $M_+'(\mathbf{r}, t)$ is the **complex transverse magnetization in the RRF** — exactly the thing that gradient encoding will modulate
- $c_+^*(\mathbf{r})$ is the **complex-conjugate transverse coil sensitivity** — a fixed property of the coil geometry
- They get integrated over the imaging volume
- The result is the **complex sample** the computer stores

This is beautiful for several reasons:
1. The fast $\omega_0$ oscillation has been removed — we work at baseband.
2. We work in the RRF — exactly where excitation and gradient analysis live.
3. The signal is *linear* in the magnetization — so signals from many small voxels just add (no superposition issues).
4. The signal is *complex* — both amplitude *and* phase are available.

> **Why this matters in Module 2 / k-space:** with a gradient $\mathbf{G}$, the off-resonance becomes spatial: $\Delta\omega(\mathbf{r}) = \gamma\mathbf{G}\cdot\mathbf{r}$. Then $M_+' = M_\perp(\mathbf{r})\,e^{i\gamma\mathbf{G}\cdot\mathbf{r}\,t}$, and the signal becomes $\omega_0\int d^3\mathbf{r}\,M_\perp(\mathbf{r})\,c_+^*(\mathbf{r})\,e^{i\mathbf{k}(t)\cdot\mathbf{r}}$ with $\mathbf{k}(t) = \gamma\mathbf{G}t/(2\pi)$. **The signal *is* the Fourier transform of $M_\perp$ weighted by $c_+^*$.** That's the whole story of MR imaging.

---

## 5. Reciprocity (§7.14)

Comparing the c(r) formula from L05:

$$
\mathbf{c}(\mathbf{r}_1) = -\frac{\mu_0}{4\pi}\oint \frac{d\mathbf{r}\times(\mathbf{r}_1 - \mathbf{r})}{|\mathbf{r}_1 - \mathbf{r}|^3}
$$

to the **Biot–Savart law** for the B-field of a current-carrying loop:

$$
\mathbf{B}(\mathbf{r}_1) = I\,\frac{\mu_0}{4\pi}\oint \frac{d\mathbf{r}\times(\mathbf{r}_1 - \mathbf{r})}{|\mathbf{r}_1 - \mathbf{r}|^3}
$$

The two are identical up to a minus sign and a factor of $I$. **The receive sensitivity of a coil at $\mathbf{r}_1$ equals (up to convention factors) the B-field that coil would produce at $\mathbf{r}_1$ if you ran current through it.** We verified this numerically in L05.

### 5.1 When reciprocity breaks down

The derivation assumed the dipole field appears *instantaneously* at the coil. This holds when the size of the coil (and the body) is much smaller than the wavelength of the RF signal.

At 3 T, the wavelength in tissue is roughly **30 cm** — already comparable to a head. At 7 T, it's about **13 cm** — *smaller* than a head. At that point you can't treat the coil as a lumped element anymore; you need to model the actual EM wave propagation, including standing-wave patterns inside the body. This is called the **dielectric or wavelength effect**, and it's why ultra-high-field MRI (7 T and above) needs special transmit/receive coil design.

The lecture flags that this issue is treated in MRI 2.

### 5.2 Practical upshot

For most clinical scanners ($B_0 \le 3$ T), reciprocity is excellent and lets coil designers verify their designs by computing transmit B-fields with EM simulators (CST, FEKO, COMSOL) instead of having to model receive sensitivity separately.

---

## 6. The §7.15 gap

Section 7.15 ("Further reading") is also marked *missing* in the PDF. Here are some pointers that fit the spirit of the lecture:

- **Hoult, D.I., and Lauterbur, P.C.** (1979). "The sensitivity of the zeugmatographic experiment involving human samples." *J. Magn. Reson.* 34:425–433. The classic paper analyzing the noise side of SNR in MRI.
- **Hoult, D.I.** (2000). "The principle of reciprocity in signal strength calculations — a mathematical guide." *Concepts in Magnetic Resonance* 12:173–187. Rigorous treatment of reciprocity, including where it fails.
- **Collins, C.M., and Wang, Z.** (2011). "Calculation of radiofrequency electromagnetic fields and their effects in MRI of human subjects." *Magn. Reson. Med.* 65:1470–1482. A modern look at the high-field wavelength effect.

---

## 🧠 Self-test questions

1. **Why $M_z$ is invisible.** Explain in one sentence why the standard MR experiment cannot detect $M_z$, even though the coil's sensitivity $\mathbf{c}(\mathbf{r})$ generally has a non-zero z-component.

2. **Verify the complex Bloch solution.** Suppose $\omega(t) = \omega_0$ (constant). Show that $M_+(t) = M_+(0)e^{i\omega_0 t}$ satisfies $\dot M_+ = i\omega M_+$ and write out the real and imaginary parts in vector form. What's $M_x(t)$ if $M_+(0) = M_0$?

3. **Accumulated phase from a gradient pulse.** A gradient pulse $G_x(t)$ is on for time $T$ and creates $\Delta\omega(\mathbf{r}, t) = \gamma G_x(t)\,x$. Write the RRF phase $\varphi(\mathbf{r}, T)$. If $G_x$ is constant, what's the relationship between $\varphi$ and $x$?

4. **The Φ vs emf 90° shift.** Why is the emf zero when $\mathbf{M}_\perp \parallel \mathbf{c}_\perp$ (peak Φ) and maximal when they're perpendicular (zero Φ)? Connect to the picture of a sinusoid and its derivative.

5. **Demodulator math.** Take the signal $\sin(\omega_0 t + 0.1 t - 0.3)$ (so $\Delta\omega = 0.1$ rad/s, $\phi_0 - \theta_c = -0.3$). Multiply it by $\sin(\omega_0 t)$. Using $2\sin a\sin b = \cos(a-b) - \cos(a+b)$, write the result. Which term survives a low-pass filter at, say, 10 Hz?

6. **Quadrature is sign-aware.** Why do you need *both* real and imaginary channels? Hint: what would happen if $\Delta\omega(\mathbf{r}) < 0$? Could a single-channel demodulator distinguish that from $\Delta\omega(\mathbf{r}) > 0$?

7. **Reciprocity check.** A loop coil of radius 4 cm produces 50 µT/A at a sample point 3 cm away (measured as the transmit field with 1 A current). What is $|\mathbf{c}(\mathbf{r})|$ at that point?

8. **High-field caveat.** Why does the dielectric wavelength matter for reciprocity? At what wavelength would a 20 cm coil start to deviate significantly from the static-B approximation?

9. **The signal is a Fourier transform.** Take the final signal equation and set $c_+^*(\mathbf{r}) = 1$ (a hypothetical uniform sensitivity) and $\Delta\omega(\mathbf{r}, t) = \gamma\mathbf{G}\cdot\mathbf{r}$ (a constant gradient). Show that $\text{signal}(t) = \omega_0\,\tilde M_\perp(\mathbf{k}(t))$ where $\mathbf{k}(t) = \gamma\mathbf{G}t/(2\pi)$ and $\tilde M$ is the Fourier transform of $M_\perp$. **This is the prelude to k-space.**

10. **A signal-to-noise sanity check.** The signal equation has a factor $\omega_0$ out front. Combined with $M_\perp \propto B_0$ (and so $M_+' \propto B_0$), what's the total $B_0$ dependence of the signal? Compare to L05 §1's $B_0^2$ scaling.

---

## 🔗 Connections to other lectures

- **L02 (Bloch equations in B₀)** gave us $M_+(t) = M_+(0)e^{i\omega_0 t}$ for the simplest case. §7.9 here generalizes to time-varying ω, which we'll need for gradients.
- **L03–L04 (excitation)** established the rotating frame and showed why working there is so much cleaner. §7.13's quadrature demodulation is *literally* the experimental implementation of going to the RRF — the demodulator at $\omega_0$ subtracts $\omega_0$ from the magnetization's phase, exactly like the math in L04 §6.6.1 did with $\mathbf{B}_\text{eff}$.
- **L05 (Faraday and c(r))** built the receive picture; this lecture finishes it.
- **Next lecture (gradients) and Module 2 (k-space):** plug $\Delta\omega(\mathbf{r}) = \gamma\mathbf{G}\cdot\mathbf{r}$ into the signal equation and you get the Fourier transform of $M_\perp$. The "k-space trajectory" is the time-integral of the gradient.
- **Module 4 (parallel imaging, SENSE/GRAPPA):** $c_+^*(\mathbf{r})$ in the signal equation becomes a per-channel quantity $c_{+,i}^*(\mathbf{r})$ for $i = 1, \dots, N_\text{channels}$. SENSE inverts the system $\{\text{signal}_i\} = \mathbf{E}\,M_+$ where $\mathbf{E}$ is the multi-coil encoding operator. GRAPPA exploits the same multi-coil structure in k-space.
- **Module 5 (deep learning recon):** even neural-network reconstruction pipelines embed this signal model in their forward operator. The data-consistency layer in MoDL/VarNet is exactly $\mathbf{E}^*\mathbf{E}M_+ - \mathbf{E}^*\,\text{signal}$ → 0.

---

*Next up: gradients. We finally make $\Delta\omega$ spatially varying — and that's how magnetization at different positions becomes distinguishable in the signal.*
