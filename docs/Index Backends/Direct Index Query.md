26、 直接索引查询
---
JanusGraph的标准全局图查询机制支持顶点或边的布尔查询。换句话说，元素要么与查询匹配，要么不匹配。没有部分匹配或结果评分。  

一些索引后端还支持模糊搜索查询。对于那些查询，计算每个匹配的分数以指示匹配的“良好性”，并且以其分数的顺序返回结果。模糊搜索在处理全文搜索查询时特别有用，其中匹配更多单词被认为更好。


由于模糊搜索实现和评分算法在索引后端之间存在显着差异，因此JanusGraph本身不支持模糊搜索。但是，JanusGraph提供了一种直接索引查询机制，允许将搜索查询直接发送到索引后端进行评估（对于那些支持它的后端）。

使用Graph.indexQuery()撰写是直接针对某个索引后端执行查询。此查询构建器需要两个参数：
1. 要查询的索引后端的名称。这必须是在JanusGraph配置中配置的名称，并在属性键索引定义中使用
2. 查询字符串

构建器允许通过其limit(int)方法配置要返回的最大元素数。构建器offset(int)控制要跳过的结果集中的初始匹配数。要检索与指定索引后端中的给定查询匹配的所有顶点或边，请分别调用vertices()或edges()。无法同时查询顶点和边。这些方法返回一个Iterableover Result对象。结果对象包含匹配的句柄，可检索的getElement()和相关的分数 - getScore()。  

请考虑以下示例：
```
ManagementSystem mgmt = graph.openManagement();
PropertyKey text = mgmt.makePropertyKey("text").dataType(String.class).make();
mgmt.buildIndex("vertexByText", Vertex.class).addKey(text).buildMixedIndex("search");
mgmt.commit();
// ... Load vertices ...
for (Result<Vertex> result : graph.indexQuery("vertexByText", "v.text:(farm uncle berry)").vertices()) {
   System.out.println(result.getElement() + ": " + result.getScore());
}
```
### 26.1 请求参数
查询字符串直接传递给索引后端进行处理，因此查询字符串语法取决于索引后端支持的内容。对于顶点查询，JanusGraph将分析以“v.”开头的属性键引用的查询字符串。并通过句柄替换那些与属性键对应的索引字段。同样，对于边缘查询，JanusGraph将替换以“e.”开头的属性键引用。因此，要引用顶点的属性，请在查询字符串中使用“v.[KEY_NAME]”。同样，对于边缘写“e.[KEY_NAME]”。

Elasticsearch和Lucene支持Lucene查询语法。有关更多信息，请参阅Lucene文档或Elasticsearch文档。上面示例中使用的查询遵循Lucene查询语法。
```
graph.indexQuery("vertexByText", "v.text:(farm uncle berry)").vertices()
```
此查询匹配所有顶点，其中文本包含三个单词中的任何一个（按括号分组），分数匹配越高，文本中匹配的单词越多。  

此外，Elasticsearch支持通配符查询，在查询字符串中使用“v.*”或“e.*”来查询元素上的任何属性是否匹配。

### 26.2 查询统计
有时需要知道从查询返回的总结果数量，而无需检索所有结果。幸运的是，Elasticsearch和Solr提供了一种不涉及检索和排名所有文档的快捷方式。此快捷方式通过".vertexTotals()", ".edgeTotals()", and ".propertyTotals()"方法对外提供。

可以使用与indexQuery构建器相同的查询语法检索总计，但是大小将被覆盖为0。

```
graph.indexQuery("vertexByText", "v.text:(farm uncle berry)").vertexTotals()
```
### 26.3 陷阱
#### 26.3.1 属性建名称

必须将包含非字母数字字符的属性键的名称放在引号中，以确保正确分析查询。
```
graph.indexQuery("vertexByText", "v.\"first_name\":john").vertices()
```
一些属性键名称可以通过JanusGraph索引后端实现进行转换。索引后端字段名称不支持空格，可能会将 "My Field Name"替换成"My•Field•Name",或者像Solr这样的索引后端可以将类型信息附加到名称，将 "myBooleanField"替换成"myBooleanField_b".这些转换发生在索引后端的IndexProvider实现中，在“mapKey2Field”方法中。索引后端可以保留特殊字符（例如•）并禁止对包含它们的字段建立索引。因此，建议避免使用属性名称中的空格和特殊字符。

通常，进行直接索引查询取决于通常对用户隐藏的JanusGraph索引后端的实现细节，因此最好根据使用中的索引后端验证查询。

#### 26.3.2 元素标识符碰撞
字符串“v.”，“e.”和“p.” 用于分别在查询中标识顶点，边或属性元素。如果字段名称或查询值包含相同的字符序列，则可能导致查询字符串中的冲突并解析错误，如以下示例所示：
```
graph.indexQuery("vertexByText", "v.name:v.john").vertices() //DOES NOT WORK!
```
要避免此类标识符冲突，请使用setElementIdentifier方法定义在查询的任何其他部分中不出现的唯一元素标识符字符串：
```
graph.indexQuery("vertexByText","$v$name:v.john").setElementIdentifier("$v$").vertices()
```
#### 26.3.3 混合索引可用性延迟
插入数据后，如果查询立即遍历混合索引，则更改可能不可见。在Elasticsearch中，确定此延迟的配置选项是索引刷新间隔。在Solr中，主要配置选项是最长时间。

