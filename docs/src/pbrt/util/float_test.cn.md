# float_test.cpp

## 概述
该测试文件针对 `pbrt/util/float.h` 中的浮点数操作工具进行全面测试。测试覆盖浮点数分解（指数、尾数）、相邻浮点值（NextFloatUp/Down）、位表示转换、原子浮点数操作，以及半精度浮点数（Half）的创建、转换、精度、遍历和舍入等功能。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(FloatingPoint, Pieces) | 测试浮点数的 Exponent（指数）和 Significand（尾数）分解函数，分别验证 float 和 double 类型的正确性 |
| TEST(FloatingPoint, NextUpDownFloat) | 测试 float 类型的 NextFloatUp 和 NextFloatDown 函数：验证负零、正负无穷的边界行为，以及对 100000 个随机浮点数与标准库 `std::nextafter` 结果一致 |
| TEST(FloatingPoint, NextUpDownDouble) | 测试 double 类型的 NextFloatUp 和 NextFloatDown 函数：验证边界行为和 100000 个随机双精度浮点数的正确性 |
| TEST(FloatingPoint, FloatBits) | 测试 float 的位表示往返转换，对 100000 个随机 uint32_t 值验证 `BitsToFloat` -> `FloatToBits` 的一致性 |
| TEST(FloatingPoint, DoubleBits) | 测试 double 的位表示往返转换，对 100000 个随机 uint64_t 值验证 `BitsToFloat` -> `FloatToBits` 的一致性 |
| TEST(FloatingPoint, AtomicFloat) | 测试原子浮点数 AtomicFloat 的基本操作，验证初始化和 Add 方法的累加结果与普通浮点数一致 |
| TEST(Half, Basics) | 测试半精度浮点数的基本属性：正零、负零、正负无穷的位模式，NaN 检测，以及对所有 65536 个半精度值验证取负运算的正确性 |
| TEST(Half, ExactConversions) | 测试半精度浮点数的精确往返转换：验证 -2048 到 2048 的整数以及各种精确可表示的浮点数经 Half 转换后值不变 |
| TEST(Half, Randoms) | 使用 1024 个随机正浮点数测试半精度转换的舍入正确性，验证转换结果是最接近的半精度表示 |
| TEST(Half, NextUp) | 测试从负无穷遍历到正无穷的 NextUp 操作，验证每一步值严格递增，且总步数正确（排除 NaN 值） |
| TEST(Half, NextDown) | 测试从正无穷遍历到负无穷的 NextDown 操作，验证每一步值严格递减，且总步数正确 |
| TEST(Half, Equal) | 测试半精度浮点数的相等性比较：验证所有非特殊值自等性，以及正零与负零相等但位表示不同 |
| TEST(Half, RoundToNearestEven) | 测试半精度浮点数的"最近偶数舍入"规则，对所有相邻半精度值之间的中间值，验证舍入到低位为零的方向 |

## 依赖关系
- `pbrt/util/float.h` - 被测试的浮点数工具模块（BitsToFloat、FloatToBits、NextFloatUp/Down、AtomicFloat、Half 等）
- `pbrt/util/math.h` - 数学工具函数
- `pbrt/util/rng.h` - 随机数生成器
- `pbrt/util/parallel.h` - 并行工具
- `pbrt/pbrt.h` - PBRT 核心头文件
