# Java 8 流简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-streams-introduction>

## 1。概述

在本文中，我们将快速浏览一下 Java 8 新增的主要功能之一——流。

我们将解释什么是流，并用简单的例子展示创建和基本的流操作。

## 2。流 API

Java 8 中一个主要的新特性是引入了流功能——[`java.util.stream`](https://web.archive.org/web/20220930004318/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/package-summary.html)——它包含了处理元素序列的类。

核心 API 类是`[Stream<T>](https://web.archive.org/web/20220930004318/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html).` ,下一节将演示如何使用现有的数据提供者源创建流。

### 2.1。流创建

借助`stream()`和`of()` 方法，可以从不同的元素源(例如集合或数组)创建流:

```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
stream = Stream.of("a", "b", "c");
```

一个`stream()`默认方法被添加到`Collection`接口中，并允许使用任何集合作为元素源`:`来创建一个`Stream<T>`

```java
Stream<String> stream = list.stream(); 
```

### 2.2。多线程与流

Stream API 还通过提供以并行模式运行 Stream 元素操作的`parallelStream()`方法来简化多线程。

下面的代码允许为流中的每个元素并行运行方法`doWork()`:

```java
list.parallelStream().forEach(element -> doWork(element));
```

在下一节中，我们将介绍一些基本的流 API 操作。

## 3。流操作

有许多有用的操作可以在流上执行。

分为**中间操作**(返回`Stream<T>`)和**终端操作**(返回一个确定类型的结果)。中间操作允许链接。

同样值得注意的是，对流的操作不会改变源代码。

这里有一个简单的例子:

```java
long count = list.stream().distinct().count();
```

因此，`distinct()`方法代表了一个中间操作，它创建了一个包含前一个流的独特元素的新流。而`count()` 方法是一个返回流大小的终端操作`,` 。

### 3.1。迭代

流 API 帮助替换`for`、`for-each`和`while`循环。它允许专注于操作的逻辑，而不是元素序列的迭代。例如:

```java
for (String string : list) {
    if (string.contains("a")) {
        return true;
    }
}
```

只需一行 Java 8 代码就可以修改这段代码:

```java
boolean isExist = list.stream().anyMatch(element -> element.contains("a"));
```

### 3.2。过滤

`filter()`方法允许我们挑选满足谓词的元素流。

例如，考虑以下列表:

```java
ArrayList<String> list = new ArrayList<>();
list.add("One");
list.add("OneAndOnly");
list.add("Derek");
list.add("Change");
list.add("factory");
list.add("justBefore");
list.add("Italy");
list.add("Italy");
list.add("Thursday");
list.add("");
list.add("");
```

下面的代码创建了一个`List<String>`的`Stream<String>`，找到了这个流中包含`char “d”`的所有元素，并创建了一个只包含被过滤元素的新流:

```java
Stream<String> stream = list.stream().filter(element -> element.contains("d"));
```

### 3.3。映射

通过应用一个特殊的函数来转换一个`Stream`的元素，并将这些新元素收集到一个`Stream`中，我们可以使用`map()`方法:

```java
List<String> uris = new ArrayList<>();
uris.add("C:\\My.txt");
Stream<Path> stream = uris.stream().map(uri -> Paths.get(uri));
```

因此，上面的代码通过对初始`Stream`的每个元素应用特定的 lambda 表达式，将`Stream<String>`转换为`Stream<Path>`。

如果您有一个流，其中每个元素都包含自己的元素序列，并且您想要创建这些内部元素的流，那么您应该使用`flatMap()`方法:

```java
List<Detail> details = new ArrayList<>();
details.add(new Detail());
Stream<String> stream
  = details.stream().flatMap(detail -> detail.getParts().stream());
```

在这个例子中，我们有一个类型为`Detail`的元素列表。`Detail`类包含一个字段`PARTS`，它是一个`List<String>`。在`flatMap()`方法的帮助下，来自字段`PARTS`的每个元素都将被提取并添加到新的结果流中。之后，初始的`Stream<Detail>`将会丢失`.`

### 3.4。匹配

Stream API 提供了一套方便的工具来根据一些谓词验证序列的元素。为此，可以使用以下方法之一:`anyMatch(), allMatch(), noneMatch().` 它们的名称不言自明。那些是返回一个`boolean`的终端操作:

```java
boolean isValid = list.stream().anyMatch(element -> element.contains("h")); // true
boolean isValidOne = list.stream().allMatch(element -> element.contains("h")); // false
boolean isValidTwo = list.stream().noneMatch(element -> element.contains("h")); // false
```

对于空流，带有任何给定谓词的`allMatch()`方法将返回`true`:

```java
Stream.empty().allMatch(Objects::nonNull); // true
```

这是一个合理的默认，因为我们找不到任何不满足谓词的元素。

类似地，`anyMatch()`方法总是为空流返回`false`:

```java
Stream.empty().anyMatch(Objects::nonNull); // false
```

这也是合理的，因为我们找不到满足这个条件的元素。

### 3.5。减少

Stream API 允许在类型`Stream`的`reduce()` 方法的帮助下，根据指定的函数将元素序列减少到某个值。这个方法有两个参数:第一个是起始值，第二个是累加器函数。

假设您有一个`List<Integer>`，并且您想要所有这些元素和一些初始的`Integer` (在这个例子中是 23)。因此，您可以运行下面的代码，结果将是 26 (23 + 1 + 1 + 1)。

```java
List<Integer> integers = Arrays.asList(1, 1, 1);
Integer reduced = integers.stream().reduce(23, (a, b) -> a + b);
```

### 3.6。收集

这种减少也可以通过类型为`Stream.`的`collect()`方法来提供。这种操作在将流转换为`Collection`或`Map`并以单个字符串的形式表示流的情况下非常方便`.` 有一个实用程序类`Collectors`，它为几乎所有典型的收集操作提供了解决方案。对于一些重要的任务，可以创建自定义的`Collector` 。

```java
List<String> resultList 
  = list.stream().map(element -> element.toUpperCase()).collect(Collectors.toList());
```

这段代码使用终端`collect()`操作将一个`Stream<String>` 减少到`List<String>.`

## 4。结论

在本文中，我们简要介绍了 Java 流——这无疑是 Java 8 最有趣的特性之一。

还有许多使用流的更高级的例子；这篇文章的目的仅仅是提供一个快速而实用的介绍，介绍您可以开始使用该功能做什么，并作为探索和进一步学习的起点。

本文附带的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220930004318/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)