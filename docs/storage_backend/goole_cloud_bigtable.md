**第20章Google Cloud Bigtable**

​                                                             ![img](https://docs.janusgraph.org/latest/images/Cloud-Bigtable.svg)  

Cloud Bigtable是Google的NoSQL大数据数据库服务。它与许多核心Google服务相同，包括搜索，分析，地图和Gmail。

Bigtable旨在以一致的低延迟和高吞吐量处理大量工作负载，因此它是运营和分析应用程序（包括物联网，用户分析和财务数据分析）的理想选择。

## 20.1。Bigtable设置

Bigtable为所有数据访问操作实现HBase接口，并且需要一些配置选项才能连接。

### 20.1.1。连接到Bigtable

配置JanusGraph以连接Bigtable是通过使用`hbase`后端，自定义连接实现，包含Bigtable实例的Google Cloud Platform项目的项目ID以及您要连接的Cloud Bigtable实例ID来实现的。

例：



 