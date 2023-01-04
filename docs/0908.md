# Java 8λ表达式中的异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lambda-exceptions>

## 1。概述

在 Java 8 中，Lambda 表达式通过提供一种简洁的方式来表达行为，从而开始促进函数式编程。然而，JDK 提供的`Functional Interfaces`不能很好地处理异常——当处理异常时，代码变得冗长而麻烦。

在本文中，我们将探索一些在编写 lambda 表达式时处理异常的方法。

## 2。处理未检查的异常

首先，我们用一个例子来理解问题。

我们有一个`List<Integer>`，我们想用这个列表的每个元素除一个常数，比如说 50，并打印结果:

```
List<Integer> integers = Arrays.asList(3, 9, 7, 6, 10, 20);
integers.forEach(i -> System.out.println(50 / i));
```

这种表达是可行的，但是有一个问题。如果列表中的任何元素是`0`，那么我们得到一个`ArithmeticException: / by zero`。让我们通过使用传统的`try-catch`块来解决这个问题，这样我们可以记录任何这样的异常，并继续执行下一个元素:

```
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(i -> {
    try {
        System.out.println(50 / i);
    } catch (ArithmeticException e) {
        System.err.println(
          "Arithmetic Exception occured : " + e.getMessage());
    }
});
```

使用`try-catch`解决了这个问题，但是失去了`Lambda Expression`的简洁性，它不再是一个小函数。

为了处理这个问题，我们可以为 lambda 函数编写一个 lambda 包装器**。让我们看看代码，看看它是如何工作的:**

```
static Consumer<Integer> lambdaWrapper(Consumer<Integer> consumer) {
    return i -> {
        try {
            consumer.accept(i);
        } catch (ArithmeticException e) {
            System.err.println(
              "Arithmetic Exception occured : " + e.getMessage());
        }
    };
}
```

```
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(lambdaWrapper(i -> System.out.println(50 / i)));
```

首先，我们编写了一个负责处理异常的包装器方法，然后将 lambda 表达式作为参数传递给这个方法。

包装器方法如预期的那样工作，但是，你可能会说它基本上是从 lambda 表达式中移除了`try-catch`块，并将其移动到另一个方法，并且它没有减少正在编写的代码的实际行数。

在包装器特定于特定用例的情况下确实如此，但是我们可以利用泛型来改进这种方法，并将其用于各种其他场景:

```
static <T, E extends Exception> Consumer<T>
  consumerWrapper(Consumer<T> consumer, Class<E> clazz) {

    return i -> {
        try {
            consumer.accept(i);
        } catch (Exception ex) {
            try {
                E exCast = clazz.cast(ex);
                System.err.println(
                  "Exception occured : " + exCast.getMessage());
            } catch (ClassCastException ccEx) {
                throw ex;
            }
        }
    };
}
```

```
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(
  consumerWrapper(
    i -> System.out.println(50 / i), 
    ArithmeticException.class));
```

正如我们所看到的，我们的包装器方法的这一次迭代需要两个参数，lambda 表达式和要捕获的`Exception`的类型。这个 lambda 包装器能够处理所有的数据类型，而不仅仅是`Integers`，并且捕捉任何特定类型的异常，而不是超类`Exception`。

另外，请注意，我们已经将方法的名称从`lambdaWrapper`更改为`consumerWrapper`。这是因为这个方法只处理类型为`Consumer`的`Functional Interface`的 lambda 表达式。我们可以为其他功能接口编写类似的包装方法，如`Function`、`BiFunction`、`BiConsumer`等等。

## 3。处理已检查的异常

让我们修改上一节中的示例，不打印到控制台，而是写入文件。

```
static void writeToFile(Integer integer) throws IOException {
    // logic to write to file which throws IOException
}
```

注意，上面的方法可能会抛出`IOException.`

```
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(i -> writeToFile(i));
```

在编译时，我们得到错误:

```
java.lang.Error: Unresolved compilation problem: Unhandled exception type IOException
```

因为 **`IOException`是一个被检查的异常，我们必须显式地处理它**。我们有两个选择。

首先，我们可以简单地将异常抛出我们的方法，并在其他地方处理它。

或者，我们可以在使用 lambda 表达式的方法内部处理它。

