# 使用 Vavr 的 Lambda 表达式中的异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/exceptions-using-vavr>

## 1。简介

JDK 提供的`Functional Interfaces`没有为检查异常的处理做好准备。如果你想了解更多关于这个问题的信息，请查看这篇文章。

在本文中，我们将看看使用函数式 Java 库 [Vavr](https://web.archive.org/web/20221017183240/http://www.vavr.io/) 来克服这些问题的各种方法。

要获得更多关于 Vavr 以及如何设置它的信息，请查看本文。

## 2。使用`CheckedFunction`

Vavr 提供了`functional Interfaces`，它有抛出检查异常的函数。这些功能是`CheckedFunction0`、`CheckedFunction1`等等，直到`CheckedFunction8`。函数名末尾的`0, 1, … 8`表示函数输入参数的数量。

让我们看一个例子:

```java
static Integer readFromFile(Integer integer) throws IOException {
    // logic to read from file which throws IOException
} 
```

我们可以在 lambda 表达式中使用上述方法，而无需处理`IOException`:

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);

CheckedFunction1<Integer, Integer> readFunction = i -> readFromFile(i);
integers.stream()
 .map(readFunction.unchecked());
```

如你所见，没有标准的`try-catch`或包装方法，我们仍然可以在 lambda 表达式中调用异常抛出方法。

在使用这个特性和`Stream` API 时，我们必须小心，因为异常会立即终止操作——放弃流的其余部分。

## 3。使用助手方法

API 类为上一节中的示例提供了一个快捷方法:

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);

integers.stream()
  .map(API.unchecked(i -> readFromFile(i)));
```

## 4。使用提升

为了优雅地处理`IOException`，我们可以在 lambda 表达式中引入标准的`try-catch`块。然而，lambda 表达式的简洁性将会丧失。瓦弗尔的电梯来救我们了。

提升是一个来自函数式编程的概念。您可以将部分函数提升为返回结果为`Option`的总函数。

部分函数是仅针对一个域的子集定义的函数，而不是针对其整个域定义的全函数。如果使用超出其支持范围的输入调用分部函数，它通常会引发异常。

让我们重写上一节中的示例:

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);

integers.stream()
  .map(CheckedFunction1.lift(i -> readFromFile(i)))
  .map(k -> k.getOrElse(-1));
```

注意，提升函数的结果是`Option`，如果出现异常，结果将是`Option.None`。方法`getOrElse()`在`Option.None`的情况下取一个替换值返回。

## 5。使用`Try`

虽然上一节中的方法`lift()`解决了程序突然终止的问题，但它实际上吞下了异常。因此，我们方法的消费者不知道是什么导致了默认值。替代方法是使用一个`Try`容器。

是一个特殊的容器，我们可以用它来封装一个可能抛出异常的操作。在这种情况下，产生的`Try`对象代表一个`Failure`，它包装了异常。

让我们看看使用`Try`的代码:

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20); 
```

```java
integers.stream()
  .map(CheckedFunction1.liftTry(i -> readFromFile(i)))
  .flatMap(Value::toJavaStream)
  .forEach(i -> processValidValue(i));
```

要了解更多关于`Try`容器以及如何使用它，请查看[这篇文章](/web/20221017183240/https://www.baeldung.com/vavr-try)。

## 6。结论

在这篇简短的文章中，我们展示了如何使用 Vavr 库中的特性来规避问题，同时处理 lambda 表达式中的异常。

虽然这些特性允许我们优雅地处理异常，但是使用时应该非常小心。使用其中的一些方法，您的方法的消费者可能会对意外的检查异常感到惊讶，尽管它们没有被显式声明。

本文中所有示例的完整源代码可以在 Github 的[中找到。](https://web.archive.org/web/20221017183240/https://github.com/eugenp/tutorials/tree/master/vavr-modules/vavr)