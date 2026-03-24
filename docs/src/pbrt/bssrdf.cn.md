# bssrdf.h / bssrdf.cpp — 次表面散射完整实现

## 第一部分：现象与问题

### 什么是次表面散射

当光照射到皮肤、蜡烛、牛奶、大理石这类半透明材质时，光线不会像金属那样在表面直接反射回来——它会**穿透表面、进入材质内部**，在微观粒子间反复弹跳（散射），同时一部分能量被吸收，最终从**另一个位置**钻出表面。这就是次表面散射（Subsurface Scattering）。

这种现象解释了为什么皮肤在背光下会透出红色（光在真皮层中散射后从较远处出来），为什么蜡烛边缘发光（光在蜡中传播后从旁边射出）。

### BSSRDF vs BRDF

普通的 BRDF 假设光在**同一个点**进出——入射点等于出射点。这对金属、塑料等不透光材质足够了。但对半透明材质，光从 $p_i$ 点进入，可能从几毫米之外的 $p_o$ 点出来。BSSRDF 正是描述这种"光从 A 进、B 出"的完整函数：

$$S(p_o, \omega_o, p_i, \omega_i)$$

它是一个**八维函数**（入射位置 2D + 入射方向 2D + 出射位置 2D + 出射方向 2D），完整计算的开销根本不可接受。因此我们需要近似。

## 第二部分：核心近似思路

### 可分离近似——把 8 维降为 3 个独立因子

pbrt 的核心策略是将 $S$ 拆成三个**可独立处理**的因子（`bssrdf.h:113-119` 构造函数中体现）：

$$S(p_o, \omega_o, p_i, \omega_i) \approx \underbrace{(1 - F_r(\cos\theta_i))}_{\text{入射菲涅尔}} \cdot \underbrace{S_p(p_o, p_i)}_{\text{空间散射}} \cdot \underbrace{S_w(\omega_o)}_{\text{出射方向}}$$

每个因子各自处理一个维度：

| 因子 | 物理含义 | 维度 | pbrt 中的实现 |
|---|---|---|---|
| $1 - F_r(\cos\theta_i)$ | 多少光能穿透表面进入介质 | 入射方向 | 菲涅尔透射公式 |
| $S_p(p_o, p_i)$ | 光在介质内从 $p_i$ 传播到 $p_o$ 的散射分布 | 两点空间关系 | 查表插值（核心） |
| $S_w(\omega_o)$ | 光从介质内穿透表面出射的方向分布 | 出射方向 | `NormalizedFresnelBxDF` |

进一步，在各向同性假设下，$S_p(p_o, p_i)$ 只依赖两点间的**欧氏距离** $r = \|p_o - p_i\|$，退化为一维函数 $S_r(r)$（`bssrdf.h:122`）。

### 光束扩散——拆解空间散射函数

空间散射函数 $S_r(r)$ 描述"距离入射点 $r$ 处能出来多少光"。pbrt 用**光束扩散**（Beam Diffusion）理论将其分解为两部分：

- **多次散射**（MS）：光在介质中弹跳多次后出来。占主导地位，用偶极子扩散近似计算。→ `BeamDiffusionMS`
- **单次散射**（SS）：光只弹跳一次就出来。贡献较小但在近场重要，精确计算。→ `BeamDiffusionSS`

$$S_r(r) = S_r^{MS}(r) + S_r^{SS}(r)$$

### 预计算策略——为什么可以提前算好

关键观察：$S_r(r)$ 在归一化到"光学空间"（令 $\sigma_t = 1$）后，只取决于两个参数——**单散射反照率** $\rho$ 和**距离** $r$。材质的具体消光系数只是一个缩放因子。

这意味着我们可以在渲染开始前，对一组 $(\rho, r)$ 的离散网格预先算好 $S_r$，存成二维查找表。渲染时只需查表插值，避免每条光线都做昂贵的数值积分。

## 第三部分：三大核心流程

下面从渲染管线的时间顺序组织，展示预计算、采样、求值三个流程如何协作。

### 流程 A — 预计算（渲染开始前）

