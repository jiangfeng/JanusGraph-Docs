

每个JanusGraph都有一个schema，该schema由edge labels，property keys,和vertex组成.JanusGraph schema可以显示（明确）定义，也可以隐式定义，鼓励用户在开发应用程式时显示定义schema。 明确定义的schema是一个健壮的图应用程序的重要组成部分，并且极大的改善软件的协同开发。要注意的是，JanusGraph schema会随着时间的推移而不断改变，并且不会中断正常的数据库操作。扩展schema并不会拖慢查询的速度，也不需要数据库停机。

## 5.1 Defining Edge Labels(定义边标签)
链接两个定点的边都有一个定义关系的标签。例如，顶点A和顶点B之间的边标签friend标志着两人之间的友谊关系。

要定义边标签，请在打开的图形或管理事务上调用MakeEdgeLabel ( String )，并提供边标签的名称作为参数。边标签名称在图中必须唯一。方法会返回一个允许定义边标签多样性的builder。边标签的多样性定义了具有该标签的边的多样性约束，既一对顶点间的最大边个数。JanusGraph支持如下的多样性。

### 5.1.1 Edge Lable Multiplicity（边标签的多样性）
Multiplicity Settings（多样性设置）

* MULTI：任意一对的顶点间允许多个标签相同的边。也就是说，该图是关于这种边缘标签的多图。边的多样性没有约束。
* SIMPLE：任意一对顶点之间只允许一条此类标签的边。也就是说，该图是关于标签的简单图。确保对于给定的标签和一对顶点边是唯一的。
* MANY2ONE:在图形中的任何顶点上最多允许此标签的一个输出边，但不限制输入边个数。边缘标签mother是MANY2ONE多样性的一个例子，因为每个人最多只有一个母亲，但母亲可以有多个孩子。
* ONE2MANY：在图形中的任何顶点上最多允许此标签的一个输入边，但不限制输出边。边标签winnerOf是ONE2MANY多样性的一个例子，因为每个比赛最多只赢一个人，但一个人可以赢得多个比赛。
* ONE2ONE：在图表的任何顶点上最多允许此标签的一个输入边和一个输出边。边标签married是ONE2ONE多样性的一个例子，因为一个人与另一个人结婚。

默认的多样性是MULTI。通过调用make()构建器上的方法来完成边标签的定义，该构建器返回定义的边标签，如以下示例所示。

	mgmt = graph.openManagement()
	follow = mgmt.makeEdgeLabel('follow').multiplicity(MULTI).make()
	mother = mgmt.makeEdgeLabel('mother').multiplicity(MANY2ONE).make()
	mgmt.commit()

## 5.2 Defining Property Keys (定义属性键)

顶点和边的属性是键值对.例如，属性name='Daniel'具有键name和值'Daniel'。 属性键是JanusGraph架构的一部分，可以约束允许的数据类型和值的Cardinality（基数）。
要定义属性键，请调用makePropertyKey(String)打开的图或管理事务器（management transaction），并提供属性键的名称作为参数。属性键名称在图形中必须是唯一的，建议避免使用属性名称中的空格或特殊字符。此方法返回属性键的构建器。

### 5.2.1 Property Key Data Type（属性键数据类型）

使用dataType(Class)来定义属性键的数据类型。JanusGraph将强制执行与关联键的所有值，确人都具有已配置的数据类型，从而确保添加到图中的数据有效。例如，可以定义为name键具有String数据类型。

定义数据类型Object.class，以便允许任何（可序列化）值与键关联。但是，鼓励尽可能使用具体的数据类型。配置的数据类型必须是实体类，而不是接口或抽象类。JanusGraph强制实现类相等，因此不允许添加已配置数据类型的子类。

JanusGraph支持以下数据类型。

表5.1 原生JanusGraph数据类型

名称	描述
String	字符串
Character	单个字符
Boolean	true或false
Byte	byte值
Short	short值
Integer	integer值
Long	long值
Float	4字节浮点数
Double	8字节浮点数
Date	时间(java.util.Date)
Geoshape	地理形状，如点，圆或框
UUID	通用唯一标识符（java.util.UUID）

### 5.2.2 Property Key Cardinality（属性键的基数）

使用cardinality(Cardinality)定义与在任何给定的顶点的键关联的值允许的基数。

基数的设置
SINGLE:对于此键，每个元素最多允许一个值。换句话说，键→值映射对于图中的所有元素都是唯一的。属性键birthDate是SINGLE基数的一个例子，因为每个人只有一个出生日期。
LIST:允许每个元素的任意数量的值用于此类键。换句话说，键与允许重复值的值列表相关联。假设我们将传感器建模为图形中的顶点，则属性键sensorReading是具有LIST基数的示例，以允许记录大量（可能重复的）传感器读数。
SET:允许多个但不重复的值用于此类键。换句话说，键与一组不重复的值相关联。name如果我们想要捕获个人的所有姓名（包括昵称，婚前姓名等），则属性键具有SET基数。
默认的基数是SINGLE。要注意的是，边和属性使用SINGLE作为基数，是不支持为为边或属性附加多个值。

	mgmt = graph.openManagement()
	birthDate = mgmt.makePropertyKey('birthDate').dataType(Long.class).cardinality(Cardinality.SINGLE).make()
	name = mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.SET).make()
	sensorReading = mgmt.makePropertyKey('sensorReading').dataType(Double.class).cardinality(Cardinality.LIST).make()
	mgmt.commit()

## 5.3 Relation Types（关系类型）

