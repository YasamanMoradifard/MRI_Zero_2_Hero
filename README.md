# MRI: Zero to Hero 

A complete, hands-on learning path through Magnetic Resonance Imaging — built from the **MRI1** and **Computational MRI (CMRI)** lecture series at **FAU Erlangen-Nürnberg**, extended with bonus material on AI for medical image segmentation and classification.

This repository takes you from the **physics of nuclear spins** all the way to **training neural networks to reconstruct, segment, and classify MRI scans**. Every concept is paired with a runnable Jupyter notebook, so you don't just *read* about MRI — you *build* it.

---

## Who This Is For

- Students learning MRI physics and reconstruction for the first time
- Engineers and ML practitioners moving into medical imaging
- Researchers wanting a structured refresher with practical code
- Anyone who's ever wondered *how does that brain scan actually get made?*

**Prerequisites:** Basic Python, basic linear algebra (matrices, vectors), basic calculus (derivatives, integrals). Everything else is built up from the ground up.

---

## The Six Modules

The repo is organized as a **progressive learning path**. Each module builds on the previous one — by the end, you'll understand the full pipeline from radiofrequency pulse to AI-assisted diagnosis.

### Module 1 — MRI Physics Fundamentals
*How does an MRI machine actually produce a signal?*

The big-picture intro to MRI, the three core principles (precession, excitation, relaxation), the dynamics of magnetization in a static field B₀, RF excitation, signal detection via Faraday induction, and how gradient fields make the signal spatially-dependent.

**Source:** MRI1 lectures 1–8

→ [`01_MRI_Physics_Fundamentals/`](./01_MRI_Physics_Fundamentals/)

### Module 2 — k-Space & Image Formation
*From signal to image: the Fourier connection.*

Fourier series and transforms (the math toolkit), how k-space encoding works, why MR images are reconstructed by FFT, Gibbs ringing and other Fourier artifacts, and slice selection. Then we extend into CMRI territory: how k-space is *actually* sampled and how Cartesian reconstruction works in practice.

**Source:** MRI1 lectures 9–11 + CMRI lectures 2–3

→ [`02_kSpace_and_Image_Formation/`](./02_kSpace_and_Image_Formation/)

### Module 3 — Contrast & Pulse Sequences
*Why does grey matter look different from white matter?*

Longitudinal (T1) relaxation, spin echo vs gradient echo sequences, spoiler gradients, and how sequence parameters (TR, TE, flip angle) shape image contrast. This is what turns physics into *useful diagnostic images*.

**Source:** MRI1 lectures 12–14

→ [`03_Contrast_and_Sequences/`](./03_Contrast_and_Sequences/)

### Module 4 — Advanced Reconstruction
*Real MRI is faster than the textbook — here's how.*

The reconstruction techniques that make modern MRI clinically practical: partial Fourier imaging, non-Cartesian reconstruction (radial, spiral), parallel imaging in image-space (SENSE) and k-space (GRAPPA), iterative reconstruction, and compressed sensing.

**Source:** CMRI lectures 4–9

→ [`04_Advanced_Reconstruction/`](./04_Advanced_Reconstruction/)

### Module 5 — AI for MR Reconstruction
*Deep learning enters the scanner.*

