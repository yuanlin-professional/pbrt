# spectrum_test.cpp

## 概述
该测试文件验证 PBRT-v4 中光谱相关功能的正确性，包括黑体辐射光谱、CIE XYZ 颜色匹配函数的积分、各种光谱类型的最大值计算，以及可见波长重要性采样的 PDF 正确性。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Spectrum, Blackbody) | 验证黑体辐射函数 `Blackbody()` 的正确性：使用已知参考值对比多组 (波长, 温度) 的辐射值（相对误差 < 0.1%）；使用维恩位移定律验证峰值波长处的辐射值确实是最大值 |
| TEST(Spectrum, XYZ) | 双重验证 CIE XYZ 颜色匹配函数：(1) 直接对匹配函数在 360-830nm 范围积分并归一化，验证 x/y/z 积分均约为 1；(2) 通过采样波长计算常数光谱的 XYZ 值，验证结果均约为 1 |
| TEST(Spectrum, MaxValue) | 测试多种光谱类型的 `MaxValue()` 方法：`ConstantSpectrum` 返回常数值；`PiecewiseLinearSpectrum` 返回节点最大值；`BlackbodySpectrum` 归一化后最大值约为 1；`DenselySampledSpectrum` 最大值与原始光谱一致；`RGBAlbedoSpectrum`/`RGBUnboundedSpectrum`/`RGBIlluminantSpectrum` 的 `MaxValue()` 确实不小于所有采样波长处的值 |
| TEST(Spectrum, SamplingPdfY) | 验证可见波长重要性采样能正确积分 Y 匹配函数：使用 `SampleVisibleWavelengths` 和对应 PDF 进行蒙特卡洛积分，结果应与 CIE_Y_integral 常数一致 |
| TEST(Spectrum, SamplingPdfXYZ) | 验证可见波长重要性采样对 X+Y+Z 匹配函数之和的积分结果与均匀采样一致，确认重要性采样的无偏性 |

## 依赖关系
- `pbrt/util/spectrum.h` — 各种光谱类型（常数、分段线性、黑体、RGB 光谱等）
- `pbrt/util/color.h` — 颜色类型和转换
- `pbrt/util/colorspace.h` — 色彩空间（sRGB 等）
- `pbrt/util/rng.h` — 随机数生成器
- `pbrt/util/sampling.h` — 采样工具（分层采样等）
