36、 数据类型和属性序列化配置


JanusGraph支持许多属性值的类。JanusGraph有效地串行化基元，基本数组和Geoshape，UUID和Date。JanusGraph支持将任意对象序列化为属性值，但这些需要定义自定义序列化程序。

要使用自定义序列化程序配置自定义属性类，请按照下列步骤操作：
- 实现自AttributeSerializer定义属性类的自定义
- 添加以下配置选项，其中[X]是自定义属性id，必须大于已配置的自定义属性的所有属性ID：
 > a.attributes.custom.attribute[X].attribute-class = [Full attribute class name]
b.attributes.custom.attribute[X].serializer-class = [Full serializer class name]

例如，假设我们要注册一个特殊的整数属性类，SpecialInt并且已经实现了一个实现的自定义序列化SpecialIntSerializer程序AttributeSerializer。我们已经在配置文件中配置了9个自定义属性，因此我们将添加以下行
```
attributes.custom.attribute10.attribute-class = com.example.SpecialInt
attributes.custom.attribute10.serializer-class = com.example.SpecialIntSerializer
```
### 36.1 自定义对象序列化
JanusGraph支持任意对象作为属性的属性，并可以将这些对象序列化到磁盘。要使此默认序列化程序适用于自定义类，必须满足以下条件：

- 该类必须实现AttributeSerializer
- 该类必须具有无参数构造函数
- 该类必须实现该equals(Object)方法
最后一个要求是必需的，因为JanusGraph将在将数据保存到磁盘之前测试自定义类的序列化和反序列化。