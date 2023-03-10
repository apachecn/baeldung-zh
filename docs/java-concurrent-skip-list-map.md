# ConcurrentSkipListMap 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrent-skip-list-map>

## 1。概述

在这篇简短的文章中，我们将看看`java.util.concurrent`包中的 [`ConcurrentSkipListMap`](https://web.archive.org/web/20221129003204/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentSkipListMap.html) 类。

这种结构允许我们以无锁的方式创建线程安全的逻辑。当其他线程仍在向 map 中插入数据时，如果我们想要制作数据的不可变快照，这是一个理想的解决方案。

我们将解决一个问题:**对事件流进行排序，并使用该构造**获得最近 60 秒内到达的事件的快照。

## 2。流分类逻辑

假设我们有一个持续来自多个线程的事件流。我们需要能够记录最近 60 秒的事件，以及超过 60 秒的事件。

首先，让我们定义事件数据的结构:

```java
public class Event {
    private ZonedDateTime eventTime;
    private String content;

    // standard constructors/getters
}
```

我们希望使用`eventTime` 字段对事件进行排序。为了使用`ConcurrentSkipListMap,`实现这一点，我们需要向其构造函数传递一个`Comparator`，同时创建它的一个实例:

```java
ConcurrentSkipListMap<ZonedDateTime, String> events
 = new ConcurrentSkipListMap<>(
 Comparator.comparingLong(v -> v.toInstant().toEpochMilli()));
```

我们将使用时间戳比较所有到达的事件。我们使用了`comparingLong()` 方法并传递了提取函数，该函数可以从`ZonedDateTime.` 中获取一个`long` 时间戳

当我们的事件到达时，我们只需要使用`put()` 方法将它们添加到地图中。请注意，此方法不需要任何显式同步:

```java
public void acceptEvent(Event event) {
    events.put(event.getEventTime(), event.getContent());
}
```

`ConcurrentSkipListMap` 将使用在构造函数中传递给它的`Comparator`来处理下面那些事件的排序。

`ConcurrentSkipListMap` 最显著的优点是能够以无锁的方式制作其数据的不可变快照的方法。要获取在过去一分钟内到达的所有事件，我们可以使用`tailMap()` 方法并传递我们想要获取元素的时间:

```java
public ConcurrentNavigableMap<ZonedDateTime, String> getEventsFromLastMinute() {
    return events.tailMap(ZonedDateTime.now().minusMinutes(1));
} 
```

它将返回过去一分钟的所有事件。这将是一个不可变的快照，最重要的是，其他写线程可以向`ConcurrentSkipListMap` 添加新事件，而无需进行显式锁定。

我们现在可以获得从现在起一分钟后到达的所有事件——通过使用`headMap()` 方法:

```java
public ConcurrentNavigableMap<ZonedDateTime, String> getEventsOlderThatOneMinute() {
    return events.headMap(ZonedDateTime.now().minusMinutes(1));
}
```

这将返回超过一分钟的所有事件的不可变快照。以上所有方法都属于`EventWindowSort` 类，我们将在下一节中使用。

## 3。测试分类流逻辑

一旦我们使用`ConcurrentSkipListMap,` 实现了我们的排序逻辑，我们现在可以通过创建两个写线程来测试它，这两个写线程将分别发送一百个事件:

```java
ExecutorService executorService = Executors.newFixedThreadPool(3);
EventWindowSort eventWindowSort = new EventWindowSort();
int numberOfThreads = 2;

Runnable producer = () -> IntStream
  .rangeClosed(0, 100)
  .forEach(index -> eventWindowSort.acceptEvent(
      new Event(ZonedDateTime.now().minusSeconds(index), UUID.randomUUID().toString()))
  );

for (int i = 0; i < numberOfThreads; i++) {
    executorService.execute(producer);
} 
```

每个线程都在调用`acceptEvent()` 方法，发送从现在到“现在减一百秒”有`eventTime`的事件。

同时，我们可以调用`getEventsFromLastMinute()` 方法，该方法将返回一分钟窗口内的事件快照:

```java
ConcurrentNavigableMap<ZonedDateTime, String> eventsFromLastMinute 
  = eventWindowSort.getEventsFromLastMinute();
```

根据生产者线程将事件发送到`EventWindowSort.` 的速度，`eventsFromLastMinute` 中的事件数量将在每次测试运行中变化。我们可以断言，返回的快照中没有一个事件超过一分钟:

```java
long eventsOlderThanOneMinute = eventsFromLastMinute
  .entrySet()
  .stream()
  .filter(e -> e.getKey().isBefore(ZonedDateTime.now().minusMinutes(1)))
  .count();

assertEquals(eventsOlderThanOneMinute, 0);
```

并且快照中在一分钟窗口内的事件多于零个:

```java
long eventYoungerThanOneMinute = eventsFromLastMinute
  .entrySet()
  .stream()
  .filter(e -> e.getKey().isAfter(ZonedDateTime.now().minusMinutes(1)))
  .count();

assertTrue(eventYoungerThanOneMinute > 0);
```

我们的`getEventsFromLastMinute()` 用下面的`tailMap()` 。

现在让我们测试使用来自`ConcurrentSkipListMap:`的`headMap()` 方法的`getEventsOlderThatOneMinute()`

```java
ConcurrentNavigableMap<ZonedDateTime, String> eventsFromLastMinute 
  = eventWindowSort.getEventsOlderThatOneMinute();
```

这一次，我们获得了一分钟之前的事件的快照。我们可以断言，这样的事件不仅仅是零:

```java
long eventsOlderThanOneMinute = eventsFromLastMinute
  .entrySet()
  .stream()
  .filter(e -> e.getKey().isBefore(ZonedDateTime.now().minusMinutes(1)))
  .count();

assertTrue(eventsOlderThanOneMinute > 0);
```

其次，没有一个事件是在最后一分钟发生的:

```java
long eventYoungerThanOneMinute = eventsFromLastMinute
  .entrySet()
  .stream()
  .filter(e -> e.getKey().isAfter(ZonedDateTime.now().minusMinutes(1)))
  .count();

assertEquals(eventYoungerThanOneMinute, 0);
```

需要注意的最重要的一点是**我们可以在其他线程仍在向`ConcurrentSkipListMap.`添加新值**时拍摄数据快照

## 4。结论

在这个快速教程中，我们看了一下`ConcurrentSkipListMap`的基础知识，以及一些实用的例子`.`

我们利用`ConcurrentSkipListMap` 的高性能实现了一个非阻塞算法，即使多个线程同时更新地图，它也能为我们提供一个不可变的数据快照。

所有这些例子和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221129003204/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)中找到；这是一个 Maven 项目，因此应该很容易导入和运行。