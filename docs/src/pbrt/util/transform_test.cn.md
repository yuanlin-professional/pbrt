# transform_test.cpp

## 概述
该测试文件验证 PBRT-v4 中几何变换相关功能的正确性，主要测试动画变换的运动包围盒计算和向量旋转变换 `RotateFromTo` 函数。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(AnimatedTransform, Randoms) | 测试动画变换的运动包围盒：生成 200 对随机变换矩阵（由平移、缩放、旋转随机组合而成），对每对变换创建 `AnimatedTransform`，对 5 个随机包围盒计算运动包围盒，然后在时间范围 [0,1] 内随机采样多个时刻，验证插值变换后的包围盒始终被运动包围盒所包含 |
| TEST(RotateFromTo, Simple) | 测试 `RotateFromTo` 在简单情况下的正确性：(1) 相同方向（from=to=(0,0,1)），变换后向量不变；(2) z 轴到 x 轴的旋转；(3) z 轴到 y 轴的旋转 |
| TEST(RotateFromTo, Randoms) | 使用 100 组随机单位向量对测试 `RotateFromTo`：验证变换后的向量长度保持为 1，且与目标向量的点积大于 0.999（即方向高度一致） |

## 依赖关系
- `pbrt/util/transform.h` — Transform、AnimatedTransform、RotateFromTo 等变换功能
- `pbrt/util/rng.h` — 随机数生成器
- `pbrt/util/sampling.h` — SampleUniformSphere 用于生成随机方向