```
ComputeBeamDiffusionBSSRDF(g, eta, table)     bssrdf.cpp:99-128
    │
    ├── 离散化半径 r[]：几何级数（近场密、远场疏）
    │       r[0]=0, r[1]=2.5e-3, r[i]=r[i-1]×1.2
    │
    ├── 离散化反照率 ρ[]：指数间距（低反照率更密）
    │       ρ[i] = (1 - e^{-8i/(n-1)}) / (1 - e^{-8})
    │
    └── 对每个 ρ[i]（并行）：
            │
            ├── 对每个 r[j]：
            │       profile[i][j] = 2πr × (BeamDiffusionSS + BeamDiffusionMS)
            │                              ↑ 单次散射         ↑ 多次散射
            │
            └── IntegrateCatmullRom → rhoEff[i] 和 profileCDF[i][]
```

**输出**：`BSSRDFTable`——一张二维表，存储散射剖面 `profile`、有效反照率 `rhoEff`、累积分布函数 `profileCDF`。

profile 存储时乘以了 $2\pi r$（极坐标积分的角度边际化），后续查表时再除回去。

### 流程 B — 采样（渲染进行中，每条光线）

当积分器遇到次表面散射材质，需要采样一个出射点。调用链：

```
BSSRDF::SampleSp(u1, u2)                       base/bssrdf.h:30-31
    │ Dispatch →
    TabulatedBSSRDF::SampleSp(u1, u2)           bssrdf.h:205-232
        │
        ├── Step 1: 选投影轴
        │     u1 < 0.25 → X 轴 (25%)
        │     u1 < 0.50 → Y 轴 (25%)
        │     u1 ≥ 0.50 → Z 轴/法线方向 (50%)
        │
        ├── Step 2: 极坐标采样
        │     r = SampleSr(u2[0])  ← 逆 CDF 采样，只用第 1 个波长通道
        │     φ = 2π × u2[1]
        │
        ├── Step 3: 计算探测线段
        │     r_max = SampleSr(0.999)
        │     l = 2√(r_max² - r²)
        │     pStart = po + r·(cosφ·f.x + sinφ·f.y) - l·f.z/2
        │     pTarget = pStart + l·f.z
        │
        └── 返回 BSSRDFProbeSegment{pStart, pTarget}
```

积分器拿到探测线段后，沿 `p0 → p1` 做光线追踪寻找表面交点。

### 流程 C — 求值（找到出射点后）

积分器找到交点 `si` 后，将其转换为完整的采样结果：

```
BSSRDF::ProbeIntersectionToSample(si, scratchBuffer)    base/bssrdf.h:33-34
    │ DispatchCPU →
    TabulatedBSSRDF::ProbeIntersectionToSample(si, bxdf) bssrdf.h:257-263
        │
        ├── 创建出射 BxDF: NormalizedFresnelBxDF(eta)
        ├── 出射方向: wo = si.ns（扩散近似下为法线方向）
        ├── 计算散射值: Sp(si.p()) → Sr(Distance(po, pi))  → 查表插值
        ├── 计算采样 PDF: PDF_Sp(si.p(), si.n)              → 3 轴合并
        │
        └── 返回 BSSRDFSample{Sp, pdf, bsdf, wo}
```

三个流程的关系一目了然：**流程 A 生成查找表 → 流程 B 用表采样出射点 → 流程 C 用表计算出射点处的散射值和 PDF**。

## 第四部分：物理模型详解

### BeamDiffusionMS — 多次散射（`bssrdf.cpp:26-75`）

多次散射基于**偶极子扩散近似**——把光在介质中的多次散射建模为扩散方程的解。直觉上，经过足够多次散射后，光的分布变得平滑、可预测，类似热扩散。

#### 参数准备

首先将各向异性散射参数 $g$ "折叠"为等效的各向同性散射（`bssrdf.cpp:31-33`）：

| 量 | 公式 | 含义 |
|---|---|---|
| 约化散射系数 $\sigma'_s$ | $\sigma_s(1-g)$ | 折叠各向异性后的等效散射强度 |
| 约化消光系数 $\sigma'_t$ | $\sigma_a + \sigma'_s$ | 等效总衰减 |
| 约化反照率 $\rho'$ | $\sigma'_s / \sigma'_t$ | 等效单散射反照率 |

#### 扩散系数