边标签和属性键共同称为关系类型。关系类型的名称在图中必须唯一，这意味着属性键和边缘标签不能具有相同的名称。JanusGraph API中有一些方法可以查询是否存在或检索包含属性键和边标签的关系类型。

	mgmt = graph.openManagement()
	if (mgmt.containsRelationType('name'))
	    name = mgmt.getPropertyKey('name')
	mgmt.getRelationTypes(EdgeLabel.class)
	mgmt.commit()

## 5.4 Defining Vertex Labels（定义顶点标签）
像边一样，顶点也有标签。但与边缘标签不同，顶点标签是可选的。顶点标签可用于区分不同类型的顶点，例如用户顶点和产品顶点。

尽管标签在概念和数据模型级别是可选的，但JanusGraph将标签作为内部实现分配各每个顶点。由AddVertex方法创建的顶点使用JanuSgraph的默认标签。

要创建标签，调用makeVertexLabel(String).make()打开的图形或管理事务，并提供顶点标签的名称作为参数。顶点标签名称在图表中必须是唯一的。

	mgmt = graph.openManagement()
	person = mgmt.makeVertexLabel('person').make()
	mgmt.commit()
	// Create a labeled vertex
	person = graph.addVertex(label, 'person')
	// Create an unlabeled vertex
	v = graph.addVertex()
	graph.tx().commit()

## 5.5 Automatic Schema Maker（自动创建schema）

如果没有明确定义边标签，属性键或顶点标签，则在添加边，顶点或属性设置期间首次使用时，将隐式定义边标签，属性键或顶点标签。DefaultSchemaMaker配置用于JanusGraph图定义这样的类型。

默认情况下，隐式创建的边标签具有多样性MULTI，隐式创建的属性键具有基数SINGLE和数据类型Object.class。用户可以通过实现和注册他们自己的DefaultSchemaMaker来控制schema元素的自动创建。

强烈建议通过schema.default=none在JanusGraph图形配置中进行设置，明确定义所有schema元素并禁用自动schema创建。

## 5.6 Changing Schema Elements(更改schema元素)
边标签，属性键或顶点标的定义一旦提交到图，就无法更改。可以通过JanusGraphManagement.changeName(JanusGraphSchemaElement, String) 更改属性键的名称，如下示例属性键place重命名为location.

	mgmt = graph.openManagement()
	place = mgmt.getPropertyKey('place')
	mgmt.changeName(place, 'location')
	mgmt.commit()

要注意的是，在当前运行的事务和集群中的其他JanusGraph图形实例中，模式名称更改可能不会立即可见。尽管存储后端向所有的JanusGraph实例通知schema名称的更改，但模式schema的更改可能需要一段时间才能生效，并且在某些故障情况，如网络分区，可能需要重新启动实例。如果这些与重命名同时发生。因此，用户必须确保满足以下任一条件：

重命名的标签和键当前未处于活跃状态（既写入或读取），并且在所有JanusGraph应答schema名称更改前不会使用。
运行事务会主动适应短暂的中间时期，在这期间，旧名称或新名称都是有效的，这取决于JanuSgraph实例和名称更改公告的状态。这可能意味着同时到查询两个名称的schema。
如果需要重新定义现有schema类型，建议将此类型的名称修改成现在未使用（并且永不使用）。之后，可以使用原始名称定义新的标签或key，从而有效的替换旧标签或键。但需要注意的是，这不会影响以前现有类型写入的顶点、边或属性。 不支持在线重新定义现有图形元素，必须通过批量图形转换完成。

## 5.7 Schema Constraints（schema约束）

模式的定义允许用户配置显式属性和连接约束。属性可以绑定到特定的顶点标签和（或）边标签。此外，链接约束允许用户明确定义那两个顶点标签允许可以通过边标签连接。这些约束可用于确保图形与给定的域模型匹配。例如在众神图，一个god可以是另一个兄弟的god，不是一个monster的兄弟，神可以有年龄属性，但是location不能有年龄属性。默认情况下禁用这些约束。

启用这些schema约束需要设置schema.constraints=true。此设置取决于设置schema.default。如果schema.default设置成none, 则违反schema约束会抛出IllegalArgumentException。如果schema.default没有设置成none，则会自动创建schema约束，但不会抛出异常。激活模式约束对现有数据没有影响，因为这些模式约束仅作用在插入的过程。因此，这些约束完全不会影响数据的读取。

可以使用JanusGraphManagement.addProperties(VertexLabel, PropertyKey...)绑定多个属性到顶点，例如：

	mgmt = graph.openManagement()
	person = mgmt.makeVertexLabel('person').make()
	name = mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.SET).make()
	birthDate = mgmt.makePropertyKey('birthDate').dataType(Long.class).cardinality(Cardinality.SINGLE).make()
	mgmt.addProperties(person, name, birthDate)
	mgmt.commit()

可以使用JanusGraphManagement.addProperties(EdgeLabel, PropertyKey...)为边绑定多个属性。例如：

	mgmt = graph.openManagement()
	follow = mgmt.makeEdgeLabel('follow').multiplicity(MULTI).make()
	name = mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.SET).make()
	mgmt.addProperties(follow, name)
	mgmt.commit()

可以使用JanusGraphManagement.addConnection(EdgeLabel, VertexLabel out, VertexLabel in)定义传出，传入和边之间的连接，例如：

	mgmt = graph.openManagement()
	person = mgmt.makeVertexLabel('person').make()
	company = mgmt.makeVertexLabel('company').make()
	works = mgmt.makeEdgeLabel('works').multiplicity(MULTI).make()
	mgmt.addConnection(works, person, company)
	mgmt.commit()