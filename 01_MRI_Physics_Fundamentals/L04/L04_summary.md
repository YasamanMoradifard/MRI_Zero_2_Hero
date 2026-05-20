# L04 — The Rotating Frame, the Effective Field, and the Resonance Condition

> **Scope note.** This PDF picks up where L03 left off (chapter 6, *Excitation of M*) and covers **sections 6.5 through 6.9**: the general differential equation of rotations, the magnetic-resonance principle derived from the Bloch equations, the effective field $\mathbf{B}_\text{eff}$, off-resonance analysis with worked examples, rotation matrices, the explicit closed-form formula for $\mathbf{M}(t)$, and a few practical words on tuning. This is the lecture where **the resonance condition stops being an observation and becomes a theorem**.
>
> Two small artifacts to flag: section 6.7.3 "General Rotations" appears twice in the PDF (once truncated as "Missing", then properly on a later page) — the proper version is what matters. Section 6.9 also mentions a "Some words on ΔB(r): missing" note. We work with the substantive content as written.

**Source:** MRI1 lecture notes, ch. 6.5 — 6.9.

---

## 🎯 What you'll be able to do after this lecture

- Write down the differential equation of *any* right-handed rotation: $\dot{\mathbf{v}} = \boldsymbol{\Omega} \times \mathbf{v}$, and use it to derive the RRF transformation rule.
- Transform the Bloch equation into a rotating reference frame and identify the **effective field**.
- Explain in one sentence why *only* an RF field tuned to the Larmor frequency produces a large flip angle, using the geometry of $\mathbf{B}_\text{eff}$.
- Compute the maximally achievable flip angle for any off-resonance: $\alpha_\text{max} = 2\arccos(\Delta\omega_\text{HF}/\omega_\text{eff})$.
- State and use the three xyz rotation matrices, prove they're orthogonal ($R^{-1} = R^T$), and compose them to rotate around arbitrary axes.
- Apply the closed-form $\mathbf{M}(t)$ formula for the off-resonant case.
- Explain why MRI scanners need a "tuning" step before each scan and where off-resonances come from in vivo.

---

## 1. The differential equation of rotations (§6.5)

A simple but powerful identity. If a vector $\mathbf{v}(t)$ is rotated right-handedly about a unit axis $\mathbf{e}_\Omega(t)$ at angular frequency $\Omega(t)$:

$$
\boxed{\dot{\mathbf{v}}(t) = \boldsymbol{\Omega}(t) \times \mathbf{v}(t)}, \qquad \boldsymbol{\Omega}(t) := \Omega(t)\,\mathbf{e}_\Omega(t)
$$

This is the *converse* of the Bloch equation in a sense: any time you see "$\dot{} = (\text{something}) \times ()$", you're looking at a rotation. The Bloch equation $\dot{\mathbf{M}} = \gamma\mathbf{B} \times \mathbf{M}$ is just this identity with $\boldsymbol{\Omega} = \gamma\mathbf{B}$ — the field *is* the rotation rate (up to γ).

> **Why this matters.** It gives us a universal tool: the *unit vectors of the RRF themselves* obey this equation, since the RRF *is* a rotation. We'll use that in 30 seconds.

---

## 2. The effective field $\mathbf{B}_\text{eff}$ (§6.6.1)

This is the centerpiece. We want to write the Bloch equation in the rotating frame instead of the lab frame.

### 2.1 Two ways to write the same vector