让我们探索这两个选项。

### 3.1。从 Lambda 表达式中抛出检查异常

让我们看看当我们在`main`方法上声明`IOException`时会发生什么:

```
public static void main(String[] args) throws IOException {
    List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
    integers.forEach(i -> writeToFile(i));
}
```

尽管如此，**在编译**的过程中，我们得到了同样的未处理的错误`IOException`。

```
java.lang.Error: Unresolved compilation problem: Unhandled exception type IOException
```

这是因为 lambda 表达式类似于[匿名内部类](/web/20221017183252/https://www.baeldung.com/java-anonymous-classes)。

在我们的例子中， **`writeToFile`方法是 [`Consumer<Integer>`](https://web.archive.org/web/20221017183252/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/class-use/Consumer.html) 功能接口**的实现。

让我们来看看`Consumer`的定义:

```
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

正如我们看到的,`accept`方法没有声明任何被检查的异常。这就是为什么`writeToFile`不被允许投掷`IOException.`

最直接的方法是使用一个`try-catch`块，将检查过的异常包装到未检查的异常中，然后重新抛出它:

```
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(i -> {
    try {
        writeToFile(i);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}); 
```

这将获得要编译和运行的代码。然而，这种方法引入了我们在上一节已经讨论过的相同问题——它冗长而繁琐。

我们可以做得更好。

让我们用一个抛出异常的`accept`方法创建一个定制的函数接口。

```
@FunctionalInterface
public interface ThrowingConsumer<T, E extends Exception> {
    void accept(T t) throws E;
}
```

现在，让我们实现一个能够抛出异常的包装器方法:

```
static <T> Consumer<T> throwingConsumerWrapper(
  ThrowingConsumer<T, Exception> throwingConsumer) {

    return i -> {
        try {
            throwingConsumer.accept(i);
        } catch (Exception ex) {
            throw new RuntimeException(ex);
        }
    };
}
```

最后，我们能够简化使用`writeToFile`方法的方式:

```
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(throwingConsumerWrapper(i -> writeToFile(i)));
```

这仍然是一种变通方法，但是**最终结果看起来非常干净，并且肯定更容易维护**。

`ThrowingConsumer`和`throwingConsumerWrapper`都是通用的，可以很容易地在应用程序的不同地方重用。

### 3.2。处理 Lambda 表达式中的检查异常

在最后一节中，我们将修改包装器来处理检查异常。

由于我们的`ThrowingConsumer`接口使用泛型，我们可以轻松处理任何特定的异常。

```
static <T, E extends Exception> Consumer<T> handlingConsumerWrapper(
  ThrowingConsumer<T, E> throwingConsumer, Class<E> exceptionClass) {

    return i -> {
        try {
            throwingConsumer.accept(i);
        } catch (Exception ex) {
            try {
                E exCast = exceptionClass.cast(ex);
                System.err.println(
                  "Exception occured : " + exCast.getMessage());
            } catch (ClassCastException ccEx) {
                throw new RuntimeException(ex);
            }
        }
    };
}
```

让我们看看如何在实践中使用它:

```
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(handlingConsumerWrapper(
  i -> writeToFile(i), IOException.class));
```

注意，上面的代码**只处理** **`IOException,`，而任何其他类型的异常都被重新抛出为`RuntimeException`** 。

## 4。结论

在本文中，我们展示了如何在包装器方法的帮助下处理 lambda 表达式中的特定异常，而不丧失简洁性。我们还学习了如何为 JDK 中的函数接口编写抛出替代方法，以抛出或处理被检查的异常。

另一种方法是[探索鬼鬼祟祟的投掷技巧。](https://web.archive.org/web/20221017183252/https://4comprehension.com/sneakily-throwing-exceptions-in-lambda-expressions-in-java/)

功能接口和包装方法的完整源代码可以从[这里](https://web.archive.org/web/20221017183252/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas/src/main/java/com/baeldung/java8/lambda/exceptions)下载，测试类可以从[这里下载，在 Github](https://web.archive.org/web/20221017183252/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas) 上。

**如果您正在寻找开箱即用的工作解决方案，那么 [ThrowingFunction](https://web.archive.org/web/20221017183252/https://github.com/pivovarit/throwing-function) 项目值得一试。**