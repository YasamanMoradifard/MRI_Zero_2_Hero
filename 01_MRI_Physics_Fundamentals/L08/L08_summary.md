# L08 — Fourier Series and Fourier Transform

> **Module 2 — k-Space & Image Formation · Day 11 of the roadmap**
> *Source: MRI1 Lecture, Chapter 9 "Fourier Series and Fourier Transform" (PDF labelled `L08.pdf`)*

---

## 🎯 Why this lecture matters

The previous lecture taught us that gradients turn position into frequency. Now we need the mathematical machinery that turns *a collection of frequencies* back into *an image*. That machinery is the **Fourier transform**.

The central claim of this lecture is breathtakingly simple: **any well-behaved function can be written as a weighted sum of sine and cosine waves**. In MRI, this lets us say:

> *"The MR signal we acquire is the Fourier transform of the image."*

Once you internalize that one sentence, the entire pipeline — k-space sampling, FFT reconstruction, undersampling artifacts, partial Fourier, parallel imaging — opens up in front of you. This lecture builds that sentence from scratch.

---

## 1. Fourier series — the discrete version

### 1.1 The core claim

For a (well-behaved) function $f(x)$ defined on an interval $x \in [-L/2,\, L/2]$:

$$
f(x) \;=\; \sum_{n=0}^{\infty}\left[ a_n \sin\!\frac{2\pi n x}{L} + b_n \cos\!\frac{2\pi n x}{L} \right]
$$

for the right choice of coefficients $a_n, b_n$. Each basis function is a **whole wave** on $[-L/2, L/2]$ — that's what the $2\pi n / L$ factor guarantees. A 19th-century proof of convergence took serious work; we just accept the result.

In 2D / 3D, the same idea: a 2D function is a sum of 2D sines and cosines along $x$ and $y$.

A useful technicality: $a_0$ is undefined (since $\sin 0 = 0$); we set $a_0 = 0$. The $n = 0$ cosine equals 1 — it's the only **non-oscillatory** (DC) term.

### 1.2 Exponentials instead of sines and cosines

Using Euler's formula $e^{i\phi} = \cos\phi + i\sin\phi$, the real Fourier series can be rewritten with a single complex coefficient per frequency:

$$
\boxed{\,f(x) = \sum_{n=-\infty}^{\infty} c_n\, e^{i 2\pi n x / L}\,}
$$

The trick: combining $a_n\sin + b_n\cos$ gives one positive-frequency exponential ($c_n$) and one negative-frequency exponential ($c_{-n}$). Hence the sum runs over **all** integers, positive and negative. This complex form is what everybody uses in practice.

### 1.3 Finding the coefficients — the orthogonality recipe

The basis functions are **orthogonal** over $[-L/2, L/2]$:

$$
\int_{-L/2}^{L/2} e^{i 2\pi (n-m) x / L}\, dx = L \cdot \delta_{n,m}
$$

where $\delta_{n,m}$ is the Kronecker delta. This is the magic. Multiply both sides of the series by $e^{-i 2\pi n x / L}$ and integrate — every term in the sum dies except the one with matching $n$:

$$
\boxed{\,c_n = \frac{1}{L}\int_{-L/2}^{L/2} f(x)\, e^{-i 2\pi n x / L}\, dx\,}
$$

A clean recipe: to find coefficient $c_n$, integrate $f(x)$ against the $n$-th wave.

### 1.4 Worked examples

| $f(x)$ | $c_n$ |
|---|---|
| $\operatorname{sign}(x)$ | $\dfrac{i}{n\pi}\bigl[(-1)^n - 1\bigr]$ |
| $x/L$ | $\dfrac{i}{2 n \pi} (-1)^n$ |

