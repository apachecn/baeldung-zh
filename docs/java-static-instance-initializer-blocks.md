# Java 中的静态与实例初始化程序块

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-static-instance-initializer-blocks>

## 1。概述

在本教程中，我们将学习静态块和实例初始化块的概念。我们还将检查类[构造函数](/web/20220915120117/https://www.baeldung.com/java-constructors)和初始化程序块的差异和执行顺序。

## 2。静态块

在 Java 中，**一个静态块在对象初始化**之前执行代码。静态块是带有 [`static`](/web/20220915120117/https://www.baeldung.com/java-static) 关键字的代码块:

```java
static {
    // definition of the static block
}
```

静态初始化块或静态初始化块，或静态子句是静态块的一些其他名称。**静态块代码在类加载期间只执行一次**。在 Java 中，静态块总是在`main()`方法之前首先执行，因为编译器在类加载时和对象创建之前将它们存储在内存中。

一个类可以有多个静态块，它们将按照在类中出现的顺序执行:

```java
public class StaticBlockExample {

    static {
        System.out.println("static block 1");
    }

    static {
        System.out.println("static block 2");
    }

    public static void main(String[] args) {
        System.out.println("Main Method");
    }
}
```

上述代码片段的输出是:

```java
static block 1
static block 2
Main Method
```

这里，编译器先执行所有的静态块，在完成静态块执行后，调用`main()`方法。Java 编译器确保静态初始化块的执行顺序与它们在源代码中出现的顺序相同。

父类的静态块首先执行，因为编译器在子类之前加载父类。

出于好奇，在 Java 1.7 之前，main()方法并不是每个 Java 应用程序都必须使用的，所以所有代码都可以写在静态块中。然而，从 Java 1.7 开始，main()方法是强制性的。

## 3。实例初始化程序块

顾名思义，**实例初始化器块的目的是初始化实例数据成员。**

实例初始化程序块看起来就像静态初始化程序块，但是没有 [`static`](/web/20220915120117/https://www.baeldung.com/java-static) 关键字:

```java
{
     // definition of the Instance initialization block
}
```

静态初始化块总是在实例初始化块之前执行，因为静态块在类加载时运行。但是，实例块在实例创建时运行。Java 编译器将初始化器块复制到每个构造函数中。因此，多个构造函数可以使用这种方法来共享一个代码块:

```java
public class InstanceBlockExample {

    {
        System.out.println("Instance initializer block 1");
    }

    {
        System.out.println("Instance initializer block 2");
    }

    public InstanceBlockExample() {
        System.out.println("Class constructor");
    }

    public static void main(String[] args) {
        InstanceBlockExample iib = new InstanceBlockExample();
        System.out.println("Main Method");
    }
}
```

因此，在这种情况下，上述代码的输出将是:

```java
Instance initializer block 1
Instance initializer block 2
Class constructor
Main Method
```

实例初始化块在每次构造函数调用期间执行，因为编译器在构造函数本身中复制初始化块。

编译器在执行当前类的实例块之前执行父类的实例块。编译器通过`super(),` 调用父类构造函数，实例块在构造函数调用时执行。

## 4.静态和实例初始值设定项块之间的差异

| **静态块** | **实例初始化程序块** |
| 它在类加载期间执行 | 它在类实例化期间执行 |
| 它只能使用静态变量 | 它可以使用静态或非静态(实例变量)。 |
| 它不能使用这个 | 它可以用这个 |
| 当类加载到内存中时，它在程序的整个执行过程中只执行一次 | 每当调用构造函数时，它都可以运行多次 |

## 5.结论

在本教程中，我们已经了解了编译器在类加载期间执行静态块。静态块可用于初始化静态变量或调用静态方法。但是，每次创建类的实例时都会执行一个实例块，它可用于初始化实例数据成员。

此外，本文的完整代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220915120117/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-constructors)