# 为什么 Lambdas 中使用的局部变量必须是 Final 或有效 Final？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lambda-effectively-final-local-variables>

## 1。简介

Java 8 给了我们 lambdas，并且通过关联，给了我们`effectively final`变量的概念。有没有想过为什么在 lambdas 中捕获的局部变量必须是最终的或者实际上是最终的？

嗯， [JLS](https://web.archive.org/web/20221105100820/https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.27.2) 给了我们一点提示，它说“对有效的最终变量的限制禁止访问动态变化的局部变量，这些变量的捕获可能会引入并发问题。”但是，这意味着什么呢？

在接下来的几节中，我们将更深入地研究这个限制，看看 Java 为什么引入它。我们将举例说明**如何影响单线程和并发应用**，我们还将**揭穿一个常见的反模式来解决这个限制。**

## 2.捕捉兰姆达斯

Lambda 表达式可以使用外部作用域中定义的变量。我们将这些λ称为`capturing lambdas`。它们可以捕获静态变量、实例变量和局部变量，但只有**局部变量必须是最终变量或有效的最终变量。**

在早期的 Java 版本中，当一个匿名内部类捕获一个方法的局部变量时，我们会遇到这种情况——我们需要在局部变量之前添加`final` 关键字，这样编译器才会满意。

作为一点语法上的好处，现在编译器可以识别这样的情况，当`final `关键字不存在时，引用根本没有改变，这意味着它是`effectively` final。**我们可以说一个变量实际上是 final 的，如果编译器不会抱怨我们声明它是 final 的话。**

## 3。捕获 Lambdas 中的局部变量

简单来说，**这个不会编译:**

```java
Supplier<Integer> incrementer(int start) {
  return () -> start++;
}
```

`start `是一个局部变量，我们试图在 lambda 表达式中修改它。

这不会编译的基本原因是 lambda 是**捕获`start`的值，这意味着复制它。**将变量强制为 final 避免给人留下这样的印象:在 lambda 中增加`start`实际上会修改`start`方法参数。

但是，它为什么要复制呢？注意，我们从方法中返回了 lambda。因此，lambda 不会运行，直到`start`方法参数被垃圾回收。Java 必须复制一个`start` 才能让这个 lambda 存在于这个方法之外。

### 3.1.并发问题

为了好玩，让我们想象一下 Java `did`允许局部变量以某种方式与它们捕获的值保持联系。

我们在这里应该做什么:

```java
public void localVariableMultithreading() {
    boolean run = true;
    executor.execute(() -> {
        while (run) {
            // do operation
        }
    });

    run = false;
}
```

虽然这看起来是无辜的，但它存在“可见性”的潜在问题。回想一下，每个线程都有自己的堆栈，那么我们如何确保我们的`while`循环`sees`改变另一个堆栈中的`run `变量呢？在其他情况下，答案可能是使用`synchronized `块或`volatile `关键字。

然而，因为 Java 施加了有效的最终限制，我们不必担心这样的复杂性。

## 4。捕获 Lambdas 中的静态或实例变量

如果我们将前面的例子与 lambda 表达式中静态或实例变量的使用进行比较，就会产生一些问题。

我们可以通过将我们的`start`变量转换成实例变量来编译我们的第一个例子:

```java
private int start = 0;

Supplier<Integer> incrementer() {
    return () -> start++;
}
```

但是，为什么我们可以在这里改变`start`的值呢？

简单来说，就是关于成员变量存储在哪里。局部变量在堆栈上，但成员变量在堆上。因为我们处理的是堆内存，编译器可以保证 lambda 可以访问最新的`start.`值

我们可以通过同样的方法修复第二个示例:

```java
private volatile boolean run = true;

public void instanceVariableMultithreading() {
    executor.execute(() -> {
        while (run) {
            // do operation
        }
    });

    run = false;
}
```

自从我们添加了`volatile `关键字后，`run `变量现在对 lambda 可见，即使它在另一个线程中执行。

一般来说，当捕获一个实例变量时，我们可以认为它是捕获最终变量`this`。无论如何，**编译器不抱怨并不意味着我们不应该采取预防措施，**尤其是在多线程环境中。

## 5.避免变通办法

为了避开对局部变量的限制，有人可能会想到使用变量容器来修改局部变量的值。

让我们看一个在单线程应用程序中使用数组存储变量的示例:

```java
public int workaroundSingleThread() {
    int[] holder = new int[] { 2 };
    IntStream sums = IntStream
      .of(1, 2, 3)
      .map(val -> val + holder[0]);

    holder[0] = 0;

    return sums.sum();
}
```

我们可以认为该流对每个值求和为 2，但是**实际上是对 0 求和，因为这是 lambda 执行时可用的最新值。**

让我们更进一步，在另一个线程中执行 sum:

```java
public void workaroundMultithreading() {
    int[] holder = new int[] { 2 };
    Runnable runnable = () -> System.out.println(IntStream
      .of(1, 2, 3)
      .map(val -> val + holder[0])
      .sum());

    new Thread(runnable).start();

    // simulating some processing
    try {
        Thread.sleep(new Random().nextInt(3) * 1000L);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }

    holder[0] = 0;
}
```

我们在这里求和的值是多少？这取决于我们的模拟处理需要多长时间。如果它足够短，可以让方法的执行在另一个线程执行之前终止，它将打印 6，否则，它将打印 12。

一般来说，这种变通方法容易出错，并且会产生不可预知的结果，所以我们应该避免使用它们。

## 6。结论

在本文中，我们解释了为什么 lambda 表达式只能使用 final 或有效的 final 局部变量。正如我们已经看到的，这种限制来自于这些变量的不同性质以及 Java 在内存中存储它们的方式。我们还展示了使用通用变通方法的危险。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221105100820/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas)