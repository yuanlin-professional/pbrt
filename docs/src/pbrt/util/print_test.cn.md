# print_test.cpp

## 概述
该测试文件验证 PBRT-v4 中格式化打印功能的正确性，主要测试 `StringPrintf` 函数和 `operator<<` 流输出运算符。涵盖基本字符串格式化、类型感知的 `%s` 格式化、`%d` 整数格式化、浮点精度保持以及各种几何类型的流输出格式。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(StringPrintf, Basics) | 测试 `StringPrintf` 的基本功能：纯字符串、整数格式化 `%d`、浮点格式化 `%f`，以及参数不足和参数过多时的死亡测试（仅非 Windows 平台） |
| TEST(StringPrintf, FancyPctS) | 测试 `%s` 对多种类型的格式化支持：布尔值（true/false）、`Vector3f`、`std::string`、`std::vector<int>`、`std::array<std::string>`、`pstd::span`、`pstd::optional`（有值和无值两种情况） |
| TEST(StringPrintf, optional) | 测试 `StringPrintf` 对 `pstd::optional<float>` 类型使用 `%s` 格式化的基本兼容性 |
| TEST(StringPrintf, FancyPctD) | 测试 `%d` 对多种整数类型的格式化：`unsigned int`、超过 4GB 的 `size_t`、`uint64_t` 最大值、`int64_t` 最大值 |
| TEST(StringPrintf, Precision) | 测试浮点数格式化的精度保持：将 `float` 和 `double` 格式化后再解析回数值，验证往返精度无损 |
| TEST(OperatorLeftShiftPrint, Basics) | 测试各种几何类型的 `operator<<` 流输出格式：`Point2f`、`Point2i`、`Point3f`、`Point3i`、`Vector2f`、`Vector2i`、`Vector3f`、`Vector3i`、`Normal3f`、`Quaternion`、`Ray`、`Bounds2f`、`Bounds3f`、`SquareMatrix<4>`、`Transform` |

## 依赖关系
- `pbrt/util/print.h` — 格式化打印核心实现
- `pbrt/util/vecmath.h` — 向量、点、法线等几何类型
- `pbrt/util/transform.h` — 变换矩阵类型
- `pbrt/util/math.h` — 数学常量（如 Pi）
- `pbrt/util/pstd.h` — pstd::optional、pstd::span 等标准库替代
- `double-conversion` — 浮点数字符串转换库
