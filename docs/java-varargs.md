# Java 中的 Varargs

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-varargs>

## 1。简介

`Varargs`是在`Java 5`中引入的，它为支持任意数量的一种类型的参数的方法提供了一个简写。

在本文中，我们将了解如何使用这个核心 Java 特性。

## 2。`Varargs`之前

在 Java 5 之前，每当我们想要传递任意数量的参数时，我们必须传递一个数组中的所有参数或者实现 N 个方法(每个附加参数一个方法):

```java
public String format() { ... }

public String format(String value) { ... }

public String format(String val1, String val2) { ... }
```

## 3。`Varargs`使用

通过引入可以自动处理任意数量参数的新语法——使用幕后的数组，帮助我们避免编写样板代码。

我们可以使用标准的类型声明来定义它们，后面跟一个省略号:

```java
public String formatWithVarArgs(String... values) {
    // ...
}
```

现在，我们可以用任意数量的参数调用我们的方法，比如:

```java
formatWithVarArgs();

formatWithVarArgs("a", "b", "c", "d");
```

如前所述， **`varargs`是数组，所以我们需要像处理普通数组一样处理它们。**

## 4。规则

使用起来很简单。但是有几条规则我们必须记住:

*   每个方法只能有一个`varargs`参数
*   `varargs`参数必须是最后一个参数

## 5。堆积污染

**使用 `varargs`会导致所谓的[堆污染](https://web.archive.org/web/20221108113401/https://en.wikipedia.org/wiki/Heap_pollution)。**为了更好地理解堆污染，考虑这种`varargs`方法:

```java
static String firstOfFirst(List<String>... strings) {
    List<Integer> ints = Collections.singletonList(42);
    Object[] objects = strings;
    objects[0] = ints; // Heap pollution

    return strings[0].get(0); // ClassCastException
}
```

如果我们在测试中调用这个奇怪的方法:

```java
String one = firstOfFirst(Arrays.asList("one", "two"), Collections.emptyList());

assertEquals("one", one);
```

**我们会得到一个`ClassCastException `，即使我们在这里没有使用任何显式类型转换:**

```java
java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String
```

### 5.1.安全使用

**每次我们使用`varargs`，Java 编译器都会创建一个数组来保存给定的参数。**在这种情况下，编译器创建一个包含泛型类型组件的数组来保存参数。

当我们对泛型类型使用`varargs`时，由于存在致命运行时异常的潜在风险，Java 编译器警告我们可能存在不安全的`varargs`用法:

```java
warning: [varargs] Possible heap pollution from parameterized vararg type T
```

**`varargs`用法是安全的当且仅当:**

*   我们不在隐式创建的数组中存储任何东西。在这个例子中，我们在数组中存储了一个`List<Integer>`
*   我们不会让对生成的数组的引用逸出该方法(稍后将详细介绍)

**如果我们确定方法本身确实安全地使用了 varargs，我们可以使用 [`@SafeVarargs`](https://web.archive.org/web/20221108113401/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/SafeVarargs.html) 来取消警告。**

简而言之，如果我们使用它们从调用者向方法传递可变数量的参数，那么`varargs`的用法是安全的，仅此而已！

### 5.2.转义`Varargs`引用

让我们考虑`varargs`的另一种不安全用法:

```java
static <T> T[] toArray(T... arguments) {
    return arguments;
}
```

起初，`toArray `方法似乎是完全无害的。**但是，因为它让 varargs 数组转义为调用者，所以违反了第二条安全规则`varargs`** 。

为了了解这种方法有多危险，让我们在另一种方法中使用它:

```java
static <T> T[] returnAsIs(T a, T b) {
    return toArray(a, b);
}
```

如果我们调用这个方法:

```java
String[] args = returnAsIs("One", "Two");
```

我们会再次得到一个`ClassCastException. `这里是我们调用`returnAsIs `方法时发生的情况:

*   为了将`a `和`b `传递给`toArray `方法，Java 需要创建一个数组
*   因为`Object[] `可以保存任何类型的项目，所以编译器创建了一个
*   `toArray `方法将给定的`Object[] `返回给调用者
*   由于调用点期望一个`String[], `，编译器试图将`Object[] `转换为期望的`String[]`，因此得到了`ClassCastException`

关于堆污染的更详细的讨论，强烈推荐阅读 Joshua Bloch 的 [Effective Java 的第 32 项。](https://web.archive.org/web/20221108113401/https://learning.oreilly.com/library/view/effective-java-3rd/9780134686097/)

## 6。结论

在 Java 中可以让许多样板文件消失。

而且，由于它们与`Array,` 之间的隐式`autoboxing`，它们在我们的代码的未来检验中扮演了一个角色。

和往常一样，本文中的所有代码示例都可以在我们的 [GitHub 库](https://web.archive.org/web/20221108113401/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)中找到。