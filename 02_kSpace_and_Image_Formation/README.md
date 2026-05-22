# Module 2 — k-Space & Image Formation

> **Days 11–17 of the ROADMAP**
> *From signal to image: the Fourier connection.*

This module covers how a 1D time-domain MR signal becomes a 2D/3D image via the Fourier transform — and what goes wrong when we sample k-space imperfectly.

## Lectures in this module

| Day | Lecture | File | Status |
|-----|---------|------|--------|
| 11 | MRI1 L09 — Fourier Series & Transforms | `L09_*` | ✅ |
| 12 | **MRI1 L10 — k-Space Encoding, Resolution, Gibbs & Aliasing** | **`L10_*`** | ✅ |
| 13 | MRI1 L11 — Gibbs Ringing & Slice Selection | `L11_*` | ✅ |
| 14 | CMRI L02 — Fourier Image Reconstruction Basics | `C02_*` | ✅ |
| 16 | CMRI L03 — Partial Fourier Imaging | `C03_*` | ✅ |

## What's in each file

For every lecture you have:

- **`L##_summary.md`** — a structured study companion with derivations, intuition, key takeaways, and 10 self-test questions
- **`L##_notebook.ipynb`** — a runnable Jupyter notebook that demonstrates every concept on synthetic data, including any relevant exercises from that lecture's exercise sheet

## How to use this module

1. Read the summary first, mathematica derivations and all.
2. Open the notebook and **run it cell-by-cell**. Don't just read — *execute*.
3. Answer the self-test questions (no peeking).
4. **Modify the notebook**. Change `k_max`, change the undersampling factor, swap the phantom. Break things and fix them — that's where the learning happens.
5. If you get stuck on anything, copy the relevant slide or code snippet to Claude and ask.

## What you should be able to do by the end of this module

By Day 17 you should be able to:

- Implement a 2D FFT-based image reconstruction from scratch
- Predict and recognize Gibbs ringing in real MR images
- Predict and recognize wrap-around artifacts
- Reconstruct an image from partially-acquired k-space (zero-fill + POCS)
- Explain why the read direction never aliases

