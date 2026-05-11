# L02 — Definitions, the Three Principles (precise form), and Dynamics in B₀

> **Heads-up on scope.** This PDF is labeled "Lecture 2" but actually contains chapters **3, 4, and the start of 5** of the MRI1 lecture notes. It covers definitions and notation, the second (precise) formulation of the three principles, and the start of the Bloch-equation analysis in a static B₀ field. In our roadmap this best aligns with **Days 3, 4, and 6** (Definitions → Three Principles → Dynamics of M in B₀), so consider this a "triple feature" — it's a lot for one sitting. Feel free to spread it across two or three sessions.

**Source:** MRI1 lecture notes, ch. 3 (Definitions), ch. 4 (The Three Principles — 2nd formulation), ch. 5 (start: Cross product, M₀ in B₀, M∥ in B₀, M⊥ in B₀).

---

## 🎯 What you'll be able to do after this lecture

- Set up the scanner coordinate system and decompose any vector into longitudinal (∥B₀) and transversal (⊥B₀) parts.
- Switch fluently between **vector notation** (vₓ, v_y, v_z) and **complex-plane notation** (v₊ = vₓ + i·v_y) for transversal vectors.
- State the three principles of MRI in their precise form, including the Bloch equations.
- Derive the equilibrium magnetization M₀ ∝ ρB₀/T_K and explain why MRI uses such strong magnets.
- Show that, in a static B₀ field with no relaxation, the longitudinal magnetization M∥ is preserved and the transversal magnetization M⊥ precesses at the Larmor frequency ω₀ = γB₀.

---

## 1. The scanner coordinate system

A right-handed Cartesian system tied to the scanner:

- **z-axis** — along the scanner bore, parallel to **B₀** = (0, 0, B₀)ᵀ. Always points head-to-feet (or feet-to-head) of the patient.
- **y-axis** — vertical, floor-to-ceiling.
- **x-axis** — horizontal, left-to-right of a patient lying supine.

Unit vectors **eₓ**, **e_y**, **e_z**. Right-handed means: thumb (x) × pointer (y) = middle finger (z). All MRI conventions in this course are right-handed.

> **Why this matters.** Once we say B₀ = B₀·**e_z**, the z-direction becomes physically special — it's the axis spins precess around. Splitting any vector into "along z" and "perpendicular to z" is the most useful decomposition in all of MRI.

---

## 2. Vector notation

### 2.1 General vectors

Vectors are bold (**v**), magnitudes are not (v = |**v**|). Components are vₓ, v_y, v_z. A subscripted bold like **vₓ** means *only the x-component as a vector*: **vₓ** = (vₓ, 0, 0)ᵀ.

### 2.2 Longitudinal and transversal parts

Any vector splits cleanly:

$$
\mathbf{v} = \mathbf{v}_\parallel + \mathbf{v}_\perp, \quad \mathbf{v}_\parallel = (0, 0, v_z)^T, \quad \mathbf{v}_\perp = (v_x, v_y, 0)^T
$$

This decomposition is *the* organizing principle of the Bloch equations: longitudinal and transversal magnetization relax differently (T₁ vs T₂) and behave differently under RF excitation.

### 2.3 Unit vectors

For any non-zero **v**, the unit vector is **e**_v ≔ **v**/v, with |**e**_v| = 1.

---

## 3. Complex-plane notation for transversal vectors

Because transversal vectors only have two components (x, y), we can pack them into a single complex number:

$$
v_+ \coloneqq v_x + i\,v_y \quad \text{(subscript "+" means complex notation)}
$$

In Gauss form:

$$
v_+ = |v_+| \cdot e^{i\theta_v} = v_\perp \cdot e^{i\theta_v}
$$

where θ_v is the angle from the x-axis. The complex conjugate gets its own name: v₋ ≔ vₓ − i·v_y = v₊*.

> **Why bother?** Precession in B₀ rotates the transversal vector at angular frequency ω₀. In complex notation, that rotation is simply multiplication by e^(iω₀t) — one number, one operation. Compared to writing out two coupled sine/cosine equations every time, this is a huge notational win.

### 3.1 Watch out

|v₊| is real (the length); v₊ itself is complex. So |v₊| ≠ v₊ in general — only when θ_v = 0.

---

## 4. Time derivatives — dot notation

Shorthand:

$$
\dot{\mathbf{v}}(t) \coloneqq \frac{d\mathbf{v}(t)}{dt}, \quad \ddot{\mathbf{v}}(t) \coloneqq \frac{d^2\mathbf{v}(t)}{dt^2}
$$

The dot is exclusively temporal — never a spatial derivative. The product rule still applies: d/dt[v(t)·**e**_v(t)] = v̇(t)**e**_v(t) + v(t)·**ė**_v(t).

---

## 5. The Three Principles — precise formulation