经典扩散系数 $D = 1/(3\sigma'_t)$ 在高吸收介质中精度不足。pbrt 使用 **Grosjean 修正**（`bssrdf.cpp:37`）：

$$D_g = \frac{2\sigma_a + \sigma'_s}{3{\sigma'_t}^2}$$

由此得到有效传输系数（`bssrdf.cpp:40`）：

$$\sigma_{tr} = \sqrt{\sigma_a / D_g}$$

#### 边界条件

偶极子方法通过在边界外放置一个**虚源**来满足零净通量边界条件。线性外推距离（`bssrdf.cpp:44-45`）：

$$z_e = \frac{-2D_g(1 + 3 F_{m2})}{1 - 2 F_{m1}}$$

其中 $F_{m1}$、$F_{m2}$ 是菲涅尔矩，描述界面对不同角度光线的平均反射特性。

出射标度因子（`bssrdf.cpp:49`）：

$$c_\Phi = \frac{1 - 2 F_{m1}}{4}, \quad c_E = \frac{1 - 3 F_{m2}}{2}$$

#### 数值积分（100 个样本）

对 100 个指数分布采样的实源深度 $z_r$（`bssrdf.cpp:51-73`），每个深度执行：

1. **虚源深度**：$z_v = -z_r + 2z_e$

2. **实源/虚源到出射点的距离**：
   $d_r = \sqrt{r^2 + z_r^2}$，$d_v = \sqrt{r^2 + z_v^2}$

3. **偶极子通量率**（`bssrdf.cpp:60-61`）：
$$\Phi_D = \frac{1}{4\pi D_g}\left(\frac{e^{-\sigma_{tr} d_r}}{d_r} - \frac{e^{-\sigma_{tr} d_v}}{d_v}\right)$$

4. **偶极子矢量辐照度**（法线分量，`bssrdf.cpp:65-67`）：
$$E_{Dn} = \frac{1}{4\pi}\left(\frac{z_r(1 + \sigma_{tr} d_r) e^{-\sigma_{tr} d_r}}{d_r^3} - \frac{z_v(1 + \sigma_{tr} d_v) e^{-\sigma_{tr} d_v}}{d_v^3}\right)$$

5. **合并**，乘以 $\kappa$ 修正因子抑制近场非物理贡献（`bssrdf.cpp:70-72`）：

