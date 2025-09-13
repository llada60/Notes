## Perspective-n-Points (PnP)

- 已知 **n 个三维点** $Xi=(Xi,Yi,Zi)$ 在 **世界坐标系**中的坐标；
- 已知它们在相机成像平面上的 **二维投影点** $xi=(ui,vi)$；
- 已知相机的 **内参矩阵** K（焦距、主点等）。

目标：
 估计相机的 **外参** (R,t)，即相机坐标系相对于世界坐标系的旋转矩阵 R 和平移向量 t。

数学上，投影关系是：
$$
s  \begin{bmatrix} u_i \\ v_i \\ 1 \end{bmatrix}  = K \, [R \mid t]  \begin{bmatrix} X_i \\ Y_i \\ Z_i \\ 1 \end{bmatrix}
$$
其中 s 是尺度因子（深度）。

#### 约束条件

- 至少需要 **3 个点 (n ≥ 3)** 才能求解（称为 P3P 问题）。
- 当 n>3 时，就成为 **超定方程组**，通常采用最小二乘、RANSAC 或非线性优化（如 BA：Bundle Adjustment）来求解。