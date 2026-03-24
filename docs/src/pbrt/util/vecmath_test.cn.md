# vecmath_test.cpp

## 概述
该测试文件验证 PBRT-v4 中向量数学库的正确性，覆盖二维/三维向量和点的基本运算、双线性插值逆变换、向量夹角计算、坐标系构建、包围盒迭代器和距离计算、包围盒合并、等面积球面映射、方向锥体操作、球面三角形面积计算、区间类型向量运算以及八面体向量编码解码。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Vector2, Basics) | 测试二维向量的基本运算：构造、类型转换、相等/不等比较、加减法、标量乘除、绝对值、向上/向下取整、逐分量最小/最大值、最小/最大分量值及索引、分量排列 |
| TEST(Vector3, Basics) | 测试三维向量的基本运算：与二维向量类似的全套操作，包括构造、算术运算、取整、逐分量操作、极值查找和分量排列 |
| TEST(Point2, InvertBilinear) | 测试二维点的双线性插值逆变换 `InvertBilinear`：使用简单矩形、平移矩形和随机参数，验证通过双线性插值得到的点能正确逆变换回原始参数 |
| TEST(Vector, AngleBetween) | 测试向量夹角函数 `AngleBetween` 的精度：(1) 相同向量夹角为 0；(2) 反向向量夹角为 Pi；(3) 正交向量夹角为 Pi/2；(4) 100000 组随机向量对的相对误差 < 5e-6；(5) 100000 组近似反向向量对的精度测试；(6) 特定近似反向向量的精确性验证 |
| TEST(Vector, CoordinateSystem) | 测试从单一向量构建正交坐标系 `CoordinateSystem`：(1) 六个坐标轴方向生成的辅助向量在主方向分量为零；(2) Duff 等人论文中的病态向量也能生成高质量正交系（误差 < 1e-10）；(3) 1000 组随机向量的正交性验证 |
| TEST(Bounds2, IteratorBasic) | 测试二维整数包围盒的迭代器：验证 range-for 遍历顺序和产生的点坐标正确性 |
| TEST(Bounds2, IteratorDegenerate) | 测试退化（零面积或默认构造）二维包围盒的迭代器不会执行循环体 |
| TEST(Bounds3, PointDistance) | 测试三维包围盒与点之间的距离计算：包围盒内部/面上的点距离为 0、与面对齐的外部点、仅两个维度在范围内的点、一般位置点的距离/距离平方，以及不规则包围盒的验证 |
| TEST(Bounds2, Union) | 测试二维包围盒合并操作：正常包围盒与退化包围盒合并、退化包围盒自身合并、包围盒与点的合并 |
| TEST(Bounds3, Union) | 测试三维包围盒合并操作：与二维类似的各种合并场景 |
| TEST(EqualArea, Randoms) | 测试等面积球面-正方形双向映射：100 组随机均匀球面点经 `EqualAreaSphereToSquare` 映射到正方形再经 `EqualAreaSquareToSphere` 映射回球面，验证往返精度 |
| TEST(EqualArea, RemapEdges) | 测试等面积映射的边界环绕处理 `WrapEqualAreaSquare`：验证正方形边界外侧的点经环绕后映射到的球面方向与边界内侧对应点的方向一致（包括四条边和四个角） |
| TEST(DirectionCone, UnionBasics) | 测试方向锥体合并的基本情况：一个包含另一个、相同方向不同宽度、完全相反方向覆盖全球面、近似相反的宽锥体、窄锥体正交合并 |
| TEST(DirectionCone, UnionRandoms) | 使用 100 组随机锥体对测试合并操作：验证被原锥体包含的方向也被合并后的锥体包含 |
| TEST(DirectionCone, BoundBounds) | 测试 `BoundSubtendedDirections` 函数：(1) 包围盒内部的点返回全球面锥体；(2) 特定距离的外部点与解析值对比；(3) 1000 组随机配置通过射线-包围盒求交验证锥体的保守性和紧致性 |
| TEST(DirectionCone, VectorInCone) | 测试 `ClosestVectorInCone` 方法：锥体内的向量返回自身；锥体外的向量通过均匀采样锥体边界圆寻找最近向量作为参考进行验证 |
| TEST(SphericalTriangleArea, Basics) | 测试球面三角形面积计算：三个坐标轴构成的球面三角形面积为 Pi/2；半个象限的面积为 Pi/4；100 组随机旋转下两种三角形的面积保持不变 |
| TEST(SphericalTriangleArea, RandomSampling) | 通过蒙特卡洛采样（射线-三角形求交）独立估计球面三角形面积，与 `SphericalTriangleArea` 的计算结果交叉验证（误差 < 3.5%） |
| TEST(PointVector, Interval) | 编译期测试：验证使用 `Interval` 类型实例化的点和向量（`Point3fi`、`Vector3fi`）能正确执行加减法、标量乘法、点积、距离计算和叉积等运算 |
| TEST(OctahedralVector, EncodeDecode) | 测试八面体向量编码解码：65535 个均匀分布球面向量经 `OctahedralVector` 编码后解码，验证长度保持约为 1 且与原向量的点积偏差 < 0.001 |

## 依赖关系
- `pbrt/util/vecmath.h` — 向量、点、包围盒、法线、方向锥体、八面体向量等核心数学类型
- `pbrt/util/transform.h` — 变换和旋转
- `pbrt/util/rng.h` — 随机数生成器
- `pbrt/util/sampling.h` — 球面采样函数
- `pbrt/shapes.h` — IntersectTriangle 用于射线-三角形求交测试
