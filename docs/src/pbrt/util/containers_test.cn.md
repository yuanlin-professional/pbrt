# containers_test.cpp

## 概述
该测试文件针对 `pbrt/util/containers.h` 中的容器数据结构进行测试。测试覆盖二维数组（Array2D）、哈希映射（HashMap）、类型包（TypePack）模板元编程工具，以及驻留缓存（InternCache）等数据结构的功能和正确性。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Array2D, Basics) | 测试二维数组的基本功能，验证创建 5x9 大小的数组后，尺寸属性正确，写入和读取的元素值一致 |
| TEST(Array2D, Bounds) | 测试使用 Bounds2i 边界创建二维数组的功能，验证支持负数索引的边界范围（如 [-4,3] 到 [10,7]），以及通过点坐标访问元素的正确性 |
| TEST(HashMap, Basics) | 测试哈希映射的基本操作：插入、查找、大小统计、键存在性检查、值更新（相同键插入新值）等 |
| TEST(HashMap, Randoms) | 使用随机数据进行大规模哈希映射测试：插入 10000 个随机键值对，验证所有键的正确查找，并通过迭代器遍历验证所有元素的完整性 |
| TEST(TypePack, Index) | 测试 TypePack 模板的 IndexOf 功能，验证能正确返回类型在类型包中的索引位置（int=0, float=1, double=2） |
| TEST(TypePack, HasType) | 测试 TypePack 模板的 HasType 功能，验证能正确判断类型是否存在于类型包中（int/float/double 存在，char/unsigned int 不存在） |
| TEST(TypePack, TakeRemove) | 测试 TypePack 模板的 TakeFirstN 和 RemoveFirstN 操作，验证从类型包中取出或移除前 N 个类型的正确性 |
| TEST(TypePack, Map) | 测试 TypePack 模板的 MapType 功能，验证对类型包中的每个类型应用模板变换（如 `Set<T>`）后的结果正确 |
| TEST(InternCache, BasicString) | 测试字符串驻留缓存的基本功能：验证相同字符串的查找返回相同指针（对象复用），不同字符串返回不同指针，以及缓存大小统计正确 |
| TEST(InternCache, BadHash) | 使用故意设计的差哈希函数（模 7）测试驻留缓存在大量哈希冲突下的正确性：插入 10000 个元素并验证两轮查找均返回相同指针和正确值 |

## 依赖关系
- `pbrt/util/containers.h` - 被测试的容器模块（Array2D、HashMap、InternCache）
- `pbrt/util/pstd.h` - TypePack 类型包模板元编程工具
- `pbrt/util/math.h` - 提供 PermutationElement 等数学工具函数
- `pbrt/util/rng.h` - 随机数生成器
