# 用 Java 流对数字求和

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-sum>

## 1.介绍

在这个快速教程中，我们将研究使用[流 API](/web/20221009225805/https://www.baeldung.com/java-8-streams-introduction) 计算整数 **之和的各种方法。**

为了简单起见，我们将在例子中使用整数；然而，我们也可以将相同的方法应用于 longs 和 doubles。

## 延伸阅读:

## [Java 8 流简介](/web/20221009225805/https://www.baeldung.com/java-8-streams-introduction)

A quick and practical introduction to Java 8 Streams.[Read more](/web/20221009225805/https://www.baeldung.com/java-8-streams-introduction) →

## [stream . reduce()指南](/web/20221009225805/https://www.baeldung.com/java-stream-reduce)

Learn the key concepts of the Stream.reduce() operation in Java and how to use it to process sequential and parallel streams.[Read more](/web/20221009225805/https://www.baeldung.com/java-stream-reduce) →

## [Java 8 的收集器指南](/web/20221009225805/https://www.baeldung.com/java-8-collectors)

The article discusses Java 8 Collectors, showing examples of built-in collectors, as well as showing how to build custom collector.[Read more](/web/20221009225805/https://www.baeldung.com/java-8-collectors) →

## 2.使用`Stream.reduce()`

**`Stream.reduce()`是** **对**流的元素进行约简的终结操作。

它将二元运算符(累加器)应用于流中的每个元素，其中第一个操作数是上一个应用程序的返回值，第二个是当前流元素。

在使用`reduce()`方法的第一种方法中，累加器函数是一个 lambda 表达式，它将两个`Integer`值相加并返回一个`Integer`值:

```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
Integer sum = integers.stream()
  .reduce(0, (a, b) -> a + b);
```

同样，我们可以使用一个已经存在的 Java 方法:

```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
Integer sum = integers.stream()
  .reduce(0, Integer::sum);
```

或者我们可以定义并使用我们的自定义方法:

```
public class ArithmeticUtils {

    public static int add(int a, int b) {
        return a + b;
    }
} 
```

然后我们可以将这个函数作为参数传递给`reduce()`方法:

```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
Integer sum = integers.stream()
  .reduce(0, ArithmeticUtils::add); 
```

## 3.使用`Stream.collect()`

计算整数列表总和的第二种方法是使用`[collect()](/web/20221009225805/https://www.baeldung.com/java-8-collectors)`终端操作:

```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
Integer sum = integers.stream()
  .collect(Collectors.summingInt(Integer::intValue));
```

类似地，`Collectors`类提供了`summingLong()`和`summingDouble()`方法来分别计算 longs 和 doubles 的和。

## 4.使用`IntStream.sum()`

Stream API 为我们提供了 **`mapToInt()`** 中间操作，将我们的流转换成一个 **`IntStream` 对象**。

这个方法将一个映射器作为参数，用来进行转换，然后我们可以调用`sum()`方法来计算流元素的总和。

让我们看一个如何使用它的快速示例:

```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
Integer sum = integers.stream()
  .mapToInt(Integer::intValue)
  .sum();
```

同样，我们可以使用`mapToLong()`和`mapToDouble()`方法分别计算 longs 和 doubles 的和。

## 5.使用`Stream#sum`和`Map`

为了计算一个`Map<Object, Integer>`数据结构的值的总和，首先我们从那个`Map.` 的值创建一个**流，接下来我们应用我们之前使用的方法之一。**

例如，通过使用`IntStream.sum()`:

```
Integer sum = map.values()
  .stream()
  .mapToInt(Integer::valueOf)
  .sum();
```

## 6.对对象使用`Stream#sum`

假设我们有一个对象列表，我们想要计算这些对象的给定字段的所有值的总和**。**

例如:

```
public class Item {

    private int id;
    private Integer price;

    public Item(int id, Integer price) {
        this.id = id;
        this.price = price;
    }

    // Standard getters and setters
} 
```

接下来，假设我们要计算以下列表中所有商品的总价:

```
Item item1 = new Item(1, 10);
Item item2 = new Item(2, 15);
Item item3 = new Item(3, 25);
Item item4 = new Item(4, 40);

List<Item> items = Arrays.asList(item1, item2, item3, item4);
```

在这种情况下，为了使用前面示例中显示的方法计算总和，我们需要调用`map()`方法**将我们的流转换成整数流**。

因此，我们可以用`Stream.reduce(), ` `Stream.collect(), and ` `IntStream.sum()`来计算总和:

```
Integer sum = items.stream()
  .map(x -> x.getPrice())
  .reduce(0, ArithmeticUtils::add); 
```

```
Integer sum = items.stream()
  .map(x -> x.getPrice())
  .reduce(0, Integer::sum);
```

```
Integer sum = items.stream()
  .map(item -> item.getPrice())
  .reduce(0, (a, b) -> a + b);
```

```
Integer sum = items.stream()
  .map(x -> x.getPrice())
  .collect(Collectors.summingInt(Integer::intValue));
```

```
items.stream()
  .mapToInt(x -> x.getPrice())
  .sum();
```

## 7.使用`Stream#sum`和`String`

假设我们有一个包含一些整数的`String`对象。

为了计算这些整数的和，首先我们需要**将那个`String`转换成一个`Array.`** ，接下来我们需要**过滤掉非整数元素，**，最后，**将该数组的剩余元素**转换成数字。

让我们看看所有这些步骤的实际操作:

```
String string = "Item1 10 Item2 25 Item3 30 Item4 45";

Integer sum = Arrays.stream(string.split(" "))
    .filter((s) -> s.matches("\\d+"))
    .mapToInt(Integer::valueOf)
    .sum();
```

## 8.结论

在本文中，我们讨论了使用 Stream API 计算整数列表总和的几种方法。我们还使用这些方法来计算对象列表的给定字段的值的总和、地图的值的总和以及给定的`String`对象中的数字。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221009225805/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)