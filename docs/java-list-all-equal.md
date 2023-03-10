# 确定 Java 列表中的所有元素是否都相同

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-all-equal>

## 1.概观

在这个快速教程中，我们将了解如何确定一个`List`中的所有元素是否都相同。

我们还将使用大 O 符号查看每个解决方案的时间复杂度，给出最坏的情况。

## 2.例子

假设我们有以下 3 个列表:

```java
notAllEqualList = Arrays.asList("Jack", "James", "Sam", "James");
emptyList = Arrays.asList();
allEqualList = Arrays.asList("Jack", "Jack", "Jack", "Jack");
```

我们的任务是提出不同的解决方案，只为`emptyList`和`allEqualList`返回`true`。

## 3.基本循环

首先，对于所有相等的元素，它们都必须等于第一个元素。让我们在循环中利用这一点:

```java
public boolean verifyAllEqualUsingALoop(List<String> list) {
    for (String s : list) {
        if (!s.equals(list.get(0)))
            return false;
    }
    return true;
}
```

这很好，因为虽然时间复杂度是`O(n)`，但它可能经常提前退出。

## 4.`HashSet`

我们也可以使用 [`HashSet`](/web/20221019231159/https://www.baeldung.com/java-hashset) ，因为它的所有元素都是不同的。I **如果我们将一个`List`转换成一个`HashSet`，并且结果大小小于或等于 1，那么我们知道列表中的所有元素都相等:**

```java
public boolean verifyAllEqualUsingHashSet(List<String> list) {
    return new HashSet<String>(list).size() <= 1;
}
```

将一个 [`List`转换成`HashSet`需要`O(n)`](/web/20221019231159/https://www.baeldung.com/java-collections-complexity) 时间，而[调用`size`需要`O(1)`](/web/20221019231159/https://www.baeldung.com/java-collections-complexity) 。因此，我们仍然有总时间复杂度`O(n)`。

## 5\. `Collections` API

另一个解决方案是使用[集合 API](/web/20221019231159/https://www.baeldung.com/java-collections) 的`frequency(Collection c, Object o)`方法。**这个方法返回一个`Collection c`中匹配一个`Object o`T6 的元素个数。**

所以，如果频率结果等于列表的大小，我们知道所有的元素都是相等的:

```java
public boolean verifyAllEqualUsingFrequency(List<String> list) {
    return list.isEmpty() || Collections.frequency(list, list.get(0)) == list.size();
}
```

类似于前面的解决方案，时间复杂度是`O(n)`，因为在内部， `Collections.frequency()`使用基本循环。

## 6.流

Java 8 中的 [`Stream` API 为我们提供了更多检测列表中所有项目是否相等的方法。](/web/20221019231159/https://www.baeldung.com/java-8-streams)

### 6.1.`distinct()`

让我们看一个利用`distinct() `方法的特殊解决方案。

为了验证一个列表中的所有元素是否相等，**我们对其流中的不同元素进行计数:**

```java
public boolean verifyAllEqualUsingStream(List<String> list) {
    return list.stream()
      .distinct()
      .count() <= 1;
}
```

如果这个流的计数小于或等于 1，那么所有的元素都相等，我们返回`true`。

操作的总成本是`O(n),`，这是遍历所有流元素所花费的时间。

### 6.2.`allMatch()`

`Stream` API 的`allMatch()`方法提供了一个完美的解决方案来确定这个流的所有元素是否匹配所提供的谓词:

```java
public boolean verifyAllEqualAnotherUsingStream(List<String> list) {
    return list.isEmpty() || list.stream()
      .allMatch(list.get(0)::equals);
}
```

类似于前面使用流的例子，这个例子有一个`O(n)`时间复杂度，即遍历整个流的时间。

## 7.第三方库

如果我们停留在 Java 的早期版本，不能使用流 API，**我们可以使用第三方库，比如`Google Guava`和`Apache Commons`。**

这里，我们有两个非常相似的解决方案，遍历元素列表并将其与第一个元素匹配。因此，我们可以很容易地计算出时间复杂度为`O(n)`。

### 7.1.Maven 依赖性

要使用其中任何一个，我们可以将 [`guava`](https://web.archive.org/web/20221019231159/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20a%3A%22guava%22) 或 [`commons-collections4`](https://web.archive.org/web/20221019231159/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-collections4%22%20g%3A%22org.apache.commons%22) 分别添加到我们的项目中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

### 7.2.谷歌番石榴

[在`Google Guava`中，如果列表中的所有元素都满足谓词，静态方法`Iterables.all()`](/web/20221019231159/https://www.baeldung.com/guava-filter-and-transform-a-collection) 返回`true`:

```java
public boolean verifyAllEqualUsingGuava(List<String> list) {
    return Iterables.all(list, new Predicate<String>() {
        public boolean apply(String s) {
            return s.equals(list.get(0));
        }
    });
}
```

### 7.3 .Apache common(Apache 公共)

类似地，`Apache Commons`库也提供了一个实用程序类 [`IterableUtils`](https://web.archive.org/web/20221019231159/https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/IterableUtils.html) ，它有一组静态实用程序方法来操作`Iterable`实例。

特别是，如果列表中的所有元素都满足谓词，静态方法`IterableUtils.matchesAll()`将返回`true`:

```java
public boolean verifyAllEqualUsingApacheCommon(List<String> list) {
    return IterableUtils.matchesAll(list, new org.apache.commons.collections4.Predicate<String>() {
        public boolean evaluate(String s) {
            return s.equals(list.get(0));
        }
    });
} 
```

## 8.结论

在本文中，我们已经学习了验证一个`List` 中的所有元素是否相等的不同方法，从简单的 Java 功能开始，然后展示使用`Stream` API 和第三方库`Google Guava`和`Apache Commons.`的替代方法

我们还了解到，每个解决方案都给了我们相同的时间复杂度`O(n)`。然而，这取决于我们根据如何使用和在哪里使用来选择最好的一个。

请务必在 GitHub 上查看完整的样本集[。](https://web.archive.org/web/20221019231159/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)