# 超级 Java 关键字指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-super>

## 1。简介

在这个快速教程中，**我们将看看*超级* Java 关键字。**

**简单来说，我们可以使用`super`关键字来访问父类。**

让我们探索核心关键字在语言中的应用。

## 2。带有构造函数的`super`关键字

**我们可以用 `super()`调用父默认构造函数**。它应该是构造函数中的第一条语句。

在我们的例子中，我们将`super(message) `与`String`参数一起使用:

```java
public class SuperSub extends SuperBase {

    public SuperSub(String message) {
        super(message);
    }
}
```

让我们创建一个子类实例，看看后面发生了什么:

```java
SuperSub child = new SuperSub("message from the child class");
```

`new`关键字调用`SuperSub`的构造函数，该构造函数本身首先调用父构造函数，并将`String`参数传递给它。

## 3。访问父类变量

让我们用`message`实例变量创建一个父类:

```java
public class SuperBase {
    String message = "super class";

    // default constructor

    public SuperBase(String message) {
        this.message = message;
    }
}
```

现在，我们用同名变量创建一个子类:

```java
public class SuperSub extends SuperBase {

    String message = "child class";

    public void getParentMessage() {
        System.out.println(super.message);
    }
}
```

我们可以使用`super`关键字从子类中访问父变量。

## 4。方法覆盖了的`super`关键字

在继续之前，我们建议先回顾一下我们的[方法，而不是](/web/20221004023336/https://www.baeldung.com/java-method-overload-override)指南。

让我们向父类添加一个实例方法:

```java
public class SuperBase {

    String message = "super class";

    public void printMessage() {
        System.out.println(message);
    }
}
```

并在我们的子类中覆盖`printMessage()`方法:

```java
public class SuperSub extends SuperBase {

    String message = "child class";

    public SuperSub() {
        super.printMessage();
        printMessage();
    }

    public void printMessage() {
        System.out.println(message);
    }
}
```

**我们可以使用`super`从子类**中访问被覆盖的方法。构造函数中的`super.printMessage()`从`SuperBase`调用父方法。

## 5。结论

在本文中，我们探索了*超级*关键字。

和往常一样，完整的代码可以在 Github 的[上找到。](https://web.archive.org/web/20221004023336/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-inheritance)