# Javatuples 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-tuples>

## 1。概述

元组是几个元素的集合，这些元素可能彼此相关，也可能彼此不相关。换句话说，元组可以被认为是匿名对象。

比如["RAM "，16，" Astra"]就是一个包含三个元素的元组。

在本文中，我们将快速浏览一个非常简单的库，它允许我们使用基于元组的数据结构，名为 [`javatuples`](https://web.archive.org/web/20220808152023/http://www.javatuples.org/index.html) 。

## 2。内置`Javatuples` 类

这个库为我们提供了十个不同的类，可以满足我们与元组相关的大部分需求:

*   `Unit<A>`
*   `Pair<A,B>`
*   `Triplet<A,B,C>`
*   `Quartet<A,B,C,D>`
*   `Quintet<A,B,C,D,E>`
*   `Sextet<A,B,C,D,E,F>`
*   `Septet<A,B,C,D,E,F,G>`
*   `Octet<A,B,C,D,E,F,G,H>`
*   `Ennead<A,B,C,D,E,F,G,H,I>`
*   `Decade<A,B,C,D,E,F,G,H,I,J>`

除了上面的类，还有两个额外的类，`KeyValue<A,B>`和`LabelValue<A,B>`，它们提供类似于`Pair<A,B>`的功能，但是语义不同。

根据[官方网站](https://web.archive.org/web/20220808152023/http://www.javatuples.org/index.html)，****Java tuples 中的所有类都是类型安全且不可变的**。每个元组类都实现了`Iterable`、`Serializable`和`Comparable`接口。**

 **## 3。添加 Maven 依赖关系

让我们将 Maven 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.javatuples</groupId>
    <artifactId>javatuples</artifactId>
    <version>1.2</version>
</dependency>
```

请检查中央 Maven 仓库中的最新版本。

## 4。创建元组

创建一个元组非常简单。我们可以使用相应的构造函数:

```java
Pair<String, Integer> pair = new Pair<String, Integer>("A pair", 55);
```

还有一种不太冗长且语义优雅的创建元组的方法:

```java
Triplet<String, Integer, Double> triplet = Triplet.with("hello", 23, 1.2);
```

我们也可以从一个`Iterable`创建元组:

```java
List<String> listOfNames = Arrays.asList("john", "doe", "anne", "alex");
Quartet<String, String, String, String> quartet
  = Quartet.fromCollection(collectionOfNames);
```

请注意**集合中的条目数量应该与我们想要创建的元组类型相匹配**。例如，我们不能使用上面的集合创建一个`Quintet`,因为它正好需要五个元素。对于比`Quintet`具有更高顺序的任何其他元组类也是如此。

然而，我们可以使用上面的集合，通过在`fromIterable()`方法中指定一个起始索引，创建一个像`Pair`或`Triplet`这样的低阶元组:

```java
Pair<String, String> pairFromList = Pair.fromIterable(listOfNames, 2);
```

上面的代码将导致创建一个包含“`anne`”和“`alex`”的`Pair`。

元组也可以方便地从任何数组中创建:

```java
String[] names = new String[] {"john", "doe", "anne"};
Triplet<String, String, String> triplet2 = Triplet.fromArray(names);
```

## 5.**从元组中获取值**

`javatuples`中的每个类都有一个从元组中获取值的`getValueX()`方法，其中`X`指定元素在元组中的顺序。像数组中的索引一样，`X`的值从零开始。

让我们创建一个新的四重奏并获取一些值:

```java
Quartet<String, Double, Integer, String> quartet 
  = Quartet.with("john", 72.5, 32, "1051 SW");

String name = quartet.getValue0();
Integer age = quartet.getValue2();

assertThat(name).isEqualTo("john");
assertThat(age).isEqualTo(32);
```

我们可以看到，“`john`”的位置是零，“`72.5`”的位置是一，以此类推。

注意，`getValueX()`方法是类型安全的。这意味着，不需要铸造。

另一种方法是`getValue(int pos)`方法。它获取要获取的元素的从零开始的位置。**这个方法不是类型安全的，需要显式强制转换**:

```java
Quartet<String, Double, Integer, String> quartet 
  = Quartet.with("john", 72.5, 32, "1051 SW");

String name = (String) quartet.getValue(0);
Integer age = (Integer) quartet.getValue(2);

assertThat(name).isEqualTo("john");
assertThat(age).isEqualTo(32);
```

请注意，类`KeyValue`和`LabelValue`有它们对应的方法`getKey()/getValue()`和`getLabel()/getValue()`。

## 6。将值设置为元组

与`getValueX()`类似，javatuples 中的所有类都有`setAtX()`方法。同样，`X`是我们要设置的元素的从零开始的位置:

```java
Pair<String, Integer> john = Pair.with("john", 32);
Pair<String, Integer> alex = john.setAt0("alex");

assertThat(john.toString()).isNotEqualTo(alex.toString());
```

这里重要的是`setAtX()`方法的返回类型是元组类型本身。这是因为**Java tuples 是不可变的**。设置任何新值都将保持原始实例不变。

## 7。从元组中添加和删除元素

我们可以方便地向元组添加新元素。然而，这将导致创建一个更高阶的新元组:

```java
Pair<String, Integer> pair1 = Pair.with("john", 32);
Triplet<String, Integer, String> triplet1 = pair1.add("1051 SW");

assertThat(triplet1.contains("john"));
assertThat(triplet1.contains(32));
assertThat(triplet1.contains("1051 SW"));
```

从上面的例子可以清楚地看出，向一个`Pair`添加一个元素将会创建一个新的`Triplet`。类似地，向一个`Triplet`添加一个元素将会创建一个新的`Quartet`。

上面的例子还演示了由`javatuples`中的所有类提供的`contains()`方法的使用。这是检验元组是否包含给定值的一种非常方便的方法。

也可以使用`add()`方法将一个元组添加到另一个元组:

```java
Pair<String, Integer> pair1 = Pair.with("john", 32);
Pair<String, Integer> pair2 = Pair.with("alex", 45);
Quartet<String, Integer, String, Integer> quartet2 = pair1.add(pair2);

assertThat(quartet2.containsAll(pair1));
assertThat(quartet2.containsAll(pair2));
```

注意`containsAll()`方法的使用。如果`pair1`的所有元素都出现在`quartet2`中，它将返回`true`。

默认情况下，`add()`方法添加元素作为元组的最后一个元素。然而，可以使用`addAtX()`方法在给定位置添加元素，其中`X`是我们想要添加元素的从零开始的位置:

```java
Pair<String, Integer> pair1 = Pair.with("john", 32);
Triplet<String, String, Integer> triplet2 = pair1.addAt1("1051 SW");

assertThat(triplet2.indexOf("john")).isEqualTo(0);
assertThat(triplet2.indexOf("1051 SW")).isEqualTo(1);
assertThat(triplet2.indexOf(32)).isEqualTo(2);
```

这个例子在位置 1 添加了`String`，然后通过`indexOf()`方法进行验证。请注意在调用`addAt1()`方法调用后`Pair<String, Integer>`和`Triplet<String, String, Integer>`类型顺序的不同。

我们还可以使用任何一种`add()`或`addAtX()`方法添加多个元素:

```java
Pair<String, Integer> pair1 = Pair.with("john", 32);
Quartet<String, Integer, String, Integer> quartet1 = pair1.add("alex", 45);

assertThat(quartet1.containsAll("alex", "john", 32, 45));
```

为了从元组中移除一个元素，我们可以使用`removeFromX()`方法。再次，`X`指定要移除的元素的从零开始的位置:

```java
Pair<String, Integer> pair1 = Pair.with("john", 32);
Unit<Integer> unit = pair1.removeFrom0();

assertThat(unit.contains(32));
```

## 8。将元组转换为`List/Array`

我们已经看到了如何将一个`List`转换成一个元组。现在让我们看看如何将一个元组转换成一个`List`:

```java
Quartet<String, Double, Integer, String> quartet
  = Quartet.with("john", 72.5, 32, "1051 SW");
List<Object> list = quartet.toList();

assertThat(list.size()).isEqualTo(4);
```

这相当简单。这里唯一需要注意的是，即使元组包含相同类型的元素，我们也总是会得到一个`List<Object>,`。

最后，让我们将元组转换为数组:

```java
Quartet<String, Double, Integer, String> quartet
  = Quartet.with("john", 72.5, 32, "1051 SW");
Object[] array = quartet.toArray();

assertThat(array.length).isEqualTo(4);
```

很清楚，`toArray()`方法总是返回一个`Object[]`。

## 9。结论

在本文中，我们探索了 javatuples 库并观察了它的简单性。它提供了优雅的语义，并且非常容易使用。

请务必在 GitHub 上查看本文[的完整源代码。完整的源代码包含的例子比这里介绍的要多一点。读完这篇文章后，附加的例子应该很容易理解。](https://web.archive.org/web/20220808152023/https://github.com/eugenp/tutorials/tree/master/libraries)**