# rgb2spec_opt — RGB 到光谱查找表生成工具

## 1. 概述与动机

基于物理的渲染器需要在光谱域（而非 RGB 域）中进行光照计算——只有使用光谱反射率，折射、色散、荧光等波长相关现象才能被正确模拟。然而，现实中绝大多数材质纹理和颜色输入都是 RGB 格式的。因此需要一种方法将 RGB 值"提升"为合理的光谱反射率曲线。

`rgb2spec_opt` 的方案是**离线预计算**：对整个 RGB 色彩空间进行离散化采样，对每个采样点用数值优化求出一组多项式系数，使得该系数通过 sigmoid 函数生成的光谱曲线在特定光源下积分回的 RGB 值与原始 RGB 尽可能匹配。所有系数存入一张三维查找表，渲染时只需三线性插值即可在 O(1) 时间内完成 RGB→光谱转换。

该方法基于 Wenzel Jakob 和 Johannes Hanika 的工作（*A Low-Dimensional Function Space for Efficiently Storing Spectral Upsampling Coefficients*）。

## 2. 光谱模型：Sigmoid 多项式

### 模型定义

用一个二次多项式通过 sigmoid 函数来参数化反射率光谱：

$$S(\lambda) = \sigma\bigl(c_0 \lambda^2 + c_1 \lambda + c_2\bigr)$$

其中 $\lambda$ 为波长（纳米），$c_0, c_1, c_2$ 为待求的三个系数。

### sigmoid 函数

使用如下形式的 sigmoid（而非常见的 logistic 函数）：

$$\sigma(x) = \frac{1}{2} + \frac{x}{2\sqrt{1 + x^2}}$$

值域为 $(0, 1)$，保证生成的反射率始终物理合理（反射率必须在 0 到 1 之间）。

### 为什么选择这个模型

- **sigmoid 映射**：将任意实数域的多项式值压缩到 $(0, 1)$，无需额外裁剪。
- **二次多项式**：仅 3 个系数，自由度足以描述大多数自然反射率光谱的基本形状（单峰、单谷、单调递增/递减等），同时存储开销极小。
- **上述 sigmoid 形式**（代替 logistic 函数 $1/(1+e^{-x})$）计算更快，且数值性质更好。

对应源码（`rgb2spec_opt.cpp:362`）：

```cpp
double sigmoid(double x) {
    return 0.5 * x / std::sqrt(1.0 + x * x) + 0.5;
}
```

运行时定义在 `color.h:357`：

```cpp
static Float s(Float x) {
    if (IsInf(x))
        return x > 0 ? 1 : 0;
    return .5f + x / (2 * std::sqrt(1 + Sqr(x)));
}
```

## 3. 查找表的参数化结构

### 三个子表

RGB 色彩空间被分成三个区域——按最大分量是 R、G 还是 B 来划分。每个区域对应一个子表，共 3 个子表。这种拆分消除了三个分量之间的排列对称性问题，将三维 RGB 立方体映射到了一个规范化的参数空间。

### 三维坐标含义

设某个 RGB 值的最大分量索引为 `maxc`，最大分量值为 `z_val = rgb[maxc]`。

| 轴 | 含义 | 范围 |
|---|---|---|
| **z 轴** | 最大分量值 `z_val`，通过 `Scale` 数组做非均匀映射 | `[0, 1]` |
| **x 轴** | `rgb[(maxc+1)%3] / z_val`（循环下一个分量与最大分量的比值） | `[0, 1]` |
| **y 轴** | `rgb[(maxc+2)%3] / z_val`（循环再下一个分量与最大分量的比值） | `[0, 1]` |

### z 轴的非均匀采样

z 轴使用双重 smoothstep 进行非均匀参数化，使得低亮度区域的采样密度更高（低亮度区域人眼更敏感，且优化更困难）：

```cpp
scale[k] = smoothstep(smoothstep(k / double(res - 1)));
```

其中 `smoothstep(x) = x² · (3 - 2x)`。`Scale` 数组存储了每个 z 层对应的实际亮度值。

### 存储布局

每个格点存储 3 个 `float` 系数 $(c_0, c_1, c_2)$，总数据结构为：

```cpp
float Scale[res];                      // z 轴非均匀采样的节点值
float Data[3][res][res][res][3];       // [子表][z][y][x][系数]
```

维度顺序：`Data[maxc][zi][yi][xi][coeff_idx]`。

