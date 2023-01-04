# 一行 Java 列表初始化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-init-list-one-line>

## 1.概观

在这个快速教程中，我们将研究如何使用一行程序初始化一个`List`。

## 延伸阅读:

## [Collections.emptyList()与新列表实例](/web/20220926193405/https://www.baeldung.com/java-collections-emptylist-new-list)

Learn the differences between the Collections.emptyList() and a new list instance.[Read more](/web/20220926193405/https://www.baeldung.com/java-collections-emptylist-new-list) →

## [Java 数组列表指南](/web/20220926193405/https://www.baeldung.com/java-arraylist)

Quick and practical guide to ArrayList in Java[Read more](/web/20220926193405/https://www.baeldung.com/java-arraylist) →

## 2.从数组创建

我们可以从数组中创建一个`List`。多亏了数组文字，我们可以在一行中初始化它们:

```java
List<String> list = Arrays.asList(new String[]{"foo", "bar"});
```

我们可以信任 varargs 机制来处理数组创建。有了它，我们可以编写更简洁、可读性更好的代码:

```java
@Test
public void givenArraysAsList_thenInitialiseList() {
    List<String> list = Arrays.asList("foo", "bar");

    assertTrue(list.contains("foo"));
}
```

**这段代码的结果实例实现了`List`接口，但它不是`java.util.ArrayList`或`LinkedList`。**相反，它是一个由原始数组支持的`List`，这有两个含义，我们将在本节的剩余部分中讨论。

虽然这个类的名字碰巧是`ArrayList`，但是它在`java.util.Arrays`包中。

### 2.1.固定大小

来自`Arrays.asList`的结果实例将具有固定的大小:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenArraysAsList_whenAdd_thenUnsupportedException() {
    List<String> list = Arrays.asList("foo", "bar");

    list.add("baz");
}
```

### 2.2.共享引用

原始数组和列表共享对对象的相同引用:

```java
@Test
public void givenArraysAsList_whenCreated_thenShareReference(){
    String[] array = {"foo", "bar"};
    List<String> list = Arrays.asList(array);
    array[0] = "baz";

    assertEquals("baz", list.get(0));
}
```

## 3.从流中创建(Java 8)

我们可以很容易地将一个`Stream`转换成任何类型的`Collection.`

因此，使用`Streams`的工厂方法，我们可以在一行中创建和初始化列表:

```java
@Test
public void givenStream_thenInitializeList(){
    List<String> list = Stream.of("foo", "bar")
      .collect(Collectors.toList());

    assertTrue(list.contains("foo"));
}
```

这里我们应该注意到，`Collectors.toList()`并不保证返回的`List`的准确实现。

没有关于返回实例的可变性、可串行化或线程安全性的通用契约。因此，我们的代码不应该依赖这些属性。

一些消息来源强调，`Stream.of(…).collect(…)`可能比`Arrays.asList()`拥有更大的内存和性能。但几乎在所有情况下，都是这样的微优化，差别不大。

## 4.工厂方法(Java 9)

JDK 9 为集合引入了几种方便的工厂方法:

```java
List<String> list = List.of("foo", "bar", "baz");
Set<String> set = Set.of("foo", "bar", "baz");
```

一个重要的细节是返回的实例是不可变的。除此之外，工厂方法在空间效率和线程安全方面有几个优势。

这个话题在本文的[中有更多的探讨。](/web/20220926193405/https://www.baeldung.com/java-9-collections-factory-methods)

## 5.双括号初始化

在几个地方，我们可以找到一个叫做双大括号初始化的方法，它看起来像这样:

```java
@Test
public void givenAnonymousInnerClass_thenInitialiseList() {
    List<String> cities = new ArrayList() {{
        add("New York");
        add("Rio");
        add("Tokyo");
    }};

    assertTrue(cities.contains("New York"));
}
```

“双括号初始化”这个名称很容易引起误解。虽然语法看起来简洁优雅，但它危险地隐藏了实际情况。

Java 中实际上没有双括号语法元素；这是两个故意以这种方式格式化的块。

**使用外部括号，我们声明一个匿名的内部类，它将是`ArrayList`的子类。**我们可以在这些括号中声明子类的细节。

像往常一样，我们可以使用实例初始化程序块，这就是内部一对大括号的来源。

这种语法的简洁很吸引人。然而，它被认为是一种反模式。

要阅读更多关于双括号初始化的内容，请看我们的文章[这里](/web/20220926193405/https://www.baeldung.com/java-double-brace-initialization)。

## 6.结论

**现代 Java 提供了几个选项来在一行中创建一个`Collection`。**我们选择的方法几乎完全取决于个人偏好，而不是技术推理。

重要的一点是，尽管看起来很优雅，**匿名内部类初始化的反模式(又名双括号)有许多负面影响。**

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220926193405/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2)