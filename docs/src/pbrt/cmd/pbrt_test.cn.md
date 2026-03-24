# pbrt_test.cpp

## 概述
`pbrt_test.cpp` 是 PBRT-v4 的单元测试入口程序。它基于 Google Test 框架，负责初始化 PBRT 运行环境、解析测试专用的命令行参数，然后启动 Google Test 运行所有已注册的测试用例。该文件本身不包含具体的测试用例定义，而是作为测试可执行文件的 `main` 函数，通过链接阶段将各模块的测试代码汇集在一起执行。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| （由链接的测试文件提供） | 该文件本身不定义 `TEST()` 宏，具体的测试用例分布在 PBRT 各模块的 `*_test.cpp` 文件中，通过编译链接合并到 `pbrt_test` 可执行文件 |

## 使用方式
```
pbrt_test [options]
```

### 参数说明
| 参数 | 说明 |
|---|---|
| `--list-tests` | 列出所有已注册的测试用例 |
| `--log-level <level>` | 设置日志等级（verbose/error/fatal，默认：error） |
| `--nthreads <num>` | 指定渲染使用的线程数 |
| `--test-filter <regexp>` | 使用正则表达式过滤要运行的测试用例名称 |
| `--gtest-filter <regexp>` | 同 `--test-filter`，直接传递给 Google Test 的过滤器 |
| `--help` / `-h` | 显示帮助信息 |

### 执行流程
1. 解析命令行参数，设置 `PBRTOptions`（默认静默模式）
2. 调用 `InitPBRT()` 初始化 PBRT 系统
3. 将过滤器和列表参数转换为 Google Test 格式
4. 调用 `testing::InitGoogleTest()` 初始化 Google Test
5. 调用 `RUN_ALL_TESTS()` 运行所有测试
6. 调用 `CleanupPBRT()` 清理资源

## 依赖关系
- **依赖**：`pbrt/pbrt.h`、`pbrt/options.h`、`pbrt/util/args.h`、`pbrt/util/error.h`、`pbrt/util/print.h`、`gtest/gtest.h`（Google Test 框架）
- **被测试的源文件**：PBRT 项目中所有编译链接到 `pbrt_test` 目标的 `*_test.cpp` 文件
