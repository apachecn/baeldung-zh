# Vavr 的介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-either>

## 1。概述

[Vavr](https://web.archive.org/web/20221208143830/http://www.vavr.io/) 是一个面向 Java 8+的开源对象函数式语言扩展库。这有助于减少代码量并增加健壮性。

在本文中，我们将了解`Vavr`的工具`Either.` ，如果你想了解更多关于`Vavr`库的信息，[查看本文。](/web/20221208143830/https://www.baeldung.com/vavr)

## 2。什么是`Either`？

在函数式编程的世界里，函数式 `values or objects`是不可修改的(即在[范式](https://web.archive.org/web/20221208143830/https://en.wikipedia.org/wiki/Normal_form_(abstract_rewriting))中)；在 Java 术语中，它被称为[不可变](https://web.archive.org/web/20221208143830/https://en.wikipedia.org/wiki/Immutable_object)变量。

[任一](https://web.archive.org/web/20221208143830/https://static.javadoc.io/io.vavr/vavr/0.9.0/io/vavr/control/Either.html)代表两种可能数据类型的值。一个`Either`要么是一个`[Left](https://web.archive.org/web/20221208143830/https://static.javadoc.io/io.vavr/vavr/0.9.0/io/vavr/control/Either.Left.html)`要么是一个`[Right](https://web.archive.org/web/20221208143830/https://static.javadoc.io/io.vavr/vavr/0.9.0/io/vavr/control/Either.Right.html)`。按照惯例，`Left`表示失败案例结果，`Right`表示成功。

## 3。Maven 依赖关系

我们需要在`pom.xml`中添加以下依赖关系:

```java
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.9.0</version>
</dependency>
```

最新版本的`Vavr`可以在[中央 Maven 资源库](https://web.archive.org/web/20221208143830/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22vavr%22)中获得。

## 4。用例

让我们考虑一个用例，我们需要创建一个接受输入的方法，基于输入，我们将返回一个`String`或者一个`Integer`。

### 4.1。普通 Java

我们可以通过两种方式实现这一点。要么我们的方法可以返回一个带有表示成功/失败结果的键的映射，要么它可以返回一个固定大小的`List/Array`，其中位置表示结果类型。

这可能是这样的:

```java
public static Map<String, Object> computeWithoutEitherUsingMap(int marks) {
    Map<String, Object> results = new HashMap<>();
    if (marks < 85) {
        results.put("FAILURE", "Marks not acceptable");
    } else {
        results.put("SUCCESS", marks);
    }
    return results;
}

public static void main(String[] args) {
    Map<String, Object> results = computeWithoutEitherUsingMap(8);

    String error = (String) results.get("FAILURE");
    int marks = (int) results.get("SUCCESS");
}
```

对于第二种方法，我们可以使用下面的代码:

```java
public static Object[] computeWithoutEitherUsingArray(int marks) {
    Object[] results = new Object[2];
    if (marks < 85) {
        results[0] = "Marks not acceptable";
    } else {
        results[1] = marks;
    }
    return results;
}
```

正如我们所看到的，这两种方法都需要大量的工作，最终的结果既不美观也不安全。

### 4.2。`Either`同

现在让我们看看如何利用`Vavr`的`Either`实用程序来实现相同的结果:

```java
private static Either<String, Integer> computeWithEither(int marks) {
    if (marks < 85) {
        return Either.left("Marks not acceptable");
    } else {
        return Either.right(marks);
    }
} 
```

不需要，需要显式类型转换、空值检查或创建未使用的对象。

此外，`Either`提供了一个非常方便的类似一元的 API 来处理这两种情况:

```java
computeWithEither(80)
  .right()
  .filter(...)
  .map(...)
  // ...
```

按照惯例，`Either's Left`属性代表失败案例，`Right`属性代表成功案例。然而，基于我们的需求，我们可以使用预测来改变这一点——`Vavr`中的`Either`并不偏向`Left`或`Right.`

**如果我们投射到`Right,`如果`Either`是** `**Left.**` 像`filter(), map()`这样的操作将没有效果

例如，让我们创建`Right`投影，并在其上定义一些操作:

```java
computeWithEither(90).right()
  .filter(...)
  .map(...)
  .getOrElse(Collections::emptyList);
```

如果结果是我们将`Left`投影到了`Right,`，我们将立即得到一个空列表。

我们可以以类似的方式与`Left`投影进行交互:

```java
computeWithEither(9).left()
  .map(FetchError::getMsg)
  .forEach(System.out::println);
```

### 4.3。附加功能

有大量的`Either`实用程序可用；让我们来看看其中的一些。

我们可以使用`isLeft`和`isRight`方法检查一个`Either`是否只包含`Left`或`Right`:

```java
result.isLeft();
result.isRight();
```

我们可以检查`Either`是否包含给定的`Right`值:

```java
result.contains(100)
```

我们可以`fold`从左到右归纳出一种常见的类型:

```java
Either<String, Integer> either = Either.right(42);
String result = either.fold(i -> i, Object::toString);
```

或者…甚至交换立场:

```java
Either<String, Integer> either = Either.right(42);
Either<Integer, String> swap = either.swap();
```

## 5。结论

在这个快速教程中，我们学习了如何使用`Vavr`框架的`Either`工具。更多关于`Either`的细节可以在这里找到[。](https://web.archive.org/web/20221208143830/https://slides.com/pivovarit/to-try-or-not-to-try-there-is-no-throws#/32)

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/vavr-modules/vavr-2)