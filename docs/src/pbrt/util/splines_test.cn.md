# splines_test.cpp

## 概述
该测试文件验证 PBRT-v4 中三次贝塞尔样条相关功能的正确性，主要测试贝塞尔曲线的包围盒计算是否能正确包含曲线上所有点。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Spline, BezierBounds) | 测试三次贝塞尔曲线包围盒 `BoundCubicBezier` 的正确性：(1) 使用默认控制点（全零）验证曲线上所有采样点都在包围盒内；(2) 使用 1000 组随机控制点（范围 [-5, 5]），对每条曲线均匀采样 1024 个点，验证所有点均在计算出的包围盒内（包围盒略微扩展以容纳浮点误差） |

## 依赖关系
- `pbrt/util/splines.h` — 贝塞尔曲线求值和包围盒计算
- `pbrt/util/vecmath.h` — 三维点和包围盒类型
- `pbrt/util/pstd.h` — pstd::MakeConstSpan 等工具
- `pbrt/util/rng.h` — 随机数生成器
