# C10 — Machine Learning for MR Image Reconstruction

> **Module 5 · AI for MR Reconstruction · Day 33**
> Source: CMRI Lecture 10 ("Machine learning for image reconstruction"), FAU Erlangen-Nürnberg
> Companion notebook: [`C10_notebook.ipynb`](../notebooks/C10_notebook.ipynb)

---

## 🎯 The one-sentence version

Classical reconstruction *hand-designs* a prior and then runs an optimizer; deep-learning reconstruction **learns the prior — or the entire optimizer — from data**, and now beats the classical methods badly enough to be shipping in commercial scanners.

---

## 🧠 Intuition

Everything in this lecture is a variation on a single optimization problem you already met in compressed sensing (C08):

$$\hat{u} = \arg\min_{u}\; \underbrace{\tfrac{\lambda}{2}\lVert Au - f\rVert_2^2}_{\text{data consistency}} \;+\; \underbrace{\mathcal{R}(u)}_{\text{prior / regularizer}}$$

- **$u$** is the image we want, **$f$** is the under-sampled, noisy k-space we measured.
- **$A$** is the *forward operator* — it turns an image into the measurement: coil sensitivities → Fourier transform → sampling mask. So $f = Au$ and (when fully sampled) $u = A^{-1}f$.
- The **data-consistency** term says: whatever you reconstruct must, when pushed back through the scanner physics, reproduce what we actually recorded.
- The **prior** $\mathcal{R}(u)$ says: among all images consistent with the data, prefer the *plausible* ones.

Compressed sensing picks $\mathcal{R}$ by hand (sparsity in wavelets, total variation). The leap in this lecture: **let the data tell us what a plausible image looks like.** That single idea splits into three engineering choices about *where you bolt the network onto the pipeline.*

---

## 🔑 Core concepts

### 1. The forward operator and why the adjoint matters

For 2-D Cartesian MRI the signal equation is

$$f(k_x,k_y) = \iint c_k(x,y)\,u(x,y)\,e^{-i(k_x x + k_y y)}\,dx\,dy \;\;\Longleftrightarrow\;\; f = Au.$$

With a centered, *unitary* FFT the adjoint equals the inverse, $A^{H} = A^{-1}$, on fully-sampled data. Every iterative and unrolled method below uses $A^{H}(Au - f)$ as its **data-consistency gradient** — it is the physics, not something you learn. Under-sampling replaces $A = F$ with $A = MF$ (a binary mask $M$), which is what causes aliasing:

- **Regular** under-sampling → **coherent** aliasing (clean folded copies, the FOV/R wraparound). This is the regime SENSE/GRAPPA were built for.
- **Variable-density random** under-sampling → **incoherent**, noise-like artifacts. This is the regime compressed sensing and most learned methods assume.

### 2. The three deep-learning approaches

| # | Approach | Network input → output | Keeps physics in the loop? | Examples |
|---|----------|------------------------|----------------------------|----------|
| **1** | **Image-domain post-processing** | aliased image → clean image | ❌ (only at the end, if at all) | FBPConvNet (Jin 2017); Gibbs/de-aliasing U-Net (Muckley MRM 2020) |
| **2** | **k-space recovery** | gappy k-space → full k-space | ✔️ implicitly (works on raw data) | RAKI (Akçakaya 2019, scan-specific); AUTOMAP (Zhu 2018) |
| **3** | **Unrolled iterative reconstruction** | k-space + $A$ → image, *inside* the optimizer | ✔️ at **every** step | Variational Network (Hammernik MRM 2018) |

**Approach 1** is the easiest to train (it is just image-to-image learning) but it can drift away from the measured data because it never re-checks the physics. A cheap fix is a **data-consistency layer**: after the CNN, in k-space, keep the measured lines and only trust the network for the gaps — $\hat k = (1-M)F u_{\text{cnn}} + M f$. That little layer is the seed of Approach 3.

