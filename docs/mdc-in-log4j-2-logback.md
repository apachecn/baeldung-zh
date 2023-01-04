# 使用映射诊断上下文(MDC)改进了 Java 日志记录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mdc-in-log4j-2-logback>

## 1.概观

在本教程中，我们将探索如何使用`Mapped Diagnostic Context` (MDC)来改进应用程序日志记录。

`Mapped Diagnostic Context`提供了一种方法来丰富日志信息，这些信息在日志记录实际发生的范围内是不可用的，但是对于更好地跟踪程序的执行确实是有用的。

## 2.为什么使用 MDC

假设我们要写一个转账的软件。

我们设置了一个`Transfer`类来表示一些基本信息——唯一的转账 id 和发送者的姓名:

```java
public class Transfer {
    private String transactionId;
    private String sender;
    private Long amount;

    public Transfer(String transactionId, String sender, long amount) {
        this.transactionId = transactionId;
        this.sender = sender;
        this.amount = amount;
    }

    public String getSender() {
        return sender;
    }

    public String getTransactionId() {
        return transactionId;
    }

    public Long getAmount() {
        return amount;
    }
} 
```

为了执行传输，我们需要使用一个由简单 API 支持的服务:

```java
public abstract class TransferService {

    public boolean transfer(long amount) {
        // connects to the remote service to actually transfer money
    }

    abstract protected void beforeTransfer(long amount);

    abstract protected void afterTransfer(long amount, boolean outcome);
} 
```

可以覆盖`beforeTransfer()`和`afterTransfer()`方法，以便在传输完成前后运行定制代码。

我们将利用`beforeTransfer()`和`afterTransfer()`来**记录一些关于转移的信息。**

让我们创建服务实现:

```java
import org.apache.log4j.Logger;
import com.baeldung.mdc.TransferService;

public class Log4JTransferService extends TransferService {
    private Logger logger = Logger.getLogger(Log4JTransferService.class);

    @Override
    protected void beforeTransfer(long amount) {
        logger.info("Preparing to transfer " + amount + "$.");
    }

    @Override
    protected void afterTransfer(long amount, boolean outcome) {
        logger.info(
          "Has transfer of " + amount + "$ completed successfully ? " + outcome + ".");
    }
} 
```

这里要注意的主要问题是，当创建日志消息时，不可能访问`Transfer`对象——只能访问金额，这使得不可能记录交易 id 或发送者。

让我们设置通常的`log4j.properties`文件来登录控制台:

```java
log4j.appender.consoleAppender=org.apache.log4j.ConsoleAppender
log4j.appender.consoleAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.consoleAppender.layout.ConversionPattern=%-4r [%t] %5p %c %x - %m%n
log4j.rootLogger = TRACE, consoleAppender 
```

最后，我们将建立一个小应用程序，它能够通过一个`ExecutorService`同时运行多个传输:

```java
public class TransferDemo {

    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        TransactionFactory transactionFactory = new TransactionFactory();
        for (int i = 0; i < 10; i++) {
            Transfer tx = transactionFactory.newInstance();
            Runnable task = new Log4JRunnable(tx);            
            executor.submit(task);
        }
        executor.shutdown();
    }
}
```

注意，为了使用`ExecutorService`，我们需要将`Log4JTransferService` 的执行封装在一个适配器中，因为`executor.submit()`需要一个`Runnable`:

```java
public class Log4JRunnable implements Runnable {
    private Transfer tx;

    public Log4JRunnable(Transfer tx) {
        this.tx = tx;
    }

    public void run() {
        log4jBusinessService.transfer(tx.getAmount());
    }
} 
```

当我们运行同时管理多个传输的演示应用程序时，我们很快发现**日志并不像我们希望的那样有用。**

跟踪每笔转账的执行很复杂，因为记录的唯一有用信息是转账金额和运行该特定转账的线程的名称。

此外，不可能区分由同一线程运行的相同数量的两个不同事务，因为相关的日志行看起来基本相同:

