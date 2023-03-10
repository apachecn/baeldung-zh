# Java 中的 Spliterator 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-spliterator>

## 1。概述

Java 8 中引入的`Spliterator` 接口可以被**用于遍历和划分序列**。是`Streams`的基础实用工具，尤其是并行的。

在本文中，我们将介绍它的用法、特性、方法以及如何创建我们自己的定制实现。

## 2。T3`Spliterator`API

### 2.1。`tryAdvance`

这是用于单步执行序列的主要方法。方法**获取一个`Consumer`，这个`Consumer`用于依次消耗`Spliterator`的元素**，如果没有要遍历的元素，则返回`false`。

在这里，我们将看看如何使用它来遍历和划分元素。

首先，让我们假设我们有一个包含 35000 篇文章的`ArrayList` ，并且`Article`类被定义为:

```java
public class Article {
    private List<Author> listOfAuthors;
    private int id;
    private String name;

    // standard constructors/getters/setters
}
```

现在，让我们实现一个任务，该任务处理文章列表并为每个文章名称添加一个后缀“`– published by Baeldung”`:

```java
public String call() {
    int current = 0;
    while (spliterator.tryAdvance(a -> a.setName(article.getName()
      .concat("- published by Baeldung")))) {
        current++;
    }

    return Thread.currentThread().getName() + ":" + current;
}
```

请注意，该任务在完成执行时会输出已处理的文章数量。

另一个关键点是，我们使用了`tryAdvance()`方法来处理下一个元素。

### 2.2。`trySplit`

接下来，让我们拆分`Spliterators` (因此得名)并独立处理分区。

`trySplit` 方法试图将它分成两部分。然后，调用方处理元素，最后，返回的实例将处理其他元素，从而允许并行处理这两个元素。

让我们首先生成我们的列表:

```java
public static List<Article> generateElements() {
    return Stream.generate(() -> new Article("Java"))
      .limit(35000)
      .collect(Collectors.toList());
}
```

接下来，我们使用`spliterator()`方法获得我们的`Spliterator`实例。然后我们应用我们的`trySplit()`方法:

```java
@Test
public void givenSpliterator_whenAppliedToAListOfArticle_thenSplittedInHalf() {
    Spliterator<Article> split1 = Executor.generateElements().spliterator(); 
    Spliterator<Article> split2 = split1.trySplit(); 

    assertThat(new Task(split1).call()) 
      .containsSequence(Executor.generateElements().size() / 2 + ""); 
    assertThat(new Task(split2).call()) 
      .containsSequence(Executor.generateElements().size() / 2 + ""); 
}
```

**拆分过程按预期进行，并平分记录**。

### 2.3。`estimatedSize`

`estimatedSize`方法给出了元素的估计数量:

```java
LOG.info("Size: " + split1.estimateSize());
```

这将输出:

```java
Size: 17500
```

### 2.4。`hasCharacteristics`

这个 API 检查给定的特征是否匹配`Spliterator.` 的属性，如果我们调用上面的方法，输出将是这些特征的`int` 表示:

```java
LOG.info("Characteristics: " + split1.characteristics());
```

```java
Characteristics: 16464
```

## 3. **`Spliterator`特点**

它有八个不同的特征来描述它的行为。这些可以作为外部工具的提示:

*   **`SIZED`** `–`如果能够用`estimateSize()`方法返回元素的精确数目
*   **`SORTED`** –如果是遍历一个已排序的源
*   **`SUBSIZED`** ——如果我们使用`trySplit()`方法拆分实例，并且获得也是`SIZED`的拆分器
*   **`CONCURRENT`**–如果可以安全地同时修改源代码
*   **`DISTINCT`** –如果对于每一对遇到的元素`x, y, !x.equals(y)`
*   **`IMMUTABLE`** –如果源持有的元素不能在结构上修改
*   **`NONNULL`** –源是否为空
*   **`ORDERED`** –如果迭代一个有序序列

## 4.一个习俗`Spliterator`

### 4.1。何时定制

首先，让我们假设以下场景:

我们有一个包含作者列表的 article 类，文章可以有多个作者。此外，如果作者的相关文章 id 与文章 id 匹配，我们就认为作者与文章相关。

我们的`Author`类将看起来像这样:

```java
public class Author {
    private String name;
    private int relatedArticleId;

    // standard getters, setters & constructors
}
```

接下来，我们将实现一个类，在遍历作者流时对作者进行计数。然后**该类将在流上执行归约**。

让我们来看看这个类的实现:

```java
public class RelatedAuthorCounter {
    private int counter;
    private boolean isRelated;

    // standard constructors/getters

    public RelatedAuthorCounter accumulate(Author author) {
        if (author.getRelatedArticleId() == 0) {
            return isRelated ? this : new RelatedAuthorCounter( counter, true);
        } else {
            return isRelated ? new RelatedAuthorCounter(counter + 1, false) : this;
        }
    }

    public RelatedAuthorCounter combine(RelatedAuthorCounter RelatedAuthorCounter) {
        return new RelatedAuthorCounter(
          counter + RelatedAuthorCounter.counter, 
          RelatedAuthorCounter.isRelated);
    }
}
```

