# Inverse Discrete Fourier Transform

In the previous lesson, we've implemented the Discrete Fourier Transform. Today, we'll implement its inverse called the Inverse Discrete Fourier Transform (IDFT).

We want to transform the frequency spectrum $F$ back to its spatial domain $f$, which is an image.
The computation of the IDFT can be easily implemented using following formula

$$
f(m, n) = \sum\limits_{k=0}^{M-1} \sum\limits_{l=0}^{N-1} F(k, l) \varphi_{k, l}(m, n)\,.
\tag{1}
$$

The basis $\varphi_{k, l}$ is defined as

$$
\varphi_{k, l}(m, n) = \frac{1}{\sqrt{MN}} e^{i 2 \pi \left( \frac{mk}{M} + \frac{nl}{N} \right) }, \quad k = 0, 1, \dots, M-1 \quad \mathrm{a} \quad l = 0, 1, \dots, N-1\,.
\tag{2}
$$

Notice that argument of $e$ is positive, so it's different from the basis used in DFT.

To compute the basis, it's advantageous to use Eulers formula $e^{ix} = \cos(x) + i \sin(x)$.

Don't forget that elements in the matrix $F$ are complex, so is the basis. Think for a while, what will be the result.

**Hint:** Use `double` data type as last time.
