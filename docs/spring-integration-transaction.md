# Spring 集成中的事务支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-integration-transaction>

## 1。概述

在本教程中，我们将看看[Spring Integration framework](/web/20220626193548/https://www.baeldung.com/spring-integration)中的事务支持。

## 2。消息流中的事务

从最早的版本开始，Spring 就提供了资源与事务同步的支持。我们经常用它来同步由多个事务管理器管理的事务。

例如，我们可以同步一个 [JMS](/web/20220626193548/https://www.baeldung.com/spring-jms) 提交和一个 [JDBC](/web/20220626193548/https://www.baeldung.com/java-jdbc) 提交。

另一方面，我们在消息流中也有更复杂的用例。它们包括非事务性资源以及各种类型的事务性资源的同步。

通常，消息传递流可以由两种不同类型的机制发起。

### 2.1。由用户进程发起的消息流

一些消息流依赖于第三方流程的启动，比如在一些消息通道上触发消息或调用消息网关方法。

我们通过 Spring 的标准[事务支持](/web/20220626193548/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)为这些流配置事务支持。Spring Integration 不必显式配置流来支持事务。Spring 集成消息流自然遵循 Spring 组件的事务语义。

例如，我们可以用`@Transactional`注释一个`ServiceActivator`或者它的方法:

```java
@Transactional
public class TxServiceActivator {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void storeTestResult(String testResult) {
        this.jdbcTemplate.update("insert into STUDENT values(?)", testResult);
        log.info("Test result is stored: {}", testResult);
    }
} 
```

我们可以从任何组件运行`storeTestResult`方法，事务上下文将照常应用。使用这种方法，我们可以完全控制事务配置。

### 2.2。由守护进程发起的消息流

我们经常使用这种类型的消息流来实现自动化。例如，`Poller`轮询消息队列以使用轮询的消息启动新的消息流，或者调度程序通过创建新消息并在预定义的时间启动消息流来调度流程。

本质上，这些是由触发进程(守护进程)发起的基于触发的流。对于这些流，每当新的消息流开始时，我们必须提供一些事务配置来创建事务上下文。

通过配置，我们将流程委托给 Spring 现有的事务支持。

在本文的其余部分，我们将重点关注对这种类型的消息流的事务支持。

## 3。轮询器交易支持

`Poller`是集成流程中的常见组件。它定期从各种来源检索数据，并通过集成链传递数据。

Spring 集成为开箱即用的轮询器提供了事务支持。任何时候我们配置一个`Poller`组件，我们都可以提供事务性配置:

```java
@Bean
@InboundChannelAdapter(value = "someChannel", poller = @Poller(value = "pollerMetadata"))
public MessageSource<File> someMessageSource() {
    ...
}

@Bean
public PollerMetadata pollerMetadata() {
    return Pollers.fixedDelay(5000)
      .advice(transactionInterceptor())
      .transactionSynchronizationFactory(transactionSynchronizationFactory)
      .get();
}

private TransactionInterceptor transactionInterceptor() {
    return new TransactionInterceptorBuilder()
      .transactionManager(txManager)
      .build();
} 
```

我们必须提供对一个`TransactionManager`和一个自定义`TransactionSynchronizationFactory`的引用，或者我们可以依赖默认值。在内部，Spring 的原生事务包装了这个过程。因此，由该轮询器发起的所有消息流都是事务性的。

## 4。交易边界

当事务启动时，事务上下文总是绑定到当前线程。不管我们的消息流中有多少端点和通道，只要消息流存在于同一个线程中，我们的事务上下文就会一直保留。

如果我们通过在某个服务中启动一个新线程来打破它，我们也将打破`Transactional`边界。本质上，交易将在该点结束。

如果线程之间发生了成功的切换，则该流程将被视为成功。这将在该点提交事务，但是流程将继续，并且仍然可能导致下游某处的`Exception`。

因此，`Exception`可以返回到流的发起者，以便事务可以在回滚中结束。这就是为什么**我们必须在线程边界可能被打破的任何地方使用事务通道**。

例如，我们应该使用 JMS、JDBC 或其他一些事务性渠道。

## 5。交易同步

在某些用例中，将某些操作与包含整个流程的事务同步是有益的。

例如，我们将演示如何使用一个`Poller`来读取一个传入的文件，并根据其内容执行数据库更新。当数据库操作完成时，它还会根据操作的成功程度来重命名文件。

在我们开始这个例子之前，理解这种方法将文件系统上的操作与事务同步是至关重要的。它并没有使本来就不是事务性的文件系统实际上变成事务性的。

事务在轮询之前开始，并在流程完成时提交或回滚，随后是文件系统上的同步操作。

首先，我们用一个简单的`Poller`定义一个`InboundChannelAdapter`:

```java
@Bean
@InboundChannelAdapter(value = "inputChannel", poller = @Poller(value = "pollerMetadata"))
public MessageSource<File> fileReadingMessageSource() {
    FileReadingMessageSource sourceReader = new FileReadingMessageSource();
    sourceReader.setDirectory(new File(INPUT_DIR));
    sourceReader.setFilter(new SimplePatternFileListFilter(FILE_PATTERN));
    return sourceReader;
}

@Bean
public PollerMetadata pollerMetadata() {
    return Pollers.fixedDelay(5000)
      .advice(transactionInterceptor())
      .transactionSynchronizationFactory(transactionSynchronizationFactory)
      .get();
} 
```

如前所述，`Poller`包含对`TransactionManager,`的引用。此外，它还包含对`TransactionSynchronizationFactory`的引用。该组件提供了文件系统操作与事务同步的机制:

```java
@Bean
public TransactionSynchronizationFactory transactionSynchronizationFactory() {
    ExpressionEvaluatingTransactionSynchronizationProcessor processor =
      new ExpressionEvaluatingTransactionSynchronizationProcessor();

    SpelExpressionParser spelParser = new SpelExpressionParser();

    processor.setAfterCommitExpression(
      spelParser.parseExpression(
        "payload.renameTo(new java.io.File(payload.absolutePath + '.PASSED'))"));

    processor.setAfterRollbackExpression(
      spelParser.parseExpression(
        "payload.renameTo(new java.io.File(payload.absolutePath + '.FAILED'))"));

    return new DefaultTransactionSynchronizationFactory(processor);
} 
```

如果事务提交，`TransactionSynchronizationFactory`将通过追加“.”来重命名文件。传递给文件名。但是，如果回滚，它将追加。失败了”。

`InputChannel`使用`FileToStringTransformer`转换有效载荷，并将其委托给`toServiceChannel`。该通道绑定到`ServiceActivator`:

```java
@Bean
public MessageChannel inputChannel() {
    return new DirectChannel();
}

@Bean
@Transformer(inputChannel = "inputChannel", outputChannel = "toServiceChannel")
public FileToStringTransformer fileToStringTransformer() {
    return new FileToStringTransformer();
} 
```

`ServiceActivator`读取传入的文件，其中包含学生的考试成绩。它将结果写入数据库。如果结果包含字符串“fail”，它将抛出`Exception`，这将导致数据库回滚:

```java
@ServiceActivator(inputChannel = "toServiceChannel")
public void serviceActivator(String payload) {

    jdbcTemplate.update("insert into STUDENT values(?)", payload);

    if (payload.toLowerCase().startsWith("fail")) {
        log.error("Service failure. Test result: {} ", payload);
        throw new RuntimeException("Service failure.");
    }

    log.info("Service success. Test result: {}", payload);
} 
```

在数据库操作成功提交或回滚后，`TransactionSynchronizationFactory`将文件系统操作与其结果同步。

## 6。结论

在本文中，我们解释了`Spring Integration`框架中的事务支持。此外，我们演示了如何将事务与文件系统等非事务性资源上的操作同步。

GitHub 上的  [提供了该示例的完整源代码。](https://web.archive.org/web/20220626193548/https://github.com/eugenp/tutorials/tree/master/spring-integration)