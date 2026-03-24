# taggedptr_test.cpp

## 概述
该测试文件验证 PBRT-v4 中 `TaggedPointer` 类型擦除指针的正确性。`TaggedPointer` 是一种通过在指针高位存储类型标签来实现轻量级多态的机制，支持最多 16 种具体类型。测试覆盖基本类型识别、标签值、分发调用等功能。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(TaggedPointer, Basics) | 全面测试 `TaggedPointer` 的基本功能：(1) 空指针的 `ptr()` 返回 nullptr；(2) `MaxTag()` 和 `NumTags()` 返回正确值（16 和 17）；(3) 16 种类型的 `Is<T>()` 类型判断正确性（包括正例和反例各 16 组）；(4) 空指针和各类型指针的 `Tag()` 值正确（0-16）；(5) `TypeIndex<T>()` 静态方法返回正确的类型索引（1-16）；(6) `CastOrNullptr<T>()` 在类型匹配时返回原指针、不匹配时返回 nullptr |
| TEST(TaggedPointer, Dispatch) | 测试 `TaggedPointer` 的动态分发功能：为 16 种 `IntType<N>` 类型创建 Handle，验证通过 Handle 调用 `func()`（可变方法）和 `cfunc()`（const 方法）能正确分发到对应具体类型并返回正确值 |

## 依赖关系
- `pbrt/util/taggedptr.h` — TaggedPointer 模板类的核心实现
