## Mathematical Foundations of Flow Encryption

We present the rigorous mathematical framework underlying **Flow Encryption** – a symmetric encryption scheme that hides bits as orthogonal perturbations along a secret, smooth color flow. The ciphertext resides in the RGB cube \([0,1]^3\) and is designed to be statistically indistinguishable from natural color gradients.

---

### 1. Color Space and Geometry

Let \(\mathcal{C} = [0,1]^3\) be the RGB color cube. A color is a vector \(\mathbf{c} = (r, g, b)\) with each component in \([0,1]\). The Euclidean distance between colors is  

\[
d(\mathbf{c}_1, \mathbf{c}_2) = \|\mathbf{c}_1 - \mathbf{c}_2\|_2.
\]

All operations are continuous; no quantization is applied until output (image encoding).

---

### 2. Flow Field Definition

A **flow field** is a smooth map from a parameter domain to \(\mathcal{C}\).

#### 2.1 1D Flow (Stream Mode)

For a 1D stream (time or index \(t \in [0,1]\)), we use a cubic Bézier curve:

\[
\mathbf{B}(t) = (1-t)^3 \mathbf{P}_0 + 3(1-t)^2 t \mathbf{P}_1 + 3(1-t)t^2 \mathbf{P}_2 + t^3 \mathbf{P}_3,
\]

where \(\mathbf{P}_k \in \mathcal{C}\) are control points derived from the secret key. Its derivative (tangent) is

\[
\mathbf{B}'(t) = 3(1-t)^2(\mathbf{P}_1-\mathbf{P}_0) + 6(1-t)t(\mathbf{P}_2-\mathbf{P}_1) + 3t^2(\mathbf{P}_3-\mathbf{P}_2).
\]

