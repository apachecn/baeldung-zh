# MapDB 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mapdb>

## 1.介绍

在本文中，我们将看看 **[MapDB](https://web.archive.org/web/20221024071539/http://www.mapdb.org/) 库——一个通过类似集合的 API 访问的嵌入式数据库引擎。**

我们从探索帮助配置、打开和管理数据库的核心类`DB `和`DBMaker `开始。然后，我们将深入一些存储和检索数据的 MapDB 数据结构的例子。

最后，在将 MapDB 与传统数据库和 Java 集合进行比较之前，我们将了解一些内存模式。

## 2.**在 MapDB 中存储数据**

首先，让我们介绍一下我们将在本教程中经常用到的两个类— `DB `和`DBMaker. `**`DB`类代表一个开放数据库。**它的方法调用创建和关闭存储集合的动作来处理数据库记录，以及处理事务性事件。

**`DBMaker`处理数据库的配置、创建和打开。**作为配置的一部分，我们可以选择在内存或文件系统中托管数据库。

### 2.1.一个简单的`HashMap`例子

为了理解这是如何工作的，让我们在内存中实例化一个新数据库。

首先，让我们使用`DBMaker `类创建一个新的内存数据库:

```java
DB db = DBMaker.memoryDB().make();
```

一旦我们的`DB `对象启动并运行，我们就可以用它构建一个`HTreeMap `来处理我们的数据库记录:

```java
String welcomeMessageKey = "Welcome Message";
String welcomeMessageString = "Hello Baeldung!";

HTreeMap myMap = db.hashMap("myMap").createOrOpen();
myMap.put(welcomeMessageKey, welcomeMessageString);
```

`HTreeMap `是 MapDB 的`HashMap `实现。所以，现在我们的数据库中有了数据，我们可以使用`get `方法来检索它:

```java
String welcomeMessageFromDB = (String) myMap.get(welcomeMessageKey);
assertEquals(welcomeMessageString, welcomeMessageFromDB);
```

最后，既然我们已经完成了数据库，我们应该关闭它以避免进一步的变异:

```java
db.close();
```

为了将数据存储在文件中，而不是存储在内存中，我们需要做的就是改变我们的`DB`对象的实例化方式:

```java
DB db = DBMaker.fileDB("file.db").make();
```

我们上面的例子没有使用类型参数。因此，我们不得不将我们的结果转换成特定的类型。在我们的下一个例子中，我们将引入`Serializers `来消除造型的需要。

### 2.2.收集

**MapDB 包括不同的集合类型。**为了演示，让我们使用`NavigableSet`从我们的数据库中添加和检索一些数据，它的工作方式就像您对 Java `Set`的期望一样:

让我们从一个简单的`DB `对象实例开始:

```java
DB db = DBMaker.memoryDB().make();
```

接下来，让我们创建我们的`NavigableSet`:

```java
NavigableSet<String> set = db
  .treeSet("mySet")
  .serializer(Serializer.STRING)
  .createOrOpen();
```

在这里，`serializer`确保来自我们数据库的输入数据使用`String` 对象进行序列化和反序列化。

接下来，让我们添加一些数据:

```java
set.add("Baeldung");
set.add("is awesome");
```

现在，让我们检查我们的两个不同的值是否已经正确地添加到数据库中:

```java
assertEquals(2, set.size());
```

最后，由于这是一个集合，让我们添加一个重复的字符串，并验证我们的数据库仍然只包含两个值:

```java
set.add("Baeldung");

assertEquals(2, set.size());
```

### 2.3.处理

与传统数据库非常相似， **`DB `类提供了对我们添加到数据库中的数据进行`commit `和`rollback `的方法。**

为了启用这个功能，我们需要用`transactionEnable `方法初始化我们的`DB `:

```java
DB db = DBMaker.memoryDB().transactionEnable().make();
```

接下来，让我们创建一个简单的集合，添加一些数据，并提交给数据库:

```java
NavigableSet<String> set = db
  .treeSet("mySet")
  .serializer(Serializer.STRING)
  .createOrOpen();

set.add("One");
set.add("Two");

db.commit();

assertEquals(2, set.size());
```

现在，让我们向数据库添加第三个未提交的字符串:

```java
set.add("Three");

assertEquals(3, set.size());
```

如果我们对数据不满意，我们可以使用`DB's rollback` 方法回滚数据:

```java
db.rollback();

assertEquals(2, set.size());
```

### 2.4.序列化程序

MapDB 提供了各种各样的[序列化器，它们处理集合](https://web.archive.org/web/20221024071539/https://jankotek.gitbooks.io/mapdb/content/htreemap/#serializers)中的数据。最重要的构造参数是名称，它标识了`DB `对象中的单个集合:

```java
HTreeMap<String, Long> map = db.hashMap("indentification_name")
  .keySerializer(Serializer.STRING)
  .valueSerializer(Serializer.LONG)
  .create();
```

虽然建议使用序列化，但它是可选的，可以跳过。然而，值得注意的是，这将导致更慢的通用序列化过程。

## 3.`HTreeMap`

MapDB 的 **`HTreeMap `为使用我们的数据库提供了`HashMap `和`HashSet `集合。** `HTreeMap`是分段哈希树，不使用固定大小的哈希表。相反，它使用自动扩展的索引树，并且不会随着表的增长而重新散列所有数据。更重要的是， **`HTreeMap `是线程安全的，支持使用多个段的并行写入。**

首先，让我们实例化一个简单的`HashMap `，它将`String`用于键和值:

```java
DB db = DBMaker.memoryDB().make();

HTreeMap<String, String> hTreeMap = db
  .hashMap("myTreeMap")
  .keySerializer(Serializer.STRING)
  .valueSerializer(Serializer.STRING)
  .create();
```

上面，我们已经为键和值定义了单独的`serializers `。既然我们的`HashMap `已经创建，让我们使用`put `方法添加数据:

```java
hTreeMap.put("key1", "value1");
hTreeMap.put("key2", "value2");

assertEquals(2, hTreeMap.size());
```

由于`HashMap `使用的是`Object's hashCode `方法，使用相同的键添加数据会导致值被覆盖:

```java
hTreeMap.put("key1", "value3");

assertEquals(2, hTreeMap.size());
assertEquals("value3", hTreeMap.get("key1"));
```

## 4.`SortedTableMap`

MapDB 的 **`SortedTableMap `将键存储在一个固定大小的表中，并使用二分搜索法进行检索。** **值得注意的是，一旦准备好，地图是只读的。**

让我们浏览一下创建和查询一个`SortedTableMap. `的过程。我们将首先创建一个内存映射卷来保存数据，并创建一个接收器来添加数据。在第一次调用我们的卷时，我们将只读标志设置为`false`，确保我们可以写入卷:

```java
String VOLUME_LOCATION = "sortedTableMapVol.db";

Volume vol = MappedFileVol.FACTORY.makeVolume(VOLUME_LOCATION, false);

SortedTableMap.Sink<Integer, String> sink =
  SortedTableMap.create(
    vol,
    Serializer.INTEGER,
    Serializer.STRING)
    .createFromSink();
```

接下来，我们将添加我们的数据并调用 sink 上的`create `方法来创建我们的地图:

```java
for(int i = 0; i < 100; i++){
  sink.put(i, "Value " + Integer.toString(i));
}

sink.create();
```

现在我们的地图已经存在，我们可以定义一个只读卷，并使用`SortedTableMap's open` 方法打开我们的地图:

```java
Volume openVol = MappedFileVol.FACTORY.makeVolume(VOLUME_LOCATION, true);

SortedTableMap<Integer, String> sortedTableMap = SortedTableMap
  .open(
    openVol,
    Serializer.INTEGER,
    Serializer.STRING);

assertEquals(100, sortedTableMap.size());
```

### 4.1.二进位检索

在我们继续之前，让我们更详细地了解一下`SortedTableMap `是如何利用二分搜索法的。

`SortedTableMap` 将存储拆分成页面，每个页面包含几个由键和值组成的节点。在这些节点中是我们在 Java 代码中定义的键值对。

`SortedTableMap`执行三次二进制搜索以检索正确的值:

1.  每页的键存储在堆上的一个数组中。`SortedTableMap` 执行二分搜索法来找到正确的页面。
2.  接下来，对节点中的每个键进行解压缩。二分搜索法根据密钥建立正确的节点。
3.  最后，`SortedTableMap` 搜索节点中的键以找到正确的值。

## 5.内存模式

MapDB 提供了三种类型的内存存储。让我们快速浏览一下每种模式，了解其工作原理，并研究其优势。

### 5.1.堆上

堆上模式**在一个简单的 Java 集合`Map`中存储对象。**它**不采用序列化，对于小数据集来说速度非常快。**

但是，由于数据存储在堆上，数据集由垃圾收集(GC)管理。GC 的持续时间随着数据集的大小而增加，导致性能下降。

让我们看一个指定堆上模式的例子:

```java
DB db = DBMaker.heapDB().make();
```

### 5.2.`Byte[]`

第二种存储类型基于字节数组。在这种模式下，**数据被序列化并存储到最大 1MB 的数组中。**虽然从技术上来说是堆上的，但这种方法对于垃圾收集来说更有效。

这是默认推荐的，并在我们的' [`Hello Baeldung'` 示例](#db)中使用:

```java
DB db = DBMaker.memoryDB().make();
```

### 5.3.`DirectByteBuffer`

最终存储基于 Java 1.4 中引入的`DirectByteBuffer.`直接内存，允许将数据直接传递到本机内存，而不是 Java 堆。因此，数据将完全存储在堆外**。**

我们可以通过以下方式调用这种类型的存储:

```java
DB db = DBMaker.memoryDirectDB().make();
```

## 6.为什么是 MapDB？

那么，为什么要使用 MapDB 呢？

### 6.1.MapDB 与传统数据库

MapDB 提供了大量的数据库功能，只需几行 Java 代码就可以完成配置。当我们使用 MapDB 时，我们可以避免为使我们的程序工作而设置各种服务和连接的耗时工作。

除此之外，MapDB 允许我们像熟悉 Java 集合一样访问数据库的复杂性。使用 MapDB，我们不需要 SQL，我们可以通过简单的`get `方法调用来访问记录。

### 6.2.MapDB 与简单 Java 集合

一旦应用程序停止执行，Java 集合就不会持久保存应用程序的数据。MapDB 提供了一个简单、灵活、可插拔的服务，允许我们快速、轻松地将数据持久化到应用程序中，同时保持 Java 集合类型的实用性。

## 7.结论

在本文中，我们深入研究了 MapDB 的嵌入式数据库引擎和收集框架。

我们从查看核心类`DB `和`DBMaker `开始，以配置、打开和管理我们的数据库。然后，我们浏览了一些 MapDB 提供的处理记录的数据结构的例子。最后，我们研究了 MapDB 相对于传统数据库或 Java 集合的优势。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221024071539/https://github.com/eugenp/tutorials/tree/master/libraries-2)