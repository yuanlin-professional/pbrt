# lightsamplers.h / lightsamplers.cpp

## 概述
该文件实现了 PBRT-v4 中的光源采样器，负责在渲染过程中高效地从场景中多个光源中选择一个进行采样。文件提供了四种光源采样策略：均匀采样、基于功率的采样、基于 BVH 的空间自适应采样和穷举采样。BVH 光源采样器是默认策略，它利用光源的空间边界和方向信息构建层次加速结构，根据着色点的位置和法线对光源进行重要性采样。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `UniformLightSampler` | 均匀光源采样器，以等概率从所有光源中随机选择一个，最简单但效率最低 |
| `PowerLightSampler` | 功率光源采样器，根据每个光源的总辐射功率按比例采样，使用别名表（AliasTable）实现 O(1) 采样 |
| `BVHLightSampler` | BVH 光源采样器（默认），将有界光源构建为二叉 BVH 树，根据着色点位置和法线估算每个光源的重要性，自顶向下遍历选择光源；无限光源单独处理 |
| `ExhaustiveLightSampler` | 穷举光源采样器，遍历所有光源计算重要性后用加权水库采样选择，结果最准确但计算量大 |
| `CompactLightBounds` | 光源边界的紧凑表示，使用量化存储减少内存占用，包含量化的包围盒、方向和角度信息 |
| `LightBVHNode` | BVH 树节点，可以是叶节点（存储光源索引）或内部节点（存储子节点索引），对齐到 32 字节 |
| `LightSampler::Create()` | 工厂方法，根据名称（"uniform"/"power"/"bvh"/"exhaustive"）创建对应的采样器 |

## 架构图
```mermaid
classDiagram
    class LightSampler {
        <<TaggedPointer>>
        +Sample(LightSampleContext, Float) SampledLight
        +PMF(LightSampleContext, Light) Float
        +Sample(Float) SampledLight
        +PMF(Light) Float
        +Create(string, span~Light~, Allocator) LightSampler
    }
    class UniformLightSampler {
        -pstd::vector~Light~ lights
        +Sample(Float) SampledLight
        +PMF(Light) Float
    }
    class PowerLightSampler {
        -pstd::vector~Light~ lights
        -HashMap~Light,size_t~ lightToIndex
        -AliasTable aliasTable
        +Sample(Float) SampledLight
        +PMF(Light) Float
    }
    class BVHLightSampler {
        -pstd::vector~Light~ lights
        -pstd::vector~Light~ infiniteLights
        -Bounds3f allLightBounds
        -pstd::vector~LightBVHNode~ nodes
        -HashMap~Light,uint32_t~ lightToBitTrail
        +Sample(LightSampleContext, Float) SampledLight
        +PMF(LightSampleContext, Light) Float
        -buildBVH() pair~int,LightBounds~
        -EvaluateCost() Float
    }
    class ExhaustiveLightSampler {
        -pstd::vector~Light~ lights, boundedLights, infiniteLights
        -pstd::vector~LightBounds~ lightBounds
        +Sample(LightSampleContext, Float) SampledLight
        +PMF(LightSampleContext, Light) Float
    }
    class CompactLightBounds {
        -OctahedralVector w
        -Float phi
        -uint16_t qb[2][3]
        +Importance(Point3f, Normal3f, Bounds3f) Float
        +Bounds(Bounds3f) Bounds3f
    }
    class LightBVHNode {
        +CompactLightBounds lightBounds
        +unsigned childOrLightIndex
        +unsigned isLeaf
        +MakeLeaf() LightBVHNode
        +MakeInterior() LightBVHNode
    }

    LightSampler <|.. UniformLightSampler
    LightSampler <|.. PowerLightSampler
    LightSampler <|.. BVHLightSampler
    LightSampler <|.. ExhaustiveLightSampler
    BVHLightSampler --> LightBVHNode
    LightBVHNode --> CompactLightBounds
```

## 算法流程图
```mermaid
flowchart TD
    A[BVHLightSampler::Sample] --> B[计算无限光源采样概率 pInfinite]
    B --> C{随机数 u < pInfinite?}
    C -->|是| D[均匀采样一个无限光源]
    C -->|否| E[从 BVH 根节点开始遍历]
    E --> F{当前节点是叶节点?}
    F -->|是| G[检查光源重要性 > 0]
    G -->|是| H[返回该光源及其 PMF]
    G -->|否| I[返回空]
    F -->|否| J[计算两个子节点的重要性 ci0, ci1]
    J --> K{ci0 和 ci1 都为 0?}
    K -->|是| I
    K -->|否| L[按重要性比例随机选择子节点]
    L --> M[更新 PMF 乘以选择概率]
    M --> F
```

## 依赖关系
- **依赖**：`pbrt/base/light.h`、`pbrt/base/lightsampler.h`、`pbrt/lights.h`（使用 `LightBounds`）、`pbrt/util/containers.h`、`pbrt/util/hash.h`、`pbrt/util/sampling.h`、`pbrt/util/vecmath.h`、`pbrt/interaction.h`
- **被依赖**：`wavefront/workitems.h`、`wavefront/subsurface.cpp`、`wavefront/integrator.cpp`、`cpu/integrators.h`、`lightsamplers_test.cpp`
