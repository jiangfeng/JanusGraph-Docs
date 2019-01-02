# **第40章JanusGraph数据模型**

JanusGraph以[邻接列表格式](http://en.wikipedia.org/wiki/Adjacency_list)存储图形，这意味着图形存储为具有邻接列表的顶点集合。顶点的邻接列表包含所有顶点的入射边（和属性）。

通过以邻接列表格式存储图形，JanusGraph确保所有顶点的入射边缘和属性紧密地存储在存储后端中，从而加速遍历。缺点是每个边缘必须存储两次 - 边缘的每个末端顶点一次。

此外，JanusGraph按排序顺序维护每个顶点的邻接列表，其顺序由排序键和边标签的排序顺序定义。排序顺序允许使用以[顶点为中心的索引](https://docs.janusgraph.org/latest/indexes.html#vertex-indexes)有效地检索邻接列表的子集。

JanusGraph将图形的邻接列表表示[存储](https://docs.janusgraph.org/latest/storage-backends.html)在支持Bigtable数据模型的任何[存储后端](https://docs.janusgraph.org/latest/storage-backends.html)中

## 40.1。Bigtable数据模型

![bigtablemodel](https://docs.janusgraph.org/latest/images/bigtablemodel.png)

在[Bigtable数据模型下，](http://en.wikipedia.org/wiki/Bigtable)每个表都是行的集合。每行由密钥唯一标识。每行由任意（大但有限）数量的单元组成。单元格由列和值组成。单元格由给定行内的列唯一标识。Bigtable模型中的行称为“宽行”，因为它们支持大量单元格，并且不必像关系数据库中那样预先定义这些单元格的列。

JanusGraph对Bigtable数据模型还有一个额外要求：单元格必须按列排序，并且列范围指定的单元格子集必须是有效可检索的（例如，通过使用索引结构，跳过列表或二进制搜索）。

此外，特定的Bigtable实现可以使行按其键的顺序排序。JanusGraph可以利用这样的键序来有效地划分图形，从而为非常大的图形提供更好的加载和遍历性能。但是，这不是必需的。

## 40.2。JanusGraph数据布局

![storagelayout](https://docs.janusgraph.org/latest/images/storagelayout.png)

JanusGraph将每个邻接列表作为一行存储在底层存储后端中。（64位）顶点id（JanusGraph唯一分配给每个顶点）是指向包含顶点邻接列表的行的键。每个边和属性都存储在行中的单个单元格中，从而允许有效的插入和删除。因此，特定存储后端中每行允许的最大单元数也是JanusGraph可以支持此顶端的最大顶点数。

如果存储后端支持键顺序，则邻接列表将按顶点id排序，并且JanusGraph可以分配顶点id，以便图形被有效地分区。分配ID使得经常共同访问的顶点具有小的绝对差异的id。

## 40.3。个别边缘布局

![relationlayout](https://docs.janusgraph.org/latest/images/relationlayout.png)

每个边和属性都作为一个单元格存储在其相邻顶点的行中。它们被序列化，使得列的字节顺序遵循边标签的排序键。变量id编码方案和压缩对象序列化用于使每个边缘/单元的存储足迹尽可能小。

考虑在上图中顶行可视化的单个边的存储布局。深蓝色框表示使用可变长度编码方案编码的数字，以减少它们消耗的字节数。红框表示使用关联属性键中引用的压缩元数据序列化的一个或多个属性值（即对象）。灰色框表示未压缩的属性值（即序列化对象）。

边的序列化表示从边标签的唯一ID开始（由JanusGraph指定）。这通常是一个小数字，并使用变量id编码进行压缩。该id的最后一位是偏移量，用于存储这是传入边缘还是传出边缘。接下来，存储包括排序键的属性值。排序键是使用边缘标签定义的，因此排序键对象元数据可以引用边缘标签。之后，存储相邻顶点的id。JanusGraph不存储实际的顶点id，而是存储拥有此邻接列表的顶点的id的差异。差异可能是比绝对id小的数字，因此压缩得更好。顶点id后跟此边的id。JanusGraph为每条边分配了一个唯一的id。这样就结束了边缘单元格的列值。边的单元格的值包含边的签名属性的压缩序列化（由标签的签名键定义）以及在未压缩序列化中添加到边的任何其他属性。

属性的序列化表示更简单，只包含列中属性的键ID。属性id和属性值存储在值中。`list()`但是，如果将属性键定义为，则属性id也存储在列中。

 