# pstd_test.cpp

## 概述
该测试文件验证 PBRT-v4 中 `pstd` 命名空间下的标准库替代实现，主要包括 `pstd::optional` 容器和 `pstd::pmr::monotonic_buffer_resource` 单调缓冲内存资源。测试涵盖基本值操作、析构函数调用正确性以及内存分配无重叠验证。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Optional, Basics) | 测试 `pstd::optional` 的基本操作：`has_value()`、布尔转换、`value()`、解引用运算符 `*`、`reset()`、`value_or()` 默认值、移动赋值等 |
| TEST(Optional, RunDestructors) | 使用 `AliveCounter` 引用计数类验证 `pstd::optional` 在赋值、重置、移动、作用域退出等各种场景下正确调用析构函数，确保无内存泄漏 |
| TEST(MonotonicBufferResource, NoOverlap) | 通过 `TrackingResource` 追踪分配器，对 `monotonic_buffer_resource` 执行 10000 次随机大小的内存分配，验证所有分配的内存区域不存在重叠 |

## 依赖关系
- `pbrt/util/pstd.h` — pstd::optional 和 pstd::pmr::monotonic_buffer_resource 的实现
- `pbrt/util/rng.h` — 随机数生成器（用于随机分配大小测试）
