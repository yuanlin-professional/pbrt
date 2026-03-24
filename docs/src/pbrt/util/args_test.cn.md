# args_test.cpp

## 概述
该测试文件针对 `pbrt/util/args.h` 中的命令行参数解析功能进行测试。主要验证 `ParseArg` 函数在各种场景下的正确性，包括简单参数解析、多参数解析、布尔类型参数、错误处理以及参数名称标准化等功能。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(Args, Simple) | 测试基本的参数解析功能，验证 `--nthreads 4`（空格分隔）和 `--nthreads=4`（等号分隔）两种格式均能正确解析整数参数值 |
| TEST(Args, Multiple) | 测试多个参数的连续解析，验证布尔参数和整数参数混合使用时的正确性，以及非参数项（如 "yolo"）不会被错误匹配 |
| TEST(Args, Bool) | 测试布尔类型参数的解析，验证 `--log=false`、`--benchmark`（无值默认为 true）、`--debug=true` 三种布尔参数格式的正确解析 |
| TEST(Args, ErrorMissingValue) | 测试缺少参数值时的错误处理，验证当 `--nthreads` 后没有提供值时，解析失败并触发错误回调，且原始值不被修改 |
| TEST(Args, ErrorMissingValueEqual) | 测试等号后缺少值时的错误处理，验证 `--nthreads=` 格式中等号后为空时，解析失败并触发错误回调 |
| TEST(Args, ErrorBogusBool) | 测试无效布尔值的错误处理，验证 `--log=tru3` 这样的无效布尔值会导致解析失败并触发错误回调 |
| TEST(Args, Normalization) | 测试参数名称的标准化，验证 `--n_threads`（下划线分隔）和 `--nThreads`（驼峰命名）均能匹配到 "nthreads" 参数 |

## 依赖关系
- `pbrt/util/args.h` - 被测试的命令行参数解析模块
- `pbrt/util/string.h` - 提供 `SplitStringsFromWhitespace` 字符串分割功能
- `pbrt/pbrt.h` - PBRT 核心头文件
