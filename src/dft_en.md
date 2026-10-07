# Discrete Fourier Transform

Today's exercise focuses on the implementation of the Discrete Fourier Transform (DFT).
First, implement the transform directly from its definition. Then, use its separability to reduce the computational complexity.
In the next lesson, we will implement the inverse transform.

The Fourier Transform computes the frequency spectrum of a given input image $f$.
This spectrum is denoted by $F$; it is a complex matrix with the same dimensions as the input image.
Throughout this exercise, $M$ is the number of rows and $N$ is the number of columns; the first matrix index denotes a row.

## Direct 2D DFT

$$
F(k, l) = \sum\limits_{m=0}^{M-1} \sum\limits_{n=0}^{N-1} f(m, n) \varphi_{k, l}(m, n)\,.
\tag{1}
$$

The basis function $\varphi_{k,l}$ is defined as

$$
\varphi_{k, l}(m, n)
=
\frac{1}{\sqrt{MN}}
e^{-i 2 \pi \left( \frac{mk}{M} + \frac{nl}{N} \right)},
\quad
k = 0, 1, \dots, M-1,
\quad
l = 0, 1, \dots, N-1\,.
\tag{2}
$$

To compute the basis function, it is useful to use Euler's formula

$$
e^{ix} = \cos(x) + i\sin(x)\,.
$$

For each of the $MN$ output coefficients, the direct implementation sums over all $MN$ input pixels. It therefore uses four nested loops and has complexity $O(M^2N^2)$. For a square image of size $N \times N$, this is $O(N^4)$.

## Separable 2D DFT

The exponential in Equation (2) can be written as a product:

$$
e^{-i2\pi\left(\frac{mk}{M}+\frac{nl}{N}\right)}
= e^{-i2\pi\frac{mk}{M}} e^{-i2\pi\frac{nl}{N}}\,.
$$

We can therefore compute the same transform in two passes. First, transform each row and store the results in a complex intermediate matrix $G$:

$$
G(m,l) = \frac{1}{\sqrt{N}}
\sum_{n=0}^{N-1} f(m,n)e^{-i2\pi\frac{nl}{N}}\,.
\tag{3}
$$

Then, transform each column of $G$:

$$
F(k,l) = \frac{1}{\sqrt{M}}
\sum_{m=0}^{M-1} G(m,l)e^{-i2\pi\frac{mk}{M}}\,.
\tag{4}
$$

The row transform $G(m,l)$ does not depend on $k$. Computing it once and reusing it for all $k$ avoids repeated work in the direct implementation. The normalization factors multiply to $1/\sqrt{MN}$, so both implementations compute the same spectrum.

### Implementing the 1D transform

Implement a function that accepts a vector of complex numbers and returns its 1D DFT. For a vector $a$ of length $L$, use

$$
b(q) = \frac{1}{\sqrt{L}}
\sum_{p=0}^{L-1} a(p)e^{-i2\pi\frac{pq}{L}},
\quad q=0,\dots,L-1\,.
\tag{5}
$$

The function must accept complex input: the column pass operates on complex values produced by the row pass. Real image pixels can be represented as complex values with a zero imaginary part.

```text
function DFT1D(input):
    L = length(input)
    output = complex vector of length L

    for q = 0 .. L-1:
        sum = 0 + 0i
        for p = 0 .. L-1:
            angle = -2 * pi * p * q / L
            sum += input[p] * (cos(angle) + i * sin(angle))
        output[q] = sum / sqrt(L)

    return output
```

Use this function to build the 2D transform:

```text
function DFT2D_SEPARABLE(image):
    M = number of rows in image
    N = number of columns in image
    G = complex matrix of size M x N
    F = complex matrix of size M x N

    for m = 0 .. M-1:
        G[m, :] = DFT1D(image[m, :])

    for l = 0 .. N-1:
        F[:, l] = DFT1D(G[:, l])

    return F
```

Here, `A[r, :]` denotes an entire row and `A[:, c]` denotes an entire column. If your matrix representation does not support these operations, copy the row or column into a temporary vector and write the transformed values back.

A 1D DFT of length $L$ has complexity $O(L^2)$. The row pass costs $O(MN^2)$, and the column pass costs $O(NM^2)$. The total complexity is therefore

$$
O\bigl(MN(M+N)\bigr)\,.
$$

For a square image, the two passes perform $2N^3$ summation terms, giving complexity $O(N^3)$. This is still a DFT computed from its definition, not an FFT. No power-of-two dimensions are required.

## Spectrum Visualization

The result can be split into real and imaginary parts:

$$
F(k, l) = R(k, l) + iI(k, l)\,.
\tag{6}
$$

The spectrum amplitude $|F(k,l)|$ is computed as follows:

$$
|F(k, l)| = \sqrt{R^2(k, l) + I^2(k, l)}\,.
\tag{7}
$$

Compute the phase $\Phi(k,l)$ using `atan2` to handle all quadrants and coefficients whose real part is zero:

$$
\Phi(k, l) = \operatorname{atan2}\bigl(I(k,l),R(k,l)\bigr)\,.
\tag{8}
$$

The phase of a zero coefficient is undefined; use zero for its visualization.

The power spectrum $P(k,l)$ can be computed as

$$
P(k, l) = |F(k, l)|^2\,.
$$

To display the power spectrum, compute $\log(1+P(k,l))$ and then normalize the resulting values to the interval $[0,1]$. If all values are equal, display zeros to avoid division by zero during normalization.

For a conventional visualization, move the zero-frequency coefficient from the top-left corner to the center. For even dimensions, swap the first and third quadrants, and also the second and fourth quadrants. For arbitrary dimensions, use the equivalent circular shift by $\lfloor M/2\rfloor$ rows and $\lfloor N/2\rfloor$ columns toward increasing indices (`fftshift`). Apply the same shift to both the real and imaginary parts. Keep the original, unshifted complex spectrum for reconstruction; the shifted spectrum will also be useful later when applying filters.

**Hint:** Use `double` for image pixels and scalar calculations, and complex numbers with `double` real and imaginary components for the spectrum and intermediate results (for example, `std::complex<double>` in C++).

## Tasks and Verification

1. Implement the direct 2D DFT using Equations (1) and (2).
2. Implement the complex 1D DFT and use it to transform all rows, followed by all columns. Implement the summation yourself; do not use a library DFT or FFT routine.
3. Compare the complex spectra of both implementations before shifting or visualization. Report the maximum absolute difference $\max_{k,l}|F_{\mathrm{direct}}(k,l)-F_{\mathrm{separable}}(k,l)|$. Small differences caused by floating-point rounding are expected.
4. Test a small rectangular image (for example, $3 \times 5$) to check row and column indexing. Also test a constant image: for pixel value $c$, the only nonzero coefficient should be $F(0,0)=c\sqrt{MN}$, up to rounding errors.
5. Display the amplitude, phase, and logarithmic power spectrum using the separable result.
6. Measure the execution time of both transforms on square images of sizes $8$, $16$, $32$, and $64$. Exclude loading and visualization from the measurement. Report the times in a table and explain the observed growth. Larger sizes are optional because the direct implementation becomes expensive.

When the side length doubles, the theoretical work grows by a factor of $16$ for the direct implementation and $8$ for the separable implementation. Actual timings also depend on implementation details and hardware.

## Expected Output

<p align="center">
  <img src="images/dft/result_images.png" alt="Expected output" width="100%">
</p>
