# samplers_test.cpp

## 概述
该测试文件验证 PBRT-v4 中所有采样器实现的正确性。测试覆盖了采样值的一致性（同一像素/样本索引应产生相同的采样值）、低差异采样器的基本区间特性（elementary intervals），以及 ZSobolSampler 的索引有效性。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(Sampler, ConsistentValues)` | 验证所有采样器（Halton、Independent、PaddedSobol 各种随机化、ZSobol 各种随机化、PMJ02BN、Stratified、Sobol 各种随机化）在相同像素和样本索引下生成相同的样本值，即使中间访问了其他像素 |
| `TEST(PaddedSobolSampler, ElementaryIntervals)` | 验证 PaddedSobol 采样器满足 (0,2)-序列的基本区间性质（None 和 PermuteDigits 随机化） |
| `TEST(ZSobolSampler, ElementaryIntervals)` | 验证 ZSobol 采样器满足基本区间性质（多种种子和随机化策略） |
| `TEST(SobolUnscrambledSampler, ElementaryIntervals)` | 验证未扰动 Sobol 采样器的基本区间性质 |
| `TEST(SobolXORScrambledSampler, ElementaryIntervals)` | 验证 XOR 扰动（PermuteDigits）Sobol 采样器的基本区间性质 |
| `TEST(SobolOwenScrambledSampler, ElementaryIntervals)` | 验证 Owen 扰动 Sobol 采样器的基本区间性质 |
| `TEST(PMJ02BNSampler, ElementaryIntervals)` | 验证 PMJ02BN 采样器满足基本区间性质（4 的幂次采样数） |
| `TEST(ZSobolSampler, ValidIndices)` | 验证 ZSobol 采样器生成的索引无重复，且同一像素的所有样本索引在同一对齐范围内 |

## 依赖关系
被测试的源文件：`pbrt/samplers.h`
