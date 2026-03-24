# buffercache.h / buffercache.cpp

## 概述
该文件实现了一个通用的缓冲区缓存（Buffer Cache）系统，用于在渲染场景加载过程中去重和共享几何数据缓冲区。当多个网格使用相同的顶点、法线或索引数据时，缓存能够避免重复存储，显著降低内存占用。在渲染管线中，该模块是场景加载和几何数据管理的核心基础设施。

## 主要类与接口
| 类/结构体/函数 | 说明 |
|---|---|
| `BufferCache<T>` | 模板类，基于哈希的缓冲区去重缓存，使用分片锁实现线程安全 |
| `BufferCache::LookupOrAdd` | 核心方法：查找缓存中是否存在相同内容的缓冲区，若存在则返回已缓存指针，否则新建并缓存 |
| `BufferCache::BytesUsed` | 返回缓存已使用的字节数 |
| `BufferCache::Buffer` | 内部结构体，封装数据指针、大小和哈希值 |
| `BufferCache::BufferHasher` | 内部哈希函数对象，用于 unordered_set |
| `intBufferCache` | 全局整数缓冲区缓存（用于索引数据） |
| `point2BufferCache` | 全局 Point2f 缓冲区缓存（用于 UV 坐标） |
| `point3BufferCache` | 全局 Point3f 缓冲区缓存（用于顶点位置） |
| `vector3BufferCache` | 全局 Vector3f 缓冲区缓存（用于切线等向量数据） |
| `normal3BufferCache` | 全局 Normal3f 缓冲区缓存（用于法线数据） |
| `InitBufferCaches` | 初始化所有全局缓冲区缓存实例 |

## 架构图
```mermaid
classDiagram
    class BufferCache~T~ {
        +LookupOrAdd(span~const T~ buf, Allocator alloc) const T*
        +BytesUsed() size_t
        -Buffer lookupBuffer
        -shared_mutex mutex[64]
        -unordered_set cache[64]
        -atomic~size_t~ bytesUsed
    }

    class Buffer {
        +const T* ptr
        +size_t size
        +size_t hash
        +operator==(Buffer) bool
    }

    class BufferHasher {
        +operator()(Buffer) size_t
    }

    BufferCache~T~ *-- Buffer
    BufferCache~T~ *-- BufferHasher

    BufferCache~int~ <|-- intBufferCache : 全局实例
    BufferCache~Point2f~ <|-- point2BufferCache : 全局实例
    BufferCache~Point3f~ <|-- point3BufferCache : 全局实例
    BufferCache~Vector3f~ <|-- vector3BufferCache : 全局实例
    BufferCache~Normal3f~ <|-- normal3BufferCache : 全局实例
```

## 算法流程图
```mermaid
flowchart TD
    A[调用 LookupOrAdd] --> B[计算缓冲区哈希]
    B --> C[确定分片索引]
    C --> D[获取共享读锁]
    D --> E{缓存中已存在?}
    E -->|是| F[释放锁, 返回已缓存指针]
    E -->|否| G[释放共享锁]
    G --> H[分配新内存并拷贝数据]
    H --> I[获取独占写锁]
    I --> J{再次检查是否已被其他线程插入?}
    J -->|是| K[释放新分配内存, 返回已缓存指针]
    J -->|否| L[插入缓存, 返回新指针]

    F --> M[统计: 缓存命中+1, 冗余字节+]
    K --> M
```

## 依赖关系
- **依赖**：
  - `pbrt/pbrt.h` — 基础类型定义
  - `pbrt/util/check.h` — CHECK/DCHECK 断言宏
  - `pbrt/util/hash.h` — `HashBuffer` 哈希函数
  - `pbrt/util/print.h` — 格式化输出
  - `pbrt/util/pstd.h` — `pstd::span` 容器
  - `pbrt/util/stats.h` — 性能统计计数器
  - `pbrt/util/vecmath.h` — `Point2f`、`Point3f`、`Vector3f`、`Normal3f` 类型
- **被依赖**：被几何形状加载模块（三角形网格等）使用，用于顶点、索引和法线缓冲区的去重
