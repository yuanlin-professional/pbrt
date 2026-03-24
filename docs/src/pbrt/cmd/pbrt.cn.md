# pbrt.cpp

## 概述
`pbrt.cpp` 是 PBRT-v4 渲染器的主入口程序。该文件实现了 `pbrt` 命令行可执行文件，负责解析命令行参数、初始化渲染系统、读取场景描述文件并执行渲染。它支持 CPU 渲染和 GPU（Wavefront）渲染两种模式，同时也提供了场景文件的格式化和版本升级功能。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| 渲染场景 | 读取 `.pbrt` 场景描述文件并渲染输出图像 |
| `--format` | 将输入文件重新格式化后输出到标准输出，不进行渲染 |
| `--toply` | 格式化输入文件并将所有三角网格转换为 PLY 文件 |
| `--upgrade` | 将 pbrt-v3 格式的场景文件升级为 pbrt-v4 格式 |
| `--gpu` | 使用 GPU 进行渲染（需编译时启用 GPU 支持） |
| `--wavefront` | 使用 Wavefront 体积路径积分器进行渲染 |
| `--interactive` | 启用交互式渲染模式（仅支持 GPU/Wavefront 模式） |

## 使用方式
```
pbrt [<options>] <filename.pbrt...>
```

### 渲染选项
| 参数 | 说明 |
|---|---|
| `--cropwindow <x0,x1,y0,y1>` | 指定图像裁剪窗口（相对于 [0,1]^2） |
| `--debugstart <values>` | 指定渲染调试起始点，用于快速调试 |
| `--disable-image-textures` | 始终返回图像纹理的平均值 |
| `--disable-pixel-jitter` | 始终在像素中心采样 |
| `--disable-texture-filtering` | 对所有纹理使用点采样 |
| `--disable-wavelength-jitter` | 始终采样相同的波长 |
| `--displacement-edge-scale <s>` | 缩放目标三角形边长（默认：1） |
| `--display-server <addr:port>` | 连接到显示服务器，在渲染过程中实时显示图像 |
| `--force-diffuse` | 将所有材质转换为漫反射材质 |
| `--fullscreen` | 全屏渲染（仅支持 `--interactive` 模式） |
| `--gpu` | 使用 GPU 渲染（默认关闭） |
| `--gpu-device <index>` | 指定用于渲染的 GPU 设备 |
| `--nthreads <num>` | 指定渲染使用的线程数 |
| `--outfile <filename>` | 指定输出图像文件名 |
| `--pixel <x,y>` | 仅渲染指定的单个像素 |
| `--pixelbounds <x0,x1,y0,y1>` | 按像素坐标指定图像裁剪窗口 |
| `--pixelmaterial <x,y>` | 打印指定像素处可见材质的信息 |
| `--pixelstats` | 记录每像素统计信息并输出额外图像 |
| `--quick` | 自动降低多项质量设置以加快渲染速度 |
| `--quiet` | 除错误信息外不输出任何文本 |
| `--render-coord-sys <name>` | 设置渲染坐标系（camera/cameraworld/world） |
| `--seed <n>` | 设置随机数生成器种子（默认：0） |
| `--spp <n>` | 覆盖场景文件中指定的每像素采样数 |
| `--stats` | 渲染完成后打印各项统计信息 |
| `--write-partial-images` | 渲染过程中定期将当前图像写入磁盘 |

### 日志选项
| 参数 | 说明 |
|---|---|
| `--log-file <filename>` | 指定日志输出文件 |
| `--log-level <level>` | 日志等级（verbose/error/fatal，默认：error） |
| `--log-utilization` | 定期记录处理器和内存使用情况 |

### 格式化选项
| 参数 | 说明 |
|---|---|
| `--format` | 输出格式化后的场景文件 |
| `--toply` | 输出格式化场景文件并将三角网格转为 PLY 格式 |
| `--upgrade` | 将 pbrt-v3 场景文件升级为 pbrt-v4 格式 |

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/cpu/render.h`、`pbrt/gpu/memory.h`（GPU 模式）、`pbrt/options.h`、`pbrt/parser.h`、`pbrt/scene.h`、`pbrt/util/args.h`、`pbrt/util/check.h`、`pbrt/util/error.h`、`pbrt/util/log.h`、`pbrt/util/memory.h`、`pbrt/util/parallel.h`、`pbrt/util/print.h`、`pbrt/util/spectrum.h`、`pbrt/util/string.h`、`pbrt/wavefront/wavefront.h`
