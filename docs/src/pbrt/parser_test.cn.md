# parser_test.cpp

## 概述
该测试文件验证 PBRT-v4 场景文件解析器中词法分析器（`Tokenizer`）的正确性。测试覆盖了基本的词法分析功能、错误处理（未终止字符串、提前 EOF），以及从文件读取并词法分析的能力。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| `TEST(Parser, TokenizerBasics)` | 验证基本的词法分析：关键字、引号字符串、方括号、数值的正确分割；支持换行分隔和注释处理 |
| `TEST(Parser, TokenizerErrors)` | 验证词法分析器的错误处理：未终止字符串时报告 "premature EOF"、字符串中包含换行时报告 "unterminated string" |
| `TEST(Parser, TokenizeFile)` | 验证从磁盘文件读取并进行词法分析的功能，包含关键字、引号字符串、注释、浮点数和科学记数法 |

## 依赖关系
被测试的源文件：`pbrt/parser.h`, `pbrt/util/pstd.h`