The "gentle" intro stated three principles in plain language. Now we make them rigorous.

### Principle 1 (precise) — Nuclear magnetization

> *"If nuclei with non-zero spin reside in a magnetic field B₀, a nuclear magnetization **M** ∥ **B₀** arises."*

**What changed from the gentle version:** Replaced "protons" with "nuclei with non-zero spin." Other useful nuclei include ²³Na, ³⁹K, ¹⁹F, ³He, ¹²⁹Xe. In this course we stick with protons because they have the highest natural abundance and the largest magnetic moment per nucleus → strongest signal. Sodium MRI is interesting clinically because Na⁺ tracks cell vitality and membrane potential.

**Implicit point:** **M** is a *position-dependent* field, **M**(**r**) — different tissues have different magnetization. That's literally what gives you image contrast.

#### 5.1 Force on a magnetized sample (and why ferromagnetic objects don't fly inside the bore)

For a small sample of volume V with homogeneous magnetization **M** in a field **B**(**r**):

$$
\mathbf{F}(\mathbf{r}) = (V \cdot \mathbf{M} \cdot \nabla) \cdot \mathbf{B}(\mathbf{r}) = -\nabla U(\mathbf{r}), \quad U(\mathbf{r}) = -\mathbf{p}_V \cdot \mathbf{B}(\mathbf{r})
$$

with **p**_V = V·**M** the total magnetic moment of the sample.

> **Key insight:** The force comes from the **gradient** of B, not from B itself. Inside a well-shimmed scanner B₀ is essentially uniform, so ∇B₀ ≈ 0 and **F** ≈ 0. *Outside* the bore, where B drops from B₀ down to ambient, the gradient is huge — that's where the "MRI ate my oxygen tank" accidents happen.

A torque can still exist in a uniform field if the dipole isn't aligned with **B** — that's what drives precession.

#### 5.2 From microscopic μ to macroscopic M

Each proton carries a magnetic moment **μ**. The macroscopic magnetization in a small volume V containing N protons is:

$$
\mathbf{M} = \frac{1}{V}\sum_{\text{protons}} \boldsymbol{\mu} = \frac{N}{V}\langle\boldsymbol{\mu}\rangle = \rho \langle\boldsymbol{\mu}\rangle
$$

where ρ = N/V is the proton density and ⟨**μ**⟩ is the expectation value of a single proton's moment.

#### 5.3 The equilibrium magnetization M₀ (bonus derivation)

The transversal components average to zero (no preferred x or y direction). Only ⟨μ_z⟩ survives. Using the Boltzmann distribution and quantum mechanics' rule that μ_z takes only two values (μ↑ = +γℏ/2, μ↓ = −γℏ/2):

$$
M_0 \approx \frac{\rho}{4} \cdot \frac{\gamma^2 \hbar^2}{k_B T_K} B_0
$$

The numerical argument γℏB₀/(2k_BT_K) ≈ 5×10⁻⁶ at 1.5 T and ≈ 10⁻⁵ at 3 T — the **high-temperature limit** is excellent at body temperature. The exact formula uses tanh(γℏB₀/(2k_BT_K)).

**The takeaway you actually need to remember:**

$$
\boxed{M_0 \propto \frac{\rho B_0}{T_K}}
$$

