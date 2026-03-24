# string_test.cpp

## 概述
该测试文件验证 PBRT-v4 中 Unicode 字符串处理功能的正确性，主要测试 UTF-8 编码的 Unicode 规范化（NFC/NFD 标准化形式转换）。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Unicode, BasicNormalization) | 测试 Unicode 规范化功能：使用法语名字 "Amelie" 的两种 Unicode 表示（NFC 组合形式 `\u00e9` 和 NFD 分解形式 `\u0065\u0301`），验证 (1) NFC 和 NFD 的 UTF-16 编码不相等；(2) 转换为 UTF-8 后仍不相等；(3) `NormalizeUTF8` 对已是 NFC 的字符串无变化；(4) `NormalizeUTF8` 能将 NFD 编码的字符串正确转换为 NFC 形式，使两者相等 |

## 依赖关系
- `pbrt/util/string.h` — Unicode 字符串处理函数（UTF8FromUTF16、NormalizeUTF8 等）
