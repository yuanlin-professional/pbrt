# nanovdb2pbrt.cpp

## 概述
`nanovdb2pbrt` 是一个将 NanoVDB 格式的体积数据文件转换为 PBRT 可用的体积数据格式的命令行工具。NanoVDB 是 OpenVDB 的轻量级 GPU 友好版本，常用于存储雾体积（fog volume）等三维体积数据。该工具读取 `.nvdb` 文件中的指定网格数据，将体素值采样并以文本格式输出到标准输出，格式可直接嵌入 PBRT 场景描述中使用。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| 网格读取 | 从 NanoVDB 文件中读取指定名称的浮点型网格 |
| 体素采样 | 遍历网格索引空间中的所有体素并提取值 |
| 格式输出 | 将网格尺寸、世界空间包围盒和体素数据以 PBRT 参数格式输出到标准输出 |

## 使用方式
```
nanovdb2pbrt [<options>] <filename.nvdb>
```

### 参数说明
| 参数 | 说明 |
|---|---|
| `<filename.nvdb>` | 输入的 NanoVDB 文件路径 |
| `--grid <name>` | 指定要提取的网格名称（默认：`"density"`） |

### 输出格式
输出内容为 PBRT 参数格式的文本，包含：
- `"integer nx"`、`"integer ny"`、`"integer nz"`：三维网格在各轴方向的尺寸
- `"point3 p0"`、`"point3 p1"`：网格在世界空间中的包围盒坐标
- `"float <grid_name>"`：展平的体素浮点值数组

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/media.h`、`pbrt/util/args.h`、`nanovdb/NanoVDB.h`（外部库）、`nanovdb/util/IO.h`（外部库）、`nanovdb/util/GridHandle.h`（外部库）、`nanovdb/util/SampleFromVoxels.h`（外部库）
