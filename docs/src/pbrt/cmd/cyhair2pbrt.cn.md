# cyhair2pbrt.cpp

## 概述
`cyhair2pbrt` 是一个将 CyHair 格式的头发数据文件转换为 PBRT 场景描述格式的命令行工具。CyHair 是一种常用的头发模型文件格式，其中的顶点数据被解释为 Catmull-Rom 样条控制点。该工具将 Catmull-Rom 样条转换为三次 Bezier 曲线，并以 PBRT 的 `Shape "curve"` 语法输出，使头发数据可以直接用于 PBRT 渲染。

该文件内嵌了一个完整的 CyHair 加载器（来自 Light Transport Entertainment, Inc.，MIT 许可证），负责解析 CyHair 二进制文件格式并执行样条转换。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| 格式转换 | 将 CyHair (.hair) 文件转换为 PBRT 场景描述文件 |
| 样条转换 | 将 Catmull-Rom 样条控制点转换为三次 Bezier 曲线控制点 |
| 坐标变换 | 自动将 Z-up 坐标系转换为 Y-up 坐标系 |
| 场景边界计算 | 计算并输出转换后几何体的场景包围盒 |

## 使用方式
```
cyhair2pbrt <CyHair 文件名> <pbrt 输出文件名> [最大发丝数] [粗细]
```

### 参数说明
| 参数 | 说明 |
|---|---|
| `<CyHair 文件名>` | 输入的 CyHair 格式文件路径 |
| `<pbrt 输出文件名>` | 输出的 PBRT 场景描述文件路径（使用 `-` 表示输出到标准输出） |
| `[最大发丝数]` | 可选参数，限制转换的最大发丝数量（默认：-1，即转换全部） |
| `[粗细]` | 可选参数，指定发丝粗细（默认：1.0；如果 CyHair 文件包含粗细信息则使用文件中的值） |

### 输出格式
输出文件包含：
- 源文件信息注释
- 发丝数量和粗细参数注释
- 场景包围盒注释
- 每根曲线使用 `Shape "curve"` 形式定义，类型为圆柱体（cylinder），包含 4 个三次 Bezier 控制点和起止宽度

## 依赖关系
- **依赖**：仅使用 C/C++ 标准库（`<cassert>`、`<cstdio>`、`<cstdlib>`、`<cstring>`、`<vector>`、`<iostream>`、`<algorithm>`），不依赖 PBRT 内部模块
