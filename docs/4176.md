# 大队列介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-big-queue>

## 1.概观

在本教程中，我们将快速浏览一下[大队列](https://web.archive.org/web/20221126215513/https://github.com/bulldog2011/bigqueue)，一个持久[队列](/web/20221126215513/https://www.baeldung.com/java-queue)的 Java 实现。

我们将谈一点它的架构，然后我们将通过快速和实际的例子来学习如何使用它。

## 2.使用

我们需要将`bigqueue`依赖项添加到我们的项目中:

```
<dependency>
    <groupId>com.leansoft</groupId>
    <artifactId>bigqueue</artifactId>
    <version>0.7.0</version>
</dependency>
```

我们还需要添加它的存储库:

```
<repository>
    <id>github.release.repo</id>
    <url>https://raw.github.com/bulldog2011/bulldog-repo/master/repo/releases/</url>
</repository>
```

如果我们习惯于使用基本队列，那么适应大队列将是轻而易举的事情，因为它的 API 非常相似。

### 2.1.初始化

我们可以通过简单地调用队列的构造函数来初始化队列:

```
@Before
public void setup() {
    String queueDir = System.getProperty("user.home");
    String queueName = "baeldung-queue";
    bigQueue = new BigQueueImpl(queueDir, queueName);
}
```

第一个参数是我们队列的主目录。

第二个参数代表我们队列的名称。它将在我们队列的主目录中创建一个文件夹，我们可以在其中保存数据。

**我们应该记得在完成后关闭队列，以防止内存泄漏:**

```
bigQueue.close();
```

### 2.2.插入

我们可以通过简单地调用`enqueue`方法向尾部添加元素:

```
@Test
public void whenAddingRecords_ThenTheSizeIsCorrect() {
    for (int i = 1; i <= 100; i++) {
        bigQueue.enqueue(String.valueOf(i).getBytes());
    }

    assertEquals(100, bigQueue.size());
}
```

我们应该注意，大队列只支持`byte[]`数据类型，所以我们负责在插入时序列化我们的记录。

### 2.3.阅读

正如我们所料，使用`dequeue`方法读取数据也很简单:

```
@Test
public void whenAddingRecords_ThenTheyCanBeRetrieved() {
    bigQueue.enqueue(String.valueOf("new_record").getBytes());

    String record = new String(bigQueue.dequeue());

    assertEquals("new_record", record);
}
```

我们还必须小心地在读取时正确地反序列化我们的数据。

从空队列中读取会抛出一个`NullPointerException`。

我们应该使用`isEmpty`方法验证我们的队列中有值:

```
if(!bigQueue.isEmpty()){
    // read
}
```

**为了清空我们的队列而不必遍历每条记录，我们可以使用`removeAll`方法:**

```
bigQueue.removeAll();
```

### 2.4.偷看

当窥视时，我们只是简单地读取记录而不消耗它:

```
@Test
public void whenPeekingRecords_ThenSizeDoesntChange() {
    for (int i = 1; i <= 100; i++) {
        bigQueue.enqueue(String.valueOf(i).getBytes());
    }

    String firstRecord = new String(bigQueue.peek());

    assertEquals("1", firstRecord);
    assertEquals(100, bigQueue.size());
}
```

### 2.5.删除消耗的记录

当我们调用`dequeue`方法时，记录被从我们的队列中删除，但是它们仍然保存在磁盘上。

这可能会用不必要的数据填满我们的磁盘。

**幸运的是，我们可以使用`gc`方法删除已消费的记录:**

```
bigQueue.gc();
```

就像 Java 中的[垃圾收集器从堆中清除未被引用的对象一样，`gc`从我们的磁盘中清除已消耗的记录。](/web/20221126215513/https://www.baeldung.com/java-system-gc)

## 3.架构和功能

Big Queue 的有趣之处在于它的代码库非常小——只有 12 个源文件，占用了大约 20KB 的磁盘空间。

在高层次上，它只是一个擅长处理大量数据的持久队列。

### 3.1.处理大量数据

队列的大小只受我们总的可用磁盘空间的限制。为了防止崩溃，我们队列中的每条记录都保存在磁盘上。

我们的瓶颈将是磁盘 I/O，这意味着 SSD 将显著提高 HDD 的平均吞吐量。

### 3.2.极快地访问数据

如果我们看一下它的源代码，我们会注意到队列是由一个内存映射文件支持的。我们队列的可访问部分(头部)保存在 RAM 中，因此访问记录将非常快。

即使我们的队列会变得非常大，会占用太多的磁盘空间，我们仍然能够以 O(1)的时间复杂度读取数据。

如果我们需要读取大量消息，并且速度是一个关键问题，我们应该考虑使用 SSD 而不是 HDD，因为将数据从磁盘移动到内存会快得多。

### 3.3.优势

一个很大的优势是它能够变得非常大。我们可以通过添加更多存储将其扩展到理论上的无穷大，因此得名“大”。

在并发环境中，大队列可以在一台商用机器上产生和消耗大约 166MBps 的数据。

如果我们的平均消息大小为 1KB，它每秒可以处理 166k 条消息。

在单线程环境中，它可以达到每秒 333，000 条消息——非常令人印象深刻！

### 3.4.不足之处

我们的消息仍然保存在磁盘上，即使在我们使用它们之后，所以当我们不再需要时，我们必须处理垃圾收集数据。

我们还负责序列化和反序列化我们的消息。

## 4.结论

在这个快速教程中，我们学习了大队列以及如何将它用作可伸缩的持久队列。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20221126215513/https://github.com/eugenp/tutorials/tree/master/data-structures)