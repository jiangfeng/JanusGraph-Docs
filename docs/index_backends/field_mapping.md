第25章 字段映射
---
### 25.1 单个字段映射

默认情况下，JanusGraph将对属性键进行编码，以便为混合索引中的属性键生成唯一的字段名称。如果想要在外部索引后端直接查询混合索引，则后端很难处理并且难以辨认。对于这种情况，可以通过参数显式指定字段名称。
```
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('bookname').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(name, Parameter.of('mapped-name', 'bookname')).buildMixedIndex("search")
mgmt.commit()
```
通过将此字段映射定义为参数，JanusGraph将使用与booksBySummary外部索引系统中创建的索引中的字段相同的名称作为属性键。请注意，必须确保给定的字段名称在索引中是唯一的。

### 25.2 全局字段映射

可以指示JanusGraph始终将外部索引中的字段名称设置为与属性键名称相同，而不是单独调整添加到混合索引的每个键的字段映射。这是通过启用map-name为每个索引后端配置的配置选项来完成的。如果为特定索引后端启用此选项，则针对所述后端定义的所有混合索引将使用与属性键名称相同的字段名称。

但是，这种方法有两个限制：1）用户必须确保属性键名称是索引后端的有效字段名称，2）重命名属性键不会重命名索引中的字段名称，这可能导致命名冲突，用户必须知道并避免。

### 25.2.1 定制分析器
默认情况下，JanusGraph将使用索引后端的默认分析器来获取Mapping.TEXT的属性，而不使用Mapping.STRING属性的分析器。如果想要使用另一个分析器，可以通过参数显式指定：Mapping.TEXT的ParameterType.TEXT_ANALYZER和Mapping.STRING的ParameterType.STRING_ANALYZER。

#### 25.2.1.1 对于Elasticsearch
必须将分析仪的名称设置为参数值。
```
mgmt = graph.openManagement()
string = mgmt.makePropertyKey('string').dataType(String.class).make()
text = mgmt.makePropertyKey('text').dataType(String.class).make()
textString = mgmt.makePropertyKey('textString').dataType(String.class).make()
mgmt.buildIndex('string', Vertex.class).addKey(string, Mapping.STRING.asParameter(), Parameter.of(ParameterType.STRING_ANALYZER.getName(), 'standard')).buildMixedIndex("search")
mgmt.buildIndex('text', Vertex.class).addKey(text, Mapping.TEXT.asParameter(), Parameter.of(ParameterType.TEXT_ANALYZER.getName(), 'english')).buildMixedIndex("search")
mgmt.buildIndex('textString', Vertex.class).addKey(text, Mapping.TEXTSTRING.asParameter(), Parameter.of(ParameterType.STRING_ANALYZER.getName(), 'standard'), Parameter.of(ParameterType.TEXT_ANALYZER.getName(), 'english')).buildMixedIndex("search")
mgmt.commit()
```
通过这些设置，JanusGraph将使用属性键字符串的标准分析器和属性键文本的英语分析器。
#### 25.2.1.2 对于solr
必须将tokenizer的类设置为参数值
```
mgmt = graph.openManagement()
string = mgmt.makePropertyKey('string').dataType(String.class).make()
text = mgmt.makePropertyKey('text').dataType(String.class).make()
mgmt.buildIndex('string', Vertex.class).addKey(string, Mapping.STRING.asParameter(), Parameter.of(ParameterType.STRING_ANALYZER.getName(), 'org.apache.lucene.analysis.standard.StandardTokenizer')).buildMixedIndex("search")
mgmt.buildIndex('text', Vertex.class).addKey(text, Mapping.TEXT.asParameter(), Parameter.of(ParameterType.TEXT_ANALYZER.getName(), 'org.apache.lucene.analysis.core.WhitespaceTokenizer')).buildMixedIndex("search")
mgmt.commit()
```
通过这些设置，JanusGraph将使用属性键字符串的标准标记化器和属性键文本的空白标记器。
#### 25.2.1.2 对于Lucene
必须将分析器的名称设置为参数值，或者默认为Mapping.STRING的KeywordAnalyzer和Mapping.TEXT的StandardAnalyzer。
```
mgmt = graph.openManagement()
string = mgmt.makePropertyKey('string').dataType(String.class).make()
text = mgmt.makePropertyKey('text').dataType(String.class).make()
name = mgmt.makePropertyKey('name').dataType(String.class).make()
document = mgmt.makePropertyKey('document').dataType(String.class).make()
mgmt.buildIndex('string', Vertex.class).addKey(string, Mapping.STRING.asParameter(), Parameter.of(ParameterType.STRING_ANALYZER.getName(), org.apache.lucene.analysis.core.SimpleAnalyzer.class.getName())).buildMixedIndex("search")
mgmt.buildIndex('text', Vertex.class).addKey(text, Mapping.TEXT.asParameter(), Parameter.of(ParameterType.TEXT_ANALYZER.getName(), org.apache.lucene.analysis.en.EnglishAnalyzer.class.getName())).buildMixedIndex("search")
mgmt.buildIndex('name', Vertex.class).addKey(string, Mapping.STRING.asParameter()).buildMixedIndex("search")
mgmt.buildIndex('document', Vertex.class).addKey(text, Mapping.TEXT.asParameter()).buildMixedIndex("search")
mgmt.commit()
```
通过这些设置，JanusGraph将使用SimpleAnalyzer分析器作为属性键字符串，使用EnglishAnalyzer 分析器获取属性键文本，使用KeywordAnalyzer分析器获取属性名称，使用StandardAnalyzer分析器获取属性文档。

### 25.2.2 自定义参数
有时需要在映射上设置其他参数（映射类型，映射名称和分析器除外）。例如，当我们想要使用不同的相似度算法（修改全文搜索的评分算法）或者如果我们想在Elasticsearch中的某些字段上使用自定义提升时，我们可以设置自定义参数（现在只有Elasticsearch支持自定义参数）。必须设置自定义参数的名称ParameterType.customParameterName("yourProperty")。

#### 25.2.2.1 对于Elasticsearch
```
mgmt = graph.openManagement()
myProperty = mgmt.makePropertyKey('my_property').dataType(String.class).make()
mgmt.buildIndex('custom_property_test', Vertex.class).addKey(myProperty, Mapping.TEXT.asParameter(), Parameter.of(ParameterType.customParameterName("boost"), 5), Parameter.of(ParameterType.customParameterName("similarity"), "boolean")).buildMixedIndex("search")
mgmt.commit()
```
通过这些设置，JanusGraph将使用属性键my_property的boost 5和布尔相似性算法。可能的映射参数取决于Elasticsearch版本。请参阅当前Elasticsearch版本的映射参数。