A vector $\mathbf{M}(t)$ is *one object*, the same arrow in space, but it has different *components* depending on whether you describe it in the lab basis $\{\mathbf{e}_x, \mathbf{e}_y, \mathbf{e}_z\}$ or in the rotating basis $\{\mathbf{e}'_x(t), \mathbf{e}'_y(t), \mathbf{e}'_z(t)\}$:

$$
\mathbf{M}(t) = \sum_n M_n(t)\,\mathbf{e}_n = \sum_n M'_n(t)\,\mathbf{e}'_n(t)
$$

In the lab frame, the basis vectors don't move, so taking $\dot{\mathbf{M}}$ is just differentiating the components. In the RRF, the basis vectors *themselves* are time-dependent and we need the product rule.

### 2.2 The derivation

Differentiating the RRF representation:

$$
\dot{\mathbf{M}}(t) = \underbrace{\sum_n \dot M'_n(t)\,\mathbf{e}'_n(t)}_{\text{change of components}} + \underbrace{\sum_n M'_n(t)\,\dot{\mathbf{e}}'_n(t)}_{\text{change of basis}}
$$

The first term is the "what the rotating observer sees." We give it the notation $(\dot{\mathbf{M}})'$ to emphasize it's the *primed-component* derivative:

$$
(\dot{\mathbf{M}})' := \sum_n \dot{M}'_n(t)\,\mathbf{e}'_n(t)
$$

The second term is where the magic happens. The RRF basis vectors are themselves rotating at $\boldsymbol{\Omega}$, so by the identity in §1:

$$
\dot{\mathbf{e}}'_n(t) = \boldsymbol{\Omega} \times \mathbf{e}'_n(t)
$$

Linearity of the cross product then gives $\sum_n M'_n \dot{\mathbf{e}}'_n = \boldsymbol{\Omega} \times \mathbf{M}$. Substituting:

$$
\dot{\mathbf{M}}(t) = (\dot{\mathbf{M}})' + \boldsymbol{\Omega} \times \mathbf{M}(t)
$$

Now plug in the Bloch equation $\dot{\mathbf{M}} = \gamma\mathbf{B} \times \mathbf{M}$ on the left:

$$
(\dot{\mathbf{M}})' = \gamma\mathbf{B}\times\mathbf{M} - \boldsymbol{\Omega}\times\mathbf{M} = (\gamma\mathbf{B} - \boldsymbol{\Omega}) \times \mathbf{M}
$$

Define the **effective field**:

$$
\boxed{\mathbf{B}_\text{eff}(t) := \mathbf{B}(t) - \frac{\boldsymbol{\Omega}(t)}{\gamma}}
$$

Then:

$$
\boxed{(\dot{\mathbf{M}})' = \gamma\,\mathbf{B}_\text{eff}(t) \times \mathbf{M}(t)}
$$

This looks like the Bloch equation again — but **in the rotating frame, with $\mathbf{B}$ replaced by $\mathbf{B}_\text{eff}$**. The rotating observer sees a "fake" magnetic field that's the real field *minus the spinning of their own frame* (in field units).

> **Conceptual punchline.** The frame's rotation appears as an extra fictitious "field" $-\boldsymbol{\Omega}/\gamma$ added to the real one. This is the magnetic analogue of the centrifugal force you feel in a spinning car. If you spin your frame at the right rate, you can *cancel out* the real B₀ field — which is exactly the resonance trick.

---

## 3. Choosing the RRF (§6.6.2)

In the on-resonance case ($\omega_\text{HF} = \omega_0$) the choice $\Omega = \omega_0 = \omega_\text{HF}$ around $\mathbf{e}_z$ killed two birds: it froze the $\mathbf{B}_1^+$ field *and* cancelled $\mathbf{B}_0$ in $\mathbf{B}_\text{eff}$.

Off resonance ($\omega_\text{HF} \neq \omega_0$) you can only freeze *one* of the two. We choose to freeze the RF field:

$$
\boldsymbol{\Omega} = \omega_\text{HF}\,\mathbf{e}_z
$$

The $\mathbf{B}_1^+$ field then sits stationary along (say) the $\mathbf{e}'_x$ axis. The lab $\mathbf{B}_0$ is still $B_0 \mathbf{e}_z = B_0\mathbf{e}'_z$ (the *physical* field doesn't change just because you started spinning) — but its **effect** in the rotating frame is reduced.

---

## 4. The total field in the RRF (§6.6.3)

Put both pieces together:

$$
\mathbf{B}(t) = \mathbf{B}_0 + \mathbf{B}_1^+(t) = \begin{pmatrix}B_1^+\\0\\B_0\end{pmatrix}'
$$

and the effective field:

$$
\mathbf{B}_\text{eff} = \mathbf{B} - \frac{\omega_\text{HF}}{\gamma}\mathbf{e}_z = \begin{pmatrix}B_1^+\\0\\B_0 - \omega_\text{HF}/\gamma\end{pmatrix}'
$$

Using $\omega_0 = \gamma B_0$ and $\omega_1 = \gamma B_1^+$ we can flip to angular-frequency units. The Bloch equation in the RRF becomes the *clean* form:

$$
\boxed{\begin{pmatrix}\dot M'_x\\ \dot M'_y\\ \dot M'_z\end{pmatrix}' = \begin{pmatrix}\omega_1\\0\\\omega_0 - \omega_\text{HF}\end{pmatrix}' \times \begin{pmatrix}M'_x\\ M'_y\\ M'_z\end{pmatrix}'}
$$

This is **a static-field Bloch equation in the RRF**. All the time-dependence is gone (for constant $B_1^+$ and constant $\omega_\text{HF}$).

### 4.1 The resolved "contradiction"

Earlier in the course (on-resonance treatment) you "saw" the $\mathbf{B}_0$ field vanish in the RRF. But $\mathbf{B}_0$ is a *real* physical field — it doesn't vanish. The resolution: in the RRF representation $\mathbf{B}_0$ still has components $(0,0,B_0)^T$, but the *effective* field that drives the magnetization in the RRF has the term $\omega_\text{HF}/\gamma$ subtracted. On resonance that subtraction is exactly $B_0$, so $\mathbf{B}_\text{eff}$ has zero z-component. The physical field is still there; its *effect on the dynamics in the rotating frame* is what gets cancelled.

---

## 5. The off-resonance case in pictures (§6.6.4)

In the RRF, $\boldsymbol{\omega}_\text{eff} = \gamma\mathbf{B}_\text{eff}$ has components:

- x'-component: $\omega_1 = \gamma B_1^+$
- z'-component: $\Delta\omega_\text{HF} := \omega_0 - \omega_\text{HF}$

Its magnitude:

$$
\omega_\text{eff} = \sqrt{\Delta\omega_\text{HF}^2 + \omega_1^2}
$$

and its tilt from the z'-axis is the angle $\Theta$:

$$
\cos\Theta = \frac{\Delta\omega_\text{HF}}{\omega_\text{eff}}, \qquad \sin\Theta = \frac{\omega_1}{\omega_\text{eff}}
$$

### 5.1 The maximum achievable flip angle

The magnetization starts at $\mathbf{M}(0) = M_0\mathbf{e}_z$, sits at angle $\Theta$ from $\boldsymbol{\omega}_\text{eff}$, and rotates around $\boldsymbol{\omega}_\text{eff}$ at frequency $\omega_\text{eff}$. **It traces out a cone.** The farthest it gets from the z'-axis is at the diametrically opposite point on the cone — twice the angle between $\mathbf{M}(0)$ and the cone axis:

$$
\boxed{\alpha_\text{max} = 2\Theta = 2\arccos\!\left(\frac{\Delta\omega_\text{HF}}{\omega_\text{eff}}\right)}
$$

This is **the resonance condition, derived**. On resonance ($\Delta\omega_\text{HF}=0$): $\Theta=90°$, $\alpha_\text{max} = 180°$ — full inversion possible. Even *small* off-resonance drops the cap dramatically because $\Delta\omega_\text{HF}/\omega_\text{eff}$ is governed by the ratio of $\Delta\omega_\text{HF}$ to a tiny $\omega_1$.

### 5.2 Worked examples from the lecture

| Case | $\omega_\text{HF}$ | $\Delta\omega_\text{HF}$ | $\boldsymbol{\omega}_\text{eff}$ | $\alpha_\text{max}$ |
|------|--------------------|---------------------------|----------------------------------|---------------------|
| On-resonance | $\omega_0$ | 0 | $(\omega_1, 0, 0)'$ | **180°** |
| Tiny off-res ($\Delta = \omega_1$) | $\omega_0 - \omega_1$ | $\omega_1$ | $(\omega_1, 0, \omega_1)'$ | **90°** |
| Wrong-direction RF | $-\omega_0$ | $2\omega_0$ | $(\omega_1, 0, 2\omega_0)'$ | **≈ 0°** |
| Negative off-resonance | $> \omega_0$ | $< 0$ | tilted into $-z'$ half | $2\arccos(\|\Delta\|/\omega_\text{eff})$ |

The "wrong-direction" case is illuminating. If the RF cycles left-handed at the same frequency, $\Delta\omega_\text{HF} = 2\omega_0$ — the effective field is essentially the full $\mathbf{B}_0$ again, $\boldsymbol{\omega}_\text{eff} \approx 2\omega_0 \hat z'$, and $\alpha_\text{max} \approx 0$. Spin direction matters: a "co-rotating" component of a linearly polarized RF excites; the counter-rotating component does basically nothing. This is the key insight behind why circularly polarized excitation coils are more efficient than linearly polarized ones.

### 5.3 The sharpness of resonance

The lecture plots $\alpha_\text{max}$ vs $\Delta\omega_\text{HF}/\omega_0$ for $B_1^+ = 5\,\mu\text{T}$. The resonance peak's half-width at half-max is roughly $\Delta\omega_\text{HF}/\omega_0 \approx 10^{-5}$. That's **parts per million** — far tighter than the $\sim 10^{-6}$ stability of clinical superconducting magnets. Hence "tuning" (§9) is essential.

---

## 6. Rotation matrices (§6.7)

To get a *closed form* for $\mathbf{M}(t)$ we need rotation matrices. The three xyz versions:

$$
R_x(\alpha) = \begin{pmatrix}1 & 0 & 0\\ 0 & \cos\alpha & -\sin\alpha\\ 0 & \sin\alpha & \cos\alpha\end{pmatrix}, \quad
R_y(\alpha) = \begin{pmatrix}\cos\alpha & 0 & \sin\alpha\\ 0 & 1 & 0\\ -\sin\alpha & 0 & \cos\alpha\end{pmatrix}, \quad
R_z(\alpha) = \begin{pmatrix}\cos\alpha & -\sin\alpha & 0\\ \sin\alpha & \cos\alpha & 0\\ 0 & 0 & 1\end{pmatrix}
$$

### 6.1 Memory pegs

- A **1** on the diagonal marks the rotation axis; that row and column are otherwise zero.
- The other two diagonal entries are cosines.
- The remaining two off-diagonal entries are sines.
- One sine takes a minus sign — the harder bit. If you forget which, draw what $R_z(\alpha)$ does to $\mathbf{e}_x$: it should land at $(\cos\alpha, \sin\alpha, 0)$, which forces the layout.

### 6.2 Orthogonality

Rotations preserve length: $|R\mathbf{v}| = |\mathbf{v}|$. From $|\mathbf{v}|^2 = \mathbf{v}^T\mathbf{v}$ we get $|R\mathbf{v}|^2 = \mathbf{v}^T R^T R \mathbf{v}$, so $R^T R = I$ — i.e. $R^{-1} = R^T$. Equivalently, "rotating back" means using the negative angle: $R_z^{-1}(\alpha) = R_z(-\alpha)$. (The lecture's "unitary" terminology is loose — the matrices here are real *orthogonal*; "unitary" is the complex generalisation. The property used is the same either way.)

### 6.3 Rotation around a general axis: the conjugation trick

To rotate by $\alpha$ around an arbitrary unit axis $\mathbf{e}_\alpha$ — without learning Rodrigues' formula — use a 3-step conjugation:

1. **Rotate** $\mathbf{e}_\alpha$ onto one of the cardinal axes using $R_x$, $R_y$, $R_z$ as needed. Apply the same rotation to $\mathbf{M}$.
2. **Do** the rotation by $\alpha$ around that cardinal axis (now trivial).
3. **Undo** step 1.

For example, if $\mathbf{e}_\alpha$ lies in the xz-plane at angle $\Theta$ from $\mathbf{e}_z$:

$$
\mathbf{M}_\text{rotated} = R_y(\Theta)\,R_z(\alpha)\,R_y(-\Theta)\,\mathbf{M}
$$

(Read right-to-left, like function composition. Step 1 is rightmost.)

---

## 7. Closed-form $\mathbf{M}(t)$ for off-resonance excitation (§6.8)

With $\mathbf{B}_1^+ \parallel \mathbf{e}'_x$, $\mathbf{M}(0) = M(0)\mathbf{e}'_z$, and $\boldsymbol{\omega}_\text{eff}$ tilted by $\Theta$ from $\mathbf{e}'_z$ in the x'z'-plane, the conjugation recipe gives:

$$
\mathbf{M}(t) = R'_y(\Theta)\,R'_z(\omega_\text{eff}\,t)\,R'_y(-\Theta)\,\mathbf{M}(0)
$$

Multiplying out (you do this in the exercises):

$$
\boxed{\mathbf{M}(t) = M(0)\begin{pmatrix}\cos\Theta\,\sin\Theta\,(1-\cos\omega_\text{eff}t)\\ -\sin\Theta\,\sin\omega_\text{eff}t\\ \cos^2\Theta + \sin^2\Theta\,\cos\omega_\text{eff}t\end{pmatrix}'}
$$

Sanity checks worth doing in your head:

- **On resonance** ($\Theta = 90°$): $M'_x = 0$, $M'_y = -\sin\omega_\text{eff}t$, $M'_z = \cos\omega_\text{eff}t$. M precesses in the y'z'-plane around $\mathbf{e}'_x$. Full inversion when $\omega_\text{eff}t = \pi$. ✓
- **Far off resonance** ($\Theta \to 0$): $M'_x \to 0$, $M'_y \to 0$, $M'_z \to 1$. The magnetization barely budges. ✓
- **Length conserved**: $\|\mathbf{M}(t)\|^2 = M(0)^2$ identically. (Algebraic exercise — try it.)

To go back to the **lab frame**, undo the RRF rotation: multiply by $R_z(-\omega_\text{HF}t)$. The lab-frame trajectory is the RRF motion *plus* a fast oscillation at $\omega_\text{HF}$.

---

## 8. Off-resonances in practice — tuning (§6.9)

### 8.1 Why tuning is needed

Two reasons:
1. **Magnet drift.** Even superconducting magnets drift a few Hz/day. With Larmor at ~128 MHz at 3 T, a few Hz is sub-ppm but easily detectable.
2. **Body-induced inhomogeneity.** Different tissues have different magnetic susceptibilities. Air–tissue interfaces (sinuses, ear canals, lungs) cause field perturbations $\Delta\mathbf{B}(\mathbf{r})$ that can reach several ppm — much larger than scanner drift.

### 8.2 How tuning works

1. Send a **broadband** RF pulse centered around the estimated Larmor frequency.
2. Whichever frequency component in the pulse hits resonance excites magnetization.
3. That magnetization radiates back $B_1^-$ at exactly the excitation frequency.
4. Measure that frequency. Set future $B_1^+$ pulses to match.

That measured frequency *is* $\omega_0$ for the current scan. The lecture flags that $\Delta\mathbf{B}(\mathbf{r})$ — the spatial variation in B₀ across the imaged volume — is a separate topic for later. Spoiler: it leads to "shimming" (active field correction with gradient and shim coils) and to artifacts like geometric distortion in EPI.

---

## 🧠 Self-test questions

1. **The rotation identity.** A vector tip is moving in a circle in the xy-plane at angular speed 5 rad/s, right-handed about $+\mathbf{e}_z$. Write $\boldsymbol{\Omega}$, then write $\dot{\mathbf{v}}$ for $\mathbf{v} = (1, 0, 0)^T$ at this instant. Verify it points in the $+\mathbf{e}_y$ direction.

2. **Two viewpoints, same vector.** Explain in one sentence why $\mathbf{M}$ has the same arrow but different *components* in the lab frame vs the RRF, and why $\dot{\mathbf{M}}$ requires the product rule in the RRF but not in the lab.

3. **The effective field formula.** Without looking, write down $\mathbf{B}_\text{eff}$ in terms of $\mathbf{B}$ and $\boldsymbol{\Omega}$. Then, for $\boldsymbol{\Omega} = \omega_\text{HF}\mathbf{e}_z$, $\mathbf{B} = \mathbf{B}_0 + B_1^+\mathbf{e}'_x$, write out its three RRF components.

4. **Maximum flip angle.** Compute $\alpha_\text{max}$ for $B_1^+ = 5\,\mu\text{T}$ and $\Delta\omega_\text{HF}/(2\pi) = 100$ Hz at 3 T. (Use $\gamma/2\pi = 42.58$ MHz/T.) Compare to the on-resonance value of 180°.

5. **The two extreme cases.** Why does an RF field rotating *with* the proton's precession excite, while one rotating *against* it does essentially nothing — even at the same magnitude?

6. **Resonance sharpness.** Why does the resonance peak get *sharper* as $B_1^+$ decreases? (Look at $\omega_\text{eff} = \sqrt{\Delta\omega_\text{HF}^2 + \omega_1^2}$.)

7. **Rotation matrices.** Verify $R_z(\pi/2)\,\mathbf{e}_x = \mathbf{e}_y$. Then verify $R_z^T(\pi/2)\,R_z(\pi/2) = I$.

8. **Conjugation rotation.** Use the conjugation recipe to write the rotation by $\alpha$ around the axis $\mathbf{e}_\alpha = (1, 0, 1)^T/\sqrt{2}$.

9. **Length conservation.** Show algebraically that the explicit $\mathbf{M}(t)$ formula has constant magnitude, i.e., $M'_x{}^2 + M'_y{}^2 + M'_z{}^2 = M(0)^2$ for all $t$.

10. **Why tune?** Estimate the off-resonance (in Hz) at which $\alpha_\text{max}$ drops from 180° to 90°, given $B_1^+ = 5\,\mu\text{T}$. Compare to the typical few-Hz daily drift of a superconducting magnet — do you need to tune for daily drift?

---

## 🔗 Connections to other lectures

- **L02 / chapter 5** gave you the static-field Bloch equation. Section 6.6.3 here shows that on-resonance, the RRF *reproduces* that same equation — the $\mathbf{B}_0$ field has been transformed away. So everything you proved about precession in a static field carries over to the rotating frame.
- **The closed-form $\mathbf{M}(t)$** is the most concrete answer you'll ever get to "what does an RF pulse do?". You'll use it throughout the rest of the course whenever you analyze a sequence.
- **Slice selection (L11)** is *intentionally* designed off-resonance — you apply a gradient so that different z-positions have different $\Delta\omega_\text{HF}$, and only the slice where $\Delta\omega_\text{HF} = 0$ matches the bandwidth-limited RF pulse. The α_max curve from §6.6.4 is the *physical basis of slice selectivity*.
- **Shimming and B₀ field maps** (mentioned in §6.9) will return in the discussion of artifacts and EPI.
- **Compressed sensing (Module 4) and ML reconstruction (Module 5)** don't directly touch RRF math — but understanding *why* B₀ inhomogeneity matters is the foundation for the off-resonance correction methods that come up in non-Cartesian recon.

---

## 📚 Suggested reading

- **Brown, Cheng, Haacke, Thompson, Venkatesan**, *Magnetic Resonance Imaging: Physical Principles and Sequence Design* (Wiley, 2014) — chapter 3 has a parallel treatment of the rotating frame.
- **Levitt, M.H.**, *Spin Dynamics: Basics of Nuclear Magnetic Resonance* (Wiley, 2008) — the gold standard, with much more on RRF formalism.
- **Hoult, D.I.** (2000). "The Solution of the Bloch Equations in the Presence of a Varying B₁ Field — An Approach to Selective Pulse Analysis." *Concepts in Magnetic Resonance.* The classical reference for designing real RF pulses with this formalism.

---

*Next up: signal detection — how a precessing $\mathbf{M}$ induces an EMF in the receive coil, and what the FID actually looks like.*
