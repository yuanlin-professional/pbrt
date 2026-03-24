# media_test.cpp

## 概述
该测试文件验证 PBRT-v4 中参与介质（Participating Media）相关的相位函数实现。主要测试 Henyey-Greenstein 相位函数（`HGPhaseFunction`）的采样一致性、方向性、归一化和不对称参数 g 值。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(HenyeyGreenstein, SamplingMatch)` | 验证 HG 相位函数的 `Sample_p()` 方法生成的样本概率与 `p()` 函数求值一致（多个 g 值） |
| `TEST(HenyeyGreenstein, SamplingOrientationForward)` | 验证 g=0.95（强前向散射）时，绝大多数采样方向与入射方向同向 |
| `TEST(HenyeyGreenstein, SamplingOrientationBackward)` | 验证 g=-0.95（强后向散射）时，绝大多数采样方向与入射方向反向 |
| `TEST(HenyeyGreenstein, Normalized)` | 验证 HG 相位函数在球面上的积分为 1/(4*pi)（归一化条件） |
| `TEST(HenyeyGreenstein, g)` | 验证 HG 相位函数的余弦加权积分还原出正确的不对称参数 g 值 |

## 依赖关系
被测试的源文件：`pbrt/media.h`, `pbrt/util/rng.h`, `pbrt/util/sampling.h`
