# Java 中字符串的排列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-permutations>

## 1.介绍

排列是集合中元素的重新排列。换句话说，它是收集顺序的所有可能的变化。

在本教程中，我们将学习如何使用第三方库在 Java 中轻松创建[排列。更具体地说，我们将处理字符串中的排列。](/web/20221008120408/https://www.baeldung.com/java-array-permutations)

## 2.排列

有时我们需要检查一个字符串值的所有可能的排列。通常用于令人难以置信的在线编码练习，较少用于日常工作任务。比如一个字符串“abc”，里面的字符会有六种不同的排列方式:“abc”、“acb”、“cab”、“bac”、“bca”、“cba”。

几个定义良好的算法可以帮助我们为特定的`String`值创建所有可能的排列。比如最著名的就是 Heap 的算法。然而，它非常复杂且不直观。除此之外，递归方法使事情变得更糟。

## 3.优雅的解决方案

实现生成排列的算法需要编写定制的逻辑。在实现过程中很容易出错，并且随着时间的推移很难测试它是否正常工作。还有，重写之前写的东西也没有意义。

此外，使用`String`值时，如果不小心的话，创建太多的实例可能会淹没`String`池。

以下是目前提供此类功能的库:

*   Apache common(Apache 公共)
*   番石榴
*   组合库

让我们尝试使用这些库找到一个字符串值的所有排列。我们将 **关注这些库是否允许对排列进行惰性遍历，以及它们如何处理输入值中的重复项。**

我们将在下面的例子中使用一个`Helper.toCharacterList`方法。这个方法封装了将一个`String`转换成`Characters`的`List`的复杂性:

```java
static List<Character> toCharacterList(final String string) {
    return string.chars().mapToObj(s -> ((char) s)).collect(Collectors.toList());
}
```

此外，我们将使用一个助手方法将`Characters`的`List `转换为`String`:

```java
static String toString(Collection<Character> collection) {
    return collection.stream().map(s -> s.toString()).collect(Collectors.joining());
}
```

## 4.Apache common(Apache 公共)

首先，让我们将 Maven 依赖项`[commons-collections4](https://web.archive.org/web/20221008120408/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-collections4)`添加到项目中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

总的来说，Apache 提供了一个简单的 API。 **`CollectionUtils` 急切地创建排列，所以我们在使用长`String`值**时要小心:

```java
public List<String> eagerPermutationWithRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    return CollectionUtils.permutations(characters)
        .stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

**同时，为了让它以一种懒惰的方式工作，我们应该使用`PermutationIterator` :**

```java
public List<String> lazyPermutationWithoutRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    final PermutationIterator<Character> permutationIterator = new PermutationIterator<>(characters);
    final List<String> result = new ArrayList<>();
    while (permutationIterator.hasNext()) {
        result.add(Helper.toString(permutationIterator.next()));
    }
    return result;
}
```

**这个库不处理副本，所以`String`“aaaaaa”将产生 720 种排列，这通常是不可取的。**同样，`PermutationIterator `也没有得到排列个数的方法。在这种情况下，我们应该根据输入大小分别计算它们。

## 5.番石榴

首先，让我们将[番石榴库](https://web.archive.org/web/20221008120408/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava)的 Maven 依赖项添加到项目中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

番石榴允许用`Collections2`创造排列。该 API 易于使用:

```java
public List<String> permutationWithRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    return Collections2.permutations(characters).stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

`Collections2.permutations`的结果是一个`PermutationCollection`，它允许容易地访问排列。**所有的排列都是懒洋洋地创造出来的。**

**此外，这个类提供了一个创建无重复排列的 API:**

```java
public List<String> permutationWithoutRepetitions(final String string) {
    final List<Character> characters = Helper.toCharacterList(string);
    return Collections2.orderedPermutations(characters).stream()
        .map(Helper::toString)
        .collect(Collectors.toList());
}
```

然而，这些方法的问题是它们用 [`@Beta`注释](https://web.archive.org/web/20221008120408/https://guava.dev/releases/18.0/api/docs/com/google/common/annotations/Beta.html)进行了注释，这不能保证这个 API 在未来的版本中不会改变。

## 6.组合库

为了在项目中使用它，让我们添加`[combinatoricslib3](https://web.archive.org/web/20221008120408/https://search.maven.org/search?q=g:com.github.dpaukov%20AND%20a:combinatoricslib3)` Maven 依赖项:

```java
<dependency>
    <groupId>com.github.dpaukov</groupId>
    <artifactId>combinatoricslib3</artifactId>
    <version>3.3.3</version>
</dependency>
```

虽然这是一个小库，但它提供了许多组合学工具，包括排列。API 本身非常直观，并且利用了 Java 流。让我们从一个特定的字符串或一个`List`字符创建排列:

```java
public List<String> permutationWithoutRepetitions(final String string) {
    List<Character> chars = Helper.toCharacterList(string);
    return Generator.permutation(chars)
      .simple()
      .stream()
      .map(Helper::toString)
      .collect(Collectors.toList());
}
```

上面的代码创建了一个生成器，它将为字符串提供排列。排列将被缓慢地检索。因此，我们只创建了一个生成器，并计算了预期的排列数。

同时，有了这个库，我们可以确定复制的策略。如果我们使用字符串“aaaaaa”作为例子，我们将只得到一个而不是 720 个相同的排列。

```java
public List<String> permutationWithRepetitions(final String string) {
    List<Character> chars = Helper.toCharacterList(string);
    return Generator.permutation(chars)
      .simple(TreatDuplicatesAs.IDENTICAL)
      .stream()
      .map(Helper::toString)
      .collect(Collectors.toList());
}
```

`TreatDuplicatesAs`允许我们定义如何处理重复项。

## 7.结论

有很多方法可以处理组合学，尤其是排列。所有这些库在这方面都有很大的帮助。值得尝试所有这些方法，并决定哪一种适合您的需求。尽管许多人有时会强烈要求编写他们所有的代码，但在已经存在并提供良好功能的东西上浪费时间是没有意义的。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221008120408/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-4)