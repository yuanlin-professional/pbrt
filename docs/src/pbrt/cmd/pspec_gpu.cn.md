# pspec_gpu.cpp

## 概述
`pspec_gpu.cpp` 是 `pspec` 功率谱分析工具的 GPU 加速辅助文件。由于 CMake 在构建 CUDA 可执行文件时对单文件编译的支持存在限制，GPU 相关的 CUDA 内核启动代码被单独放置在此文件中。该文件仅在编译时定义了 `PBRT_BUILD_GPU_RENDERER` 宏时才会被编译。

该文件实现了两个关键函数：
1. `UPSInit`：初始化 GPU 端的缓冲池（环形缓冲区），用于异步传输采样点数据到 GPU。
2. `UpdatePowerSpectrum`：将一组采样点上传到 GPU，并在 GPU 上并行执行傅里叶变换以更新功率谱图像。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| `UPSInit(int nPoints)` | 初始化 GPU 缓冲池，分配 GPU 内存、CUDA 事件和固定主机内存 |
| `UpdatePowerSpectrum(...)` | 异步上传采样点到 GPU 并执行 GPU 并行傅里叶变换，累积功率谱 |

## 使用方式
该文件不是独立的命令行工具，而是 `pspec` 工具的组成部分。它在 `pspec.cpp` 中通过前向声明被调用。

### 实现细节
- 使用环形缓冲池（16 个缓冲区）管理 GPU 端的采样点内存
- 每个缓冲区包含 GPU 端设备内存、CUDA 事件（用于同步）和主机端固定内存（用于异步传输）
- 通过 `cudaMemcpyAsync` 异步传输数据，通过 `cudaEventRecord`/`cudaEventSynchronize` 管理缓冲区生命周期
- 傅里叶变换内核使用 `GPUParallelFor` 在 GPU 上并行计算每个频率分量

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/gpu/util.h`、`pbrt/util/image.h`、`pbrt/util/vecmath.h`、`cuda.h`（CUDA 运行时）、`cuda_runtime_api.h`（CUDA 运行时）
