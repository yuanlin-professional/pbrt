# math_test.cpp

## 概述
该测试文件针对 `pbrt/util/math.h` 中的数学工具函数和数据结构进行全面测试。测试覆盖 2 的幂运算、Morton 编码、编译期求幂、牛顿二分法求根、多项式求值、补偿求和、对数运算、素数查找、误差函数反函数、差积/和积运算、快速指数函数、高斯积分、方阵运算、区间查找、浮点区间算术以及排列元素等功能。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Pow2, Basics) | 测试 `IsPowerOf2` 函数，验证 2 的 0 次方到 31 次方被正确识别，以及非 2 的幂值被正确排除 |
| TEST(RoundUpPow2, Basics) | 测试 `RoundUpPow2` 函数，分别对 32 位和 64 位整数验证向上取整到最近 2 的幂的正确性 |
| TEST(Morton2, Basics) | 测试二维 Morton 编码/解码，验证特定值的编码结果，以及使用 100000 个随机值进行编码-解码往返验证 |
| TEST(Math, Pow) | 测试编译期 `Pow<N>` 模板函数，验证 Pow<0> 到 Pow<29> 对底数 2 的计算结果均正确 |
| TEST(Math, NewtonBisection) | 测试牛顿-二分法混合求根算法，验证简单线性函数、余弦函数、错误导数情况、多零点情况以及导数趋于无穷的病态函数 |
| TEST(Math, EvaluatePolynomial) | 测试 `EvaluatePolynomial` 多项式求值函数，验证常数多项式、一次多项式和三次多项式的计算结果 |
| TEST(Math, CompensatedSum) | 测试 Kahan 补偿求和算法的精度优势，对 1600 万个跨越多个数量级的随机浮点数求和，验证补偿求和比普通求和精确得多 |
| TEST(Log2Int, Basics) | 测试 `Log2Int` 函数，验证 32 位和 64 位整数的以 2 为底对数，以及 float/double 类型的正确性（包括最近偶数舍入行为） |
| TEST(Log4Int, Basics) | 测试 `Log4Int` 函数，对 1 到 16384 范围验证以 4 为底整数对数的正确性 |
| TEST(Pow4, Basics) | 测试 4 的幂相关函数：`IsPowerOf4` 判断和 `RoundUpPow4` 向上取整 |
| TEST(NextPrime, Basics) | 测试 `NextPrime` 函数，验证已知值（如 2->3, 10->11）和大范围素数查找的正确性 |
| TEST(Math, ErfInv) | 测试误差函数反函数 `ErfInv`，验证对多个已知值的 erf -> ErfInv 往返精度 |
| TEST(Math, DifferenceOfProducts) | 测试 `DifferenceOfProducts(a,b,c,d)` 即 a*b-c*d 的高精度计算，使用双精度 FMA 作为参考，验证误差不超过 2 ULP |
| TEST(Math, SumOfProducts) | 测试 `SumOfProducts(a,b,c,d)` 即 a*b+c*d 的高精度计算，验证误差不超过 2 ULP |
| TEST(FastExp, Accuracy) | 测试快速指数函数 `FastExp` 的精度，验证对 [-20, 20] 范围内随机值的相对误差不超过 0.03% |
| TEST(Math, GaussianIntegral) | 测试高斯积分函数 `GaussianIntegral`，使用蒙特卡洛积分作为参考，在多组均值和标准差参数下验证计算结果 |
| TEST(SquareMatrix, Basics2) | 测试 2x2 方阵的基本操作：单位矩阵、对角矩阵、转置、矩阵-向量乘法、矩阵求逆以及退化矩阵检测 |
| TEST(SquareMatrix, Basics3) | 测试 3x3 方阵的基本操作，功能同 2x2 矩阵测试 |
| TEST(SquareMatrix, Basics4) | 测试 4x4 方阵的基本操作，功能同 2x2 矩阵测试 |
| TEST(SquareMatrix, Inverse) | 使用随机矩阵大规模测试 2x2、3x3、4x4 方阵的求逆正确性，验证矩阵与其逆矩阵的乘积接近单位矩阵 |
| TEST(FindInterval, Basics) | 测试 `FindInterval` 二分查找函数，验证越界钳位以及正确定位目标值所在区间 |
| TEST(FloatInterval, Abs) | 测试浮点区间的绝对值运算，使用 100 万次随机测试验证精确值始终在区间范围内 |
| TEST(FloatInterval, Sqrt) | 测试浮点区间的平方根运算精度 |
| TEST(FloatInterval, Add) | 测试浮点区间的加法运算精度 |
| TEST(FloatInterval, Sub) | 测试浮点区间的减法运算精度 |
| TEST(FloatInterval, Mul) | 测试浮点区间的乘法运算精度 |
| TEST(FloatInterval, Div) | 测试浮点区间的除法运算精度（跳过分母跨零的情况） |
| TEST(FloatInterval, FMA) | 测试浮点区间的融合乘加运算（FMA），验证结果区间包含精确值，且比分步计算（a*b+c）产生更紧的区间 |
| TEST(FloatInterval, Sqr) | 测试浮点区间的平方运算，特别验证跨零区间的处理（下界应为 0） |
| TEST(FloatInterval, SumSquares) | 测试浮点区间的平方和运算，验证 SumSquares 对 1 到 3 个参数的正确性 |
| TEST(FloatInterval, DifferenceOfProducts) | 测试浮点区间的差积运算（a*b-c*d），使用双精度参考验证精确值在结果区间内 |
| TEST(FloatInterval, SumOfProducts) | 测试浮点区间的和积运算（a*b+c*d），使用双精度参考验证精确值在结果区间内 |
| TEST(Math, TwoProd) | 测试 `TwoProd` 补偿乘法函数，验证其 Float 结果等于普通乘法，double 结果等于双精度乘法 |
| TEST(Math, TwoSum) | 测试 `TwoSum` 补偿加法函数，验证其 Float 结果等于普通加法，double 结果等于双精度加法 |
| TEST(Math, InnerProduct) | 测试补偿内积函数 `InnerProduct`，验证 4 维向量的内积结果精度等同于双精度计算 |
| TEST(PermutationElement, Valid) | 验证 `PermutationElement` 函数对长度 2 到 1023 的排列均生成有效排列（每个元素恰好出现一次） |
| TEST(PermutationElement, Uniform) | 验证排列函数的均匀性，统计各种排列长度下每个位置映射到每个目标位置的频率，验证分布均匀（误差不超过 3%） |
| TEST(PermutationElement, UniformDelta) | 验证排列函数的位移均匀性，统计每个元素的位移量（目标位置-原始位置）分布是否均匀 |

## 依赖关系
- `pbrt/util/math.h` - 被测试的数学工具模块（大量数学函数和 SquareMatrix、Interval 等类）
- `pbrt/util/float.h` - 浮点数工具（BitsToFloat、FloatToBits 等）
- `pbrt/util/rng.h` - 随机数生成器
- `pbrt/util/vecmath.h` - 向量数学工具
- `pbrt/pbrt.h` - PBRT 核心头文件
