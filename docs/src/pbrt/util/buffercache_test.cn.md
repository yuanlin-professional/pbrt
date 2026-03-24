# buffercache_test.cpp

## 概述
该测试文件针对 `pbrt/util/buffercache.h` 中的缓冲区缓存（BufferCache）功能进行测试。BufferCache 用于对相同内容的缓冲区进行去重，避免重复存储相同数据，从而节省内存。测试验证了缓存的查找、添加、去重以及内存统计功能。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(BufferCache, Basics) | 测试缓冲区缓存的基本功能：(1) 验证全局 `intBufferCache` 非空；(2) 向缓存添加一个包含 5 个整数的向量，验证返回的指针内容正确且内存使用量增加了 `5 * sizeof(int)`；(3) 再次查找相同的向量，验证返回相同的指针（去重），且内存使用量不变；(4) 添加不同内容的向量，验证返回不同指针且内存使用量相应增加到 `9 * sizeof(int)` |

## 依赖关系
- `pbrt/util/buffercache.h` - 被测试的缓冲区缓存模块
- `pbrt/pbrt.h` - PBRT 核心头文件