**Approach 2** completes the missing k-space lines directly. GRAPPA (C06) is the *linear* version; RAKI makes it nonlinear and **scan-specific** (trained on the scan's own ACS region, so no training-database bias); AUTOMAP learns the whole k-space→image map.

**Approach 3** is the heart of the lecture and the consistent winner.

### 3. Unrolling = "learning gradient descent"

Take the CS gradient-descent (Landweber) step:

$$u_t = u_{t-1} - \frac{\partial}{\partial u}\Big(\tfrac{\lambda}{2}\lVert Au_{t-1}-f\rVert_2^2 + \mathcal{R}(u_{t-1})\Big).$$

Now replace the hand-crafted regularizer with **learned filters** $K_i$ and **learned nonlinearities** $\rho'_i$, and let the step sizes $\lambda_t$ be learnable per stage:

$$\boxed{\,u_t = u_{t-1} - \sum_{i=1}^{N} K_{i,t}^{\top}\,\rho'_{i,t}\!\big(K_{i,t}\,u_{t-1}\big) - \lambda_t\,A^{H}\!\big(A u_{t-1}-f\big)\,}$$

The first sum is the **learned regularization gradient**; the last term is the **data-consistency gradient** (pure physics). Stack $T$ of these steps, train end-to-end against a reference, and you have Hammernik's **variational network (VN)**. Reading the slide dictionary:

- *sparsifying transform* → learned **spatial filter kernels** $K_i$
- *L1 norm* → learned **potential functions** $\rho_i$
- *fixed iterations* → **T unrolled, trained stages**

The supervised training loss is just

$$\mathcal{L}(\Theta) = \tfrac{1}{S}\sum_{s=1}^{S}\big\lVert u_s^{T}(\Theta) - u_{\text{ref},s}\big\rVert_2^2.$$

### 4. Beyond supervised learning

Real clinical data rarely comes with fully-sampled references, so the field developed alternatives:

- **Self-supervised:** **SSDU** (Yaman 2020) splits the *acquired* k-space into a set to fit and a held-out set to validate — training from under-sampled data alone. Relatives: Noise2Noise / Noise2Self / Noise2Void / Noise2Recon.
- **No-database, single-scan:** **Deep Image Prior** (Ulyanov) and **zero-shot** self-supervised learning fit one scan with no training set at all.
- **Generative priors:** **GANs** (cyclic loss), **score-based diffusion models** (Chung & Ye), **plug-and-play** (drop a pretrained denoiser into the CS iteration).
- **Architectures:** cascaded CNNs (Schlemper), **transformers** for fast MRI.
- **Safety:** **Bayesian VN** (Narnhofer TMI 2021) emits a per-pixel **uncertainty (std) map** — a direct antidote to silent hallucinations.

### 5. ⚠️ The failure modes (the most important slide)

Learned reconstruction is not a free lunch:

- **Hallucinations** (Muckley TMI 2021): the network can **invent** structure that was not measured, or **erase** a real finding — the slide's blunt warning, *"acceleration was too high → a disease was removed."*
- **Out-of-distribution collapse** (Radmanesh 2022): a network trained at $R=4$ degrades sharply on $R=8$ or on anatomy/scanners it never saw. Detail vanishes faster than for honest zero-filling.
- **Metrics hide it:** global NRMSE/PSNR/SSIM average over the whole image, so a mangled 3-pixel lesion can be invisible in the score while being catastrophic for the patient.

Mitigations: data-consistency layers, uncertainty maps, conservative acceleration limits, and — non-negotiably — **clinical reader studies**.

---

## 📐 Equations to remember

| Concept | Equation |
|---------|----------|
| Forward model | $f = Au$, $\;A = MF\,\mathrm{diag}(c_k)$ |
| Reconstruction objective | $\min_u \tfrac{\lambda}{2}\lVert Au-f\rVert_2^2 + \mathcal{R}(u)$ |
| CS gradient-descent step | $u_t = u_{t-1} - \partial_u\big(\tfrac{\lambda}{2}\lVert Au_{t-1}-f\rVert_2^2 + \mathcal{R}(u_{t-1})\big)$ |
| Learned (variational-net) step | $u_t = u_{t-1} - \sum_i K_{i,t}^{\top}\rho'_{i,t}(K_{i,t}u_{t-1}) - \lambda_t A^{H}(Au_{t-1}-f)$ |
| Hard data-consistency layer | $\hat k = (1-M)Fu_{\text{cnn}} + Mf$ |
| Training loss | $\mathcal{L}(\Theta)=\tfrac{1}{S}\sum_s \lVert u_s^T(\Theta)-u_{\text{ref},s}\rVert_2^2$ |

---

## 📊 Datasets & clinical translation

- **fastMRI** (NYU + Meta AI): fully-sampled raw k-space — ~1,398 knee and ~7,002 brain cases, with GE/Philips transfer tracks. The standard benchmark; ships PyTorch dataloaders. → your "try it on real data" target.
- **Reconstruction challenges** (Knoll MRM 2020; Muckley TMI 2021): ranked on **SSIM** at $R=4$/$R=8$; surfaced the hallucination problem.
- **First prospective clinical study** (Johnson Radiology 2023): learned recon on 300 prospectively under-sampled knee exams; blinded readers agreed with conventional reads at high rates across cartilage/ligament/meniscus/marrow findings — evidence it is clinically viable *within its trained acceleration range*.
- **Shipping products:** Siemens **Deep Resolve**, Philips **SmartSpeed**, GE **AIR Recon DL**, plus startups (AIRS Medical SwiftMR, Subtle Medical SubtleMR).

---

## ✅ Key takeaways

1. **One equation rules everything:** $\min_u \tfrac{\lambda}{2}\lVert Au-f\rVert_2^2 + \mathcal{R}(u)$. Classical CS hand-designs $\mathcal{R}$; deep learning learns it, or learns the whole solver.
2. **Three places to put the network** — image domain, k-space, or inside the optimizer. Keeping the physics ($A$) in the loop (Approach 3, the variational network) consistently wins.
3. **An unrolled network is literally learned gradient descent** — finitely many GD steps with learned regularizers and step sizes, trained end-to-end.
4. **It can lie.** Hallucinations and out-of-distribution collapse are real, and global metrics can hide them.
5. **Mitigations are part of the method:** data-consistency layers, self-supervision (SSDU), uncertainty estimation, reader studies.
6. **This is deployed technology**, not a research curiosity.

---

## 📝 Self-test questions

1. Why is the **adjoint** $A^{H}$ (rather than another learned layer) used for the data-consistency gradient in a variational network?
2. What artifact pattern distinguishes **regular** from **variable-density** under-sampling, and why does each suit a different family of reconstruction methods?
3. In the VN update, which term enforces fidelity to the scan and which encodes the learned prior? What is $\lambda_t$, and why make it learnable per stage?
4. Give one concrete clinical scenario where a hallucinated reconstruction could cause harm — and one safeguard that would catch it.
5. How does **SSDU** train a reconstruction network *without any fully-sampled reference*?
6. You trained at $R=4$ but the clinic wants $R=8$ for speed. Predict what happens, and name two things you would do before deploying.
7. Approach 1 (post-processing) and Approach 3 (unrolled) can use the *same* U-Net inside them. What is the essential difference that makes Approach 3 more robust?

*(Worked answers to the coding versions of Q1–Q6 are in the notebook's exercise-solution cells.)*

---

## 🔗 Connections

- **Builds on:** C08 Compressed Sensing (the objective + sparsity prior), C05/C06 SENSE/GRAPPA (the operator $A$, coil sensitivities, ACS), C02 Fourier reconstruction (the FFT pair).
- **Leads to:** Day 36 — go deeper on the unrolled family (a MoDL-style block with a shared denoiser and a conjugate-gradient data-consistency layer), and Module 6's segmentation/classification networks, which reuse the same CNN/U-Net machinery for diagnosis rather than reconstruction.

---

## 📚 References (from the slides)

- Pruessmann et al., *SENSE* (MRM 1999) and *arbitrary-trajectory SENSE* (MRM 2001); Sodickson & Manning, *SMASH* (1997); Griswold et al., *GRAPPA* (MRM 2002).
- Lustig, Donoho & Pauly, *Sparse MRI* (MRM 2007).
- Jin et al., *Deep CNN for Inverse Problems in Imaging* (IEEE TIP 2017).
- Hammernik et al., *Learning a Variational Network for Reconstruction* (ISMRM 2016; MRM 2018).
- Zhu et al., *AUTOMAP* (2018); Akçakaya et al., *RAKI* (MRM 2019).
- Yaman et al., *SSDU* (2020) and *Zero-Shot Self-Supervised Learning*; Ulyanov et al., *Deep Image Prior*.
- Knoll et al., *fastMRI challenge* (MRM 2020); Muckley et al., *2020 challenge / hallucinations* (IEEE TMI 2021).
- Radmanesh et al., *Performance at progressive acceleration* (Radiology AI 2022); Narnhofer et al., *Bayesian VN* (TMI 2021); Johnson et al., *clinical validation* (Radiology 2023); Vornehm et al., *real-time cardiac cine* (ISMRM 2023).
