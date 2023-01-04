# Java 8——与 Lambdas 的强大对比

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-sort-lambda>

## 1。概述

在本教程中，我们将首先看看 Java 8 中的 **Lambda 支持，特别是如何利用它来编写`Comparator`和对集合**进行排序。

本文是 Baeldung 网站上的[“Java—回到基础”系列文章](/web/20220703150536/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 延伸阅读:

## [Java 8 Stream API 教程](/web/20220703150536/https://www.baeldung.com/java-8-streams)

The article is an example-heavy introduction of the possibilities and operations offered by the Java 8 Stream API.[Read more](/web/20220703150536/https://www.baeldung.com/java-8-streams) →

## [Java 8 的收集器指南](/web/20220703150536/https://www.baeldung.com/java-8-collectors)

The article discusses Java 8 Collectors, showing examples of built-in collectors, as well as showing how to build custom collector.[Read more](/web/20220703150536/https://www.baeldung.com/java-8-collectors) →

## [Lambda 表达式和函数接口:技巧和最佳实践](/web/20220703150536/https://www.baeldung.com/java-8-lambda-expressions-tips)

Tips and best practices on using Java 8 lambdas and functional interfaces.[Read more](/web/20220703150536/https://www.baeldung.com/java-8-lambda-expressions-tips) →

首先，让我们定义一个简单的实体类:

```java
public class Human {
    private String name;
    private int age;

    // standard constructors, getters/setters, equals and hashcode
} 
```

## 2。不带 Lambdas 的基本排序

在 Java 8 之前，对集合进行排序需要**为排序中使用的`Comparator`** 创建一个匿名内部类:

```java
new Comparator<Human>() {
    @Override
    public int compare(Human h1, Human h2) {
        return h1.getName().compareTo(h2.getName());
    }
}
```

这将简单地用于对`Human`实体的`List`进行排序:

```java
@Test
public void givenPreLambda_whenSortingEntitiesByName_thenCorrectlySorted() {
    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 10), 
      new Human("Jack", 12)
    );

    Collections.sort(humans, new Comparator<Human>() {
        @Override
        public int compare(Human h1, Human h2) {
            return h1.getName().compareTo(h2.getName());
        }
    });
    Assert.assertThat(humans.get(0), equalTo(new Human("Jack", 12)));
}
```

## 3。支持 Lambda 的基本排序

随着 Lambdas 的引入，我们现在可以绕过匿名内部类，用**简单的函数语义**获得相同的结果:

```java
(final Human h1, final Human h2) -> h1.getName().compareTo(h2.getName());
```

类似地，我们现在可以像以前一样测试行为:

```java
@Test
public void whenSortingEntitiesByName_thenCorrectlySorted() {
    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 10), 
      new Human("Jack", 12)
    );

    humans.sort(
      (Human h1, Human h2) -> h1.getName().compareTo(h2.getName()));

    assertThat(humans.get(0), equalTo(new Human("Jack", 12)));
}
```

注意，我们还使用了 Java 8 中添加到`java.util.List`的新的`sort` API，而不是旧的`Collections.sort` API。

## 4。没有类型定义的基本排序

我们可以通过不指定类型定义来进一步简化表达式；**编译器能够自己推断出这些**:

```java
(h1, h2) -> h1.getName().compareTo(h2.getName())
```

同样，测试仍然非常相似:

```java
@Test
public void 
  givenLambdaShortForm_whenSortingEntitiesByName_thenCorrectlySorted() {

    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 10), 
      new Human("Jack", 12)
    );

    humans.sort((h1, h2) -> h1.getName().compareTo(h2.getName()));

    assertThat(humans.get(0), equalTo(new Human("Jack", 12)));
}
```

## 5。使用对静态方法的引用进行排序

接下来，我们将使用一个引用静态方法的 Lambda 表达式来执行排序。

首先，我们将使用与`Comparator<Human>`对象中的`compare`方法完全相同的签名来定义方法`compareByNameThenAge`:

```java
public static int compareByNameThenAge(Human lhs, Human rhs) {
    if (lhs.name.equals(rhs.name)) {
        return Integer.compare(lhs.age, rhs.age);
    } else {
        return lhs.name.compareTo(rhs.name);
    }
}
```

然后我们将用这个引用调用`humans.sort`方法:

```java
humans.sort(Human::compareByNameThenAge);
```

最终结果是使用静态方法作为`Comparator`对集合进行有效排序:

```java
@Test
public void 
  givenMethodDefinition_whenSortingEntitiesByNameThenAge_thenCorrectlySorted() {

    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 10), 
      new Human("Jack", 12)
    );

    humans.sort(Human::compareByNameThenAge);
    Assert.assertThat(humans.get(0), equalTo(new Human("Jack", 12)));
}
```

## 6。对提取的比较器进行排序

通过使用一个**实例方法引用**和`Comparator.comparing`方法，我们甚至可以避免定义比较逻辑本身，该方法基于那个函数提取并创建一个`Comparable`。

我们将使用 getter `getName()`来构建 Lambda 表达式，并按名称对列表进行排序:

```java
@Test
public void 
  givenInstanceMethod_whenSortingEntitiesByName_thenCorrectlySorted() {

    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 10), 
      new Human("Jack", 12)
    );

    Collections.sort(
      humans, Comparator.comparing(Human::getName));
    assertThat(humans.get(0), equalTo(new Human("Jack", 12)));
}
```

## 7。反向排序

JDK 8 还引入了一个助手方法用于**反转比较器。**我们可以快速利用这一点来反转我们的排序:

```java
@Test
public void whenSortingEntitiesByNameReversed_thenCorrectlySorted() {
    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 10), 
      new Human("Jack", 12)
    );

    Comparator<Human> comparator
      = (h1, h2) -> h1.getName().compareTo(h2.getName());

    humans.sort(comparator.reversed());

    Assert.assertThat(humans.get(0), equalTo(new Human("Sarah", 10)));
}
```

## 8。多条件排序

比较 lambda 表达式不需要这么简单。我们还可以编写**更复杂的表达式，**例如先按名称，然后按年龄对实体进行排序:

```java
@Test
public void whenSortingEntitiesByNameThenAge_thenCorrectlySorted() {
    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 12), 
      new Human("Sarah", 10), 
      new Human("Zack", 12)
    );

    humans.sort((lhs, rhs) -> {
        if (lhs.getName().equals(rhs.getName())) {
            return Integer.compare(lhs.getAge(), rhs.getAge());
        } else {
            return lhs.getName().compareTo(rhs.getName());
        }
    });
    Assert.assertThat(humans.get(0), equalTo(new Human("Sarah", 10)));
}
```

## 9。多条件排序–合成

同样的比较逻辑，先按姓名排序，再按年龄排序，也可以通过对`Comparator`的新构图支持来实现。

**从 JDK 8 开始，我们现在可以将多个比较器**链接在一起，构建更复杂的比较逻辑:

```java
@Test
public void 
  givenComposition_whenSortingEntitiesByNameThenAge_thenCorrectlySorted() {

    List<Human> humans = Lists.newArrayList(
      new Human("Sarah", 12), 
      new Human("Sarah", 10), 
      new Human("Zack", 12)
    );

    humans.sort(
      Comparator.comparing(Human::getName).thenComparing(Human::getAge)
    );

    Assert.assertThat(humans.get(0), equalTo(new Human("Sarah", 10)));
}
```

## 10。用`Stream.sorted()` 排序列表

**我们还可以使用 Java 8 的`Stream` `sorted()` API 对集合进行排序。**

我们可以使用自然排序以及由`Comparator.`提供的排序来对流进行排序。为此，我们有两个`sorted()` API 的重载变体:

*   `sort` `ed()` **`–`** 使用自然排序对`Stream`的元素进行排序；元素类必须实现`Comparable`接口。
*   `sorted(Comparator``<?` `super` `T>` `compa``rator)`–根据`Comparator`实例对元素进行排序

让我们来看一个如何使用自然排序的**方法的例子:**

```java
@Test
public final void 
  givenStreamNaturalOrdering_whenSortingEntitiesByName_thenCorrectlySorted() {
    List<String> letters = Lists.newArrayList("B", "A", "C");

    List<String> sortedLetters = letters.stream().sorted().collect(Collectors.toList());
    assertThat(sortedLetters.get(0), equalTo("A"));
}
```

现在让我们看看如何通过`sorted()` API 来**使用自定义的`Comparator `:**

```java
@Test
public final void 
  givenStreamCustomOrdering_whenSortingEntitiesByName_thenCorrectlySorted() {	
    List<Human> humans = Lists.newArrayList(new Human("Sarah", 10), new Human("Jack", 12));
    Comparator<Human> nameComparator = (h1, h2) -> h1.getName().compareTo(h2.getName());

    List<Human> sortedHumans = 
      humans.stream().sorted(nameComparator).collect(Collectors.toList());
    assertThat(sortedHumans.get(0), equalTo(new Human("Jack", 12)));
}
```

如果我们**使用`Comparator.comparing()`方法**，我们可以进一步简化上面的例子:

```java
@Test
public final void 
  givenStreamComparatorOrdering_whenSortingEntitiesByName_thenCorrectlySorted() {
    List<Human> humans = Lists.newArrayList(new Human("Sarah", 10), new Human("Jack", 12));

    List<Human> sortedHumans = humans.stream()
      .sorted(Comparator.comparing(Human::getName))
      .collect(Collectors.toList());

    assertThat(sortedHumans.get(0), equalTo(new Human("Jack", 12)));
}
```

## 11.用`Stream.sorted()`对列表进行反向排序

**我们也可以使用`Stream.sorted()`对集合进行反向排序。**

首先，让我们来看一个例子，看看**如何将`sorted()`** **方法与`Comparator.reverseOrder()`方法结合起来，以相反的自然顺序**对列表进行排序:

```java
@Test
public final void 
  givenStreamNaturalOrdering_whenSortingEntitiesByNameReversed_thenCorrectlySorted() {
    List<String> letters = Lists.newArrayList("B", "A", "C");

    List<String> reverseSortedLetters = letters.stream()
      .sorted(Comparator.reverseOrder())
      .collect(Collectors.toList());

    assertThat(reverseSortedLetters.get(0), equalTo("C"));
}
```

现在让我们看看**如何使用`sorted()`方法和一个自定义的`Comparator`** :

```java
@Test
public final void 
  givenStreamCustomOrdering_whenSortingEntitiesByNameReversed_thenCorrectlySorted() {
    List<Human> humans = Lists.newArrayList(new Human("Sarah", 10), new Human("Jack", 12));
    Comparator<Human> reverseNameComparator = 
      (h1, h2) -> h2.getName().compareTo(h1.getName());

    List<Human> reverseSortedHumans = humans.stream().sorted(reverseNameComparator)
      .collect(Collectors.toList());
    assertThat(reverseSortedHumans.get(0), equalTo(new Human("Sarah", 10)));
}
```

注意，`compareTo` 的调用是翻转的，负责反转。

最后，让我们通过使用`Comparator.comparing()`方法的**来简化上面的例子:**

```java
@Test
public final void 
  givenStreamComparatorOrdering_whenSortingEntitiesByNameReversed_thenCorrectlySorted() {
    List<Human> humans = Lists.newArrayList(new Human("Sarah", 10), new Human("Jack", 12));

    List<Human> reverseSortedHumans = humans.stream()
      .sorted(Comparator.comparing(Human::getName, Comparator.reverseOrder()))
      .collect(Collectors.toList());

    assertThat(reverseSortedHumans.get(0), equalTo(new Human("Sarah", 10)));
}
```

## 12.空值

到目前为止，我们实现了我们的`Comparator` s，使得它们不能对包含`null `值的集合进行排序。也就是说，如果集合包含至少一个`null `元素，那么`sort `方法抛出一个`NullPointerException`:

```java
@Test(expected = NullPointerException.class)
public void givenANullElement_whenSortingEntitiesByName_thenThrowsNPE() {
    List<Human> humans = Lists.newArrayList(null, new Human("Jack", 12));

    humans.sort((h1, h2) -> h1.getName().compareTo(h2.getName()));
}
```

最简单的解决方案是在我们的`Comparator `实现中手动处理`null `值:

```java
@Test
public void givenANullElement_whenSortingEntitiesByNameManually_thenMovesTheNullToLast() {
    List<Human> humans = Lists.newArrayList(null, new Human("Jack", 12), null);

    humans.sort((h1, h2) -> {
        if (h1 == null) {
            return h2 == null ? 0 : 1;
        }
        else if (h2 == null) {
            return -1;
        }
        return h1.getName().compareTo(h2.getName());
    });

    Assert.assertNotNull(humans.get(0));
    Assert.assertNull(humans.get(1));
    Assert.assertNull(humans.get(2));
}
```

在这里，我们将所有的`null `元素都推到了集合的末尾。为此，比较器认为`null`大于非空值。当两者都是`null`时，他们被认为是平等的。

此外，**我们可以将任何非空安全的`Comparator `传递给`[Comparator.nullsLast()](https://web.archive.org/web/20220703150536/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#nullsLast(java.util.Comparator)) `方法，并获得相同的结果**:

```java
@Test
public void givenANullElement_whenSortingEntitiesByName_thenMovesTheNullToLast() {
    List<Human> humans = Lists.newArrayList(null, new Human("Jack", 12), null);

    humans.sort(Comparator.nullsLast(Comparator.comparing(Human::getName)));

    Assert.assertNotNull(humans.get(0));
    Assert.assertNull(humans.get(1));
    Assert.assertNull(humans.get(2));
}
```

类似地，我们可以使用`[Comparator.nullsFirst()](https://web.archive.org/web/20220703150536/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#nullsFirst(java.util.Comparator)) `将`null `元素移向集合的开头:

```java
@Test
public void givenANullElement_whenSortingEntitiesByName_thenMovesTheNullToStart() {
    List<Human> humans = Lists.newArrayList(null, new Human("Jack", 12), null);

    humans.sort(Comparator.nullsFirst(Comparator.comparing(Human::getName)));

    Assert.assertNull(humans.get(0));
    Assert.assertNull(humans.get(1));
    Assert.assertNotNull(humans.get(2));
} 
```

**强烈推荐使用`nullsFirst() `或`nullsLast() `装饰器，因为它们更灵活，可读性更好。**

## 13。结论

本文展示了使用 Java 8 Lambda 表达式对**列表进行排序的各种令人兴奋的方法，**越过了语法的限制，进入了真正强大的函数语义。

所有这些例子和代码片段的实现都可以在 GitHub 的[中找到。](https://web.archive.org/web/20220703150536/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas "Java 8 - Lambda Expressions for Comparison examples")