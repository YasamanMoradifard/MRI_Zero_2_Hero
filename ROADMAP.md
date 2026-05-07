# 🗺️ ROADMAP — Day-by-Day Study Plan

A structured ~6-week learning path through MRI: Zero to Hero, designed for **~1 hour/day**.

This roadmap tells you exactly **what to study each day**, **what to upload to Claude that day**, and **what you'll be able to do** by the end of each module.

---

## 📅 At a Glance

| Phase | Duration | Lectures | What You'll Master |
|-------|----------|----------|--------------------|
| **Module 1** — Physics | Days 1–10 | MRI1 L1–L8 | The physics of MR signal generation |
| **Module 2** — k-Space | Days 11–17 | MRI1 L9–L11 + CMRI 2–3 | How signals become images |
| **Module 3** — Sequences | Days 18–22 | MRI1 L12–L14 | How contrast is created |
| **Module 4** — Recon | Days 23–32 | CMRI 4–9 | Modern accelerated reconstruction |
| **Module 5** — AI Recon | Days 33–37 | CMRI 10–11 | Deep learning for reconstruction |
| **Module 6** — AI Diagnosis | Days 38–45 | Bonus | Segmentation & classification |

**Total:** ~45 days of ~1 hour each. Take rest days as needed — consistency matters more than speed.

---

## 🔁 The Daily Rhythm

Each day follows the same pattern:

1. **Upload** the day's lecture PDF here (5 min)
2. **Read** the `L##_summary.md` Claude produces (10–15 min)
3. **Run** the `L##_notebook.ipynb` cell by cell (20–30 min)
4. **Practice** by modifying the notebook or answering the self-test questions (10–15 min)
5. **Ask** Claude follow-up questions where you're stuck

**Skip days are fine.** This is a learning marathon, not a sprint.

---

## 📘 MODULE 1 — MRI Physics Fundamentals (Days 1–10)

> **Goal:** Understand how an MRI machine generates a spatially-encoded signal.

### Day 1 — Intro to MRI
- **Upload:** MRI1 Lecture 1 (Intro)
- **Notebook focus:** Set up environment, plot a spinning vector, visualize a magnetic field
- **You will be able to:** Explain why MRI exists and what it competes with (CT, ultrasound)

### Day 2 — Basics of MRI Imaging
- **Upload:** MRI1 Lecture 2 (Basics of MRI Imaging)
- **Notebook focus:** Load and display a real MRI image, explore voxel intensities
- **You will be able to:** Describe the high-level components of an MRI scanner

### Day 3 — Definitions
- **Upload:** MRI1 Lecture 3 (Definitions)
- **Notebook focus:** Implement nuclear spin, gyromagnetic ratio, Larmor frequency calculator
- **You will be able to:** Compute the Larmor frequency for any nucleus at any field strength

### Day 4 — The Three Principles
- **Upload:** MRI1 Lecture 4 (The Three Principles)
- **Notebook focus:** Animate the three principles (precession, excitation, relaxation)
- **You will be able to:** Explain the full MR experiment in one paragraph

### Day 5 — Practice & Recap Day 🌿
- **No new upload** — review L1–L4 summaries, redo notebooks from scratch
- **Self-test:** Answer all questions from the four summaries

### Day 6 — Dynamics of M in B₀
- **Upload:** MRI1 Lecture 5 (Dynamics of Magnetization in B₀)
- **Notebook focus:** Solve and animate the Bloch equations in the rotating frame
- **You will be able to:** Numerically simulate magnetization precessing in any B-field

### Day 7 — Excitation of M
- **Upload:** MRI1 Lecture 6 (Excitation of M)
- **Notebook focus:** Simulate RF excitation, plot flip angle vs pulse duration
- **You will be able to:** Design an RF pulse that produces a desired flip angle

### Day 8 — Signal Detection
- **Upload:** MRI1 Lecture 7 (Signal Detection)
- **Notebook focus:** Simulate FID signal, compute its spectrum
- **You will be able to:** Explain how a coil "hears" magnetization via Faraday induction

