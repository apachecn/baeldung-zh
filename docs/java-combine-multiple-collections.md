# Java——组合多个集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-combine-multiple-collections>

## 1。概述

在本教程中，我们将说明如何将多个集合连接成一个逻辑集合。

我们将探索五种不同的方法——两种使用 Java 8，一种使用 Guava，一种使用 Apache Commons 集合，还有一种只使用标准的 Java 7 SDK。

在下面的示例中，让我们考虑以下集合:

```java
Collection<String> collectionA = Arrays.asList("S", "T");
Collection<String> collectionB = Arrays.asList("U", "V");
```

## 2。使用 Java 8 `Stream` API

Java API 中的`Stream`接口提供了有用的方法，使得处理集合变得更加容易。让我们来看看它的两个方法——`concat()`和`flatMap()`——用于组合集合。

一旦获得了一个`Stream`，就可以对它执行聚合操作。

### 2.1。使用`concat()` 方法

静态方法`concat()`通过创建一个延迟连接的`Stream`来逻辑地组合两个`Streams`，其元素是第一个`Stream`的所有元素，后跟第二个`Stream`的所有元素。

在下面的例子中，让我们使用`concat()` 方法组合`collectionA`和`collectionB`:

```java
Stream<String> combinedStream = Stream.concat(
  collectionA.stream(),
  collectionB.stream());
```

如果需要组合两个以上的`Streams`，可以从最初的调用中再次调用`concat()`方法:

```java
Stream<String> combinedStream = Stream.concat(
  Stream.concat(collectionA.stream(), collectionB.stream()), 
  collectionC.stream());
```

值得注意的是，Java 8 `Streams`是不可重用的，所以在将它们赋给变量时应该考虑到这一点。

### 2.2。使用`flatMap()` 方法

在用映射的`Stream`的内容替换这个`Stream`的每个元素之后， `flatMap()`方法返回一个`Stream`，该映射的内容是通过将提供的映射函数应用于每个元素而产生的。

下面的例子演示了使用`flatMap()`方法合并集合。最初，您得到一个`Stream`,它的元素是两个集合，然后您在将`Stream`收集到一个合并的列表之前将其展平:

```java
Stream<String> combinedStream = Stream.of(collectionA, collectionB)
  .flatMap(Collection::stream);
Collection<String> collectionCombined = 
  combinedStream.collect(Collectors.toList());
```

## 3。使用番石榴

Google 的 Guava 库提供了几种操作集合的方便方法，可以用于 Java 6 或更高版本。

### 3.1。使用`Iterables.concat()`方法

`Iterables.concat()`方法是用于合并集合的番石榴便利方法之一:

```java
Iterable<String> combinedIterables = Iterables.unmodifiableIterable(
  Iterables.concat(collectionA, collectionA));
```

返回的`Iterable`可以转换成集合:

```java
Collection<String> collectionCombined = Lists.newArrayList(combinedIterables);
```

### 3.2。Maven 依赖关系

将以下依赖项添加到 Maven `pom.xml`文件中，以便在项目中包含 Guava 库:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

您可以在 [Maven Central](https://web.archive.org/web/20221126220129/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 资源库中找到最新版本的番石榴库。

## 4。使用 Apache Commons 集合

Apache Commons Collections 是另一个帮助使用各种集合的实用程序库。该库提供了两种可用于组合集合的实用方法。在本节中，让我们了解这些方法是如何工作的。

### 4.1。使用`IterableUtils.chainedIterable()`方法

`IterableUtils`类为`Iterable`实例提供了实用方法和装饰器。它提供了`chainedIterable()`方法，可用于将多个`Iterable`组合成一个。

```java
Iterable<String> combinedIterables = IterableUtils.chainedIterable(
  collectionA, collectionB);
```

### 4.2。使用`CollectionUtils.union()`方法

`Collection`实例的实用方法和装饰器由`CollectionUtils`类提供。这个类的`union()`方法返回一个包含给定的`Iterable` 实例的并集的`Collection`。

```java
Iterable<String> combinedIterables = CollectionUtils.union(
  collectionA, collectionB);
```

在使用`union()`方法的情况下，返回集合中每个元素的基数将等于两个给定`Iterables`中该元素的最大基数。这意味着组合的集合只包含第一个集合中的元素和第二个集合中第一个集合中没有的元素。

### 4.3。Maven 依赖关系

将以下依赖项添加到您的 Maven `pom.xml`文件中，以便将 Apache Commons Collections 库包含在您的项目中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

您可以在 [Maven Central](https://web.archive.org/web/20221126220129/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-collections4%22) 存储库中找到 Apache Commons 库的最新版本。

## 5。使用 Java 7

如果您仍然使用 Java 7，并且希望避免使用第三方库，比如 Guava，那么您可以使用`addAll()`方法来组合来自多个集合的元素，或者您可以编写自己的实用程序方法来组合`Iterables`。

### 5.1。使用`addAll()`方法

当然，组合集合的最简单的解决方案是使用`addAll()`方法，如下面的`List`示例所示，但是值得注意的是，该方法创建了一个新的集合，其中附加了对前两个集合中相同对象的引用:

```java
List<String> listC = new ArrayList<>();
listC.addAll(listA);
listC.addAll(listB);
```

### 5.2。编写自定义`concat()`方法

下面的例子定义了一个接受两个`Iterables`并返回一个合并的`Iterable`对象的`concat()`方法:

```java
public static <E> Iterable<E> concat(
  Iterable<? extends E> i1,
  Iterable<? extends E> i2) {
        return new Iterable<E>() {
            public Iterator<E> iterator() {
                return new Iterator<E>() {
                    Iterator<? extends E> listIterator = i1.iterator();
                    Boolean checkedHasNext;
                    E nextValue;
                    private boolean startTheSecond;

                    void theNext() {
                        if (listIterator.hasNext()) {
                            checkedHasNext = true;
                            nextValue = listIterator.next();
                        } else if (startTheSecond)
                            checkedHasNext = false;
                        else {
                            startTheSecond = true;
                            listIterator = i2.iterator();
                            theNext();
                        }
                    }

                    public boolean hasNext() {
                        if (checkedHasNext == null)
                            theNext();
                        return checkedHasNext;
                    }

                    public E next() {
                        if (!hasNext())
                            throw new NoSuchElementException();
                        checkedHasNext = null;
                        return nextValue;
                    }

                    public void remove() {
                        listIterator.remove();
                    }
                };
            }
        };
    }
```

通过将两个集合作为参数传递，可以调用`concat()`方法:

```java
Iterable<String> combinedIterables = concat(collectionA, collectionB);
Collection<String> collectionCombined = makeListFromIterable(combinedIterables);
```

如果您需要`Iterable`作为`List`可用，您也可以使用`makeListFromIterable()`方法，该方法使用`Iterable`的成员创建一个`List`:

```java
public static <E> List<E> makeListFromIterable(Iterable<E> iter) {
    List<E> list = new ArrayList<E>();
    for (E item : iter) {
        list.add(item);
    }
    return list;
}
```

## 6。结论

本文讨论了在 Java 中逻辑地组合两个集合的几种不同方法，而无需创建对它们包含的对象的额外引用。

本教程的代码可以在 Github 的[上找到。](https://web.archive.org/web/20221126220129/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2)