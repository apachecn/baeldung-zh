# Apache Ignite 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-ignite>

## 1。简介

Apache Ignite 是一个开源的以内存为中心的分布式平台。我们可以将其用作数据库、缓存系统或用于内存中的数据处理。

该平台使用内存作为存储层，因此具有令人印象深刻的性能比率。简单地说，**这是目前生产中使用的最快的原子数据处理平台之一。**

## 2。安装和设置

首先，查看[入门页面](https://web.archive.org/web/20220628161709/https://apacheignite.readme.io/docs/getting-started)了解初始设置和安装说明。

我们将要构建的应用程序的 Maven 依赖性:

```java
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-core</artifactId>
    <version>${ignite.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-indexing</artifactId>
    <version>${ignite.version}</version>
</dependency>
```

**[`ignite-core`](https://web.archive.org/web/20220628161709/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.ignite%22%20AND%20a%3A%22ignite-core%22) 是项目**的唯一强制依赖项。由于我们也想与 SQL 交互，`ignite-indexing`也在这里。 [`${ignite.version}`](https://web.archive.org/web/20220628161709/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.ignite%22%20AND%20a%3A%22ignite-core%22) 是 Apache Ignite 的最新版本。

最后一步，我们启动 Ignite 节点:

```java
Ignite node started OK (id=53c77dea)
Topology snapshot [ver=1, servers=1, clients=0, CPUs=4, offheap=1.2GB, heap=1.0GB]
Data Regions Configured:
^-- default [initSize=256.0 MiB, maxSize=1.2 GiB, persistenceEnabled=false]
```

上面的控制台输出显示我们已经准备好了。

## 3.存储器体系结构

**该平台基于耐用内存架构**。这使得能够在磁盘和内存中存储和处理数据。它通过有效地使用集群的 RAM 资源来提高性能。

内存和磁盘上的数据具有相同的二进制表示。这意味着在从一层移动到另一层时，没有额外的数据转换。

持久内存架构分为固定大小的块，称为页面。页面存储在 Java 堆之外，组织在 RAM 中。它有唯一的标识符:`FullPageId`。

页面使用`PageMemory`抽象与内存交互。

它有助于读取、写入页面，也有助于分配页面 id。**在内存中，Ignite 将页面与`Memory Buffers`** 关联起来。

## 4。内存页面

页面可以有以下状态:

*   unloaded–内存中没有加载页面缓冲区
*   clear–页面缓冲区被加载并与磁盘上的数据同步
*   durty–页面缓冲区保存的数据不同于磁盘中的数据
*   检查点中的脏数据—在第一个修改保存到磁盘之前，又有一个修改开始。这里启动一个检查点，`PageMemory`为每页保留两个内存缓冲区。

**持久内存在本地分配一个叫做`Data Region`的内存段。**默认情况下，它有 20%的集群内存容量。多区域配置允许将可用数据保存在存储器中。

该区域的最大容量是一个内存段。它是一个物理内存或连续字节数组。

为了避免内存碎片，一个页面包含多个键值条目。每个新条目都将被添加到最佳页面。如果键-值对的大小超过了页面的最大容量，Ignite 会将数据存储在多个页面中。同样的逻辑也适用于更新数据。

SQL 和缓存索引存储在称为 B+树的结构中。缓存键按其键值排序。

## 5。生命周期

**每个 Ignite 节点运行在一个 JVM 实例上**。但是，可以配置在单个 JVM 进程中运行多个 Ignite 节点。

让我们看一下生命周期事件类型:

*   `BEFORE_NODE_START`–点火节点启动前
*   `AFTER_NODE_START`–在 Ignite 节点启动后立即触发
*   `BEFORE_NODE_STOP`–启动节点停止前
*   `AFTER_NODE_STOP`–点火节点停止后

要启动默认 Ignite 节点，请执行以下操作:

```java
Ignite ignite = Ignition.start();
```

或者来自配置文件:

```java
Ignite ignite = Ignition.start("config/example-cache.xml");
```

如果我们需要对初始化过程进行更多的控制，有另一种方法可以借助`LifecycleBean`接口:

```java
public class CustomLifecycleBean implements LifecycleBean {

    @Override
    public void onLifecycleEvent(LifecycleEventType lifecycleEventType) 
      throws IgniteException {

        if(lifecycleEventType == LifecycleEventType.AFTER_NODE_START) {
            // ...
        }
    }
}
```

在这里，我们可以使用生命周期事件类型在节点启动/停止之前或之后执行操作。

为此，我们将带有`CustomLifecycleBean`的配置实例传递给启动方法:

```java
IgniteConfiguration configuration = new IgniteConfiguration();
configuration.setLifecycleBeans(new CustomLifecycleBean());
Ignite ignite = Ignition.start(configuration);
```

## 6.内存数据网格

**Ignite 数据网格是一个分布式键值存储**，对于分区`HashMap`非常熟悉。它被水平缩放。这意味着我们添加的集群节点越多，缓存或存储在内存中的数据就越多。

作为一个额外的缓存层，它可以为 NoSql、RDMS 数据库等第三方软件提供显著的性能提升。

### 6.1。缓存支持

数据访问 API 基于 JCache JSR 107 规范。

例如，让我们使用模板配置创建一个缓存:

```java
IgniteCache<Employee, Integer> cache = ignite.getOrCreateCache(
  "baeldingCache");
```

让我们来看看这里发生了什么。首先，Ignite 会找到存储缓存的内存区域。

然后，B+树索引页将基于键散列码被定位。如果索引存在，将找到相应键的数据页。

当索引为空时，平台使用给定的键创建新的数据条目。

接下来，让我们添加一些`Employee`对象:

```java
cache.put(1, new Employee(1, "John", true));
cache.put(2, new Employee(2, "Anna", false));
cache.put(3, new Employee(3, "George", true));
```

同样，持久内存将寻找缓存所属的内存区域。基于缓存键，索引页将位于 B+树结构中。

当索引页不存在时，会请求一个新的索引页并将其添加到树中。

接下来，将数据页分配给索引页。

要从缓存中读取雇员，我们只需使用键值:

```java
Employee employee = cache.get(1);
```

### 6.2。流媒体支持

在内存中，数据流为基于磁盘和文件系统的数据处理应用程序提供了一种替代方法。**流式 API 将高负载数据流分成多个阶段，并对其进行路由处理**。

我们可以修改我们的示例，并从文件中传输数据。首先，我们定义一个数据流:

```java
IgniteDataStreamer<Integer, Employee> streamer = ignite
  .dataStreamer(cache.getName());
```

接下来，我们可以注册一个流转换器，将接收到的雇员标记为已雇用:

```java
streamer.receiver(StreamTransformer.from((e, arg) -> {
    Employee employee = e.getValue();
    employee.setEmployed(true);
    e.setValue(employee);
    return employee;
}));
```

最后一步，我们遍历`employees.txt`文件行，并将它们转换成 Java 对象:

```java
Path path = Paths.get(IgniteStream.class.getResource("employees.txt")
  .toURI());
Gson gson = new Gson();
Files.lines(path)
  .forEach(l -> streamer.addData(
    employee.getId(), 
    gson.fromJson(l, Employee.class)));
```

**使用`streamer.addData()`将雇员对象放入流中。**

## 7。SQL 支持

该平台提供以内存为中心的容错 SQL 数据库。

我们可以连接纯 SQL API 或 JDBC。这里的 SQL 语法是 ANSI-99，因此支持查询、DML、DDL 语言操作中的所有标准聚合函数。

### 7.1。JDBC

为了更加实际，让我们创建一个雇员表，并向其中添加一些数据。

为此，**我们注册了一个 JDBC 驱动程序，并在下一步打开了一个连接**:

```java
Class.forName("org.apache.ignite.IgniteJdbcThinDriver");
Connection conn = DriverManager.getConnection("jdbc:ignite:thin://127.0.0.1/");
```

在标准 DDL 命令的帮助下，我们填充了`Employee`表:

```java
sql.executeUpdate("CREATE TABLE Employee (" +
  " id LONG PRIMARY KEY, name VARCHAR, isEmployed tinyint(1)) " +
  " WITH \"template=replicated\"");
```

在 WITH 关键字之后，我们可以设置缓存配置模板。这里我们使用`REPLICATED`。**默认情况下，模板模式是`PARTITIONED`** 。**要指定数据的份数，我们也可以在这里指定`BACKUPS`参数，默认为 0。**

然后，让我们使用 INSERT DML 语句添加一些数据:

```java
PreparedStatement sql = conn.prepareStatement(
  "INSERT INTO Employee (id, name, isEmployed) VALUES (?, ?, ?)");

sql.setLong(1, 1);
sql.setString(2, "James");
sql.setBoolean(3, true);
sql.executeUpdate();

// add the rest 
```

之后，我们选择记录:

```java
ResultSet rs 
  = sql.executeQuery("SELECT e.name, e.isEmployed " 
    + " FROM Employee e " 
    + " WHERE e.isEmployed = TRUE ")
```

### 7.2。查询对象

**还可以对存储在缓存中的 Java 对象执行查询**。Ignite 将 Java 对象视为单独的 SQL 记录:

```java
IgniteCache<Integer, Employee> cache = ignite.cache("baeldungCache");

SqlFieldsQuery sql = new SqlFieldsQuery(
  "select name from Employee where isEmployed = 'true'");

QueryCursor<List<?>> cursor = cache.query(sql);

for (List<?> row : cursor) {
    // do something with the row
}
```

## 8。总结

在本教程中，我们快速浏览了 Apache Ignite 项目。本指南强调了该平台相对于其他类似产品的优势，如性能提升、耐用性、轻量级 API。

结果，**我们学会了如何使用 SQL 语言和 Java API 来存储、检索、流式传输持久性或内存网格中的数据。**

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628161709/https://github.com/eugenp/tutorials/tree/master/libraries-data/)