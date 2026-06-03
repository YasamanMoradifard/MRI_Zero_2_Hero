# L14 — Spoiler & Crusher Gradients, Contrast Agents, and MRI Safety

> **Module 3 — Contrast & Pulse Sequences · Day 20**
> *Source: MRI1 Lecture 14 (Frederik Laun, FAU Erlangen-Nürnberg, WS23-24)*

This is the final lecture of MRI 1, and it is a "loose ends" lecture: it ties off four
topics that don't each need a full lecture but matter a lot in practice. Two are about
**cleaning up unwanted signal with gradients** (spoiler and crusher gradients), one is about
**actively manipulating contrast pharmacologically** (gadolinium contrast agents), and one is
about **not hurting anyone** (MRI safety). The lecture closes with a not-exam-relevant teaser
of what MRI 2 covers.

The ROADMAP labels this day "Spoiler Gradients," but the lecture is broader, so this summary
covers all of it. The blocks are largely independent — you can read them in any order.

---

## 1. Spoiler gradients

### The problem
After you excite spins and read out an echo, there is usually **leftover transverse
magnetization** $M_+$ (the lecture's notation for $M_\perp = M_x + iM_y$) hanging around. In a
sequence that repeats every $\mathrm{TR}$ — multi-slice imaging, gradient echo, anything
steady-state — that residual coherence survives into the *next* repetition and adds a stray,
uncontrolled contribution to the signal. The result is contrast errors and banding/ghosting
artifacts.

### The idea
A **spoiler gradient** is a gradient pulse fired at the end of $\mathrm{TR}$ whose job is to make
that leftover $M_+$ **average to zero inside each voxel**. It doesn't physically erase the
magnetization — the spins are still transverse — it just *dephases* them so coherently that
their vector sum cancels.

The mechanism is pure phase. A gradient $G$ applied for a time $\tau$ gives a spin at position
$x$ a phase

$$\varphi(x) = \gamma\, x \int G(t)\,dt = \gamma\, G\, \tau\, x .$$

The phase grows **linearly across the voxel**. If the *total spread* of phase across a voxel of
width $\Delta x$ is

$$\Delta\varphi = \gamma\, G\, \tau\, \Delta x = 2\pi n, \qquad n = 1, 2, 3, \dots$$

then the little transverse vectors inside the voxel fan out into one (or $n$) complete turns of
a circle — point in every direction equally — and sum to nothing. This is exactly the slide's
picture of arrows spinning around to "zero signal."

### The math (worth doing once)
For a uniform spin density across a voxel centred at $x=0$, the voxel-averaged transverse signal is

$$S(\Delta\varphi) = \frac{1}{\Delta x}\int_{-\Delta x/2}^{+\Delta x/2} e^{\,i\,(\Delta\varphi/\Delta x)\,x}\,dx
= \frac{\sin(\Delta\varphi/2)}{\Delta\varphi/2} = \operatorname{sinc}\!\left(\frac{\Delta\varphi}{2}\right).$$

So the residual signal is a **sinc** of the intra-voxel dephasing, with its **first zero at
$\Delta\varphi = 2\pi$** (one full twist) and zeros at every $2\pi n$ afterwards. You don't need an
enormous gradient — you need one that produces an integer number of $2\pi$ wraps across a voxel.
The notebook reproduces this sinc and the arrow picture.

> **Caveat the lecture flags:** a single gradient spoiler is the simple story. Real
> steady-state spoiling also uses **RF spoiling** (incrementing the RF phase each TR) because a
> gradient alone leaves the spins coherent — they're dephased but recoverable, and stimulated
> echoes can refocus them. "Details in MRI 2."

---

## 2. Crusher gradients

### The problem
Slice-selective RF pulses have **imperfect slice profiles**. A nominal 180° refocusing pulse
delivers 180° only in the middle of the slice; toward the edges the flip angle is wrong (the
slide notes it is "rather 90° than 180°" there). A flip angle that isn't a true 180° doesn't
just refocus the echo — it also tips fresh longitudinal magnetization into the transverse plane,
creating a **spurious FID** on top of the wanted spin echo. That stray signal produces the heavy
artifacts you see in the "without crusher gradients" brain image.

### The idea
A **crusher gradient** is a *pair* of identical gradient lobes placed **symmetrically around the
180° pulse**. The key is that the wanted and unwanted signals see different total gradient moments:

- The **wanted spin-echo pathway** is inverted by the (good) 180° pulse between the two lobes.
  The phase from the first lobe is reversed by the refocusing pulse and then **undone by the
  second lobe** → net zero gradient moment → the echo is preserved.
- The **spurious FID** created *at* the imperfect pulse only experiences the **second lobe**
  → it picks up a $2\pi n$ dephasing across the voxel → it is crushed to zero, exactly like a
  spoiler.

So crushers selectively **null the contamination from the imperfect refocusing pulse** while
letting the proper echo through. The effect (slide images): with crushers the image is clean;
without them it is wrecked.

### Spoiler vs. crusher — the one-line distinction
| | Spoiler | Crusher |
|---|---|---|
| **Where** | single lobe at end of TR | balanced pair around a 180° pulse |
| **Kills** | leftover $M_+$ from the previous TR | spurious FID from an imperfect refocusing pulse |
| **Shared mechanism** | $2\pi n$ intra-voxel dephasing → voxel-averaged $M_+ \to 0$ | same |

Both "generate $2\pi n$ phase dispersion in a voxel." The difference is *which* unwanted signal
they target and *how* they're placed. Details (and the proper coherence-pathway / EPG treatment)
are again "MRI 2."

---

## 3. Contrast agents

### What they are
- Administered **intravenously**.
- In the clinic, **almost exclusively gadolinium-based**. Gd³⁺ is strongly paramagnetic (seven
  unpaired electrons), which makes it a powerful relaxation enhancer.
- **Free Gd³⁺ is very toxic**, so it is locked inside a **chelate**. Three families: *macrocyclic*,
  *linear ionic*, *linear non-ionic*. Chelated agents are generally quite safe.

### How they create contrast — the relaxivity model
Gd³⁺ **shortens relaxation times** of nearby water protons. This is an **indirect** effect: you
never see the agent itself, only its action on the tissue's $T_1$ and $T_2$. The observed
relaxation *rates* rise linearly with concentration $C$:

$$R_{1,\text{obs}} = R_{1,\text{intrinsic}} + r_1\, C, \qquad
R_{2,\text{obs}} = R_{2,\text{intrinsic}} + r_2\, C,$$

where $r_1, r_2$ are the **relaxivities** (units mM⁻¹ s⁻¹). Example values for **Gadovist in blood
@ 1.5 T**: $r_1 = 5.3\ \text{mM}^{-1}\text{s}^{-1}$, $r_2 = 5.4\ \text{mM}^{-1}\text{s}^{-1}$
(Rohrer et al., *Invest. Radiol.* 40:715–724, 2005).

### Why Gd is a *T₁* agent (the key intuition)
The relaxivities are almost equal ($r_1 \approx r_2$), yet the *clinical* effect is overwhelmingly
on $T_1$. The reason is the **starting point**. Worked example ($C = 0.5$ mM):

| | intrinsic rate | + $r\,C$ | observed time | relative change |
|---|---|---|---|---|
| $T_1$: 1000 ms | $R_1 = 1\ \text{s}^{-1}$ | $+2.65$ | $T_1 \to$ **274 ms** | ↓ to ~27% |
| $T_2$: 100 ms | $R_2 = 10\ \text{s}^{-1}$ | $+2.7$ | $T_2 \to$ **79 ms** | ↓ to ~79% |

The *same* added rate (~2.7 s⁻¹) is huge next to the small intrinsic $R_1$ but minor next to the
large intrinsic $R_2$. Hence **much stronger effect on $T_1$** → tissue with contrast agent gets
**bright on $T_1$-weighted images**. The notebook reproduces these exact numbers and the
relaxation-time-vs-concentration curves.

### Safety profile
- With normal kidney function, **>90 % is excreted in urine within 24 h**.
- **Transient reactions:** headache, nausea, dizziness, coldness at the injection site.
- **Allergy-like reactions:** ~**1 in 1,000** itchy skin minutes after injection (usually settles
  within an hour); ~**1 in 10,000** a severe reaction (patients usually respond well to standard
  emergency treatment).

### Nephrogenic systemic fibrosis (NSF)
- A **very severe** disease: fibrosis of skin, joints, eyes, organs. Timeline: first case **1997**,
  first publication **2000**, link to Gd agents established **2006**.
- Occurs in patients with **severe kidney failure**; in **end-stage** renal disease the risk is
  roughly **1 in 100 injections**.
- Risk depends on agent class: **lower for macrocyclic**, middle for linear ionic, **highest for
  linear non-ionic**.
- **Testing kidney function before injection prevents NSF** — this is why eGFR is checked.

### Gadolinium retention
- Up to **~1 %** of the injected dose is retained in tissue, **mostly in bone**; small amounts
  reach the **brain** (visible on $T_1$-weighted MRI).
- Retention is **much stronger for linear** agents → linear agents have been **largely withdrawn
  from the German market**; **macrocyclic agents remain available** (Saake et al., *Radiology* 290, 2019).

### Two clinical uses worth knowing
- **Oncology:** tumour vessels are often **leaky**, so the agent **extravasates** into the tumour and
  makes it **bright on contrast-enhanced (CE) $T_1$-weighted images**.
- **Subtraction images:** *post-contrast minus pre-contrast*. This cancels the static background and
  leaves only what enhanced — widely used, e.g. in breast MRI.

---

## 4. Basics of MRI safety

There is **no ionising radiation** in MRI. The hazards come from the three field types plus the
sheer strength of the static magnet. Crucially, the scanner continuously **monitors gradient and
RF safety limits**, so the dangerous regimes "do not occur in practice" — but the static field
respects no software limit, which is why it is the one that kills.

### Static main field $B_0$ — the dangerous one
- **Missile / projectile effect:** ferromagnetic objects are accelerated violently into the bore.
  The single most repeated rule of the lecture: **never enter the scanner room with a ferromagnetic
  object.** (The slides show real cases — chairs, tools, a floor scrubber — that flew into magnets.)
- **Moving through the field:** because $B_0$ is spatially non-uniform near the bore, a moving
  person experiences $dB/dt$, which **induces currents** in the body. Normally unnoticeable, but at
  **high field (e.g. 7 T)** induced currents in the **inner ear** can cause **vertigo/dizziness**.
  This is a **transient** effect.

### Gradient fields — switching
- Rapid gradient switching → $dB/dt$ → induced currents → **peripheral nerve stimulation (PNS)** →
  muscle twitching.
- **Cardiac stimulation** is possible only at *very* high switching, roughly a **factor ~10 above
  the PNS threshold** — and **does not occur in practice** because the scanner enforces limits.

### RF field $B_1^+$ — heating
- Some RF energy stays in the body as **heat**. Quantified by the **specific absorption rate (SAR)** =
  local power deposited per unit tissue mass (W/kg). **SAR limits are monitored by the scanner.**
  (RF power scales steeply with field strength and flip angle, which is part of why 7 T is harder —
  see the notebook.)

### Implants, signs, and the two emergency buttons
- **Prohibited:** anything ferromagnetic, **non-compatible implants** (pacemakers, defibrillators,
  some hearing aids/insulin pumps), open fire, clocks/cameras, magnetic fire extinguishers, loose
  metal, credit cards (data get wiped). **Warning signs:** strong B-field, high-frequency fields.
- **Magnet Stop (quench):** shuts down the **magnetic field** by quenching the superconducting
  magnet. Used **only** when a person must be freed and *only* a field shutdown can do it. Procedure:
  press the magnet-stop button → quench → **everyone leaves immediately** (boil-off helium can
  displace oxygen → **suffocation risk**) → make sure no one else enters. **Expensive and bad for the
  magnet.**
- **"Not-Aus MRT" (emergency power-off):** shuts down the **electricity** (gradients, RF, etc.) but
  **NOT the magnetic field**. Use it for a **burst water pipe** or **smoke from the technical room**.
  **Cheap, and the magnet is unaffected.**

> **The distinction to memorise:** Magnet Stop kills $B_0$ (quench, costly, last resort);
> Not-Aus kills the electronics but leaves $B_0$ — and the missile hazard — fully on.

---

## 5. Outlook: MRI 2 (not exam-relevant)

A teaser of where the story goes next, all under the banner of **speed and SNR**:

- **Parallel imaging** (SENSE/GRAPPA): use multiple receive coils' spatial sensitivity to skip
  k-space lines and reconstruct anyway → faster scans. *(This is Module 4!)*
- **Echo-planar imaging (EPI):** grab the whole of k-space after one excitation — very fast, but
  with characteristic artifacts (geometric distortion, "squeezed eyeballs").
- **Fast gradient echo:** e.g. real-time cardiac imaging.
- **Time-of-flight angiography:** image flowing blood without contrast.
- **Flow / phase-contrast measurements:** quantify velocities (e.g. aortic dissection).
- **Contrast agents & perfusion**, **ultra-high-field (7 T)** (ultra-high resolution opportunities;
  B₁ standing-wave artifacts as a challenge), **X-nuclei imaging** (e.g. ²³Na — metabolic info tied
  to the Na/K pump), **diffusion MRI** (white-matter tractography), and **functional MRI** (BOLD —
  brain activity).

You'll actually *build* several of these in Modules 4–6. This closes MRI 1.

---

## Key takeaways

1. **Spoiler and crusher gradients both work by creating $2\pi n$ phase dispersion across a voxel**,
   making the voxel-averaged transverse magnetization cancel. The residual signal is a **sinc** of
   the intra-voxel dephasing, with its first zero at one full $2\pi$ twist.
2. A **spoiler** is a single end-of-TR lobe that kills leftover $M_+$; a **crusher** is a balanced
   pair around a 180° pulse that kills the spurious FID from an imperfect slice profile while the
   true echo (net-zero moment) survives.
3. **Gd contrast agents shorten relaxation times** via a linear relaxivity law,
   $R_{i,\text{obs}} = R_{i,\text{int}} + r_i C$. Even with $r_1 \approx r_2$, the effect is mostly
   on $T_1$ because intrinsic $R_1 \ll R_2$ → **bright on $T_1$-weighted images**.
4. **Free Gd³⁺ is toxic → chelates**; the main risks are **NSF** (severe renal failure; prevented by
   checking kidney function; lowest for macrocyclic agents) and **Gd retention** (mostly bone; worse
   for linear agents, now largely withdrawn in Germany).
5. **MRI's worst hazard is the always-on static field** (missile effect). **Magnet Stop = quench the
   field** (last resort, expensive, suffocation risk); **Not-Aus = kill the electronics, keep the
   field.** Gradient (PNS) and RF (SAR) limits are software-monitored, so their danger regimes don't
   arise in normal practice.

---

## Self-test questions

1. Explain in one sentence *why* a gradient pulse can make the signal from a voxel vanish, even
   though it does not change the magnitude of any individual spin's magnetization.
2. A spoiler produces a total intra-voxel phase spread of $\Delta\varphi = \pi$ (half a turn). Is the
   voxel signal zero? Estimate $|S|$ relative to the un-spoiled case. (Hint: $\operatorname{sinc}$.)
3. Sketch the residual voxel signal as a function of the number of $2\pi$ twists. Where are the
   zeros, and why isn't the signal zero *between* them?
4. Why must a crusher gradient come as a **balanced pair around the 180° pulse**, rather than a
   single lobe? What would a single lobe do to the wanted echo?
5. Without crusher gradients, what specifically is the artifact-causing signal, and where in the
   slice does it originate?
6. Write down the relaxivity equations. Given $r_1 = 5.3$, $r_2 = 5.4$ mM⁻¹s⁻¹, intrinsic
   $T_1 = 1000$ ms, $T_2 = 100$ ms, compute $T_1$ and $T_2$ at $C = 0.5$ mM. (Target: 274 ms, 79 ms.)
7. Both relaxivities are nearly equal, yet gadolinium is called a "$T_1$ agent." Explain the apparent
   contradiction using the intrinsic rates.
8. A patient with end-stage renal disease needs contrast. Which agent class would you prefer and why,
   and what single test most reduces the risk of NSF?
9. Distinguish **Magnet Stop** from **Not-Aus MRT**: what does each shut down, when do you press each,
   and which carries a suffocation risk?
10. The scanner enforces gradient and RF safety limits in software, so PNS-to-cardiac escalation and
    SAR overheating "don't occur in practice." Name the one hazard that this software monitoring does
    **not** protect against, and state the rule that does.

*(Try these before opening the notebook — several map directly onto cells you can run and verify.)*
