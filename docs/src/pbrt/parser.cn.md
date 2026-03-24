# parser.h / parser.cpp

## 概述
该文件实现了 PBRT-v4 场景描述文件的词法分析器（Tokenizer）和语法解析器（Parser），是渲染管线的入口模块。它将 `.pbrt` 格式的文本场景描述解析为结构化的 API 调用，驱动场景构建过程。解析器支持文件包含（Include）、异步导入（Import）、内存映射文件读取，以及场景格式从 pbrt-v3 到 v4 的自动升级转换。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `ParserTarget` | 解析目标的抽象基类，定义了所有 PBRT 场景描述指令的虚函数接口（如 `Shape`、`Material`、`LightSource`、`Camera`、`Film` 等），解析器将解析结果回调到此接口 |
| `Token` | 词法单元结构体，包含 token 字符串视图和源码位置（`FileLoc`） |
| `Tokenizer` | 词法分析器，支持从文件（含 mmap 和 gzip）、字符串和标准输入读取，逐个产出 Token。处理引号字符串、注释、转义字符和 UTF 编码检测 |
| `FormattingParserTarget` | 格式化解析目标，继承自 `ParserTarget`，用于 `--format` 和 `--toply` 模式，将场景描述重新格式化输出。在 `--upgrade` 模式下自动将 pbrt-v3 材质/光源/采样器等转换为 v4 等价物 |
| `ParseFiles()` | 顶层解析函数，接受文件名列表，依次创建 Tokenizer 并调用内部 `parse()` 驱动解析 |
| `ParseString()` | 从字符串解析场景描述的便捷函数 |
| `parse()` | 核心解析循环（内部函数），使用递归下降方式解析 token 流，识别指令关键字并调用 `ParserTarget` 对应方法 |

## 架构图
```mermaid
classDiagram
    class ParserTarget {
        <<abstract>>
        +Scale(Float, Float, Float, FileLoc)*
        +Shape(string, ParsedParameterVector, FileLoc)*
        +Identity(FileLoc)*
        +Translate(Float, Float, Float, FileLoc)*
        +Rotate(Float, Float, Float, Float, FileLoc)*
        +LookAt(9xFloat, FileLoc)*
        +Transform(Float[16], FileLoc)*
        +Camera(string, ParsedParameterVector, FileLoc)*
        +Film(string, ParsedParameterVector, FileLoc)*
        +Integrator(string, ParsedParameterVector, FileLoc)*
        +Sampler(string, ParsedParameterVector, FileLoc)*
        +Material(string, ParsedParameterVector, FileLoc)*
        +LightSource(string, ParsedParameterVector, FileLoc)*
        +AreaLightSource(string, ParsedParameterVector, FileLoc)*
        +WorldBegin(FileLoc)*
        +AttributeBegin(FileLoc)*
        +AttributeEnd(FileLoc)*
        +EndOfFiles()*
        #ErrorExitDeferred(...)
    }
    class Tokenizer {
        -string contents
        -const char* pos, end
        -string sEscaped
        -FileLoc loc
        +CreateFromFile(string, callback) unique_ptr~Tokenizer~
        +CreateFromString(string, callback) unique_ptr~Tokenizer~
        +Next() optional~Token~
        -getChar() int
        -ungetChar()
        -CheckUTF(void*, int)
    }
    class Token {
        +string_view token
        +FileLoc loc
        +ToString() string
    }
    class FormattingParserTarget {
        -int catIndentCount
        -bool toPly, upgrade
        -map definedTextures
        -map definedNamedMaterials
        -map namedMaterialDictionaries
        +Material(string, ParsedParameterVector, FileLoc)
        +Shape(string, ParsedParameterVector, FileLoc)
        +LightSource(string, ParsedParameterVector, FileLoc)
        -upgradeMaterial(string*, ParameterDictionary*, FileLoc) string
        -upgradeMaterialIndex(string, ParameterDictionary*, FileLoc) string
    }

    ParserTarget <|-- FormattingParserTarget
    Tokenizer --> Token : 产出
    ParserTarget <.. Tokenizer : 驱动解析
```

## 算法流程图
```mermaid
flowchart TD
    A[ParseFiles 入口] --> B{文件列表为空?}
    B -->|是| C[从标准输入创建 Tokenizer]
    B -->|否| D[遍历文件列表, 创建 Tokenizer]
    C --> E[调用 parse 进入解析循环]
    D --> E

    E --> F[nextToken 获取下一个 token]
    F --> G{token 为空?}
    G -->|是| H[解析完成, 调用 EndOfFiles]
    G -->|否| I{匹配指令关键字}
    I -->|Shape/Material/Camera...| J[解析名称字符串]
    J --> K[parseParameters 解析参数列表]
    K --> L[调用 ParserTarget 对应方法]
    L --> F
    I -->|Transform/Translate/Scale...| M[解析数值参数]
    M --> N[调用 ParserTarget 变换方法]
    N --> F
    I -->|Include| O[解析文件名, 将新 Tokenizer 压入栈]
    O --> F
    I -->|Import| P[创建子 Builder, 异步解析导入文件]
    P --> F
    I -->|AttributeBegin/End| Q[调用属性作用域方法]
    Q --> F
    I -->|未知| R[报告语法错误]

    S[parseParameters 参数解析] --> T[循环获取 token]
    T --> U{是引号字符串?}
    U -->|否| V[回退 token, 返回参数列表]
    U -->|是| W[解析类型和名称]
    W --> X[解析值: 数组 bracket 或单值]
    X --> Y[创建 ParsedParameter 加入列表]
    Y --> T
```

## 依赖关系
- **依赖**：`pbrt/paramdict.h`、`pbrt/options.h`、`pbrt/scene.h`（cpp 中）、`pbrt/shapes.h`（cpp 中）、`pbrt/util/check.h`、`pbrt/util/containers.h`、`pbrt/util/error.h`、`pbrt/util/file.h`、`pbrt/util/memory.h`、`pbrt/util/mesh.h`、`pbrt/util/print.h`、`pbrt/util/string.h`、`pbrt/util/stats.h`、`pbrt/util/args.h`、`pbrt/util/progressreporter.h`
- **被依赖**：`scene.h`、`cmd/pbrt.cpp`、`cmd/pspec.cpp`、`parser_test.cpp`
