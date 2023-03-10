# 具有历史记录映射的键值存储

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-chronicle-map>

## 1。概述

在本教程中，我们将看到如何使用[历史映射](https://web.archive.org/web/20221206104322/https://github.com/OpenHFT/Chronicle-Map/blob/master/docs/CM_Tutorial.adoc)来存储键值对。我们还将创建简短的示例来演示它的行为和用法。

## 2。什么是编年史地图？

根据文档，**“Chronicle Map 是一个超快速、内存中、非阻塞、键值存储，专为低延迟和/或多进程应用程序而设计”。**

简而言之，它是一个堆外键值存储。地图不需要大量的内存就能正常工作。**它可以根据可用磁盘容量**增长。此外，它支持多主服务器设置中的数据复制。

现在让我们看看如何设置和使用它。

## 3。Maven 依赖关系

首先，我们需要将[历史记录-地图依赖关系](https://web.archive.org/web/20221206104322/https://search.maven.org/search?q=g:net.openhft%20AND%20a:chronicle-map)添加到我们的项目中:

```java
<dependency>
    <groupId>net.openhft</groupId>
    <artifactId>chronicle-map</artifactId>
    <version>3.17.2</version>
</dependency>
```

## 4。编年史地图的类型

我们可以用两种方式创建映射:要么作为内存中的映射，要么作为持久化的映射。

让我们详细看看这两种情况。

### 4.1。内存映射

内存中的历史记录映射是在服务器的物理内存中创建的映射存储。这意味着**它只能在创建映射存储的 JVM 进程中访问**。

让我们看一个简单的例子:

```java
ChronicleMap<LongValue, CharSequence> inMemoryCountryMap = ChronicleMap
  .of(LongValue.class, CharSequence.class)
  .name("country-map")
  .entries(50)
  .averageValue("America")
  .create();
```

为了简单起见，我们创建了一个存储 50 个国家 id 及其名称的地图。正如我们在代码片段中看到的，除了`averageValue()`配置之外，创建非常简单。这告诉 map 配置 map 条目值所占用的平均字节数。

换句话说，**在创建映射时，历史映射决定了值的序列化形式所占用的平均字节数。它通过使用配置的值封送拆收器序列化给定的平均值来实现这一点。然后，它将为每个映射条目的值分配确定数量的字节。**

谈到内存映射，我们必须注意的一点是，只有当 JVM 进程处于活动状态时，数据才是可访问的。当进程终止时，库将清除数据。

### 4.2。持久化映射

与内存中的映射不同，**实现将持久化的映射保存到磁盘**。现在让我们看看如何创建持久化地图:

```java
ChronicleMap<LongValue, CharSequence> persistedCountryMap = ChronicleMap
  .of(LongValue.class, CharSequence.class)
  .name("country-map")
  .entries(50)
  .averageValue("America")
  .createPersistedTo(new File(System.getProperty("user.home") + "/country-details.dat"));
```

这将在指定的文件夹中创建一个名为`country-details.dat`的文件。如果这个文件在指定的路径中已经可用，那么构建器实现将从这个 JVM 进程打开一个到现有数据存储的链接。

我们可以在需要的情况下使用持久化映射:

*   超越创造者过程而生存；例如，为了支持热应用程序重新部署
*   使其在服务器中全局化；例如，为了支持多个并发进程访问
*   充当我们将保存到磁盘的数据存储

## 5。尺寸配置

在创建历史映射时，必须配置平均值和平均键，除非我们的键/值类型是装箱原语或值接口。在我们的例子中，我们没有配置平均键，因为键类型`LongValue`是一个[值接口](https://web.archive.org/web/20221206104322/https://github.com/OpenHFT/Chronicle-Values)。

现在，让我们看看配置键/值字节平均数的选项有哪些:

*   `averageValue()`–用于确定分配给映射条目值的平均字节数的值
*   `averageValueSize()`–分配给映射条目值的平均字节数
*   `constantValueSizeBySample()`–当值的大小始终相同时，为映射条目的值分配的字节数
*   `averageKey()`–确定分配给映射条目关键字的平均字节数的关键字
*   `averageKeySize()`–分配给映射条目关键字的平均字节数
*   `constantKeySizeBySample()`–当键的大小始终相同时，分配给映射条目的键的字节数

## 6。键和值类型

在创建历史图时，尤其是在定义键和值时，我们需要遵循某些标准。当我们使用推荐的类型创建键和值时，映射工作得最好。

以下是一些推荐的类型:

*   `Value`接口
*   从[历史记录字节](https://web.archive.org/web/20221206104322/https://github.com/OpenHFT/Chronicle-Bytes)实现`Byteable`接口的任何类
*   从历史记录字节实现`BytesMarshallable`接口的任何类；实现类应该有一个公共的无参数构造函数
*   `byte[]`和`ByteBuffer`
*   `CharSequence`、`String`和`StringBuilder`
*   `Integer`、`Long`和`Double`
*   任何实现`java.io.Externalizable`的类；实现类应该有一个公共的无参数构造函数
*   任何实现`java.io.Serializable`的类型，包括装箱的原始类型(上面列出的除外)和数组类型
*   任何其他类型(如果提供了自定义序列化程序)

## 7 .**。查询历史地图**

历史记录映射支持单键查询和多键查询。

### 7.1。单键查询

单键查询是处理单个键的操作。`ChronicleMap`支持来自 Java `Map`接口和`ConcurrentMap`接口的所有操作；

```java
LongValue qatarKey = Values.newHeapInstance(LongValue.class);
qatarKey.setValue(1);
inMemoryCountryMap.put(qatarKey, "Qatar");

//...

CharSequence country = inMemoryCountryMap.get(key);
```

除了正常的 get 和 put 操作， **`ChronicleMap`增加了一个特殊的操作，`getUsing(),`在检索和处理条目**时减少内存占用。让我们来看看实际情况:

```java
LongValue key = Values.newHeapInstance(LongValue.class);
StringBuilder country = new StringBuilder();
key.setValue(1);
persistedCountryMap.getUsing(key, country);
assertThat(country.toString(), is(equalTo("Romania")));

key.setValue(2);
persistedCountryMap.getUsing(key, country);
assertThat(country.toString(), is(equalTo("India")));
```

这里我们使用了同一个`StringBuilder`对象，通过将它传递给`getUsing()`方法来检索不同键的值。它基本上重用同一个对象来检索不同的条目。在我们的例子中，`getUsing()`方法相当于:

```java
country.setLength(0);
country.append(persistedCountryMap.get(key));
```

### 7.2。多键查询

可能会有我们需要同时处理多个键的用例。为此，我们可以使用`queryContext()`功能。**`queryContext()`方法将创建一个使用地图条目的上下文。**

让我们首先创建一个 multimap 并给它添加一些值:

```java
Set<Integer> averageValue = IntStream.of(1, 2).boxed().collect(Collectors.toSet());
ChronicleMap<Integer, Set<Integer>> multiMap = ChronicleMap
  .of(Integer.class, (Class<Set<Integer>>) (Class) Set.class)
  .name("multi-map")
  .entries(50)
  .averageValue(averageValue)
  .create();

Set<Integer> set1 = new HashSet<>();
set1.add(1);
set1.add(2);
multiMap.put(1, set1);

Set<Integer> set2 = new HashSet<>();
set2.add(3);
multiMap.put(2, set2);
```

**为了处理多个条目，我们必须锁定这些条目，以防止由于并发更新而可能出现的不一致:**

```java
try (ExternalMapQueryContext<Integer, Set<Integer>, ?> fistContext = multiMap.queryContext(1)) {
    try (ExternalMapQueryContext<Integer, Set<Integer>, ?> secondContext = multiMap.queryContext(2)) {
        fistContext.updateLock().lock();
        secondContext.updateLock().lock();

        MapEntry<Integer, Set<Integer>> firstEntry = fistContext.entry();
        Set<Integer> firstSet = firstEntry.value().get();
        firstSet.remove(2);

        MapEntry<Integer, Set<Integer>> secondEntry = secondContext.entry();
        Set<Integer> secondSet = secondEntry.value().get();
        secondSet.add(4);

        firstEntry.doReplaceValue(fistContext.wrapValueAsData(firstSet));
        secondEntry.doReplaceValue(secondContext.wrapValueAsData(secondSet));
    }
} finally {
    assertThat(multiMap.get(1).size(), is(equalTo(1)));
    assertThat(multiMap.get(2).size(), is(equalTo(2)));
}
```

## 8。关闭历史地图

现在我们已经完成了对地图的处理，让我们对地图对象调用`close()`方法来释放堆外内存和与之相关的资源:

```java
persistedCountryMap.close();
inMemoryCountryMap.close();
multiMap.close();
```

这里需要记住的一点是，所有的地图操作必须在关闭地图之前完成。否则，JVM 可能会意外崩溃。

## 9。结论

在本教程中，我们学习了如何使用历史映射来存储和检索键值对。尽管社区版提供了大部分核心功能，商业版也有一些高级特性，比如跨多台服务器的数据复制和远程调用。

我们在这里讨论的所有例子都可以在 Github 项目中找到。