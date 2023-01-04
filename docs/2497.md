# Java 中的交换器简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-exchanger>

## 1.概观

在本教程中，我们将研究`java.util.concurrent.Exchanger<T>.`这是 Java 中两个线程交换对象的共同点。

## 2.交换器介绍

**Java 中的`Exchanger`类可以用来在两个 `T`类型的线程之间共享对象。**该类只提供了一个重载方法`exchange(T t)`。

当被调用时，`exchange`等待线程对中的另一个线程也调用它。此时，第二个线程发现第一个线程正在等待它的对象。线程交换它们持有的对象并发出交换信号，现在它们可以返回了。

让我们看一个例子来理解两个带`Exchanger`的线程之间的消息交换:

```
@Test
public void givenThreads_whenMessageExchanged_thenCorrect() {
    Exchanger<String> exchanger = new Exchanger<>();

    Runnable taskA = () -> {
        try {
            String message = exchanger.exchange("from A");
            assertEquals("from B", message);
        } catch (InterruptedException e) {
            Thread.currentThread.interrupt();
            throw new RuntimeException(e);
        }
    };

    Runnable taskB = () -> {
        try {
            String message = exchanger.exchange("from B");
            assertEquals("from A", message);
        } catch (InterruptedException e) {
            Thread.currentThread.interrupt();
            throw new RuntimeException(e);
        }
    };
    CompletableFuture.allOf(
      runAsync(taskA), runAsync(taskB)).join();
}
```

这里，我们让两个线程使用公共交换器在彼此之间交换消息。让我们看一个例子，我们用一个新线程交换主线程中的一个对象:

```
@Test
public void givenThread_WhenExchangedMessage_thenCorrect() throws InterruptedException {
    Exchanger<String> exchanger = new Exchanger<>();

    Runnable runner = () -> {
        try {
            String message = exchanger.exchange("from runner");
            assertEquals("to runner", message);
        } catch (InterruptedException e) {
            Thread.currentThread.interrupt();
            throw new RuntimeException(e);
        }
    };
    CompletableFuture<Void> result 
      = CompletableFuture.runAsync(runner);
    String msg = exchanger.exchange("to runner");
    assertEquals("from runner", msg);
    result.join();
}
```

注意，我们需要先启动`runner `线程，然后在主线程中调用`exchange()`。

另外，请注意，如果第二个线程没有及时到达交换点，第一个线程的调用可能会超时。使用重载`exchange(T t, long timeout, TimeUnit timeUnit).`可以控制第一个线程应该等待多长时间

## 3.无 GC 数据交换

`Exchanger`可用于创建流水线类型的模式，将数据从一个线程传递到另一个线程。在这一节中，我们将创建一个简单的线程堆栈，作为一个管道不断地在彼此之间传递数据。

```
@Test
public void givenData_whenPassedThrough_thenCorrect() throws InterruptedException {

    Exchanger<Queue<String>> readerExchanger = new Exchanger<>();
    Exchanger<Queue<String>> writerExchanger = new Exchanger<>();

    Runnable reader = () -> {
        Queue<String> readerBuffer = new ConcurrentLinkedQueue<>();
        while (true) {
            readerBuffer.add(UUID.randomUUID().toString());
            if (readerBuffer.size() >= BUFFER_SIZE) {
                readerBuffer = readerExchanger.exchange(readerBuffer);
            }
        }
    };

    Runnable processor = () -> {
        Queue<String> processorBuffer = new ConcurrentLinkedQueue<>();
        Queue<String> writerBuffer = new ConcurrentLinkedQueue<>();
        processorBuffer = readerExchanger.exchange(processorBuffer);
        while (true) {
            writerBuffer.add(processorBuffer.poll());
            if (processorBuffer.isEmpty()) {
                processorBuffer = readerExchanger.exchange(processorBuffer);
                writerBuffer = writerExchanger.exchange(writerBuffer);
            }
        }
    };

    Runnable writer = () -> {
        Queue<String> writerBuffer = new ConcurrentLinkedQueue<>();
        writerBuffer = writerExchanger.exchange(writerBuffer);
        while (true) {
            System.out.println(writerBuffer.poll());
            if (writerBuffer.isEmpty()) {
                writerBuffer = writerExchanger.exchange(writerBuffer);
            }
        }
    };
    CompletableFuture.allOf(
      runAsync(reader), 
      runAsync(processor),
      runAsync(writer)).join();
}
```

这里，我们有三个线程:`reader`、`processor`和`writer`。它们共同作为一个管道在它们之间交换数据。

`readerExchanger `由`reader`和`processor`线程共享，`writerExchanger` 由`processor`和`writer`线程共享。

请注意，此处的示例仅用于演示。在用`while(true)`创建无限循环时，我们必须小心。同样为了保持代码的可读性，我们省略了一些异常处理。

这种在重用缓冲区的同时交换数据的模式允许更少的垃圾收集。交换方法返回相同的队列实例，因此这些对象没有 GC。与任何阻塞队列不同，交换器不创建任何节点或对象来保存和共享数据。

创建这样一个管道类似于 Disrupter 模式，一个关键的区别是，Disrupter 模式支持多个生产者和消费者，而一个交换器可以在一对消费者和生产者之间使用。

## 4。结论

所以，我们已经了解了 Java 中的 `Exchanger<T>`是什么，它是如何工作的，我们也看到了如何使用`Exchanger`类。此外，我们创建了一个管道，并演示了线程之间的无 GC 数据交换。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220926182702/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-3)