Machine learning and neural network fundamentals (CMRI's ML refresher), then how deep learning is replacing or augmenting classical reconstruction: unrolled networks, learned priors, and end-to-end image reconstruction from undersampled k-space.

**Source:** CMRI lectures 10–11

→ [`05_AI_for_Reconstruction/`](./05_AI_for_Reconstruction/)

### Module 6 — AI for Segmentation & Classification (Bonus Track)
*Beyond reconstruction: AI for diagnosis.*

Going beyond your courses to round out the "MRI hero" curriculum: U-Net architecture for medical segmentation, 3D segmentation with MONAI, brain tumor segmentation (BraTS), CNNs and transfer learning for disease classification, handling 3D volumes, and explainability with Grad-CAM.

**Source:** Curated material, public datasets

→ [`06_AI_for_Segmentation_Classification/`](./06_AI_for_Segmentation_Classification/)

---

## 📂 Repository Structure

```
MRI-Zero-to-Hero/
├── README.md                          ← you are here
├── ROADMAP.md                         ← detailed day-by-day study plan
├── requirements.txt
├── 01_MRI_Physics_Fundamentals/
│   ├── README.md                      module overview
│   ├── lectures/                      L01_summary.md, L02_summary.md, ...
│   ├── notebooks/                     L01_notebook.ipynb, ...
│   └── resources.md                   papers, books, links
├── 02_kSpace_and_Image_Formation/
├── 03_Contrast_and_Sequences/
├── 04_Advanced_Reconstruction/
├── 05_AI_for_Reconstruction/
├── 06_AI_for_Segmentation_Classification/
└── assets/                            shared images, diagrams, phantoms
```

Each lecture produces two files:
- **`L##_summary.md`** — structured notes with intuition, equations, key takeaways, and self-test questions
- **`L##_notebook.ipynb`** — runnable code that demonstrates the lecture's core concepts on synthetic data, with an optional "try it on real data" section using public datasets

---

## 🛠️ Setup

### Requirements

- Python 3.10+
- 8 GB RAM minimum (16+ GB recommended for Modules 5 and 6)
- GPU recommended for Modules 5 and 6 (Google Colab works great if you don't have one)

### Installation

```bash
git clone https://github.com/yourusername/MRI-Zero-to-Hero.git
cd MRI-Zero-to-Hero
python -m venv venv
source venv/bin/activate          # on Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Core dependencies

| Library          | Used in modules | Purpose                                |
|------------------|-----------------|----------------------------------------|
| `numpy`          | All             | Numerical computation                  |
| `scipy`          | 1, 2, 3, 4      | FFT, signal processing                 |
| `matplotlib`     | All             | Visualization                          |
| `nibabel`        | 2, 4, 5, 6      | Reading/writing NIfTI files            |
| `pydicom`        | 2, 4            | Reading DICOM files                    |
| `scikit-image`   | 2, 3, 4         | Image processing utilities             |
| `sigpy`          | 4, 5            | MRI reconstruction algorithms          |
| `pytorch`        | 5, 6            | Deep learning framework                |
| `monai`          | 5, 6            | Medical-imaging-specific DL utilities  |

---

## 📊 Datasets We'll Use

All datasets are **publicly available** and free for educational use. Download instructions are in each module.

| Dataset | Used in | What it is |
|---|---|---|
| **Synthetic phantoms** | Modules 1–4 | Built into notebooks (Shepp-Logan, etc.) — no download needed |
| **[fastMRI](https://fastmri.med.nyu.edu/)** | Modules 4, 5 | Raw k-space data from real scans |
| **[IXI](https://brain-development.org/ixi-dataset/)** | Modules 2, 3, 6 | Healthy brain T1/T2/PD scans |
| **[BraTS](http://braintumorsegmentation.org/)** | Module 6 | Brain tumor segmentation challenge |
| **[OASIS](https://www.oasis-brains.org/)** | Module 6 | Alzheimer's classification |

---

## Study Plan

This repo is designed to be worked through at **~1 hour/day**: read one lecture summary, run its notebook, do the self-test questions. At that pace, the full course takes about **6 weeks**.

→ See [`ROADMAP.md`](./ROADMAP.md) for the full day-by-day schedule.

---

## 📈 Progress Tracker

**Module 1 — MRI Physics Fundamentals** (8 lectures)
- [ ] L01 — Intro to MRI
- [ ] L02 — Basics of MRI Imaging
- [ ] L03 — Definitions
- [ ] L04 — The Three Principles
- [ ] L05 — Dynamics of Magnetization in B₀
- [ ] L06 — Excitation of M
- [ ] L07 — Signal Detection
- [ ] L08 — Gradients

**Module 2 — k-Space & Image Formation** (5 lectures)
- [ ] L09 — Fourier Series and Fourier Transforms
- [ ] L10 — k-Space Encoding
- [ ] L11 — Gibbs Ringing & Slice Selection
- [ ] C02 — Fourier Image Reconstruction Basics
- [ ] C03 — Partial Fourier Imaging

**Module 3 — Contrast & Pulse Sequences** (3 lectures)
- [ ] L12 — Longitudinal Relaxation
- [ ] L13 — Spin Echo vs Gradient Echo
- [ ] L14 — Spoiler Gradients

**Module 4 — Advanced Reconstruction** (6 lectures)
- [ ] C04 — Non-Cartesian Reconstruction
- [ ] C05 — Image-Space Parallel Imaging (SENSE)
- [ ] C06 — k-Space Parallel Imaging (GRAPPA)
- [ ] C07 — Non-Cartesian Parallel Imaging & Iterative Recon
- [ ] C08 — Compressed Sensing
- [ ] C09 — Recap / advanced topics

**Module 5 — AI for MR Reconstruction** (2 lectures)
- [ ] C10 — Intro to ML & Neural Networks
- [ ] C11 — ML for MR Image Reconstruction

**Module 6 — AI for Segmentation & Classification** (bonus)
- [ ] B01 — CNN Refresher for Medical Imaging
- [ ] B02 — U-Net Architecture
- [ ] B03 — Brain Tumor Segmentation with MONAI
- [ ] B04 — Classification CNNs for MRI
- [ ] B05 — 3D CNNs and Transfer Learning
- [ ] B06 — Explainability with Grad-CAM

---

## 📚 Recommended Reading

For deeper dives beyond the lectures:

- **Nishimura** — *Principles of Magnetic Resonance Imaging* (rigorous intro)
- **Bernstein, King, Zhou** — *Handbook of MRI Pulse Sequences* (the bible)
- **Liang & Lauterbur** — *Principles of Magnetic Resonance Imaging: A Signal Processing Perspective*
- **Brown, Cheng, Haacke, Thompson, Venkatesan** — *Magnetic Resonance Imaging: Physical Principles and Sequence Design*
- **Zhou et al.** — *Deep Learning for Medical Image Analysis*

---

## Contributing

This is a personal learning resource, but corrections and improvements are very welcome. Open an issue or PR.

---

## License

MIT License — feel free to use this material for learning, teaching, or building on top of.

---

## Acknowledgments

Built from the **MRI1** and **Computational MRI** lecture series at **Friedrich-Alexander-Universität Erlangen-Nürnberg (FAU)**. Deep gratitude to the instructors and to the open-source medical imaging community — MONAI, sigpy, fastMRI, BraTS organizers — for making world-class tools and data freely available.

---

*"From spinning protons to neural networks — one lecture at a time."*
