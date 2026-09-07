# Convolution

In this exercise, we will implement a convolution algorithm. Convolution is a mathematical operation that combines two functions to produce a third function. In digital image processing, it is commonly used to implement various image filters.

In digital image processing, convolution usually takes the following form:

\\[
(f * h)(x, y) = \sum\limits_{i=-k}^{k} \sum\limits_{j=-k}^{k} f(x - i, y - j) \cdot h(i, j) \\, ,
\tag{1}
\\]

where \\(f\\) is and input image, \\(h\\) is a convolution matrix (mask), and \\(k\\) is the width of the convolution mask.

Convolution mask is a matrix usually of size \\(3 \times 3\\) or \\(5 \times 5\\). Some examples of convolution masks follow:

\\[
\text{Box blur:} \quad\quad \frac{1}{9}
\\begin{bmatrix}
    1 & 1 & 1 \\\\
    1 & 1 & 1 \\\\
    1 & 1 & 1
\\end{bmatrix}
\\, ,
\tag{2}
\\]

\\[
\text{Gaussian blur } 3 \times 3\text{:} \quad\quad \frac{1}{16}
\begin{bmatrix}
    1 & 2 & 1 \\\\
    2 & 4 & 2 \\\\
    1 & 2 & 1
\end{bmatrix}
\\, ,
\tag{3}
\\]

\\[
\text{Gaussian blur } 5 \times 5\text{:} \quad\quad \frac{1}{256}
\begin{bmatrix}
    1 & 4  & 6  & 4  & 1 \\\\
    4 & 16 & 24 & 16 & 4 \\\\
    6 & 24 & 36 & 24 & 6 \\\\
    4 & 16 & 24 & 16 & 4 \\\\
    1 & 4  & 6  & 4  & 1
\end{bmatrix}
\\, .
\tag{4}
\\]

Informally, convolution computes each output pixel as a weighted sum of the neighbouring pixels in the input image. The weights are given by the corresponding values of the convolution kernel.

When the kernel is centred close to the image boundary, some of its elements extend beyond the image. Without introducing an additional boundary-handling strategy, convolution therefore cannot be computed for these pixels. The width of this border depends on the kernel size. For a \\(3 \times 3\\) kernel, the border is \\(1\\) pixel wide, while for a \\(5 \times 5\\) kernel, it is \\(2\\) pixels wide.

Fig. 1 illustrates the convolution operation at a particular pixel location.

<a id="fig-convolution-example"></a>

![An example of convolution operation at a pixel location.](images/conv.png)

*Fig. 1: An example of convolution operation at a pixel location.*
