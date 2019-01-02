30 schema的高级特性
---
### 30.1 Static Vertices
顶点标签可以定义为静态，这意味着具有该标签的顶点不能在创建它们的事务之外被修改。  
```
mgmt = graph.openManagement()
tweet = mgmt.makeVertexLabel('tweet').setStatic().make()
mgmt.commit()
```
静态顶点标签是控制数据生命周期的一种方法，在将数据加载到图形中时很有用，图形创建后不应修改这些数据。

### 30.2  Edge and Vertex TTL
边可顶点标签可以设置生命周期。标签在初始创建通过后，当TTL过期后带有此类标签的边和顶点会被自动移除。当仅仅是临时加载大量数据到图时TTL是非常有用的。定义TTL消除了手动清理的需要，并且非常有效地处理了删除的需求。例如，对于TTL事件的边（例如用户页面访问）在一段时间之后进行汇总或者仅仅不再需要进行分析或操作查询处理时，这将是有意义的。
以下存储后端支持边和顶点TTL：
- Cassandra
- HBase

#### 30.2.1  Edge TTL
边的生命周期基于每个边标签定义，这意味着该标签的所有边具有相同的生命周期。需要注意的是，后端必须支持单元级别生命周期。当前只有Cassandra和HBase支持：
```
mgmt = graph.openManagement()
visits = mgmt.makeEdgeLabel('visits').make()
mgmt.setTTL(visits, Duration.ofDays(7))
mgmt.commit()
```
注意，修改边会重置边的生命周期。另外需要注意的是，边标签的生命周期可以改变，但更改需要一段时间传播到所有正在运行的JanusGraph实例，这就意味着，同一个标签可以临时使用两个不同的生命周期。

#### 30.2.2. Property TTL
属性的生命周期与边生命周期非常相似，并且基于每个属性键定义，这意味着该键的所有属性具有相同的生存时间。请注意，后端必须支持单元级别生命周期。目前只有Cassandra和HBase支持这一点:
```
mgmt = graph.openManagement()
sensor = mgmt.makePropertyKey('sensor').cardinality(Cardinality.LIST).dataType(Double.class).make()
mgmt.setTTL(sensor, Duration.ofDays(21))
mgmt.commit()
```
与边的生命周期一样，修改现有属性会重置该属性的生命周期，并且修改属性键的可能不会立即生效。

#### 30.2.3. Vertex TTL
顶点的生命周期是基于每个顶点标签定义的，也就是说该标签的所有顶点具有相同的生存时间。配置的生命周期应用于其顶点，属性和所有的入射边，以确保从图中删除整个顶点。因此，必须先将顶点标签定义为静态，然后才能设置生命周期以排除任何可能是顶点生命周期失效的修改。顶点生命周期仅适用于静态顶点标签。要注意的是后端必须支持存储级别的生命周期。目前只有Cassandra和HBase支持。
```
mgmt = graph.openManagement()
tweet = mgmt.makeVertexLabel('tweet').setStatic().make()
mgmt.setTTL(tweet, Duration.ofHours(36))
mgmt.commit()
```
请注意，可以修改顶点标签的生命周期，但此更改可能需要一些时间才能传播到所有正在运行的JanusGraph实例，这意味着两个不同的生命周期可能作用于同一标签。
#### 30.3. Multi-Properties
正如第5章“架构和数据建模”中所讨论的，JanusGraph支持具有SET和LIST基数的属性键。因此，JanusGraph在单个顶点上支持具有相同键的多个属性。此外，JanusGraph对属性的处理类似于对边的处理，在属性上允许使用单值属性注释，如下面的示例所示：
```
mgmt = graph.openManagement()
mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.LIST).make()
mgmt.commit()
v = graph.addVertex()
p1 = v.property('name', 'Dan LaRocque')
p1.property('source', 'web')
p2 = v.property('name', 'dalaro')
p2.property('source', 'github')
graph.tx().commit()
v.properties('name')
==> Iterable over all name properties
```
这些特性在许多应用程序中都很有用，例如需要向属性附加出处信息(例如谁添加了属性、何时以及从何处添加?)。如第31章Eventually-Consistent Storage Backends（最终存储一致性）所述，支持跟高的基数属性和属性上的注释在高并发性，水平扩展的设计模式中也很有用。  

属性支持以顶点为中心的索引和全局图索引，其方式与边支持的方式相同。有关定义边的索引信息，请参考11章“ Indexing for Better Performance”，并使用相应的API方法为属性定义相同的索引。

#### 30.4. Unidirected Edges
单向边是只能沿输出方向遍历的边。单向边具有较低的存储占用，但他们支持的遍历类型有限。单向边在概念上类似于万维网的超链接，既输出顶点可以通过边遍历，但输入顶点不知道它的存在。
```
mgmt = graph.openManagement()
mgmt.makeEdgeLabel('author').unidirected().make()
mgmt.commit()
```
注意，当它们输入的顶点被删除时，单向边不会自动删除。用户必须通过明确的检查事物中的顶点的存在，以确保这些不一致不会在查询时出现或解决这些问题。请参见 31.2.2节 “Ghost Vertices”获取更多信息。