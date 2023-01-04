# 将整数列表转换为字符串列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-list-integers-to-list-strings>

## 1.概观

从版本 5 开始，Java 就支持[泛型](/web/20221224002731/https://www.baeldung.com/java-generics)。【Java 泛型带给我们的一个好处就是类型安全。例如，当我们将一个`[List](/web/20221224002731/https://www.baeldung.com/tag/java-list)`对象`myList`声明为`List<Integer>`时，我们不能将一个类型不是`Integer`的元素放到`myList`中。

然而，当我们使用泛型集合时，我们经常想要将`Collection<TypeA>`转换成`Collection<TypeB>`。

在本教程中，我们将以`List<Integer>`为例，探讨如何将`List<Integer>`转换为`List<String>`。

## 2.准备一个`List<Integer>`对象作为示例

为了简单起见，我们将使用单元测试断言来验证我们的转换是否如预期的那样工作。因此，让我们先对整数列表进行初始化:

```
List<Integer> INTEGER_LIST = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
```

如上面的代码所示，我们在`INTEGER_LIST`对象中有七个整数。现在，我们的目标是**将`INTEGER_LIST`中的每个整数元素转换成一个** `**String**,` ，例如`1`转换成`“1”`，`2`转换成`“2”`，等等。最后，结果应该等于:

```
List<String> EXPECTED_LIST = Arrays.asList("1", "2", "3", "4", "5", "6", "7");
```

在本教程中，我们将介绍三种不同的方法:

*   使用 Java 8 的[流 API](/web/20221224002731/https://www.baeldung.com/java-8-streams)
*   使用 Java `for`循环
*   使用[番石榴](/web/20221224002731/https://www.baeldung.com/guava-guide)库

接下来，让我们看看他们的行动。

## 3.使用 Java 8 `Stream`的`map()`方法

Java Stream API 在 Java 8 和更高版本上可用。它提供了许多方便的接口，允许我们轻松地将`Collection`作为流来处理。

例如，**将`List<TypeA>`转换为`List<TypeB>`的一个典型方法是`Stream`的`map()`方法**:

```
theList.stream().map( .. the conversion logic.. ).collect(Collectors.toList());
```

那么接下来，让我们看看如何使用`map()`方法将`List<Integer>`转换为`List<String>`:

```
List<String> result = INTEGER_LIST.stream().map(i -> i.toString()).collect(Collectors.toList());
assertEquals(EXPECTED_LIST, result);
```

如上面的代码示例所示，我们将一个 [lambda 表达式](/web/20221224002731/https://www.baeldung.com/java-8-lambda-expressions-tips)传递给`map()`，调用每个元素(`Integer`)的`toString()`方法将其转换为`String`。

如果我们运行它，测试通过。因此，`Stream`的`map()`方法完成了这项工作。

## 4.使用`for`循环

我们已经看到`Stream`的`map()`方法可以解决问题。然而，正如我们提到的，Stream API 只在 Java 8 和更高版本中可用。因此，如果我们正在使用一个旧的 Java 版本，我们需要用另一种方法来解决这个问题。

例如，我们可以通过一个简单的`for`循环进行转换:

```
List<String> result = new ArrayList<>();
for (Integer i : INTEGER_LIST) {
    result.add(i.toString());
}

assertEquals(EXPECTED_LIST, result);
```

上面的代码显示**我们首先创建了一个新的`List<String>`对象，`result`。然后，我们在一个`for`循环中迭代`List<Integer>`列表中的元素，将每个`Integer`元素转换为`String,`，并将字符串添加到`result`列表中。**

如果我们试一试，测试就会通过。

## 5.使用番石榴图书馆

当我们使用集合时，转换集合的类型是一个非常标准的操作，一些流行的外部库提供了实用方法来完成转换。

在这一节中，我们将使用番石榴来说明如何解决我们的问题。

首先，让我们在`pom.xml`中添加 Guava 库依赖项:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

当然，我们可以在 Maven 中央存储库中检查最新的版本。

接下来，我们可以用芭乐的 [`Lists.transform()`](/web/20221224002731/https://www.baeldung.com/guava-filter-and-transform-a-collection#transform-a-collection) 方法来解决我们的问题:

```
List<String> result = Lists.transform(INTEGER_LIST, Functions.toStringFunction());
assertEquals(EXPECTED_LIST, result);
```

**`transform()`方法对`INTEGER_LIST`中的每个元素应用`toStringFunction()`，并返回转换后的列表。**

如果我们运行它，测试就通过了。

## 6.结论

在这篇短文中，我们学习了三种将`List<Integer>`转换成`List<String>`的方法。**如果我们的 Java 版本是 8+，Stream API 将是最直接的转换方法。**否则，我们可以通过循环应用转换，或者转向外部库，比如 Guava。