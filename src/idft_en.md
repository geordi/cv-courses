# Inverse Discrete Fourier Transform

In the previous lesson, we implemented the direct and separable Discrete Fourier Transform. Today, we will implement its inverse, called the Inverse Discrete Fourier Transform (IDFT), using the same two approaches.

We want to transform the complex frequency spectrum $F$ back to its spatial domain $f$, which is an image. As before, $M$ is the number of rows and $N$ is the number of columns; the first matrix index denotes a row.

## Direct 2D IDFT

The computation of the IDFT can be implemented using the following formula:

$$
f(m, n) = \sum\limits_{k=0}^{M-1} \sum\limits_{l=0}^{N-1} F(k, l) \varphi_{k, l}(m, n)\,.
\tag{1}
$$

The basis function $\varphi_{k,l}$ is defined as

$$
\varphi_{k, l}(m, n)
= \frac{1}{\sqrt{MN}}
e^{i 2 \pi \left( \frac{mk}{M} + \frac{nl}{N} \right)},
\quad k = 0, 1, \dots, M-1,
\quad l = 0, 1, \dots, N-1\,.
\tag{2}
$$

Notice that the exponent has a positive sign, unlike the basis used in the DFT. Both transforms use the normalization factor $1/\sqrt{MN}$; do not introduce an additional factor $1/(MN)$ in the inverse transform.

To compute the basis function, use Euler's formula $e^{ix}=\cos(x)+i\sin(x)$.
Both the spectrum coefficients and the basis values are complex, so the multiplication and accumulation must use complex arithmetic.

As with the direct DFT, this implementation has complexity $O(M^2N^2)$, or $O(N^4)$ for a square image.

## Separable 2D IDFT

The inverse transform is separable as well:

$$
e^{i2\pi\left(\frac{mk}{M}+\frac{nl}{N}\right)}
= e^{i2\pi\frac{mk}{M}} e^{i2\pi\frac{nl}{N}}\,.
$$

First, apply the 1D inverse transform to each row of $F$ and store the results in a complex intermediate matrix $H$:

$$
H(k,n) = \frac{1}{\sqrt{N}}
\sum_{l=0}^{N-1} F(k,l)e^{i2\pi\frac{nl}{N}}\,.
\tag{3}
$$

Then, apply the 1D inverse transform to each column of $H$:

$$
f(m,n) = \frac{1}{\sqrt{M}}
\sum_{k=0}^{M-1} H(k,n)e^{i2\pi\frac{mk}{M}}\,.
\tag{4}
$$

The two normalization factors again multiply to $1/\sqrt{MN}$.

### Reusing the 1D transform

Extend your 1D DFT function from the previous lesson with a `sign` parameter:

$$
b(q) = \frac{1}{\sqrt{L}}
\sum_{p=0}^{L-1} a(p)e^{s i2\pi\frac{pq}{L}},
\quad s\in\{-1,+1\}\,.
\tag{5}
$$

Use `sign = -1` for the forward DFT and `sign = +1` for the inverse DFT. With the normalization used in these exercises, only the exponent sign changes.

```text
function TRANSFORM1D(input, sign):
    L = length(input)
    output = complex vector of length L

    for q = 0 .. L-1:
        sum = 0 + 0i
        for p = 0 .. L-1:
            angle = sign * 2 * pi * p * q / L
            sum += input[p] * (cos(angle) + i * sin(angle))
        output[q] = sum / sqrt(L)

    return output
```

Build the separable inverse transform in the same way as the forward transform:

```text
function IDFT2D_SEPARABLE(F):
    M = number of rows in F
    N = number of columns in F
    H = complex matrix of size M x N
    result = complex matrix of size M x N

    for k = 0 .. M-1:
        H[k, :] = TRANSFORM1D(F[k, :], +1)

    for n = 0 .. N-1:
        result[:, n] = TRANSFORM1D(H[:, n], +1)

    return result
```

The notation `A[r, :]` denotes a row, and `A[:, c]` denotes a column. Copy values into temporary vectors if necessary. Keep the intermediate matrix complex; do not discard its imaginary components.

The row pass costs $O(MN^2)$ and the column pass costs $O(NM^2)$. The total complexity is $O(MN(M+N))$, or $O(N^3)$ for a square image. This algorithm uses ordinary 1D IDFT sums, not an FFT.

## Reconstruction and Verification

Use the original complex spectrum for reconstruction, not its amplitude, phase, power, logarithmic visualization, or normalized display image.

If the spectrum has been shifted to center the zero-frequency coefficient, undo that shift before computing either IDFT. For even dimensions, swapping the quadrants again is sufficient. For arbitrary dimensions, undo the circular shift by moving $\lfloor M/2\rfloor$ rows and $\lfloor N/2\rfloor$ columns toward decreasing indices (`ifftshift`). Applying the forward shift twice does not generally restore the original order for odd dimensions.

For an unmodified spectrum of a real image, the reconstructed values should be real, apart from small imaginary residuals caused by floating-point rounding. Measure those residuals, then use the real part as the reconstructed image. Do not use the magnitude: it would lose the sign of reconstructed values. Round and clamp values only when converting the result to an integer image for display or saving.

**Hint:** Use complex numbers with `double` real and imaginary components for all transform calculations, and `double` for the reconstructed image before conversion.

## Tasks

1. Implement the direct 2D IDFT using Equations (1) and (2).
2. Extend your 1D transform with a direction parameter and implement the separable 2D IDFT. Implement the sums yourself; do not use a library DFT or FFT routine.
3. Reconstruct the same unshifted spectrum using both inverse implementations. Compare their complex results using the maximum absolute difference.
4. Verify the full round trip: input image $\rightarrow$ separable DFT $\rightarrow$ separable IDFT $\rightarrow$ reconstructed image. Include a small rectangular image (for example, $3 \times 5$), a constant image, and a nonconstant image.
5. Before rounding or clamping, report the maximum reconstruction error and maximum imaginary residual:

$$
E_{\max} = \max_{m,n}\left|\operatorname{Re}(\hat f(m,n))-f(m,n)\right|,
\qquad
I_{\max} = \max_{m,n}\left|\operatorname{Im}(\hat f(m,n))\right|\,.
$$

Here, $f$ is the original image and $\hat f$ is the complex output of the IDFT. Both quantities should be close to zero; exact equality is not required because of floating-point rounding.

6. Test a round trip that includes centering and uncentering the spectrum, including an image with odd dimensions.
7. Measure the execution time of the direct and separable IDFT on the same square image sizes used in the DFT exercise. Exclude visualization and image conversion. Compare the observed growth with $O(N^4)$ and $O(N^3)$.
