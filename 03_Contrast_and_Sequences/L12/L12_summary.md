# L12 — Relaxation & Contrast (T1, T2, weighting, FLAIR/STIR, mapping)

> **Module 3 — Contrast & Pulse Sequences · Day 18 of the ROADMAP**
> *Source: MRI1 Lecture 12 (Frederik Laun, FAU) + Exercise Sheet 12*
> *Why does grey matter look different from white matter? This is the lecture where physics becomes a diagnostic image.*

---

## 🎯 TL;DR

Lecture 12 is the bridge from *signal* to *contrast*. Up to now magnetization just precessed and got detected. Now we add the missing ingredient — **relaxation** — and discover that the relaxation *times* $T_1$ and $T_2$ are **tissue-specific**. That single fact is the entire basis of MRI contrast.

The whole lecture hangs on two solved differential equations:

$$
\boxed{\;M_\parallel(t) = \big(M_\parallel(0) - M_0\big)\,e^{-t/T_1} + M_0\;}
\qquad
\boxed{\;M_+(t) = e^{\,i\varphi(t) - t/T_2}\,M_+(0)\;}
$$

Everything else — $T_1$-weighting, $T_2$-weighting, FLAIR, STIR, $T_1/T_2$ mapping — is just **choosing *when* to measure** so that these two exponentials separate the tissues you care about.

The two operational mantras:

| Weighting | Strategy | Knob | Bright tissue |
|---|---|---|---|
| **$T_1$-weighted** | *Be impatient* — measure before everything recovers | short **TR** | short-$T_1$ (fat) |
| **$T_2$-weighted** | *Be patient* — wait for the fast-decaying signals to die | long **TE** | long-$T_2$ (fluid) |

---

## 1. The signal chain

The lecture frames the whole game as a pipeline:

$$
M_\parallel \;\longrightarrow\; M_\perp \;\longrightarrow\; \text{Signal}
$$

You can only detect **transverse** magnetization $M_\perp$ (Faraday induction needs something rotating in the $xy$-plane). But $M_\perp$ is created by tipping the **longitudinal** magnetization $M_\parallel$ that has built up along $B_0$. So for the rest of the lecture we use the simplifying proportionality

$$
\text{Signal} \;\propto\; M_\parallel \quad(\text{at the moment of excitation}).
$$

This is why $T_1$ (which governs how much $M_\parallel$ is available) controls one kind of contrast, and $T_2$ (which governs how fast $M_\perp$ fades after excitation) controls another.

---

## 2. Bloch equations *with relaxation* (the "second formulation")

In Lecture 5 the Bloch equation only had precession. The full version adds two relaxation terms:

$$
\frac{d\boldsymbol{M}(t)}{dt}
= \underbrace{\gamma\,\boldsymbol{B}(t)\times\boldsymbol{M}(t)}_{\text{precession (rotates } \boldsymbol M)}
+ \underbrace{\frac{1}{T_1}\big(\boldsymbol{M}_0 - \boldsymbol{M}_\parallel(t)\big)}_{\text{regrows } M_\parallel}
- \underbrace{\frac{1}{T_2}\,\boldsymbol{M}_\perp(t)}_{\text{kills } M_\perp}
$$

- The **precession term** *only rotates* $\boldsymbol M$ — its magnitude is conserved.
- The **relaxation terms** *change the magnitude*: $M_\parallel$ is pulled back toward equilibrium $M_0$, while $M_\perp$ is dragged toward zero.
- $\gamma$ is the **nucleus-specific** gyromagnetic ratio ($2.675\times10^{8}\ \mathrm{rad\,s^{-1}T^{-1}}$ for protons). $T_1$, $T_2$ are **tissue-specific** coefficients.

### Complex-plane notation

With $\boldsymbol{B}(t)=\gamma^{-1}\omega(t)\,\boldsymbol e_z$, $\;\omega(t)=\gamma\big(B_0+B_{G,z}(t)\big)$, define the complex transverse component

$$
M_+ \equiv M_x + i M_y .
$$

Working the cross product and relaxation terms component-by-component collapses the 3 coupled real ODEs into **two decoupled equations**:

