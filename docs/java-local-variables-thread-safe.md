# 为什么局部变量在 Java 中是线程安全的

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-local-variables-thread-safe>

## 1.介绍

之前我们介绍了[线程安全，以及如何实现它](/web/20221129005355/https://www.baeldung.com/java-thread-safety)。

在本文中，我们将看看局部变量以及为什么它们是线程安全的。

## 2.堆栈内存和线程

让我们先快速回顾一下 JVM 内存模型。

最重要的是，JVM 将其可用内存分成[堆栈和](/web/20221129005355/https://www.baeldung.com/java-stack-heap)堆内存。首先，它在堆上存储所有对象。其次，**在栈上存储本地原语和本地对象引用**。

此外，重要的是要认识到每个线程，包括主线程，都有自己的私有堆栈。因此，**其他线程不共享我们的局部变量，这就是为什么它们是线程安全的**。

## 3.例子

现在让我们继续看一个包含一个本地原语和一个(原语)字段的小代码示例:

```java
public class LocalVariables implements Runnable {
    private int field;

    public static void main(String... args) {
        LocalVariables target = new LocalVariables();
        new Thread(target).start();
        new Thread(target).start();
    }

    @Override
    public void run() {
        field = new SecureRandom().nextInt();
        int local = new SecureRandom().nextInt();
        System.out.println(field + ":" + local);
    }
}
```

在第五行，我们实例化了一个`LocalVariables`类的副本。在接下来的两行中，我们启动了两个线程。两者都将执行同一个实例的`run`方法。

在`run`方法中，我们更新了`LocalVariables`类的字段`field`。其次，我们看到一个局部原语的赋值。最后，我们将这两个字段打印到控制台。

我们来看看所有字段的内存位置。

首先，`field`是类`LocalVariables`的一个字段。因此，它生活在堆上。其次，局部变量`number`是一个原语。因此，它位于堆栈上。

**`println`语句是运行两个线程时可能出错的地方**。

首先，`field` 字段很有可能引起麻烦，因为引用和对象都在堆上，并且在我们的线程之间共享。原语`local`没有问题，因为值存在于堆栈中。因此，JVM 不会在线程间共享`local`。

例如，在执行时，我们可以得到以下输出:

```java
 821695124:1189444795
821695124:47842893
```

在这种情况下，我们可以看到两个线程之间确实**发生了冲突。我们肯定这一点，因为两个线程生成相同的随机整数的可能性极小。**

## 4.Lambdas 内部的局部变量

Lambdas(和[匿名内部类](/web/20221129005355/https://www.baeldung.com/java-anonymous-classes))可以在方法内部声明，并且可以访问方法的局部变量。然而，如果没有任何额外的警卫，这可能会导致很多麻烦。

在 JDK 8 之前，有一个明确的规则，即**匿名内部类只能访问`final`局部变量**。JDK 8 引入了有效终结的新概念，规则也变得不那么严格了。我们之前比较过 [final 和有效 final](/web/20221129005355/https://www.baeldung.com/java-effectively-final) ，也讨论过更多关于使用 lambdas 时的[有效 final。](/web/20221129005355/https://www.baeldung.com/java-lambda-effectively-final-local-variables)

这个规则的结果是，在 lambdas 内部被访问的**字段必须是 final 或者有效的 final** (它们因此没有被改变)，这使得它们[由于不变性](/web/20221129005355/https://www.baeldung.com/java-thread-safety#immutable-implementations)而成为线程安全的。

我们可以在下面的例子中看到这种行为:

```java
public static void main(String... args) {
    String text = "";
    // text = "675";
    new Thread(() -> System.out.println(text))
            .start();
}
```

在这种情况下，取消第 3 行代码的注释将导致编译错误。因为这样一来，局部变量`text`实际上不再是 final。

## 5.结论

在本文中，我们研究了局部变量的线程安全，发现这是 JVM 内存模型的结果。我们还研究了局部变量与 lambdas 的结合使用。JVM 通过要求不变性来保护它们的线程安全。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221129005355/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-2)