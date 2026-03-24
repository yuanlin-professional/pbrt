# color_test.cpp

## 概述
该测试文件针对 PBRT 中颜色空间转换和光谱相关功能进行全面测试。测试覆盖 RGB 颜色空间（sRGB、Rec2020、ACES2065-1）之间的 XYZ 转换、标准光源白点验证、RGB 无界光谱和反照率光谱的往返转换精度、RGB 照明光谱的往返转换，以及 sRGB 伽马编码/解码的正确性。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(RGBColorSpace, RGBXYZ) | 验证在 ACES2065-1、Rec2020、sRGB 三种颜色空间中，白色 RGB(1,1,1) 经过 RGB->XYZ->RGB 往返转换后仍为白色 |
| TEST(RGBColorSpace, sRGB) | 验证 sRGB 颜色空间的 XYZ 到 RGB 转换矩阵的列向量值是否与标准值（3.2406, -1.5372 等）吻合 |
| TEST(RGBColorSpace, StdIllumWhitesRGB) | 验证 sRGB 标准光源的光谱转换为 XYZ 再转换为 RGB 后，结果接近白色 (1, 1, 1) |
| TEST(RGBColorSpace, StdIllumWhiteRec2020) | 验证 Rec2020 标准光源的光谱转换为 XYZ 再转换为 RGB 后，结果接近白色 (1, 1, 1) |
| TEST(RGBColorSpace, StdIllumWhiteACES2065_1) | 验证 ACES2065-1 标准光源的光谱转换为 XYZ 再转换为 RGB 后，结果接近白色 (1, 1, 1) |
| TEST(RGBUnboundedSpectrum, MaxValue) | 使用随机 RGB 值在三种颜色空间中创建 RGBUnboundedSpectrum，验证 `MaxValue()` 方法返回的最大值与通过密集采样获得的最大值一致 |
| TEST(RGBAlbedoSpectrum, MaxValue) | 使用随机 RGB 值在三种颜色空间中创建 RGBAlbedoSpectrum，验证 `MaxValue()` 方法返回的最大值与通过密集采样获得的最大值一致 |
| TEST(RGBAlbedoSpectrum, RoundTripsRGB) | 验证在 sRGB 颜色空间中，随机 RGB 值通过 RGBAlbedoSpectrum 光谱表示后，再乘以光源光谱并转换回 RGB，结果与原始 RGB 值误差小于 0.01 |
| TEST(RGBAlbedoSpectrum, RoundTripRec2020) | 验证在 Rec2020 颜色空间中，RGB 反照率光谱的往返转换精度（RGB 值范围限制在 [0.1, 0.8]） |
| TEST(RGBAlbedoSpectrum, RoundTripACES) | 验证在 ACES2065-1 颜色空间中，RGB 反照率光谱的往返转换精度（RGB 值范围限制在 [0.3, 0.7]） |
| TEST(RGBIlluminantSpectrum, RoundTripsRGB) | 验证在 sRGB 颜色空间中，随机 RGB 值通过 RGBIlluminantSpectrum 光谱表示后的往返转换精度 |
| TEST(RGBIlluminantSpectrum, RoundTripRec2020) | 验证在 Rec2020 颜色空间中，RGB 照明光谱的往返转换精度 |
| TEST(RGBIlluminantSpectrum, RoundTripACES) | 验证在 ACES2065-1 颜色空间中，RGB 照明光谱的往返转换精度 |
| TEST(sRGB, Conversion) | 测试 sRGB 伽马编码/解码的正确性：(1) 验证 `SRGBToLinear` 和 `SRGB8ToLinear` 对 0-255 所有 8 位值结果一致；(2) 验证线性值经 sRGB 编码再解码的往返精度；(3) 验证 sRGB 值经线性化再编码的往返精度 |

## 依赖关系
- `pbrt/util/color.h` - sRGB 编码/解码功能
- `pbrt/util/colorspace.h` - RGB 颜色空间定义（sRGB、Rec2020、ACES2065-1）
- `pbrt/util/spectrum.h` - 光谱相关类（RGBUnboundedSpectrum、RGBAlbedoSpectrum、RGBIlluminantSpectrum）
- `pbrt/util/sampling.h` - 随机数生成
- `pbrt/pbrt.h` - PBRT 核心头文件
