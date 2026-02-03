---
weight: 1
title: "Singular Value Decomposition (SVD) in Computer Graphics"
date: 2026-02-01T21:29:01+08:00
# lastmod: 2020-03-06T21:29:01+08:00
draft: false
author: "Cristian"

resources:
- name: featured-image
  src: images/thumbnail.png

tags: ["rendering", "depth-of-field", "circle-of-confusion"]
categories: ["blogs"]

rssFullText: true

lightgallery: true

math: true

toc:
  enable: true
  auto: true

share:
  enable: true
---

How do you find structure in a cloud of scattered points? In this post, we examine the utility of Singular Value Decomposition (SVD) in solving this geometric problem, focusing on how to extract the best-fit plane around a specific point in a dataset.

<!--more-->

## Problem
Sometimes, we acquire point cloud data without prior information about the underlying surface or mesh it was sampled from. This makes it significantly harder to derive an **orthonormal frame** (a local coordinate system) for each point.

An orthonormal frame is usually required to determine the overall orientation and the "flow" of the points across a surface.

- For **simple surfaces** (like spheres), we could approximate a normal vector by simply calculating the vector from the object's center to the point.
    
- For **complex meshes** (like twisting ribbons), this gets progressively more complicated. If the surface twists or bends, the simple "center-to-point" approximation fails because it assumes vectors always point "outward." It cannot account for complex curvature where faces might point inwards or where the object has no single center.

![point-cloud-animation](images/SpherePointCloud2.gif "Visualization of a point cloud sampled from a spherical surfaces.")

## Solutions
A powerful solution to this problem is **Singular Value Decomposition (SVD)**. The concept of SVD is elegant in its simplicity. It states that any matrix $M$ can be decomposed into a product of three specific matrices[^1]:

$$M = U \Sigma V^T$$

Where:

- **$U$ and $V$** represent rotation or reflection transformations. Their columns contain the **left singular vectors** ($U$) and **right singular vectors** ($V$) of matrix $M$.
    
- **Orthogonality:** If matrix $M$ is real, then $U$ and $V$ are **orthogonal matrices** (meaning $U^T = U^{-1}$). This implies that their columns form orthonormal bases, so every column vector is strictly perpendicular to every other column vector in that matrix.
    
- **$\Sigma$ (or $S$)** is a **rectangular diagonal matrix**. It acts as a scaling matrix containing the **singular values** of $M$. It is "rectangular" because it keeps the dimensions of the original input, and all entries are zero except for the diagonal elements.

## Relation to Eigen decomposition:

If you work through the math, you will discover that the columns of $V$ correspond to the eigenvectors of the matrix $M^T M$, while the columns of $U$ are the eigenvectors of $M M^T$ [^1]:

Eigendecomposition generally requires **diagonalizable matrices** (square matrices that possess a full set of linearly independent eigenvectors). In contrast, SVD is universally applicable to any $M \times N$ matrix, regardless of its dimensions or properties[^1].

However, if matrix $M$ is symmetric with real values, and has only positive eigenvalues ($\lambda_i$), then it follows that eigenvalues and singular values are equal: $M = U \Sigma V^T = QDQ^{-1} = QDQ^T$. This comes from the following properties[^2]:

1. **Symmetry aligns the input and output axes ($U = V$)**: In a general SVD ($M = U \Sigma V^T$), the input basis ($V$) and the output basis ($U$) are usually different. The matrix rotates a vector from one coordinate system, stretches it, and lands it in a different coordinate system.
However, because the matrix is symmetric, its eigenvectors (collumns of U, V and Q) form orthogonal bases that works for both the domain and the range, therfore, it results that $Q^{-1} = Q^T \implies QDQ^{-1} = QDQ^T \implies M = USV^T = QDQ^{-1} = QDQ^T$. The "input axes" and "output axes" are identical. This forces the left singular vectors ($U$) and right singular vectors ($V$) to be the same (up to a sign), meaning $U = V = Q$[^2].
    
2. **Positive Semi-Definiteness keeps the signs ($\Sigma = \Lambda$)**:
Symmetry alone isn't enough to make SVD and Eigendecomposition identical. If a symmetric matrix had a negative eigenvalue (e.g., $\lambda = -5$), the Eigendecomposition would keep the $-5$. But SVD requires singular values ($\sigma$) to be non-negative. SVD would handle this by taking the absolute value ($\sigma = |-5| = 5$) and "hiding" the negative sign inside one of the singular vectors. If $M$ is Positive Semi-Definite, all eigenvalues are already non-negative ($\lambda \ge 0$). Therefore, no sign adjustment is needed. The scaling factors are identical: $\sigma_i = \lambda_i$[^2].

## How does eigen decomposition help with our problem?:

