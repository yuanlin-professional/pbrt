# subsurface.cpp

## 概述
该文件是 `WavefrontPathIntegrator` 的次表面散射（Subsurface Scattering, SSS）实现部分，不对应独立的头文件。它实现了 `SampleSubsurface()` 方法，处理具有 BSSRDF（双向次表面散射反射分布函数）属性的材质。该方法包含三个关键步骤：获取 BSSRDF 并生成探测光线、执行随机求交采样、处理出射散射（包括间接光照和直接光照的采样）。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `WavefrontPathIntegrator::SampleSubsurface(wavefrontDepth)` | 次表面散射主方法，协调 BSSRDF 评估、探测光线求交和出射散射处理三个阶段 |

## 算法流程图
```mermaid
flowchart TD
    A[SampleSubsurface 入口] --> B{场景中有次表面散射材质?}
    B -- 否 --> Z[返回]
    B -- 是 --> C["阶段1: ForAllQueued 处理 bssrdfEvalQueue"]
    C --> D[获取 SubsurfaceMaterial 的 BSSRDF]
    D --> E[调用 bssrdf.SampleSp 采样探测段]
    E --> F{采样成功?}
    F -- 是 --> G[Push 到 subsurfaceScatterQueue]
    F -- 否 --> H[跳过]

    G --> I["阶段2: aggregate->IntersectOneRandom"]
    I --> J[蓄水池采样求交结果]

    J --> K["阶段3: ForAllQueued 处理 subsurfaceScatterQueue"]
    K --> L{reservoirPDF > 0?}
    L -- 否 --> M[跳过]
    L -- 是 --> N[ProbeIntersectionToSample 转换为样本]
    N --> O{样本有效?}
    O -- 否 --> M
    O -- 是 --> P[计算 betap 和 r_u]

    P --> Q["间接光照: 采样 BSDF"]
    Q --> R{bsdfSample 有效?}
    R -- 是 --> S[计算 beta, 俄罗斯轮盘赌]
    S --> T{beta 非零?}
    T -- 是 --> U[SpawnRay 入队 NextRayQueue]

    P --> V{BSDF 非镜面?}
    V -- 是 --> W["直接光照: 采样光源"]
    W --> X[计算 MIS 权重]
    X --> Y[入队 ShadowRayQueue]

    Y --> AA[TraceShadowRays 追踪次表面散射的阴影光线]
```

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/bssrdf.h`、`pbrt/interaction.h`、`pbrt/lightsamplers.h`、`pbrt/samplers.h`、`pbrt/util/sampling.h`、`pbrt/util/spectrum.h`、`pbrt/wavefront/integrator.h`
- **被依赖**：作为 `WavefrontPathIntegrator` 方法的实现文件，由 `integrator.cpp` 中的 `Render()` 循环在每个波前深度的材质评估之后调用