上述类中的每个方法在遍历时执行特定的计数操作。

首先， **`accumulate()`方法以迭代的方式逐个遍历作者**，然后 **`combine()`使用它们的值**对两个计数器求和。最后，`getCounter()`返回计数器。

现在，为了测试我们到目前为止所做的。让我们将文章的作者列表转换为作者流:

```java
Stream<Author> stream = article.getListOfAuthors().stream();
```

并实现一个 **`countAuthor()`方法，使用`RelatedAuthorCounter`** 对流进行约简:

```java
private int countAutors(Stream<Author> stream) {
    RelatedAuthorCounter wordCounter = stream.reduce(
      new RelatedAuthorCounter(0, true), 
      RelatedAuthorCounter::accumulate, 
      RelatedAuthorCounter::combine);
    return wordCounter.getCounter();
}
```

如果我们使用顺序流，输出将如预期的那样`“count = 9”`，然而，当我们试图并行化操作时，问题出现了。

让我们来看看下面的测试用例:

```java
@Test
void 
  givenAStreamOfAuthors_whenProcessedInParallel_countProducesWrongOutput() {
    assertThat(Executor.countAutors(stream.parallel())).isGreaterThan(9);
}
```

显然，有些地方出错了——在任意位置分割流导致一个作者被计算两次。

### 4.2。如何定制

为了解决这个问题，我们需要**实现一个`Spliterator`，只有当相关的`id`和`articleId`匹配**时，它才会拆分作者。下面是我们的自定义`Spliterator`的实现:

```java
public class RelatedAuthorSpliterator implements Spliterator<Author> {
    private final List<Author> list;
    AtomicInteger current = new AtomicInteger();
    // standard constructor/getters

    @Override
    public boolean tryAdvance(Consumer<? super Author> action) {
        action.accept(list.get(current.getAndIncrement()));
        return current.get() < list.size();
    }

    @Override
    public Spliterator<Author> trySplit() {
        int currentSize = list.size() - current.get();
        if (currentSize < 10) {
            return null;
        }
        for (int splitPos = currentSize / 2 + current.intValue();
          splitPos < list.size(); splitPos++) {
            if (list.get(splitPos).getRelatedArticleId() == 0) {
                Spliterator<Author> spliterator
                  = new RelatedAuthorSpliterator(
                  list.subList(current.get(), splitPos));
                current.set(splitPos);
                return spliterator;
            }
        }
        return null;
   }

   @Override
   public long estimateSize() {
       return list.size() - current.get();
   }

   @Override
   public int characteristics() {
       return CONCURRENT;
   }
}
```

现在应用`countAuthors()`方法将给出正确的输出。下面的代码演示了:

```java
@Test
public void
  givenAStreamOfAuthors_whenProcessedInParallel_countProducesRightOutput() {
    Stream<Author> stream2 = StreamSupport.stream(spliterator, true);

    assertThat(Executor.countAutors(stream2.parallel())).isEqualTo(9);
}
```

另外，自定义`Spliterator`是从作者列表中创建的，并通过保持当前位置来遍历它。

让我们更详细地讨论每种方法的实现:

*   `**tryAdvance** –`将作者传递到当前索引位置的`Consumer`,并增加其位置
*   `**trySplit** –`定义了拆分机制，在我们的例子中，`RelatedAuthorSpliterator`是在 id 匹配时创建的，拆分将列表分为两部分
*   **`estimatedSize`** –是列表大小和当前迭代作者位置之间的差值
*   `**characteristics**` –返回`Spliterator`特征，在我们的例子中是`SIZED`，因为`estimatedSize()`方法返回的值是精确的；此外，`CONCURRENT`表示这个`Spliterator`的源代码可以被其他线程安全地修改

## 5。支持原始值

**`Spliterator` `API`支持原始值，包括`double`、`int`、`long`、**。

使用泛型和原语专用`Spliterator`的唯一区别是给定的`Consumer`和`Spliterator`的类型。

例如，当我们需要一个`int`值时，我们需要传递一个`intConsumer`。此外，这里还有一份原始人奉献的清单`Spliterators`:

*   `OfPrimitive<T, T_CONS, T_SPLITR extends Spliterator.OfPrimitive<T, T_CONS, T_SPLITR>>`:其他图元的父接口
*   `OfInt`:专用于`int`的一款`Spliterator`
*   `OfDouble`:一个专用于`double`的`Spliterator`
*   `OfLong`:一个专用于`long`的`Spliterator`

## 6。结论

在本文中，我们介绍了 Java 8 **`Spliterator`的用法、方法、特性、拆分过程、原语支持以及如何定制它。**

和往常一样，这篇文章的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220626104311/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8)