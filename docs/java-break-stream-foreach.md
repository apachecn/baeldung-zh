# 如何脱离 Java 流 forEach

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-break-stream-foreach>

## 1.概观

作为 Java 开发人员，我们经常编写迭代一组元素并对每个元素执行操作的代码。Java 8 流库和它的`[forEach](/web/20221111182512/https://www.baeldung.com/foreach-java)`方法允许我们以一种清晰的、声明性的方式编写代码。

虽然这类似于循环，**我们缺少了中止迭代**的`break`语句的等价物。**一个流可以很长，或者可能是无限的**，如果我们没有理由继续处理它，我们会想从它那里中断，而不是等待它的最后一个元素。

在本教程中，我们将看看一些机制，**允许我们在`Stream.forEach`操作中模拟`break`语句。**

## 2.Java 9 的`Stream.takeWhile()`

假设我们有一个由*字符串*项组成的流，我们希望处理它的元素，只要它们的长度是奇数。

让我们试试 Java 9 `Stream.takeWhile`方法:

```java
Stream.of("cat", "dog", "elephant", "fox", "rabbit", "duck")
  .takeWhile(n -> n.length() % 2 != 0)
  .forEach(System.out::println);
```

如果我们运行这个，我们得到输出:

```java
cat
dog
```

让我们使用一个`for`循环和一个`break`语句将它与普通 Java 中的等价代码进行比较，以帮助我们了解它是如何工作的:

```java
List<String> list = asList("cat", "dog", "elephant", "fox", "rabbit", "duck");
for (int i = 0; i < list.size(); i++) {
    String item = list.get(i);
    if (item.length() % 2 == 0) {
        break;
    }
    System.out.println(item);
} 
```

正如我们所见，`takeWhile`方法允许我们准确地实现我们所需要的。

但是如果我们还没有采用 Java 9 呢？如何使用 Java 8 实现类似的事情？

## 3.一个习俗`Spliterator`

**让我们创建一个自定义 [`Spliterator`](/web/20221111182512/https://www.baeldung.com/java-spliterator) ，它将作为`Stream.spliterator`的[装饰器](/web/20221111182512/https://www.baeldung.com/java-decorator-pattern)。我们可以让这个`Spliterator`为我们表演`break`。**

首先，我们将从我们的流中获取`Spliterator`，然后用我们的`CustomSpliterator`来修饰它，并提供`Predicate`来控制`break`操作。最后，我们将从`CustomSpliterator:`创建一个新的流

```java
public static <T> Stream<T> takeWhile(Stream<T> stream, Predicate<T> predicate) {
    CustomSpliterator<T> customSpliterator = new CustomSpliterator<>(stream.spliterator(), predicate);
    return StreamSupport.stream(customSpliterator, false);
}
```

让我们看看如何创建`CustomSpliterator`:

```java
public class CustomSpliterator<T> extends Spliterators.AbstractSpliterator<T> {

    private Spliterator<T> splitr;
    private Predicate<T> predicate;
    private boolean isMatched = true;

    public CustomSpliterator(Spliterator<T> splitr, Predicate<T> predicate) {
        super(splitr.estimateSize(), 0);
        this.splitr = splitr;
        this.predicate = predicate;
    }

    @Override
    public synchronized boolean tryAdvance(Consumer<? super T> consumer) {
        boolean hadNext = splitr.tryAdvance(elem -> {
            if (predicate.test(elem) && isMatched) {
                consumer.accept(elem);
            } else {
                isMatched = false;
            }
        });
        return hadNext && isMatched;
    }
}
```

那么，我们来看看`tryAdvance`方法。我们在这里可以看到，自定义`Spliterator`处理了装饰过的`Spliterator`的元素。**只要我们的谓词匹配并且初始流仍然有元素，处理就完成了。**当任一条件变为`false`时，我们的`Spliterator` `“breaks”`和串流操作结束。

让我们测试一下新的助手方法:

```java
@Test
public void whenCustomTakeWhileIsCalled_ThenCorrectItemsAreReturned() {
    Stream<String> initialStream = 
      Stream.of("cat", "dog", "elephant", "fox", "rabbit", "duck");

    List<String> result = 
      CustomTakeWhile.takeWhile(initialStream, x -> x.length() % 2 != 0)
        .collect(Collectors.toList());

    assertEquals(asList("cat", "dog"), result);
}
```

我们可以看到，在满足条件后，流停止了。出于测试的目的，我们将结果收集到一个列表中，但是我们也可以使用一个`forEach`调用或者`Stream`的任何其他函数。

## 4.一个习俗`forEach`

虽然提供嵌入了`break`机制的`Stream`可能有用，但是只关注`forEach`操作可能更简单。

让我们直接使用 **`Stream.spliterator `** 而不用装饰器:

```java
public class CustomForEach {

    public static class Breaker {
        private boolean shouldBreak = false;

        public void stop() {
            shouldBreak = true;
        }

        boolean get() {
            return shouldBreak;
        }
    }

    public static <T> void forEach(Stream<T> stream, BiConsumer<T, Breaker> consumer) {
        Spliterator<T> spliterator = stream.spliterator();
        boolean hadNext = true;
        Breaker breaker = new Breaker();

        while (hadNext && !breaker.get()) {
            hadNext = spliterator.tryAdvance(elem -> {
                consumer.accept(elem, breaker);
            });
        }
    }
}
```

正如我们所看到的，新的自定义`forEach`方法调用了一个 [`BiConsumer`](/web/20221111182512/https://www.baeldung.com/java-8-functional-interfaces) ，为我们的代码提供了下一个元素和一个可以用来停止流的 breaker 对象。

让我们在单元测试中尝试一下:

```java
@Test
public void whenCustomForEachIsCalled_ThenCorrectItemsAreReturned() {
    Stream<String> initialStream = Stream.of("cat", "dog", "elephant", "fox", "rabbit", "duck");
    List<String> result = new ArrayList<>();

    CustomForEach.forEach(initialStream, (elem, breaker) -> {
        if (elem.length() % 2 == 0) {
            breaker.stop();
        } else {
            result.add(elem);
        }
    });

    assertEquals(asList("cat", "dog"), result);
}
```

## 5.结论

在本文中，我们研究了在流上提供调用`break`的等效方法。我们看到了 Java 9 的`takeWhile`如何为我们解决大部分问题，以及如何为 Java 8 提供一个版本。

最后，我们看了一个实用方法，它可以在迭代`Stream`时为我们提供相当于`break`操作的功能。

和往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221111182512/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-streams)