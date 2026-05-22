# Module 1 — MRI Physics Fundamentals

> **Days 1–10 of the ROADMAP**
> *How does an MRI machine actually produce a signal?*

This module covers the physics that gets us from a static magnetic field B₀ to a measurable, spatially-localizable MR signal. It's the foundation everything else builds on — k-space, contrast, AI reconstruction, all of it assumes you understand what's actually happening to the nuclear spins.

## Lectures in this module

| Day | Lecture | File | Status |
|-----|---------|------|--------|
| 1 | MRI1 L01 — Intro to MRI | `L01_*` | Done |
| 2 | MRI1 L02 — Basics of MRI Imaging | `L02_*` | Done |
| 3 | MRI1 L03 — Definitions | `L03_*` | Done |
| 4 | MRI1 L04 — The Three Principles | `L04_*` | Done |
| 5 | *Practice & recap day — no new lecture* | — | Done |
| 6 | MRI1 L05 — Dynamics of Magnetization in B₀ | `L05_*` | Done |
| 7 | MRI1 L06 — Excitation of M | `L06_*` | Done |
| 8 | MRI1 L07 — Signal Detection | `L07_*` | Done |
| 9 | MRI1 L08 — Gradients | `L08_*` | Done |
| 10 | *Module review — no new lecture* | — | Done |

## What's in each file

For every lecture you have:

- **`L##_summary.md`** — a structured study companion with derivations, intuition, key takeaways, and 10 self-test questions
- **`L##_notebook.ipynb`** — a runnable Jupyter notebook that demonstrates every concept on synthetic data, including any relevant exercises from that lecture's exercise sheet

## How to use this module

1. Read the summary first, mathematical derivations and all.
2. Open the notebook and **run it cell-by-cell**. Don't just read — *execute*.
3. Answer the self-test questions (no peeking).
4. **Modify the notebook**. Change the field strength, the flip angle, the gyromagnetic ratio. Break things and fix them — that's where the learning happens.
5. If you get stuck on anything, copy the relevant slide or code snippet to Claude and ask.

## What you should be able to do by the end of this module

By Day 10 you should be able to:

- Compute the Larmor frequency for any nucleus at any field strength
- Explain precession, excitation, and relaxation in one paragraph each
- Numerically solve the Bloch equations in the rotating frame
- Design an RF pulse that produces a specified flip angle
- Simulate a free induction decay (FID) and compute its spectrum
- Explain why a linear gradient creates a position-dependent Larmor frequency — and why that's the foundation of all spatial encoding

By the end you'll have built (in code) the entire MR experiment from B₀ to detected signal, before a single FFT.

