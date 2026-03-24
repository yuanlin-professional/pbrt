# file_test.cpp

## 概述
该测试文件针对 `pbrt/util/file.h` 中的文件操作工具函数进行测试。测试覆盖文件扩展名检查、扩展名移除、文件读写，以及浮点数文件的解析功能，包括正常解析和错误处理两种情况。

## 测试用例列表
| 测试用例 | 说明 |
|---|---|
| TEST(File, HasExtension) | 测试文件扩展名匹配功能，验证大小写不敏感匹配（如 "foo.Exr" 匹配 "exr"），以及不匹配情况（如 "foo.xr" 不匹配 "exr"） |
| TEST(File, RemoveExtension) | 测试扩展名移除功能，验证正常移除（"foo.exr" -> "foo"）、无扩展名的文件名保持不变（"fooexr" -> "fooexr"），以及多层扩展名只移除最后一个（"foo.exr.png" -> "foo.exr"） |
| TEST(File, ReadWriteFile) | 测试文件读写往返功能，写入字符串后再读取，验证内容一致，最后删除临时文件 |
| TEST(File, Success) | 测试浮点数文件的成功解析，验证包含注释行（以 # 开头）、空行、科学计数法（如 3e2、-4.75E-1）的浮点数文件能被正确解析为 6 个浮点数值 |
| TEST(File, Failures) | 测试浮点数文件的错误处理：(1) 验证读取不存在的文件返回空结果；(2) 验证包含非法字符（如 "l5l"）的文件解析失败并返回空结果 |

## 依赖关系
- `pbrt/util/file.h` - 被测试的文件操作模块（HasExtension、RemoveExtension、WriteFileContents、ReadFileContents、ReadFloatFile）
- `pbrt/pbrt.h` - PBRT 核心头文件
