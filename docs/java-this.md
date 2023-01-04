# this Java 关键字指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-this>

## 1。简介

在本教程中，**我们将看看*这个* Java 关键字。**

在 Java 中， **`this`关键字是对当前对象的引用，该对象的方法被调用**。

让我们探讨一下如何以及何时使用关键字。

## 2。消除字段阴影的歧义

**关键字有助于从局部参数**中区分实例变量。最常见的原因是当我们的构造函数参数与实例字段同名时:

```java
public class KeywordTest {

    private String name;
    private int age;

    public KeywordTest(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

正如我们在这里看到的，我们将`this`与`name`和`age`实例字段一起使用——以将它们与参数区分开来。

另一种用法是使用`this`在局部范围内隐藏或隐藏参数。一个使用的例子可以在[变量和方法隐藏](/web/20220625173413/https://www.baeldung.com/java-variable-method-hiding)文章中找到。

## 3。引用同一个类的构造函数

**从一个构造函数中，我们可以使用`this()`来调用同一个类的不同构造函数**。这里，我们将`this()`用于构造函数链接，以减少代码的使用。

最常见的用例是从参数化构造函数中调用默认构造函数:

```java
public KeywordTest(String name, int age) {
    this();

    // the rest of the code
}
```

或者，我们可以从无参数构造函数调用参数化构造函数，并传递一些参数:

```java
public KeywordTest() {
    this("John", 27);
}
```

注意，`this()`应该是构造函数中的第一条语句，否则会出现编译错误。

## 4。将`this`作为参数传递

这里我们有`printInstance()`方法，其中定义了`this Keyword`参数:

```java
public KeywordTest() {
    printInstance(this);
}

public void printInstance(KeywordTest thisKeyword) {
    System.out.println(thisKeyword);
}
```

在构造函数内部，我们调用`printInstance()`方法。使用`this`，我们传递一个对当前实例的引用。

## 5。返回`this`

**我们也可以使用`this`关键字从方法中返回当前的类实例**。

为了不重复代码，这里有一个完整的实际例子，说明如何在[构建器设计模式](/web/20220625173413/https://www.baeldung.com/creational-design-patterns)中实现它。

## 6。内部类中的`this`关键字

我们还使用`this`从内部类中访问外部类实例:

```java
public class KeywordTest {

    private String name;

    class ThisInnerClass {

        boolean isInnerClass = true;

        public ThisInnerClass() {
            KeywordTest thisKeyword = KeywordTest.this;
            String outerString = KeywordTest.this.name;
        }
    }
}
```

这里，在构造函数内部，我们可以通过调用*得到对`KeywordTest`实例的引用。*我们可以更深入，访问像`KeywordTest.this.name `字段这样的实例变量。

## 7。结论

在本文中，我们探索了 Java 中的关键字`this`。

和往常一样，完整的代码可以在 Github 的[上找到。](https://web.archive.org/web/20220625173413/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types)