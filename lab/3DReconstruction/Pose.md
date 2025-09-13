## Ground Truth Pose Acquisition Methods

| 方法 Method                                              | 场景 Scene                                           | 原理 Principle                                                                                                                               | 优点 Advantages                                                                                                                   | 缺点 Disadvantages                                                                                                             |
| ------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **GNSS (GPS) + RTK (Real-Time Kinematic)**             | 室外 Outdoor (KITTI, SemanticKITTI)                  | GNSS 提供全局坐标，RTK 通过差分修正误差，可达厘米级精度。<br>GNSS gives global position, RTK corrects error using reference stations, achieving cm-level accuracy. | - 全球可用<br>- 高精度（室外无遮挡时）<br>- 和 IMU 融合可补偿短时丢星<br>- Globally available<br>- High accuracy outdoors<br>- IMU fusion for robustness | - 室内/地下无法使用<br>- 城市峡谷信号差<br>- 需要基站支持<br>- Not usable indoors/underground<br>- Poor in urban canyons<br>- Needs base stations |
| **高精度 IMU + 轮速计 / 里程计** <br> High-grade IMU + Odometry | 室外 Outdoor, 车辆平台                                   | 测量加速度和角速度，结合轮速计积分得到位姿。<br>Measure acceleration & angular velocity, fuse with wheel odometry.                                               | - 高频输出<br>- 对短时动态精度高<br>- Works in GPS-denied areas short-term                                                                  | - 漂移严重（误差随时间积累）<br>- 无法长期独立使用<br>- Strong drift over time<br>- Not reliable alone                                            |
| **运动捕捉系统 (Motion Capture, Vicon, OptiTrack)**          | 室内 Indoor (TUM RGB-D, 7-Scenes)                    | 多个外部摄像机追踪安装在设备上的反光标记点，三角定位得到 6DoF。<br>Multiple IR cameras track reflective markers, triangulate 6DoF pose.                                 | - 亚毫米级精度<br>- 高帧率实时输出<br>- Very high accuracy (sub-mm)<br>- Real-time                                                           | - 只能在实验室环境<br>- 成本高<br>- Not scalable beyond lab<br>- Expensive                                                              |
| **激光雷达地图配准 (Lidar SLAM with offline optimization)**    | 室内/室外 Indoor/Outdoor (Newer datasets, e.g. MulRan) | 使用高精度 3D Lidar 构建点云地图，离线全局优化（pose graph, bundle adjustment）。<br>Use Lidar scans to build global map, refine with offline optimization.     | - 可在大范围环境获得较好精度<br>- 不依赖 GPS<br>- Works indoors/outdoors<br>- Good large-scale accuracy                                         | - 成本高（需要高端 Lidar）<br>- 仍依赖离线优化<br>- Expensive sensors<br>- Offline processing required                                       |
| **人工标定 (手工对齐, calibration rigs)**                      | 小规模 Indoor scenes                                  | 人工设置基准物体（棋盘格、已知几何场景），用几何对齐方式获得相机位姿。<br>Place calibration objects (checkerboards, known geometry), compute camera pose.                     | - 简单直接<br>- 适合小场景<br>- Simple & low-cost<br>- Works for small scenes                                                            | - 无法扩展到大规模场景<br>- 易受人工误差影响<br>- Not scalable<br>- Prone to human error                                                       |
| **模拟数据 (Synthetic datasets)**                          | 室内/虚拟 Virtual scenes (ICL-NUIM, Replica, Habitat)  | 使用渲染引擎，直接输出虚拟相机的精确位姿。<br>Use rendering engine, directly output ground-truth camera pose.                                                   | - 精确无误差<br>- 可生成大规模数据<br>- Perfect ground truth<br>- Scalable                                                                   | - 与真实传感器 gap 大<br>- 模拟噪声需额外建模<br>- Gap from real sensors<br>- Needs simulated noise                                          |

### Lidar SLAM with Offline Optimization
1. Local Map Creation: frame-to-map ICP
    - 维护局部点云子地图（由最近N帧点云融合）
    - 新帧$P_t$与子地图匹配
2. Pose Graph Optimization
    把每个帧（or关键帧）的位姿作为一个节点：
    - 节点（Node）：传感器在某一时刻的位姿$T_i\in SE(3)$
    - 边（Edge）：两个节点之间的相对约束（ICP给出的相对位姿变换）
    （1）邻接约束（Odometry edges）
        - 相邻帧之间的相对位姿，通过ICP / scan matching获得。
    （2）回环约束（Loop closure edges）
        - 通过检测闭环，添加回环约束（建立一条回环边）
        - 提供了“全局约束”，防止轨迹无限drift
    每条边$e_{ij}$表示观测的相对位姿
    $$
        \hat{T}_{ij} = T_i^{-1} T_j
    $$
    优化目标是最小化所有边的重投影误差：
    $$
    \min \sum_{(i,j)\in E} \| \hat{T}_{ij} - T_{ij} \|^2
    $$
