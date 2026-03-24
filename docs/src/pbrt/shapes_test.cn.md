# shapes_test.cpp

## 概述
该测试文件验证 PBRT-v4 中几何形状（Shape）实现的正确性。测试重点在于防止自相交（reintersection）问题——从形状表面出发的光线不应再次与同一形状相交，以及三角形立体角计算、双线性面片偏移等关键功能。覆盖了球体、圆柱、三角形和双线性面片等形状类型。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(Triangle, Reintersect)` | 验证三角形的反自相交：从三角形表面交点出发的光线不应再次与同一三角形相交（1000 个随机三角形 x 1000 条出射光线） |
| `TEST(Triangle, SolidAngle)` | 验证三角形的立体角闭式计算（`SolidAngle()`）与基于 `Triangle::Sample()` 的蒙特卡洛估计一致 |
| `TEST(FullSphere, Reintersect)` | 验证完整球体的反自相交（包括单位变换和随机变换下的球体） |
| `TEST(ParialSphere, Normal)` | 验证部分球体交点法线与交点位置方向一致（归一化后点积为 1） |
| `TEST(PartialSphere, Reintersect)` | 验证部分球体（任意 zMin/zMax/phiMax 裁剪）的反自相交 |
| `TEST(Cylinder, Reintersect)` | 验证圆柱体的反自相交（包括单位变换和随机变换） |
| `TEST(Triangle, BadCases)` | 测试特定的三角形退化情况，确保不会产生错误的相交结果 |
| `TEST(BilinearPatch, Offset)` | 验证双线性面片的反自相交：从交点 `SpawnRay()` 出射的光线不应再次与面片相交 |

## 依赖关系
被测试的源文件：`pbrt/shapes.h`, `pbrt/interaction.h`, `pbrt/util/lowdiscrepancy.h`, `pbrt/util/sampling.h`, `pbrt/util/rng.h`, `pbrt/util/parallel.h`