```java
...
519  [pool-1-thread-3]  INFO Log4JBusinessService 
  - Preparing to transfer 1393$.
911  [pool-1-thread-2]  INFO Log4JBusinessService 
  - Has transfer of 1065$ completed successfully ? true.
911  [pool-1-thread-2]  INFO Log4JBusinessService 
  - Preparing to transfer 1189$.
989  [pool-1-thread-1]  INFO Log4JBusinessService 
  - Has transfer of 1350$ completed successfully ? true.
989  [pool-1-thread-1]  INFO Log4JBusinessService 
  - Preparing to transfer 1178$.
1245 [pool-1-thread-3]  INFO Log4JBusinessService 
  - Has transfer of 1393$ completed successfully ? true.
1246 [pool-1-thread-3]  INFO Log4JBusinessService 
  - Preparing to transfer 1133$.
1507 [pool-1-thread-2]  INFO Log4JBusinessService 
  - Has transfer of 1189$ completed successfully ? true.
1508 [pool-1-thread-2]  INFO Log4JBusinessService 
  - Preparing to transfer 1907$.
1639 [pool-1-thread-1]  INFO Log4JBusinessService 
  - Has transfer of 1178$ completed successfully ? true.
1640 [pool-1-thread-1]  INFO Log4JBusinessService 
  - Preparing to transfer 674$.
... 
```

**幸，`MDC`能助。**

## 3.Log4j 中的 MDC

Log4j 中的`MDC`允许我们在一个类似 map 的结构中填充一些信息，当日志消息被实际写入时，这些信息可以被 appender 访问。

MDC 结构以与`ThreadLocal`变量相同的方式在内部附加到执行线程上。

这是一个高层次的想法:

1.  用我们想让 appender 使用的信息填充 MDC
2.  然后记录一条消息
3.  并最终清除 MDC

应该更改 appender 的模式，以便检索存储在 MDC 中的变量。

因此，让我们根据这些准则来更改代码:

```java
import org.apache.log4j.MDC;

public class Log4JRunnable implements Runnable {
    private Transfer tx;
    private static Log4JTransferService log4jBusinessService = new Log4JTransferService();

    public Log4JRunnable(Transfer tx) {
        this.tx = tx;
    }

    public void run() {
        MDC.put("transaction.id", tx.getTransactionId());
        MDC.put("transaction.owner", tx.getSender());
        log4jBusinessService.transfer(tx.getAmount());
        MDC.clear();
    }
} 
```

`MDC.put()`用于在 MDC 中添加一个键和一个对应的值，而`MDC.clear()`清空 MDC。

现在让我们更改`log4j.properties`来打印我们刚刚存储在 MDC 中的信息。

改变转换模式就足够了，对 MDC 中包含的每个条目使用`%X{}`占位符，我们希望记录:

```java
log4j.appender.consoleAppender.layout.ConversionPattern=
  %-4r [%t] %5p %c{1} %x - %m - tx.id=%X{transaction.id} tx.owner=%X{transaction.owner}%n
```

现在，如果我们运行该应用程序，我们会注意到每一行还包含有关正在处理的事务的信息，这使我们更容易跟踪应用程序的执行:

```java
638  [pool-1-thread-2]  INFO Log4JBusinessService 
  - Has transfer of 1104$ completed successfully ? true. - tx.id=2 tx.owner=Marc
638  [pool-1-thread-2]  INFO Log4JBusinessService 
  - Preparing to transfer 1685$. - tx.id=4 tx.owner=John
666  [pool-1-thread-1]  INFO Log4JBusinessService 
  - Has transfer of 1985$ completed successfully ? true. - tx.id=1 tx.owner=Marc
666  [pool-1-thread-1]  INFO Log4JBusinessService 
  - Preparing to transfer 958$. - tx.id=5 tx.owner=Susan
739  [pool-1-thread-3]  INFO Log4JBusinessService 
  - Has transfer of 783$ completed successfully ? true. - tx.id=3 tx.owner=Samantha
739  [pool-1-thread-3]  INFO Log4JBusinessService 
  - Preparing to transfer 1024$. - tx.id=6 tx.owner=John
1259 [pool-1-thread-2]  INFO Log4JBusinessService 
  - Has transfer of 1685$ completed successfully ? false. - tx.id=4 tx.owner=John
1260 [pool-1-thread-2]  INFO Log4JBusinessService 
  - Preparing to transfer 1667$. - tx.id=7 tx.owner=Marc 
```

## 4.Log4j2 中的 MDC

Log4j2 中也有完全相同的特性，所以让我们看看如何使用它。