以 `res = 64` 为例，总大小为 `3 × 64 × 64 × 64 × 3 × 4 bytes ≈ 9.4 MB`。

## 4. 预计算表的初始化

`init_tables()` 函数（`rgb2spec_opt.cpp:408`）完成以下工作：

### 4.1 选择光源与转换矩阵

根据目标色域选择匹配的标准光源（illuminant）和 XYZ↔RGB 转换矩阵：

| 色域 | 光源 |
|---|---|
| sRGB, REC2020, DCI_P3 | CIE D65 |
| ProPhoto RGB | CIE D50 |
| ACES2065-1 | CIE D60 |
| eRGB, XYZ | CIE E（等能光源）|

### 4.2 预计算积分权重表 `rgb_tbl`

将连续积分离散化为求和形式。对于 `rgb_tbl[k][i]`，其含义为：

$$\text{rgb\_tbl}[k][i] = w_i \cdot \sum_{j=0}^{2} \text{xyz\_to\_rgb}[k][j] \cdot \overline{c}_j(\lambda_i) \cdot I(\lambda_i)$$

其中：
- $\overline{c}_j(\lambda)$ 是 CIE 1931 颜色匹配函数 $(\bar{x}, \bar{y}, \bar{z})$
- $I(\lambda)$ 是光源光谱功率分布
- $w_i$ 是 Simpson 3/8 积分规则的权重

有了 `rgb_tbl`，从任意反射率光谱 $S(\lambda)$ 计算 RGB 只需做一次点积：

$$\text{rgb}[k] = \sum_{i} \text{rgb\_tbl}[k][i] \cdot S(\lambda_i)$$

### 4.3 Simpson 3/8 积分规则

波长范围 360—830nm 被细分为 `CIE_FINE_SAMPLES = (95-1)×3+1 = 283` 个采样点（在原始 5nm 间隔的每个区段内取 4 个点）。权重分配：

| 采样点位置 | 权重倍率 |
|---|---|
| 首尾端点 | ×1 |
| 每 3 个一组中的第 3 个（即 `(i-1)%3 == 2`） | ×2 |
| 其余 | ×3 |

基础权重为 $\frac{3}{8}h$，其中 $h = \frac{830 - 360}{282}$。

### 4.4 白点计算

#### 白点的物理含义

白点（white point）是光源本身在 CIE XYZ 空间中的三刺激值，代表"完美白色反射体（反射率 $S(\lambda) \equiv 1$）在该光源下的颜色"。计算公式：

$$X_w = \int \bar{x}(\lambda) \cdot I(\lambda) \, d\lambda, \quad Y_w = \int \bar{y}(\lambda) \cdot I(\lambda) \, d\lambda, \quad Z_w = \int \bar{z}(\lambda) \cdot I(\lambda) \, d\lambda$$

离散化时复用与 `rgb_tbl` 相同的 Simpson 3/8 权重（`rgb2spec_opt.cpp:483`）：

```cpp
for (int i = 0; i < 3; ++i)
    xyz_whitepoint[i] += xyz[i] * I * weight;   // i = 0,1,2 对应 X,Y,Z
```

#### 白点在 L\*a\*b\* 中的作用

CIE L\*a\*b\* 转换公式中，XYZ 值需要除以白点做归一化（`cie_lab()`，`rgb2spec_opt.cpp:374`）：

- $L^* = 116 \cdot f(Y / Y_w) - 16$
- $a^* = 500 \cdot \bigl(f(X / X_w) - f(Y / Y_w)\bigr)$
- $b^* = 200 \cdot \bigl(f(Y / Y_w) - f(Z / Z_w)\bigr)$

其中 $f(t)$ 为 CIE 标准的分段函数：当 $t > \delta^3$（$\delta = 6/29$）时 $f(t) = \sqrt[3]{t}$，否则 $f(t) = t/(3\delta^2) + 4/29$。

白点归一化确保在参考光源下，完美白色反射体映射到 $L^*a^*b^* = (100, 0, 0)$。不同色域使用不同光源（D65/D50/D60/E），白点也不同，所以必须为每个色域分别计算。

#### 变量遮蔽说明

上述白点累加的内层循环变量 `i` 与外层波长循环（第 463 行）使用了相同的变量名：

```cpp
for (int i = 0; i < CIE_FINE_SAMPLES; ++i) {  // 外层 i: 0..282
    double xyz[3] = {...}, I = ...;
    double weight = ...;
    // rgb_tbl 累加（使用外层 i）...

    for (int i = 0; i < 3; ++i)                // 内层 i: 新声明，0..2（遮蔽外层 i）
        xyz_whitepoint[i] += xyz[i] * I * weight;
}
```

