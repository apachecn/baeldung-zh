# Java 传输队列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-transfer-queue>

## 1。概述

在本文中，我们将从标准的`java.util.concurrent`包中寻找`[TransferQueue](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/TransferQueue.html)` 构造。

简单地说，这个队列允许我们根据生产者-消费者模式创建程序，并协调从生产者到消费者的消息传递。

该实现实际上与`[BlockingQueue](/web/20220706104850/https://www.baeldung.com/java-blocking-queue) –` 类似，但是给了我们实现一种背压形式的新能力。这意味着，当生产者使用`transfer()` 方法向消费者发送消息时，生产者将一直被阻止，直到消息被消费。

## 2。一个生产者——零消费者

让我们测试来自`TransferQueue` 的`transfer()`方法——预期的行为是生产者将被阻塞，直到消费者使用`take()` 方法从队列中收到消息。

为了实现这一点，我们将创建一个只有一个生产者而没有消费者的程序。生产者线程对`transfer()` 的第一次调用将无限期阻塞，因为我们没有任何消费者从队列中获取该元素。

让我们看看`Producer` 类是什么样子的:

```
class Producer implements Runnable {
    private TransferQueue<String> transferQueue;

    private String name;

    private Integer numberOfMessagesToProduce;

    public AtomicInteger numberOfProducedMessages
      = new AtomicInteger();

    @Override
    public void run() {
        for (int i = 0; i < numberOfMessagesToProduce; i++) {
            try {
                boolean added 
                  = transferQueue.tryTransfer("A" + i, 4000, TimeUnit.MILLISECONDS);
                if(added){
                    numberOfProducedMessages.incrementAndGet();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    // standard constructors
}
```

我们将传递一个`TransferQueue` 的实例给构造函数，同时传递一个名字给我们的生产者，以及应该传递给队列的元素数量。

注意，我们使用的是`tryTransfer()`方法，有一个给定的超时。我们等待 4 秒钟，如果生产者无法在给定的超时时间内传输消息，它将返回`false`并继续传输下一条消息。生成器有一个`numberOfProducedMessages` 变量来跟踪生成了多少条消息。

接下来，我们来看看`Consumer`类:

```
class Consumer implements Runnable {

    private TransferQueue<String> transferQueue;

    private String name;

    private int numberOfMessagesToConsume;

    public AtomicInteger numberOfConsumedMessages
     = new AtomicInteger();

    @Override
    public void run() {
        for (int i = 0; i < numberOfMessagesToConsume; i++) {
            try {
                String element = transferQueue.take();
                longProcessing(element);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void longProcessing(String element)
      throws InterruptedException {
        numberOfConsumedMessages.incrementAndGet();
        Thread.sleep(500);
    }

    // standard constructors
}
```

它类似于生产者，但是我们通过使用`take()` 方法从队列中接收元素。我们还通过使用`longProcessing()` 方法来模拟一些长时间运行的动作，在这个方法中，我们增加了`numberOfConsumedMessages` 变量，它是接收到的消息的计数器。

现在，让我们从一个制片人开始我们的节目:

```
@Test
public void whenUseOneProducerAndNoConsumers_thenShouldFailWithTimeout() 
  throws InterruptedException {
    // given
    TransferQueue<String> transferQueue = new LinkedTransferQueue<>();
    ExecutorService exService = Executors.newFixedThreadPool(2);
    Producer producer = new Producer(transferQueue, "1", 3);

    // when
    exService.execute(producer);

    // then
    exService.awaitTermination(5000, TimeUnit.MILLISECONDS);
    exService.shutdown();

    assertEquals(producer.numberOfProducedMessages.intValue(), 0);
}
```

我们希望向队列发送三个元素，但是生产者在第一个元素上被阻塞，并且没有消费者从队列中获取该元素`.`我们使用的是`tryTransfer()` 方法，它将阻塞，直到消息被消费或超时。超时后，它将返回`false`表示传输失败，并尝试传输下一个。这是上一个示例的输出:

```
Producer: 1 is waiting to transfer...
can not add an element due to the timeout
Producer: 1 is waiting to transfer...
```

## 3。一个生产者——一个消费者

让我们测试一个有一个生产者和一个消费者的情况:

