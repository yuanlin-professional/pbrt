# base/bssrdf.h — BSSRDF 接口定义

## 这是什么

BSSRDF（双向散射表面反射分布函数）描述光如何进入半透明材质、在内部散射、再从另一个位置射出。`base/bssrdf.h` 定义了这一机制的**抽象接口**——它不包含任何物理计算逻辑，只声明了两个方法签名和类型分派机制。

> 完整的物理模型、预计算流程、采样算法和数学推导，请参阅 [bssrdf.cn.md](../bssrdf.cn.md)。

## 接口设计

`BSSRDF` 继承自 `TaggedPointer<TabulatedBSSRDF>`（`base/bssrdf.h:25`），通过类型标签在运行时将方法调用分派到具体实现。当前唯一的具体实现是 `TabulatedBSSRDF`（定义在 `pbrt/bssrdf.h`），但架构上支持未来扩展其他 BSSRDF 类型。

```
BSSRDF (TaggedPointer)
  └── TabulatedBSSRDF  ← 基于预计算查找表的实现（当前唯一）
```

## 两个接口方法

渲染管线通过以下两个方法与 BSSRDF 交互，它们构成一次完整的次表面散射采样的前后两步：

### 第一步：`SampleSp` — 生成探测线段

```cpp
PBRT_CPU_GPU inline pstd::optional<BSSRDFProbeSegment>
SampleSp(Float u1, Point2f u2) const;           // base/bssrdf.h:30-31
```

从入射点周围按散射剖面的概率分布采样一条**探测线段**（`BSSRDFProbeSegment`，由起点 `p0` 和终点 `p1` 定义）。积分器随后沿此线段做光线追踪，寻找材质表面的交点作为可能的出射点。

- **分派方式**：`Dispatch`（GPU/CPU 通用）——这是高频路径，每条次表面散射光线都会调用。
- 采样失败时返回空（例如消光系数为零、采样半径超出有效范围）。

### 第二步：`ProbeIntersectionToSample` — 将交点转为采样结果

```cpp
inline BSSRDFSample
ProbeIntersectionToSample(const SubsurfaceInteraction &si,
                          ScratchBuffer &scratchBuffer) const;  // base/bssrdf.h:33-34
```

积分器沿探测线段找到表面交点后，调用此方法将交点信息转换为完整的 `BSSRDFSample`——包含散射光谱值 `Sp`、概率密度 `pdf`、出射处的 BSDF `Sw` 和出射方向 `wo`。

- **分派方式**：`DispatchCPU`（仅 CPU）——需要通过 `ScratchBuffer` 动态分配出射处的 BxDF 对象。
- 分派 lambda 通过模板类型推导自动获取具体实现关联的 BxDF 类型（`TabulatedBSSRDF` 关联 `NormalizedFresnelBxDF`）。

## 前向声明的类型

该文件还前向声明了以下类型，它们的完整定义在 `pbrt/bssrdf.h` 中：

| 类型 | 用途 |
|---|---|
| `BSSRDFSample` | 采样结果（光谱值 + PDF + 出射 BSDF + 出射方向） |
| `BSSRDFProbeSegment` | 探测线段（起点 `p0` → 终点 `p1`） |
| `SubsurfaceInteraction` | 次表面交互点的轻量几何数据 |
| `BSSRDFTable` | 预计算散射分布查找表 |
| `TabulatedBSSRDF` | 唯一的 BSSRDF 具体实现 |
