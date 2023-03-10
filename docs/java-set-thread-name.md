# 在 Java 中设置线程的名称

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-set-thread-name>

## 1.概观

在本教程中，我们将看看在 Java 中设置`Thread`名称的不同方法。首先，我们将创建一个运行两个`Threads.`的例子，一个只打印偶数，另一个只打印奇数。然后，我们会给我们的`Threads`一个自定义名称并显示它们。

## 2.设置`Thread`名称的方法

一个 [`Thread`](/web/20220627165451/https://www.baeldung.com/java-runnable-vs-extending-thread) 是一个可以并发执行的轻量级进程。**Java 中的`Thread`类为线程提供了一个默认名称。**

在某些情况下，我们可能需要知道哪个线程正在运行，所以给一个`Thread`起一个自定义的名字可以使它更容易在其他正在运行的线程中被发现。

让我们首先定义一个简单的类，它创建两个`Threads.`，第一个`Thread`将打印 1 和 N 之间的偶数，第二个`Thread`将打印 1 和 N 之间的奇数，在我们的例子中，N 是 5。

我们还将打印出`Thread`的默认名称。

首先，让我们创建两个`Threads:`

```java
public class CustomThreadNameTest {

    public int currentNumber = 1;

    public int N = 5;

    public static void main(String[] args) {

        CustomThreadNameTest test = new CustomThreadNameTest();

        Thread oddThread = new Thread(() -> {
            test.printOddNumber();
        });

        Thread evenThread = new Thread(() -> {
            test.printEvenNumber();
        });

        evenThread.start();
        oddThread.start();

    }
    // printEvenNumber() and printOddNumber()
} 
```

这里，在`printEvenNumber`和`printOddNumber`方法中，我们将检查当前数字是偶数还是奇数，并打印数字和`Thread`的名字:

```java
public void printEvenNumber() {
    synchronized (this) {
        while (currentNumber < N) {
            while (currentNumber % 2 == 1) {
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + " --> " + currentNumber);
            currentNumber++;
            notify();
        }
    }
}

public void printOddNumber() {
    synchronized (this) {
        while (currentNumber < N) {
            while (currentNumber % 2 == 0) {
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + " --> " + currentNumber);
            currentNumber++;
            notify();
        }
    }
}
```

运行该代码会产生以下输出:

```java
Thread-0 --> 1
Thread-1 --> 2
Thread-0 --> 3
Thread-1 --> 4
Thread-0 --> 5
```

所有线程都有一个默认名称，`Thread-0, Thread-1,`等等。

### 2.1.使用`Thread`构造函数

**`Thread`类提供了一些构造函数，我们可以在`Thread`创建**时提供`Thread`的名字，比如:

*   `Thread(Runnable target, String name)`
*   `Thread(String name)`

在这种情况下，参数`name,`是名称`Thread`。

使用`Thread`构造函数，我们可以在线程创建时提供线程名称。

让我们为我们的`Thread`起一个自定义名称:

```java
Thread oddThread = new Thread(() -> {
    test.printOddNumber();
}, "ODD");

Thread evenThread = new Thread(() -> {
    test.printEvenNumber();
}, "EVEN");
```

现在，当我们运行代码时，会显示自定义名称:

```java
ODD --> 1
EVEN --> 2
ODD --> 3
EVEN --> 4
ODD --> 5
```

### 2.2.使用`setName()`方法

另外，**`Thread`类提供了一个`setName`方法**。

让我们通过`Thread.currentThread().setName()`呼叫`setName`:

```java
Thread oddThread = new Thread(() -> {
    Thread.currentThread().setName("ODD");
    test.printOddNumber();
});

Thread evenThread = new Thread(() -> {
    Thread.currentThread().setName("EVEN");
    test.printEvenNumber();
});
```

此外，通过`Thread.setName()`:

```java
Thread oddThread = new Thread(() -> {
    test.printOddNumber();
});
oddThread.setName("ODD");

Thread evenThread = new Thread(() -> {
    test.printEvenNumber();
});
evenThread.setName("EVEN");
```

再次，运行代码显示我们的`Thread`的自定义名称:

```java
ODD --> 1
EVEN --> 2
ODD --> 3
EVEN --> 4
ODD --> 5
```

## 3.结论

在本文中，我们研究了如何在 Java 中设置一个`Thread`的名称。首先，我们用默认名称创建了一个`Thread`，然后使用`Thread`构造函数和`setName`方法设置了一个自定义名称。

和往常一样，本文的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627165451/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-2/)