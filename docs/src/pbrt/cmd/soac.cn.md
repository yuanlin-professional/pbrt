# soac.cpp

## 概述
`soac`（Structure of Arrays Compiler）是 PBRT-v4 的一个代码生成工具，用于自动生成"结构体数组"（Structure of Arrays, SOA）的 C++ 代码。在 GPU 渲染中，SOA 数据布局相比传统的"数组结构体"（AOS）布局可以显著提升内存访问效率。该工具读取一种自定义的 `.soa` 声明文件，解析其中定义的类型和成员，然后生成对应的 `SOA<T>` 模板特化代码。

生成的代码包含完整的 SOA 容器实现，支持构造、赋值、通过下标运算符进行元素读取和写入等操作，并兼容 CPU 和 GPU（通过 `PBRT_CPU_GPU` 宏标记）。

## 主要功能
| 命令/子命令 | 说明 |
|---|---|
| 解析 `.soa` 文件 | 读取并解析自定义的 SOA 类型声明文件 |
| `flat` 类型声明 | 声明"扁平"类型（基本类型），这些类型以指针数组形式存储 |
| `soa` 类型声明 | 声明 SOA 类型，支持模板参数和嵌套 SOA 类型 |
| C++ 代码生成 | 生成完整的 `SOA<T>` 模板特化代码到标准输出 |

## 使用方式
```
soac <soac 文件名>
```

### 参数说明
| 参数 | 说明 |
|---|---|
| `<soac 文件名>` | 输入的 `.soa` 类型声明文件路径 |

### 输入文件格式
`.soa` 文件使用自定义的声明语法：

**`flat` 声明**：声明一个扁平（基本）类型
```
flat TypeName;
```

**`soa` 声明**：声明一个 SOA 结构体
```
soa TypeName {
    MemberType name1, name2;
    MemberType name3[ArraySize];
    const MemberType *ptr;
};
```

**模板 SOA 声明**：
```
soa TypeName<T> {
    T member;
};
```

**外部 SOA 引用**：
```
soa ExternalType;
```

### 生成的代码结构
对于每个 `soa` 类型声明，生成的代码包含：
- `SOA<T>` 模板特化结构体
- 默认构造函数和分配器构造函数（接受元素数量和 `Allocator`）
- 赋值运算符
- `GetSetIndirector` 内部类，实现通过下标进行读写操作
- `operator[]` 的常量和非常量版本
- 所有成员变量的声明（扁平类型使用指针，SOA 类型使用嵌套 `SOA<T>`）

## 依赖关系
- **依赖**：仅使用 C/C++ 标准库（`<assert.h>`、`<ctype.h>`、`<stdio.h>`、`<string.h>`、`<fstream>`、`<functional>`、`<map>`、`<set>`、`<string>`、`<utility>`、`<vector>`），不依赖 PBRT 内部模块