内层 `for (int i = 0; i < 3; ++i)` 声明了一个**新的** `i`，遮蔽（shadow）了外层循环的 `i`。C++ 语义上完全合法——内层 `i` 仅在内层 for 的作用域中有效，外层 `i` 在内层循环结束后不受影响。`xyz[]`、`I`、`weight` 在进入内层循环前已由外层 `i` 计算完毕，所以结果正确。这是代码风格问题（`-Wshadow` 会报警告），不是逻辑 bug。

## 5. 优化过程：Gauss-Newton 迭代

### 5.1 优化目标

对每个目标 RGB 值，找到系数 $(c_0, c_1, c_2)$，使得由 sigmoid 多项式生成的光谱积分回的 RGB 值在 **CIE L\*a\*b\* 空间**中与目标 RGB 尽可能接近。

选择 L\*a\*b\* 空间是因为它是感知均匀的——在该空间中等距的颜色差异对人眼而言也是等距的，比直接在 RGB 空间优化能获得更好的视觉效果。

### 5.2 残差计算 `eval_residual()`

`eval_residual()`（`rgb2spec_opt.cpp:488`）计算当前系数下的重建误差：

1. 对每个波长采样点 $\lambda_i$，将波长归一化到 $[0,1]$：$t = \frac{\lambda_i - 360}{830 - 360}$
2. 计算多项式值：$p = c_0 \cdot t^2 + c_1 \cdot t + c_2$（注意：**优化阶段**在归一化波长空间工作）
3. 通过 sigmoid 得到反射率：$s = \sigma(p)$
4. 累加 `rgb_tbl[k][i] × s` 得到重建 RGB
5. 将重建 RGB 和目标 RGB 分别转换到 L\*a\*b\* 空间
6. 返回 L\*a\*b\* 差值作为残差向量（3 维）

### 5.3 雅可比矩阵 `eval_jacobian()`

`eval_jacobian()`（`rgb2spec_opt.cpp:516`）用**中心差分法**估计 3×3 雅可比矩阵：

$$J_{jk} = \frac{r_j(c_k + \varepsilon) - r_j(c_k - \varepsilon)}{2\varepsilon}$$

其中 $\varepsilon = 10^{-4}$（`RGB2SPEC_EPSILON`），$r_j$ 为残差的第 $j$ 个分量。

### 5.4 Gauss-Newton 求解

`gauss_newton()`（`rgb2spec_opt.cpp:533`）的每一步：

1. 计算当前残差 $\mathbf{r}$ 和雅可比矩阵 $\mathbf{J}$
2. 用 LU 分解（带部分主元选取）求解线性系统 $\mathbf{J} \cdot \Delta\mathbf{c} = \mathbf{r}$
3. 更新系数：$\mathbf{c} \leftarrow \mathbf{c} - \Delta\mathbf{c}$
4. **系数裁剪**：若三个系数中最大值超过 200，将所有系数按比例缩放回 200（防止数值溢出）
5. 终止条件：迭代 15 次或残差平方和 $< 10^{-6}$

## 6. 主循环：遍历色彩空间

### 6.1 循环结构

主循环（`rgb2spec_opt.cpp:825`）包含三层：

```
for l in 0..2:               // 三个子表（R-max, G-max, B-max）
  ParallelFor j in 0..res-1: // y 轴（循环第二个分量的比值）
    for i in 0..res-1:       // x 轴（循环第一个分量的比值）
      沿 z 轴扫描...
```

对每个 `(l, j, i)` 位置，构造 RGB 值的方式为：

```
rgb[l]          = scale[k]       // 最大分量 = z 轴对应的亮度
rgb[(l+1) % 3]  = x · scale[k]  // 循环下一个分量 = x比值 × 亮度
rgb[(l+2) % 3]  = y · scale[k]  // 循环再下一个分量 = y比值 × 亮度
```

### 6.2 z 轴扫描策略

对每个 `(x, y)` 位置，z 轴不是从 0 扫到 res-1，而是从中间向两端扫描：

1. **向上扫描**：从 `k = res/5` 到 `k = res-1`（亮度递增）
2. **重置系数为零**
3. **向下扫描**：从 `k = res/5` 到 `k = 0`（亮度递减）