One powerful application of eigenvectors is their ability to approximate a **best-fit plane** for a given set of points.

So, what should we choose as our matrix $M$? As previously mentioned, we require $M$ to be a positive semi-definite matrix. The perfect candidate for this is the **covariance matrix**, which inherently possesses this property[^3].

Given a set of 3D points, the covariance matrix describes the statistical spread of the data. To be precise, **variance** measures the spread of a single dimension (e.g., how much the X values vary from the mean), while **covariance** measures how two different dimensions vary together (e.g., how X changes in relation to Y).

- The **diagonal** entries of the matrix contain the **variances** (squared deviation of each axis).
    
- The **off-diagonal** entries contain the **covariances** (correlations between axes).

$$
\begin{bmatrix}
var(x) & cov(x,y) & cov(x,z)\\
cov(x,y) & var(y) & cov(y,z)\\
cov(x,z) & cov(y,z) & var(z)
\end{bmatrix}    
$$

The covariance matrix is also **symmetric**, meaning the elements are mirrored across the main diagonal ($M_{ij} = M_{ji}$)[^3].

For a set of points $X$ in $R^3$ with a mean position $\overline{X}$, the covariance matrix is calculated using the following formula:

$$\frac{1}{N}\sum_{i=0}^{N - 1} (X_i - \overline{X})(X_i - \overline{X})^T$$

### Algorithm: Finding the Best-Fit Plane via SVD

1. To approximate the orthonormal frame (local coordinate system) for a specific point, gather its $N$ nearest neighbors and calculate the covariance matrix $M$ for this subset.
    
2. As established, matrix $M$ is symmetric and positive semi-definite. When we perform SVD, the columns of $U$ correspond to the eigenvectors of $M$, ordered from the most significant (largest singular value) to the least significant.
    
3. **Frame Construction:**
    
    - The **first two columns** of $U$ (corresponding to the largest variance) define the span of the best-fit plane. These act as the tangent vectors of the surface (called tangent and binormal in curve geometry)
        
    - The **third column** of $U$ (corresponding to the least variance) represents the direction where the points vary the least. Since $U$ is an orthogonal matrix (the real-number equivalent of a unitary matrix), this third vector is perpendicular to the first two. Therefore, this third column represents the **normal** to the plane.

![point-cloud-animation](images/3dsvd-animation.gif "Visualization of the best-fit plane algorithm. For a selected target point (green), we identify the $N$ nearest neighbors (yellow). A covariance matrix is computed from this local subset, and SVD is applied to derive a coherent orthonormal frame. The resulting best-fit plane is shown in blue.")

## Implementation Details

While analytic solutions exist for $3 \times 3$ matrices, numerical iterative methods are preferred in practice due to their stability. Two of the most common algorithms for computing SVD are:

1. **Power Iteration**: This method finds the dominant eigenvector (the direction of largest variance) by repeatedly multiplying a random vector by the covariance matrix. It is computationally cheap and easy to implement. It is best suited for real-time applications where you only need the primary axes or a quick approximation[^4].
2. **Jacobi Method**: This approach applies a sequence of rotations (Givens rotations) to iteratively zero out the off-diagonal elements of the matrix until it becomes diagonal. It is more computationally intensive but offers higher precision and numerical stability. It is often the choice for offline processing or when the full, accurate orthonormal frame is strictly required[^5].

## Conclusion

One of the main reasons for writing this blog is that I found the SVD approximation of an orthonormal frame to work exceptionally well for specific surface shapes. However, there is a notable downside I could not easily resolve: the **orientation of the normal**.

Unfortunately, SVD does not guarantee a consistent sign for the normal vector (it might point "out" or "in" arbitrarily). Consequently, additional post-processing is usually required to unify the winding order.

Overall, I find this method intriguing, and I believe it serves as an excellent starting point for anyone interested in learning about eigendecomposition from a practical perspective. Thank you for taking the time to read through my work. If you found this blog useful, don't forget to fuel my [coffee addiction](https://www.paypal.com/donate/?hosted_button_id=TQA9LV2HACZDN)!

## References
[^1]: [Singular Value Decomposition - Wikipedia](https://en.wikipedia.org/wiki/Singular_value_decomposition)
[^2]: [Fundamentals of Computer Graphics, By Steve Marschner, Peter Shirley](https://www.amazon.com/Fundamentals-Computer-Graphics-Steve-Marschner/dp/0367505037)
[^3]: [Covariance Matrix - Wikipedia](https://en.wikipedia.org/wiki/Covariance_matrix)
[^4]: [Power Iteration - Wikipedia](https://en.wikipedia.org/wiki/Power_iteration)
[^5]: [Jacobi Eigenvalue Algorithm - Wikipedia](https://en.wikipedia.org/wiki/Jacobi_eigenvalue_algorithm)