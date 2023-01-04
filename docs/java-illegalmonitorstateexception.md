# IllegalMonitorStateException in Java

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-illegalmonitorstateexception>

## 1.概观

在这个简短的教程中，我们将学习`java.lang.IllegalMonitorStateException. `

我们将创建一个简单的发送者-接收者应用程序来抛出这个异常。然后，我们将讨论可能的预防方法。最后，我们将展示如何正确实现这些发送者和接收者类。

## 2.什么时候扔？

`IllegalMonitorStateException`与 Java 中的多线程编程有关。如果我们有一个我们想要同步的`[monitor](/web/20221126231824/https://www.baeldung.com/cs/monitor)`,这个异常被抛出来表明一个线程试图等待或者通知其他线程等待这个监视器，而不拥有它。**简而言之，如果我们在 [`synchronized`](/web/20221126231824/https://www.baeldung.com/java-synchronized) 块之外调用`Object`类的`wait()`、`notify(),`或`notifyAll()`方法之一，我们就会得到这个异常。**

现在让我们构建一个抛出`IllegalMonitorStateException`的例子。为此，我们将使用`wait()`和`notifyAll()`方法来同步发送方和接收方之间的数据交换。

首先，让我们看看保存我们要发送的消息的`Data`类:

```java
public class Data {
    private String message;

    public void send(String message) {
        this.message = message;
    }

    public String receive() {
        return message;
    }
}
```

其次，让我们创建一个 sender 类，它在调用`.` 时抛出一个`IllegalMonitorStateException` 。为此，我们将调用`notifyAll()` 方法，而不将其封装在`synchronized`块中:

```java
class UnsynchronizedSender implements Runnable {
    private static final Logger log = LoggerFactory.getLogger(UnsychronizedSender.class);
    private final Data data;

    public UnsynchronizedSender(Data data) {
        this.data = data;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(1000);

            data.send("test");

            data.notifyAll();
        } catch (InterruptedException e) {
            log.error("thread was interrupted", e);
            Thread.currentThread().interrupt();
        }
    }
}
```

接收器也将抛出一个`IllegalMonitorStateException.` 。与前面的例子类似，我们将在一个`synchronized`块之外调用`wait()`方法:

```java
public class UnsynchronizedReceiver implements Runnable {
    private static final Logger log = LoggerFactory.getLogger(UnsynchronizedReceiver.class);
    private final Data data;
    private String message;

    public UnsynchronizedReceiver(Data data) {
        this.data = data;
    }

    @Override
    public void run() {
        try {
            data.wait();
            this.message = data.receive();
        } catch (InterruptedException e) {
            log.error("thread was interrupted", e);
            Thread.currentThread().interrupt();
        }
    }

    public String getMessage() {
        return message;
    }
}
```

最后，让我们实例化这两个类，并在它们之间发送消息:

```java
public void sendData() {
    Data data = new Data();

    UnsynchronizedReceiver receiver = new UnsynchronizedReceiver(data);
    Thread receiverThread = new Thread(receiver, "receiver-thread");
    receiverThread.start();

    UnsynchronizedSender sender = new UnsynchronizedSender(data);
    Thread senderThread = new Thread(sender, "sender-thread");
    senderThread.start();

    senderThread.join(1000);
    receiverThread.join(1000);
}
```

当我们试图运行这段代码时，我们将从`UnsynchronizedReceiver`和`UnsynchronizedSender`类收到一个`IllegalMonitorStateException`:

```java
[sender-thread] ERROR com.baeldung.exceptions.illegalmonitorstate.UnsynchronizedSender - illegal monitor state exception occurred
java.lang.IllegalMonitorStateException: null
	at java.base/java.lang.Object.notifyAll(Native Method)
	at com.baeldung.exceptions.illegalmonitorstate.UnsynchronizedSender.run(UnsynchronizedSender.java:15)
	at java.base/java.lang.Thread.run(Thread.java:844)

[receiver-thread] ERROR com.baeldung.exceptions.illegalmonitorstate.UnsynchronizedReceiver - illegal monitor state exception occurred
java.lang.IllegalMonitorStateException: null
	at java.base/java.lang.Object.wait(Native Method)
	at java.base/java.lang.Object.wait(Object.java:328)
	at com.baeldung.exceptions.illegalmonitorstate.UnsynchronizedReceiver.run(UnsynchronizedReceiver.java:12)
	at java.base/java.lang.Thread.run(Thread.java:844) 
```

## 3.怎么修

为了摆脱`IllegalMonitorStateException, `，我们需要在一个`synchronized`块中对`wait()`、`notify(),`和`notifyAll()` 方法进行每次调用。记住这一点，让我们看看`Sender`类的正确实现应该是什么样子:

```java
class SynchronizedSender implements Runnable {
    private final Data data;

    public SynchronizedSender(Data data) {
        this.data = data;
    }

    @Override
    public void run() {
        synchronized (data) {
            data.send("test");

            data.notifyAll();
        }
    }
}
```

注意，我们在同一个`Data`实例上使用了`synchronized`块，我们稍后称之为它的`notifyAll()`方法。

让我们以同样的方式修复`Receiver`:

```java
class SynchronizedReceiver implements Runnable {
    private static final Logger log = LoggerFactory.getLogger(SynchronizedReceiver.class);
    private final Data data;
    private String message;

    public SynchronizedReceiver(Data data) {
        this.data = data;
    }

    @Override
    public void run() {
        synchronized (data) {
            try {
                data.wait();
                this.message = data.receive();
            } catch (InterruptedException e) {
                log.error("thread was interrupted", e);
                Thread.currentThread().interrupt();
            }
        }
    }

    public String getMessage() {
        return message;
    }
}
```

如果我们再次创建这两个类，并尝试在它们之间发送相同的消息，一切都会很好，不会抛出异常。

## 4.结论

在本文中，我们了解了`IllegalMonitorStateException` 的成因以及如何预防。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221126231824/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)