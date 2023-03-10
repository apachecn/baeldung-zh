# Java 接口中的私有方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-interface-private-methods>

## 1.概观

从 Java 9 开始，[私有方法](https://web.archive.org/web/20220525011840/https://openjdk.java.net/jeps/213)可以添加到 Java 的接口中。在这个简短的教程中，让我们讨论如何定义这些方法以及它们的好处。

## 2.在接口中定义私有方法

**私有方法可以被实现`static`或非`static`。**这意味着在一个接口中，我们能够创建私有方法来封装来自`default`和`static` 公共方法签名的代码。

首先，让我们看看如何使用默认接口方法中的私有方法:

```java
public interface Foo {

    default void bar() {
        System.out.print("Hello");
        baz();
    }

    private void baz() {
        System.out.println(" world!");
    }
}
```

`bar()`能够通过从它的默认方法调用私有方法`baz()`来使用它。

接下来，让我们向我们的`Foo`接口添加一个静态定义的私有方法:

```java
public interface Foo {

    static void buzz() {
        System.out.print("Hello");
        staticBaz();
    }

    private static void staticBaz() {
        System.out.println(" static world!");
    }
}
```

在接口中，其他静态定义的方法可以利用这些私有静态方法。

最后，让我们从一个具体的类中调用已定义的默认和静态方法:

```java
public class CustomFoo implements Foo {

    public static void main(String... args) {
        Foo customFoo = new CustomFoo();
        customFoo.bar();
        Foo.buzz();
    }
}
```

输出是字符串“Hello world！”从调用到`bar()`方法和“你好静态世界！”从对`buzz()` 方法的调用。

## 3.接口中私有方法的好处

既然我们已经定义了私有方法，我们就来谈谈私有方法的好处。

在前一节中提到，接口能够使用私有方法对实现接口的类隐藏实现细节。因此，在接口中包含这些的主要好处之一就是封装。

另一个好处是(和一般的私有方法一样),为具有相似功能的方法的接口增加了更少的重复和更多的可重用代码。

## 4.结论

在本教程中，我们已经介绍了如何在接口中定义私有方法，以及如何在静态和非静态上下文中使用它们。我们在本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220525011840/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9)