# GPU 渲染后端

## 概述

`src/pbrt/gpu/` 目录实现了 PBRT-v4 的 GPU 渲染后端基础设施。该模块提供基于 CUDA 的 GPU 计算核心工具，包括 GPU 统一内存管理、CUDA 内核启动封装、GPU/OpenGL 互操作显示，以及 GPU 设备初始化与性能分析等功能。此模块是 GPU 渲染路径（Wavefront Path Tracing）的底层支撑层，为上层的 OptiX 光线追踪加速和 Wavefront 积分器提供基础服务。

## 文件列表

| 文件 | 用途说明 |
|------|---------|
| `util.h` | GPU 工具函数头文件：定义 CUDA 错误检查宏（`CUDA_CHECK`、`CU_CHECK`）、`GPUParallelFor` 并行执行模板、`Kernel` CUDA 内核函数、GPU 初始化与同步接口、性能分析事件管理 |
| `util.cpp` | GPU 工具函数实现：`GPUInit` 设备初始化（CUDA 驱动/运行时版本检测、设备选择、栈大小及缓存配置）、`GPUWait` 同步、`ReportKernelStats` 性能报告、NVTX 标注支持 |
| `memory.h` | CUDA 统一内存资源头文件：定义 `CUDAMemoryResource`（基础 CUDA 托管内存分配器）和 `CUDATrackedMemoryResource`（带跟踪的内存分配器，支持分配统计与 GPU 预取） |
| `memory.cpp` | CUDA 内存资源实现：通过 `cudaMallocManaged` 分配统一内存、线程安全的分配跟踪、`PrefetchToGPU` 批量预取所有托管内存到 GPU |
| `cudagl.h` | CUDA-OpenGL 互操作头文件：定义 `BufferDisplay`（通过 OpenGL 着色器将渲染结果显示到屏幕）和 `CUDAOutputBuffer`（管理 CUDA/GL 共享像素缓冲区的映射、异步回读） |

## 架构图

```mermaid
graph TB
    subgraph "GPU 基础设施层 (src/pbrt/gpu/)"
        UTIL["util.h / util.cpp<br/>GPU 初始化与内核管理"]
        MEM["memory.h / memory.cpp<br/>CUDA 统一内存管理"]
        CUDAGL["cudagl.h<br/>CUDA-OpenGL 互操作显示"]
    end

    subgraph "上层依赖模块"
        OPTIX_AGG["gpu/optix/aggregate<br/>OptiX 加速结构"]
        OPTIX_DN["gpu/optix/denoiser<br/>OptiX 降噪器"]
        OPTIX_SC["gpu/optix/scaler<br/>OptiX 超分辨率"]
        WF["wavefront/integrator<br/>Wavefront 积分器"]
    end

    subgraph "外部依赖"
        CUDA["CUDA Runtime / Driver API"]
        GL["OpenGL / GLAD"]
        NVTX["NVTX (可选, 性能标注)"]
    end

    UTIL --> CUDA
    UTIL --> NVTX
    MEM --> CUDA
    CUDAGL --> CUDA
    CUDAGL --> GL
    CUDAGL --> UTIL

    OPTIX_AGG --> MEM
    OPTIX_AGG --> UTIL
    OPTIX_DN --> MEM
    OPTIX_DN --> UTIL
    OPTIX_SC --> MEM
    OPTIX_SC --> UTIL
    WF --> UTIL
    WF --> MEM
    WF --> CUDAGL
```

### 类关系图

```mermaid
classDiagram
    class CUDAMemoryResource {
        +do_allocate(size, alignment) void*
        +do_deallocate(p, bytes, alignment)
        +do_is_equal(other) bool
    }

    class CUDATrackedMemoryResource {
        -mutex : std::mutex
        -bytesAllocated : atomic~size_t~
        -allocations : unordered_map
        +do_allocate(size, alignment) void*
        +do_deallocate(p, bytes, alignment)
        +PrefetchToGPU()
        +BytesAllocated() size_t
        +singleton$ : CUDATrackedMemoryResource
    }

    class BufferDisplay {
        -m_render_tex : GLuint
        -m_program : GLuint
        -m_image_format : BufferImageFormat
        +BufferDisplay(format)
        +display(screen_res_x, screen_res_y, framebuf_res_x, framebuf_res_y, pbo)
    }

    class CUDAOutputBuffer~PIXEL_FORMAT~ {
        -m_width : int32_t
        -m_height : int32_t
        -m_cuda_gfx_resource : cudaGraphicsResource*
        -m_pbo : GLuint
        -m_device_pixels : PIXEL_FORMAT*
        -m_host_pixels : PIXEL_FORMAT*
        -display : BufferDisplay*
        +Map() PIXEL_FORMAT*
        +Unmap()
        +Draw(windowWidth, windowHeight)
        +StartAsynchronousReadback()
        +GetReadbackPixels() PIXEL_FORMAT*
    }

    class KernelStats {
        +description : string
        +numLaunches : int
        +sumMS : float
        +minMS : float
        +maxMS : float
    }

    class ProfilerEvent {
        +start : cudaEvent_t
        +stop : cudaEvent_t
        +active : bool
        +stats : KernelStats*
        +Sync()
    }

    CUDATrackedMemoryResource --|> CUDAMemoryResource
    CUDAMemoryResource --|> pstd_pmr_memory_resource : 继承
    CUDAOutputBuffer --> BufferDisplay : 持有
    ProfilerEvent --> KernelStats : 引用
```

## 核心类与接口

### CUDAMemoryResource

