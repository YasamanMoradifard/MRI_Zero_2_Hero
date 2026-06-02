# Lecture 11 — Slice Selection 🍞

> **Module 2 · ROADMAP Day 13**
> *How do we excite just one thin slab of tissue instead of the whole body?*

Up to now, every RF pulse you've fired hit **all** the spins at once: if `ω_HF ≈ ω₀`, the whole sample is excited. That's useless for imaging — you'd get one signal mixed from the entire body. This lecture fixes that. The trick is to make the Larmor frequency **depend on position** (with a gradient) and then transmit RF at only the frequencies that live in the slab you care about. Three sub-topics:

1. **11.1 Slice selection** — *where* the slice sits and how it's oriented.
2. **11.2 Slice thickness** — *how thick* it is, and the RF pulse shape that produces it.
3. **11.3 Full Bloch treatment** — the honest version, plus the **slice-rephasing gradient** you always need in practice.

---

## 11.1 Selecting a slice

### The starting point (recap from Chapter 6)

A B₁⁺ pulse only tips magnetization where it is **on resonance**:

- if `ω_HF ≈ ω₀` → excitation,
- otherwise → `M(r,t)` essentially untouched.

With only `B = B₀ + B₁⁺`, every spin shares the same `ω₀`, so excitation is **global**. Nothing is spatially selective yet.

### Add a gradient → frequency becomes a map of position

Switch on a gradient field while the RF plays: `B(t) = B₀ + B_G(r,t) + B₁⁺(t)`. For an x-gradient the local angular frequency becomes

$$\omega(x) = \omega_0 + \gamma G_x x .$$

Now frequency is a **ruler for position**. Transmit at `ω_HF` and *only* the spins whose local frequency equals `ω_HF` are excited. Solving for that position:

$$\boxed{\,x_1 = \frac{1}{\gamma G_x}\,(\omega_{\mathrm{HF}} - \omega_0)\,}$$

`ω₀` is fixed for a given scanner; **you** choose `G_x` and `ω_HF`. Note they are *not* independent — doubling `G_x` and the offset `(ω_HF − ω₀)` together excites the same position, just with a steeper frequency-vs-position line.

### ⚠️ The gradient *field* always points along z

This is the single most-tested confusion in the lecture (it's literally a pop-quiz). The **gradient vector** `G = (G_x, G_y, G_z)` describes *along which spatial direction the field strength changes*. The **field it produces is always parallel to `e_z`** (parallel to B₀); only its **magnitude** varies with position:

$$\mathbf B_G(\mathbf r) = (\mathbf G\cdot\mathbf r)\,\mathbf e_z .$$

So for a y-gradient that excites the plane `y = y₁`, the correct field is `(0, 0, G_y·y)` — *not* `(0, G_y·y, 0)`. The field points along z; its *value* ramps along y.

### You always excite a whole *plane*

An x-gradient excites every spin at `x = x₁` regardless of `y` and `z` → the plane `r = (x₁, y, z)ᵀ`. A y-gradient excites `y = y₁`. The excited plane is always **perpendicular to `G`**, because lines of constant field (iso-B) are lines of constant frequency (iso-ω), and those are perpendicular to the gradient direction.

### Oblique slices (the general case)

Use all three gradient channels, `G = (G_x, G_y, G_z)ᵀ`. The resonance condition generalizes from one term to a dot product:

$$\omega_{\mathrm{HF}} = \omega_0 + \gamma\,(G_x x_1 + G_y y_1 + G_z z_1) = \omega_0 + \gamma\,\mathbf G\cdot\mathbf r_1 .$$

Rearrange and divide by `G = |G|` to read off a clean geometric meaning. With the unit vector `e_G = G/G`:

$$\mathbf e_G\cdot\mathbf r_1 = \frac{\omega_{\mathrm{HF}}-\omega_0}{\gamma G}
\quad\Longrightarrow\quad
\boxed{\,r_{\parallel \mathbf G} = \frac{\omega_{\mathrm{HF}}-\omega_0}{\gamma G}\,}$$

`r_∥G = e_G·r₁` is just the **signed distance of the excited plane from the origin**, measured along the gradient direction. Same physics as the 1D case, dressed in vector clothes: pick a gradient *direction* to set the slice *orientation*, pick the offset frequency to set its *position*.

---

## 11.2 Slice thickness

### Why we don't want an infinitely thin slice

A single frequency excites a plane of thickness ≈ 0 → ≈ 0 protons → **no signal**. Signal ∝ number of protons in the slice ∝ slice thickness. Typical human MRI slices are **0.5–5 mm**.

### A band of frequencies → a slab of finite thickness

Transmit a **range** of frequencies `Δω` centered on `ω_center` instead of a single tone. Because position is a linear function of frequency, `x = (ω − ω₀)/(γG_x)`, a frequency band maps directly to a spatial slab:

$$x_{\text{center}} = \frac{\omega_{\text{center}}-\omega_0}{\gamma G_x},\qquad
x_{\text{left/right}} = \frac{\omega_{\text{left/right}}-\omega_0}{\gamma G_x}$$

$$\boxed{\,\Delta x = x_{\text{right}}-x_{\text{left}} = \frac{\Delta\omega}{\gamma G_x}\,}$$

**Read this carefully:** `Δx ∝ G_x⁻¹` (another pop-quiz). Slice position *and* thickness are jointly controlled by **gradient amplitude** and **RF frequency content**:

- want a **thinner** slice at the same bandwidth → **stronger** gradient;
- keep the gradient, want it thinner → use a **narrower** RF bandwidth;
- neither `(ω_center − ω₀)` nor `Δω` depends on `B₀` — `B₀` only sets the absolute carrier `ω₀`, not the *offset* or the *bandwidth*. (This is exactly the closing question of Exercise 11.1.)

### What RF pulse shape gives a rectangular slab?

We want the **spectrum** of B₁⁺ to be a boxcar (rect) of width `Δν` centered at `ν_center`. The time-domain pulse is therefore the **inverse Fourier transform of a rect, which is a sinc**:

$$\mathcal F^{-1}\{\Pi(\nu;\nu_{\text{center}},\Delta\nu)\} = \Delta\nu\,\operatorname{sinc}(\pi t\,\Delta\nu)\,e^{\,i 2\pi t\,\nu_{\text{center}}}.$$

So in the rotating frame `B₁⁺'(t) ∝ Δν·sinc(πtΔν)`, and the carrier `exp(i2πt·ν_center)` shifts the band — i.e. **a linear phase in time = a shift in frequency = a shift in slice position**. Three knobs, three jobs:

| Factor in `B₁⁺(t)` | Controls |
|---|---|
| overall amplitude `α γ⁻¹` | **flip angle** (stronger B₁⁺ → bigger flip) |
| `exp(−i2πt ν_center)` | **slice center position** (linear phase ⇔ shift) |
| `Δν · sinc(πtΔν)` | **slice thickness** (FT of sinc = boxcar of width Δν) |

### Fixing the flip-angle constant (a tidy little derivation)

The flip angle generalizes from `α = γ B₁⁺ t_HF` to the integral of the envelope:

$$\alpha = \gamma\!\int_{-\infty}^{\infty} B_1^{+\prime}(t)\,dt .$$

Write `B₁⁺'(t) = c·Δν·sinc(πtΔν)` and use the standard integral `∫_{−∞}^{∞} sinc(x)\,dx = π` (with the unnormalized `sinc(x)=sin x/x`). Substituting `u = πΔν t`:

$$\alpha = c\,\gamma\,\Delta\nu\!\int \operatorname{sinc}(\pi t\Delta\nu)\,dt
= c\,\gamma\cdot\frac{1}{\pi}\!\int \operatorname{sinc}(u)\,du = c\,\gamma .$$

Hence `c = α γ⁻¹`, giving the pulse used on the slides:

$$\boxed{\,B_1^{+\prime}(t) = \alpha\,\gamma^{-1}\,\Delta\nu\,\operatorname{sinc}(\pi t\,\Delta\nu)\,}$$

### The catch: small-flip-angle approximation

The neat "**slice profile = Fourier transform of the RF pulse**" rule is **only valid for small flip angles** (roughly `α ≲ 30°`). It comes from linearizing the Bloch equations. For large flips (90°, 180°) the Bloch nonlinearity distorts the profile, and the pulse must be **optimized numerically** to get a clean rectangle (e.g. Shinnar–Le Roux pulses — beyond this lecture). The notebook simulates exactly this: a small-flip sinc gives a near-rectangular slab; a 90°/180° sinc develops ripples and a rounded, distorted profile.

### Reality bites

- A perfect rect profile needs a sinc from `t = −∞` to `+∞`. We can only play a **truncated** pulse → ripples and rounded edges (a Gibbs-like artifact). The real profile has sloped shoulders.
- Sloped shoulders mean adjacent slices overlap → **slice crosstalk**. Mitigation: leave gaps and acquire slices **interleaved** (1, 3, 5, 7, 2, 4, 6, 8 …) so neighbors aren't excited back-to-back.
- **Timing matters:** `B_G(r,t)` and `B₁⁺(t)` must be on **at the same time**. Right gradient + offset RF at the wrong moment ≠ slice selection.

---

## 11.3 Full Bloch treatment & the slice rephaser

### The honest equation

Slice selection really means solving the Bloch equation with all three fields on at once:

$$\frac{d\mathbf M(t)}{dt} = \gamma\,\mathbf B(t)\times\mathbf M(t),\qquad
\mathbf B = \mathbf B_0 + \mathbf B_G(\mathbf r,t) + \mathbf B_1^{+}(t).$$

For a z-gradient the effective field in the rotating frame mixes the gradient (`e_z'`) and RF (`e_x'`) components, giving a coupled system for `(M_x', M_y', M_z')`. This is genuinely hard; closed form exists only in the small-flip regime, where it reproduces the sinc/FT result above.

### Why you always need a slice-rephasing gradient

Here's the part the simple picture hides. During the **second half** of the (symmetric) slice-select gradient lobe — after the pulse has effectively deposited the magnetization at its center — spins across the slice thickness accumulate a **linear phase**: a nonzero k-vector along the slice direction,

$$k_{\text{slice}}(t) = \gamma\!\int G_{\text{slice}}\,dt' \neq 0 .$$

Different depths in the slice point in different directions → they **dephase** → the slice's net transverse signal collapses. The cure is a **slice-rephasing gradient**: a negative lobe whose zeroth moment is **half** that of the slice-select gradient. It rewinds `k_slice` back to **0**, bringing every spin in the slice back into phase and restoring full signal. (For large flip angles, the rephaser amplitude is found numerically.)

> **Mnemonic:** the slice-select gradient *spreads* the slice's spins in phase; the rephaser (half the area, opposite sign) *gathers* them back. `k_slice: 0 → up → back to 0`.

---

## 📐 Key equations (cheat sheet)

| Quantity | Formula |
|---|---|
| Frequency map (x-grad) | `ω(x) = ω₀ + γ G_x x` |
| Gradient field | `B_G(r) = (G·r) e_z` (always ∥ z!) |
| Slice position (1D) | `x₁ = (ω_HF − ω₀)/(γ G_x)` |
| Oblique slice distance | `r_∥G = (ω_HF − ω₀)/(γ G)`, plane ⊥ **G** |
| Slice thickness | `Δx = Δω/(γ G_x)`  → `Δx ∝ G_x⁻¹` |
| Slice-select RF | `B₁⁺'(t) = α γ⁻¹ Δν · sinc(πtΔν)` |
| Flip angle | `α = γ ∫ B₁⁺'(t) dt`, using `∫ sinc dx = π` |
| Rephaser | zeroth moment = ½ that of slice-select lobe; `k_slice → 0` |

---

## 🧠 Intuition in one paragraph

A gradient turns the Larmor frequency into a position label. Transmit one frequency and you light up one plane perpendicular to the gradient; transmit a *band* of frequencies and you light up a *slab* whose thickness is the bandwidth divided by `γG`. The pulse shape that carves out a clean rectangular slab is a sinc in time (because a rect in frequency ↔ a sinc in time). That's exactly true only for small flips; big flips need numerically designed pulses. And because the slice-select gradient leaves the slice's spins fanned out in phase, you always tack on a half-area rephasing lobe to snap them back together before reading out.

---

## 🔗 Connections to Exercise Sheet 11

- **E11.1 (Slice selection 1)** is pure plug-and-play with `B_G(r) = (G·r) e_z`, `(ν_center − ν₀) = (γ/2π) G·r_center`, and `Δν = (γ/2π) G Δx`. The notebook builds these as reusable functions and tabulates sagittal/coronal/axial/oblique cases for `G = ±10` and `20 mT/m`, and confirms the punchline: **offsets and bandwidths are independent of B₀**.
- **E11.2 (Slice selection 2)** is the "gymnastics" exercise: two perpendicular slice selections about the `x'`-axis. The notebook composes rotation matrices `R_x(α)` for the four regions (slice 1 only, slice 2 only, the crossing region, the rest) and reproduces the given hand-solution `M(t_after α₁) = M(t_after α₂) = (0, −M₀, 0)ᵀ` exactly — then finishes both cases (i) 90°/90° and (ii) 90°/180°.

---

## 🚩 Common pitfalls

1. **Thinking the gradient field tilts.** It never does — `B_G ∥ e_z` always. Only its magnitude varies with position.
2. **Forgetting `Δx ∝ 1/G`.** A *stronger* gradient makes a *thinner* slice (at fixed bandwidth), not a thicker one.
3. **Believing "slice profile = FT of pulse" universally.** True only for small flip angles.
4. **Dropping the rephaser.** Without it the slice is dephased and the signal tanks.
5. **Confusing slice-select direction with in-plane direction.** `G` sets the slice *normal*; the slice is the plane ⊥ G.

---

## ✅ Self-test (no peeking!)

1. Write `ω(x)` for an x-gradient and explain in words why it makes excitation spatially selective.
2. A scanner has `ω₀` fixed. You want to excite `x₁ = 3 cm` with `G_x = 8 mT/m`. What offset `(ω_HF − ω₀)` do you transmit? (Set up the formula; the notebook computes the number.)
3. Why must `B_G` point along `e_z`? What does the gradient *vector* `G` actually describe?
4. For a y-gradient exciting `y = y₁`, which of `(0, G_y y, 0)` or `(0, 0, G_y y)` is the correct `B_G`, and why?
5. Derive `Δx = Δω/(γG_x)` starting from `x = (ω − ω₀)/(γG_x)`.
6. You halve the slice thickness but keep the RF bandwidth fixed. What must happen to `G_x`?
7. Why is the slice-select RF envelope a sinc? What sets (a) its flip angle, (b) its slice center, (c) its slice thickness?
8. State the small-flip-angle approximation and explain what goes wrong at 90°/180°.
9. What is the slice-rephasing gradient, what is its zeroth moment relative to the slice-select lobe, and what does it accomplish in k-space?
10. Does `(ν_center − ν₀)` or `Δν` depend on `B₀`? Justify.

<details>
<summary><b>Answer key (open only after attempting)</b></summary>

1. `ω(x) = ω₀ + γG_xx`. Frequency becomes a function of position, so a single transmit frequency resonates with only one position (plane).
2. `(ω_HF − ω₀) = γ G_x x₁`. In ordinary frequency, `(ν − ν₀) = (γ/2π)·8e-3·0.03 ≈ 10.2 kHz`.
3. The field must add to B₀ along z to change `|B|` (and hence frequency) without tilting it; `G = ∇B_z` tells you *along which spatial axis* `B_z` ramps.
4. `(0, 0, G_y y)`. The field is along z; its magnitude varies along y.
5. `x` is linear in `ω` with slope `1/(γG_x)`, so a band `Δω` maps to `Δx = Δω·(1/(γG_x))`.
6. `G_x` must **double** (since `Δx ∝ 1/G_x`).
7. A rectangular frequency band ↔ a sinc in time. (a) overall amplitude `αγ⁻¹`; (b) carrier/linear phase `exp(−i2πν_center t)`; (c) the `Δν·sinc(πtΔν)` factor.
8. "Slice profile ≈ FT of B₁⁺" holds when `α ≲ 30°`. At large flips the Bloch equation is nonlinear, profiles distort/ripple, and B₁⁺ must be optimized numerically.
9. A negative gradient lobe with **half** the zeroth moment of the slice-select lobe; it returns `k_slice` to 0, rephasing spins across the slice and restoring signal.
10. Neither. Both depend only on `γ`, `G`, and geometry (position/thickness). `B₀` only sets the absolute `ν₀`.

</details>

---

*Next up (Day 14, CMRI L2): from a slice to an image — Fourier image reconstruction. The frequency-as-position idea you just learned is the seed of k-space.*
