# 延迟队列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-delay-queue>

## 1。概述

在本文中，我们将关注来自`java.util.concurrent`包的`[DelayQueue](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/DelayQueue.html)` 构造。这是一个可以在生产者-消费者程序中使用的阻塞队列。

它有一个非常有用的特性—**当消费者想要从队列中取出一个元素时，他们只能在该特定元素的延迟到期时才能取出它。**

## 2。对`DelayQueue` 中的元素实施`Delayed`

我们想要放入`DelayQueue` 的每个元素都需要实现`[Delayed](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Delayed.html)` 接口。假设我们想要创建一个`DelayObject`类。该类的实例将被放入`DelayQueue.`

我们将把`String`数据和`delayInMilliseconds` 作为参数传递给它的构造函数:

```
public class DelayObject implements Delayed {
    private String data;
    private long startTime;

    public DelayObject(String data, long delayInMilliseconds) {
        this.data = data;
        this.startTime = System.currentTimeMillis() + delayInMilliseconds;
    }
```

我们正在定义一个`startTime –` 这是一个元素应该从队列中被消耗的时间。接下来，我们需要实现`getDelay()` 方法——它应该返回给定时间单位内与该对象相关的剩余延迟。

因此，我们需要使用`TimeUnit.convert()` 方法来返回适当的`TimeUnit:`中的剩余延迟

```
@Override
public long getDelay(TimeUnit unit) {
    long diff = startTime - System.currentTimeMillis();
    return unit.convert(diff, TimeUnit.MILLISECONDS);
}
```

当消费者试图从队列中取出一个元素时，`DelayQueue` 将执行`getDelay()` 来确定该元素是否被允许从队列中返回。如果`getDelay()` 方法将返回零或负数，这意味着可以从队列中检索它。

我们还需要实现`compareTo()` 方法，因为`DelayQueue` 中的元素将根据到期时间进行排序。将首先到期的项目被保存在队列的头部，而具有最高到期时间的元素被保存在队列的尾部:

```
@Override
public int compareTo(Delayed o) {
    return Ints.saturatedCast(
      this.startTime - ((DelayObject) o).startTime);
}
```

## 3。消费者和生产者

为了能够测试我们的`DelayQueue` ,我们需要实现生产者和消费者逻辑。producer 类将队列、要生成的元素数量以及每条消息的延迟(以毫秒为单位)作为参数。

然后，当调用`run()` 方法时，它将元素放入队列，并在每次放入后休眠 500 毫秒:

```
public class DelayQueueProducer implements Runnable {

    private BlockingQueue<DelayObject> queue;
    private Integer numberOfElementsToProduce;
    private Integer delayOfEachProducedMessageMilliseconds;

    // standard constructor

    @Override
    public void run() {
        for (int i = 0; i < numberOfElementsToProduce; i++) {
            DelayObject object
              = new DelayObject(
                UUID.randomUUID().toString(), delayOfEachProducedMessageMilliseconds);
            System.out.println("Put object: " + object);
            try {
                queue.put(object);
                Thread.sleep(500);
            } catch (InterruptedException ie) {
                ie.printStackTrace();
            }
        }
    }
}
```

**消费者实现**非常相似，但是它也跟踪被消费的消息的数量:

```
public class DelayQueueConsumer implements Runnable {
    private BlockingQueue<DelayObject> queue;
    private Integer numberOfElementsToTake;
    public AtomicInteger numberOfConsumedElements = new AtomicInteger();

    // standard constructors

    @Override
    public void run() {
        for (int i = 0; i < numberOfElementsToTake; i++) {
            try {
                DelayObject object = queue.take();
                numberOfConsumedElements.incrementAndGet();
                System.out.println("Consumer take: " + object);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 4。`DelayQueue`使用测试

为了测试`DelayQueue,`的行为，我们将创建一个生产者线程和一个消费者线程。

生产者将两个对象以 500 毫秒的延迟放到队列中。该测试断言消费者消费了两条消息:

```
@Test
public void givenDelayQueue_whenProduceElement
  _thenShouldConsumeAfterGivenDelay() throws InterruptedException {
    // given
    ExecutorService executor = Executors.newFixedThreadPool(2);

    BlockingQueue<DelayObject> queue = new DelayQueue<>();
    int numberOfElementsToProduce = 2;
    int delayOfEachProducedMessageMilliseconds = 500;
    DelayQueueConsumer consumer = new DelayQueueConsumer(
      queue, numberOfElementsToProduce);
    DelayQueueProducer producer = new DelayQueueProducer(
      queue, numberOfElementsToProduce, delayOfEachProducedMessageMilliseconds);

    // when
    executor.submit(producer);
    executor.submit(consumer);

    // then
    executor.awaitTermination(5, TimeUnit.SECONDS);
    executor.shutdown();

    assertEquals(consumer.numberOfConsumedElements.get(), numberOfElementsToProduce);
}
```

我们可以观察到，运行该程序将产生以下输出:

```
Put object: {data='86046157-e8a0-49b2-9cbb-8326124bcab8', startTime=1494069868007}
Consumer take: {data='86046157-e8a0-49b2-9cbb-8326124bcab8', startTime=1494069868007}
Put object: {data='d47927ef-18c7-449b-b491-5ff30e6795ed', startTime=1494069868512}
Consumer take: {data='d47927ef-18c7-449b-b491-5ff30e6795ed', startTime=1494069868512}
```

生产者放置对象，过一会儿，延迟到期的第一个对象被使用。

第二个要素也出现了同样的情况。

## 5。消费者无法在给定时间内消费

假设我们有一个生产者正在生产一个元素，这个元素将在 10 秒后过期:

```
int numberOfElementsToProduce = 1;
int delayOfEachProducedMessageMilliseconds = 10_000;
DelayQueueConsumer consumer = new DelayQueueConsumer(
  queue, numberOfElementsToProduce);
DelayQueueProducer producer = new DelayQueueProducer(
  queue, numberOfElementsToProduce, delayOfEachProducedMessageMilliseconds);
```

我们将开始测试，但它将在 5 秒后终止。由于`DelayQueue,` 的特征，消费者将不能消费队列中的消息，因为元素还没有过期:

```
executor.submit(producer);
executor.submit(consumer);

executor.awaitTermination(5, TimeUnit.SECONDS);
executor.shutdown();
assertEquals(consumer.numberOfConsumedElements.get(), 0);
```

注意，消费者的`numberOfConsumedElements` 的值等于零。

## 6。产生立即到期的元素

当`Delayed` message `getDelay()`方法的实现返回负数时，这意味着给定的元素已经过期。在这种情况下，生产者将立即消耗该元素。

我们可以测试产生具有负延迟的元素的情况:

```
int numberOfElementsToProduce = 1;
int delayOfEachProducedMessageMilliseconds = -10_000;
DelayQueueConsumer consumer = new DelayQueueConsumer(queue, numberOfElementsToProduce);
DelayQueueProducer producer = new DelayQueueProducer(
  queue, numberOfElementsToProduce, delayOfEachProducedMessageMilliseconds);
```

当我们开始测试用例时，消费者将立即消费该元素，因为它已经过期:

```
executor.submit(producer);
executor.submit(consumer);

executor.awaitTermination(1, TimeUnit.SECONDS);
executor.shutdown();
assertEquals(consumer.numberOfConsumedElements.get(), 1);
```

## 7 .**。结论**

在本文中，我们看到了来自`java.util.concurrent`包的`DelayQueue` 构造。

我们实现了一个从队列中产生和消费的`Delayed` 元素。

我们利用`DelayQueue` 的实现来消费已经过期的元素。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220706104850/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。