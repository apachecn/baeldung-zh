# Java 编译器错误:“需要类、接口或枚举”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-class-interface-enum-expected>

## 1。概述

在这个快速教程中，我们将讨论 java 编译器错误`“class, interface, or enum expected”.`。这个错误主要是由刚接触 Java 世界的开发人员面临的。

让我们看几个这种错误的例子，并讨论如何修复它们。

## 2。错位的花括号

`“class, interface, or enum expected”`错误的根本原因通常是放错了花括号` “}”`。这可以是课后额外的花括号。也可能是意外在类外编写的方法。

让我们看一个例子:

```java
public class MyClass {
    public static void main(String args[]) {
      System.out.println("Baeldung");
    }
}
}
```

```java
/MyClass.java:6: error: class, interface, or enum expected
}
^
1 error
```

在上面的代码示例中，最后一行多了一个`“}”`花括号，这会导致编译错误。如果我们移除它，那么代码将会编译。

让我们看看发生此错误的另一个场景:

```java
public class MyClass {
    public static void main(String args[]) {
        //Implementation
    }
}
public static void printHello() {
    System.out.println("Hello");
}
```

```java
/MyClass.java:6: error: class, interface, or enum expected
public static void printHello()
^
/MyClass.java:8: error: class, interface, or enum expected
}
^
2 errors
```

在上面的例子中，我们会得到错误，因为方法`printHello()`在类`MyClass`之外。我们可以通过将右花括号`“}”`移到文件末尾来解决这个问题。换句话说，将`printHello()`方法移到`MyClass`中。

## 3。结论

在这篇简短的教程中，我们讨论了“预期的类、接口或枚举”Java 编译器错误，并展示了两个可能的根本原因。