# 带 Java 的 HBase

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hbase>

## 1。概述

在本文中，我们将关注 [`HBase`](https://web.archive.org/web/20220815030127/https://hbase.apache.org/) 数据库 Java 客户端库。HBase 是一个分布式数据库，使用 Hadoop 文件系统来存储数据。

我们将创建一个 Java 示例客户机和一个表，我们将向其中添加一些简单的记录。

## 2。HBase 数据结构

在 HBase 中，数据被分组到列族中。**列族的所有列成员都有相同的前缀。**

例如，列`family1:qualifier1`和`family1` `:qualifier2`都是`family1`列家族的成员。所有列族成员都存储在文件系统中。

在列族内部，我们可以放置一个具有指定限定符的行。我们可以把限定符看作是一种列名。

让我们看看 Hbase 中的一个示例记录:

```java
Family1:{  
   'Qualifier1':'row1:cell_data',
   'Qualifier2':'row2:cell_data',
   'Qualifier3':'row3:cell_data'
}
Family2:{  
   'Qualifier1':'row1:cell_data',
   'Qualifier2':'row2:cell_data',
   'Qualifier3':'row3:cell_data'
}
```

我们有两个列族，每个列族都有三个限定符，其中包含一些单元格数据。每一行都有一个行键，它是唯一的行标识符。我们将使用行键来插入、检索和删除数据。

## 3。HBase 客户端 Maven 依赖关系

在我们连接到 HBase 之前，我们需要添加 [`hbase-client`](https://web.archive.org/web/20220815030127/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.hbase%22%20AND%20a%3A%22hbase-client%22) 和`[hbase](https://web.archive.org/web/20220815030127/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.hbase%22%20AND%20a%3A%22hbase%22)`依赖项:

```java
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>${hbase.version}</version>
</dependency>
<dependency>
     <groupId>org.apache.hbase</groupId>
     <artifactId>hbase</artifactId>
     <version>${hbase.version}</version>
</dependency>
```

## 4\. HBase Setup

我们需要设置 HBase，以便能够从 Java 客户端库连接到它。安装超出了本文的范围，但是您可以在线查看一些 HBase 安装指南。

接下来，我们需要通过执行以下命令在本地启动 HBase master:

```java
hbase master start
```

## 5。从 Java 连接到 HBase】

为了以编程方式从 Java 连接到 HBase，我们需要定义一个 XML 配置文件。我们在本地主机上启动了 HBase 实例，因此我们需要将其输入到配置文件中:

```java
<configuration>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>localhost</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
    </property>
</configuration> 
```

现在，我们需要将 HBase 客户端指向该配置文件:

```java
Configuration config = HBaseConfiguration.create();

String path = this.getClass()
  .getClassLoader()
  .getResource("hbase-site.xml")
  .getPath();
config.addResource(new Path(path)); 
```

接下来，我们将检查与 HBase 的连接是否成功——如果失败，将抛出`MasterNotRunningException` :

```java
HBaseAdmin.checkHBaseAvailable(config);
```

## 6。创建数据库结构

在开始向 HBase 添加数据之前，我们需要创建用于插入行的数据结构。我们将创建一个包含两个柱族的表:

```java
private TableName table1 = TableName.valueOf("Table1");
private String family1 = "Family1";
private String family2 = "Family2";
```

首先，我们需要创建一个到数据库的连接，并获得`admin`对象，我们将使用它来操作数据库结构:

```java
Connection connection = ConnectionFactory.createConnection(config)
Admin admin = connection.getAdmin();
```

然后，我们可以通过将`HTableDescriptor` 类的实例传递给`admin` 对象上的`createTable()` 方法来创建一个表:

```java
HTableDescriptor desc = new HTableDescriptor(table1);
desc.addFamily(new HColumnDescriptor(family1));
desc.addFamily(new HColumnDescriptor(family2));
admin.createTable(desc);
```

## 7 .**。添加和检索元素**

随着表的创建，我们可以通过创建一个`Put` 对象并在`Table` 对象上调用一个`put()` 方法来添加新数据:

```java
byte[] row1 = Bytes.toBytes("row1")
Put p = new Put(row1);
p.addImmutable(family1.getBytes(), qualifier1, Bytes.toBytes("cell_data"));
table1.put(p);
```

检索先前创建的行可以通过使用一个`Get` 类来实现:

```java
Get g = new Get(row1);
Result r = table1.get(g);
byte[] value = r.getValue(family1.getBytes(), qualifier1);
```

`row1` 是一个行标识符——我们可以用它从数据库中检索特定的行。拨打电话时:

```java
Bytes.bytesToString(value)
```

返回的结果将是先前插入的`cell_data.`

## 8。扫描和过滤

我们可以扫描该表，通过使用一个`Scan` 对象检索给定限定符内的所有元素(注意`ResultScanner`扩展了`Closable`，所以在完成后一定要对其调用`close()`):

```java
Scan scan = new Scan();
scan.addColumn(family1.getBytes(), qualifier1);

ResultScanner scanner = table.getScanner(scan);
for (Result result : scanner) {
    System.out.println("Found row: " + result);
} 
```

该操作将打印一个`qualifier1`中的所有行以及一些附加信息，如时间戳:

```java
Found row: keyvalues={Row1/Family1:Qualifier1/1488202127489/Put/vlen=9/seqid=0}
```

我们可以通过使用过滤器来检索特定的记录。

首先，我们创建两个过滤器。`filter1` 指定扫描查询将检索大于`row1,` 的元素，而`filter2` 指定我们只对限定符等于`qualifier1`的行感兴趣:

```java
Filter filter1 = new PrefixFilter(row1);
Filter filter2 = new QualifierFilter(
  CompareOp.GREATER_OR_EQUAL, 
  new BinaryComparator(qualifier1));
List<Filter> filters = Arrays.asList(filter1, filter2);
```

然后我们可以从一个`Scan`查询中得到一个结果集:

```java
Scan scan = new Scan();
scan.setFilter(new FilterList(Operator.MUST_PASS_ALL, filters));

try (ResultScanner scanner = table.getScanner(scan)) {
    for (Result result : scanner) {
        System.out.println("Found row: " + result);
    }
}
```

当创建一个`FilterList` 时，我们传递了一个`Operator.MUST_PASS_ALL` ——这意味着必须满足所有的过滤器。如果只需要满足一个滤波器，我们可以选择`Operation.MUST_PASS_ONE` 。在结果集中，我们将只有匹配指定过滤器的行。

## 9。删除行

最后，要删除一行，我们可以使用一个`Delete` 类:

```java
Delete delete = new Delete(row1);
delete.addColumn(family1.getBytes(), qualifier1);
table.delete(delete);
```

我们正在删除驻留在`family1` `.`中的`row1`

## 10。结论

在这个快速教程中，我们重点讨论了与 HBase 数据库的通信。我们看到了如何从 Java 客户端库连接到 HBase，以及如何运行各种基本操作。

所有这些例子和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220815030127/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hbase)中找到；这是一个 Maven 项目，因此应该很容易导入和运行。