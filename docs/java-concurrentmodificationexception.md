# 避免 Java 中的并发修改异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrentmodificationexception>

## 1。简介

在本文中，我们将看看`[ConcurrentModificationException](https://web.archive.org/web/20220928095038/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ConcurrentModificationException.html)` 类。

首先，我们将解释它是如何工作的，然后通过一个触发它的测试来证明。

最后，我们将通过实际例子尝试一些变通方法。

## 2。触发一个`ConcurrentModificationException`

本质上，`ConcurrentModificationException` 用于当我们迭代的东西被修改时**快速失效。**让我们用一个简单的测试来证明这一点:

```java
@Test(expected = ConcurrentModificationException.class)
public void whilstRemovingDuringIteration_shouldThrowException() throws InterruptedException {

    List<Integer> integers = newArrayList(1, 2, 3);

    for (Integer integer : integers) {
        integers.remove(1);
    }
}
```

正如我们所看到的，在完成迭代之前，我们删除了一个元素。这就是触发异常的原因。

## 3。解决方案

有时，我们实际上可能想在迭代时从集合中移除元素。如果是这种情况，那么有一些解决方法。

### 3.1。直接使用迭代器

一个`for-each`循环在幕后使用一个`Iterator` ,但是不那么冗长。然而，如果我们重构之前的测试，使用一个`Iterator,` ，我们将可以访问额外的方法，比如`remove().` ，让我们尝试使用这个方法来修改我们的列表:

```java
for (Iterator<Integer> iterator = integers.iterator(); iterator.hasNext();) {
    Integer integer = iterator.next();
    if(integer == 2) {
        iterator.remove();
    }
}
```

现在我们会注意到没有例外。原因是`remove()` 方法不会导致在迭代时调用`ConcurrentModificationException.` 是安全的。

### 3.2。迭代期间不移除

如果我们想保持我们的`for-each` 循环，那么我们可以。只是我们需要等到迭代之后再移除元素。让我们通过在迭代时将我们想要删除的内容添加到一个`toRemove`列表中来尝试一下:

```java
List<Integer> integers = newArrayList(1, 2, 3);
List<Integer> toRemove = newArrayList();

for (Integer integer : integers) {
    if(integer == 2) {
        toRemove.add(integer);
    }
}
integers.removeAll(toRemove);

assertThat(integers).containsExactly(1, 3); 
```

这是解决问题的另一个有效方法。

### 3.3。使用`removeIf()`

Java 8 向`Collection`接口引入了`removeIf()` 方法。这意味着，如果我们正在使用它，我们可以使用函数式编程的思想来再次实现相同的结果:

```java
List<Integer> integers = newArrayList(1, 2, 3);

integers.removeIf(i -> i == 2);

assertThat(integers).containsExactly(1, 3);
```

这种声明式风格为我们提供了最少的冗长。然而，根据不同的用例，我们可能会发现其他更方便的方法。

### 3.4。使用流过滤

当深入到函数式/声明式编程的世界时，我们可以忘记改变集合，相反，我们可以专注于应该实际处理的元素:

```java
Collection<Integer> integers = newArrayList(1, 2, 3);

List<String> collected = integers
  .stream()
  .filter(i -> i != 2)
  .map(Object::toString)
  .collect(toList());

assertThat(collected).containsExactly("1", "3");
```

我们做了与上一个例子相反的事情，通过提供一个谓词来确定要包含而不是排除的元素。这样做的好处是，我们可以在删除的同时将其他功能链接在一起。在这个例子中，我们使用了一个函数`map(),` ,但是如果我们愿意，可以使用更多的操作。

## 4。结论

在本文中，我们展示了在迭代过程中从集合中移除项目时可能遇到的问题，并提供了一些解决方案。

这些例子的实现可以在 GitHub 的[中找到。这是一个 Maven 项目，所以应该很容易运行。](https://web.archive.org/web/20220928095038/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)