# C10 — Introduction to Machine Learning & Neural Networks

**Module 5 · AI for MR Reconstruction**
**Source:** Computational MRI lecture *"Introduction to machine learning and neural networks"* (FAU Erlangen-Nürnberg). *(The course PDF labels this Lecture 9; in the repo's Module-5 numbering it is C10.)*

---

## Big picture

Classical reconstruction (Modules 1–4) is built from hand-derived physics and math. Machine learning takes a different route: instead of *deriving* the mapping from data to answer, we **fit a flexible function to examples**. Give the model many pairs $(x_i, y_i)$ — an image and its label, a voxel and its tissue class — and let it adjust its own internal parameters until its outputs match the targets. This lecture builds that idea up from a single neuron to convolutional networks, and explains the engine that makes it all work: **gradient descent powered by backpropagation**. The next lecture (C11) turns these tools on the reconstruction problem itself.

---

## 1. The artificial neuron

A neuron takes the outputs of the previous layer, forms a weighted sum, adds a bias, and passes the result through a non-linear **activation** $\sigma$:

$$a_j^{\,l} = \sigma\!\Big(\sum_k w_{jk}^{\,l}\,a_k^{\,l-1} + b_j^{\,l}\Big)$$

- $w_{jk}^{\,l}$ — weight into neuron $j$ of layer $l$ from neuron $k$ of layer $l-1$
- $b_j^{\,l}$ — bias of neuron $j$ in layer $l$
- $\sigma$ — activation; the lecture uses the **sigmoid**

$$\sigma(x) = \frac{1}{1+e^{-x}}, \qquad \sigma'(x) = \sigma(x)\big(1-\sigma(x)\big)$$

The perceptron dates back to **Rosenblatt (1957)**. The non-linearity is essential: without it, stacking layers would collapse into a single linear map.

## 2. Networks as function composition

A deep network is just nested functions, one per layer:

$$a^{4} = F(x) = f^{4}\!\big(f^{3}\!\big(f^{2}\!\big(f^{1}(x)\big)\big)\big)$$

Many parameters ⇒ **high descriptive capacity**. **Cybenko (1989)** showed a network with a single sufficiently-wide hidden layer can approximate any continuous function (universal approximation). Depth makes this efficient in practice.

## 3. Supervised learning setup

Training data is a set of labelled pairs $\{(x_1,y_1),\dots,(x_N,y_N)\}$. For the lecture's chest X-ray example the labels are **one-hot vectors**:

$$\text{Normal} = \begin{bmatrix}1\\0\end{bmatrix},\qquad \text{COVID-19} = \begin{bmatrix}0\\1\end{bmatrix}$$

## 4. Cost function

We need a single number that says how wrong the network currently is. The lecture uses the **quadratic (MSE) cost**:

$$C(w,b) = \frac{1}{2N}\sum_{x=1}^{N}\big\lVert y_x - a_x \big\rVert_2^{2}$$

Training = find the weights and biases that **minimise** $C$.

## 5. Gradient descent

We cannot solve $\nabla C = 0$ in closed form, so we walk downhill along the negative gradient. For one variable:

$$x_{i+1} = x_i - \alpha\,\frac{\partial f(x_i)}{\partial x_i}$$

and for the network parameters:

$$\tilde w_{jk}^{\,l} = w_{jk}^{\,l} - \alpha\,\frac{\partial C}{\partial w_{jk}^{\,l}}, \qquad \tilde b_j^{\,l} = b_j^{\,l} - \alpha\,\frac{\partial C}{\partial b_j^{\,l}}$$

$\alpha$ is the **learning rate**. Gradient descent finds a *local* minimum, so initialisation and $\alpha$ matter.

## 6. Backpropagation — the heart of the lecture

Gradient descent needs $\partial C/\partial w$ and $\partial C/\partial b$ for **every** parameter. **Backpropagation** (Rumelhart, Hinton & Williams, 1986) is an efficient recursion that computes them all via the chain rule. It uses two passes:

- **Forward pass:** feed $x$ through the network, caching every $z^l = w^l a^{l-1}+b^l$ and $a^l = \sigma(z^l)$, ending at the cost $C_0 = \tfrac12 (y - a^L)^2$.
- **Backward pass:** propagate the error from the output back toward the input.

### The output-layer derivatives

Reading off the computation graph $w^L, a^{L-1}, b^L \to z^L \to a^L \to C_0$:

$$\frac{\partial z^{L}}{\partial w^{L}} = a^{L-1}, \qquad \frac{\partial z^{L}}{\partial b^{L}} = 1, \qquad \frac{\partial a^{L}}{\partial z^{L}} = \sigma'(z^{L}), \qquad \frac{\partial C_0}{\partial a^{L}} = a^{L} - y$$

Chaining gives the two key results:

$$\frac{\partial C_0}{\partial w^{L}} = a^{L-1}\,\sigma'(z^{L})\,(a^{L}-y), \qquad \frac{\partial C_0}{\partial b^{L}} = \sigma'(z^{L})\,(a^{L}-y)$$

### Propagating to the previous layer

$$\frac{\partial C_0}{\partial a^{L-1}} = w^{L}\,\sigma'(z^{L})\,(a^{L}-y)$$

### General recursion (any layer $l$)

$$\frac{\partial C_0}{\partial w_{jk}^{\,l}} = a_k^{\,l-1}\,\sigma'(z_j^{\,l})\,\frac{\partial C_0}{\partial a_j^{\,l}}, \qquad \frac{\partial C_0}{\partial b_j^{\,l}} = \sigma'(z_j^{\,l})\,\frac{\partial C_0}{\partial a_j^{\,l}}$$

$$\frac{\partial C_0}{\partial a_j^{\,l}} = \sum_j w_{jk}^{\,l+1}\,\sigma'(z_j^{\,l+1})\,\frac{\partial C_0}{\partial a_j^{\,l+1}}$$

A compact way to see it: define the layer error $\delta^l = \sigma'(z^l)\,\partial C_0/\partial a^l$. Then $\partial C_0/\partial b^l = \delta^l$, $\partial C_0/\partial w^l = \delta^l (a^{l-1})^{\!\top}$, and the error rolls back via $\partial C_0/\partial a^{l-1} = (w^l)^{\!\top}\delta^l$.

### Why it's efficient (and insightful)

- Each step touches just **two adjacent layers**.
- Intermediate quantities are reused instead of recomputed — no repeated chain-rule expansions.
- The gradient magnitudes reveal **what limits learning speed** (e.g. saturated sigmoids have tiny $\sigma'$, so those weights barely move — the seed of the vanishing-gradient problem).

> **Practical tip from the notebook:** always validate a hand-written backprop against a **numerical gradient** (perturb each weight by $\pm\varepsilon$, compare finite-difference slopes). Agreement to ~$10^{-7}$ confirms correctness.

## 7. From fully-connected to convolutional networks

Feeding a $320\times320$ image ($102{,}400$ pixels) into a fully-connected layer mapping pixels to pixels needs

$$102400 \times 102400 \approx 1.05\times10^{10}$$

parameters — for **one** layer. **CNNs** (LeCun, 1989) fix this with two principles:

- **Local connectivity** — each output depends only on a small neighbourhood.
- **Parameter sharing** — the same small filter slides across the whole image.

A single $3\times3$ convolution computes $w^\top x + b$ at every position with just $3\cdot3+1 = 10$ parameters, independent of image size — roughly a **billion-fold** reduction versus the fully-connected layer. This is why CNNs are the default for image inputs.

---

## Exercises in the notebook

1. **Brain-tissue classification from DTI features.** Each voxel is $x_i = [\,\mathrm{T1w}, \mathrm{FA}, \mathrm{MD}, \mathrm{AD}, \mathrm{RD}\,]$; the label is the tissue class (Thalamus = GM, corpus callosum = aligned WM, cortical WM). An MLP on standardised features separates the classes cleanly. *(HCP data; notebook uses synthetic voxels around the lecture's representative values.)*
2. **Image-quality classification of accelerated reconstructions.** Decide whether a reconstructed image is a fully-sampled reference or a 4×-accelerated PI-CS result (which can blur and lose information, per Knoll 2011). A small CNN learns the local texture cues. Both come with fully worked solutions.

---

## Key takeaways

- A neuron is $a = \sigma(w^\top a^{-} + b)$; a network is a composition of such layers with high descriptive capacity.
- **Training = minimising a cost** via gradient descent: $\theta \leftarrow \theta - \alpha\,\partial C/\partial \theta$.
- **Backpropagation** is the efficient chain-rule recursion that supplies every gradient; each step involves only two layers and reuses computation.
- The derivative $\sigma'(z)$ appears in every gradient — saturated activations slow learning.
- For images, **CNNs** beat fully-connected nets by orders of magnitude in parameter count through local connectivity and weight sharing.

## Self-test questions

1. Why is a non-linear activation necessary? What happens to a deep network if $\sigma$ is the identity?
2. Derive $\partial C_0/\partial b^L$ from the computation graph and explain why the factor multiplying $\sigma'(z^L)(a^L-y)$ is exactly $1$.
3. The weight gradient contains $\sigma'(z^l)$. Using the shape of $\sigma'$, explain why very large $|z|$ stalls learning.
4. How does the recursion let backprop reuse work, and why does that make it far cheaper than computing each gradient independently?
5. A $256\times256$ image goes into a pixel-to-pixel fully-connected layer. How many weights is that? Compare to a single $5\times5$ conv filter.
6. Name the two CNN principles that cut the parameter count, and explain in one sentence each why they are reasonable for images.
7. Why standardise input features (e.g. T1w $\approx 900$ vs FA $\approx 0.3$) before training a sigmoid network?

## References

- Rosenblatt, *Psychological Review* (1957) — the perceptron.
- Cybenko, *Math. Control Signals Systems* (1989) — universal approximation.
- Rumelhart, Hinton & Williams (1986) — *Learning representations by back-propagating errors*.
- LeCun et al., *NIPS* (1989) — convolutional networks.
- Knoll et al., *MRM* (2011) — parallel-imaging / compressed-sensing accelerated MRI.

---

*Next up — **C11: ML for MR image reconstruction**: applying these networks to reconstruct images directly from undersampled k-space (learned priors and unrolled networks).*
