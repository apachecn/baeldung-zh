# Vavr 的集合工厂方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-collection-factory-methods>

## 1。概述

[`Vavr`](https://web.archive.org/web/20220625221852/http://www.vavr.io/vavr-docs/) 是一个强大的 Java 8+库，建立在 Java lambda 表达式之上。受 Scala 语言的启发， **`Vavr`在 Java 语言**中加入了函数式编程构造，比如模式匹配、控制结构、数据类型、持久和不可变集合等等。

在这篇短文中，我们将向**展示如何使用一些工厂方法来创建`Vavr`集合**。如果你是 Vavr 的新手，你可以从[这篇介绍性教程](/web/20220625221852/https://www.baeldung.com/vavr-tutorial)开始，它反过来也参考了其他有用的文章。

## 2。Maven 依赖关系

要将`Vavr`库添加到您的 Maven 项目中，请编辑您的`pom.xml`文件以包含以下依赖项:

```
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.9.1</version>
</dependency> 
```

你可以在 [Maven Central repository](https://web.archive.org/web/20220625221852/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.vavr%22%20AND%20a%3A%22vavr%22) 上找到这个库的最新版本。

## 3。静态工厂方法

使用静态导入:

```
static import io.vavr.API.*;
```

我们可以使用构造函数`List(…):`创建一个列表

```
List numbers = List(1,2,3);
```

代替使用静态工厂方法`of(…):`

```
List numbers = List.of(1,2,3);
```

或者还有:

```
Tuple t = Tuple('a', 3);
```

而不是:

```
Tuple t = Tuple.of('a', 3);
```

这种语法类似于 Scala/Kotlin 中的结构。从现在开始，我们将在文章中使用这些缩写。

## 4。创建`Option` 元素

元素不是集合，但是它们可以是 Vavr 库的非常有用的构造。这种类型的**允许我们持有一个对象或者一个`None`元素**(相当于一个`null`对象):

```
Option<Integer> none = None();
Option<Integer> some = Some(1);
```

## 5。`Vavr` `Tuples`

同样，Java 没有元组，像有序对、三元组等。在`Vavr`中，我们可以**定义一个元组，它可以容纳多达八个不同类型的对象**。这里有一个包含一个`Character`、一个`String`和一个`Integer`对象的例子:

```
Tuple3<Character, String, Integer> tuple
  = Tuple('a', "chain", 2); 
```

## 6。`Try`式

`Try`类型可用于**模型计算，该模型计算可能会也可能不会引发异常**:

```
Try<Integer> integer
  = Success(55);
Try<Integer> failure
  = Failure(new Exception("Exception X encapsulated here"));
```

在这种情况下，如果我们对`integer.get()`求值，我们将获得整数对象 55。如果我们评估`failure.get()`，将会抛出一个异常。

## 7。`Vavr`收藏

我们可以用许多不同的方式创建集合。对于`List` s，我们可以用`List.of(), List.fill(), List.tabulate()`等。如前所述，默认的工厂方法是`List.of()`，可以使用 Scala 风格的构造函数来简化:

```
List<Integer> list = List(1, 2, 3, 4, 5); 
```

我们还可以创建一个空列表(在`Vavr`中称为`Nil`对象):

```
List()
```

以类似的方式，我们可以创建其他类型的`Collection`:

```
Array arr = Array(1, 2, 3, 4, 5);
Stream stm = Stream(1, 2, 3, 4, 5);
Vector vec = Vector(1, 2, 3, 4, 5); 
```

## 8。结论

我们已经看到了最常见的`Vavr`类型和集合的构造函数。第 3 节中提到的静态导入提供的语法糖使得在库中创建所有类型变得容易。

你可以在 [GitHub 项目](https://web.archive.org/web/20220625221852/https://github.com/eugenp/tutorials/tree/master/vavr)中找到本文使用的所有代码示例。