### Day 9 — Gradients
- **Upload:** MRI1 Lecture 8 (Gradients)
- **Notebook focus:** Visualize how a gradient creates a position-dependent Larmor frequency
- **You will be able to:** Explain *why* gradients let us spatially localize MR signal

### Day 10 — Module 1 Review 🎯
- **No upload** — write a 1-page summary of the entire module in your own words
- **Mini-project:** Build a single notebook that walks through the full MR experiment from scratch
- **Milestone:** *You now understand MR physics from B₀ to detected signal.*

---

## 📗 MODULE 2 — k-Space & Image Formation (Days 11–17)

> **Goal:** Understand how a 1D time signal becomes a 2D/3D image via Fourier transforms.

### Day 11 — Fourier Series and Transforms
- **Upload:** MRI1 Lecture 9 (Fourier Series and FT)
- **Notebook focus:** Build intuition for FT — square wave decomposition, 2D FFT of images
- **You will be able to:** Explain the Fourier transform without using the words "frequency domain"

### Day 12 — k-Space Encoding
- **Upload:** MRI1 Lecture 10 (k-Space Encoding)
- **Notebook focus:** Implement the MR signal equation S(k) = ∫ρ(x)e^(-i2πkx)dx
- **You will be able to:** Explain why k-space *is* the Fourier transform of the image

### Day 13 — Gibbs Ringing & Slice Selection
- **Upload:** MRI1 Lecture 11 (Gibbs Ringing, Slice Selection)
- **Notebook focus:** Reproduce Gibbs ringing artifacts; design a slice-selective sinc pulse
- **You will be able to:** Recognize and explain Gibbs ringing in real MR images

### Day 14 — Fourier Image Reconstruction Basics
- **Upload:** CMRI Lecture 2 (Fourier Image Reconstruction Basics)
- **Notebook focus:** Reconstruct an image from synthetic k-space, examine FFT properties
- **You will be able to:** Reconstruct an image from raw Cartesian k-space data

### Day 15 — Practice & Recap Day 🌿
- **No upload** — replay the whole image-formation chain in one notebook

### Day 16 — Partial Fourier Imaging
- **Upload:** CMRI Lecture 3 (Partial Fourier Imaging)
- **Notebook focus:** Implement zero-filling and POCS partial-Fourier reconstruction
- **You will be able to:** Reconstruct an image from a half-acquired k-space

### Day 17 — Module 2 Review 🎯
- **No upload** — explain k-space to a rubber duck (or write a tutorial)
- **Mini-project:** Take an image, transform to k-space, undersample it, reconstruct
- **Milestone:** *You can reconstruct MR images from raw k-space data.*

---

## 📙 MODULE 3 — Contrast & Pulse Sequences (Days 18–22)

> **Goal:** Understand how to make grey matter look different from white matter.

### Day 18 — Longitudinal Relaxation
- **Upload:** MRI1 Lecture 12 (Longitudinal Relaxation)
- **Notebook focus:** Simulate T1 recovery curves for different tissue types
- **You will be able to:** Predict signal intensity given TR, T1, and flip angle

### Day 19 — Spin Echo vs Gradient Echo
- **Upload:** MRI1 Lecture 13 (Spin Echo vs Gradient Echo)
- **Notebook focus:** Simulate both sequences and visualize signal evolution
- **You will be able to:** Choose between SE and GRE for a given clinical question

### Day 20 — Spoiler Gradients
- **Upload:** MRI1 Lecture 14 (Spoiler Gradients)
- **Notebook focus:** Show how spoilers destroy residual transverse magnetization
- **You will be able to:** Explain why spoilers are needed in steady-state sequences

### Day 21 — Practice Day 🌿
- **Mini-project:** Implement a sequence simulator that takes (TR, TE, FA, T1, T2) → signal

### Day 22 — Module 3 Review 🎯
- **Milestone:** *You can predict MR contrast for any tissue and any sequence parameters.*