$$
\boxed{\;\dot M_+(t) = i\,\omega(t)\,M_+(t) - \frac{1}{T_2}M_+(t)\;}
\qquad
\boxed{\;\dot M_\parallel(t) = \frac{1}{T_1}\big(M_0 - M_\parallel(t)\big)\;}
$$

This is the payoff of complex notation: transverse and longitudinal dynamics **separate cleanly**, and the precession just becomes a phase factor $e^{i\varphi}$.

---

## 3. Longitudinal relaxation ($T_1$, "spin–lattice")

Solving $\dot M_\parallel = \tfrac{1}{T_1}(M_0 - M_\parallel)$:

$$
M_\parallel(t) = \big(M_\parallel(0) - M_0\big)\,e^{-t/T_1} + M_0
$$

Two cases you will use constantly:

| Starting condition | Solution | Physical setup |
|---|---|---|
| $M_\parallel(0)=0$ | $M_\parallel(t)=M_0\big(1-e^{-t/T_1}\big)$ | after a **90°** pulse (saturation recovery) |
| $M_\parallel(0)=-M_0$ | $M_\parallel(t)=M_0\big(1-2e^{-t/T_1}\big)$ | after a **180°** pulse (inversion recovery → FLAIR/STIR) |

**Typical $T_1$ times (ms):**

| Tissue | 1.5 T | 3 T |
|---|---|---|
| Fat | 340 | 380 |
| Liver | 600 | 800 |
| White matter | 900 | 1100 |
| Gray matter | 1100 | 1800 |
| Spleen | 1050 | 1300 |
| CSF | ≈4500 | ≈4500 |

Rules of thumb: **$T_1$ increases with $B_0$** (except water/CSF, which barely changes), $T_1$ is **tissue-dependent**, fluids long, fat short. Relaxation rate $R_1 = 1/T_1$.

---

## 4. Transversal relaxation ($T_2$, "spin–spin")

Solving $\dot M_+ = (i\omega - 1/T_2)M_+$:

