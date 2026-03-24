# lightsamplers_test.cpp

## 概述
该测试文件验证 PBRT-v4 中光源采样器（LightSampler）的正确性。测试覆盖了 `BVHLightSampler`（基于 BVH 的光源采样）、`ExhaustiveLightSampler`（穷举光源采样）、`UniformLightSampler`（均匀光源采样）和 `PowerLightSampler`（基于功率的光源采样）。验证内容包括单光源采样、多点光源采样权重一致性、不同功率光源的采样频率比例，以及 `PMF()` 方法与 `Sample()` 返回概率的一致性。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(BVHLightSampling, OneSpot)` | 验证 BVH 采样器对单个聚光灯的采样：确保在有光照的区域能采样到光源，在锥体外无法采样到 |
| `TEST(BVHLightSampling, Point)` | 验证 BVH 采样器对 33 个随机点光源的采样权重一致性（加权频率应接近 1） |
| `TEST(BVHLightSampling, PointVaryPower)` | 验证 BVH 采样器对不同功率点光源的采样：近距离权重一致性 + 远距离采样频率与功率成正比 |
| `TEST(BVHLightSampling, OneTri)` | 验证 BVH 采样器对单个三角形面光源的采样，确保非发光侧不会被采样到 |
| `TEST(BVHLightSampling, PdfMethod)` | 验证 BVH 采样器的 `Sample()` 返回的 PDF 与 `PMF()` 方法的返回值一致 |
| `TEST(ExhaustiveLightSampling, PdfMethod)` | 验证穷举光源采样器的 `Sample()` PDF 与 `PMF()` 一致 |
| `TEST(UniformLightSampling, PdfMethod)` | 验证均匀光源采样器的 `Sample()` PDF 与 `PMF()` 一致 |
| `TEST(PowerLightSampling, PdfMethod)` | 验证功率光源采样器的 `Sample()` PDF 与 `PMF()` 一致 |

## 依赖关系
被测试的源文件：`pbrt/lightsamplers.h`, `pbrt/lights.h`, `pbrt/shapes.h`（用于创建面光源三角形）, `pbrt/interaction.h`, `pbrt/util/spectrum.h`, `pbrt/util/transform.h`, `pbrt/util/rng.h`, `pbrt/util/math.h`
