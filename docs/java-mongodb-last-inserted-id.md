# 用 Java 驱动程序获取 MongoDB 中最后插入的文档 ID

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-last-inserted-id>

## 1.概观

有时，我们需要刚刚插入到 [MongoDB](/web/20220524071051/https://www.baeldung.com/java-mongodb) 数据库中的文档的 ID。例如，我们可能希望将 ID 作为响应发送回调用者，或者记录所创建的对象以便进行调试。

在本教程中，我们将看到如何在 MongoDB 中实现 IDs，以及如何通过 Java 程序检索我们刚刚插入到集合中的文档的 ID。

## 2.MongoDB 文档的 ID 是什么？

与每个数据存储系统一样，MongoDB 需要为存储在集合中的每个文档提供一个惟一的标识符。这个标识符相当于关系数据库中的主键。

在 MongoDB 中，这个 ID 由 12 个字节组成:

*   一个 4 字节的时间戳值表示自 Unix 纪元以来的秒数
*   每个进程生成一次的 5 字节随机值。这个随机值对于机器和过程是唯一的。
*   一个 3 字节递增计数器

**ID 存储在名为`_id`的字段中，由客户端生成。**这意味着必须在将文档发送到数据库之前生成 ID。在客户端，我们可以使用驱动程序生成的 ID，也可以生成自定义 ID。

我们可以看到，由同一个客户机在同一秒钟内创建的文档将有共同的前 9 个字节。因此，在这种情况下，ID 的唯一性依赖于计数器。该计数器让客户在同一秒钟内创建超过 1600 万个文档。

虽然它以时间戳开始，但是我们应该注意不要将标识符用作排序标准。这是因为在同一秒内创建的文档不能保证按创建日期排序，因为计数器不能保证是单调的。此外，不同的客户端可能有不同的系统时钟。

Java 驱动程序为计数器使用随机数生成器，它不是单调的。这就是为什么我们不应该使用驱动程序生成的 ID 来按创建日期排序。

## 3.`ObjectId`类

唯一标识符存储在一个`ObjectId`类中，该类提供了方便的方法来获取存储在 ID 中的数据，而无需手动解析。

例如，下面是我们如何获取 ID 的创建日期:

```java
Date creationDate = objectId.getDate();
```

同样，我们可以以秒为单位检索 ID 的时间戳:

```java
int timestamp = objectId.getTimestamp();
```

`ObjectId`类也提供了获取计数器、机器标识符或进程标识符的方法，但是它们都被否决了。

## 4.正在检索 ID

需要记住的主要事情是，在 MongoDB 中，客户机在将一个`Document`发送到集群之前生成它的惟一标识符。这与关系数据库中的序列相反。这使得检索这个 ID 非常容易。

### 4.1.驱动程序生成的 ID

生成`Document`的唯一 ID 的标准且简单的方法是让驱动程序来完成这项工作。当我们向一个`Collection`插入一个新的`Document`时，如果`Document`中没有`_id`字段，驱动程序会在向集群发送插入命令之前生成一个新的`ObjectId`。

我们将新的`Document`插入到您的集合中的代码可能如下所示:

```java
Document document = new Document();
document.put("name", "Shubham");
document.put("company", "Baeldung");
collection.insertOne(document);
```

我们可以看到，我们从来没有指出必须如何生成 ID。

当`insertOne()`方法返回时，我们可以从`Document`中得到生成的`ObjectId`:

```java
ObjectId objectId = document.getObjectId("_id");
```

我们也可以像检索`Document`的标准字段一样检索`ObjectId`，然后将其转换为`ObjectId`:

```java
ObjectId oId = (ObjectId) document.get("_id");
```

### 4.2.自定义 ID

**检索 ID 的另一种方法是在我们的代码中生成 ID，并像其他字段一样将其放入`Document`中。**如果我们发送一个带有`_id`字段的`Document`给驱动程序，它不会生成一个新的。

在将`Document`插入`Collection`之前，我们可能需要 MongoDB `Document`的 ID。

我们可以通过创建该类的一个新实例来生成一个新的`ObjectId`:

```java
ObjectId generatedId = new ObjectId();
```

或者，我们也可以调用`ObjectId`类的静态`get()`方法:

```java
ObjectId generatedId = ObjectId.get();
```

然后，我们只需创建我们的`Document`并使用生成的 ID。为此，我们可以在`Document`构造函数中提供它:

```java
Document document = new Document("_id", generatedId); 
```

或者，我们可以使用`put()`方法:

```java
document.put("_id", generatedId);
```

当使用用户生成的 ID 时，我们必须小心在每次插入前生成一个新的`ObjectId`，因为重复的 ID 是禁止的。重复的 id 将导致`MongoWriteException`带有重复的关键消息。

`ObjectId`类提供了其他几个构造函数，允许我们设置标识符的某些部分:

```java
public ObjectId(final Date date)
public ObjectId(final Date date, final int counter)
public ObjectId(final int timestamp, final int counter)
public ObjectId(final String hexString)
public ObjectId(final byte[] bytes)
public ObjectId(final ByteBuffer buffer)
```

但是，当我们使用这些构造函数时，我们应该非常小心，因为提供给驱动程序的 ID 的唯一性完全依赖于我们的代码。在这些特殊情况下，我们可能会遇到重复键错误:

*   如果我们多次使用同一个日期(或时间戳)和计数器组合
*   如果我们多次使用同一个十六进制的`String`、`byte`数组或`ByteBuffer`

## 5.结论

在本文中，我们了解了什么是 MongoDB 文档唯一标识符以及它是如何工作的。然后，我们看到了如何在将一个`Document`插入到一个`Collection` 中之后，甚至在插入之前检索它。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524071051/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb)