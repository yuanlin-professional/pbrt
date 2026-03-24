# filters_test.cpp

## 概述
该测试文件验证 PBRT-v4 中像素重建滤波器（Filter）的正确性。测试覆盖了所有内置滤波器类型（Box、Gaussian、Mitchell、LanczosSinc、Triangle），验证滤波器在半径外返回零值、以及滤波器的解析积分与数值积分的一致性。还包含对 Sinc 函数在零点附近行为的测试。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(Sinc, ZeroHandling)` | 验证 Sinc 函数在 x=0 附近的连续性和单调递减性（正方向和负方向） |
| `TEST(Filter, ZeroPastRadius)` | 验证所有滤波器类型在半径之外的求值结果为零，测试多种半径组合 |
| `TEST(Filter, Integral)` | 验证所有滤波器类型的 `Integral()` 方法与数值积分（分层采样 Monte Carlo）的结果一致 |

## 依赖关系
被测试的源文件：`pbrt/filters.h`, `pbrt/util/math.h`, `pbrt/util/sampling.h`
