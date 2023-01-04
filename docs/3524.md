# 为什么不在构造函数中启动一个线程？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-constructor>

## 1.概观

在这个快速教程中，我们将看到为什么我们不应该在构造函数中启动线程。

首先，我们将简要介绍 Java 和 JVM 中的发布概念。然后，我们将看到这个概念如何影响我们启动线程的方式。

## 2.出版和逃脱

**每次我们让一个对象对其当前范围之外的任何其他代码可用时，我们基本上发布了那个对象**。例如，当我们返回一个对象，将它存储到一个`public `引用中，或者甚至将它传递给另一个方法时，就会发生发布。

**当我们发布一个不该发布的对象时，我们说这个对象已经逃逸了**。

我们有很多方法可以让对象引用转义，比如在完全构造之前发布对象。事实上，这是常见的转义形式之一:**当`this `引用在对象构造期间转义时。**

当`this `引用在构建期间转义时，其他线程可能会看到该对象处于不正确且未完全构建的状态。这反过来会导致奇怪的线程安全并发症。

## 3.用线程转义

**让`this`引用转义的最常见方式之一是在构造函数中启动一个线程。**为了更好地理解这一点，让我们考虑一个例子:

```
public class LoggerRunnable implements Runnable {

    public LoggerRunnable() {
        Thread thread = new Thread(this); // this escapes
        thread.start();
    }

    @Override
    public void run() {
        System.out.println("Started...");
    }
}
```

这里，我们显式地将`this `引用传递给`Thread `构造函数。因此，**新启动的线程可能会在它的完整构造完成之前看到封闭对象。**在并发环境中，这可能会导致细微的错误。

**也可以隐式地传递`this `引用**:

```
public class ImplicitEscape {

    public ImplicitEscape() {
        Thread t = new Thread() {

            @Override
            public void run() {
                System.out.println("Started...");
            }
        };

        t.start();
    }
}
```

如上所示，我们正在创建一个从`Thread`派生的匿名内部类。**因为内部类维护了对其封闭类的引用，所以`this `引用再次从构造函数中转义。**

在构造函数中创建一个`Thread `本身并没有什么错。**然而，非常不鼓励立即启动它**，因为大多数时候，我们以一个转义的`this `引用结束，不管是显式的还是隐式的。

### 3.1.可供选择的事物

我们可以为这个场景声明一个专用的方法，而不是在构造函数中启动一个线程:

```
public class SafePublication implements Runnable {

    private final Thread thread;

    public SafePublication() {
        thread = new Thread(this);
    }

    @Override
    public void run() {
        System.out.println("Started...");
    }

    public void start() {
        thread.start();
    }
};:
```

如上所示，我们仍然发布对`Thread. `的`this `引用，但是，这一次，我们在构造函数返回后启动线程:

```
SafePublication publication = new SafePublication();
publication.start();
```

因此，对象引用在其完全构造之前不会转义到另一个线程。

## 4.结论

在这个快速教程中，在简要介绍了安全发布之后，我们看到了为什么我们不应该在构造函数中启动线程。

关于 Java 中的发布和转义的更多详细信息可以在[Java Concurrency in Practice](https://web.archive.org/web/20221107231800/https://learning.oreilly.com/library/view/java-concurrency-in/0321349601/)一书中找到。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221107231800/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-3)