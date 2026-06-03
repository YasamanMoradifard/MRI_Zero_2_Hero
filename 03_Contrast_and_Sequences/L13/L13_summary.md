# L13 — Spin Echo vs Gradient Echo

> **Module 3 · Day 19 of the ROADMAP**
> *Why does grey matter look different from white matter — and why do the sinuses turn black on some scans but not others?*
> **Source:** MRI1 Lecture 13 (Frederik Laun, FAU Erlangen-Nürnberg, WS23/24) + Exercise Sheet 13

This lecture introduces the **single most important trick in MRI pulse-sequence design**: the spin echo. To understand it, we first have to admit something the earlier lectures swept under the rug — the static field `B₀` is *never* perfectly uniform. That imperfection creates a new, fast signal decay called `T₂'`, which combines with intrinsic `T₂` into the experimentally observed `T₂*`. The spin echo is the clever 180° pulse that *undoes* the `T₂'` part. A sequence that doesn't bother is called a gradient echo, and the difference between them is exactly what makes air-tissue interfaces glow dark on a GRE but stay bright on a SE.

---

## 1. The big picture in one paragraph

A real magnet has field perturbations `ΔB(r)` on top of the nominal `B₀`. Each spin packet therefore precesses at its own slightly-off Larmor frequency. Within a single voxel this spread of frequencies makes the little transverse magnetization "arrows" fan out (**phase dispersion**), and since the measured signal is the *vector sum* of all arrows, it shrinks. We call this extra decay `T₂'`. Combined with the genuine, irreversible `T₂` decay, we observe `T₂*`. The good news: the `T₂'` part is *reversible*, because the dephasing is deterministic (a fixed `ΔB(r)` produces a fixed phase rate). A 180° pulse at time `TE/2` flips every accumulated phase to its negative; the same field then "un-winds" the dephasing, and at time `TE` every arrow points the same way again — a **spin echo**. The signal at the echo is governed by `T₂` alone. A **gradient echo** is simply any sequence *without* that refocusing pulse, so its signal at `TE` is governed by `T₂*`.

---

## 2. Off-resonance and `T₂'`

### 2.1 Where `ΔB(r)` comes from
The field experienced by a spin at position `r` is `B₀ + ΔB(r)`. Sources of `ΔB(r)`:
- **Magnet imperfections** and imperfect shimming.
- **Susceptibility differences** between tissues — especially **air–tissue interfaces** (sinuses, ear canals, lungs), which generate large local field gradients.
- **Paramagnetic / diamagnetic content**, e.g. deoxyhaemoglobin (see venography below).

### 2.2 From field offset to phase
A local field offset gives a local frequency offset, and frequency integrated over time is phase:

```
Δω(r)   = γ · ΔB_z(r)            (off-resonance angular frequency)
Δφ(r,t) = Δω(r) · t             (accumulated phase, relative to on-resonance)
```

In the rotating frame, the transverse magnetization of spin packet `n` is a complex number (an "arrow"):

```
M'_⊥(rₙ, t) = |M'_⊥| · e^{i φ(rₙ,t)}    →   components |M'_⊥|·cos φ , |M'_⊥|·sin φ
```

### 2.3 Signal = "sum of arrows"
Neglecting the coil sensitivity profile and intrinsic `T₂`, the signal from a voxel is the integral (sum) of all transverse arrows:

```
S(t) ∝ ∫_voxel M'_⊥(r, t) d³r        ("sum of arrows in the box")
```

- At `t = 0` (right after the 90° pulse) all arrows are aligned → maximum signal.
- As `t` grows, arrows fan out by `Δφ = Δω·t`. Partial cancellation → **smaller signal**.
- When the spread reaches a full turn, the arrows cancel completely → **signal = 0**.

This monotone shrinking is modelled by the **often-used approximation**:

```
S(t) = S(0) · exp(−t / T₂')
```

> ⚠️ "Often-used approximation" is doing real work here. The exponential is *exact only if the distribution of off-resonances is Lorentzian*. For a uniform/box distribution (e.g. a linear gradient across a box voxel — Exercise 13.1) the true decay is a **sinc**, which oscillates and even goes negative. The notebook makes this vivid.

### 2.4 Time scale of `T₂'` (the pop-quiz logic)
If the off-resonances within a voxel have spread `σ_ω`, the signal decays on the time scale where the phase spread reaches ~1 radian:

```
σ_ω · t ≈ 1     ⇒     T₂' ≈ 1 / σ_ω
```

**Pop quiz worked example:** a distribution of width `σ_ω ≈ 40 Hz` gives `T₂' ≈ 1/40 Hz = 25 ms`. (A two-spin toy model reaches `signal = 0` when `Δω·t = π/2`; the exact `T₂'` is fiddly — hence the exercise.)

---

## 3. `T₂*` = `T₂` and `T₂'` combined

Two things attenuate the transverse signal simultaneously:
- **`T₂'`** — *reversible* dephasing from static `ΔB(r)`.
- **`T₂`** — *irreversible* intrinsic relaxation (random spin–spin interactions).

They add as **rates** (rate = 1/time):

```
R₂* = R₂ + R₂'        with   R₂* := 1/T₂* ,  R₂' := 1/T₂'
⇔   T₂* = (1/T₂ + 1/T₂')⁻¹
⇔   S(t) = S(0) · exp(−t / T₂*)
```

Because rates add, `T₂* < T₂` and `T₂* < T₂'` always — `T₂*` is the *fastest* decay. On a plot the three envelopes nest: `T₂` (slowest, top), `T₂'` (middle), `T₂*` (fastest, bottom).

**Tissue/field intuition**
- Larger `B₀`, larger susceptibility differences, worse shim → larger `R₂'` → shorter `T₂*`.
- Spin echo recovers the `R₂'` part, so a SE signal at the echo decays with `T₂`, not `T₂*`.

### 3.1 Venography — a clinical payoff
Deoxygenated venous blood (low O₂) is **more paramagnetic** than surrounding tissue → it creates `ΔB(r)` → shortens `T₂'` in and around veins → local signal drop. On a `T₂*`-weighted image (especially at **7 T**) the veins appear **dark**. This is the basis of susceptibility-weighted imaging / venography.

---

## 4. The spin echo

### 4.1 Recipe
```
90°  →  wait TE/2  →  180°  →  wait TE/2  →  SPIN ECHO at t = TE
```
`TE` (echo time) is fixed by the spacing of the 90° and 180° pulses. By convention the **centre of k-space is sampled at `TE`**, so the image contrast is set by what the signal is doing at the echo.

### 4.2 Why it works (the rephasing argument)
1. After the 90° pulse, all packets start in phase: `φ(rₙ, 0) = 0`.
2. By `TE/2`, the static offset has dephased them: `Δφ(rₙ, TE/2) = Δω(rₙ)·TE/2`. The fast/slow spins have fanned out (assuming RF duration `t_HF ≈ 0`).
3. The **180° pulse about `x'`** flips every phase to its negative: `Δφ(rₙ) → −Δφ(rₙ)`. The fan is mirrored across the `x'` axis — the spins that had run *ahead* are now *behind* by the same amount.
4. The **same `ΔB(r)`** keeps winding phase at the same rate for another `TE/2`. So each packet gains exactly `+Δω(rₙ)·TE/2` again.
5. At `t = TE`: `Δφ(rₙ, TE) = −Δω·TE/2 + Δω·TE/2 = 0`. **Complete rephasing.** The `T₂'` signal loss vanishes — *dephasing = rephasing*.

Crucially, the 180° pulse cannot undo the *random* `T₂` decay (it isn't a deterministic, position-fixed phase), so at the echo the residual signal is `S(0)·exp(−TE/T₂)`.

### 4.3 The classic picture
Plot the oscillating FID: its amplitude collapses fast along the `T₂*` envelope. After the 180° at `TE/2`, the amplitude *rebuilds*, peaking at `TE` on the `T₂` envelope (the spin echo), then dephases again along `T₂*`. The echo peak rides the slow `T₂` curve; the instantaneous envelope rides `T₂*`.

### 4.4 Why the 180° axis matters
A 180° rotation about `x'` sends `(M_x, M_y, M_z) → (M_x, −M_y, −M_z)`, i.e. `M₊ → M₊*` (complex conjugation). Conjugating `e^{−i2πk·r}` gives `e^{+i2πk·r}` — this is exactly the **`k → −k` reflection** that drives the echo (Exercise 13.2). The rotation axis of the RF pulses determines the final position of the magnetization.

---

## 5. Spin echo vs gradient echo

A **gradient echo (GRE)** sequence is *defined* as a sequence **without** a refocusing spin echo (no 180°). The echo is formed only by reversing a readout gradient, so static `ΔB(r)` dephasing is **never undone**.

| | **Spin Echo (SE)** | **Gradient Echo (GRE)** |
|---|---|---|
| Refocusing pulse | 90° + 180° | 90° (or low flip), **no 180°** |
| Off-resonance / `ΔB(r)` | **Refocused** → robust | Not refocused → sensitive |
| Long-TE contrast | **`T₂`**-weighted | **`T₂*`**-weighted |
| Speed | Slower (time for 180°) | **Fast**, often 3D |
| SAR | Higher (extra 180° RF) | **Lower** |
| Air–tissue interfaces | Look normal | **Signal dropout** (short `T₂'`) |

**Reading an image:** the scan where the sinuses/skull base show dark signal voids is the **GRE** (`T₂*`); the one that stays clean there is the **SE** (`T₂`). `SAR` = specific absorption rate (RF heating limit).

---

## 6. The Cartesian spin-echo sequence diagram

Four channels (top to bottom): RF `B₁⁺`, `G_read`, `G_phase`, `G_slice`.

- **`B₁⁺`**: 90° at `t=0`, 180° at `TE/2`, spin echo + signal detection around `TE`, next 90° at `TR`.
- **`G_read`**: a **dephaser** lobe before the echo and the **readout** lobe during detection (frequency encoding).
- **`G_phase`**: the **stepped/blipped** lobe (the comb of grey lines) — one phase-encode step per `TR`. The standard **phase-encoding notation** (stacked horizontal lines) means: one black line is played now, the others are played after later excitations, each filling a different `k_y` line.
- **`G_slice`**: slice-selection lobe (+ rephaser) for 2D; **crusher gradients** straddling the 180° to kill spurious FID; a **spoiler gradient** at the end of `TR` to destroy leftover transverse magnetization.

**2D GRE diagram:** same layout but **no 180°** and no crushers; the echo at `TE` is a gradient echo.

**3D SE diagram:** there is **no slice-selection gradient** — in 3D you phase-encode *both* through-plane and in-plane directions, so the bottom two channels are `G_phase1` and `G_phase2`. (The slides flag the old "`G_slice`/`G_phase`" labels as corrected to `G_phase1`/`G_phase2`.)

---

## 7. Two pop-quiz results worth remembering

1. **Signal at `t = 3TE/2`** (one more `TE/2` past the echo): the extra dephasing from `TE→3TE/2` is *identical* to the `0→TE/2` dephasing, so the `T₂'` factor equals the one at `TE/2`, while `T₂` keeps decaying. Answer: **`S(3TE/2) = S(TE/2)·exp(−TE/T₂)`** (option IV).
2. **A spin echo can never be `T₂*`-weighted** (option III), because the echo compensates `T₂'` (k-space centre sampled at `TE`). It *can* be `T₁`-, `T₂`-, and even ρ-weighted (for tissues with `T₂` ≳ 10 ms); true UTE / `T₂*` weighting needs the very short `TE` that SE can't provide.

---

## 8. Key takeaways

- `ΔB(r)` is unavoidable; it adds reversible dephasing `T₂'` on top of irreversible `T₂`.
- Rates add: `R₂* = R₂ + R₂'`, so `T₂*` is always the fastest decay.
- `T₂' ≈ 1/σ_ω`: a wider off-resonance spread within a voxel → faster `T₂*` decay.
- The exponential `exp(−t/T₂')` is an **approximation** (exact only for a Lorentzian spread; a box voxel gives a sinc).
- Spin echo = `90° – TE/2 – 180° – TE/2 – echo`; it refocuses `T₂'` so the echo decays with `T₂`.
- A 180° pulse reflects k-space (`k → −k`) — that *is* the echo mechanism.
- GRE = no 180°; fast, low SAR, but `T₂*`-weighted and sensitive to `ΔB(r)` (dark sinuses).
- Choose SE for clean `T₂` contrast and off-resonance robustness; GRE for speed, 3D, and susceptibility contrast (venography).

---

## 9. Self-test questions (no peeking)

1. Write the relationship between `ΔB_z(r)`, `Δω(r)`, and the accumulated phase `Δφ(r,t)`. Which one is what the receiver coil ultimately "feels" as signal loss?
2. Explain in words why the voxel signal *decreases* as time goes on, using the "sum of arrows" picture. When does it reach zero?
3. State the rate equation linking `T₂`, `T₂'`, and `T₂*`. Order the three from slowest to fastest decay and justify.
4. A voxel has an off-resonance spread of `σ_ω ≈ 100 Hz`. Estimate `T₂'`. If the spread halves, what happens to `T₂*` (assume `T₂` unchanged and `T₂' ≪ T₂`)?
5. Walk through the four steps of spin-echo rephasing. At which step does the *axis* of the 180° pulse matter, and what does it do to `M₊`?
6. Why can the 180° pulse undo `T₂'` decay but **not** `T₂` decay?
7. You see two brain scans; one shows dark signal voids near the frontal sinus, the other doesn't. Which is SE and which is GRE, and why?
8. List two practical *disadvantages* of spin-echo sequences relative to gradient echo, and one practical advantage.
9. In a 2D spin-echo sequence diagram, what are the **crusher** gradients for, and what is the **spoiler** gradient for? Why does the 3D version have no slice-selection gradient?
10. At `t = 3TE/2` after a single 90°–180° spin echo, express the signal in terms of `S(TE/2)`, `TE`, and `T₂`. Explain the `T₂'` bookkeeping that makes the `exp(−TE/T₂)` factor appear.

---

## 10. On to the exercises

The companion notebook (`L13_notebook.ipynb`) demonstrates every concept above on synthetic data and works **both problems from Exercise Sheet 13** in code:

- **Exercise 13.1 (`T₂'`):** integrate the voxel signal for a linear `ΔB_z` across a box voxel, recover the **sinc** (not an exponential!), define a reasonable `T₂'`, show that doubling the voxel *halves* `T₂'`, and discuss why "SNR ∝ voxel size" breaks down for `T₂*`-weighted sequences.
- **Exercise 13.2 (k-space & spin echo):** show that a 180° pulse about `x'` sends `k₁ → −k₁`, the reflection that makes the echo refocus.
