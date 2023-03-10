# 在 Java 中获取 Iterable 的大小

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterable-size>

## 1.概观

在这个快速教程中，我们将学习在 Java 中获取 [`Iterable`](https://web.archive.org/web/20220711232709/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Iterable.html) 大小的各种方法。

## 2.可迭代和迭代器

**`Iterable`是 Java 中集合类的主要接口之一。**

[`Collection`](https://web.archive.org/web/20220711232709/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html) 接口扩展了`Iterable`，因此`Collection`的所有子类也实现了`Iterable`。

`Iterable`只有一种方法产生一个 [`Iterator`](https://web.archive.org/web/20220711232709/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html) `:`

```java
public interface Iterable<T> {
    public Iterator<T> iterator();    
}
```

这个`Iterator`可以用来迭代`Iterable`中的元素。

## 3.使用核心 Java 的可迭代大小

### 3.1.`for-each`循环

**所有实现`Iterable`的类都有资格使用 Java 中的 [`for-each`](https://web.archive.org/web/20220711232709/https://docs.oracle.com/javase/8/docs/technotes/guides/language/foreach.html) 循环。**

这允许我们循环遍历`Iterable`中的元素，同时递增计数器以获得其大小:

```java
int counter = 0;
for (Object i : data) {
    counter++;
}
return counter;
```

### 3.2.`Collection.size()`

**在大多数情况下，`Iterable`将是`Collection, `的一个实例，如`List`或`Set`。**

在这种情况下，我们可以检查`Iterable`的类型，并对其调用`size()`方法来获得元素的数量。

```java
if (data instanceof Collection) {
    return ((Collection<?>) data).size();
}
```

调用`size()`通常比遍历整个集合要快得多。

**下面的例子展示了上述两种解决方案的组合:**

```java
public static int size(Iterable data) {

    if (data instanceof Collection) {
        return ((Collection<?>) data).size();
    }
    int counter = 0;
    for (Object i : data) {
        counter++;
    }
    return counter;
}
```

### 3.3.`Stream.count()`

**如果我们使用 Java 8，我们可以从`Iterable.`** 创建一个 [`Stream`](https://web.archive.org/web/20220711232709/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html)

然后可以使用流对象来获得`Iterable`中元素的计数。

```java
return StreamSupport.stream(data.spliterator(), false).count();
```

## 4.使用第三方库的可迭代大小

### 4.1.`IterableUtils#size()`

**`Apache Commons Collections`库有一个漂亮的 [`IterableUtils`](https://web.archive.org/web/20220711232709/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/IterableUtils.html) 类，为`Iterable`实例提供静态实用方法。**

在开始之前，我们需要从 [Maven Central](https://web.archive.org/web/20220711232709/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22) 导入最新的依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

我们可以在一个`Iterable`对象上调用`IterableUtils`的`size()`方法来获取它的大小。

```java
return IterableUtils.size(data);
```

### 4.2.`Iterables#size()`

**类似地，`Google Guava`库也在其 [`Iterables`](https://web.archive.org/web/20220711232709/https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#size(java.lang.Iterable)) 类中提供了一组静态实用方法来操作`Iterable`实例。**

在开始之前，我们需要从 [Maven Central](https://web.archive.org/web/20220711232709/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 导入最新的依赖项:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

调用`Iterables`类上的静态`size()`方法给我们提供了元素的数量。

```java
return Iterables.size(data);
```

在发动机罩下，`IterableUtils`和`Iterables`都使用 3.1 和 3.2 中描述的方法组合来确定尺寸。

## 5.结论

在本文中，我们研究了在 Java 中获取`Iterable`大小的不同方法。

GitHub 上的[提供了本文的源代码和相关的测试用例。](https://web.archive.org/web/20220711232709/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2)