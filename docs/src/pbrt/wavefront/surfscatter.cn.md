# surfscatter.cpp

## 概述
该文件是 `WavefrontPathIntegrator` 的表面散射（Surface Scattering）实现部分，不对应独立的头文件。它实现了 `EvaluateMaterialsAndBSDFs()` 及其辅助模板方法，负责在光线与表面交点处评估材质属性、构建 BSDF、计算纹理滤波微分、处理法线贴图和位移贴图、采样间接光照方向和直接光照光源。这是波前渲染管线中计算量最大、逻辑最复杂的核心模块之一。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `EvaluateMaterialCallback` | 回调结构体，用于通过 `ForEachType` 对每种具体材质类型调用 `EvaluateMaterialAndBSDF`（跳过 `MixMaterial`） |
| `WavefrontPathIntegrator::EvaluateMaterialsAndBSDFs(wavefrontDepth, movingFromCamera)` | 入口方法，对所有材质类型执行分发 |
| `WavefrontPathIntegrator::EvaluateMaterialAndBSDF<ConcreteMaterial>(wavefrontDepth, movingFromCamera)` | 中间模板方法，根据纹理评估器类型（Basic/Universal）选择相应队列并调用具体实现 |
| `WavefrontPathIntegrator::EvaluateMaterialAndBSDF<ConcreteMaterial, TextureEvaluator>(evalQueue, movingFromCamera, wavefrontDepth)` | 核心模板方法，执行完整的材质评估和光线散射采样流程 |

## 算法流程图
```mermaid
flowchart TD
    A[EvaluateMaterialsAndBSDFs 入口] --> B["ForEachType 遍历 Material::Types"]
    B --> C["对每种 ConcreteMaterial（跳过 MixMaterial）"]
    C --> D{haveBasicEvalMaterial?}
    D -- 是 --> E["处理 basicEvalMaterialQueue"]
    C --> F{haveUniversalEvalMaterial?}
    F -- 是 --> G["处理 universalEvalMaterialQueue"]

    E --> H[ForAllQueued 处理队列中每个工作项]
    G --> H

    H --> I["计算屏幕空间微分 dpdx, dpdy"]
    I --> J["估计 du/dx, du/dy, dv/dx, dv/dy"]
    J --> K{有法线贴图?}
    K -- 是 --> L[NormalMap 计算着色法线]
    K -- 否 --> M{有位移贴图?}
    M -- 是 --> N[BumpMap 计算着色法线]
    M -- 否 --> O[使用原始着色法线]

    L --> P[构建 BSDF]
    N --> P
    O --> P
    P --> Q{需要正则化?}
    Q -- 是 --> R[bsdf.Regularize]
    Q -- 否 --> S[继续]
    R --> S

    S --> T{depth==0 且需要 VisibleSurface?}
    T -- 是 --> U[计算反照率并初始化 VisibleSurface]

    S --> V["间接光照: bsdf.Sample_f"]
    V --> W{采样成功?}
    W -- 是 --> X[计算 beta, r_u, r_l]
    X --> Y[更新 etaScale]
    Y --> Z[俄罗斯轮盘赌]
    Z --> AA{beta 非零?}
    AA -- 是 --> AB{透射 + 次表面散射?}
    AB -- 是 --> AC[入队 bssrdfEvalQueue]
    AB -- 否 --> AD[SpawnRay 入队 NextRayQueue]

    S --> AE{BSDF 非镜面?}
    AE -- 是 --> AF[采样光源]
    AF --> AG[计算 BSDF 值 f]
    AG --> AH{f 非零?}
    AH -- 是 --> AI["计算 MIS 权重 r_u, r_l"]
    AI --> AJ["Ld = beta × ls.L"]
    AJ --> AK[入队 ShadowRayQueue]
```

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/base/bxdf.h`、`pbrt/bxdfs.h`、`pbrt/cameras.h`、`pbrt/interaction.h`、`pbrt/materials.h`、`pbrt/options.h`、`pbrt/textures.h`、`pbrt/util/check.h`、`pbrt/util/containers.h`、`pbrt/util/spectrum.h`、`pbrt/util/vecmath.h`、`pbrt/wavefront/integrator.h`
- **被依赖**：作为 `WavefrontPathIntegrator` 方法的实现文件，由 `integrator.cpp` 中的 `Render()` 循环在每个波前深度的求交和逃逸/发射处理之后调用
