# image_test.cpp

## 概述
该测试文件针对 `pbrt/util/image.h` 中的图像处理功能进行全面测试。测试覆盖图像的基本属性、像素读写、通道操作、矩形区域复制、多种图像格式（PFM、EXR、PNG、QOI）的读写往返、图像元数据的序列化/反序列化、图像采样分布以及通道选择等功能。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Image, Basics) | 测试不同像素格式（U256/Half/Float）和通道数（1/3）的图像基本属性，验证通道数和内存占用量的正确性 |
| TEST(Image, GetSetY) | 测试单通道（Y）图像在三种像素格式下的像素写入和读取，验证量化误差在预期范围内，并测试通道描述符的获取 |
| TEST(Image, GetSetRGB) | 测试三通道（RGB）图像在三种像素格式下的像素读写正确性，验证通过通道描述符和索引访问的一致性 |
| TEST(Image, GetSetBGR) | 测试通过 BGR 顺序的通道描述符访问 RGB 图像，验证通道重排映射的正确性 |
| TEST(Image, CopyRectOut) | 测试矩形区域像素数据的导出功能，在三种像素格式和多种通道数下验证导出的缓冲区数据与逐像素读取一致 |
| TEST(Image, CopyRectIn) | 测试矩形区域像素数据的导入功能，使用随机数据写入子区域后验证像素值的正确性 |
| TEST(Image, PfmIO) | 测试 PFM 格式的图像读写往返，验证分辨率、格式、通道数和像素值的完全一致性 |
| TEST(Image, ExrIO) | 测试 EXR 格式在三种像素格式下的图像读写往返，验证颜色空间元数据的保留和像素值精度 |
| TEST(Image, ExrNoMetadata) | 测试无附加元数据的 EXR 文件读写，验证颜色空间保留以及其他元数据字段为未设置状态 |
| TEST(Image, ExrMetadata) | 测试 EXR 文件的完整元数据读写，包括渲染时间、相机矩阵、NDC 矩阵、像素边界、完整分辨率、自定义字符串等 |
| TEST(Image, PngYIO) | 测试 PNG 格式单通道灰度图像的读写往返，验证 sRGB 编码的正确解码 |
| TEST(Image, PngRgbIO) | 测试 PNG 格式三通道 RGB 图像的读写往返，验证像素值和颜色空间元数据的正确性 |
| TEST(Image, PngEmojiIO) | 测试使用 Unicode emoji 字符（恐龙）作为文件名的 PNG 图像读写，验证 UTF-8 文件名支持 |
| TEST(Image, QoiRgbIO) | 测试 QOI 格式三通道 RGB 图像的读写往返 |
| TEST(Image, QoiRgbaIO) | 测试 QOI 格式四通道 ARGB 图像的读写往返，验证非标准通道顺序的正确处理 |
| TEST(Image, SampleSimple) | 测试图像采样分布的基本功能，使用简单的 2x2 图像（仅一个像素非零）验证采样点集中在正确区域 |
| TEST(Image, SampleLinear) | 测试线性函数图像的采样分布精度，验证采样 PDF 与函数值的一致性 |
| TEST(Image, SampleSinCos) | 测试复杂三角函数（sin/cos 组合）图像的采样分布精度，验证采样 PDF 与函数积分的关系 |
| TEST(Image, Wrap2D) | 测试图像边界包裹模式（Clamp、Repeat、Black）对越界像素访问和双线性插值的影响 |
| TEST(Image, Select) | 测试通道选择功能，从 4 通道图像中提取单通道和双通道子图像，验证通道名称和像素值的正确映射 |
| TEST(ImageIO, RoundTripEXR) | 测试 EXR 格式的完整读写往返精度 |
| TEST(ImageIO, RoundTripPFM) | 测试 PFM 格式的完整读写往返精度（精确匹配） |
| TEST(ImageIO, RoundTripPNG) | 测试 PNG 格式的完整读写往返精度（允许 sRGB 量化误差） |
| TEST(ImageIO, RoundTripQOI) | 测试 QOI 格式的完整读写往返精度 |

## 依赖关系
- `pbrt/util/image.h` - 被测试的图像处理模块
- `pbrt/util/color.h` - sRGB 编码/解码
- `pbrt/util/colorspace.h` - RGB 颜色空间定义
- `pbrt/util/file.h` - 文件操作工具
- `pbrt/util/float.h` - 浮点数工具（Half 类型等）
- `pbrt/util/mipmap.h` - MipMap 相关功能
- `pbrt/util/rng.h` - 随机数生成器
- `pbrt/util/sampling.h` - 采样分布（PiecewiseConstant2D）
- `pbrt/pbrt.h` - PBRT 核心头文件
