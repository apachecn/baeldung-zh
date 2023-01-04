# Java 并行收集器库指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-parallel-collectors>

## 1.介绍

[Parallel-collectors](https://web.archive.org/web/20220811164633/https://github.com/pivovarit/parallel-collectors) 是一个小型库，它提供了一组 Java 流 API 收集器，支持并行处理，同时避免了标准并行流的主要缺陷。

## 2.Maven 依赖性

如果我们想开始使用这个库，我们需要在 Maven 的`pom.xml`文件中添加一个条目:

```
<dependency>
    <groupId>com.pivovarit</groupId>
    <artifactId>parallel-collectors</artifactId>
    <version>1.1.0</version>
</dependency>
```

或者 Gradle 的构建文件中的一行:

```
compile 'com.pivovarit:parallel-collectors:1.1.0'
```

最新版本[可以在 Maven Central](https://web.archive.org/web/20220811164633/https://search.maven.org/search?q=g:com.pivovarit%20AND%20a:parallel-collectors&core=gav) 上找到。

## 3.平行流警告

并行流是 Java 8 的亮点之一，但事实证明它们只适用于繁重的 CPU 处理。

这样做的原因是，**并行流由 JVM 范围的共享`ForkJoinPool`在内部提供支持，这提供了有限的并行性**,并由运行在单个 JVM 实例上的所有并行流使用。

例如，假设我们有一个 id 列表，我们想用它们来获取一个用户列表，而这个操作代价很高。

为此，我们可以使用并行流:

```
List<Integer> ids = Arrays.asList(1, 2, 3); 
List<String> results = ids.parallelStream() 
  .map(i -> fetchById(i)) // each operation takes one second
  .collect(Collectors.toList()); 

System.out.println(results); // [user-1, user-2, user-3]
```

事实上，我们可以看到有明显的加速。但是，如果我们开始并行运行多个并行阻塞操作，就会出现问题。**这可能会很快使池**饱和，并导致潜在的巨大延迟。这就是为什么通过创建单独的线程池来构建舱壁很重要——以防止不相关的任务影响彼此的执行。

为了提供一个定制的`ForkJoinPool`实例，我们可以利用[这里描述的技巧](/web/20220811164633/https://www.baeldung.com/java-8-parallel-streams-custom-threadpool)，但是这种方法依赖于一个未记录的黑客，并且在 JDK10 之前是错误的。我们可以在问题本身中阅读更多内容-[【JDK 8190974】](https://web.archive.org/web/20220811164633/https://bugs.openjdk.java.net/browse/JDK-8190974)。

## 4.并行收集器在工作

**并行收集器，顾名思义，就是标准的流 API 收集器，允许在`collect()`阶段并行执行额外的操作。**

`ParallelCollectors`(镜像`Collectors`类)类是一个门面，提供对库的全部功能的访问。

如果我们想重做上面的例子，我们可以简单地写:

```
ExecutorService executor = Executors.newFixedThreadPool(10);

List<Integer> ids = Arrays.asList(1, 2, 3);

CompletableFuture<List<String>> results = ids.stream()
  .collect(ParallelCollectors.parallelToList(i -> fetchById(i), executor, 4));

System.out.println(results.join()); // [user-1, user-2, user-3]
```

然而，结果是相同的，**我们能够提供我们的自定义线程池，指定我们的自定义并行级别，并且结果被包装在一个 [`CompletableFuture`](/web/20220811164633/https://www.baeldung.com/java-completablefuture) 实例中到达，而不阻塞当前线程。**

另一方面，标准并行流不能实现这些。

## 4.1.`ParallelCollectors.parallelToList/ToSet()`

尽管很直观，但如果我们想并行处理一个`Stream`，并将结果收集到一个`List`或`Set`，我们可以简单地使用`ParallelCollectors.parallelToList`或`parallelToSet`:

```
List<Integer> ids = Arrays.asList(1, 2, 3);

List<String> results = ids.stream()
  .collect(parallelToList(i -> fetchById(i), executor, 4))
  .join();
```

## 4.2.`ParallelCollectors.parallelToMap()`

如果我们想要将`Stream`元素收集到一个`Map`实例中，就像使用 Stream API 一样，我们需要提供两个映射器:

```
List<Integer> ids = Arrays.asList(1, 2, 3);

Map<Integer, String> results = ids.stream()
  .collect(parallelToMap(i -> i, i -> fetchById(i), executor, 4))
  .join(); // {1=user-1, 2=user-2, 3=user-3}
```

我们还可以提供一个定制的`Map`实例`Supplier`:

```
Map<Integer, String> results = ids.stream()
  .collect(parallelToMap(i -> i, i -> fetchById(i), TreeMap::new, executor, 4))
  .join(); 
```

和自定义冲突解决策略:

```
List<Integer> ids = Arrays.asList(1, 2, 3);

Map<Integer, String> results = ids.stream()
  .collect(parallelToMap(i -> i, i -> fetchById(i), TreeMap::new, (s1, s2) -> s1, executor, 4))
  .join();
```

## 4.3.`ParallelCollectors.parallelToCollection()`

与上面类似，如果我们想要获得打包在自定义容器中的结果，我们可以传递我们的自定义`Collection Supplier`:

```
List<String> results = ids.stream()
  .collect(parallelToCollection(i -> fetchById(i), LinkedList::new, executor, 4))
  .join();
```

## 4.4.`ParallelCollectors.parallelToStream()`

如果以上还不够，我们实际上可以获得一个`Stream`实例，并在那里继续定制处理:

```
Map<Integer, List<String>> results = ids.stream()
  .collect(parallelToStream(i -> fetchById(i), executor, 4))
  .thenApply(stream -> stream.collect(Collectors.groupingBy(i -> i.length())))
  .join();
```

## 4.5.`ParallelCollectors.parallel()`

这允许我们以完成顺序流式传输结果:

```
ids.stream()
  .collect(parallel(i -> fetchByIdWithRandomDelay(i), executor, 4))
  .forEach(System.out::println);

// user-1
// user-3
// user-2 
```

在这种情况下，我们可以预期收集器每次都会返回不同的结果，因为我们引入了随机处理延迟。

## 4.6.`ParallelCollectors.parallelOrdered()`

该工具允许像上面一样流式传输结果，但保持原始顺序:

```
ids.stream()
  .collect(parallelOrdered(i -> fetchByIdWithRandomDelay(i), executor, 4))
  .forEach(System.out::println);

// user-1
// user-2 
// user-3 
```

在这种情况下，收集器将始终保持顺序，但可能比上面的慢。

## 5.限制

在撰写本文时，**并行收集器不能与无限流**一起工作，即使使用了短路操作——这是由流 API 内部强加的设计限制。简单地说，`Stream` s 将收集器视为非短路操作，因此流需要在终止之前处理所有上游元素。

另一个限制是**短路操作不会中断短路后剩余的任务**。

## 6.结论

我们看到了并行收集器库如何允许我们通过使用定制的 Java 流 API `Collectors`和`CompletableFutures`来执行并行处理，以利用定制的线程池、并行性和`CompletableFutures.`的非阻塞风格

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220811164633/https://github.com/eugenp/tutorials/tree/master/libraries-2)

如需进一步阅读，请参见 GitHub 上的[水货收藏库](https://web.archive.org/web/20220811164633/https://github.com/pivovarit/parallel-collectors)、[作者的博客](https://web.archive.org/web/20220811164633/http://4comprehension.com/)以及作者的[推特账户](https://web.archive.org/web/20220811164633/https://twitter.com/pivovarit)。