ρ is fixed by the tissue. T_K is fixed (you can't cool the patient). γ, ℏ, k_B are constants of nature. **The only knob you can turn is B₀.** That's why MRI scanners keep getting bigger and stronger.

### Principle 2 (precise) — Magnetization generates a magnetic dipole field

> *"A sample with **M** ≠ 0 generates a magnetic dipole field, as described in classical electrodynamics."*

The explicit form (you don't need to memorize it, but should recognize it):

$$
\mathbf{B}_{\text{dipole}}(\mathbf{r}) = \frac{\mu_0 V}{4\pi}\left[\frac{3(\mathbf{r}\cdot\mathbf{M})\mathbf{r}}{r^5} - \frac{\mathbf{M}}{r^3}\right]
$$

Two things to extract from this formula:

1. **Falls off as 1/r³** — much faster than an electric monopole's 1/r². Practical consequence: receive coils must be placed *as close as possible* to the body part being imaged. A head coil sits right on the skull.
2. **Linear in M** — doubling **M** doubles the received field B₁⁻. Stronger M₀ → stronger signal. (And M₀ scales with B₀, so you get a *quadratic* signal boost from B₀: once for more polarization, once for stronger induction.)

The most useful general form: **classical electrodynamics applies to MRI.** Faraday induction, Lorentz forces, dipole fields — all of it carries over.

### Principle 3 (precise) — The Bloch equations

This is where the precise version looks **radically different** from the gentle one. The gentle version listed three separate observations (resonance, T₂ decay, T₁ recovery). The precise version compresses all three into a single vector ODE:

$$
\boxed{\dot{\mathbf{M}}(t) = \gamma \mathbf{B}(t) \times \mathbf{M}(t) + \frac{1}{T_1}\left(\mathbf{M}_0 - \mathbf{M}_\parallel(t)\right) - \frac{1}{T_2}\mathbf{M}_\perp(t)}
$$

In components:

$$
\frac{d}{dt}\begin{pmatrix}M_x\\M_y\\M_z\end{pmatrix} = \gamma\begin{pmatrix}B_x\\B_y\\B_z\end{pmatrix}\times\begin{pmatrix}M_x\\M_y\\M_z\end{pmatrix} + \frac{1}{T_1}\begin{pmatrix}0\\0\\M_0-M_z\end{pmatrix} - \frac{1}{T_2}\begin{pmatrix}M_x\\M_y\\0\end{pmatrix}
$$

**Reading the three terms:**

| Term | What it does | Maps to old principle |
|------|-------------|----------------------|
| γ**B**×**M** | Drives precession & RF excitation | 3a (resonance) |
| (1/T₁)(**M₀** − **M**∥) | Restores longitudinal mag toward M₀ | 3c (T₁ recovery) |
| −(1/T₂)**M**⊥ | Damps transversal mag toward 0 | 3b (T₂ decay) |

**Constants:**
- **γ** — gyromagnetic ratio. Nucleus-specific. For protons: γ ≈ 2.675×10⁸ rad·s⁻¹·T⁻¹, or γ/(2π) ≈ 42.58 MHz/T.
- **T₁, T₂** — tissue-specific relaxation times. T₁ ≳ T₂ always. Typical brain values at 3 T: white matter T₁ ≈ 800 ms, T₂ ≈ 70 ms; CSF T₁ ≈ 4000 ms, T₂ ≈ 2000 ms.

**Status of the equations:** The precession term γ**B**×**M** can be derived from quantum mechanics (the factor γ is fundamentally quantum). The relaxation terms are **observational** — phenomenological exponentials that fit human tissue extremely well but aren't derived from first principles. Bloch wrote them down in 1946 and they've worked ever since.

#### 5.4 The missing minus sign

The strictly correct Bloch equation is:

$$
\dot{\mathbf{M}}(t) = -\gamma \mathbf{B}\times\mathbf{M} + \text{(relaxation)} \;\;=\;\; \gamma \mathbf{M}\times\mathbf{B} + \text{(relaxation)}
$$

The course **drops the minus sign** by convention to keep the rotating frame right-handed. Different textbooks/papers use different conventions — sometimes inconsistently within the same document. When comparing formulas across sources, always check which sign convention is in play.

---

## 6. The cross product (refresher)

Definition:

$$
\mathbf{a}\times\mathbf{b} = \begin{pmatrix}a_y b_z - a_z b_y \\ a_z b_x - a_x b_z \\ a_x b_y - a_y b_x\end{pmatrix}
$$

Magnitude: |**a**×**b**| = a·b·sin θ. Direction: perpendicular to both, right-handed.

**Properties to keep in your back pocket:**

- **a** ∥ **b** ⟹ **a**×**b** = **0**. (In particular, **a**×**a** = **0**.)
- **a**⊥**b** ⟹ |**a**×**b**| = a·b.
- Anti-commutative: **a**×**b** = −**b**×**a**.
- Linear in each argument: (**a**+**d**)×**b** = **a**×**b** + **d**×**b**.

These four facts are enough to power-walk through the Bloch equation analysis.

---

## 7. Dynamics of M in B₀ (no relaxation)

Set **B**(t) = **B**₀ = (0, 0, B₀)ᵀ and ignore relaxation:

$$
\dot{\mathbf{M}}(t) = \gamma \mathbf{B}_0 \times \mathbf{M}(t)
$$

### 7.1 If we start at equilibrium: nothing happens

If **M**(0) = **M**₀ = (0, 0, M₀)ᵀ, then **B**₀ ∥ **M**₀ ⟹ **B**₀×**M**₀ = **0** ⟹ **Ṁ** = **0**. The magnetization stays put. Equilibrium is equilibrium.

### 7.2 The longitudinal component is invariant

Decompose **M** = **M**∥ + **M**⊥. Because the cross product is linear and **B**₀×**M**∥ = **0** (parallel vectors):

$$
\dot{\mathbf{M}}(t) = \gamma\mathbf{B}_0\times\mathbf{M}_\perp(t)
$$

The right-hand rule places **B**₀×**M**⊥ in the xy-plane — meaning the *change* in **M** has no z-component. Therefore:

$$
\dot{\mathbf{M}}_\parallel(t) = 0 \;\;\Longrightarrow\;\; \mathbf{M}_\parallel(t) = \mathbf{M}_\parallel(0)
$$

In a static B₀ alone, longitudinal magnetization is conserved.

### 7.3 The transversal component precesses at the Larmor frequency

The transversal part obeys:

$$
\dot{\mathbf{M}}_\perp(t) = \gamma\mathbf{B}_0\times\mathbf{M}_\perp(t)
$$

For initial condition **M**(0) = (M₀, 0, 0)ᵀ, the solution is (to be verified in exercise 2.2):

$$
\mathbf{M}(t) = M_0\begin{pmatrix}\cos\omega_0 t \\ \sin\omega_0 t \\ 0\end{pmatrix}, \quad \omega_0 = \gamma B_0
$$

This is the **Larmor precession**. The transversal magnetization rotates in the xy-plane at angular frequency ω₀ = γB₀, or in Hz: ν₀ = γB₀/(2π).

**Numbers to remember:**
- 1.5 T → ν₀ ≈ 64 MHz
- 3 T → ν₀ ≈ 128 MHz
- 7 T → ν₀ ≈ 298 MHz

These are radio frequencies — that's why the field is called **N**uclear **M**agnetic **R**esonance and the excitation pulses are called **R**adio**F**requency pulses.

> **The key picture:** In a static B₀, the longitudinal axis is "frozen" and the transversal plane "spins" at ω₀. Everything we'll do in the next several lectures — RF excitation, signal detection, relaxation, gradient encoding — is built on top of this precession.

---

## 🧠 Self-test questions

Try these without scrolling back. If you can answer them, you've got the lecture.

1. **Coordinate sanity.** A patient lies supine, head-first into the bore. In which anatomical direction does the +z axis point? What about +y?
2. **Vector decomposition.** Write **v** = (3, 4, 5)ᵀ as a sum of its longitudinal and transversal parts. What is v₊ in complex-plane notation? What is θ_v (in degrees)?
3. **Equilibrium magnetization.** If you double B₀, what happens to M₀? If you halve T_K (an obviously hypothetical scenario in vivo), what happens to M₀?
4. **Why such big magnets?** Looking at M₀ ≈ (ρ/4)·(γ²ℏ²/k_BT_K)·B₀, identify *every* parameter and explain why B₀ is the only one a scanner designer can adjust to increase signal.
5. **Force on a paperclip.** Why is the force on a paperclip near the *center* of the bore tiny, but enormous in the doorway of the scanner room?
6. **Bloch term map.** Without looking, write down the Bloch equation and label which term is responsible for (a) RF excitation, (b) T₁ recovery, (c) T₂ decay.
7. **Larmor frequency.** Compute ν₀ for protons at 1.5 T, 3 T, and 7 T using γ/(2π) = 42.58 MHz/T.
8. **Cross-product reasoning.** Without computing components, explain why **B**₀×**M**₀ = **0** when both point along z.
9. **Conservation in B₀.** In a static B₀ with no relaxation, is energy conserved? Is |**M**| conserved? Is M_z conserved? Defend each answer using the Bloch equation.
10. **The minus sign.** If a textbook writes Ṁ = −γ**B**×**M** and our notes write Ṁ = +γ**B**×**M**, are they describing the same physics? What's the practical consequence of the sign choice?

---

## 🔗 Connections to upcoming lectures

- **Excitation (next lecture):** We needed to assume "an initial condition like (M₀, 0, 0)ᵀ exists." But how do you tip equilibrium magnetization into the transversal plane in the first place? That's what an **RF pulse B₁⁺(t)** does, and we'll analyze it by adding it to the Bloch equation.
- **Signal detection:** The precessing transversal magnetization induces an EMF in a receive coil via Faraday's law. The math of "M⊥ rotating at ω₀ → coil voltage" is principle 2 + electrodynamics.
- **Relaxation:** Once we restore the T₁/T₂ terms we dropped here, M⊥ doesn't just precess forever — it decays. M∥ doesn't stay put — it recovers toward M₀.
- **k-space (Module 2):** When we add gradient fields **B**_G(**r**, t), the precession frequency becomes *position-dependent* — and that's the origin of all spatial encoding in MRI.

---

## 📚 Suggested reading (from the lecture notes)

- **Tipler & Mosca**, *Physics for Scientists and Engineers* — for an electrodynamics refresher.
- **Nolting**, *Theoretical Physics 3: Electrodynamics* (2016) — more rigorous treatment.
- **Brown, Cheng, Haacke, Thompson, Venkatesan**, *Magnetic Resonance Imaging: Physical Principles and Sequence Design* (Wiley, 2014) — the standard reference for the Bloch equation derivation.
- **Bloch, F.**, *Nuclear Induction*, Phys. Rev. 70, 460 (1946). The first-hand source.

---

*Next up: how to actually tip M₀ into the transversal plane — RF excitation.*
