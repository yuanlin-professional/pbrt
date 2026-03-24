# pspec.cpp

## 概述
`pspec` 是 PBRT-v4 提供的功率谱分析命令行工具。该工具用于计算 PBRT 各种采样器生成的点集的功率谱，是分析和评估采样质量的重要工具。它对给定采样器生成的多组点集执行傅里叶变换，计算并输出平均功率谱图像（EXR 格式）以及径向平均功率谱数据（TXT 格式）。支持 GPU 加速计算（如果编译时启用了 GPU 支持）。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| 功率谱计算 | 对采样器生成的二维点集执行离散傅里叶变换并累积功率谱 |
| 径向平均 | 计算功率谱图像的径向平均值并输出为文本文件 |
| 多采样器支持 | 支持十余种不同的采样器/点集类型 |
| GPU 加速 | 在支持 GPU 的构建中使用 CUDA 加速傅里叶变换计算 |

## 使用方式
```
pspec <sampler> [<options...>]
```

### 支持的采样器类型
| 采样器名称 | 说明 |
|---|---|
| `cwd.pts` | 从当前目录中名为 `pts-*` 的文件读取点集 |
| `grid` | 规则网格点集 |
| `halton` | Halton 序列的前两个维度 |
| `halton.owen` | 使用 Owen 扰乱的 Halton 序列 |
| `halton.permutedigits` | 使用随机数位排列的 Halton 序列 |
| `independent` | 独立均匀随机采样 |
| `lhs` | 拉丁超方体采样 |
| `pmj02bn` | 渐进多抖动 (0,2) 蓝噪声采样（使用预计算表，最多 5 组） |
| `sobol` | Sobol' 序列的前两个维度 |
| `sobol.fastowen` | 使用快速哈希并行位运算扰乱的 Sobol' 序列 |
| `sobol.owen` | 使用 Owen 扰乱的 Sobol' 序列 |
| `sobol.permutedigits` | 使用位排列扰乱的 Sobol' 序列 |
| `sobol.z` | 对应 ZSobolSampler 的随机化 Morton Z 曲线 Sobol' |
| `stdin.binary` | 从标准输入读取二进制 32 位浮点数点集 |
| `stdin.dat` | 从标准输入读取文本格式点集 |
| `stratified` | 分层采样网格 |

### 选项
| 参数 | 说明 |
|---|---|
| `--npoints <n>` | 每组点集中的采样点数量（默认：1024） |
| `--nsets <n>` | 独立点集的组数（默认：4） |
| `--outbase <name>` | 输出文件的基础名称（默认：`<sampler>-<npoints>pts-<nsets>sets`） |
| `--resolution <res>` | 功率谱图像的分辨率（默认：1500） |

### 输出文件
- `<outbase>.exr`：功率谱图像（EXR 格式）
- `<outbase>.txt`：径向平均功率谱数据（文本格式）

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/base/sampler.h`、`pbrt/paramdict.h`、`pbrt/parser.h`、`pbrt/samplers.h`、`pbrt/util/args.h`、`pbrt/util/colorspace.h`、`pbrt/util/file.h`、`pbrt/util/image.h`、`pbrt/util/parallel.h`、`pbrt/util/print.h`、`pbrt/util/progressreporter.h`、`pbrt/util/pstd.h`、`pbrt/gpu/memory.h`（GPU 模式）、`pbrt/gpu/util.h`（GPU 模式）
