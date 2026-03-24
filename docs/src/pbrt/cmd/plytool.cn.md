# plytool.cpp

## 概述
`plytool` 是 PBRT-v4 提供的 PLY 网格处理命令行工具。它支持对 PLY 格式的三角形/四边形网格文件进行多种操作，包括查看网格内容、获取网格信息、应用位移贴图以及将大网格拆分为多个较小的文件。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| `cat` | 以文本形式打印网格的所有数据（三角形索引、四边形索引、顶点位置、法线和 UV 坐标） |
| `info` | 打印网格的概要信息（三角形/四边形数量、顶点数、法线数、UV 数、包围盒等） |
| `displace` | 对网格应用位移贴图，根据图像和法线方向对顶点进行偏移 |
| `split` | 将大网格拆分为多个较小的 PLY 文件 |
| `help` | 显示帮助信息或指定子命令的详细用法 |

## 使用方式
```
plytool <command> [options]
```

### 子命令详情

**cat**：`plytool cat <filename>`
- 打印指定 PLY 文件的完整网格数据

**info**：`plytool info <filename...>`
- 支持同时查看多个 PLY 文件的信息
- 输出三角形数量、四边形数量、顶点位置/法线/UV 数量、未使用顶点以及包围盒

**displace**：`plytool displace [options] <filename>`
| 参数 | 说明 |
|---|---|
| `--scale <s>` | 位移值的缩放系数（默认：1） |
| `--uvscale <s>` | UV 纹理坐标的缩放系数（默认：1） |
| `--edge-length <s>` | 未位移网格的最大边长（默认：1） |
| `--image <name>` | 用于定义位移的图像文件名 |
| `--outfile <name>` | 输出 PLY 文件名 |

**split**：`plytool split [options] <filename>`
| 参数 | 说明 |
|---|---|
| `--maxfaces <n>` | 单个输出 PLY 文件中的最大面数（默认：1000000） |
| `--outbase <name>` | 输出 PLY 文件的基础名称（默认：基于源文件名） |

## 依赖关系
- **依赖**：`pbrt/options.h`、`pbrt/pbrt.h`、`pbrt/util/args.h`、`pbrt/util/file.h`、`pbrt/util/image.h`、`pbrt/util/mesh.h`、`pbrt/util/string.h`、`pbrt/util/transform.h`、`pbrt/util/vecmath.h`