For \(t\) where \(\|\mathbf{B}'(t)\| = 0\) (rare), we set the tangent to a default direction.

#### 2.2 2D Flow (Image Mode)

For a 2D image with normalized coordinates \((x,y) \in [0,1]^2\), we use bilinear interpolation on a control grid of size \(H \times W\). Let \(\mathbf{C}_{i,j} \in \mathcal{C}\) for \(i=0..H-1\), \(j=0..W-1\) be control colors. Then

\[
\mathbf{B}(x,y) = (1-\alpha)(1-\beta)\mathbf{C}_{i,j} + \alpha(1-\beta)\mathbf{C}_{i+1,j} + (1-\alpha)\beta\mathbf{C}_{i,j+1} + \alpha\beta\mathbf{C}_{i+1,j+1},
\]

where  
\(i = \lfloor x(W-1) \rfloor,\; j = \lfloor y(H-1) \rfloor,\; \alpha = x(W-1)-i,\; \beta = y(H-1)-j\).  
The partial derivatives (gradient) are computed numerically or via finite differences.

---

### 3. Orthogonal Perturbation Encoding

#### 3.1 Stream Mode: 2 Bits per Symbol

At a given parameter \(t\), the flow gives a base color \(\mathbf{B}(t)\) and a tangent vector \(\mathbf{T}(t) = \mathbf{B}'(t)\). Choose two orthonormal vectors \(\mathbf{v}_1(t), \mathbf{v}_2(t)\) spanning the plane orthogonal to \(\mathbf{T}(t)\):

\[
\mathbf{v}_1 = \frac{\mathbf{T} \times \mathbf{a}}{\|\mathbf{T} \times \mathbf{a}\|}, \quad
\mathbf{v}_2 = \frac{\mathbf{T} \times \mathbf{v}_1}{\|\mathbf{T} \times \mathbf{v}_1\|},
\]

where \(\mathbf{a}\) is a fixed axis (e.g., \((1,0,0)\) if \(|\mathbf{T}_x| < 0.9\), else \((0,1,0)\)). This construction is deterministic given \(\mathbf{T}\).

Two bits \(b_0, b_1 \in \{0,1\}\) are encoded as signed displacements:

\[
\mathbf{C}(t) = \mathbf{B}(t) + \delta \big( (2b_0-1)\mathbf{v}_1 + (2b_1-1)\mathbf{v}_2 \big),
\]

where \(\delta > 0\) is the perturbation amplitude (typically \(\delta \approx 0.05\)). The ciphertext is the sequence \(\{\mathbf{C}(t_k)\}\) for \(k=0,\dots,N-1\) with \(t_k = k/(N-1)\).

**Decryption:** Given \(\mathbf{C}(t)\), compute the projections

\[
p_0 = \frac{(\mathbf{C}-\mathbf{B})\cdot \mathbf{v}_1}{\delta}, \quad
p_1 = \frac{(\mathbf{C}-\mathbf{B})\cdot \mathbf{v}_2}{\delta}.
\]

Then recover bits as \(b_i = 1\) if \(p_i > 0\), else \(0\). The orthonormality ensures that the two dimensions do not interfere.

#### 3.2 Image Mode: 1 Bit per Pixel

For each pixel \((x,y)\), we use a single random unit vector \(\mathbf{r}(x,y)\) that is **pseudo‑random but deterministic** from the key and coordinates. It does not need to be orthogonal to the flow gradient; we only require that the receiver can generate the same \(\mathbf{r}\). The bit \(b\) is encoded as

\[
\mathbf{C}(x,y) = \mathbf{B}(x,y) + \delta (2b-1) \mathbf{r}(x,y).
\]

Decryption projects the difference onto \(\mathbf{r}\):

\[
\hat{b} = \mathbb{1}\big[ (\mathbf{C}-\mathbf{B})\cdot \mathbf{r} > 0 \big].
\]

Because \(\mathbf{r}\) is a unit vector, the signal‑to‑noise ratio for a single bit is \(2\delta / \sigma_{\text{noise}}\) when noise is additive white Gaussian with variance \(\sigma_{\text{noise}}^2\) in each channel.

---

### 4. Noise Robustness and Parameter Choice

Assume the ciphertext is transmitted through a noisy channel that adds independent Gaussian noise \(\mathcal{N}(0, \sigma^2)\) to each RGB component. For a single bit encoded with amplitude \(\delta\), the received projection is

\[
p = \delta (2b-1) + \eta,
\]

where \(\eta \sim \mathcal{N}(0, \sigma^2)\) because \(\mathbf{r}\) is unit. The bit error probability is

\[
P_e = \frac{1}{2}\operatorname{erfc}\left(\frac{\delta}{\sigma\sqrt{2}}\right).
\]

To achieve \(P_e < 10^{-6}\), we need \(\delta / \sigma > 4.75\). For a typical image noise standard deviation of 0.01 (1% of full scale), \(\delta = 0.05\) gives \(\delta/\sigma = 5\), so \(P_e \approx 2.9\times10^{-7}\). This is excellent.

For stream mode with 2 bits, the two projections are independent, so the per‑bit error probability is the same.

---

### 5. Key Space and Entropy

The secret key is a 256‑bit integer used to seed a pseudo‑random number generator. For image mode, the key generates:
- The control grid \(\mathbf{C}_{i,j}\) ( \(H \times W \times 3\) independent colors, each in \([0,1]\) ) → \(3HW\) continuous values.
- The per‑pixel random direction vectors \(\mathbf{r}(x,y)\) are also derived from the key, providing additional entropy.

Thus the effective key space is infinite in principle (continuous parameters), but we fix the key to a 256‑bit seed, giving \(2^{256}\) possibilities – computationally infeasible to brute force.

For stream mode, the key generates 4 control points (12 continuous values) and the orthonormal basis is determined deterministically, so the entropy is again \(2^{256}\).

---

### 6. Invertibility and Correctness

**Theorem:** For a given key, the encryption map is injective (no two different plaintexts produce the same ciphertext) and decryption recovers the original plaintext exactly in the absence of noise.

*Proof (sketch):* For image mode, at each pixel the mapping from bit \(b\) to \(\mathbf{C}\) is \(\mathbf{C} = \mathbf{B} + \delta(2b-1)\mathbf{r}\). Since \(\mathbf{r} \neq \mathbf{0}\), this is a bijection between \(\{0,1\}\) and the two points \(\mathbf{B} \pm \delta \mathbf{r}\). Decryption projects onto \(\mathbf{r}\) and compares the sign, which uniquely identifies \(b\). For stream mode, the same holds for each pair of bits because \(\mathbf{v}_1, \mathbf{v}_2\) are orthonormal. Hence the mapping from \((b_0,b_1)\) to \(\mathbf{C}\) is injective. ∎

---

### 7. Steganographic Security (Indistinguishability)

The ciphertext consists of colors that lie on two parallel hyperplanes at distance \(\delta\) from the base flow. An adversary who does not know the flow \(\mathbf{B}\) or the directions \(\mathbf{r}\) sees only a set of points in \(\mathcal{C}\). If the base flow is a smooth, low‑frequency function (e.g., a linear gradient), the set \(\{\mathbf{B}(t_k)\}\) forms a smooth curve. Adding the perturbations creates points that oscillate around this curve. To an adversary without the key, the sequence looks like a noisy smooth curve – indistinguishable from a natural color gradient (e.g., a sunset or a smooth interpolated image). Statistical tests (e.g., autocorrelation, entropy) cannot distinguish it from a genuine gradient because the perturbations are small and the flow itself is random but smooth.

More formally, let \(\mathcal{D}_{\text{real}}\) be the distribution of ciphertexts produced by Flow Encryption for a uniformly random key and uniformly random plaintext. Let \(\mathcal{D}_{\text{grad}}\) be the distribution of random smooth gradients (e.g., random control points and no hidden data). The total variation distance between these distributions is bounded by the probability of detecting the perturbations. For \(\delta\) small and without knowledge of the flow, an optimal detector would need to estimate the flow, which requires solving a non‑linear regression problem with high variance. Thus, for practical purposes, the ciphertext is computationally indistinguishable from a natural gradient.

---

### 8. Complexity and Performance

- **Image mode:** For an \(H \times W\) image, encryption requires evaluating the bilinear flow at each pixel (\(O(1)\) operations) and generating a pseudo‑random unit vector (precomputed or on‑the‑fly). Total time \(O(HW)\). Decryption is identical.
- **Stream mode:** For \(N\) symbols (each carrying 2 bits), evaluation of the Bézier curve and its tangent at each \(t_k\) is \(O(1)\). Orthonormal basis computation requires a few cross products. Total time \(O(N)\).

Both are highly parallelizable.

---

### 9. Summary of Equations

| Operation | Formula |
|-----------|---------|
| Base flow (1D) | \(\mathbf{B}(t) = \sum_{k=0}^3 \binom{3}{k} (1-t)^{3-k} t^k \mathbf{P}_k\) |
| Base flow (2D) | Bilinear interpolation of control grid |
| Tangent (1D) | \(\mathbf{B}'(t) = 3(1-t)^2(\mathbf{P}_1-\mathbf{P}_0) + 6(1-t)t(\mathbf{P}_2-\mathbf{P}_1) + 3t^2(\mathbf{P}_3-\mathbf{P}_2)\) |
| Orthonormal basis (stream) | \(\mathbf{v}_1 = \frac{\mathbf{T}\times\mathbf{a}}{\|\mathbf{T}\times\mathbf{a}\|},\; \mathbf{v}_2 = \frac{\mathbf{T}\times\mathbf{v}_1}{\|\mathbf{T}\times\mathbf{v}_1\|}\) |
| Encoding (stream, 2 bits) | \(\mathbf{C} = \mathbf{B} + \delta\big((2b_0-1)\mathbf{v}_1 + (2b_1-1)\mathbf{v}_2\big)\) |
| Encoding (image, 1 bit) | \(\mathbf{C} = \mathbf{B} + \delta(2b-1)\mathbf{r}\) |
| Decoding (projection) | \(b = \mathbb{1}[(\mathbf{C}-\mathbf{B})\cdot \mathbf{r} > 0]\) |
| Bit error probability | \(P_e = \frac{1}{2}\operatorname{erfc}\left(\frac{\delta}{\sigma\sqrt{2}}\right)\) |

These equations form the complete mathematical basis for the Flow Encryption implementation. The code directly translates these formulas into efficient Python routines using NumPy and SciPy.
