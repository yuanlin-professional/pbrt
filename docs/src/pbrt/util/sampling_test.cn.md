# sampling_test.cpp

## 概述
该测试文件是 PBRT-v4 采样模块的综合测试，覆盖面非常广泛。测试内容包括：离散采样、各种几何形状的采样及其逆变换（半球、球面、三角形、锥体、圆盘）、分段常数一维/二维分布、球面三角形和四边形采样、多种参数分布的采样与逆采样（SmoothStep、线性、帐篷、Catmull-Rom、双线性、Logistic、指数、正态）、方差估计器、加权水库采样、均匀/分层/Hammersley 采样生成器、别名表、累积面积表，以及 Sobol 低差异序列。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(SampleDiscrete, Basics) | 测试离散采样函数 `SampleDiscrete` 的基本功能：单元素、等权重双元素采样，验证返回的索引、PDF 和重映射 u 值 |
| TEST(SampleDiscrete, VsPiecewiseConstant1D) | 将 `SampleDiscrete` 的结果与 `PiecewiseConstant1D` 分布的采样结果进行交叉验证，确保一致性 |
| TEST(Sampling, InvertUniformHemisphere) | 验证均匀半球采样的逆变换：采样后再逆变换应恢复原始均匀随机数 |
| TEST(Sampling, InvertCosineHemisphere) | 验证余弦加权半球采样的逆变换正确性 |
| TEST(Sampling, InvertUniformSphere) | 验证均匀球面采样的逆变换正确性 |
| TEST(Sampling, InvertUniformTriangle) | 验证均匀三角形采样的逆变换正确性 |
| TEST(Sampling, InvertUniformCone) | 验证均匀锥体采样的逆变换正确性（使用随机 cosThetaMax） |
| TEST(Sampling, InvertUniformDiskPolar) | 验证极坐标均匀圆盘采样的逆变换正确性 |
| TEST(Sampling, InvertUniformDiskConcentric) | 验证同心映射均匀圆盘采样的逆变换正确性 |
| TEST(LowDiscrepancy, RadicalInverse) | 验证基数逆函数 `RadicalInverse` 与位反转 `ReverseBits32` 的一致性 |
| TEST(LowDiscrepancy, SobolFirstDimension) | 验证 Sobol 序列第一维与标准基数 2 逆序列一致 |
| TEST(Sobol, IntervalToIndex) | 系统测试 `SobolIntervalToIndex`：遍历多个分辨率级别，验证映射后的采样点落在正确像素范围内 |
| TEST(Sobol, IntervalToIndexRandoms) | 使用 100000 个随机参数测试 `SobolIntervalToIndex` 的正确性 |
| TEST(PiecewiseConstant1D, Continuous) | 测试一维分段常数分布的连续采样：验证端点、中点的采样值和 PDF |
| TEST(PiecewiseConstant1D, Range) | 测试带自定义范围的一维分段常数分布，与解析二次方程的精确解进行对比 |
| TEST(PiecewiseConstant1D, InverseUniform) | 测试均匀分布的 CDF 逆变换功能，包括默认范围和自定义范围 |
| TEST(PiecewiseConstant1D, InverseGeneral) | 测试非均匀分布的 CDF 逆变换功能，验证多个特定值点 |
| TEST(PiecewiseConstant1D, InverseRandoms) | 使用随机均匀数测试采样-逆变换往返一致性 |
| TEST(PiecewiseConstant1D, Integral) | 验证分段常数分布积分值的正确性，覆盖默认范围和多种自定义范围 |
| TEST(PiecewiseConstant2D, InverseUniform) | 测试二维均匀分段常数分布的逆变换，包括默认域和自定义域 |
| TEST(PiecewiseConstant2D, InverseRandoms) | 使用随机权重测试二维分布的采样-逆变换往返一致性 |
| TEST(PiecewiseConstant2D, FromFuncLInfinity) | 测试从函数采样构建的二维分布与精确分布的 L-infinity 范数比较 |
| TEST(PiecewiseConstant2D, Integral) | 验证二维分段常数分布在不同分辨率和域配置下的积分值 |
| TEST(Sampling, SphericalTriangle) | 通过蒙特卡洛积分比较球面三角形采样与面积采样的结果一致性 |
| TEST(Sampling, SphericalTriangleInverse) | 使用随机三角形和观察点测试球面三角形采样逆变换的精度 |
| TEST(Sampling, SphericalQuad) | 通过蒙特卡洛积分比较球面四边形采样与面积采样的结果一致性 |
| TEST(Sampling, SphericalQuadInverse) | 使用随机观察点测试球面矩形采样逆变换的精度 |
| TEST(Sampling, SmoothStep) | 测试 SmoothStep 分布的采样、PDF 计算和逆变换，与分段常数近似分布交叉验证 |
| TEST(Sampling, Linear) | 测试线性分布采样的均匀性、与分段常数分布的一致性，以及逆变换正确性 |
| TEST(Sampling, Tent) | 测试帐篷分布采样的中点分层保持、不同半径下与分段常数分布的一致性，以及逆变换 |
| TEST(Sampling, CatmullRom) | 测试 Catmull-Rom 样条分布的采样和 PDF，与分段常数近似分布交叉验证 |
| TEST(Sampling, Bilinear) | 测试双线性分布的二维采样、PDF 计算和逆变换，与分段常数二维分布交叉验证 |
| TEST(Sampling, Logistic) | 测试截断 Logistic 分布在多组参数下的采样、PDF 和逆变换正确性 |
| TEST(Sampling, TrimmedExponential) | 测试截断指数分布在多组参数下的采样、PDF 和逆变换正确性 |
| TEST(Sampling, Normal) | 测试正态分布在多组 (mu, sigma) 参数下的采样、PDF 和逆变换正确性 |
| TEST(VarianceEstimator, Zero) | 验证方差估计器对常数输入返回零方差 |
| TEST(VarianceEstimator, VsClosedForm) | 将方差估计器的结果与 [-1,1] 上均匀分布的理论方差 1/3 进行比较 |
| TEST(VarianceEstimator, Merge) | 测试将多个方差估计器合并后结果与理论值的一致性 |
| TEST(VarianceEstimator, MergeTwo) | 测试两个方差估计器合并后与单个估计器处理所有数据的结果一致性 |
| TEST(WeightedReservoir, Basic) | 通过百万次采样验证加权水库采样器的概率与权重成正比 |
| TEST(WeightedReservoir, MergeReservoirs) | 测试合并两个加权水库采样器后采样概率仍与权重成正比 |
| TEST(Generators, Uniform1D) | 验证一维均匀采样生成器产生正确数量且在 [0,1) 范围内的样本 |
| TEST(Generators, Uniform1DSeed) | 验证不同种子的一维均匀生成器产生不同的样本序列 |
| TEST(Generators, Uniform2D) | 验证二维均匀采样生成器产生正确数量且在 [0,1)^2 范围内的样本 |
| TEST(Generators, Uniform2DSeed) | 验证不同种子的二维均匀生成器产生不同的样本序列 |
| TEST(Generators, Uniform3D) | 验证三维均匀采样生成器产生正确数量且在 [0,1)^3 范围内的样本 |
| TEST(Generators, Stratified1D) | 验证一维分层采样生成器的样本落在对应的分层区间内 |
| TEST(Generators, Stratified2D) | 验证二维分层采样生成器的样本落在对应的二维分层网格内 |
| TEST(Generators, Stratified3D) | 验证三维分层采样生成器的样本落在对应的三维分层网格内 |
| TEST(Generators, Hammersley2D) | 验证二维 Hammersley 序列生成器的第一维为均匀分布、第二维为基数逆函数 |
| TEST(Generators, Hammersley3D) | 验证三维 Hammersley 序列生成器的三个维度分别为均匀、基数逆（基数2）、基数逆（基数3） |
| TEST(AliasTable, Uniform) | 测试等权重别名表的采样概率均匀性和 PDF 正确性 |
| TEST(AliasTable, Varying) | 测试随机权重别名表的采样概率与权重成正比，PDF 与理论值一致 |
| TEST(SummedArea, Constant) | 测试常数值累积面积表在多种矩形区域上的积分值正确性 |
| TEST(SummedArea, Rect) | 测试递增值矩形数组的累积面积表，遍历所有对齐格子的子矩形验证积分 |
| TEST(SummedArea, Randoms) | 使用多种尺寸的随机值数组，验证累积面积表对随机矩形区域的积分精度 |
| TEST(SummedArea, NonCellAligned) | 测试累积面积表对非格子对齐的连续矩形区域的积分，与蒙特卡洛采样估计交叉验证 |
| TEST(Sampling, HGExtremes) | 测试 Henyey-Greenstein 相函数采样在极端参数（g 接近 +/-1，u 为 0 或接近 1）下不产生 NaN |

## 依赖关系
- `pbrt/util/sampling.h` — 各种采样函数和分布的核心实现
- `pbrt/util/lowdiscrepancy.h` — Sobol 序列、基数逆函数等低差异序列
- `pbrt/util/rng.h` — 随机数生成器
- `pbrt/util/float.h` — 浮点数工具
- `pbrt/util/image.h` — 图像类
- `pbrt/util/math.h` — 数学工具函数
- `pbrt/util/parallel.h` — 并行执行框架
- `pbrt/util/print.h` — 格式化打印