$$\kappa = 1 - e^{-2\sigma'_t(d_r + z_r)}$$
$$E_d \mathrel{+}= \kappa \cdot {\rho'}^2 \cdot (\Phi_D \cdot c_\Phi + E_{Dn} \cdot c_E)$$

最终返回 $E_d / 100$。

### BeamDiffusionSS — 单次散射（`bssrdf.cpp:77-97`）

单次散射精确计算光仅在介质内弹跳**一次**就从表面射出的贡献。不使用扩散近似。

**几何直觉**：光从入射点折射进入介质，沿折射方向传播到深度 $t$，在那里散射一次，然后直线传播到距入射点水平距离 $r$ 处的出射点。

**临界深度**（`bssrdf.cpp:80`）：
$$t_{crit} = r\sqrt{\eta^2 - 1}$$

这是光线能到达水平距离 $r$ 处所需的最小深度——更浅的深度几何上不可能连接到距离 $r$ 的出射点。

**数值积分**（100 个样本，`bssrdf.cpp:84-96`）：

从 $t_{crit}$ 开始，对指数采样的深度 $t_i$：

1. 连接距离：$d = \sqrt{r^2 + t_i^2}$
2. 出射余弦角：$\cos\theta_o = t_i / d$
3. 累加贡献——包含 Henyey-Greenstein 相函数（描述散射方向偏好）和菲涅尔透射（光穿出表面的概率）：

$$E_{ss} \mathrel{+}= \rho \cdot \frac{e^{-\sigma_t(d + t_{crit})}}{d^2} \cdot p_{HG}(\cos\theta_o, g) \cdot (1 - F_r(-\cos\theta_o, \eta)) \cdot |\cos\theta_o|$$

### 预计算表的离散化策略

为什么不用均匀间距？因为散射剖面的形状决定了需要非均匀采样：

**半径**（几何级数 `r[i] = r[i-1] × 1.2`）：散射剖面在近场（小 $r$）急剧变化、远场（大 $r$）平缓衰减。几何级数自然在近场密集、远场稀疏，匹配这一特性。

**反照率**（指数间距）：低反照率（$\rho \to 0$）区域的散射剖面形状变化更剧烈——吸收主导时，散射剖面快速坍缩。指数间距在此区域提供更多采样点。

## 第五部分：查表与采样的数学细节

### Sr(r) — 查表插值（`bssrdf.h:125-158`）

这是查表的核心路径。逐波长处理：

1. **物理空间 → 光学空间**：$r_{optical} = r \cdot \sigma_t[i]$

   预计算表在 $\sigma_t = 1$ 的光学空间中建立。将物理距离乘以消光系数得到无量纲的光学距离。

2. **Catmull-Rom 2D 张量样条插值**：分别在 $\rho$ 和 $r_{optical}$ 两个维度计算 4 个样条权重，然后做 $4 \times 4$ 张量积插值：

$$sr = \sum_{j=0}^{3}\sum_{k=0}^{3} w_\rho[j] \cdot w_r[k] \cdot \text{profile}[\rho_{off}+j][r_{off}+k]$$

3. **消除边际 PDF 因子**：profile 存储时乘了 $2\pi r$，查表后除回：$sr \mathrel{/}= 2\pi r_{optical}$

4. **光学空间 → 物理空间**：所有波长完成后，$S_r \mathrel{*}= \sigma_t^2$

   $\sigma_t^2$ 来自二维空间坐标变换的雅可比行列式。

5. `ClampZero` 钳位到非负值，防止样条插值产生的轻微负值。

### SampleSr(u) — 逆 CDF 采样（`bssrdf.h:161-167`）

使用 `SampleCatmullRom2D` 从预计算的 CDF 中逆变换采样径向距离。

**关键设计**：只使用**第一个波长通道**（`rho[0]`、`sigma_t[0]`）进行采样。原因是不同波长的散射剖面形状不同，不可能同时对所有波长做完美重要性采样。只用一个波长采样，其他波长的偏差通过 `PDF_Sp` 中的多波长 MIS 权重补偿。

### PDF_Sr(r) — 逐波长 PDF（`bssrdf.h:170-202`）

结构与 `Sr()` 类似，但有两个区别：

1. 同步插值有效反照率 `rhoEff`，用于归一化
2. 归一化为概率密度：$\text{pdf}[i] = sr \cdot \sigma_t[i]^2 / \rho_{eff}$

除以 $\rho_{eff}$（profile 的积分）确保 PDF 在 $[0, \infty)$ 上积分为 1。

### 3 轴投影采样的几何意义（`bssrdf.h:205-254`）

`SampleSp` 在入射点周围的圆盘上采样，然后沿某个轴方向发射探测线段穿过材质寻找出射点。

**为什么需要 3 个轴？** 如果只沿法线方向投影，对于弯曲表面（如球体），探测线段可能与表面近乎平行，很难找到交点。加入 X、Y 轴方向的投影增强了对复杂几何的鲁棒性。

**概率分配**：Z 轴（法线方向）50%——大部分光在表面附近散射，沿法线探测最高效；X、Y 各 25%——覆盖切平面方向的变化。

**PDF_Sp 中的轴合并**（`bssrdf.h:235-254`）：

$$\text{pdf} = \sum_{axis=0}^{2} \text{PDF\_Sr}(r_{proj}[axis]) \cdot |n_{local}[axis]| \cdot p_{axis}$$

$|n_{local}[axis]|$ 因子实现了从立体角 PDF 到面积 PDF 的转换——当探测线段与表面近乎平行时（$|n_{local}[axis]| \to 0$），该轴几乎不会产生有效交点，权重自然降低。

## 第六部分：数据结构与 API 参考

### TabulatedBSSRDF（`bssrdf.h:105-276`）

唯一的 BSSRDF 具体实现，基于预计算查找表和 Catmull-Rom 样条插值。

**私有成员**（`bssrdf.h:270-275`）：

| 成员 | 类型 | 说明 |
|---|---|---|
| `po` | `Point3f` | 入射点位置 |
| `wo` | `Vector3f` | 入射方向 |
| `ns` | `Normal3f` | 着色法线 |
| `eta` | `Float` | 折射率 |
| `sigma_t` | `SampledSpectrum` | 消光系数 $\sigma_a + \sigma_s$ |
| `rho` | `SampledSpectrum` | 单散射反照率 $\sigma_s / \sigma_t$ |
| `table` | `const BSSRDFTable*` | 预计算查找表指针 |

**公开方法**：

| 方法 | 说明 |
|---|---|
| `Sp(pi)` | 空间散射函数，委托给 `Sr(Distance(po, pi))` |
| `Sr(r)` | 径向散射剖面，查表 + Catmull-Rom 2D 插值 |
| `SampleSr(u)` | 逆 CDF 采样径向距离（仅用第 1 波长通道） |
| `PDF_Sr(r)` | 逐波长概率密度 |
| `SampleSp(u1, u2)` | 空间探测线段采样（3 轴投影 + 极坐标） |
| `PDF_Sp(pi, ni)` | 空间采样 PDF（3 轴合并） |
| `ProbeIntersectionToSample(si, bxdf)` | 将探测交点转换为 `BSSRDFSample` |

### BSSRDFTable（`bssrdf.h:74-92`）

预计算散射分布查找表。

| 成员 | 类型 | 说明 |
|---|---|---|
| `rhoSamples` | `pstd::vector<Float>` | 反照率采样点（指数间距） |
| `radiusSamples` | `pstd::vector<Float>` | 光学半径采样点（几何级数） |
| `profile` | `pstd::vector<Float>` | 二维散射剖面，一维展开：`profile[i * nRadius + j]` |
| `rhoEff` | `pstd::vector<Float>` | 每个反照率的有效反照率（profile 积分值） |
| `profileCDF` | `pstd::vector<Float>` | 每行的 CDF，用于逆变换采样 |

### BSSRDFSample（`bssrdf.h:25-29`）

| 成员 | 类型 | 说明 |
|---|---|---|
| `Sp` | `SampledSpectrum` | 出射点的散射光谱值 |
| `pdf` | `SampledSpectrum` | 逐波长采样 PDF（用于光谱 MIS） |
| `Sw` | `BSDF` | 出射处的 BSDF（封装 `NormalizedFresnelBxDF`） |
| `wo` | `Vector3f` | 出射方向（设为法线方向） |

### SubsurfaceInteraction（`bssrdf.h:32-65`）

从 `SurfaceInteraction` 提取的轻量几何数据。

| 成员 | 说明 |
|---|---|
| `pi` | 交点位置（带浮点误差区间） |
| `n` / `ns` | 几何法线 / 着色法线 |
| `dpdu, dpdv` | 几何偏导数 |
| `dpdus, dpdvs` | 着色偏导数 |

### BSSRDFProbeSegment（`bssrdf.h:95-102`）

探测线段，由起点 `p0` 和终点 `p1` 定义。积分器沿此线段追踪寻找出射点。

### TaggedPointer 分派（`bssrdf.h:291-304`）

| 方法 | 分派方式 | 原因 |
|---|---|---|
| `BSSRDF::SampleSp` | `Dispatch`（GPU/CPU 通用） | 高频路径，需要 GPU 支持 |
| `BSSRDF::ProbeIntersectionToSample` | `DispatchCPU`（仅 CPU） | 需要 `ScratchBuffer` 动态分配 BxDF |

## 第七部分：辅助函数

### SubsurfaceFromDiffuse — 参数反推（`bssrdf.h:279-289`）

```cpp
void SubsurfaceFromDiffuse(const BSSRDFTable &t,
                           const SampledSpectrum &rhoEff,
                           const SampledSpectrum &mfp,
                           SampledSpectrum *sigma_a,
                           SampledSpectrum *sigma_s)
```

用户通常不直接指定物理散射系数 $\sigma_a$、$\sigma_s$，而是使用更直观的参数：

- **有效反照率** $\rho_{eff}$：材质在漫反射照明下的表观亮度（0 = 纯吸收，1 = 纯散射）
- **平均自由程** $mfp$：光在介质中的平均传播距离，控制散射的空间尺度

该函数逐波长通道反推：

1. 通过逆 Catmull-Rom 插值从 `rhoEff` 表反查单散射反照率 $\rho$
2. $\sigma_s = \rho / mfp$
3. $\sigma_a = (1 - \rho) / mfp$
