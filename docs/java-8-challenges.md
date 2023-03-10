# Java 8 面临的挑战

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-challenges>

## 1。概述

Java 8 引入了一些新特性，主要围绕 lambda 表达式的使用。在这篇简短的文章中，我们将看看其中一些缺点。

虽然这不是一个完整的列表，但它是关于 Java 8 新特性的最常见和最受欢迎的抱怨的主观集合。

## 2。Java 8 流和线程池

首先，并行流旨在使序列的简单并行处理成为可能，这对于简单的场景来说是很好的。

该流使用默认的通用`[ForkJoinPool](https://web.archive.org/web/20220629002327/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinPool.html)`–将序列分割成更小的块，并使用多线程执行操作。

然而，有一个问题。没有好办法让**指定哪个`ForkJoinPool`使用**，因此，如果其中一个线程被阻塞，所有其他使用共享池的线程将不得不等待长时间运行的任务完成。

幸运的是，有一个解决方法:

```java
ForkJoinPool forkJoinPool = new ForkJoinPool(2);
forkJoinPool.submit(() -> /*some parallel stream pipeline */)
  .get();
```

这将创建一个新的、单独的`ForkJoinPool`，所有由并行流生成的任务将使用指定的池，而不是共享的默认池。

值得注意的是，还有另一个潜在的问题:`“this technique of submitting a task to a fork-join pool, to run the parallel stream in that pool is an implementation ‘trick' and is not guaranteed to work”`，来自甲骨文的 Java 和 OpenJDK 开发者 Stuart Marks 说。使用这种技术时要记住一个重要的细微差别。

## 3。可调试性降低

新的编码风格简化了我们的源代码，然而 **在调试它的时候会让人头疼**。

首先，我们来看这个简单的例子:

```java
public static int getLength(String input) {
    if (StringUtils.isEmpty(input) {
        throw new IllegalArgumentException();
    }
    return input.length();
}

List lengths = new ArrayList();

for (String name : Arrays.asList(args)) {
    lengths.add(getLength(name));
}
```

这是一个不言自明的标准命令式 Java 代码。

如果我们传递空的`String`作为输入——结果是——代码将抛出一个异常，在调试控制台中，我们可以看到:

```java
at LmbdaMain.getLength(LmbdaMain.java:19)
at LmbdaMain.main(LmbdaMain.java:34)
```

现在，让我们使用 Stream API 重新编写相同的代码，看看当一个空的`String`被传递时会发生什么:

```java
Stream lengths = names.stream()
  .map(name -> getLength(name));
```

调用堆栈将类似于:

```java
at LmbdaMain.getLength(LmbdaMain.java:19)
at LmbdaMain.lambda$0(LmbdaMain.java:37)
at LmbdaMain$Lambda$1/821270929.apply(Unknown Source)
at java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:193)
at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:512)
at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:502)
at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708)
at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
at java.util.stream.LongPipeline.reduce(LongPipeline.java:438)
at java.util.stream.LongPipeline.sum(LongPipeline.java:396)
at java.util.stream.ReferencePipeline.count(ReferencePipeline.java:526)
at LmbdaMain.main(LmbdaMain.java:39)
```

这是我们在代码中利用多个抽象层所付出的代价。然而，ide 已经开发了调试 Java 流的可靠工具。

## 4.返回`Null`或`Optional`的方法

Java 8 中引入了 [`Optional`](https://web.archive.org/web/20220629002327/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html) 来提供一种类型安全的方式来表达可选性。

`Optional`，明确表示返回值可能不存在。因此，调用一个方法可能会返回一个值，而`Optional`用于将该值包装在里面——这被证明是很方便的。

不幸的是，由于 Java 的向后兼容性，我们有时会以混合了两种不同约定的 Java APIs 而告终。在同一个类中，我们可以找到返回空值的方法以及返回`Optionals.`的方法

## 5。过多的功能接口

在`[java.util.function](https://web.archive.org/web/20220629002327/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/package-summary.html)`包中，我们有一个 lambda 表达式的目标类型集合。我们可以将它们区分和分组为:

*   `Consumer`–表示接受一些参数但不返回结果的操作
*   `Function`–表示接受一些参数并产生结果的函数
*   `Operator`–表示对某些类型参数的运算，并返回与操作数相同类型的结果
*   `Predicate`–表示一些参数的谓词(`boolean`-值函数)
*   `Supplier`–表示不接受参数并返回结果的供应商

此外，我们还获得了处理原语的其他类型:

*   `IntConsumer`
*   `IntFunction`
*   `IntPredicate`
*   `IntSupplier`
*   `IntToDoubleFunction`
*   `IntToLongFunction`
*   …对于`Longs`和`Doubles`也有相同的替代方案

此外，arity 为 2 的函数的特殊类型:

*   `BiConsumer`
*   `BiPredicate`
*   `BinaryOperator`
*   `BiFunction`

结果，整个包包含了 44 个函数类型，这肯定会引起混乱。

## 6。检查异常和 Lambda 表达式

在 Java 8 之前，检查异常就已经是一个有争议的问题。自从 Java 8 的到来，新的问题出现了。

必须立即捕获或声明检查过的异常。由于`java.util.function`函数接口不声明抛出异常，抛出检查异常的代码将在编译期间失败:

```java
static void writeToFile(Integer integer) throws IOException {
    // logic to write to file which throws IOException
}
```

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(i -> writeToFile(i));
```

克服这个问题的一个方法是将检查过的异常封装在一个`try-catch`块中，然后重新抛出`RuntimeException`:

```java
List<Integer> integers = Arrays.asList(3, 9, 7, 0, 10, 20);
integers.forEach(i -> {
    try {
        writeToFile(i);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
});
```

这会有用的。然而，抛出`RuntimeException`与检查异常的目的相矛盾，并使整个代码包裹在样板代码中，我们正试图通过利用 lambda 表达式来减少这种情况。其中一个黑客解决方案是[依靠偷偷摸摸的扔黑客。](https://web.archive.org/web/20220629002327/https://4comprehension.com/sneakily-throwing-exceptions-in-lambda-expressions-in-java/)

另一个解决方案是编写一个消费者功能接口——它可以抛出一个异常:

```java
@FunctionalInterface
public interface ThrowingConsumer<T, E extends Exception> {
    void accept(T t) throws E;
}
```

```java
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

不幸的是，我们仍然将检查过的异常包装在运行时异常中。

最后，对于问题的深入解决方案和解释，我们可以深入探讨以下内容:[Java 8 Lambda 表达式中的异常](/web/20220629002327/https://www.baeldung.com/java-lambda-exceptions)。

## 8T2。结论

在这篇快速的文章中，我们讨论了 Java 8 的一些缺点。

虽然其中一些是 Java 语言架构师经过深思熟虑做出的设计选择，但在许多情况下都有变通方法或替代解决方案；我们确实需要意识到它们可能存在的问题和局限性。