$$
M_+(t) = e^{\,i\varphi(t)}\,e^{-t/T_2}\,M_+(0),
\qquad \varphi(t)=\int_0^t \omega(t')\,dt'
$$

so the **magnitude** decays purely exponentially:

$$
M_\perp(t) = M_\perp(0)\,e^{-t/T_2}.
$$

Geometrically this is the **inward spiral** in the $xy$-plane: the vector keeps precessing (the phase $\varphi$) while shrinking (the $e^{-t/T_2}$ envelope).

**Typical $T_2$ times (ms):**

| Tissue | 1.5 T | 3 T |
|---|---|---|
| Bone | — | ≈0.3 |
| Myelin | — | 0.1–0.3 |
| Liver | 45 | 35 |
| Fat | 60 | 70 |
| White matter | 70 | 70 |
| Gray matter | 95 | 100 |
| Spleen | 80 | 60 |
| CSF | ≈2200 | ≈2200 |

Key facts: **$T_2 \ll T_1$** always (magnetization fades faster than it regrows); $T_2$ has **little $B_0$ dependence**; **solids have ultra-short $T_2$** (sub-millisecond) → invisible to conventional sequences whose minimum TE is a few ms (this motivates UTE, §6). $M_\parallel$ and $M_\perp$ relax **independently and at different rates**. Rate $R_2 = 1/T_2$.

> ⚠️ **Slide self-correction worth remembering.** One summary slide carries a red "Error" note: the deck used to claim T2 is "longer for WM than for GM." The corrected (and table-consistent) statement is **longer for GM than for WM** — gray matter T2 (95–100 ms) > white matter T2 (70 ms). The mnemonic from the pop quiz: *gray matter is more "fluid-like"/isotropic, so it sits closer to CSF than white matter does* → GM has both the longer $T_1$ **and** the longer $T_2$.

---

## 5. $T_2$-weighting — *be patient*

Procedure: excite, then **wait** a time $\text{TE}$ (echo time) before detecting. During the wait, $M_\perp$ has decayed by $e^{-\text{TE}/T_2}$, which is *different for each tissue*:

$$
S \;\propto\; M_\perp(0)\,e^{-\text{TE}/T_2}.
$$

- **Short-$T_2$ tissues → hypointense (dark)**; **long-$T_2$ tissues → hyperintense (bright)**.
- Classic example: **fluid bright, liver dark**. CSF lights up in a $T_2$-weighted brain image.
- **Small TE** → all tissues nearly isointense (no contrast, ≈ proton-density). **Large TE** → maximal $T_2$ contrast, but lower overall SNR.

---

## 6. $T_1$-weighting — *be impatient*

After a 90° pulse $M_\parallel=0$; it must **regrow** before the next excitation can produce signal. Apply a second 90° pulse at $t=\text{TR}$ (repetition time). Whatever $M_\parallel$ has recovered by then becomes the next signal:

$$
S \;\propto\; M_\parallel(\text{TR}) = M_0\big(1-e^{-\text{TR}/T_1}\big)
$$

(for the simple 90°-saturation picture).

- **Short-$T_1$ tissues → hyperintense (bright)**; **long-$T_1$ tissues → hypointense (dark)**.
- Example: **fat bright, liver darker, CSF darkest**.
- Strategy: *don't let the system fully relax to $M_0$* — use short TR so the fast-recovering (short-$T_1$) tissues stand out.

**Steady state.** With repeated equally-spaced 90° pulses, after a few TRs each tissue settles to a constant pre-pulse value. This matters practically: it means **every k-space line of a given tissue carries the same $T_1$-weighting**, so the contrast is consistent across the image. (Flip angles below 90° and leftover $M_\perp$ are an MRI-2 topic.)

---

## 7. The $T_1$w / $T_2$w / $\rho$w contrast matrix

Combine both effects. The signal of a spin-echo–like acquisition is approximately

$$
S \;\propto\; \rho \,\big(1-e^{-\text{TR}/T_1}\big)\,e^{-\text{TE}/T_2}
$$

where $\rho$ is proton density. The two knobs TR and TE define a 2×2 map:

| | **short TE** (ignore $T_2$) | **long TE** (emphasize $T_2$) |
|---|---|---|
| **short TR** (emphasize $T_1$) | **$T_1$-weighted** | (mixed — usually avoided) |
| **long TR** (ignore $T_1$) | **$\rho$ (proton-density) weighted** | **$T_2$-weighted** |

- **$T_1$w:** short TR, short TE.
- **$T_2$w:** long TR, long TE.
- **$\rho$w:** long TR, short TE — both exponentials ≈ neutral, so signal ≈ proton density.

> 🔬 **Excursion — UTE imaging.** Because solids ($T_2\sim0.1$–$0.3$ ms) vanish before a conventional TE of a few ms, **ultra-short echo time** (UTE) sequences with radial readout push TE down to ~0.26 ms to *make bone visible* — e.g. the FAU project aiming to replace pediatric orthodontic X-rays with radiation-free MRI.

---

## 8. FLAIR & STIR — inversion recovery that *nulls* one tissue

Both add a **180° inversion** pulse first (so $M_\parallel(0)=-M_0$), then wait an **inversion time** $\text{TI}$, then excite. From the inversion-recovery solution $M_\parallel(t)=M_0(1-2e^{-t/T_1})$, the signal of a tissue passes through **zero** at

$$
M_\parallel(\text{TI})=0 \;\Longrightarrow\; \boxed{\;\text{TI} = \ln(2)\,T_1 \approx 0.693\,T_1\;}
$$

Choose TI to null the tissue you want to *suppress*:

| Sequence | Suppresses | Tissue $T_1$ | TI $=\ln2\cdot T_1$ | Use |
|---|---|---|---|---|
| **FLAIR** (FLuid Attenuated IR) | water / CSF | 4500 ms | ≈ **3119 ms** | $T_2$-type contrast *without* the dominant CSF glare (e.g. periventricular lesions, MS plaques) |
| **STIR** (Short-Tau IR) | fat | 340 ms @1.5 T (380 @3 T) | ≈ **236 ms** (263 @3 T) | suppress bright fat so lesions/edema stand out (e.g. breast, musculoskeletal) |

The trick: at the chosen TI the **target** tissue has $M_\parallel=0$ (no signal), while *other* tissues, having different $T_1$, are off the null and still give signal.

---

## 9. $T_1$ and $T_2$ mapping (vs weighting)

**Weighting** gives a qualitative image whose brightness *blends* $\rho$, $T_1$, $T_2$. **Mapping** measures the *physical parameter itself*, voxel by voxel:

- **$T_2$ map:** acquire at several TE; fit $S(\text{TE}) = M_0\,e^{-\text{TE}/T_2}$ per voxel → image of $T_2$ values.
- **$T_1$ map:** acquire at several TI; fit $S(\text{TI}) = M_0\big(1-2e^{-\text{TI}/T_1}\big)$ per voxel → image of $T_1$ values.

Maps are **quantitative and scanner-comparable** but **slower and less robust** than weightings, so clinics mostly use weightings. A handy sanity check from the lecture's pop quiz: a $T_1$ map has a colorbar up to ~3000 ms; a $T_2$ map only up to ~300 ms (because $T_2 \ll T_1$).

---

## 🧮 Equation cheat-sheet

| Quantity | Equation |
|---|---|
| Bloch + relaxation | $\dot{\boldsymbol M}=\gamma\boldsymbol B\times\boldsymbol M+\tfrac1{T_1}(\boldsymbol M_0-\boldsymbol M_\parallel)-\tfrac1{T_2}\boldsymbol M_\perp$ |
| Complex transverse | $\dot M_+ = (i\omega - 1/T_2)\,M_+$ |
| Longitudinal | $M_\parallel(t)=(M_\parallel(0)-M_0)e^{-t/T_1}+M_0$ |
| Saturation recovery | $M_\parallel(t)=M_0(1-e^{-t/T_1})$ |
| Inversion recovery | $M_\parallel(t)=M_0(1-2e^{-t/T_1})$ |
| Transverse magnitude | $M_\perp(t)=M_\perp(0)\,e^{-t/T_2}$ |
| SE signal | $S\propto\rho(1-e^{-\text{TR}/T_1})\,e^{-\text{TE}/T_2}$ |
| IR null time | $\text{TI}=\ln(2)\,T_1$ |

---

## 🪤 Common pitfalls

- **$T_2$ does *not* depend on $B_0$ the way $T_1$ does.** $T_1$ grows with field strength (water excepted); $T_2$ is roughly field-independent.
- **GM > WM for both $T_1$ and $T_2$.** Don't trust the uncorrected slide; check the tables (the deck itself flags the old wording as an error).
- **Signal ∝ $|M_\parallel|$, not $M_\parallel$.** With magnitude reconstruction the *sign* after an inversion is lost — a tissue with $M_\parallel=-0.6\,M_0$ shows up just as bright as one with $+0.6\,M_0$. This is exactly why a tissue can be "nulled" only at its single TI crossing.
- **Short TR ≠ automatically good $T_1$ contrast.** Too short and *every* tissue is near zero (low signal); the best $T_1$ contrast sits at intermediate TR comparable to the tissue $T_1$ values.
- **Fun gotcha (Prof. Laun's pop quiz):** flip the sign of the $T_2$ term so $\dot{\boldsymbol M}=\dots+\tfrac1{T_2}\boldsymbol M_\perp$ and $M_\perp$ grows *exponentially* instead of decaying — the "image" would literally blow up. A good reminder that the **minus sign on the $T_2$ term is doing real work.**

---

## ✅ Self-test (10 questions)

Try these before peeking at the answer key.

1. Write the full Bloch equation with relaxation and name what each of the three terms does to $\boldsymbol M$.
2. Why can complex notation $M_+=M_x+iM_y$ simplify the Bloch equations? What does the $i\omega$ term represent physically?
3. Starting from $\dot M_\parallel=\tfrac1{T_1}(M_0-M_\parallel)$, derive $M_\parallel(t)$ for a general initial value $M_\parallel(0)$.
4. A tissue has $T_1=900$ ms. After a 90° pulse, what fraction of $M_0$ has recovered at $t=900$ ms? At $t=2700$ ms?
5. Rank CSF, gray matter, white matter, and fat by (a) $T_1$ and (b) $T_2$, longest to shortest. State the mnemonic.
6. To make CSF appear **bright**, do you use short or long TE? Short or long TR? What weighting is this?
7. Explain in one sentence each why $T_1$-weighting wants short TR while $T_2$-weighting wants long TE.
8. Derive the inversion time that nulls a tissue, and compute TI for fat ($T_1=340$ ms) and for CSF ($T_1=4500$ ms). Which is FLAIR and which is STIR?
9. Why are bone and myelin invisible in conventional MRI, and what sequence trick recovers them?
10. What is the difference between a $T_2$-**weighted** image and a $T_2$ **map**? Why are maps less common in the clinic?

<details>
<summary><b>Answer key</b> (click to expand)</summary>

1. $\dot{\boldsymbol M}=\gamma\boldsymbol B\times\boldsymbol M+\tfrac1{T_1}(\boldsymbol M_0-\boldsymbol M_\parallel)-\tfrac1{T_2}\boldsymbol M_\perp$. Term 1 rotates $\boldsymbol M$ (precession, magnitude conserved); term 2 regrows the longitudinal component toward equilibrium $M_0$ with rate $1/T_1$; term 3 destroys the transverse component with rate $1/T_2$.

2. Because the transverse $(M_x,M_y)$ dynamics decouple from $M_\parallel$ and become a single complex ODE $\dot M_+=(i\omega-1/T_2)M_+$. The $i\omega$ term is **precession** — a continuous rotation (phase accumulation) in the complex plane at angular frequency $\omega=\gamma(B_0+B_{G,z})$.

3. General solution $M_\parallel(t)=(M_\parallel(0)-M_0)e^{-t/T_1}+M_0$ (verify by differentiating: $\dot M_\parallel=-\tfrac1{T_1}(M_\parallel(0)-M_0)e^{-t/T_1}=-\tfrac1{T_1}(M_\parallel(t)-M_0)=\tfrac1{T_1}(M_0-M_\parallel(t))$ ✓).

4. $M_\parallel/M_0=1-e^{-t/T_1}$. At one $T_1$ (900 ms): $1-e^{-1}\approx0.63$. At three $T_1$ (2700 ms): $1-e^{-3}\approx0.95$.

5. (a) $T_1$: CSF > GM > WM > fat. (b) $T_2$: CSF > GM > WM > fat (fat $T_2\approx60$ ms slightly below WM at 1.5 T; the key point is CSF longest, GM > WM). Mnemonic: fluids longest, solids/fat shortest; **GM is more CSF-like than WM**, so GM > WM for both.

6. Long TE and long TR → **$T_2$-weighted**. CSF has the longest $T_2$, so it retains signal at long TE and appears bright.

7. $T_1$w: short TR stops long-$T_1$ tissues from recovering, so their reduced $M_\parallel$ makes them dark and short-$T_1$ tissues bright → contrast from $T_1$ differences. $T_2$w: long TE lets short-$T_2$ tissues decay away while long-$T_2$ tissues survive → contrast from $T_2$ differences.

8. Set $M_0(1-2e^{-\text{TI}/T_1})=0\Rightarrow \text{TI}=\ln2\,T_1$. Fat: $0.693\times340\approx236$ ms → **STIR**. CSF: $0.693\times4500\approx3119$ ms → **FLAIR**.

9. Their $T_2$ is sub-millisecond (≈0.1–0.3 ms), so transverse magnetization is essentially gone before the minimum TE (a few ms) of conventional Cartesian sequences. **UTE** (ultra-short echo time, radial readout, TE ~0.26 ms) captures them.

10. A weighted image's brightness blends $\rho$, $T_1$, and $T_2$ qualitatively; a map fits the relaxation model per voxel to report the actual $T_1$ (or $T_2$) value in milliseconds — quantitative and scanner-independent. Maps require multiple acquisitions (range of TE or TI) plus fitting, so they're slower and less robust, hence weightings dominate clinically.

</details>

---

## 🔗 Where this leads

You can now predict MR contrast for any tissue and any $(\text{TR},\text{TE},\text{TI})$. Next up in Module 3: **L13 — Spin Echo vs Gradient Echo** (how the echo is actually formed, and the difference between $T_2$ and $T_2^\*$), then **L14 — Spoiler Gradients** (why steady-state sequences need to destroy leftover $M_\perp$).

*Run `L12_notebook.ipynb` next — it simulates the Bloch equations, reproduces every figure in this lecture on synthetic data, and works through Exercises 12.1, 12.2, and 12.4.*
