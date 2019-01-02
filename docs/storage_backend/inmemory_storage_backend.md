# **第22章InMemory存储后端**

JanusGraph附带内存存储后端，可通过以下配置使用：

```
storage.backend = inmemory
```

或者，可以直接在Gremlin控制台中打开内存中的JanusGraph图：

```
graph = JanusGraphFactory.build().set('storage.backend', 'inmemory').open()
```

内存存储后端没有其他配置选项。顾名思义，这个后端将所有数据保存在内存中。关闭图形或终止承载JanusGraph图形的过程将不可撤销地删除图形中的所有数据。此后端是特定JanusGraph图形实例的本地后端，不能跨多个JanusGraph图表共享。

## 22.1。理想的用例

内存存储后端主要是为了简化测试（针对那些不需要持久性的测试）和图形探索而开发的。内存存储后端不适用于生产用途，大型图形或高性能用例。内存存储后端不是性能或内存优化。所有数据都存储在分配给Java虚拟机的堆空间中。

 