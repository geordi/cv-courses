# Perspective Transform

In this exercise, you will implement a perspective transform that places one image into another. An example of the expected result is shown at the end of the exercise.

A perspective transform is defined by

$$
\begin{bmatrix}
\phi(x,y) \\
\psi(x,y) \\
\omega(x,y)
\end{bmatrix}
=
\begin{bmatrix}
p_{11} & p_{12} & p_{13} \\
p_{21} & p_{22} & p_{23} \\
p_{31} & p_{32} & p_{33}
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}.
\tag{1}
$$

After the transformation, $\phi(x,y)$, $\psi(x,y)$, and $\omega(x,y)$ are homogeneous coordinates. The corresponding Cartesian coordinates are

$$
\frac{\phi(x,y)}{\omega(x,y)}
\qquad \text{and} \qquad
\frac{\psi(x,y)}{\omega(x,y)}.
$$

<a id="fig-projective-transform"></a>

<p align="center">
  <img src="images/img_6_7.png" alt="Projective transformation between two images" width="70%">
</p>

*Fig. 1: Projective transformation between two images.*

Consider a projective transform defined by four corresponding point pairs.

Let $\widetilde{A}_1,\ \widetilde{A}_2,\ \widetilde{A}_3,\ \widetilde{A}_4$ be points in the input image and
$A_1,A_2,A_3,A_4$ their corresponding points in the output image. In Fig. 1, the points
$\widetilde{A}_1,\ \widetilde{A}_2,\ \widetilde{A}_3,\ \widetilde{A}_4$ are chosen as the corners of the input image.

Let the coordinates of $\widetilde{A}_i$ be $\widetilde{x}_i,\ \widetilde{y}_i$, and let the coordinates of
the corresponding point $A_i$ be $x_i,y_i$.

Define

$$
\mathbf{p}_1 =
\begin{bmatrix}
p_{11} \\
p_{12} \\
p_{13}
\end{bmatrix},
\qquad
\mathbf{p}_2 =
\begin{bmatrix}
p_{21} \\
p_{22} \\
p_{23}
\end{bmatrix},
\qquad
\mathbf{p}_3 =
\begin{bmatrix}
p_{31} \\
p_{32} \\
p_{33}
\end{bmatrix},
$$

and

$$
\mathbf{x}_i =
\begin{bmatrix}
x_i \\
y_i \\
1
\end{bmatrix}.
$$

From Eq. (1), each point correspondence gives

$$
\begin{bmatrix}
\widetilde{w}_i \widetilde{x}_i \\
\widetilde{w}_i \widetilde{y}_i \\
\widetilde{w}_i
\end{bmatrix}
=
\begin{bmatrix}
\mathbf{p}_1^\top \\
\mathbf{p}_2^\top \\
\mathbf{p}_3^\top
\end{bmatrix}
\mathbf{x}_i.
\tag{2}
$$

From the third row of Eq. (2),

$$
\widetilde{w}_i = \mathbf{p}_3^\top \mathbf{x}_i.
$$

Substituting this expression into the first two rows gives

$$
\begin{bmatrix}
\mathbf{x}_i^\top & \mathbf{0} & -\widetilde{x}_i \mathbf{x}_i^\top \\
\mathbf{0} & \mathbf{x}_i^\top & -\widetilde{y}_i \mathbf{x}_i^\top
\end{bmatrix}
\begin{bmatrix}
\mathbf{p}_1 \\
\mathbf{p}_2 \\
\mathbf{p}_3
\end{bmatrix}
=
\begin{bmatrix}
0 \\
0
\end{bmatrix}.
\tag{3}
$$

Written explicitly, Eq. (3) becomes

$$
\begin{bmatrix}
x_i & y_i & 1 & 0 & 0 & 0 &
-\widetilde{x}_i x_i & -\widetilde{x}_i y_i & -\widetilde{x}_i \\
0 & 0 & 0 & x_i & y_i & 1 &
-\widetilde{y}_i x_i & -\widetilde{y}_i y_i & -\widetilde{y}_i
\end{bmatrix}
\begin{bmatrix}
p_{11} \\
p_{12} \\
p_{13} \\
p_{21} \\
p_{22} \\
p_{23} \\
p_{31} \\
p_{32} \\
p_{33}
\end{bmatrix}
=
\begin{bmatrix}
0 \\
0
\end{bmatrix}.
\tag{4}
$$

Four point correspondences provide eight equations, but the perspective matrix contains nine coefficients. Since a projective transform is defined only up to a non-zero scale factor, one coefficient can be fixed. For example, set

$$
p_{11} = 1.
$$

Move the first column of the system in Eq. (4) to the right-hand side. The resulting system is

$$
\begin{bmatrix}
y_i & 1 & 0 & 0 & 0 &
-\widetilde{x}_i x_i & -\widetilde{x}_i y_i & -\widetilde{x}_i \\
0 & 0 & x_i & y_i & 1 &
-\widetilde{y}_i x_i & -\widetilde{y}_i y_i & -\widetilde{y}_i
\end{bmatrix}
\begin{bmatrix}
p_{12} \\
p_{13} \\
p_{21} \\
p_{22} \\
p_{23} \\
p_{31} \\
p_{32} \\
p_{33}
\end{bmatrix}
=
\begin{bmatrix}
-x_i \\
0
\end{bmatrix}.
\tag{5}
$$

Using all four point correspondences produces an $8 \times 8$ linear system that can be solved for the remaining eight coefficients.


## Implementation Details

Use the following point correspondences, listed clockwise starting from the top-left corner:

- **Flag:** $(0,0)$, $(323,0)$, $(323,215)$, $(0,215)$
- **Building:** $(69,107)$, $(227,76)$, $(228,122)$, $(66,134)$

Use OpenCV's `cv::solve` function to solve the system of linear equations.

When generating the transformed image, remember that the transformed coordinates are generally real-valued. Use an appropriate interpolation method when sampling the source image.


## Expected Output

<p align="center">
  <img src="images/persp_proj.png" alt="Input images and perspective-transform result" width="80%">
</p>

*Fig. 2: Input images (top left and bottom); output image after the perspective transform (top right).*