3. Loop Closure Detection
    （1）检测方法：
        - 几何方法：当前点云与history submap匹配（ICP, scan context, NDT）
        - 外观方法：基于描述符（Scan Context, M2DP）做全局场景匹配。
    （2）建立约束
        - 成功匹配后，计算相对位姿，添加回环边到pose graph。（以修正累计drift）
4. Offline Global Optimization
    - 通过全局优化算法（如 g2o, Ceres Solver, GTSAM）对位姿图进行优化，减小整体误差。
    将问题转换为图优化 Pose Graph Optimization (PGO)
    $$
    \min_{T_i} \sum_{(i,j)\in E} \| \hat{T}_{ij} - T_{ij} \|^2_{\sum_{ij}}
    $$
    其中：
    - $\hat{T}_{ij}$：ICP得到的相对位姿
    - $T_{i}$：优化变量，每个节点的绝对位姿
    - \sum_{ij}：边的置信度
5. Map Reconstruction
    - 通过优化后的位姿图重建稠密地图。

### Inertial Measurement Unit (IMU)
**两个核心传感器组成**：
- 加速度计（accelerometer）：测量线性加速度 $(a_x, a_y, a_z)$
- 陀螺仪（gyroscope）：测量角速度 $(\omega_x, \omega_y, \omega_z)$
- 磁力计（magnetometer）：估计绝对航向角 $(m_x, m_y, m_z)$

IMU可以通过积分得到velocity, position and orientation。
$$
v(t) = \int a(t) dt\\
p(t) = \int v(t) dt\\
R(t) = \int \omega(t) dt
$$

## 实际应用pose estimation的必要性
1.  lab内用**外部高精度跟踪系统**

    - 用 外部高精度跟踪系统（如 motion capture、激光 tracker、全站仪）获得 ground truth pose。

    - 这些 setup 复杂昂贵，不适合大规模移动场景。
2. 普通RGB-D传感器(Kinect, RealSense):
    - 相机本身只输出 RGB 图像和 depth，不提供全局精确位姿。
    - 内置的 **IMU ((Inertial Measurement Unit, 惯性测量单元))** 可以辅助，但 drift 依然存在。
3. 大规模场景 (真实应用)：
    - 没有 GT pose，只能通过 **SLAM/VO 算法** 去估计。

#### Inertial Measurement Unit (IMU)
**两个核心传感器组成**：
- 加速度计（accelerometer）：测量线性加速度 $(a_x, a_y, a_z)$
- 陀螺仪（gyroscope）：测量角速度 $(\omega_x, \omega_y, \omega_z)$
- 磁力计（magnetometer）：估计绝对航向角 $(m_x, m_y, m_z)$

IMU可以通过积分得到velocity, position and orientation。
$$
v(t) = \int a(t) dt\\
p(t) = \int v(t) dt\\
R(t) = \int \omega(t) dt
$$

**IMU产生drift的原因**：误差会随积分累积放大。
1. 噪声与偏置
    - 加速度计和陀螺仪的输出中存在 噪声 (noise) 和 零偏 (bias)。

    - 这些小误差在积分过程中不断累积，导致位置/姿态估计快速漂移。

    - e.g.: 陀螺仪零偏 0.01 °/s，看起来很小，但积分一分钟后会导致 0.6° 姿态误差。加速度计零偏 0.01 m/s²，积分两次会导致位置估计在几十秒内就偏差数米。
2. 双重积分放大误差
    - 姿态误差会导致加速度投影到世界坐标系时错误，从而进一步放大位置误差。

    - 位置是 加速度的二重积分，即使是极小的测量误差也会迅速积累成很大的漂移。

3. 没有外部参照
    - IMU 本质上是 相对运动传感器，它只能估计相对于起点的运动。

    - 没有 GPS、视觉或地图约束时，IMU 无法校正自身漂移。

#### 现有带pose的公开数据集
- KITTI / SemanticKITTI (自动驾驶)

    - 车顶安装 高精度 GNSS (GPS) + IMU + RTK (Real-Time Kinematic) 定位系统。

    - GNSS 提供全局坐标，RTK 能把误差压到厘米级。

    - IMU 用来补偿短时高频运动。

    - 最终经过 离线优化 (sensor fusion / smoothing) 得到接近 ground truth 的轨迹。

- TUM RGB-D, ICL-NUIM, 7-Scenes (室内)

    - 室内没有 GPS，所以常用 外部高精度跟踪系统 (motion capture, Vicon, OptiTrack)。

    - 这些系统通过布置摄像头阵列，追踪安装在 RGB-D 相机上的 反光标志点，从而得到亚毫米级的 6DoF pose。

    - 还有一些数据集（如 ICL-NUIM）是 模拟数据 (synthetic)，直接输出真实 pose。

**实际应用需要用SLAM/VO估计pose的必要性**：
1. 数据集的 pose ≈ 通过昂贵外部设备（GNSS+RTK，motion capture）或仿真直接提供的 GT。
2. 实际应用 ≈ 没有这些外部条件，只能依赖传感器自身 (RGB-D, Lidar, IMU) 做 online estimation。
3. 因此实际系统不可避免要面对 drift。