我们将首先设置一个使用 Log4j2 记录日志的 *TransferService* 子类:

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class Log4J2TransferService extends TransferService {
    private static final Logger logger = LogManager.getLogger();

    @Override
    protected void beforeTransfer(long amount) {
        logger.info("Preparing to transfer {}$.", amount);
    }

    @Override
    protected void afterTransfer(long amount, boolean outcome) {
        logger.info("Has transfer of {}$ completed successfully ? {}.", amount, outcome);
    }
} 
```

接下来让我们更改使用 MDC 的代码，在 Log4j2 中实际上它被称为`ThreadContext`:

```java
import org.apache.log4j.MDC;

public class Log4J2Runnable implements Runnable {
    private final Transaction tx;
    private Log4J2BusinessService log4j2BusinessService = new Log4J2BusinessService();

    public Log4J2Runnable(Transaction tx) {
        this.tx = tx;
    }

    public void run() {
        ThreadContext.put("transaction.id", tx.getTransactionId());
        ThreadContext.put("transaction.owner", tx.getOwner());
        log4j2BusinessService.transfer(tx.getAmount());
        ThreadContext.clearAll();
    }
} 
```

同样，`ThreadContext.put()`在 MDC 中添加一个条目，`ThreadContext.clearAll()`删除所有现有条目。

我们仍然缺少配置日志记录的`log4j2.xml`文件。

我们可以注意到，指定应该记录哪些 MDC 条目的语法与 Log4j 中使用的语法相同:

```java
<Configuration status="INFO">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout
              pattern="%-4r [%t] %5p %c{1} - %m - tx.id=%X{transaction.id} tx.owner=%X{transaction.owner}%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.baeldung.log4j2" level="TRACE" />
        <AsyncRoot level="DEBUG">
            <AppenderRef ref="stdout" />
        </AsyncRoot>
    </Loggers>
</Configuration> 
```

同样，让我们运行应用程序，我们将看到 MDC 信息被打印在日志中:

```java
1119 [pool-1-thread-3]  INFO Log4J2BusinessService 
  - Has transfer of 1198$ completed successfully ? true. - tx.id=3 tx.owner=Samantha
1120 [pool-1-thread-3]  INFO Log4J2BusinessService 
  - Preparing to transfer 1723$. - tx.id=5 tx.owner=Samantha
1170 [pool-1-thread-2]  INFO Log4J2BusinessService 
  - Has transfer of 701$ completed successfully ? true. - tx.id=2 tx.owner=Susan
1171 [pool-1-thread-2]  INFO Log4J2BusinessService 
  - Preparing to transfer 1108$. - tx.id=6 tx.owner=Susan
1794 [pool-1-thread-1]  INFO Log4J2BusinessService 
  - Has transfer of 645$ completed successfully ? true. - tx.id=4 tx.owner=Susan 
```

## 5.SLF4J 的 MDC 后勤基地

在底层日志库支持的情况下，在 SLF4J 也是可用的。

正如我们刚刚看到的，Logback 和 Log4j 都支持 MDC，所以我们不需要什么特殊的东西来使用它进行标准设置。

让我们准备通常的`TransferService`子类，这次使用 Java 的简单日志门面:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

final class Slf4TransferService extends TransferService {
    private static final Logger logger = LoggerFactory.getLogger(Slf4TransferService.class);

    @Override
    protected void beforeTransfer(long amount) {
        logger.info("Preparing to transfer {}$.", amount);
    }

    @Override
    protected void afterTransfer(long amount, boolean outcome) {
        logger.info("Has transfer of {}$ completed successfully ? {}.", amount, outcome);
    }
} 
```

现在让我们使用 MDC 的 SLF4J 风格。

在这种情况下，语法和语义与 log4j 中的相同:

```java
import org.slf4j.MDC;

public class Slf4jRunnable implements Runnable {
    private final Transaction tx;

    public Slf4jRunnable(Transaction tx) {
        this.tx = tx;
    }

    public void run() {
        MDC.put("transaction.id", tx.getTransactionId());
        MDC.put("transaction.owner", tx.getOwner());
        new Slf4TransferService().transfer(tx.getAmount());
        MDC.clear();
    }
} 
```

我们必须提供回退配置文件，`logback.xml`:

```java
<configuration>
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%-4r [%t] %5p %c{1} - %m - tx.id=%X{transaction.id} tx.owner=%X{transaction.owner}%n</pattern>
	</encoder>
    </appender>
    <root level="TRACE">
        <appender-ref ref="stdout" />
    </root>
</configuration> 
```

