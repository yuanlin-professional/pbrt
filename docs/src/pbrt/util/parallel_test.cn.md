# parallel_test.cpp

## 概述
该测试文件针对 `pbrt/util/parallel.h` 中的并行计算工具进行测试。测试覆盖一维并行循环（ParallelFor）、二维并行循环（ParallelFor2D）、空范围处理、线程遍历（ForEachThread）以及线程局部存储（ThreadLocal）的正确性和一致性。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Parallel, Basics) | 测试并行循环的基本功能：(1) 使用 `ParallelFor` 对 [0, 1000) 范围执行单元素回调，验证计数器达到 1000；(2) 使用 `ParallelFor` 的范围回调版本（start, end），验证范围有效性和总计数正确；(3) 使用 `ParallelFor2D` 对 15x14 的二维区域执行并行迭代，验证总计数为 210 |
| TEST(Parallel, DoNothing) | 测试空范围的并行循环：(1) 验证 `ParallelFor(0, 0, ...)` 不执行任何回调；(2) 验证 `ParallelFor2D` 对空边界不执行任何回调 |
| TEST(Parallel, ForEachThread) | 测试 `ForEachThread` 函数，验证回调在每个运行线程上恰好执行一次，初始计数器等于运行线程数，每次回调减 1 后最终为 0 |
| TEST(ThreadLocal, Consistency) | 测试线程局部存储的一致性，创建存储线程 ID 的 ThreadLocal 对象：(1) 在 ParallelFor 中验证每次访问获取的线程 ID 与当前线程一致；(2) 在 RunAsync 异步任务中验证同样的一致性；(3) 再次在 ParallelFor 中验证，确保线程局部值在多轮使用后仍然正确 |

## 依赖关系
- `pbrt/util/parallel.h` - 被测试的并行计算模块（ParallelFor、ParallelFor2D、ForEachThread、RunAsync、ThreadLocal 等）
- `pbrt/pbrt.h` - PBRT 核心头文件
