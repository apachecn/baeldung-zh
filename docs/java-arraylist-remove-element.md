# 从数组列表中移除元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arraylist-remove-element>

## 1.概观

在本教程中，我们将会看到如何使用不同的技术从 Java 的`ArrayList`中移除元素。给定一个体育项目列表，让我们看看如何去掉下面列表中的一些元素:

```java
List<String> sports = new ArrayList<>();
sports.add("Football");
sports.add("Basketball");
sports.add("Baseball");
sports.add("Boxing");
sports.add("Cycling");
```

## 2.`ArrayList#remove`

`ArrayList`有两种方法可以删除元素，**传递要删除的元素的索引**，或者**传递要删除的元素本身**，如果存在的话。我们将会看到这两种用法。

### 2.1.按索引删除

使用`remove`传递一个索引作为参数，我们可以**移除列表中指定位置**的元素，并将任何后续元素向左移动，从它们的索引中减去 1。执行后，`remove` 方法将返回被删除的元素:

```java
sports.remove(1); // since index starts at 0, this will remove "Basketball"
assertEquals(4, sports.size());
assertNotEquals(sports.get(1), "Basketball");
```

### 2.2.按元素移除

另一种方法是使用这种方法从列表中删除第一个出现的元素。从形式上讲，如果存在，我们将删除索引最低的元素，如果不存在，列表不变:

```java
sports.remove("Baseball");
assertEquals(4, sports.size());
assertFalse(sports.contains("Baseball"));
```

## 3.迭代时移除

有时候我们想在循环的时候从`ArrayList`中移除一个元素。由于没有生成一个`ConcurrentModificationException,` ，我们需要使用`Iterator`类来正确地完成它。

让我们看看如何在循环中去掉一个元素:

```java
Iterator<String> iterator = sports.iterator();
while (iterator.hasNext()) {
    if (iterator.next().equals("Boxing")) {
        iterator.remove();
    }
}
```

## 4\. `ArrayList#removeIf` (JDK 8+)

如果我们使用的是 **JDK 8 或更高的**版本，我们可以利用`ArrayList#``removeIf`**移除满足给定谓词的`ArrayList`的所有元素。**

```java
sports.removeIf(p -> p.equals("Cycling"));
assertEquals(4, sports.size());
assertFalse(sports.contains("Cycling"));
```

最后，我们可以使用像 Apache Commons 这样的第三方库[，如果我们想更深入，我们可以看看如何](/web/20221205191516/https://www.baeldung.com/java-array-remove-element)[以一种有效的方式](/web/20221205191516/https://www.baeldung.com/java-remove-value-from-list)删除所有特定事件。

## 5.结论

在本教程中，我们学习了在 Java 中从数组列表中移除元素的各种方法。

像往常一样，本教程中使用的所有示例都可以在 GitHub 上找到。