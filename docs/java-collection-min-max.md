# 查找列表或集合的最大值/最小值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collection-min-max>

## 1。概述

本教程简要介绍了如何使用 Java 8 中强大的`Stream` API 从给定的列表或集合中找到最小值和最大值。

## 2。在整数列表中查找 Max

我们可以使用通过`java.util.Stream`接口提供的`max()`方法，它接受一个方法引用:

```java
@Test
public void whenListIsOfIntegerThenMaxCanBeDoneUsingIntegerComparator() {
    // given
    List<Integer> listOfIntegers = Arrays.asList(1, 2, 3, 4, 56, 7, 89, 10);
    Integer expectedResult = 89;

    // then
    Integer max = listOfIntegers
      .stream()
      .mapToInt(v -> v)
      .max().orElseThrow(NoSuchElementException::new);

    assertEquals("Should be 89", expectedResult, max);
}
```

让我们仔细看看代码:

1.  在列表上调用`stream()`方法以从列表中获取值流
2.  在流上调用`mapToInt(value -> value)`以获得整数流
3.  在流上调用`max()`方法以获得最大值
4.  如果没有从`max()`收到值，调用`orElseThrow()`抛出异常

## 3。用自定义对象查找最小值

为了找到定制对象的最小值/最大值，我们还可以为我们首选的排序逻辑提供一个 lambda 表达式。

让我们首先定义自定义 POJO:

```java
class Person {
    String name;
    Integer age;

    // standard constructors, getters and setters
}
```

我们想找到最小年龄的`Person`对象:

```java
@Test
public void whenListIsOfPersonObjectThenMinCanBeDoneUsingCustomComparatorThroughLambda() {
    // given
    Person alex = new Person("Alex", 23);
    Person john = new Person("John", 40);
    Person peter = new Person("Peter", 32);
    List<Person> people = Arrays.asList(alex, john, peter);

    // then
    Person minByAge = people
      .stream()
      .min(Comparator.comparing(Person::getAge))
      .orElseThrow(NoSuchElementException::new);

    assertEquals("Should be Alex", alex, minByAge);
}
```

让我们来看看这个逻辑:

1.  在列表上调用`stream()`方法以从列表中获取值流
2.  在流上调用`min()`方法以获得最小值。我们传递一个 lambda 函数作为比较器，它用来决定决定最小值的排序逻辑。
3.  如果没有从`min()`收到值，调用`orElseThrow()`抛出异常

## 4。结论

在这篇简短的文章中，我们探索了如何使用 Java 8 `Stream` API 中的`max()`和`min()`方法来**从`List`或`Collection`中找到最大值和最小值。**

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221208143859/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list)