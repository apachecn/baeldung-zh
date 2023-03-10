# 了解 java.lang.Thread.State:等待(停车)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lang-thread-state-waiting-parking>

## 1.概观

在本文中，我们将检查一个 Java [线程状态](/web/20221030132701/https://www.baeldung.com/java-thread-lifecycle) —具体来说就是`Thread.State.WAITING`。我们将看看线程进入这种状态的方法以及它们之间的区别。最后，我们将仔细看看`LockSupport`类，它提供了几个用于同步的静态实用方法。

## 2.输入`Thread.State.WAITING`

Java 提供了多种方式将线程置于`WAITING`状态。

### 2.1.`Object.wait()`

我们可以将线程置于`WAITING`状态的最标准的方法之一是通过 [`wait()`方法](/web/20221030132701/https://www.baeldung.com/java-wait-notify)。**当一个线程[拥有一个对象的监视器](/web/20221030132701/https://www.baeldung.com/cs/monitor)时，我们可以暂停它的执行，直到另一个线程完成一些工作，并使用`notify()`方法**唤醒它。当执行暂停时，线程处于`WAITING (on object monitor)`状态，这也在[程序的线程转储](/web/20221030132701/https://www.baeldung.com/java-thread-dump)中报告:

```java
"WAITING-THREAD" #11 prio=5 os_prio=0 tid=0x000000001d6ff800 nid=0x544 in Object.wait() [0x000000001de4f000]
   java.lang.Thread.State: WAITING (on object monitor)
```

### 2.2.`Thread.join()`

我们可以用来暂停线程执行的另一种方法是通过 [`join()`调用](/web/20221030132701/https://www.baeldung.com/java-thread-join)。**当我们的主线程需要等待一个工作线程先完成时，我们可以从主线程**调用工作线程实例上的`join()`。执行将暂停，主线程将进入`WAITING`状态，从 [`jstack`](/web/20221030132701/https://www.baeldung.com/java-thread-dump#1-jstack) 报告为`WAITING (on object monitor)`:

```java
"MAIN-THREAD" #12 prio=5 os_prio=0 tid=0x000000001da4f000 nid=0x25f4 in Object.wait() [0x000000001e28e000]
   java.lang.Thread.State: WAITING (on object monitor)
```

### 2.3.`LockSupport.park()`

最后，我们还可以用`LockSupport`类的`park()`静态方法将线程设置为`WAITING`状态。**调用`park()`将停止当前线程的执行，并将其置于`WAITING`状态——更具体地说，是`WAITING (parking)`状态**，如`jstack`报告所示:

```java
"PARKED-THREAD" #11 prio=5 os_prio=0 tid=0x000000001e226800 nid=0x43cc waiting on condition [0x000000001e95f000]
   java.lang.Thread.State: WAITING (parking)
```

因为我们想更好地理解线程暂停和解除暂停，让我们仔细看看这是如何工作的。

## 3.暂停和解除暂停线程

正如我们之前看到的，我们可以通过使用`LockSupport`类提供的工具来暂停和取消暂停线程。这个类是 [`Unsafe`类](/web/20221030132701/https://www.baeldung.com/java-unsafe)的包装器，它的大多数方法直接委托给它。**然而，由于`Unsafe`被认为是一个内部 Java API，不应该被使用，`LockSupport`是我们可以访问停车实用程序**的官方途径。

### 3.1.如何使用`LockSupport`

使用`LockSupport`很简单。如果我们想停止一个线程的执行，我们调用`park()`方法。我们不必提供对线程对象本身的引用——代码会停止调用它的线程。

让我们看一个简单的停车例子:

```java
public class Application {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            int acc = 0;
            for (int i = 1; i <= 100; i++) {
                acc += i;
            }
            System.out.println("Work finished");
            LockSupport.park();
            System.out.println(acc);
        });
        t.setName("PARK-THREAD");
        t.start();
    }
}
```

我们创建了一个最小的控制台应用程序，它累积从 1 到 100 的数字并打印出来。如果我们运行它，我们会看到它打印了`Work finished`，但没有打印结果。这当然是因为我们之前调用了`park()`。

**要让`PARK-THREAD`结束，我们必须解开它**。为此，我们必须使用不同的线程。我们可以使用`main`线程(运行`main()`方法的线程)或者创建一个新的线程。

为了简单起见，让我们使用`main`线程:

```java
t.setName("PARK-THREAD");
t.start();

Thread.sleep(1000);
LockSupport.unpark(t);
```

我们在`main`线程中增加一秒钟的睡眠，让`PARK-THREAD`完成累加并自行停放。在那之后，我们通过调用`unpark(Thread)`来打开它。不出所料，在解包期间，我们必须提供一个对我们想要启动的线程对象的引用。

经过我们的修改，程序现在可以正常终止并打印结果`5050`。

### 3.2.停车许可证

停车 API 的内部是通过使用许可证来工作的。实际上，这就像一个单独的许可证 [`Semaphore`](/web/20221030132701/https://www.baeldung.com/java-semaphore) 。park permit 在内部被用来管理线程的状态，用**方法`park()`消耗它，而`unpark()`使它可用。**

**由于每个线程只能有一个许可证，多次调用`unpark()`方法没有任何效果。**一个单独的`park()` 调用将禁用线程。

然而，有趣的是，被暂停的线程等待一个许可变得可用来再次启用它自己。**如果在调用`park()`时许可已经可用，那么线程永远不会被禁用。**许可被消耗，`park()`调用立即返回，线程继续执行。

我们可以通过删除前面代码片段中对`sleep()`的调用来看到这种效果:

```java
//Thread.sleep(1000);
LockSupport.unpark(t);
```

如果我们再次运行我们的程序，我们将看到在`PARK-THREAD`执行中没有延迟。这是因为我们立即调用了`unpark()`，这使得许可对于`park()`可用。

### 3.3.停车超载

`LockSupport`类包含了`park(Object blocker)`重载方法。`blocker`参数是负责线程暂停的同步对象。**我们提供的对象不会影响那个停放过程，但它会在线程转储中报告，这可以帮助我们诊断并发问题。**

让我们更改代码以包含一个同步器对象:

```java
Object syncObj = new Object();
LockSupport.park(syncObj);
```

如果我们删除对`unpark()`的调用并再次运行应用程序，它将会挂起。如果我们使用`jstack`来查看`PARK-THREAD`在做什么，我们会得到:

```java
"PARK-THREAD" #11 prio=5 os_prio=0 tid=0x000000001e401000 nid=0xfb0 waiting on condition [0x000000001eb4f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076b4a8690> (a java.lang.Object)
```

正如我们所见，最后一行包含了`PARK-THREAD`正在等待的对象。**这有助于调试，这就是为什么我们应该选择`park(Object)`重载。**

## 4.停车与等待

既然这两个 API 给了我们相似的功能，我们应该选择哪一个呢？一般来说，`LockSupport`类及其工具被认为是低级 API。此外，API 很容易被误用，导致难以理解的死锁。**大多数情况下，我们应该使用`Thread`级的`wait()`和`join()`** 。

使用 parking 的好处是我们不需要输入一个`synchronized`块来禁用线程。这很重要，因为`synchronized`块在代码中建立了一个[发生在关系](/web/20221030132701/https://www.baeldung.com/java-volatile#happens-before)之前，这会强制刷新所有变量，如果不需要的话，可能会降低性能。然而，这种优化应该很少发挥作用。

## 5.结论

在本文中，我们探索了`LockSupport`类及其 parking API。我们研究了如何使用它来禁用线程，并解释了它的内部工作原理。最后，我们将它与更常见的`wait()/join()` API 进行了比较，并展示了它们的不同之处。

和往常一样，代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20221030132701/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)