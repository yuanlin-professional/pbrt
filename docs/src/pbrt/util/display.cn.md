# display.h / display.cpp

## 概述
该文件实现了 PBRT 渲染器与外部显示服务器之间的实时通信系统，允许在渲染过程中将图像数据以 IPC（进程间通信）方式发送到远程显示服务器进行实时预览。支持静态图像（一次性发送）和动态图像（定期更新）两种模式。在渲染管线中，该模块用于提供渲染进度的实时可视化反馈。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `ConnectToDisplayServer(host)` | 连接到指定地址的显示服务器，并启动后台更新线程 |
| `DisconnectFromDisplayServer()` | 断开与显示服务器的连接，等待更新线程结束 |
| `DisplayStatic(title, resolution, channels, getValues)` | 发送静态图像数据（回调函数版本） |
| `DisplayStatic(title, image, channelDesc)` | 发送静态 Image 对象 |
| `DisplayStatic(title, values, xResolution)` | 发送一维标量数据作为静态图像 |
| `DisplayDynamic(title, resolution, channels, getValues)` | 注册动态更新图像（回调函数版本） |
| `DisplayDynamic(title, image, channelDesc)` | 注册动态更新的 Image 对象 |
| `DisplayDynamic(title, values, xResolution)` | 注册一维标量数据的动态更新 |
| `IPCChannel` | 内部类，封装 TCP socket 连接，处理消息序列化和发送 |
| `DisplayItem` | 内部类，管理单个显示项的 tile 分块、哈希比较和增量更新 |
| `ImageChannelBuffer` | 内部类，管理单个通道的 tile 缓冲区和变更检测 |

## 架构图
```mermaid
graph TD
    subgraph 公共接口
        A[ConnectToDisplayServer]
        B[DisconnectFromDisplayServer]
        C[DisplayStatic]
        D[DisplayDynamic]
    end

    subgraph 内部实现
        E[IPCChannel]
        F[DisplayItem]
        G[ImageChannelBuffer]
        H[后台更新线程]
    end

    subgraph 网络通信
        I[TCP Socket]
        J[显示服务器]
    end

    A --> E
    B --> H
    C --> F
    D --> F
    F --> G
    F --> E
    E --> I
    I --> J
    H --> F
    H --> E

    D --> H
```

## 算法流程图
```mermaid
flowchart TD
    A[DisplayDynamic 调用] --> B[创建 DisplayItem]
    B --> C[添加到 dynamicItems 列表]

    D[后台线程循环] --> E[等待 250ms]
    E --> F[遍历 dynamicItems]
    F --> G[Display 方法]
    G --> H{已发送 OpenImage?}
    H -->|否| I[发送 CreateImage 指令]
    H -->|是| J[分块遍历 tiles]
    J --> K[调用 getValues 获取像素值]
    K --> L[计算 tile 哈希]
    L --> M{哈希与上次相同?}
    M -->|是| N[跳过发送]
    M -->|否| O[通过 IPCChannel 发送 tile 数据]
    O --> P[更新存储的哈希值]
```

## 依赖关系
- **依赖**：
  - `pbrt/pbrt.h` — 基础类型
  - `pbrt/util/check.h` — 断言宏
  - `pbrt/util/color.h` — RGB 颜色类型
  - `pbrt/util/containers.h` — 容器工具
  - `pbrt/util/image.h` — `Image`, `ImageChannelDesc`, `ImageChannelValues` 图像类
  - `pbrt/util/pstd.h` — `pstd::span`, `pstd::optional`
  - `pbrt/util/vecmath.h` — `Point2i`, `Bounds2i` 几何类型
  - `pbrt/util/error.h` — 错误报告
  - `pbrt/util/hash.h` — `HashBuffer` 哈希函数
  - `pbrt/util/print.h` — 格式化输出
  - `pbrt/util/string.h` — 字符串工具
  - 系统网络库（`<sys/socket.h>` / `<winsock2.h>`）
- **被依赖**：被渲染器主循环和错误处理模块 (`error.cpp`) 调用，用于实时显示和优雅退出