```
@Test
public void whenUseOneConsumerAndOneProducer_thenShouldProcessAllMessages() 
  throws InterruptedException {
    // given
    TransferQueue<String> transferQueue = new LinkedTransferQueue<>();
    ExecutorService exService = Executors.newFixedThreadPool(2);
    Producer producer = new Producer(transferQueue, "1", 3);
    Consumer consumer = new Consumer(transferQueue, "1", 3);

    // when
    exService.execute(producer);
    exService.execute(consumer);

    // then
    exService.awaitTermination(5000, TimeUnit.MILLISECONDS);
    exService.shutdown();

    assertEquals(producer.numberOfProducedMessages.intValue(), 3);
    assertEquals(consumer.numberOfConsumedMessages.intValue(), 3);
}
```

`TransferQueue`被用作交换点，在消费者消费完队列中的一个元素之前，生产者不能继续向队列中添加另一个元素。让我们看看程序输出:

```
Producer: 1 is waiting to transfer...
Consumer: 1 is waiting to take element...
Producer: 1 transferred element: A0
Producer: 1 is waiting to transfer...
Consumer: 1 received element: A0
Consumer: 1 is waiting to take element...
Producer: 1 transferred element: A1
Producer: 1 is waiting to transfer...
Consumer: 1 received element: A1
Consumer: 1 is waiting to take element...
Producer: 1 transferred element: A2
Consumer: 1 received element: A2
```

我们看到，由于`TransferQueue.`的规范，从队列中生产和消费元素是连续的

## 4。许多生产者——许多消费者

在最后一个例子中，我们将考虑拥有多个消费者和多个生产者:

```
@Test
public void whenMultipleConsumersAndProducers_thenProcessAllMessages() 
  throws InterruptedException {
    // given
    TransferQueue<String> transferQueue = new LinkedTransferQueue<>();
    ExecutorService exService = Executors.newFixedThreadPool(3);
    Producer producer1 = new Producer(transferQueue, "1", 3);
    Producer producer2 = new Producer(transferQueue, "2", 3);
    Consumer consumer1 = new Consumer(transferQueue, "1", 3);
    Consumer consumer2 = new Consumer(transferQueue, "2", 3);

    // when
    exService.execute(producer1);
    exService.execute(producer2);
    exService.execute(consumer1);
    exService.execute(consumer2);

    // then
    exService.awaitTermination(10_000, TimeUnit.MILLISECONDS);
    exService.shutdown();

    assertEquals(producer1.numberOfProducedMessages.intValue(), 3);
    assertEquals(producer2.numberOfProducedMessages.intValue(), 3);
}
```

在这个例子中，我们有两个消费者和两个生产者。当程序启动时，我们看到两个生产者都可以生产一个元素，之后，他们将阻塞，直到其中一个消费者从队列中取出该元素:

```
Producer: 1 is waiting to transfer...
Consumer: 1 is waiting to take element...
Producer: 2 is waiting to transfer...
Producer: 1 transferred element: A0
Producer: 1 is waiting to transfer...
Consumer: 1 received element: A0
Consumer: 1 is waiting to take element...
Producer: 2 transferred element: A0
Producer: 2 is waiting to transfer...
Consumer: 1 received element: A0
Consumer: 1 is waiting to take element...
Producer: 1 transferred element: A1
Producer: 1 is waiting to transfer...
Consumer: 1 received element: A1
Consumer: 2 is waiting to take element...
Producer: 2 transferred element: A1
Producer: 2 is waiting to transfer...
Consumer: 2 received element: A1
Consumer: 2 is waiting to take element...
Producer: 1 transferred element: A2
Consumer: 2 received element: A2
Consumer: 2 is waiting to take element...
Producer: 2 transferred element: A2
Consumer: 2 received element: A2
```

## 5。结论

在本文中，我们看到了来自`java.util.concurrent`包的`TransferQueue` 构造。

我们看到了如何使用该结构实现生产者-消费者程序。我们使用了一个`transfer()` 方法来创建一种反压力形式，其中生产者不能发布另一个元素，直到消费者从队列中检索到一个元素。

当我们不想让过量生产的生产者用消息淹没队列，从而导致`OutOfMemory` 错误时，`TransferQueue`非常有用。在这样的设计中，消费者将决定生产者产生信息的速度。

所有这些例子和代码片段都可以在 GitHub 上找到[——这是一个 Maven 项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220706104850/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections-2)