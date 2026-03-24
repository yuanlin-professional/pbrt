# rgb2spec_opt.cpp

## 概述
`rgb2spec_opt` 是一个用于预计算 RGB 到光谱转换查找表的命令行工具。该工具基于 Wenzel Jakob 的方法，通过优化求解一组多项式系数，使得给定 RGB 颜色值可以通过 sigmoid 函数参数化的光谱反射率曲线来近似表示。生成的查找表以 C++ 源代码格式输出，可以直接编译到 PBRT 渲染器中使用。

该工具支持多种色域（gamut），包括 sRGB、ProPhoto RGB、ACES2065-1、Rec2020 等。它使用 CIE 1931 色彩匹配函数和标准光源光谱数据，通过 Gauss-Newton 迭代在 CIE L*a*b* 色彩空间中最小化残差，以获得高质量的光谱重建。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| 查找表优化 | 对三维 RGB 色彩空间进行离散化，为每个格点计算最优光谱系数 |
| 多色域支持 | 支持 sRGB、eRGB、XYZ、ProPhoto RGB、ACES2065-1、Rec2020、DCI-P3 色域 |
| 多光源适配 | 根据色域自动选择标准光源（D65、D50、D60 或 E） |
| 并行计算 | 使用多线程并行计算以加速查找表的生成 |
| C++ 代码输出 | 将结果以 C++ 数组常量形式写入输出文件 |

## 使用方式
```
rgb2spec_opt <resolution> <output> [<gamut>]
```

### 参数说明
| 参数 | 说明 |
|---|---|
| `<resolution>` | 查找表每个维度的分辨率 |
| `<output>` | 输出的 C++ 源代码文件路径 |
| `[<gamut>]` | 色域名称（可选，默认为 sRGB） |

### 支持的色域
| 色域名称 | 使用光源 | 说明 |
|---|---|---|
| `sRGB` | D65 | 标准 sRGB 色彩空间 |
| `eRGB` | E（等能光源） | 扩展 RGB 色彩空间 |
| `XYZ` | E（等能光源） | CIE XYZ 色彩空间 |
| `ProPhotoRGB` | D50 | ProPhoto RGB 色彩空间 |
| `ACES2065_1` | D60 | ACES 2065-1 色彩空间 |
| `REC2020` | D65 | Rec. 2020 色彩空间 |
| `DCI_P3` | D65 | DCI-P3 色彩空间 |

### 输出格式
生成的文件包含 `pbrt` 命名空间下的以下常量：
- `<gamut>ToSpectrumTable_Res`：查找表分辨率
- `<gamut>ToSpectrumTable_Scale`：缩放因子数组
- `<gamut>ToSpectrumTable_Data`：四维系数数组 `[3][res][res][res][3]`

## 依赖关系
- **依赖**：仅使用 C/C++ 标准库，不依赖 PBRT 内部模块。内嵌了 CIE 色彩匹配函数数据、多种标准光源光谱数据、色彩空间转换矩阵以及一个简化的并行计算框架（`ParallelFor`、`ThreadPool`）
