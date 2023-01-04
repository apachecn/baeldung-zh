# 在 Java 中寻找两个列表之间的差异

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lists-difference>

## 1.概观

寻找相同数据类型的对象集合之间的差异是一项常见的编程任务。例如，假设我们有一份申请考试的学生名单和另一份通过考试的学生名单。这两个列表之间的差异会给我们没有通过考试的学生。

在`Java`中，没有明确的方法来找出`List` API 中两个列表之间的差异，尽管有一些辅助方法比较接近。

在这个快速教程中，**我们将看看如何找出两个列表之间的差异**。我们将尝试一些不同的方法，包括普通的`Java`(有和没有`Streams`)和使用第三方库，比如`Guava`和`Apache Commons Collections`。

## 2.测试设置

让我们从定义两个列表开始，我们将用它们来测试我们的例子:

```java
public class FindDifferencesBetweenListsUnitTest {

    private static final List listOne = Arrays.asList("Jack", "Tom", "Sam", "John", "James", "Jack");
    private static final List listTwo = Arrays.asList("Jack", "Daniel", "Sam", "Alan", "James", "George");

}
```

## 3.使用 Java `List` API

我们可以创建一个列表的**副本，然后使用`List`方法`removeAll()`删除与另一个**相同的所有元素:

```java
List<String> differences = new ArrayList<>(listOne);
differences.removeAll(listTwo);
assertEquals(2, differences.size());
assertThat(differences).containsExactly("Tom", "John");
```

让我们颠倒一下，以另一种方式找出不同之处:

```java
List<String> differences = new ArrayList<>(listTwo);
differences.removeAll(listOne);
assertEquals(3, differences.size());
assertThat(differences).containsExactly("Daniel", "Alan", "George");
```

我们还应该注意，如果我们想找到两个列表之间的公共元素，`List`还包含一个`retainAll` 方法。

## 4.使用流 API

Java [`Stream`](/web/20220706105054/https://www.baeldung.com/java-streams) 可以用来对集合中的数据进行顺序操作，包括**过滤列表之间的差异**:

```java
List<String> differences = listOne.stream()
            .filter(element -> !listTwo.contains(element))
            .collect(Collectors.toList());
assertEquals(2, differences.size());
assertThat(differences).containsExactly("Tom", "John");
```

与第一个例子一样，我们可以切换列表的顺序，从第二个列表中找到不同的元素:

```java
List<String> differences = listTwo.stream()
            .filter(element -> !listOne.contains(element))
            .collect(Collectors.toList());
assertEquals(3, differences.size());
assertThat(differences).containsExactly("Daniel", "Alan", "George");
```

我们要注意的是`List`的反复调用。对于较大的列表，`contains()`可能是一个开销很大的操作。

## 5.使用第三方库

### 5.1.使用谷歌番石榴

**`Guava`包含了得心应手的`Sets`。`difference`方法**，但是要使用它我们需要首先将我们的`List`转换成一个`Set`:

```java
List<String> differences = new ArrayList<>(Sets.difference(Sets.newHashSet(listOne), Sets.newHashSet(listTwo)));
assertEquals(2, differences.size());
assertThat(differences).containsExactlyInAnyOrder("Tom", "John");
```

我们应该注意到，将`List`转换为`Set`会产生重复数据删除和重新排序的效果。

### 5.2.使用 Apache Commons 集合

来自 `Apache Commons Collections`的`CollectionUtils` 类包含一个`removeAll` 方法。

这个方法做的**和`List`一样。`removeAll`，同时也为结果**创建一个新的集合:

```java
List<String> differences = new ArrayList<>((CollectionUtils.removeAll(listOne, listTwo)));
assertEquals(2, differences.size());
assertThat(differences).containsExactly("Tom", "John");
```

## 6.处理重复值

现在让我们看看当两个列表包含重复值时，如何找出差异。

为了实现这一点，**我们需要从第一个列表中删除重复的元素，删除的次数正好与第二个列表中包含的次数一样多。**

在我们的例子中，值`“Jack”`在第一个列表中出现了两次，在第二个列表中只出现了一次:

```java
List<String> differences = new ArrayList<>(listOne);
listTwo.forEach(differences::remove);
assertThat(differences).containsExactly("Tom", "John", "Jack");
```

我们也可以使用 `Apache Commons Collections`中的`subtract`方法来实现这一点:

```java
List<String> differences = new ArrayList<>(CollectionUtils.subtract(listOne, listTwo));
assertEquals(3, differences.size());
assertThat(differences).containsExactly("Tom", "John", "Jack");
```

## 7.结论

在本文中，**我们探索了几种方法来找出列表**之间的差异。

在示例中，我们介绍了一个基本的 Java 解决方案`,` ，一个使用`Streams` API 的解决方案，以及像`Google Guava`和`Apache Commons Collections.`这样的第三方库

我们还看到了如何处理重复值。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220706105054/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)