同样，我们将看到 MDC 中的信息被正确地添加到记录的消息中，尽管该信息没有在`log.info()`方法中明确提供:

```java
1020 [pool-1-thread-3]  INFO c.b.m.s.Slf4jBusinessService 
  - Has transfer of 1869$ completed successfully ? true. - tx.id=3 tx.owner=John
1021 [pool-1-thread-3]  INFO c.b.m.s.Slf4jBusinessService 
  - Preparing to transfer 1303$. - tx.id=6 tx.owner=Samantha
1221 [pool-1-thread-1]  INFO c.b.m.s.Slf4jBusinessService 
  - Has transfer of 1498$ completed successfully ? true. - tx.id=4 tx.owner=Marc
1221 [pool-1-thread-1]  INFO c.b.m.s.Slf4jBusinessService 
  - Preparing to transfer 1528$. - tx.id=7 tx.owner=Samantha
1492 [pool-1-thread-2]  INFO c.b.m.s.Slf4jBusinessService 
  - Has transfer of 1110$ completed successfully ? true. - tx.id=5 tx.owner=Samantha
1493 [pool-1-thread-2]  INFO c.b.m.s.Slf4jBusinessService 
  - Preparing to transfer 644$. - tx.id=8 tx.owner=John
```

值得注意的是，如果我们将 SLF4J 后端设置为不支持 MDC 的日志系统，所有相关的调用都会被跳过，而不会产生副作用。

## 6.MDC 和线程池

**MDC 实现通常使用`ThreadLocal`来存储上下文信息。**这是实现线程安全的一种简单而合理的方法。

然而，我们应该小心使用带有线程池的 MDC。

让我们看看基于`ThreadLocal`的 MDC 和线程池的组合有多危险:

1.  我们从线程池中获取一个线程。
2.  然后我们使用`MDC.put()`或`ThreadContext.put()`在 MDC 中存储一些上下文信息。
3.  我们在一些日志中使用了这些信息，但不知何故，我们忘记了清除 MDC 上下文。
4.  借用的线程回到线程池。
5.  过了一会儿，应用程序从池中获得相同的线程。
6.  因为我们上次没有清理 MDC，所以这个线程仍然拥有来自上一次执行的一些数据。

这可能会导致执行之间出现一些意外的不一致。

防止这种情况发生的一个方法是，在每次执行结束时都要记得清理 MDC 上下文。这种方法通常需要严格的人工监督，因此容易出错。

**另一种方法是使用`ThreadPoolExecutor`钩子，并在每次执行后进行必要的清理。**

为此，我们可以扩展`ThreadPoolExecutor`类并覆盖`afterExecute()`钩子:

```java
public class MdcAwareThreadPoolExecutor extends ThreadPoolExecutor {

    public MdcAwareThreadPoolExecutor(int corePoolSize, 
      int maximumPoolSize, 
      long keepAliveTime, 
      TimeUnit unit, 
      BlockingQueue<Runnable> workQueue, 
      ThreadFactory threadFactory, 
      RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        System.out.println("Cleaning the MDC context");
        MDC.clear();
        org.apache.log4j.MDC.clear();
        ThreadContext.clearAll();
    }
}
```

这样，MDC 清理将在每次正常或异常执行后自动发生。

因此，不需要手动操作:

```java
@Override
public void run() {
    MDC.put("transaction.id", tx.getTransactionId());
    MDC.put("transaction.owner", tx.getSender());

    new Slf4TransferService().transfer(tx.getAmount());
}
```

现在，我们可以用新的 executor 实现重写相同的演示:

```java
ExecutorService executor = new MdcAwareThreadPoolExecutor(3, 3, 0, MINUTES, 
  new LinkedBlockingQueue<>(), Thread::new, new AbortPolicy());

TransactionFactory transactionFactory = new TransactionFactory();

for (int i = 0; i < 10; i++) {
    Transfer tx = transactionFactory.newInstance();
    Runnable task = new Slf4jRunnable(tx);

    executor.submit(task);
}

executor.shutdown();
```

## 7.结论

MDC 有很多应用程序，主要是在运行几个不同的线程会导致交错的日志消息难以阅读的情况下。

正如我们所见，它受到 Java 中三种最广泛使用的日志框架的支持。

和往常一样，这些资源可以在 GitHub 上找到[。](https://web.archive.org/web/20220523142445/https://github.com/eugenp/tutorials/tree/master/logging-modules/log-mdc)