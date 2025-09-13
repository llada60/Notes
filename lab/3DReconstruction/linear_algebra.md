[TOC]

## Hilbert Space, Dot and Cross Products

- **Hilbert Space**: A complete inner product space that generalizes the notion of Euclidean space. Euclidean spaces are finite-dimensional, while Hilbert spaces can be infinite-dimensional.

- **Dot Product**: used to measure the similarity between two vectors. Particularly in 3D data from multiple cameras or a Kinect sensor.

- **Cross Product**: used to find normal vector to a plane. Especially useful in aligning 3D point clouds (images with depth information). This method is extensively used in self driving cars to make a map using LIDAR scans.

### Point-to-Point ICP

minimizes the distance between corresponding points.

$$
\min \sum_{i=1}^{N} \| P_i - (R \cdot Q_i + t) \|^2
$$

The optimal (R,t) has a closed-form solution via SVD:

1. Compute the centroids of $\{p_i\}, \{q_i\}$
2. Subtract centroids to get centered point sets.

3. Form the cross-covariance matrix $H = \sum (P_i - \bar{P})(Q_i - \bar{Q})^T$
4. Compute the SVD of $H = U \Sigma V^T$

5. The rotation matrix is given by $R = V U^T$ and the translation vector is $t = \bar{P} - R \bar{Q}$.

This is the Umeyama method.

### Point-to-Plane ICP
**Reason**: Scanned objects are surfaces and not individual distinct points.

**Point-to-plane ICP**: considers the surface normal of the target scan.

$$
\min \sum_{i=1}^{N} \| (P_i - (R \cdot Q_i + t)) \cdot n_i \|^2
$$

faster convergence, but require surface normal and sensitive to the normal estimation error.

it breaks the direct ICP solution based on SVD. And requires least squares approach to solve. Typically, we use Gauss-Newton approach iteratively compute.

## Eigenvalues and Eigenvectors

we have a vector $v\in R^n$. A linear transformation of $v$ is given by a matrix A multiplied by $v$. One could have a special vector $v$ such that the $Av$ returns a scaled version of v, i.e., the direction of the $v$ is maintained upon a linear transformation by A.

$$
Av = \lambda v
$$

solve the equation as follows

$$
(A - \lambda I)v = 0
$$

Once the equation is solved, one would find n pairs of $\lambda_i, v_i$. These set of $\lambda_i$ values are called **eigenvalues** and the corresponding $v_i$ values are called **eigenvectors**.

If we have a matrix Q whose columns are eigenvectors, i.e.,

$$
Q = [v_1, v_2, \ldots, v_n]
$$

consider the fact that $Av=\lambda v$

$$
AQ = [\lambda_1 v_1, \lambda_2 v_2, \ldots, \lambda_n v_n]
$$

This can be re-written as 

$$
AQ = [v_1, v_2, \ldots, v_n]\Lambda
$$

here $\Lambda$ is a diagonal matrix with $\Lambda_{ii} = \lambda_i$. We also know that columns of Q are linearly independent, this means that Q is invertible.

$$
A = Q\Lambda Q^{-1}
$$

The above is called **Eigen Decomposition**. And this is very common used in Principal Component Analysis (PCA). PCA is used to find the most important linearly independent basis of a given data.

The intuition to Eigen-decomposition. The eigenvalues represent the covariance and eigenvectors represent the directions of variation in data.

## Singular Value Decomposition (SVD)

SVD can be seen as a generalization of Eigen Decomposition. While Eigen Decomposition is applicable to square matrices, SVD can be applied to any $A_{m\times n}$

The SVD of a matrix \( A \) is given by:

$$
A = U \Sigma V^T
$$

where:
- **$U$**: is an \( m $\times$ m \) square orthonormal (unit normal) basis function.
- **$\Sigma$**: is an \( m $\times$ n \) diagonal matrix whose diagonal entries are the singular values.
- **$V^T$**: is the transpose of an \( n $\times$ n \) orthonormal basis function as well.

The $U, V^T$ as matrices which rotate the data. $\Sigma$ acts as a scaling matrix.

note that for SVD to be valid $A$ has to be Positive Semi-Definite (PSD), i.e., $A \succeq 0$ or all the eigenvalues have to be non-negative.

- $U$ (left singular values) of A gives us the eigenvectors of $AA^T$
- $V$ (right singular values) gives us the eigenvectors of $A^TA$. 
- The non-zero singular values of A (found on diagonal entries of $\Sigma$) are the square roots of non-zero eigenvalues of $A^T A$ (or $AA^T$).