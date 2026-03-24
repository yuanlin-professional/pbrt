# aggregates.h / aggregates.cpp

## 概述

该文件实现了 PBRT-v4 渲染器中的空间加速结构（Acceleration Structures），用于高效地组织场景中的几何图元，从而加速光线与场景的求交运算。文件提供了两种经典加速结构：**BVH（层次包围盒）** 和 **KdTree（k-d 树）**，它们是光线追踪渲染管线中至关重要的性能核心模块。工厂函数 `CreateAccelerator` 根据用户参数创建对应的加速结构实例。

## 主要类与接口

| 类/结构体/函数 | 说明 |
|---|---|
| `CreateAccelerator` | 工厂函数，根据名称（`"bvh"` 或 `"kdtree"`）和参数字典创建对应的加速结构 `Primitive` |
| `BVHAggregate` | 基于层次包围盒（Bounding Volume Hierarchy）的加速结构，支持多种划分策略 |
| `BVHAggregate::SplitMethod` | 枚举类型，定义 BVH 的划分策略：`SAH`（表面积启发式）、`HLBVH`（层次线性 BVH）、`Middle`（中点划分）、`EqualCounts`（等量划分） |
| `BVHBuildNode` | BVH 构建阶段的树节点，包含包围盒、子节点指针和图元偏移信息 |
| `BVHPrimitive` | BVH 构建时每个图元的包围盒及索引信息 |
| `LinearBVHNode` | BVH 的紧凑线性存储节点（32 字节对齐），用于高效遍历 |
| `MortonPrimitive` | HLBVH 构建中使用的 Morton 编码图元，用于空间排序 |
| `LBVHTreelet` | HLBVH 中的子树片段，包含起始索引和构建节点 |
| `BVHSplitBucket` | SAH 划分中的桶结构，记录图元计数和包围盒 |
| `KdTreeAggregate` | 基于 k-d 树的加速结构，使用 SAH 启发式选择最优划分平面 |
| `KdTreeNode` | k-d 树节点（8 字节对齐），通过 flags 位域区分内部节点和叶子节点 |
| `BoundEdge` | k-d 树构建中表示图元包围盒在某轴上的边界事件 |
| `RadixSort` | 针对 Morton 编码的基数排序函数，用于 HLBVH 构建 |

## 架构图

```mermaid
classDiagram
    class Primitive {
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
    }

    class BVHAggregate {
        -maxPrimsInNode : int
        -primitives : vector~Primitive~
        -splitMethod : SplitMethod
        -nodes : LinearBVHNode*
        +BVHAggregate(prims, maxPrimsInNode, splitMethod)
        +Create(prims, parameters) BVHAggregate*
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
        -buildRecursive(...)
        -buildHLBVH(...)
        -emitLBVH(...)
        -buildUpperSAH(...)
        -flattenBVH(node, offset)
    }

    class KdTreeAggregate {
        -isectCost : int
        -traversalCost : int
        -maxPrims : int
        -emptyBonus : Float
        -primitives : vector~Primitive~
        -nodes : KdTreeNode*
        -bounds : Bounds3f
        +KdTreeAggregate(prims, isectCost, traversalCost, emptyBonus, maxPrims, maxDepth)
        +Create(prims, parameters) KdTreeAggregate*
        +Bounds() Bounds3f
        +Intersect(ray, tMax) ShapeIntersection
        +IntersectP(ray, tMax) bool
        -buildTree(...)
    }

    class LinearBVHNode {
        +bounds : Bounds3f
        +primitivesOffset : int
        +secondChildOffset : int
        +nPrimitives : uint16_t
        +axis : uint8_t
    }

    class KdTreeNode {
        +split : Float
        +onePrimitiveIndex : int
        +primitiveIndicesOffset : int
        +InitLeaf(primNums, primitiveIndices)
        +InitInterior(axis, aboveChild, s)
        +IsLeaf() bool
        +SplitAxis() int
        +SplitPos() Float
    }

    class BVHBuildNode {
        +bounds : Bounds3f
        +children : BVHBuildNode*[2]
        +splitAxis : int
        +firstPrimOffset : int
        +nPrimitives : int
        +InitLeaf(first, n, bounds)
        +InitInterior(axis, c0, c1)
    }

    Primitive <|-- BVHAggregate
    Primitive <|-- KdTreeAggregate
    BVHAggregate --> LinearBVHNode : 存储
    BVHAggregate --> BVHBuildNode : 构建时使用
    KdTreeAggregate --> KdTreeNode : 存储
```

## 算法流程图

### BVH 构建流程（SAH 策略）

```mermaid
flowchart TD
    A[输入图元列表] --> B[计算所有图元包围盒]
    B --> C{划分策略}
    C -->|SAH / Middle / EqualCounts| D[递归构建 buildRecursive]
    C -->|HLBVH| E[计算 Morton 编码]

    D --> F[计算质心包围盒]
    F --> G{图元数 <= 2?}
    G -->|是| H[等量划分]
    G -->|否| I[SAH 桶划分]
    I --> J[初始化 12 个桶]
    J --> K[分配图元到桶中]
    K --> L[前向扫描计算代价]
    L --> M[后向扫描计算代价]
    M --> N[选择最小代价桶]
    N --> O{分裂代价 < 叶代价?}
    O -->|是| P[分裂并递归构建子树]
    O -->|否| Q[创建叶节点]

    E --> R[基数排序]
    R --> S[构建 LBVH 子树]
    S --> T[SAH 合并子树]

    P --> U[展平为线性数组 flattenBVH]
    Q --> U
    T --> U
    H --> P
```

### 光线-BVH 求交流程

```mermaid
flowchart TD
    A[输入光线] --> B[计算方向倒数 invDir]
    B --> C[初始化遍历栈]
    C --> D{当前节点包围盒相交?}
    D -->|否| E{栈为空?}
    E -->|是| F[返回结果]
    E -->|否| G[弹出栈顶节点]
    G --> D
    D -->|是| H{是叶节点?}
    H -->|是| I[遍历叶节点中的图元求交]
    I --> J[更新最近交点 tMax]
    J --> E
    H -->|否| K[根据光线方向决定先近后远]
    K --> L[将远子节点压栈]
    L --> M[前进到近子节点]
    M --> D
```

## 依赖关系

- **依赖**：
  - `pbrt/pbrt.h` — 全局类型定义和宏
  - `pbrt/cpu/primitive.h` — `Primitive` 类型定义
  - `pbrt/util/parallel.h` — 并行计算工具（`ParallelFor`、`ThreadLocal`）
  - `pbrt/interaction.h` — `ShapeIntersection` 交互数据
  - `pbrt/paramdict.h` — 参数字典
  - `pbrt/shapes.h` — 形状接口
  - `pbrt/util/error.h` — 错误处理
  - `pbrt/util/log.h` — 日志系统
  - `pbrt/util/math.h` — 数学工具（`EncodeMorton3`、`Log2Int`、`FindInterval`）
  - `pbrt/util/memory.h` — 内存分配器
  - `pbrt/util/stats.h` — 性能统计计数器
  - `pbrt/util/print.h` — 格式化打印

- **被依赖**：
  - `pbrt/cpu/primitive.cpp` — 图元分派调用加速结构的求交方法
  - `pbrt/cpu/render.cpp` — CPU 渲染入口，创建加速结构
  - `pbrt/cpu/integrators_test.cpp` — 集成测试中构建 BVH
  - `pbrt/scene.cpp` — 场景构建中创建加速结构
  - `pbrt/wavefront/aggregate.cpp` — GPU 波前渲染路径中引用加速结构
