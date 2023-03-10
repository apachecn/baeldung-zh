# 亚特兰蒂斯赋格曲简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-fugue>

## 1。简介

[Fugue](https://web.archive.org/web/20220630135101/https://bitbucket.org/atlassian/fugue) 是 Atlassian 的 Java 库；它是支持**功能编程**的实用程序的集合。

在这篇文章中，我们将关注并探索最重要的赋格 API。

## 2。神游入门

要开始在我们的项目中使用 Fugue，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>io.atlassian.fugue</groupId>
    <artifactId>fugue</artifactId>
    <version>4.5.1</version>
</dependency>
```

我们可以在 Maven Central 上找到最新版本的[赋格](https://web.archive.org/web/20220630135101/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.atlassian.fugue%22%20AND%20a%3A%22fugue%22)。

## 3。`Option`

让我们从查看`Option` 类开始我们的旅程，它是 Fugue 对`java.util.Optional.`的回答

正如我们从名字中可以猜到的， **`Option'`是一个容器，表示一个可能不存在的值。**

换句话说，`Option` 要么是某种类型的`Some` 值，要么是`None`:

```java
Option<Object> none = Option.none();
assertFalse(none.isDefined());

Option<String> some = Option.some("value");
assertTrue(some.isDefined());
assertEquals("value", some.get());

Option<Integer> maybe = Option.option(someInputValue);
```

### 3.1。`map` 行动

标准的函数式编程 API 之一是`map()`方法，它允许将提供的函数应用于底层元素。

该方法将提供的函数应用于`Option`的值(如果它存在的话):

```java
Option<String> some = Option.some("value") 
  .map(String::toUpperCase);
assertEquals("VALUE", some.get());
```

### 3.2。`Option` 和一个`Null`值

除了命名上的不同，Atlassian 确实为`Option` 做了一些与`Optional`不同的设计选择；现在让我们来看看它们。

**我们不能直接创建一个持有`null`值**的非空`Option` :

```java
Option.some(null);
```

上面抛出了一个异常。

**然而，我们可以通过使用`map()`操作得到一个:**

```java
Option<Object> some = Option.some("value")
  .map(x -> null);
assertNull(some.get());
```

这在简单使用`java.util.Optional.`时是不可能的

### 3.3。`Option I`s`Iterable` 

`Option` 可以被视为最多容纳一个元素的集合，因此它实现`Iterable` 接口是有意义的。

这极大地提高了处理集合/流时的互操作性。

现在，例如，可以与另一个系列连接:

```java
Option<String> some = Option.some("value");
Iterable<String> strings = Iterables
  .concat(some, Arrays.asList("a", "b", "c"));
```

### 3.4。将`Option` 转换为`Stream`

由于一个`Option`是一个 `Iterable,` ，它也可以很容易地转换成一个 `Stream` 。

转换后，如果选项存在，`Stream`实例将只有一个元素，否则为零:

```java
assertEquals(0, Option.none().toStream().count());
assertEquals(1, Option.some("value").toStream().count());
```

### 3.5。`java.util.Optional`互通

如果我们需要一个标准的`Optional`实现，我们可以使用`toOptional()`方法轻松获得:

```java
Optional<Object> optional = Option.none()
  .toOptional();
assertTrue(Option.fromOptional(optional)
  .isEmpty());
```

### 3.6。`Options` 公用事业类

最后，Fugue 提供了一些实用方法来处理名副其实的`Options`类中的`Option`。

它的特点是使用了一些方法，比如用`filterNone` 从集合中移除空的`Options`，用`flatten`将`Options`的集合`ing`转化为封闭对象的集合，过滤掉空的`Options.`

此外，它还有几个将`Function<A,B>`提升为`Function<Option<A>, Option<B>>`的`lift` 方法的变体:

```java
Function<Integer, Integer> f = (Integer x) -> x > 0 ? x + 1 : null;
Function<Option<Integer>, Option<Integer>> lifted = Options.lift(f);

assertEquals(2, (long) lifted.apply(Option.some(1)).get());
assertTrue(lifted.apply(Option.none()).isEmpty());
```

当我们想将一个不知道`Option` 的函数传递给某个使用`Option`的方法时，这很有用。

注意，就像`map` 方法一样， **`lift`不会将 null 映射到`None` :**

```java
assertEquals(null, lifted.apply(Option.some(0)).get());
```

## 4.`Either` 对于有两种可能结果的计算

正如我们所见，`Option` 类允许我们以函数的方式处理缺少值的情况。

但是，有时候我们需要返回比“没有价值”更多的信息；例如，我们可能想要返回一个合法的值或者一个错误对象。

`Either` 类涵盖了这个用例。

**`Either` 的一个实例可以是一个`Right`或一个`Left but never both at the same time`。**

按照惯例，右边是成功计算的结果，而左边是例外情况。

### 4.1。构建一个`Either`

我们可以通过调用它的两个静态工厂方法之一来获得一个`Either` 实例。

如果我们想要一个包含`Right`值`:`的`Either`，我们就调用`right`

```java
Either<Integer, String> right = Either.right("value");
```

否则，我们称之为`left`:

```java
Either<Integer, String> left = Either.left(-1);
```

这里，我们的计算可以或者返回一个`String`或者一个`Integer.`

### 4.2。使用`Either`

当我们有一个`Either` 实例时，我们可以检查它是左还是右，并相应地采取行动:

```java
if (either.isRight()) {
    ...
}
```

更有趣的是，我们可以使用函数式风格来链接操作:

```java
either
  .map(String::toUpperCase)
  .getOrNull();
```

### 4.3。预测

与其他像`Option, Try,`这样的一元工具的主要区别在于它通常是无偏的。简单来说，如果我们调用 map()方法，`Either`不知道是与`Left`还是`Right`方合作。

这就是投影派上用场的地方。

**左投影和右投影是分别聚焦在左值或右值**上的`Either` 的镜面视图:

```java
either.left()
  .map(x -> decodeSQLErrorCode(x));
```

在上面的代码片段中，如果`Either`为`Left, decodeSQLErrorCode()`将应用于底层元素。如果`Either`是`Right,`就不会。使用右投影时，反过来也一样。

### 4.4。实用方法

和`Options`一样，Fugue 也为`Eithers`提供了一个充满实用程序的类，它的名字就像这样:`Eithers`。

它包含过滤、转换和迭代`Either`集合的方法。

## 5.使用`Try`进行异常处理

我们用另一种叫做`Try`的变体来结束我们在赋格中非此即彼的数据类型之旅。

`Try` 类似于`Either`，但不同之处在于它专门处理异常。

与`Option` 一样，与`Either`不同的是，`Try` 在单一类型上被参数化，因为“其他”类型被固定为`Exception`(而对于`Option` 来说，它是隐式的`Void`)。

因此，`Try` 既可以是`Success` 也可以是`Failure`:

```java
assertTrue(Try.failure(new Exception("Fail!")).isFailure());
assertTrue(Try.successful("OK").isSuccess());
```

### 5.1。实例化一个`Try`

通常，我们不会明确地将`Try` 创建为成功或失败；相反，我们将从方法调用中创建一个。

`Checked.of` 调用一个给定的函数并返回一个`Try` 封装它的返回值或任何抛出的异常:

```java
assertTrue(Checked.of(() -> "ok").isSuccess());
assertTrue(Checked.of(() -> { throw new Exception("ko"); }).isFailure());
```

另一个方法，`Checked.lift`采用一个潜在的抛出函数，`lifts`将其传递给一个返回`Try`的函数:

```java
Checked.Function<String, Object, Exception> throwException = (String x) -> {
    throw new Exception(x);
};

assertTrue(Checked.lift(throwException).apply("ko").isFailure());
```

### 5.2。与`Try`一起工作

一旦我们有了一个`Try`，我们可能最终想要用它做的三件最常见的事情是:

1.  提取其价值
2.  将一些操作链接到成功值
3.  用函数处理异常

此外，显然，丢弃`Try`或将其传递给其他方法，以上三个不是我们仅有的选择，但所有其他内置方法都只是这三个方法的一种便利。

### 5.3。提取成功值

为了提取这个值，我们使用了`getOrElse`方法:

```java
assertEquals(42, failedTry.getOrElse(() -> 42));
```

如果存在，它返回成功值，否则返回某个计算值。

没有`getOrThrow` 或类似的，但是由于`getOrElse`没有捕捉到任何异常，我们可以很容易地编写它:

```java
someTry.getOrElse(() -> {
    throw new NoSuchElementException("Nothing to get");
});
```

### 5.4。成功后链接呼叫

在函数式风格中，我们可以将函数应用于成功值(如果存在的话),而无需首先显式提取它。

这是我们在`Option`、`Either` 以及大多数其他容器和集合中发现的典型的`map` 方法:

```java
Try<Integer> aTry = Try.successful(42).map(x -> x + 1);
```

它返回一个`Try` 以便我们可以链接进一步的操作。

当然，我们也有`flatMap` 品种:

```java
Try.successful(42).flatMap(x -> Try.successful(x + 1));
```

### 5.5。从异常中恢复

我们有类似的映射操作，除了一个`Try` (如果存在的话)，而不是它的成功值。

然而，这些方法的不同之处在于，它们的含义是**从异常中恢复，即在默认情况下产生成功的`Try`** 。

因此，我们可以用`recover`产生一个新值:

```java
Try<Object> recover = Try
  .failure(new Exception("boo!"))
  .recover((Exception e) -> e.getMessage() + " recovered.");

assertTrue(recover.isSuccess());
assertEquals("boo! recovered.", recover.getOrElse(() -> null));
```

正如我们所看到的，恢复函数将异常作为唯一的参数。

如果恢复函数本身抛出，结果是另一个失败的`Try`:

```java
Try<Object> failure = Try.failure(new Exception("boo!")).recover(x -> {
    throw new RuntimeException(x);
});

assertTrue(failure.isFailure());
```

与`flatMap` 类似的称为`recoverWith`:

```java
Try<Object> recover = Try
  .failure(new Exception("boo!"))
  .recoverWith((Exception e) -> Try.successful("recovered again!"));

assertTrue(recover.isSuccess());
assertEquals("recovered again!", recover.getOrElse(() -> null));
```

## 6.其他公用事业

在我们结束之前，让我们快速浏览一下赋格中的其他一些实用程序。

### 6.1。配对

一个`Pair` 是一个非常简单和通用的数据结构，由两个同等重要的组件组成，Fugue 称之为`left` 和`right`:

```java
Pair<Integer, String> pair = Pair.pair(1, "a");

assertEquals(1, (int) pair.left());
assertEquals("a", pair.right());
```

除了映射和适用的函子模式，Fugue 没有在`Pair`上提供很多内置方法。

然而，`Pair`在整个程序库中都被使用，并且它们很容易被用户程序使用。

下一个可怜人的 Lisp 实现只差几个按键！

### 6.2。单位

`Unit` 是具有单个值的枚举，表示“没有值”。

它取代了 void 返回类型和`Void` 类，去掉了`null`:

```java
Unit doSomething() {
    System.out.println("Hello! Side effect");
    return Unit();
}
```

然而，令人惊讶的是， **`Option` 并不理解`Unit`，认为它有价值而不是没有价值。**

### 6.3。静态实用程序

我们有几个装满静态实用方法的类，我们不需要编写和测试它们。

`Functions`类提供了以各种方式使用和转换函数的方法:组合、应用、currying、使用`Option`的部分函数、弱记忆等等。

`Suppliers` 类为`Supplier`提供了一个类似的、但更有限的实用程序集合，即没有参数的函数。

最后，`Iterables` 和`Iterators`包含了许多静态方法，用于操作这两个广泛使用的标准 Java 接口。

## 7。结论

在这篇文章中，我们已经给出了 Atlassian 的赋格库的概述。

我们还没有触及像`Monoid` 和`Semigroups` 这样的代数类，因为它们不适合写通才文章。

然而，你可以在赋格 [javadocs](https://web.archive.org/web/20220630135101/https://docs.atlassian.com/fugue/4.5.1/fugue/apidocs/index.html) 和[源代码](https://web.archive.org/web/20220630135101/https://bitbucket.org/atlassian/fugue/src)中读到更多关于它们的内容。

我们还没有触及任何可选模块，例如提供与 Guava 和 Scala 的集成。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到——这是一个 Maven 项目，所以应该很容易导入和运行。