这种策略的目的：在同一扫描方向上，相邻 z 层的 RGB 值亮度相近，前一层的优化结果是下一层的良好初始值，可显著加速 Gauss-Newton 收敛。从 `res/5`（而非 0 或 res-1）开始，是因为中等亮度区域的优化更容易收敛，可以为后续的极暗和极亮区域提供更好的初始猜测。

### 6.3 系数重参数化

优化阶段在**归一化波长空间** $t \in [0, 1]$ 中工作，多项式为：

$$A \cdot t^2 + B \cdot t + C, \quad t = \frac{\lambda - 360}{470}$$

但存入查找表的系数需要直接接受**纳米波长** $\lambda$，即：

$$c_0 \cdot \lambda^2 + c_1 \cdot \lambda + c_2$$

展开 $t = (\lambda - 360) / 470$ 并整理，得到变换公式（`rgb2spec_opt.cpp:845`）：

```
c₀ = A / 470²
c₁ = B / 470 − 2 · 360 · A / 470²
c₂ = C − 360 · B / 470 + 360² · A / 470²
```

## 7. 运行时查表过程

运行时查表逻辑定义在 `RGBToSpectrumTable::operator()(RGB)` 中（`color.cpp:31`）。步骤：

1. **特殊情况**：若 R = G = B（灰色），直接用解析公式计算系数，返回 `RGBSigmoidPolynomial(0, 0, c₂)`。
2. **选子表**：找到最大分量索引 `maxc`，选择 `Data[maxc]` 子表。
3. **归一化坐标**：
   - `x = rgb[(maxc+1)%3] / rgb[maxc] × (res-1)`
   - `y = rgb[(maxc+2)%3] / rgb[maxc] × (res-1)`
4. **z 轴定位**：在 `Scale` 数组中用二分搜索找到 `rgb[maxc]` 所在区间 `[zi, zi+1]`，计算插值权重 `dz`。
5. **三线性插值**：对 8 个相邻格点的 3 个系数分别做三线性插值。
6. **返回**：`RGBSigmoidPolynomial(c₀, c₁, c₂)`，后续可对任意波长 $\lambda$ 求值得到反射率 $\sigma(c_0\lambda^2 + c_1\lambda + c_2)$。

## 8. 使用方式与输出格式

### 命令行

```
rgb2spec_opt <resolution> <output> [<gamut>]
```

| 参数 | 说明 |
|---|---|
| `<resolution>` | 查找表每个维度的分辨率（如 64） |
| `<output>` | 输出的 C++ 源代码文件路径 |
| `[<gamut>]` | 色域名称（默认 sRGB） |

### 支持的色域

`sRGB`、`eRGB`、`XYZ`、`ProPhotoRGB`、`ACES2065_1`、`REC2020`、`DCI_P3`

### 输出格式

生成的 `.cpp` 文件包含 `pbrt` 命名空间下的三个常量：

```cpp
namespace pbrt {
extern const int    <gamut>ToSpectrumTable_Res;              // 分辨率
extern const float  <gamut>ToSpectrumTable_Scale[res];       // z 轴非均匀采样节点
extern const float  <gamut>ToSpectrumTable_Data[3][res][res][res][3]; // 系数数据
}
```

这些常量被编译进 pbrt 后，由 `RGBToSpectrumTable` 类在运行时引用。

## 9. 依赖关系

该工具**完全独立于 pbrt 渲染器的其他模块**。所有需要的数据和算法都内嵌在 `rgb2spec_opt.cpp` 单个文件中：

- CIE 1931 颜色匹配函数（$\bar{x}$, $\bar{y}$, $\bar{z}$，5nm 间隔，95 个采样点）
- 标准光源光谱（D65、D50、D60、E，均归一化到单位亮度）
- 各色彩空间的 XYZ↔RGB 转换矩阵
- LU 分解线性方程组求解器
- 简化的并行计算框架（`ThreadPool` + `ParallelFor`）

仅使用 C/C++ 标准库，无外部依赖。

## 关键源码文件参考

| 文件 | 内容 |
|---|---|
| `src/pbrt/cmd/rgb2spec_opt.cpp` | 查找表生成工具（本文档的主角） |
| `src/pbrt/util/color.h:331-395` | `RGBSigmoidPolynomial` 和 `RGBToSpectrumTable` 类定义 |
| `src/pbrt/util/color.cpp:31-68` | 运行时查表与三线性插值实现 |
| `src/pbrt/util/spectrum.h` | `RGBAlbedoSpectrum` 等使用查找表的光谱类 |
