# Java 中的 wait 和 notify()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-wait-notify>

## 1。概述

在本教程中，我们将了解 Java 中最基本的机制之一——线程同步。

我们将首先讨论一些与并发相关的基本术语和方法。

## 延伸阅读:

## [Java 中同步关键字指南](/web/20220625164946/https://www.baeldung.com/java-synchronized)

This article discusses thread synchronization of methods, static methods, and instances in Java.[Read more](/web/20220625164946/https://www.baeldung.com/java-synchronized) →

## [如何在 Java 中启动线程](/web/20220625164946/https://www.baeldung.com/java-start-thread)

Explore different ways to start a thread and execute parallel tasks.[Read more](/web/20220625164946/https://www.baeldung.com/java-start-thread) →

我们将开发一个简单的应用程序来处理并发问题，目的是更好地理解`wait()` 和`notify()`。

## 2。Java 中的线程同步

在多线程环境中，多个线程可能会尝试修改同一资源。不正确地管理线程当然会导致一致性问题。

### 2.1。Java 中的保护块

在 Java 中，我们可以使用一个工具来协调多线程的动作，这个工具就是保护块。这种块在继续执行之前保持对特定条件的检查。

考虑到这一点，我们将利用以下内容:

*   `[Object.wait()](https://web.archive.org/web/20220625164946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#wait())`暂停线程
*   `[Object.notify()](https://web.archive.org/web/20220625164946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#notify()) `唤醒一根线

我们可以从下面描述`Thread`生命周期的图表中更好地理解这一点:

[![Java - Wait and Notify](img/fd13fff87f5b12a09861d07798aafe75.png)](/web/20220625164946/https://www.baeldung.com/wp-content/uploads/2018/02/Java_-_Wait_and_Notify.png)

请注意，有许多方法可以控制这个生命周期。然而，在本文中，我们将只关注`wait()` 和`notify()`。

## 3。`wait()` 法

简单地说，调用`wait()`会迫使当前线程等待，直到其他线程对同一对象调用`notify()`或`notifyAll()`。

为此，当前线程必须拥有对象的[监视器](/web/20220625164946/https://www.baeldung.com/cs/monitor)。根据 [Javadocs](https://web.archive.org/web/20220625164946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#notify()) 的说法，这可以通过以下方式发生:

*   当我们已经为给定对象执行了`synchronized`实例方法时
*   当我们在给定对象上执行了一个`synchronized` 块的主体时
*   通过对类型为`Class`的对象执行`synchronized static` 方法

注意，一次只有一个活动线程可以拥有一个对象的监视器。

这个`wait()` 方法带有三个重载签名。让我们看看这些。

### 3.1。`wait()`

`wait()` 方法使当前线程无限期等待，直到另一个线程调用这个对象的`notify()`或`notifyAll()`。

### 3.2。`wait(long timeout)`

使用这种方法，我们可以指定一个超时时间，在此之后线程将被自动唤醒。可以使用`notify()`或`notifyAll()`在达到超时前唤醒线程。

注意调用`wait(0)`和调用`wait()`是一样的。

### 3.3。`wait(long timeout, int nanos)`

这是提供相同功能的另一个签名。这里唯一的区别是我们可以提供更高的精度。

总超时时间(以纳秒为单位)计算为`1_000_000*timeout + nanos`。

## 4。`notify()` 和 `notifyAll()`

我们使用`notify()` 方法来唤醒等待访问这个对象的监视器的线程。

有两种方法通知等待的线程。

### 4.1。`notify()`

对于等待这个对象的监视器的所有线程(通过使用任何一个`wait()` 方法)，方法`notify()`通知它们中的任何一个任意唤醒。选择唤醒哪个线程是不确定的，取决于实现。

由于`notify()` 唤醒了一个单独的随机线程，我们可以用它来实现互斥锁，其中线程正在做类似的任务。但是在大多数情况下，实现`notifyAll()`会更可行。

### 4.2。`notifyAll()`

这个方法只是唤醒所有等待这个对象的监视器的线程。

被唤醒的线程将以通常的方式完成，就像任何其他线程一样。

但是在我们允许它们继续执行之前，总是**定义一个快速检查来检查继续执行线程所需的条件。**这是因为在某些情况下，线程可能在没有收到通知的情况下被唤醒(这种情况将在后面的示例中讨论)。

## 5。发送方-接收方同步问题

现在我们已经了解了基础知识，让我们来看一个简单的`Sender`–`Receiver`应用程序，它将利用`wait()`和`notify()`方法在它们之间建立同步:

*   `Sender`应该向`Receiver`发送数据包。
*   在`Sender`完成发送之前，`Receiver`不能处理数据包。
*   类似地，`Sender`不应该试图发送另一个包，除非`Receiver`已经处理了前一个包。

让我们首先创建一个由将从`Sender`发送到`Receiver`的数据`packet`组成的`Data`类。我们将使用`wait()`和`notifyAll()` 来设置它们之间的同步:

```java
public class Data {
    private String packet;

    // True if receiver should wait
    // False if sender should wait
    private boolean transfer = true;

    public synchronized String receive() {
        while (transfer) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                System.out.println("Thread Interrupted");
            }
        }
        transfer = true;

        String returnPacket = packet;
        notifyAll();
        return returnPacket;
    }

    public synchronized void send(String packet) {
        while (!transfer) {
            try { 
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                System.out.println("Thread Interrupted");
            }
        }
        transfer = false;

        this.packet = packet;
        notifyAll();
    }
}
```

让我们来分析一下这里发生了什么:

*   `packet`变量表示通过网络传输的数据。
*   我们有一个`boolean`变量`transfer`,`Sender`和`Receiver`将使用它进行同步:
    *   如果这个变量是`true`，那么`Receiver`应该等待`Sender`发送消息。
    *   如果是`false`，`Sender`应该等待`Receiver`收到消息。
*   `Sender`使用`send()`方法向`Receiver`发送数据:
    *   如果`transfer`是`false`，我们将通过在这个线程上调用`wait()`来等待。
    *   但是当它是`true`时，我们切换状态，设置我们的消息，并调用`notifyAll()` 来唤醒其他线程，以指定一个重要事件已经发生，并且它们可以检查它们是否可以继续执行。
*   类似地，`Receiver`将使用`receive()` 方法:
    *   如果`transfer`被`Sender`设置为`false` ，那么它才会继续，否则我们将在这个线程上调用`wait()` 。
    *   当条件满足时，我们切换状态，通知所有等待的线程醒来，并返回收到的数据包。

### 5.1。为什么要把`wait()` 放在`while`循环中？

因为`notify()` 和`notifyAll()` 随机唤醒等待这个对象的监视器的线程，所以满足条件并不总是重要的。有时线程被唤醒了，但是条件实际上还没有满足。

我们还可以定义一个检查来避免虚假唤醒——线程可能在没有收到通知的情况下从等待中醒来。

### 5.2。为什么我们需要同步 s `end()`和 `receive()`方法？

我们将这些方法放在`synchronized` 方法中，以提供内在锁。如果调用`wait()` 方法的线程不拥有固有锁，将会抛出一个错误。

我们现在将创建`Sender`和`Receiver`并在两者上实现`Runnable`接口，这样它们的实例就可以被一个线程执行。

首先，我们来看看`Sender`是如何工作的:

```java
public class Sender implements Runnable {
    private Data data;

    // standard constructors

    public void run() {
        String packets[] = {
          "First packet",
          "Second packet",
          "Third packet",
          "Fourth packet",
          "End"
        };

        for (String packet : packets) {
            data.send(packet);

            // Thread.sleep() to mimic heavy server-side processing
            try {
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 5000));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                Log.error("Thread interrupted", e); 
            }
        }
    }
}
```

让我们仔细看看这个`Sender`:

*   我们正在创建一些随机数据包，它们将以`packets[]` 数组的形式通过网络发送。
*   对于每个包，我们仅仅调用 send()。
*   然后我们以随机的时间间隔调用`Thread.sleep()`来模拟繁重的服务器端处理。

最后，让我们实现我们的`Receiver`:

```java
public class Receiver implements Runnable {
    private Data load;

    // standard constructors

    public void run() {
        for(String receivedMessage = load.receive();
          !"End".equals(receivedMessage);
          receivedMessage = load.receive()) {

            System.out.println(receivedMessage);

            // ...
            try {
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 5000));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                Log.error("Thread interrupted", e); 
            }
        }
    }
}
```

这里，我们只是在循环中调用`load.receive()`，直到我们得到最后一个`“End”` 数据包。

现在让我们来看看这个应用程序的运行情况:

```java
public static void main(String[] args) {
    Data data = new Data();
    Thread sender = new Thread(new Sender(data));
    Thread receiver = new Thread(new Receiver(data));

    sender.start();
    receiver.start();
}
```

我们将收到以下输出:

```java
First packet
Second packet
Third packet
Fourth packet 
```

我们在这里。**我们已经按照正确的顺序接收了所有数据包**，并成功地在发送方和接收方之间建立了正确的通信。

## 6。结论

在本文中，我们讨论了 Java 中的一些核心同步概念。更具体地说，我们关注如何使用`wait()` 和`notify()` 来解决有趣的同步问题。最后，我们浏览了一个代码示例，在其中我们将这些概念应用到了实践中。

在我们结束之前，值得一提的是，所有这些底层 API，如`wait()`、`notify()`和`notifyAll()`，都是工作良好的传统方法，但更高层的机制往往更简单、更好——如 Java 的原生`Lock`和`Condition`接口(在`java.util.concurrent.locks`包中可用)。

关于`java.util.concurrent`包的更多信息，请访问我们的[Java . util . concurrent](/web/20220625164946/https://www.baeldung.com/java-util-concurrent)文章概述。而`Lock`和`Condition`在 java.util.concurrent.Locks 的[指南中有所涉及。](/web/20220625164946/https://www.baeldung.com/java-concurrent-locks)

和往常一样，本文中使用的完整代码片段可以在 GitHub 上获得。