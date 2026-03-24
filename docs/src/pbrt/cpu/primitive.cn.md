# primitive.h / primitive.cpp

## 概述

该文件定义了 PBRT-v4 中"图元"（Primitive）的类型系统，是光线追踪渲染管线中连接几何形状（Shape）与材质（Material）、光源（Light）、介质（Medium）等属性的核心抽象层。`Primitive` 使用 `TaggedPointer` 模式实现多态分派，避免了虚函数开销，并统一了简单图元、几何图元、变换图元、动画图元以及加速结构聚合体的接口。

## 主要类与接口

| 类/结构体/函数 | 说明 |
|---|---|
| `Primitive` | 基于 `TaggedPointer` 的多态图元类型，可指向 `SimplePrimitive`、`GeometricPrimitive`、`TransformedPrimitive`、`AnimatedPrimitive`、`BVHAggregate`、`KdTreeAggregate` 中的任一类型 |
| `SimplePrimitive` | 最简单的图元，仅包含形状和材质，不支持面光源、alpha 纹理或介质接口 |
| `GeometricPrimitive` | 完整功能的几何图元，支持形状、材质、面光源、介质接口和 alpha 透明度纹理 |
| `TransformedPrimitive` | 变换图元，将一个子图元包装在一个静态变换下，用于实例化 |
| `AnimatedPrimitive` | 动画图元，将一个子图元包装在时变动画变换下，支持运动模糊 |

## 架构图

```mermaid
classDiagram
    class Primitive {
        <<TaggedPointer>>
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    class SimplePrimitive {
        -shape : Shape
        -material : Material
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    class GeometricPrimitive {
        -shape : Shape
        -material : Material
        -areaLight : Light
        -mediumInterface : MediumInterface
        -alpha : FloatTexture
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    class TransformedPrimitive {
        -primitive : Primitive
        -renderFromPrimitive : Transform*
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    class AnimatedPrimitive {
        -primitive : Primitive
        -renderFromPrimitive : AnimatedTransform
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    class BVHAggregate {
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    class KdTreeAggregate {
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    Primitive --> SimplePrimitive : 可指向
    Primitive --> GeometricPrimitive : 可指向
    Primitive --> TransformedPrimitive : 可指向
    Primitive --> AnimatedPrimitive : 可指向
    Primitive --> BVHAggregate : 可指向
    Primitive --> KdTreeAggregate : 可指向

    TransformedPrimitive --> Primitive : 包含子图元
    AnimatedPrimitive --> Primitive : 包含子图元
```

## 算法流程图

### GeometricPrimitive::Intersect 求交流程（含 alpha 测试）

```mermaid
flowchart TD
    A[输入光线 ray] --> B[shape.Intersect 求交]
    B --> C{有交点?}
    C -->|否| D[返回空]
    C -->|是| E{有 alpha 纹理?}
    E -->|否| J[设置交互属性并返回]
    E -->|是| F[计算 alpha 值]
    F --> G{alpha < 1?}
    G -->|否| J
    G -->|是| H[随机 alpha 测试]
    H --> I{通过测试?}
    I -->|是| J
    I -->|否| K[从交点位置发射继续光线]
    K --> L[递归调用 Intersect]
    L --> M[累加 tHit 并返回]
    J --> N[SetIntersectionProperties<br>设置材质/光源/介质]
    N --> O[返回 ShapeIntersection]
```

## 依赖关系

- **依赖**：
  - `pbrt/pbrt.h` — 全局类型定义
  - `pbrt/base/light.h` — 光源基类
  - `pbrt/base/material.h` — 材质基类
  - `pbrt/base/medium.h` — 介质基类
  - `pbrt/base/shape.h` — 形状基类
  - `pbrt/base/texture.h` — 纹理基类
  - `pbrt/util/stats.h` — 内存统计计数器
  - `pbrt/util/taggedptr.h` — `TaggedPointer` 多态机制
  - `pbrt/util/transform.h` — 变换和动画变换
  - `pbrt/cpu/aggregates.h` — 加速结构（在 .cpp 中引用）
  - `pbrt/interaction.h` — 表面交互
  - `pbrt/materials.h` — 材质实现
  - `pbrt/shapes.h` — 形状实现
  - `pbrt/textures.h` — 纹理实现

- **被依赖**：
  - `pbrt/cpu/aggregates.h` — 加速结构存储 `Primitive` 数组
  - `pbrt/cpu/integrators.h` — 积分器持有聚合体 `Primitive`
  - `pbrt/scene.h` — 场景管理中使用 `Primitive`
  - `pbrt/wavefront/aggregate.h` — GPU 波前路径中引用 `Primitive`
