
JanusGraph支持两种不同的索引来提高查询速度：图索引和以顶点为中心的索引。 大多数图形查询从一些由其属性标识的顶点或边开始遍历。 图索引使这些全局检索操作在大图上有效。 以顶点为中心的索引可加快整个图的实际遍历速度，特别是在遍历具有许多有入射边的顶点时有效。

## 11.1. 图索引

图形索引是整个图形上的全局索引结构，允许通过其属性有效地检索顶点或边缘，以获得足够的选择条件。 例如，请看以下查询：
```
g.V().has('name', 'hercules')
g.E().has('reason', textContains('loves'))
```
第一个查询所有名字为hercules的顶点。 第二个查询所有reason属性中有love关键字的边。 如果没有图表索引执行这些查询，则需要对图中的所有顶点或边进行全面扫描，以找到与给定条件匹配的结果，这对于巨大的图形而言是非常低效且不可行的。

区分JanusGraph两种图索引类型：复合索引和混合索引。 复合索引非常快速且高效，但仅限于特定的，先前定义的属性键组合的等值查找。 混合索引可用于对索引键的任何组合进行查找，并且除了等值之外还支持多个条件谓词查询，具体取决于支持的后端索引存储类型。

两种类型的索引的创建都是通过JanusGraph管理系统和索引构建器JanusGraphManagement.buildIndex（String，Class）的返回结果，其中第一个参数定义索引的名称，第二个参数指定要索引的元素的类型（例如，Vertex.class）。图索引的名称必须是唯一的。针对新定义的属性键构建的图索引，即在与索引相同的管理事务中定义的属性键，可立即使用。这同样适用于受限于与索引在同一管理事务中创建的标签的图索引。针对已经使用但未限制为新创建的标签的属性键构建的图索引需要执行reindex过程以确保索引包含所有先前添加的元素。在reindex过程完成之前，索引将不可用。建议在与初始模式相同的事务中定义图索引。

> *注意*

> *在没有索引的情况下，JanusGraph将默认为完整图形扫描，以便检索所需的顶点列表。 虽然这会产生正确的结果集，但图形扫描效率非常低，导致生产环境中的整体系统性能较差。 *
> *在JanusGraph的生产部署中启用强制索引配置选项以禁止图扫描。*

### 11.1.1. 复合索引

复合索引通过一个或多个（固定）组合的多个键来检索顶点或边。 请考虑以下复合索引定义。
```
graph.tx().rollback() //Never create new indexes while a transaction is active
mgmt = graph.openManagement()
name = mgmt.getPropertyKey('name')
age = mgmt.getPropertyKey('age')
mgmt.buildIndex('byNameComposite', Vertex.class).addKey(name).buildCompositeIndex()
mgmt.buildIndex('byNameAndAgeComposite', Vertex.class).addKey(name).addKey(age).buildCompositeIndex()
mgmt.commit()
//Wait for the index to become available
ManagementSystem.awaitGraphIndexStatus(graph, 'byNameComposite').call()
ManagementSystem.awaitGraphIndexStatus(graph, 'byNameAndAgeComposite').call()
//Reindex the existing data
mgmt = graph.openManagement()
mgmt.updateIndex(mgmt.getGraphIndex("byNameComposite"), SchemaAction.REINDEX).get()
mgmt.updateIndex(mgmt.getGraphIndex("byNameAndAgeComposite"), SchemaAction.REINDEX).get()
mgmt.commit()
```

首先，已经定义了两个属性键名称和年龄。 接下来，构建一个仅对name属性键的简单复合索引。 JanusGraph将使用此索引来回答以下查询。

```
g.V().has('name', 'hercules')
```
其次，复合图索引包括这两个键（name和age）。 JanusGraph将使用此索引来响应以下查询。
```
g.V().has('age', 30).has('name', 'hercules')
```
请注意，必须在查询的相等条件中找到复合图索引的所有键，才能使用此索引。 例如，以下查询无法使用任一索引来回答，因为它仅包含对age的约束，而不包含name。
```
g.V().has('age', 30)
```
再请注意，复合图索引只能用于上述查询中的相等约束。 仅使用name键上定义的简单复合索引来回答以下查询，因为age约束不是等式约束
```
g.V().has('name', 'hercules').has('age', inside(20, 50))
```
 
复合索引不需要配置外部索引后端，并且通过主存储后端支持。 因此，复合索引修改通过与图修改相同的事务持久化，这意味着如果底层存储后端支持原子性and/or一致性，那些更改是原子的and/or一致性的。

> *注意*
>
> *复合索引可以仅包括一个或多个键。 仅具有一个键的复合索引有时被称为键索引。*

##### 11.1.1.1. 索引唯一性

Composite indexes can also be used to enforce property uniqueness in the graph. If a composite graph index is defined as unique() there can be at most one vertex or edge for any given concatenation of property values associated with the keys of that index. For instance, to enforce that names are unique across the entire graph the following composite graph index would be defined.

```
graph.tx().rollback()  //Never create new indexes while a transaction is active
mgmt = graph.openManagement()
name = mgmt.getPropertyKey('name')
mgmt.buildIndex('byNameUnique', Vertex.class).addKey(name).unique().buildCompositeIndex()
mgmt.commit()
//Wait for the index to become available
ManagementSystem.awaitGraphIndexStatus(graph, 'byNameUnique').call()
//Reindex the existing data
mgmt = graph.openManagement()
mgmt.updateIndex(mgmt.getGraphIndex("byNameUnique"), SchemaAction.REINDEX).get()
mgmt.commit()
```