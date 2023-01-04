# 使用 2 个线程打印偶数和奇数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-even-odd-numbers-with-2-threads>

## 1.介绍

在本教程中，我们将看看如何使用两个线程打印偶数和奇数。

目标是按顺序打印数字，而一个线程只打印偶数，另一个线程只打印奇数。我们将使用线程同步和线程间通信的概念来解决这个问题。

## 2.Java 中的线程

线程是可以并发执行的轻量级进程。多线程的并发执行对性能和 CPU 利用率有好处，因为我们可以通过并行运行的不同线程同时处理多个任务。

关于 Java 中线程的更多信息可以在这篇文章中找到。

**在 Java 中，我们可以通过扩展`Thread`类或者实现`Runnable`接口**来创建线程。在这两种情况下，我们都覆盖了`run`方法，并在其中编写了线程的实现。

关于如何使用这些方法创建线程的更多信息可以在[这里](/web/20220628093728/https://www.baeldung.com/java-runnable-vs-extending-thread)找到。

## 3.线程同步

在多线程环境中，可能有两个或更多线程几乎同时访问同一资源。这可能是致命的，并导致错误的结果。为了防止这种情况，我们需要确保在给定的时间点只有一个线程访问资源。

我们可以使用线程同步来实现这一点。

在 Java 中，我们可以将一个方法或块标记为 synchronized，这意味着在给定的时间点，只有一个线程能够进入那个方法或块。

关于 Java 线程同步的更多细节可以在[这里](/web/20220628093728/https://www.baeldung.com/java-synchronized)找到。

## 4.线程间通信

线程间通信允许同步线程使用一组方法相互通信。

使用的方法有`wait`、`notify,`和`notifyAll,`，它们都继承自`Object`类。

**`Wait()`使当前线程无限期等待，直到其他线程对同一对象调用`notify() or notifyAll()`。**我们可以调用`notify()` 来唤醒等待访问这个对象的监视器的线程。

关于这些方法工作的更多细节可以在[这里](/web/20220628093728/https://www.baeldung.com/java-wait-notify)找到。

## 5.交替打印奇数和偶数

### 5.1.使用`wait()`和`notify()`

我们将使用讨论过的同步和线程间通信的概念，使用两个不同的线程按升序打印奇数和偶数。

**第一步，我们将实现`Runnable`接口来定义两个线程**的逻辑。在`run`方法中，我们检查数字是偶数还是奇数。

如果数字是偶数，我们调用`Printer`类的`printEven`方法，否则我们调用`printOdd`方法:

```
class TaskEvenOdd implements Runnable {
    private int max;
    private Printer print;
    private boolean isEvenNumber;

    // standard constructors

    @Override
    public void run() {
        int number = isEvenNumber ? 2 : 1;
        while (number <= max) {
            if (isEvenNumber) {
                print.printEven(number);
            } else {
                print.printOdd(number);
            }
            number += 2;
        }
    }
} 
```

我们将`Printer`类定义如下:

```
class Printer {
    private volatile boolean isOdd;

    synchronized void printEven(int number) {
        while (!isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println(Thread.currentThread().getName() + ":" + number);
        isOdd = false;
        notify();
    }

    synchronized void printOdd(int number) {
        while (isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println(Thread.currentThread().getName() + ":" + number);
        isOdd = true;
        notify();
    }
}
```

**在 main 方法中，我们使用已定义的类来创建两个线程。**我们创建一个`Printer`类的对象，并将其作为参数传递给`TaskEvenOdd`构造函数:

```
public static void main(String... args) {
    Printer print = new Printer();
    Thread t1 = new Thread(new TaskEvenOdd(print, 10, false),"Odd");
    Thread t2 = new Thread(new TaskEvenOdd(print, 10, true),"Even");
    t1.start();
    t2.start();
}
```

第一个线程将是奇数线程，因此我们将`false`作为参数`isEvenNumber`的值传递。对于第二个线程，我们改为传递`true`。我们将两个线程的`maxValue`都设置为 10，因此只打印从 1 到 10 的数字。

然后我们通过调用`start()`方法来启动这两个线程。这将调用上面定义的两个线程的`run()`方法，其中我们检查数字是奇数还是偶数并打印它们。

当奇数线程开始运行时，变量`number`的值将为 1。由于小于`maxValue`且标志`isEvenNumber`为假，故称`printOdd()`。在该方法中，我们检查标志`isOdd`是否为真，当它为真时，我们调用`wait(). `，因为`isOdd`最初为假，不调用`wait()`，并且打印值。

**然后我们将`isOdd`的值设置为真，这样奇数线程进入等待状态，并调用`notify()`** 来唤醒偶数线程。然后偶数线程醒来并打印偶数，因为`odd`标志为假。然后它调用`notify()`来唤醒这个奇怪的线程。

执行相同的过程，直到变量`number`的值大于 `maxValue`。

### 5.2.使用信号量

信号量通过使用计数器来控制对共享资源的访问。如果**计数器大于零，则允许访问**。如果为零，则拒绝访问。

Java 在`java.util.concurrent`包中提供了`Semaphore`类，我们可以用它来实现所解释的机制。更多关于信号量的细节可以在[这里](/web/20220628093728/https://www.baeldung.com/java-semaphore)找到。

我们创建两个线程，一个奇数线程和一个偶数线程。奇数线程将打印从 1 开始的奇数，偶数线程将打印从 2 开始的偶数。

两个线程都有一个`SharedPrinter`类的对象。**`SharedPrinter`类将有两个信号量，`semOdd`和`semEven `，它们将有 1 和 0 个许可以**开始。这将确保首先打印奇数。

我们有两个方法`printEvenNum()`和`printOddNum(). `奇数线程调用`printOddNum()`方法，偶数线程调用`printEvenNum()`方法。

为了打印一个奇数，在`semOdd`上调用`acquire()`方法，由于初始许可是 1，它成功地获得了访问权，打印奇数并在`semEven.`上调用`release()`

调用`release()`将使`semEven`的许可增加 1，然后偶数线程可以成功地获得访问权并打印偶数。

这是上述工作流的代码:

```
public static void main(String[] args) {
    SharedPrinter sp = new SharedPrinter();
    Thread odd = new Thread(new Odd(sp, 10),"Odd");
    Thread even = new Thread(new Even(sp, 10),"Even");
    odd.start();
    even.start();
}
```

```
class SharedPrinter {

    private Semaphore semEven = new Semaphore(0);
    private Semaphore semOdd = new Semaphore(1);

    void printEvenNum(int num) {
        try {
            semEven.acquire();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println(Thread.currentThread().getName() + num);
        semOdd.release();
    }

    void printOddNum(int num) {
        try {
            semOdd.acquire();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println(Thread.currentThread().getName() + num);
        semEven.release();

    }
}

class Even implements Runnable {
    private SharedPrinter sp;
    private int max;

    // standard constructor

    @Override
    public void run() {
        for (int i = 2; i <= max; i = i + 2) {
            sp.printEvenNum(i);
        }
    }
}

class Odd implements Runnable {
    private SharedPrinter sp;
    private int max;

    // standard constructors 
    @Override
    public void run() {
        for (int i = 1; i <= max; i = i + 2) {
            sp.printOddNum(i);
        }
    }
}
```

## 6.结论

在本教程中，我们了解了如何在 Java 中使用两个线程交替打印奇数和偶数。我们看了一下实现相同结果的两种方法:**使用`wait()`和`notify()`T5，**使用`Semaphore`** `.`**

和往常一样，GitHub 上有完整的工作代码[。](https://web.archive.org/web/20220628093728/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)