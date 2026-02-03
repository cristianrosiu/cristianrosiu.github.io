---
weight: 1
title: "Singular Value Decomposition (SVD) in Computer Graphics"
date: 2026-02-01T21:29:01+08:00
# lastmod: 2020-03-06T21:29:01+08:00
draft: true
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

## Problem:
Sometimes, we acquire point cloud data without prior information about the underlying surface or mesh it was sampled from. This makes it significantly harder to derive an **orthonormal frame** (a local coordinate system) for each point.

![Aperture Animation](images/SpherePointCloud2.gif "")

An orthonormal frame is usually required to determine the overall orientation and the "flow" of the points across a surface.

- For **simple surfaces** (like spheres), we could approximate a normal vector by simply calculating the vector from the object's center to the point.
    
- For **complex meshes** (like twisting ribbons), this gets progressively more complicated. If the surface twists or bends, the simple "center-to-point" approximation fails because it assumes vectors always point "outward." It cannot account for complex curvature where faces might point inwards or where the object has no single center.

## Solutions:
A powerful solution to this problem is **Singular Value Decomposition (SVD)**. The concept of SVD is elegant in its simplicity. It states that any matrix $M$ can be decomposed into a product of three specific matrices:

$$M = U \Sigma V^T$$

Where:

- **$U$ and $V$** represent rotation or reflection transformations. Their columns contain the **left singular vectors** ($U$) and **right singular vectors** ($V$) of matrix $M$.
    
- **Orthogonality:** If matrix $M$ is real, then $U$ and $V$ are **orthogonal matrices** (meaning $U^T = U^{-1}$). This implies that their columns form orthonormal bases, so every column vector is strictly perpendicular to every other column vector in that matrix.
    
- **$\Sigma$ (or $S$)** is a **rectangular diagonal matrix**. It acts as a scaling matrix containing the **singular values** of $M$. It is "rectangular" because it keeps the dimensions of the original input, and all entries are zero except for the diagonal elements.

## Relation to Eigen decomposition:

If you work through the math, you will discover that the columns of $V$ correspond to the eigenvectors of the matrix $M^T M$, while the columns of $U$ are the eigenvectors of $M M^T$:

- Eigendecomposition generally requires **diagonalizable matrices** (square matrices that possess a full set of linearly independent eigenvectors). In contrast, SVD is universally applicable to any $M \times N$ matrix, regardless of its dimensions or properties.
    
- Eigendecomposition yields the same result as SVD specifically when the matrix is **symmetric positive semi-definite**. You can think of a **positive semi-definite matrix** as the multi-dimensional version of a "non-negative real number" (or a squared number). It represents a transformation that scales space along specific axes but never "flips" a direction into its opposite. Visually, this describes a surface shaped like a bowl or a valley that always curves upward or stays flat, ensuring that the "energy" or "variance" of the system never drops below zero.
    
- In this blog, we assume our input is a real, positive semi-definite matrix. Therefore, for our purposes, performing SVD is strictly equivalent to the eigendecomposition of the matrix (for more information regarding SVD on other types of matrices, please check the references).

## How does eigen decomposition help with our problem?:

One powerful application of eigenvectors is their ability to approximate a **best-fit plane** for a given set of points.

So, what should we choose as our matrix $M$? As previously mentioned, we require $M$ to be a positive semi-definite matrix. The perfect candidate for this is the **covariance matrix**, which inherently possesses this property.

Given a set of 3D points, the covariance matrix describes the statistical spread of the data. To be precise, **variance** measures the spread of a single dimension (e.g., how much the X values vary from the mean), while **covariance** measures how two different dimensions vary together (e.g., how X changes in relation to Y).

- The **diagonal** entries of the matrix contain the **variances** (squared deviation of each axis).
    
- The **off-diagonal** entries contain the **covariances** (correlations between axes).
    

![[Pasted image 20260202113402.png]]

The covariance matrix is also **symmetric**, meaning the elements are mirrored across the main diagonal ($M_{ij} = M_{ji}$).

![[Pasted image 20260202113515.png]]

For a set of 3D points $X$ with a mean position $\overline{X}$, the covariance matrix is calculated using the following formula:

$$\frac{1}{N}\sum_{i=0}^{N - 1} (X_i - \overline{X})(X_i - \overline{X})^T$$

### Algorithm: Finding the Best-Fit Plane via SVD

1. To approximate the orthonormal frame (local coordinate system) for a specific point, gather its $N$ nearest neighbors and calculate the covariance matrix $M$ for this subset.
    
2. As established, matrix $M$ is symmetric and positive semi-definite. When we perform SVD, the columns of $U$ correspond to the eigenvectors of $M$, ordered from the most significant (largest singular value) to the least significant.
    
3. **Frame Construction:**
    
    - The **first two columns** of $U$ (corresponding to the largest variance) define the span of the best-fit plane. These act as the tangent vectors of the surface (called tangent and binormal in curve geometry)
        
    - The **third column** of $U$ (corresponding to the least variance) represents the direction where the points vary the least. Since $U$ is an orthogonal matrix (the real-number equivalent of a unitary matrix), this third vector is perpendicular to the first two. Therefore, this third column represents the **normal** to the plane.
        

### Conclusion

One of the main reasons for writing this blog is that I found the SVD approximation of an orthonormal frame to work exceptionally well for specific surface shapes. However, there is a notable downside I could not easily resolve: the **orientation of the normal**.

Unfortunately, SVD does not guarantee a consistent sign for the normal vector (it might point "out" or "in" arbitrarily). Consequently, additional post-processing is usually required to unify the winding order.

Overall, I find this method intriguing, and I believe it serves as an excellent starting point for anyone interested in learning about eigendecomposition from a practical perspective.

Thank you for taking the time to read through my work. If you found this blog useful, don't forget to fuel my coffee addiction:

