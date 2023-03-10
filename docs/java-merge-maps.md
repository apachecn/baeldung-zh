# 用 Java 8 合并两个地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-merge-maps>

## 1。简介

在这个快速教程中，**我们将演示如何使用 Java 8 功能**合并两个地图。

更具体地说，我们将检查不同的合并场景，包括具有重复条目的地图。

## 2。初始化

首先，让我们定义两个`Map`实例:

```java
private static Map<String, Employee> map1 = new HashMap<>();
private static Map<String, Employee> map2 = new HashMap<>();
```

`Employee`类看起来像这样:

```java
public class Employee {

    private Long id;
    private String name;

    // constructor, getters, setters
}
```

然后，我们可以将一些数据推入`Map`实例:

```java
Employee employee1 = new Employee(1L, "Henry");
map1.put(employee1.getName(), employee1);
Employee employee2 = new Employee(22L, "Annie");
map1.put(employee2.getName(), employee2);
Employee employee3 = new Employee(8L, "John");
map1.put(employee3.getName(), employee3);

Employee employee4 = new Employee(2L, "George");
map2.put(employee4.getName(), employee4);
Employee employee5 = new Employee(3L, "Henry");
map2.put(employee5.getName(), employee5);
```

请注意，我们在地图中为`employee1`和`employee5`条目使用了相同的键，我们稍后会用到它们。

## 3。`Map.merge()`

**Java 8 在`java.util.Map`接口**中增加了一个新的`merge()`函数。

下面是`merge()`函数的工作方式:如果指定的键还没有与一个值相关联或者该值为空，它将该键与给定值相关联。

否则，它用给定的重新映射函数的结果替换该值。如果重新映射函数的结果为空，则删除该结果。

首先，让我们通过复制来自`map1`的所有条目来构造一个新的`HashMap`:

```java
Map<String, Employee> map3 = new HashMap<>(map1);
```

接下来，让我们介绍一下`merge()`函数以及合并规则:

```java
map3.merge(key, value, (v1, v2) -> new Employee(v1.getId(),v2.getName())
```

最后，我们将迭代`map2`并将条目合并到`map3`:

```java
map2.forEach(
  (key, value) -> map3.merge(key, value, (v1, v2) -> new Employee(v1.getId(),v2.getName())));
```

让我们运行程序并打印出`map3`的内容:

```java
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
George=Employee{id=2, name='George'}
Henry=Employee{id=1, name='Henry'}
```

因此，**我们的组合`Map`拥有前面`HashMap`条目的所有元素。具有重复关键字的条目已被合并为一个条目**。

此外，我们注意到最后一个条目的`Employee`对象具有来自`map1`的`id`，并且值是从`map2`选取的。

这是因为我们在合并函数中定义的规则:

```java
(v1, v2) -> new Employee(v1.getId(), v2.getName())
```

## 4。`Stream.concat()`

Java 8 中的`Stream` API 也可以为我们的问题提供一个简单的解决方案。首先，**我们需要将我们的`Map`实例合并成一个`Stream`** 。这正是`Stream.concat()`操作的作用:

```java
Stream combined = Stream.concat(map1.entrySet().stream(), map2.entrySet().stream());
```

这里，我们将映射条目集作为参数传递。**接下来，我们需要将我们的结果收集到一个新的`Map`** 中。为此，我们可以使用`Collectors.toMap()`:

```java
Map<String, Employee> result = combined.collect(
  Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

因此，收集器将使用我们地图中现有的键和值。但是这个解决方案远非完美。一旦我们的收集器遇到具有重复键的条目，它就会抛出一个`IllegalStateException`。

为了解决这个问题，我们只需在收集器中添加第三个“合并”lambda 参数:

```java
(value1, value2) -> new Employee(value2.getId(), value1.getName())
```

它将在每次检测到重复键时使用 lambda 表达式。

最后，将所有这些放在一起:

```java
Map<String, Employee> result = Stream.concat(map1.entrySet().stream(), map2.entrySet().stream())
  .collect(Collectors.toMap(
    Map.Entry::getKey, 
    Map.Entry::getValue,
    (value1, value2) -> new Employee(value2.getId(), value1.getName())));
```

最后，让我们运行代码，看看结果:

```java
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
Henry=Employee{id=3, name='Henry'}
```

正如我们所看到的，带有键`“Henry”`的重复条目被合并成一个新的键-值对，其中**的 id 是从`map2`中选取的，值是从`map1`T5 选取的。**

## `5\. Stream.of()`

为了继续使用`Stream` API，我们可以在`Stream.of()`的帮助下将我们的`Map`实例转换成一个统一的流。

在这里，我们不必创建额外的集合来处理流:

```java
Map<String, Employee> map3 = Stream.of(map1, map2)
  .flatMap(map -> map.entrySet().stream())
  .collect(Collectors.toMap(
    Map.Entry::getKey,
    Map.Entry::getValue,
    (v1, v2) -> new Employee(v1.getId(), v2.getName())));
```

首先，**我们将`map1`和`map2`转换成一个单独的流**。接下来，我们将流转换成地图。我们可以看到，`toMap()`的最后一个参数是一个合并函数。它通过从`v1`条目中选取 id 字段，从`v2`中选取名称来解决重复键问题。

运行程序后打印出的`map3`实例:

```java
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
Henry=Employee{id=1, name='Henry'}
```

## 6。简单流媒体

此外，我们可以使用一个`stream() `管道来组装我们的地图条目。下面的代码片段演示了如何通过忽略重复条目来添加来自`map2`和`map1`的条目:

```java
Map<String, Employee> map3 = map2.entrySet()
  .stream()
  .collect(Collectors.toMap(
    Map.Entry::getKey,
    Map.Entry::getValue,
    (v1, v2) -> new Employee(v1.getId(), v2.getName()),
  () -> new HashMap<>(map1)));
```

正如我们所料，合并后的结果是:

```java
{John=Employee{id=8, name='John'}, 
Annie=Employee{id=22, name='Annie'}, 
George=Employee{id=2, name='George'}, 
Henry=Employee{id=1, name='Henry'}}
```

## 7。`StreamEx`

除了 JDK 提供的解决方案，我们还可以使用流行的`StreamEx `库。

简单地说， **[`StreamEx`](/web/20220930004318/https://www.baeldung.com/streamex) 是对`Stream` API** 的增强，提供了许多额外的有用方法。我们将使用一个 **`EntryStream`实例来操作键值对**:

```java
Map<String, Employee> map3 = EntryStream.of(map1)
  .append(EntryStream.of(map2))
  .toMap((e1, e2) -> e1);
```

这个想法是将我们的地图流合并成一个。然后我们将条目收集到新的`map3`实例中。值得一提的是，`(e1, e2) -> e1`表达式有助于定义处理重复键的规则。没有它，我们的代码将抛出一个`IllegalStateException`。

现在，结果是:

```java
{George=Employee{id=2, name='George'}, 
John=Employee{id=8, name='John'}, 
Annie=Employee{id=22, name='Annie'}, 
Henry=Employee{id=1, name='Henry'}}
```

## 8。总结

在这篇短文中，我们学习了在 Java 8 中合并地图的不同方法。更确切地说，**我们使用了`Map.merge(), Stream API, StreamEx`库**。

和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220930004318/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)