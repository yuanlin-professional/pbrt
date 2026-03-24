# lights_test.cpp

## 概述
该测试文件验证 PBRT-v4 中各种光源类型的功率计算和采样方法的正确性。测试覆盖了聚光灯（SpotLight）、测角光源（GoniometricLight）和投影光源（ProjectionLight），并验证光源的解析功率与蒙特卡洛采样估计的功率一致。还包含 `LightBounds` 光源重要性边界的基本测试。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(SpotLight, Power)` | 验证聚光灯的解析功率 `Phi()` 与 QMC 数值积分估计值的一致性 |
| `TEST(SpotLight, Sampling)` | 验证聚光灯 `SampleLe()` 方法的正确性，确保采样无偏（单样本即可精确计算功率） |
| `TEST(GoniometricLight, Power)` | 验证测角光源的解析功率与 Hammersley 采样估计值的一致性 |
| `TEST(GoniometricLight, Sampling)` | 验证测角光源 `SampleLe()` 采样估计功率与 `Phi()` 的一致性 |
| `TEST(ProjectionLight, Power)` | 验证投影光源在不同分辨率下的解析功率与数值估计的一致性 |
| `TEST(ProjectionLight, Sampling)` | 验证投影光源 `SampleLe()` 采样估计功率的正确性 |
| `TEST(LightBounds, Basics)` | 验证 `LightBounds` 的重要性计算：发光侧正重要性、非发光侧零重要性、边缘低重要性 |

## 依赖关系
被测试的源文件：`pbrt/lights.h`, `pbrt/util/spectrum.h`, `pbrt/util/sampling.h`, `pbrt/util/lowdiscrepancy.h`, `pbrt/util/transform.h`, `pbrt/util/image.h`, `pbrt/util/color.h`, `pbrt/util/colorspace.h`