---

## 📕 MODULE 4 — Advanced Reconstruction (Days 23–32)

> **Goal:** Understand the modern reconstruction techniques that make clinical MRI fast.

### Day 23 — Non-Cartesian Reconstruction
- **Upload:** CMRI Lecture 4 (Non-Cartesian Reconstruction)
- **Notebook focus:** Reconstruct from radial and spiral trajectories using gridding/NUFFT
- **You will be able to:** Reconstruct from arbitrary k-space sampling patterns

### Day 24 — Image-Space Parallel Imaging (SENSE)
- **Upload:** CMRI Lecture 5 (Image Space Parallel Imaging)
- **Notebook focus:** Implement SENSE from scratch with simulated multi-coil data
- **You will be able to:** Reconstruct an aliased image using coil sensitivities

### Day 25 — Practice Day 🌿
- **Apply SENSE on a more realistic dataset**

### Day 26 — k-Space Parallel Imaging (GRAPPA)
- **Upload:** CMRI Lecture 6 (k-Space Based Parallel Imaging)
- **Notebook focus:** Implement GRAPPA kernel estimation and missing-line synthesis
- **You will be able to:** Compare SENSE vs GRAPPA on the same undersampled data

### Day 27 — Non-Cartesian Parallel Imaging & Iterative Recon
- **Upload:** CMRI Lecture 7 (Non-Cartesian PI & Iterative Recon)
- **Notebook focus:** Set up iterative SENSE with conjugate gradients
- **You will be able to:** Solve MR reconstruction as a linear inverse problem

### Day 28 — Practice Day 🌿
- **Try iterative recon on real fastMRI data**

### Day 29 — Compressed Sensing (Part 1)
- **Upload:** CMRI Lecture 8 (Compressed Sensing — first half)
- **Notebook focus:** Sparsity in wavelet domain, L1 minimization with synthetic signals
- **You will be able to:** Explain why incoherent sampling + sparsity = recoverable signal

### Day 30 — Compressed Sensing (Part 2)
- **Upload:** CMRI Lecture 8 continued (or 9 if separate)
- **Notebook focus:** Apply CS to MRI — variable-density sampling + L1-wavelet recon
- **You will be able to:** Reconstruct an MR image from highly undersampled k-space

### Day 31 — CMRI Lecture 9 (recap or advanced)
- **Upload:** CMRI Lecture 9 (whatever it covers)
- **Notebook focus:** Determined after reading the lecture

### Day 32 — Module 4 Review 🎯
- **Mini-project:** Take a fastMRI scan, undersample 4×, reconstruct it three ways (zero-fill, SENSE, CS), compare quality
- **Milestone:** *You can implement modern MRI reconstruction algorithms.*

---

## 📔 MODULE 5 — AI for MR Reconstruction (Days 33–37)

> **Goal:** Replace classical recon with deep learning.

### Day 33 — Intro to ML & Neural Networks
- **Upload:** CMRI Lecture 10 (Intro to ML and NNs)
- **Notebook focus:** Train a tiny MLP, then a simple CNN, on toy image data
- **You will be able to:** Explain what gradient descent and backpropagation actually do

### Day 34 — Practice Day 🌿
- **Train a denoising autoencoder on simulated noisy MR images**

### Day 35 — ML for MR Image Reconstruction (Part 1)
- **Upload:** CMRI Lecture 11 (ML for MR Reconstruction)
- **Notebook focus:** Train a U-Net to reconstruct undersampled k-space (image-domain)
- **You will be able to:** Explain how learned priors replace handcrafted regularization

### Day 36 — ML for MR Reconstruction (Part 2)
- **No new upload** — go deeper: implement an unrolled network (e.g., one block of MoDL or Variational Network)
- **You will be able to:** Build a hybrid model that combines physics and learning

### Day 37 — Module 5 Review 🎯
- **Mini-project:** Compare CS reconstruction vs neural network reconstruction on the same fastMRI data
- **Milestone:** *You can train a neural network to reconstruct MRI from undersampled k-space.*

