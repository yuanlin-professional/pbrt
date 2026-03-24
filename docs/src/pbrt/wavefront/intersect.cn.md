# intersect.h

## 概述
该头文件定义了波前路径追踪中光线求交后的工作分发辅助函数。它提供了三个关键的内联函数，用于在光线与场景求交后根据结果类型（未命中、命中、阴影光线遮挡）将工作项入队到对应的工作队列。此外还包含了带参与介质的阴影光线透射率追踪算法模板。这些函数同时支持 CPU 和 GPU 执行。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `EnqueueWorkAfterMiss()` | 光线未命中时的分发逻辑：如果光线在介质中则入队 `MediumSampleQueue`，否则入队 `EscapedRayQueue` |
| `RecordShadowRayResult()` | 记录阴影光线结果：如果未被遮挡，使用 MIS 权重计算直接光照贡献并累加到像素 |
| `EnqueueWorkAfterIntersection()` | 光线命中后的分发逻辑：处理介质接口、MixMaterial 解析、null 材质透射、面光源命中和材质评估队列选择 |
| `TransmittanceTraceResult` | 透射率追踪的结果结构体，包含是否命中、命中点位置和材质 |
| `TraceTransmittance()` | 模板函数，实现带参与介质的阴影光线透射率追踪，使用比率追踪（ratio tracking）和俄罗斯轮盘赌终止策略 |

## 架构图
```mermaid
graph LR
    subgraph 求交结果分发
        A[光线求交完成] --> B{命中?}
        B -- 否 --> C[EnqueueWorkAfterMiss]
        B -- 是 --> D[EnqueueWorkAfterIntersection]
        C --> E[MediumSampleQueue]
        C --> F[EscapedRayQueue]
        D --> G[MediumSampleQueue]
        D --> H[NextRayQueue]
        D --> I[HitAreaLightQueue]
        D --> J[MaterialEvalQueue]
    end
    subgraph 阴影光线处理
        K[阴影光线求交] --> L[RecordShadowRayResult]
        L --> M[更新 PixelSampleState.L]
        N[带介质阴影光线] --> O[TraceTransmittance]
        O --> P[比率追踪 + 俄罗斯轮盘赌]
        P --> M
    end
```

## 算法流程图
```mermaid
flowchart TD
    A[TraceTransmittance 开始] --> B[初始化 T_ray = 1, r_u = 1, r_l = 1]
    B --> C{光线方向有效?}
    C -- 否 --> K[结束]
    C -- 是 --> D[调用 trace 获取求交结果]
    D --> E{命中不透明表面?}
    E -- 是 --> F[T_ray = 0, 终止]
    E -- 否 --> G{光线在介质中?}
    G -- 是 --> H[SampleT_maj 进行比率追踪]
    H --> I[更新 T_ray, r_u, r_l]
    I --> J{T_ray 过小?}
    J -- 是 --> J1[俄罗斯轮盘赌]
    J -- 否 --> C
    J1 --> C
    G -- 否 --> L{是否还有交点?}
    L -- 是 --> M[生成新光线继续追踪]
    M --> C
    L -- 否 --> N[计算最终 Ld 并更新像素]
    F --> K
    N --> K
```

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/util/spectrum.h`、`pbrt/wavefront/workitems.h`
- **被依赖**：`pbrt/wavefront/aggregate.cpp`（`CPUAggregate` 的求交方法中使用这些辅助函数）、GPU 端 OptiX 代码
