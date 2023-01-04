# Java IntStream 转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-intstream-convert>

## 1.介绍

在这个快速教程中，我们将讨论**转换成其他类型**的所有可能性。

推荐关于[装箱和拆箱](/web/20221205122221/https://www.baeldung.com/java-8-primitive-streams)或者[迭代](/web/20221205122221/https://www.baeldung.com/java-stream-indices)的有趣读物，作为本教程的补充。

## 2.`IntStream`至`Array`

让我们开始探索如何将**从一个`IntStream`对象转换为一个`int` s** 的数组。

对于这个例子，让我们生成前 50 个偶数，并将它们存储在一个数组中，结果是:

```java
@Test
public void intStreamToArray() {
  int[] first50EvenNumbers = IntStream.iterate(0, i -> i + 2)
    .limit(50)
    .toArray();

  assertThat(first50EvenNumbers).hasSize(50);
  assertThat(first50EvenNumbers[2]).isEqualTo(4);
}
```

首先，让我们创建一个无限整数流，从 0 开始，通过给每个元素加 2 进行迭代。紧接着，我们需要添加一个中间操作`limit`，以便以某种方式终止这个操作。

最后，让我们使用终止操作`collect`将这个`Stream`聚集到一个数组中。

这是生成一个`int` s ``.`` 数组的直接方法

## 3.`IntStream`至`List`

让我们把现在的**一个`IntStream`转换成`Integers`** 的一个`List`。

在这种情况下，为了增加例子的多样性，我们使用方法`range`代替方法`iterate`。该方法将产生一个从`int` 0 到`int` 50 的`IntStream`(因为是开放范围，所以不包括在内):

```java
@Test
public void intStreamToList() {
  List<Integer> first50IntegerNumbers = IntStream.range(0, 50)
    .boxed()
    .collect(Collectors.toList());

  assertThat(first50IntegerNumbers).hasSize(50);
  assertThat(first50IntegerNumbers.get(2)).isEqualTo(2);
}
```

在这个例子中，我们使用了方法`range`。这里最臭名昭著的部分是使用**方法`boxed`，正如它的名字所指出的，它将装箱`IntStream`中的所有`int`元素，并返回一个`Stream<Integer>`** 。

最后，我们可以使用一个收集器来获取一个`integer`列表。

## 4.`IntStream`至`String`

对于我们的最后一个主题，让我们来探索如何从一个`IntStream` 中得到**一个`String`。**

在这种情况下，我们将只生成前 3 个`int`(0、1 和 2):

```java
@Test
public void intStreamToString() {
  String first3numbers = IntStream.of(0, 1, 2)
    .mapToObj(String::valueOf)
    .collect(Collectors.joining(", ", "[", "]"));

  assertThat(first3numbers).isEqualTo("[0, 1, 2]");
}
```

首先，在这种情况下，我们用构造函数`IntStream.of()`构造一个`IntStream`。有了`Stream`之后，我们需要以某种方式**从一个`IntStream`生成一个`Stream<String>`。因此，我们**可以使用中间的`mapToObj`方法**，该方法将接受一个`IntStream`并将返回一个`Stream`，该类型是在被调用的方法中映射的结果对象的类型。**

最后，我们使用收集器`joining`,它接受一个`Stream<String>`,并可以通过使用一个分隔符以及可选的前缀和后缀来附加`Stream`的每个元素。

## 5.结论

在这个快速教程中，当我们需要将一个`IntStream`转换成任何其他类型时，我们已经探索了所有的替代方法。特别是，我们通过例子生成了一个数组、`List`和`String`。

和往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221205122221/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)