Both have **purely imaginary $c_n$** — a symptom of $f(x)$ being an odd function. (Even functions give real $c_n$, odd functions give imaginary $c_n$. We'll formalize this later.)

Look at the magnitudes: $|c_n|$ decays like $1/n$. Higher frequencies contribute less. Truncating the series at finite $N$ gives a partial sum $S_N(x)$ that approximates $f(x)$ — the more terms, the better. At a discontinuity, $S_\infty(x_0) = \tfrac{1}{2}[f(x_0^-) + f(x_0^+)]$ (Gibbs's classic result, which we'll meet again in L09).

---

## 2. From n-space to k-space

### 2.1 Wavelength

Each basis function $e^{i 2\pi n x / L}$ has wavelength

$$
\lambda_n = \frac{L}{n}.
$$

So $\lambda_1 = L$ (one full wave fits in the interval), $\lambda_2 = L/2$ (two full waves), and so on. Plotting $c_n$ versus $\lambda_n$ gives a $\lambda$-space — but the points are unevenly spaced, which is awkward.

### 2.2 The wavevector $k$

Switch to the **inverse** of the wavelength:

$$
\boxed{\,k_n = \frac{1}{\lambda_n} = \frac{n}{L}\,}
$$

Units: m$^{-1}$ (or mm$^{-1}$, etc.) The k-axis is uniformly spaced with step

$$
\Delta k = \frac{1}{L}.
$$

That tiny $\Delta k$ is one of the most important quantities in MRI: **it is the spacing of samples in the k-space we acquire**. The smallest non-zero wavevector $k_1 = 1/L$ corresponds to the longest wave that fits in the FOV.

Rewritten in terms of $k$:

$$
f(x) = \sum_{n=-\infty}^{\infty} c_n\, e^{i 2\pi k_n x} = \sum_n c_n\, e^{i 2\pi n\,\Delta k\, x}
$$

Your first k-space plot 🎉 — $c_n$ as a function of $k_n$.

### 2.3 Periodicity — the price of a discrete k-space

The basis functions are periodic with period $L$ in $x$, so the *reconstructed* $f(x)$ is also periodic: $f(x) = f(x + L)$. If you evaluate the Fourier series outside $[-L/2, L/2]$, you get **periodic copies** of the original signal.

In k-space terms: $f(x) = f(x + 1/\Delta k)$. **Smaller $\Delta k$ ↔ larger period ↔ less aliasing.** This is exactly the FOV / aliasing relationship in MRI: if you sample k-space too sparsely, your reconstructed image wraps around. Module 4 will spend a lot of time on this.

### 2.4 Two-way transformation

The pair $\{c_n\} \leftrightarrow f(x)$ is invertible: given one, you compute the other.

- **x-space → k-space:** $c_n = \dfrac{1}{L}\!\int_{-L/2}^{L/2} f(x)\, e^{-i 2\pi n \Delta k\, x}\, dx$
- **k-space → x-space:** $f(x) = \sum_n c_n\, e^{i 2\pi n \Delta k\, x}$

---

## 3. The Fourier transform — the continuous version

### 3.1 Taking $L \to \infty$

The Fourier series only describes $f(x)$ on an interval. If you want $f(x)$ on the whole real line, you need $L \to \infty$ — equivalently, $\Delta k \to 0$. The discrete sum becomes a continuous integral. Relabel $c_n = \tilde f_n \Delta k$ and take the limit:

$$
\sum_n \tilde f_n\, e^{i 2\pi n \Delta k\, x}\, \Delta k
\quad \xrightarrow{\text{continuum}} \quad
\int_{-\infty}^{\infty} \tilde f(k)\, e^{i 2\pi k x}\, dk.
$$

That's the **inverse Fourier transform**. The forward version comes from the limit of the $c_n$ formula:

$$
\boxed{\;\tilde f(k) = \mathcal{F}\{f(x)\} = \int_{-\infty}^{\infty} f(x)\, e^{-i 2\pi k x}\, dx\;}
$$

$$
\boxed{\;f(x) = \mathcal{F}^{-1}\{\tilde f(k)\} = \int_{-\infty}^{\infty} \tilde f(k)\, e^{+i 2\pi k x}\, dk\;}
$$

The tilde marks k-space functions; the only difference between forward and inverse is the sign in the exponent.

> ⚠️ **Convention warning.** Other textbooks use $e^{-i\omega t}$ without the $2\pi$ (and add prefactors like $1/\sqrt{2\pi}$ or $1/2\pi$). Always check which convention a paper uses. We stick to the **$2\pi$-in-exponent, no-prefactor** convention throughout this course.

Generalizes immediately to 2D and 3D by replacing $k x$ with $\boldsymbol k \cdot \boldsymbol r$ in the exponent.

### 3.2 The Dirac delta function

A function defined by what it does *under an integral*:

$$
\int_{-\infty}^{\infty} f(x')\, \delta_\mathrm{D}(x - x')\, dx' = f(x)
$$

It's the continuous version of the Kronecker delta — infinitely tall, infinitely narrow, with unit area. One particularly important representation:

$$
\delta_\mathrm{D}(x) = \int_{-\infty}^{\infty} e^{i 2\pi k x}\, dk
$$

This is just the inverse Fourier transform of $\tilde f(k) = 1$. We'll use it constantly.

### 3.3 Key Fourier pairs

These are the four pairs every MRI engineer carries in their head:

| $f(x)$ | $\tilde f(k) = \mathcal{F}\{f(x)\}$ |
|---|---|
| Boxcar $\Pi(x, 0, \Delta x)$ | $\Delta x \cdot \operatorname{sinc}(\pi k \Delta x)$ |
| $\delta_\mathrm{D}(x - x_0)$ | $e^{-i 2\pi k x_0}$ |
| $e^{-c\lvert x\rvert}$ | $\dfrac{2c}{c^2 + 4\pi^2 k^2}$ (Lorentzian) |
| $e^{-c \pi x^2}$ | $\dfrac{1}{\sqrt{c}}\, e^{-\pi k^2 / c}$ (Gaussian → Gaussian) |

> **Note on the sinc convention.** This lecture uses $\operatorname{sinc}(x) = \sin(x)/x$ (unnormalized — sometimes called the "math" sinc). The other common convention is $\operatorname{sinc}(x) = \sin(\pi x)/(\pi x)$ (normalized — the "signal-processing" sinc, what `numpy.sinc` uses). Be careful when reading other sources.

Because the forward and inverse transforms differ only by a sign in the exponent, the same pairs work in reverse (with appropriate sign flips in arguments). E.g., the FT of a sinc is a boxcar.

### 3.4 The theorems you'll use forever

| Property | $\mathcal{F}\{\cdot\}$ |
|---|---|
| **Shift theorem** | $f(x - x_0) \;\longleftrightarrow\; e^{-i 2\pi k x_0}\,\tilde f(k)$ |
| **Convolution theorem** | $f(x) \cdot g(x) \;\longleftrightarrow\; \tilde f(k) * \tilde g(k)$ |
| **Scaling theorem** | $f(c x) \;\longleftrightarrow\; \dfrac{1}{\lvert c\rvert}\,\tilde f(k/c)$ |
| **Linearity** | $a f(x) + b g(x) \;\longleftrightarrow\; a\tilde f(k) + b\tilde g(k)$ |
| **Reality** | if $f(x) \in \mathbb{R}$, then $\tilde f(k) = \tilde f^*(-k)$ (Hermitian symmetry) |

**Things to internalize:**
- A **shift in position space** is a **linear phase in k-space**. (Move the image one pixel → multiply k-space by a complex exponential.) ← This is how parallel imaging coil sensitivity maps work.
- A **product in one domain** is a **convolution in the other**. (Window an image → convolve its k-space with a sinc; multiply k-space by a mask → convolve image with the mask's FT.) ← This is why undersampling causes aliasing.
- A **narrow function** in one domain is a **wide function** in the other. Time/bandwidth tradeoff in a single equation.
- **Real signals have Hermitian-symmetric k-space.** ← This is what partial Fourier imaging exploits (L10 / CMRI 3).

---

## 4. Why this matters for MRI

For now, accept the punch line that L10 (k-space encoding) will derive properly:

> The MR signal acquired in the presence of a gradient $\boldsymbol G(t)$ is the **Fourier transform** of the spin density $\rho(\boldsymbol r)$, sampled at the k-space location $\boldsymbol k(t) = \tfrac{\gamma}{2\pi}\int_0^t \boldsymbol G(t')\, dt'$.

This means:

- **Image** $\rho(\boldsymbol r)$ lives in position space.
- **Acquired signal** $s(\boldsymbol k)$ lives in k-space.
- **Reconstruction** = inverse FFT.
- **Sampling pattern in k-space** ↔ **artifacts in image space** (via the convolution theorem).

Everything else in modules 2–5 of this course is a refinement of these four bullets.

---

## 5. Key takeaways

1. Any well-behaved $f(x)$ on $[-L/2, L/2]$ equals a sum of complex exponentials with discrete frequencies $k_n = n/L$.
2. The coefficients are computed by **integration against the basis** ($c_n = \tfrac{1}{L}\int f e^{-i 2\pi n x/L} dx$) — orthogonality is the engine.
3. Discrete k-space ↔ periodic real space (period $L = 1/\Delta k$). **This is the source of aliasing in MRI.**
4. Letting $L \to \infty$ (equivalently $\Delta k \to 0$) takes you from Fourier series to the **continuous Fourier transform**.
5. The four pairs to memorize: **box ↔ sinc, delta ↔ exponential wave, exponential decay ↔ Lorentzian, Gaussian ↔ Gaussian**.
6. The four theorems to memorize: **shift, convolution, scaling, Hermitian symmetry**.
7. In MRI: **acquired signal = FT of image**. Reconstruction = inverse FT.

---

## 📖 Glossary

| Term | Meaning |
|---|---|
| Fourier series | Decomposition of a periodic (or interval-restricted) function into a discrete sum of sines/cosines. |
| Fourier transform | Continuous version of the above; works on functions on the whole real line. |
| Fourier pair | $(f(x), \tilde f(k))$ — two functions related by the Fourier transform. |
| Wavevector $k$ | Inverse wavelength. Units: m$^{-1}$. The "frequency" axis in k-space. |
| $\Delta k$ | Spacing between samples in k-space. Determines the FOV in image space via $L = 1/\Delta k$. |
| k-space | The Fourier domain of the image. In MRI, this is *literally* the space we sample in. |
| Dirac delta | $\delta_\mathrm{D}(x)$ — infinitely peaked unit-area function. Continuous analog of Kronecker delta. |
| sinc | $\sin(x)/x$ here. The FT of a boxcar. The natural "ringing" function in MRI. |
| Convolution | $(f * g)(x) = \int f(x')g(x - x')\, dx'$ — local averaging weighted by $g$. |
| Hermitian symmetry | $\tilde f(k) = \tilde f^*(-k)$ — the k-space signature of a real-valued image. |

---

## 🧪 Exercises

### Exercise 1 — Wavelength ↔ wavevector
The FOV in your scan is $L = 25$ cm. You acquire 256 k-space samples evenly spaced across $k \in [-k_\mathrm{max},\, k_\mathrm{max}]$.
**(a)** What is the k-space step $\Delta k$ (in mm$^{-1}$) that places the first non-zero wave at one full oscillation per FOV?
**(b)** What is $k_\mathrm{max}$?
**(c)** What spatial resolution does that imply ($\Delta x = 1/(2 k_\mathrm{max})$)?

### Exercise 2 — Fourier coefficients of $f(x) = x/L$
Following the lecture's derivation pattern, show that for $f(x) = x/L$ on $[-L/2, L/2]$,
$$ c_n = \frac{i(-1)^n}{2 n \pi} \quad (n \neq 0), \qquad c_0 = 0. $$
Compute the integral by parts.

### Exercise 3 — Symmetry and reality
**(a)** Show that if $f(x)$ is real and **even** ($f(-x) = f(x)$), then $c_n$ is real.
**(b)** Show that if $f(x)$ is real and **odd** ($f(-x) = -f(x)$), then $c_n$ is purely imaginary.
**(c)** Check both facts against the sign and linear examples in the lecture.

### Exercise 4 — Shift theorem
You acquire k-space data and reconstruct an image. Then you realize your patient was 1 cm off-isocenter along $x$. How does the k-space data of the *shifted* image relate to the k-space data of the *centered* image? Be explicit about the form of the multiplicative factor.

### Exercise 5 — Convolution theorem
You apply a hard cutoff to your k-space data: $\tilde f_\text{trunc}(k) = \tilde f(k) \cdot \Pi(k, 0, 2 k_\mathrm{max})$. What does this do to your reconstructed image? Identify the artifact and name it.

### Exercise 6 — Scaling theorem
The FT of $e^{-\pi x^2}$ is $e^{-\pi k^2}$ (Gaussian → Gaussian, both with width 1). What is the FT of $e^{-\pi (x/\sigma)^2}$ for $\sigma > 0$? Comment on the width relationship.

### Exercise 7 — Sinc as the FT of a box
A boxcar of width $\Delta x = 2$ mm in image space has what full-width-at-half-maximum (FWHM) when Fourier-transformed (i.e., in k-space)? Use $\operatorname{sinc}(x) = \sin(x)/x$ and the half-max occurs near $x \approx 1.895$.

### Exercise 8 — Aliasing from undersampling
You sample k-space with $\Delta k = 2/L$ instead of the correct $\Delta k = 1/L$. What happens in image space? Sketch (or describe) the result for an object of size $L$ centered in the FOV.

### Exercise 9 — Hermitian symmetry and partial Fourier
Argue from $\tilde f(k) = \tilde f^*(-k)$ (which holds when the image is real) that you only need to sample **half** of k-space to reconstruct the image. Why doesn't this work in practice for real MR images? (Hint: the image is not exactly real.)

### Exercise 10 — The DC term
What does the $n = 0$ (or $k = 0$) coefficient $c_0$ represent geometrically? Compute it for $f(x) = x/L$, $f(x) = \operatorname{sign}(x)$, and $f(x) = 1$ on $[-L/2, L/2]$.

---

## ✅ Solutions

### Solution 1
**(a)** One full oscillation per FOV means $\lambda_1 = L = 25$ cm $= 250$ mm, so $\Delta k = k_1 = 1/L = 1/250$ mm$^{-1} = 4.0\times 10^{-3}$ mm$^{-1}$.
**(b)** With 256 samples spanning $[-k_\mathrm{max}, k_\mathrm{max}]$ at spacing $\Delta k$: $2 k_\mathrm{max} = 255 \cdot \Delta k$ (or $256\Delta k$ depending on whether you include the endpoint — common convention is $256\Delta k$). Taking the symmetric convention: $k_\mathrm{max} \approx 128 \cdot \Delta k = 0.512$ mm$^{-1}$.
**(c)** $\Delta x = 1/(2 k_\mathrm{max}) \approx 1/(1.024 \text{ mm}^{-1}) \approx 0.98$ mm. So roughly **1 mm resolution**.

### Solution 2
$c_n = \frac{1}{L}\int_{-L/2}^{L/2} (x/L) e^{-i 2\pi n x / L} dx$. Integration by parts with $u = x/L$, $dv = e^{-i 2\pi n x / L} dx$:
$$ c_n = \frac{1}{L}\left[\frac{x/L}{-i 2\pi n / L} e^{-i 2\pi n x/L}\Big|_{-L/2}^{L/2} - \int_{-L/2}^{L/2} \frac{1/L}{-i 2\pi n / L} e^{-i 2\pi n x/L} dx\right] $$
The boundary term evaluates to $\frac{1}{-i 2\pi n}\bigl[\tfrac{1}{2} e^{-i\pi n} + \tfrac{1}{2} e^{i\pi n}\bigr] = \frac{(-1)^n}{-i 2\pi n} = \frac{i(-1)^n}{2\pi n}$. The remaining integral evaluates to zero (whole wave). Hence $c_n = i(-1)^n / (2\pi n)$. For $n = 0$, $c_0 = \frac{1}{L}\int_{-L/2}^{L/2} (x/L) dx = 0$ by oddness. ∎

### Solution 3
**(a)** $c_n = \tfrac{1}{L}\int f(x)[\cos\!-\!i\sin]\, dx$. Even $f$ × even $\cos$ contributes; even $f$ × odd $\sin$ integrates to zero. So $c_n \in \mathbb{R}$.
**(b)** Mirror argument: odd $f$ × odd $\sin$ contributes; odd $f$ × even $\cos$ integrates to zero. So $c_n \in i\mathbb{R}$.
**(c)** sign is odd → $c_n$ pure imaginary ✓. $x/L$ is odd → $c_n$ pure imaginary ✓.

### Solution 4
By the shift theorem, $\mathcal{F}\{f(x - x_0)\} = e^{-i 2\pi k x_0}\,\tilde f(k)$. With $x_0 = 1$ cm: every k-space sample is multiplied by $e^{-i 2\pi k \cdot 0.01\text{ m}}$ — a **linear phase ramp across k-space**. The magnitude is unchanged; only the phase changes. (And this is exactly what GRAPPA-style autocalibration has to undo when the patient shifts.)

### Solution 5
Multiplication by $\Pi$ in k-space ↔ **convolution with a sinc** in image space (convolution theorem applied to the box ↔ sinc pair). The image gets blurred and exhibits oscillatory ringing near sharp edges. This is **Gibbs ringing** — the artifact you'll meet head-on in L09 / CMRI 2.

### Solution 6
Apply the scaling theorem with $c = 1/\sigma$: $\mathcal{F}\{e^{-\pi (x/\sigma)^2}\} = \sigma\, e^{-\pi (\sigma k)^2}$. A Gaussian of width $\sigma$ in image space has FT a Gaussian of width $1/\sigma$ in k-space, scaled by $\sigma$. **Wide ↔ narrow.** The product of widths is invariant — this is the uncertainty principle in one line.

### Solution 7
$\mathcal{F}\{\Pi(x, 0, 2\text{ mm})\} = 2\,\operatorname{sinc}(2\pi k)$ in this lecture's convention. Half-max occurs at $2\pi k \approx 1.895$, i.e. $k \approx 0.302$ mm$^{-1}$. So the FWHM in k-space is $\approx 2 \cdot 0.302 \approx 0.60$ mm$^{-1}$. **Wider box → narrower sinc**: doubling the box width to 4 mm would halve the FWHM to ~0.30 mm$^{-1}$.

### Solution 8
$\Delta k = 2/L$ means the periodicity in image space is $1/\Delta k = L/2$, *half* the FOV. The reconstructed image shows two **periodic copies** of the original object, each shifted by $L/2$. If the object is larger than $L/2$, the copies overlap → **wraparound / aliasing**. (This is the bread-and-butter undersampling artifact you'll deal with in parallel imaging.)

### Solution 9
If $f(x)$ is real, $\tilde f(-k) = \tilde f^*(k)$. So the negative half of k-space is the complex conjugate of the positive half — redundant. In principle, sample only $k \geq 0$ and mirror.
**In practice:** MR images have **phase variations** from off-resonance, motion, field inhomogeneity, coil sensitivities, etc., so the image is not purely real. Partial Fourier methods (L10 / CMRI 3) acquire slightly more than half and use **phase-aware** reconstruction (zero-filling, POCS, homodyne) to fill in the rest with the unavoidable error budget.

### Solution 10
The $k = 0$ coefficient is the **mean** of $f$:
$$ c_0 = \frac{1}{L}\int_{-L/2}^{L/2} f(x)\, dx. $$
- $f = x/L$: $c_0 = 0$ (odd).
- $f = \operatorname{sign}(x)$: $c_0 = 0$ (odd).
- $f = 1$: $c_0 = 1$.
**In MRI, the central k-space sample carries the total integrated signal — the "DC" component of the image, which is by far the brightest point of k-space.**

---

## 📚 Suggested reading

- **Bracewell**, *The Fourier Transform and Its Applications* — the classic.
- **Prince & Links**, *Medical Imaging Signals and Systems*, Ch. 5 — gentle and applied.
- **Nishimura**, *Principles of Magnetic Resonance Imaging*, Ch. 3 — Fourier framing tailored for MRI.

---

*Next up: L09 — k-Space Encoding. We've built the math; now we'll plug it into the actual MR signal equation and see why the acquired signal literally lives in k-space.*
