# imgtool.cpp

## 概述
`imgtool` 是 PBRT-v4 提供的图像处理命令行工具。它支持多种图像操作，包括格式转换、图像比较、辉光效果、色调映射、伪彩色可视化、天空环境贴图生成、白平衡调整、降噪等功能。该工具以子命令方式组织，每个子命令负责一项具体的图像处理任务。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| `assemble` | 将多个表示大图像裁剪区域的图像拼接成一张完整的合成图像 |
| `average` | 对多张图像计算平均值，生成一张平均结果图像 |
| `bloom` | 对图像应用辉光效果，将高于阈值的亮像素模糊后叠加到附近像素 |
| `cat` | 将指定图像的像素值打印到标准输出 |
| `convert` | 转换图像格式，并支持多种图像处理操作（裁剪、缩放、色调映射、伽马校正等） |
| `diff` | 计算图像与参考图像之间的差异，支持多种误差度量（MAE/MSE/MRSE/FLIP） |
| `denoise-optix` | 使用 NVIDIA OptiX 深度神经网络去噪器对图像去噪（需 GPU 支持） |
| `scale-optix` | 使用 NVIDIA OptiX 对图像进行 2 倍超分辨率放大（需 GPU 支持） |
| `error` | 计算一组图像相对于参考图像的平均误差 |
| `falsecolor` | 生成伪彩色图像，以色彩编码显示每个像素各通道平均值的大小 |
| `info` | 打印图像信息，包括分辨率、色彩空间和像素统计数据 |
| `makeequiarea` | 将等距柱状（equirectangular）环境贴图转换为等面积参数化格式 |
| `makeemitters` | 为图像中的每个像素生成一个小型四边形发光体描述 |
| `makesky` | 基于 Hosek-Wilkie 天空模型生成环境贴图 |
| `scalenormalmap` | 对法线贴图应用缩放因子并输出处理后的法线贴图 |
| `splitn` | 从多张图像中各取一条像素带，按顺序拼合生成合成对比图 |
| `whitebalance` | 对指定图像应用白平衡校正 |

## 使用方式
```
imgtool <command> [options]
```

### 子命令详情

**assemble**：`imgtool assemble [options] <filenames...>`
- `--outfile <name>`：输出图像文件名

**average**：`imgtool average [options] <filename base>`
- `--outfile <name>`：输出图像文件名

**bloom**：`imgtool bloom [options] <filename>`
- `--iterations <n>`：滤波迭代次数（默认：5）
- `--level <n>`：像素贡献辉光的最小 RGB 值（默认：无穷大）
- `--outfile <name>`：输出图像文件名
- `--scale <s>`：辉光图像的缩放系数（默认：0.3）
- `--width <w>`：生成辉光的高斯宽度（默认：15）

**cat**：`imgtool cat [options] <filename>`
- `--csv`：以 CSV 格式输出
- `--list`：以花括号列表格式输出（兼容 Mathematica）
- `--sort`：按像素亮度排序输出

**convert**：`imgtool convert [options] <filename>`
- `--aces-filmic`：应用 ACES 胶片 S 曲线
- `--bw`：转换为黑白图像
- `--channels <names>`：指定处理的通道（默认：R,G,B）
- `--clamp <value>`：将像素值限制在指定范围内
- `--crop <x0,x1,y0,y1>`：裁剪图像
- `--colorspace <n>`：转换到指定色彩空间
- `--despike <v>`：去除亮度超过阈值的异常像素
- `--flipy`：沿 Y 轴翻转图像
- `--fp16`：转换为 fp16 像素格式
- `--gamma <v>`：应用伽马曲线
- `--maxluminance <n>`：色调映射中映射为白色的亮度值（默认：1）
- `--outfile <name>`：输出图像文件名
- `--preservecolors`：保留色彩比例关系
- `--repeatpix <n>`：在两个方向上将每个像素重复 n 次
- `--scale <scale>`：按给定比例缩放像素值
- `--tonemap`：应用色调映射

**diff**：`imgtool diff [options] <filename>`
- `--channels <names>`：指定比较的通道
- `--crop <x0,x1,y0,y1>`：比较前裁剪图像
- `--difftol <v>`：可接受的差异百分比（默认：0）
- `--metric <name>`：误差度量方法（MAE/MSE/MRSE/FLIP）
- `--outfile <name>`：差异图像输出文件名
- `--reference <name>`：参考图像文件名

**falsecolor**：`imgtool falsecolor [options] <filename>`
- `--maxvalue <v>`：映射到色彩渐变末端的值
- `--outfile <name>`：输出图像文件名
- `--plusminus`：正值为绿色、负值为红色的可视化模式
- `--ramp`：仅生成色彩渐变条图像

**info**：`imgtool info <filename>`
- 无额外选项

**makeequiarea**：`imgtool makeequiarea [options] <filename>`
- `--outfile <name>`：输出图像文件名
- `--resolution <n>`：输出图像分辨率

**makeemitters**：`imgtool makeemitters [options] <filename>`
- `--downsample <n>`：降采样倍数（默认：1）

**makesky**：`imgtool makesky [options] <filename>`
- `--albedo <a>`：地面反照率（0-1，默认：0.5）
- `--elevation <e>`：太阳仰角（0-90 度，默认：10）
- `--outfile <name>`：输出文件名
- `--turbidity <t>`：大气浑浊度（1.7-10，默认：3）
- `--resolution <r>`：环境贴图分辨率（默认：2048）

**scalenormalmap**：`imgtool scalenormalmap [options] <filename>`
- `--scale <s>`：缩放因子（默认：1）
- `--outfile <name>`：输出文件名

**splitn**：`imgtool splitn [options] <filenames>`
- `--crop <x,y>`：裁剪区域左上角坐标（可多次指定）
- `--cropsize <n>`：裁剪区域大小（默认：96）
- `--outfile <name>`：输出文件名

**whitebalance**：`imgtool whitebalance [options] <filename>`
- `--illuminant <n>`：按标准光源进行白平衡（如 D65、D50、A、F1 等）
- `--outfile <name>`：输出文件名
- `--primaries <x,y>`：按指定色度坐标进行白平衡
- `--temperature <T>`：按色温进行白平衡

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/filters.h`、`pbrt/options.h`、`pbrt/gpu/optix/denoiser.h`（GPU 模式）、`pbrt/gpu/optix/scaler.h`（GPU 模式）、`pbrt/gpu/util.h`（GPU 模式）、`pbrt/util/args.h`、`pbrt/util/check.h`、`pbrt/util/color.h`、`pbrt/util/colorspace.h`、`pbrt/util/error.h`、`pbrt/util/file.h`、`pbrt/util/image.h`、`pbrt/util/log.h`、`pbrt/util/math.h`、`pbrt/util/parallel.h`、`pbrt/util/print.h`、`pbrt/util/progressreporter.h`、`pbrt/util/rng.h`、`pbrt/util/sampling.h`、`pbrt/util/spectrum.h`、`pbrt/util/string.h`、`pbrt/util/vecmath.h`、`skymodel/ArHosekSkyModel.h`（外部库）、`flip.h`（外部库）
