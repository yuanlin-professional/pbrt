# integrators_test.cpp

## 概述

该文件是 PBRT-v4 中 CPU 积分器的参数化集成测试，使用 Google Test 框架验证各类积分器在已知解析解场景下的渲染正确性。测试通过构建简单的封闭球体场景（已知理论辐亮度值），搭配多种采样器和相机类型，验证不同积分器（路径追踪、体积路径追踪、简单路径追踪、双向路径追踪、MLT）渲染结果的像素平均值是否接近理论期望值。

## 测试用例列表

| 测试用例 | 说明 |
|---|---|
| `TEST_P(RenderTest, RadianceMatches)` | 参数化测试：运行积分器渲染场景，读取输出 EXR 图像，检验所有像素 RGB 通道的平均值是否接近期望辐亮度值（容差 0.025） |

### 测试场景（由 `GetScenes` 生成）

| 场景 | 说明 |
|---|---|
| 场景 1 | 单位球体，Kd=0.5，中心点光源 I=Pi。在全局光照下理论辐亮度为 1.0 |
| 场景 2 | 单位球体，Kd=0.5，4 个点光源各 I=Pi/4。在全局光照下理论辐亮度为 1.0 |
| 场景 3 | 单位球体，Kd=0.5，自发光 Le=0.5。在全局光照下理论辐亮度为 1.0 |

### 测试积分器（由 `GetIntegrators` 生成）

| 积分器 | 相机类型 | 说明 |
|---|---|---|
| `PathIntegrator` | 透视/正交 | 路径追踪，深度 8 |
| `VolPathIntegrator` | 透视/正交 | 体积路径追踪，深度 8 |
| `SimplePathIntegrator` | 透视 | 简单路径追踪，深度 8，同时采样光源和 BSDF |
| `BDPTIntegrator` | 透视 | 双向路径追踪，深度 6 |
| `MLTIntegrator` | 透视 | 梅特罗波利斯光传输，深度 8，100000 次自举采样 |

### 测试采样器（由 `GetSamplers` 生成）

| 采样器 | 说明 |
|---|---|
| `HaltonSampler` | Halton 低差异序列 |
| `PaddedSobolSampler` | 填充 Sobol 序列（数位置换随机化） |
| `ZSobolSampler` | Z-curve Sobol 序列 |
| `SobolSampler` (无随机化) | Sobol 序列，不进行随机化 |
| `SobolSampler` (XOR 扰乱) | Sobol 序列，数位置换随机化 |
| `SobolSampler` (Owen 扰乱) | Sobol 序列，Owen 扰乱随机化 |
| `IndependentSampler` | 独立均匀随机采样 |
| `StratifiedSampler` | 分层采样 |
| `PMJ02BNSampler` | PMJ02BN 采样器 |

### 辅助结构和函数

| 名称 | 说明 |
|---|---|
| `TestScene` | 测试场景结构体，包含聚合体、光源列表、描述和期望辐亮度值 |
| `TestIntegrator` | 测试积分器结构体，包含积分器指针、胶片、描述和关联场景 |
| `CheckSceneAverage` | 读取 EXR 图像文件，计算所有像素 RGB 通道的平均值并与期望值比较 |
| `GetScenes` | 生成所有测试场景 |
| `GetSamplers` | 生成所有测试采样器 |
| `GetIntegrators` | 排列组合场景、采样器和积分器生成完整的参数化测试用例 |

## 依赖关系

- **被测试的源文件**：
  - `pbrt/cpu/integrators.h` / `pbrt/cpu/integrators.cpp` — 所有被测积分器的实现
  - `pbrt/cpu/aggregates.h` / `pbrt/cpu/aggregates.cpp` — BVH 加速结构，测试中构建场景图元使用
- **测试框架**：
  - `gtest/gtest.h` — Google Test 框架
- **其他依赖**：
  - `pbrt/cameras.h` — 透视相机、正交相机
  - `pbrt/filters.h` — BoxFilter
  - `pbrt/lights.h` — 点光源、面光源
  - `pbrt/materials.h` — DiffuseMaterial
  - `pbrt/samplers.h` — 各类采样器
  - `pbrt/shapes.h` — Sphere 形状
  - `pbrt/textures.h` — 常量纹理
  - `pbrt/util/image.h` — 图像读取
  - `pbrt/util/spectrum.h` — 光谱工具