基础 CUDA 统一内存分配器，继承自 `pstd::pmr::memory_resource`。使用 `cudaMallocManaged` 分配 CUDA 托管内存（Unified Memory），CPU 和 GPU 均可直接访问。

### CUDATrackedMemoryResource

扩展的 CUDA 内存分配器，额外维护所有分配记录。关键功能：
- **线程安全分配跟踪**：通过互斥锁保护分配映射表
- **PrefetchToGPU()**：在渲染开始前将所有托管内存批量预取到 GPU，避免运行时页面迁移带来的性能损失
- **BytesAllocated()**：查询当前分配的总字节数
- **singleton**：全局单例，供整个 GPU 渲染管线使用

### GPUParallelFor

核心 GPU 并行执行模板函数，将任意可调用对象包装为 CUDA 内核并启动：
- 自动计算最优线程块大小（通过 `cudaOccupancyMaxPotentialBlockSize`）
- 集成 CUDA 事件进行性能计时
- 支持 NVTX 范围标注用于 Nsight 性能分析
- Debug 模式下自动同步以便调试

### GPUInit

GPU 设备初始化函数，执行以下操作：
- 检测 CUDA 驱动版本和运行时版本
- 枚举并选择 GPU 设备
- 设置栈大小（8192 字节）和 printf 缓冲区大小
- 配置 L1 缓存优先策略
- Windows 平台多 GPU 限制处理

### BufferDisplay / CUDAOutputBuffer

实现 CUDA 渲染结果到 OpenGL 窗口的实时显示：
- `BufferDisplay`：通过 OpenGL 着色器程序将纹理渲染到全屏四边形
- `CUDAOutputBuffer`：管理 CUDA 和 OpenGL 之间共享的像素缓冲对象（PBO），支持异步回读到 CPU

## 依赖关系

### 本模块依赖

| 依赖模块 | 说明 |
|---------|------|
| `pbrt/util/check.h` | 断言与检查宏 |
| `pbrt/util/error.h` | 错误处理 |
| `pbrt/util/log.h` | 日志系统 |
| `pbrt/util/parallel.h` | 并行工具 |
| `pbrt/util/pstd.h` | 标准库多态内存资源适配 |
| `pbrt/options.h` | 运行时选项（GPU 设备选择等） |
| CUDA Runtime / Driver API | GPU 计算基础 |
| OpenGL / GLAD | 图形显示（仅 `cudagl.h`） |
| NVTX（可选） | NVIDIA 性能分析标注 |

### 被以下模块依赖

| 模块 | 说明 |
|------|------|
| `gpu/optix/aggregate` | OptiX 加速结构构建与光线追踪 |
| `gpu/optix/denoiser` | OptiX AI 降噪 |
| `gpu/optix/scaler` | OptiX AI 超分辨率 |
| `wavefront/integrator` | Wavefront 路径追踪积分器 |
| `wavefront/workqueue` | 工作队列管理 |
| `pbrt/scene.cpp` | 场景构建 |
| `pbrt/lights.cpp` | 光源创建 |
| `pbrt/shapes.cpp` | 几何形状创建 |
| 多个 `util/` 模块 | 日志、光谱、颜色空间等工具 |

## 数据流

```mermaid
flowchart LR
    subgraph "初始化阶段"
        A["GPUInit()"] --> B["设备选择与配置"]
        B --> C["CUDATrackedMemoryResource<br/>创建全局单例"]
    end

    subgraph "场景加载阶段"
        C --> D["场景数据分配<br/>(统一内存)"]
        D --> E["PrefetchToGPU()<br/>预取到 GPU"]
    end

    subgraph "渲染阶段"
        E --> F["GPUParallelFor<br/>启动 CUDA 内核"]
        F --> G["Kernel 执行"]
        G --> H["GetProfilerEvents<br/>性能计时"]
    end

    subgraph "显示阶段"
        G --> I["CUDAOutputBuffer::Map()<br/>获取设备指针"]
        I --> J["写入渲染结果"]
        J --> K["CUDAOutputBuffer::Unmap()"]
        K --> L["BufferDisplay::display()<br/>OpenGL 显示"]
    end

    subgraph "性能报告"
        H --> M["ReportKernelStats()<br/>汇总内核执行时间"]
    end
```

### GPU 与 CPU 渲染路径对比

```mermaid
flowchart TB
    SCENE["场景描述"] --> PARSE["场景解析"]

    PARSE --> CPU_PATH["CPU 路径"]
    PARSE --> GPU_PATH["GPU 路径"]

    subgraph "CPU 渲染路径"
        CPU_PATH --> CPU_AGG["CPUAggregate<br/>(wavefront/aggregate.h)"]
        CPU_AGG --> CPU_BVH["CPU BVH 遍历"]
        CPU_BVH --> CPU_SHADE["CPU 着色计算"]
    end

    subgraph "GPU 渲染路径"
        GPU_PATH --> GPU_INIT["GPUInit()<br/>(gpu/util.h)"]
        GPU_INIT --> GPU_MEM["CUDATrackedMemoryResource<br/>(gpu/memory.h)"]
        GPU_MEM --> OPTIX_AGG["OptiXAggregate<br/>(gpu/optix/aggregate.h)"]
        OPTIX_AGG --> OPTIX_BVH["OptiX 硬件加速 BVH"]
        OPTIX_BVH --> GPU_SHADE["GPU 内核着色<br/>(GPUParallelFor)"]
    end

    CPU_SHADE --> FILM["胶片/图像输出"]
    GPU_SHADE --> FILM
    GPU_SHADE -.-> DISPLAY["实时显示<br/>(cudagl.h)"]
```
