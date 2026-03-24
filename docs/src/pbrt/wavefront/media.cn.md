# media.cpp

## 概述
该文件是 `WavefrontPathIntegrator` 的参与介质交互实现部分，不对应独立的头文件。它实现了 `SampleMediumInteraction()` 和 `SampleMediumScattering()` 方法，负责处理光线在参与介质（如烟雾、云层等体积媒介）中的传播、吸收、散射和发射。该文件是波前渲染管线中处理体积渲染效果的核心模块。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `SampleMediumScatteringCallback` | 回调结构体，用于通过 `ForEachType` 对每种相函数类型调用 `SampleMediumScattering` |
| `WavefrontPathIntegrator::SampleMediumInteraction(wavefrontDepth)` | 处理介质采样队列中的所有工作项：执行 majorant 传输函数采样（`SampleT_maj`），根据随机选择的事件类型（吸收、散射、null散射）进行相应处理 |
| `WavefrontPathIntegrator::SampleMediumScattering<PhaseFunction>(wavefrontDepth)` | 模板方法，处理介质散射事件：采样直接光照（从光源采样）和间接光照（从相函数采样新方向），包含俄罗斯轮盘赌终止 |

## 算法流程图
```mermaid
flowchart TD
    A[SampleMediumInteraction 入口] --> B{场景中有介质?}
    B -- 否 --> Z[返回]
    B -- 是 --> C[ForAllQueued 处理 mediumSampleQueue]
    C --> D[SampleT_maj 沿光线采样]
    D --> E[计算吸收/散射/null散射概率]
    E --> F{随机选择事件类型}
    F -- 吸收 --> G["beta = 0, 终止光线"]
    F -- 散射 --> H["更新 beta 和 r_u"]
    H --> I[入队 MediumScatterQueue]
    F -- null散射 --> J["更新 beta, r_u, r_l"]
    J --> K{beta 和 r_u 非零?}
    K -- 是 --> D
    K -- 否 --> G

    C --> L{存在发射介质?}
    L -- 是 --> M[累加发射辐射度到像素 L]

    C --> N{未散射且有剩余 beta?}
    N -- 是 --> O{"tMax == Infinity?"}
    O -- 是 --> P[入队 EscapedRayQueue]
    O -- 否 --> Q[解析 MixMaterial]
    Q --> R{材质为 null?}
    R -- 是 --> S[生成透射光线入队 NextRayQueue]
    R -- 否 --> T[入队相应的 MaterialEvalQueue]

    A --> U[ForEachType 对每种 PhaseFunction]
    U --> V[SampleMediumScattering]
    V --> W[采样光源 + 入队 ShadowRayQueue]
    V --> X[采样相函数 + 俄罗斯轮盘赌]
    X --> Y[入队 NextRayQueue 间接光线]
```

## 依赖关系
- **依赖**：`pbrt/wavefront/integrator.h`、`pbrt/media.h`
- **被依赖**：作为 `WavefrontPathIntegrator` 方法的实现文件，由 `integrator.cpp` 中的 `Render()` 循环在每个波前深度中调用
