# 使用 Kafka 模拟消费者

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-mockconsumer>

## 1.概观

在本教程中，我们将探索`MockConsumer`，卡夫卡的`Consumer`实现之一[。](/web/20220926181713/https://www.baeldung.com/tag/kafka/)

首先，我们将讨论在测试一个卡夫卡`Consumer`时需要考虑的主要事情。然后，我们将看看如何使用`MockConsumer`来实现测试。

## 2.测试卡夫卡`Consumer`

从 Kafka 消费数据包括两个主要步骤。首先，我们必须手动订阅主题或分配主题分区。其次，我们使用`poll `方法轮询记录批次。

轮询通常在无限循环中完成。这是因为我们通常希望持续消费数据。

例如，让我们考虑一个简单的消费逻辑，它只包含订阅和轮询循环:

```java
void consume() {
    try {
        consumer.subscribe(Arrays.asList("foo", "bar"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
            records.forEach(record -> processRecord(record));
        }
    } catch (WakeupException ex) {
        // ignore for shutdown
    } catch (RuntimeException ex) {
        // exception handling
    } finally {
        consumer.close();
    }
}
```

查看上面的代码，我们可以看到有一些东西我们可以测试:

*   订阅
*   轮询循环
*   异常处理
*   如果`Consumer`正确关闭

我们有多种选择来测试消费逻辑。

我们可以使用内存中的 Kafka 实例。但是，这种方法有一些缺点。一般来说，内存中的 Kafka 实例会使测试变得非常繁重和缓慢。此外，设置它不是一项简单的任务，可能会导致不稳定的测试。

或者，我们可以使用一个模仿框架来模仿`Consumer. `,尽管使用这种方法使测试变得轻量级，但是设置它可能有些棘手。

最后一个选项，也许是最好的，是使用`MockConsumer`，这是一个用于测试的`Consumer`实现。**它不仅帮助我们构建轻量级测试，而且设置**也很容易。

让我们来看看它提供的功能。

## 3.使用`MockConsumer`

**`MockConsumer`实现了** `**kafka-clients**` **库提供的** `.` 接口`Consumer`因此，**它模仿了一个真实的`Consumer` 的整个行为，而不需要我们写很多代码**。

让我们来看看`MockConsumer`的一些用法示例。特别是，我们将采用一些在测试消费者应用程序时可能遇到的常见场景，并使用`MockConsumer`来实现它们。

对于我们的示例，让我们考虑一个应用程序，它使用 Kafka 主题中的国家人口更新。更新仅包含国家名称及其当前人口:

```java
class CountryPopulation {

    private String country;
    private Integer population;

    // standard constructor, getters and setters
}
```

我们的消费者只是使用 Kafka `Consumer` 实例轮询更新，处理它们，最后使用`commitSync` 方法提交偏移量:

```java
public class CountryPopulationConsumer {

    private Consumer<String, Integer> consumer;
    private java.util.function.Consumer<Throwable> exceptionConsumer;
    private java.util.function.Consumer<CountryPopulation> countryPopulationConsumer;

    // standard constructor

    void startBySubscribing(String topic) {
        consume(() -> consumer.subscribe(Collections.singleton(topic)));
    }

    void startByAssigning(String topic, int partition) {
        consume(() -> consumer.assign(Collections.singleton(new TopicPartition(topic, partition))));
    }

    private void consume(Runnable beforePollingTask) {
        try {
            beforePollingTask.run();
            while (true) {
                ConsumerRecords<String, Integer> records = consumer.poll(Duration.ofMillis(1000));
                StreamSupport.stream(records.spliterator(), false)
                    .map(record -> new CountryPopulation(record.key(), record.value()))
                    .forEach(countryPopulationConsumer);
                consumer.commitSync();
            }
        } catch (WakeupException e) {
            System.out.println("Shutting down...");
        } catch (RuntimeException ex) {
            exceptionConsumer.accept(ex);
        } finally {
            consumer.close();
        }
    }

    public void stop() {
        consumer.wakeup();
    }
}
```

### 3.1.创建一个`MockConsumer`实例

接下来，让我们看看如何创建一个`MockConsumer`的实例:

```java
@BeforeEach
void setUp() {
    consumer = new MockConsumer<>(OffsetResetStrategy.EARLIEST);
    updates = new ArrayList<>();
    countryPopulationConsumer = new CountryPopulationConsumer(consumer, 
      ex -> this.pollException = ex, updates::add);
}
```

基本上，我们需要提供的只是失调复位策略。

注意，我们使用`updates` 来收集`countryPopulationConsumer`将接收的记录。这将有助于我们断言预期的结果。

同样，我们使用` pollException `来收集和断言异常。

对于所有的测试用例，我们将使用上面的设置方法。现在，让我们看一些消费者应用程序的测试用例。

### 3.2.分配主题分区

首先，让我们为`startByAssigning`方法创建一个测试:

```java
@Test
void whenStartingByAssigningTopicPartition_thenExpectUpdatesAreConsumedCorrectly() {
    // GIVEN
    consumer.schedulePollTask(() -> consumer.addRecord(record(TOPIC, PARTITION, "Romania", 19_410_000)));
    consumer.schedulePollTask(() -> countryPopulationConsumer.stop());

    HashMap<TopicPartition, Long> startOffsets = new HashMap<>();
    TopicPartition tp = new TopicPartition(TOPIC, PARTITION);
    startOffsets.put(tp, 0L);
    consumer.updateBeginningOffsets(startOffsets);

    // WHEN
    countryPopulationConsumer.startByAssigning(TOPIC, PARTITION);

    // THEN
    assertThat(updates).hasSize(1);
    assertThat(consumer.closed()).isTrue();
}
```

首先，我们设置了`MockConsumer.` ,首先使用`addRecord` 方法`.`向消费者添加一条记录

首先要记住的是**我们不能在分配或订阅主题**之前添加记录。这就是为什么我们使用`schedulePollTask` 方法`.`来调度轮询任务。我们调度的任务将在第一次轮询时运行，然后才获取记录。因此，记录的添加将发生在分配发生之后。

同样重要的是，**不能添加到`MockConsumer` 记录中，这些记录不属于分配给它的主题和分区**。

然后，为了确保消费者不会无限期地运行，我们将其配置为在第二次轮询时关闭。

此外，我们必须设置起始偏移量。我们使用`updateBeginningOffsets `方法来完成这项工作。

最后，我们检查我们是否正确地使用了更新，并且关闭了`consumer`。

### 3.3.订阅主题

现在，让我们为我们的`startBySubscribing`方法创建一个测试:

```java
@Test
void whenStartingBySubscribingToTopic_thenExpectUpdatesAreConsumedCorrectly() {
    // GIVEN
    consumer.schedulePollTask(() -> {
        consumer.rebalance(Collections.singletonList(new TopicPartition(TOPIC, 0)));
        consumer.addRecord(record("Romania", 1000, TOPIC, 0));
    });
    consumer.schedulePollTask(() -> countryPopulationConsumer.stop());

    HashMap<TopicPartition, Long> startOffsets = new HashMap<>();
    TopicPartition tp = new TopicPartition(TOPIC, 0);
    startOffsets.put(tp, 0L);
    consumer.updateBeginningOffsets(startOffsets);

    // WHEN
    countryPopulationConsumer.startBySubscribing(TOPIC);

    // THEN
    assertThat(updates).hasSize(1);
    assertThat(consumer.closed()).isTrue();
}
```

在这种情况下，**在添加记录之前要做的第一件事是重新平衡**。我们通过调用模拟重新平衡的`rebalance `方法来实现这一点。

其余同`startByAssigning`测试用例。

### 3.4.控制轮询循环

我们可以用多种方式控制轮询循环。

第一个选项是**安排一个轮询任务**，就像我们在上面的测试中所做的那样。我们通过`schedulePollTask, `来实现，它将一个`Runnable`作为参数。当我们调用`poll`方法时，我们调度的每个任务都会运行。

我们的第二个选择是**调用`wakeup `方法**。通常，我们就是这样打断一个长时间的民意调查电话的。实际上，这就是我们如何在`CountryPopulationConsumer.`中实现`stop`方法的

最后，我们可以使用`setPollException` 方法**设置要抛出的异常:**

```java
@Test
void whenStartingBySubscribingToTopicAndExceptionOccurs_thenExpectExceptionIsHandledCorrectly() {
    // GIVEN
    consumer.schedulePollTask(() -> consumer.setPollException(new KafkaException("poll exception")));
    consumer.schedulePollTask(() -> countryPopulationConsumer.stop());

    HashMap<TopicPartition, Long> startOffsets = new HashMap<>();
    TopicPartition tp = new TopicPartition(TOPIC, 0);
    startOffsets.put(tp, 0L);
    consumer.updateBeginningOffsets(startOffsets);

    // WHEN
    countryPopulationConsumer.startBySubscribing(TOPIC);

    // THEN
    assertThat(pollException).isInstanceOf(KafkaException.class).hasMessage("poll exception");
    assertThat(consumer.closed()).isTrue();
}
```

### 3.5.模拟末端偏移和分区信息

如果我们的消费逻辑基于结束偏移量或分区信息，我们也可以使用`MockConsumer`来模拟它们。

当我们想要模拟末端偏移量时，我们可以使用`addEndOffsets`和`updateEndOffsets` 方法。

如果我们想要模仿分区信息，我们可以使用`updatePartitions`方法。

## 4.结论

在本文中，我们探索了如何使用`MockConsumer` 来测试 Kafka 消费应用程序。

首先，我们看了一个消费者逻辑的例子，哪些是需要测试的重要部分。然后，我们使用`MockConsumer`测试了一个简单的 Kafka 消费者应用程序。

在这个过程中，我们了解了`MockConsumer`的特性以及如何使用它。

和往常一样，所有这些代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926181713/https://github.com/eugenp/tutorials/tree/master/apache-kafka)