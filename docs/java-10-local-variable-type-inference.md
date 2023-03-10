# Java 10 局部变量类型推理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-10-local-variable-type-inference>

[This article is part of a series:](javascript:void(0);)• Java 10 LocalVariable Type-Inference (current article)[• Java 10 Performance Improvements](/web/20220627144008/https://www.baeldung.com/java-10-performance-improvements)
[• New Features in Java 10](/web/20220627144008/https://www.baeldung.com/java-10-overview)

## 1.概观

JDK 10 中最明显的改进之一是使用初始化器对局部变量进行类型推断。

本教程通过示例提供了该特性的详细信息。

## 2.介绍

在 Java 9 之前，我们必须明确提到局部变量的类型，并确保它与用于初始化它的初始化器兼容:

```java
String message = "Good bye, Java 9";
```

在 Java 10 中，我们可以这样声明一个局部变量:

```java
@Test
public void whenVarInitWithString_thenGetStringTypeVar() {
    var message = "Hello, Java 10";
    assertTrue(message instanceof String);
}
```

**我们不提供`message`的数据类型。相反，我们将`the message `标记为`var`，编译器从右边出现的初始化器的类型推断出`message `的类型。**

在上面的例子中，`message `的类型应该是`String`。

**注意，这个特性只适用于带有初始化器的局部变量。**它不能用于成员变量、方法参数、返回类型等——初始化器是必需的，因为没有它编译器将不能推断类型。

这种增强有助于减少样板代码；例如:

```java
Map<Integer, String> map = new HashMap<>();
```

这现在可以重写为:

```java
var idToNameMap = new HashMap<Integer, String>();
```

这也有助于关注变量名而不是变量类型。

另一件要注意的事情是 **`var `不是关键字**——这确保了使用`var `作为函数或变量名的程序的向后兼容性。`var`是一个保留类型名称，就像`int`一样。

最后，注意使用`var `没有运行时开销，也没有使 Java 成为动态类型语言。变量的类型仍然是在编译时推断出来的，以后不能更改。

## 3.非法使用`var`

如前所述，`var `没有初始化器就无法工作:

```java
var n; // error: cannot use 'var' on variable without initializer
```

如果用`null`初始化，它也不会工作:

```java
var emptyList = null; // error: variable initializer is 'null'
```

它不适用于非局部变量:

```java
public var = "hello"; // error: 'var' is not allowed here
```

Lambda 表达式需要显式目标类型，因此不能使用`var `:

```java
var p = (String s) -> s.length() > 10; // error: lambda expression needs an explicit target-type
```

数组初始值设定项也是如此:

```java
var arr = { 1, 2, 3 }; // error: array initializer needs an explicit target-type
```

## 4。`var`使用指南

有些情况下可以合法使用`var `,但这样做可能不是一个好主意。

例如，在代码可读性变差的情况下:

```java
var result = obj.prcoess();
```

这里，虽然合法使用了`var`，但是理解由`process()`返回的类型变得很困难，使得代码可读性更差。

[java.net](https://web.archive.org/web/20220627144008/https://openjdk.java.net/)有一篇关于 Java 中局部变量类型推理的[风格指南的专门文章，讲述了我们在使用这个特性时应该如何使用判断。](https://web.archive.org/web/20220627144008/https://openjdk.java.net/projects/amber/guides/lvti-style-guide)

最好避免`var `的另一种情况是在具有长管道的流中:

```java
var x = emp.getProjects.stream()
  .findFirst()
  .map(String::length)
  .orElse(0);
```

使用`var `也可能产生意想不到的结果。

例如，如果我们将它与 Java 7 中引入的菱形运算符一起使用:

```java
var empList = new ArrayList<>();
```

`empList`的类型将是`ArrayList<Object>`而不是`List<Object>`。如果我们想让它成为`ArrayList<Employee>`，我们必须明确:

```java
var empList = new ArrayList<Employee>();
```

**对不可命名类型使用`var `可能会导致意外错误。**

例如，如果我们对匿名类实例使用`var `:

```java
@Test
public void whenVarInitWithAnonymous_thenGetAnonymousType() {
    var obj = new Object() {};
    assertFalse(obj.getClass().equals(Object.class));
}
```

现在，如果我们试图将另一个`Object`赋值给`obj`，我们会得到一个编译错误:

```java
obj = new Object(); // error: Object cannot be converted to <anonymous Object>
```

这是因为`obj `的推断类型不是`Object`。

## 5.结论

在本文中，我们看到了新的 Java 10 局部变量类型推断特性和示例。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220627144008/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-10)

Next **»**[Java 10 Performance Improvements](/web/20220627144008/https://www.baeldung.com/java-10-performance-improvements)