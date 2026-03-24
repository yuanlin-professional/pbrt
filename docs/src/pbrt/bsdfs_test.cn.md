# bsdfs_test.cpp

## 概述
该测试文件对 PBRT-v4 中的双向散射分布函数（BSDF）进行全面测试，主要包括两方面：（1）基于卡方检验的采样一致性测试，验证 BSDF 的采样方法生成的样本分布与其概率密度函数是否一致；（2）能量守恒测试，验证 BSDF 反射的总能量不超过入射能量。还包括针对头发 BSDF（`HairBxDF`）的白炉测试和采样一致性测试。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(BSDFSampling, Lambertian)` | 对 Lambertian 漫反射 BSDF 进行卡方检验采样一致性测试 |
| `TEST(BSDFSampling, TRCondIso)` | 对各向同性导体微面 BSDF（Trowbridge-Reitz, eta=2, k=4, roughness=0.5）进行卡方检验 |
| `TEST(BSDFSampling, TRCondAniso)` | 对各向异性导体微面 BSDF（roughness 0.3x0.15）进行卡方检验 |
| `TEST(BSDFSampling, TRDielIso)` | 对各向同性电介质微面 BSDF（IOR=1.5, roughness=0.5）进行卡方检验 |
| `TEST(BSDFSampling, TRDielAniso)` | 对各向异性电介质微面 BSDF（IOR=1.5, roughness 0.3x0.15）进行卡方检验 |
| `TEST(BSDFSampling, TRDielIsoInv)` | 对各向同性电介质微面 BSDF（IOR=1/1.5，从内部出射）进行卡方检验 |
| `TEST(BSDFSampling, TRDielAnisoInv)` | 对各向异性电介质微面 BSDF（IOR=1/1.5，从内部出射）进行卡方检验 |
| `TEST(BSDFEnergyConservation, LambertianReflection)` | 验证 Lambertian 反射的能量守恒（反射率不超过 1） |
| `TEST(Hair, WhiteFurnace)` | 对头发 BSDF 进行白炉测试（零吸收时积分应为 1） |
| `TEST(Hair, HOnTheEdge)` | 测试头发 BSDF 在边缘情况（h=-1）下的数值稳定性 |
| `TEST(Hair, WhiteFurnaceSampled)` | 使用重要性采样的头发白炉测试 |
| `TEST(Hair, SamplingWeights)` | 验证头发 BSDF 采样权重接近 1 |
| `TEST(Hair, SamplingConsistency)` | 验证头发 BSDF 重要性采样与均匀采样的一致性 |

## 依赖关系
被测试的源文件：`pbrt/bsdf.h`, `pbrt/interaction.h`, `pbrt/shapes.h`（用于创建测试表面）, `pbrt/util/sampling.h`, `pbrt/util/spectrum.h`