---

## 🎓 MODULE 6 — AI for Segmentation & Classification (Days 38–45)

> **Goal:** Use AI not for reconstruction, but for diagnosis.

### Day 38 — CNN Refresher for Medical Imaging
- **No upload needed** — Claude provides bonus material
- **Notebook focus:** CNN architecture review with focus on medical-imaging quirks (3D, intensity range, anisotropic voxels)

### Day 39 — U-Net Architecture
- **No upload needed** — bonus material
- **Notebook focus:** Implement a U-Net from scratch in PyTorch

### Day 40 — Practice Day 🌿
- **Train your U-Net on a tiny synthetic segmentation task**

### Day 41 — Brain Tumor Segmentation with MONAI (Part 1)
- **No upload needed** — bonus material
- **Notebook focus:** Set up MONAI, load BraTS data, build the data pipeline
- **You will be able to:** Use industry-standard tooling for medical image segmentation

### Day 42 — Brain Tumor Segmentation with MONAI (Part 2)
- **Notebook focus:** Train and evaluate a 3D U-Net on BraTS

### Day 43 — Classification CNNs for MRI
- **No upload needed** — bonus material
- **Notebook focus:** Train a CNN classifier on OASIS for Alzheimer's classification
- **You will be able to:** Build a clinical classification pipeline

### Day 44 — 3D CNNs, Transfer Learning, and Explainability
- **Notebook focus:** Add transfer learning and Grad-CAM visualizations

### Day 45 — Capstone 🏆
- **Capstone project:** End-to-end pipeline — undersampled k-space → reconstructed image → segmentation → classification
- **Milestone:** *You are now an MRI hero.* 🎉

---

## 💡 Tips for Success

- **Type out the code yourself** instead of copy-pasting. Muscle memory matters.
- **When stuck, ask Claude.** Paste the error or the slide you don't understand.
- **Take rest days.** This roadmap has them built in. Use them.
- **Build the GitHub repo as you go.** Commit each day. Future-you will thank present-you.
- **Modify the notebooks.** Change parameters, break things, fix them. That's where learning happens.
- **Re-explain to someone (or to a rubber duck).** If you can't explain it, you don't understand it yet.

---

## 📤 What to Upload, When

Here's a quick reference of *exactly* what to upload on which day:

| Day | Upload | Day | Upload |
|-----|--------|-----|--------|
| 1 | MRI1 L1 | 23 | CMRI L4 |
| 2 | MRI1 L2 | 24 | CMRI L5 |
| 3 | MRI1 L3 | 25 | (practice) |
| 4 | MRI1 L4 | 26 | CMRI L6 |
| 5 | (practice) | 27 | CMRI L7 |
| 6 | MRI1 L5 | 28 | (practice) |
| 7 | MRI1 L6 | 29 | CMRI L8 |
| 8 | MRI1 L7 | 30 | CMRI L8 (cont.) |
| 9 | MRI1 L8 | 31 | CMRI L9 |
| 10 | (review) | 32 | (review) |
| 11 | MRI1 L9 | 33 | CMRI L10 |
| 12 | MRI1 L10 | 34 | (practice) |
| 13 | MRI1 L11 | 35 | CMRI L11 |
| 14 | CMRI L2 | 36 | (advanced) |
| 15 | (practice) | 37 | (review) |
| 16 | CMRI L3 | 38–45 | (bonus material) |
| 17 | (review) | | |
| 18 | MRI1 L12 | | |
| 19 | MRI1 L13 | | |
| 20 | MRI1 L14 | | |
| 21 | (practice) | | |
| 22 | (review) | | |

---

## 🚀 Ready?

**Day 1 is upload day.** When you're ready, send me the **MRI1 Lecture 1 (Intro) PDF**, and I'll produce your first `L01_summary.md` and `L01_notebook.ipynb`.

Let's begin! 🧲
