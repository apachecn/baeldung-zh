# 具有嵌套诊断上下文的 Java 日志记录(NDC)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-logging-ndc-log4j>

## 1。概述

嵌套诊断上下文(NDC)是一种机制，有助于区分来自不同来源的交叉日志消息。NDC 通过向每个日志条目添加独特的上下文信息来实现这一点。

在本文中，我们将探索 NDC 的使用及其在各种 Java 日志框架中的用法/支持。

## 2。诊断环境

在典型的多线程应用程序中，如 web 应用程序或 REST APIs，每个客户端请求由不同的线程提供服务。从这种应用程序生成的日志将是所有客户端请求和来源的混合。这使得很难从业务角度理解日志或进行调试。

嵌套诊断上下文(NDC)管理每个线程的上下文信息堆栈。NDC 中的数据可用于代码中的每个日志请求，并且可以配置为记录每个日志消息——即使在数据不在范围内的地方。每个日志消息中的上下文信息有助于根据日志的来源和上下文来区分日志。

[Mapped Diagnostic Context(MDC)](/web/20220815043922/https://www.baeldung.com/mdc-in-log4j-2-logback)也管理基于每个线程的信息，但是作为一个映射。

## 3。示例应用程序中的 NDC 堆栈

为了演示 NDC 堆栈的用法，让我们以一个向投资账户汇款的 REST API 为例。

作为输入所需的信息在`Investment`类中表示:

```
public class Investment {
    private String transactionId;
    private String owner;
    private Long amount;

    public Investment (String transactionId, String owner, Long amount) {
        this.transactionId = transactionId;
        this.owner = owner;
        this.amount = amount;
    }

    // standard getters and setters
}
```

使用`InvestmentService`向投资账户转账。这些类的完整源代码可以在 github 项目中找到。

在示例应用程序中，数据`transactionId`和`owner`被放在处理给定请求的线程中的 NDC 堆栈中。该线程中的每个日志消息中都有该数据。通过这种方式，可以跟踪每个唯一的事务，并且可以识别每个日志消息的相关上下文。

## 4。Log4j 中的 NDC

Log4j 提供了一个名为`NDC`的类，它提供静态方法来管理 NDC 堆栈中的数据。基本用法:

*   进入上下文时，使用`NDC.push()` 在当前线程中添加上下文数据
*   离开上下文时，使用`NDC.pop()`取出上下文数据
*   当退出线程时，调用`NDC.remove()`删除线程的诊断上下文并确保内存被释放(从 Log4j 1.3 开始，不再需要)

在示例应用程序中，让我们使用 NDC 在代码的相关位置添加/删除上下文数据:

```
import org.apache.log4j.NDC;

@RestController
public class Log4JController {
    @Autowired
    @Qualifier("Log4JInvestmentService")
    private InvestmentService log4jBusinessService;

    @RequestMapping(
      value = "/ndc/log4j", 
      method = RequestMethod.POST)
    public ResponseEntity<Investment> postPayment(
      @RequestBody Investment investment) {

        NDC.push("tx.id=" + investment.getTransactionId());
        NDC.push("tx.owner=" + investment.getOwner());

        log4jBusinessService.transfer(investment.getAmount());

        NDC.pop();
        NDC.pop();

        NDC.remove();

        return 
          new ResponseEntity<Investment>(investment, HttpStatus.OK);
    }
}
```

通过使用`log4j.properties`中添加器使用的`ConversionPattern`中的`%x`选项，可以在日志消息中显示 NDC 的内容:

```
log4j.appender.consoleAppender.layout.ConversionPattern 
  = %-4r [%t] %5p %c{1} - %m - [%x]%n
```

让我们将 REST API 部署到 tomcat。样品申请:

```
POST /logging-service/ndc/log4j
{
  "transactionId": "4",
  "owner": "Marc",
  "amount": 2000
}
```

我们可以在日志输出中看到诊断上下文信息:

```
48569 [http-nio-8080-exec-3]  INFO Log4JInvestmentService 
  - Preparing to transfer 2000$. 
  - [tx.id=4 tx.owner=Marc]
49231 [http-nio-8080-exec-4]  INFO Log4JInvestmentService 
  - Preparing to transfer 1500$. 
  - [tx.id=6 tx.owner=Samantha]
49334 [http-nio-8080-exec-3]  INFO Log4JInvestmentService 
  - Has transfer of 2000$ completed successfully ? true. 
  - [tx.id=4 tx.owner=Marc] 
50023 [http-nio-8080-exec-4]  INFO Log4JInvestmentService 
  - Has transfer of 1500$ completed successfully ? true. 
  - [tx.id=6 tx.owner=Samantha]
...
```

## 5。Log4j 2 中的 NDC

Log4j 2 中的 NDC 被称为线程上下文堆栈:

```
import org.apache.logging.log4j.ThreadContext;

@RestController
public class Log4J2Controller {
    @Autowired
    @Qualifier("Log4J2InvestmentService")
    private InvestmentService log4j2BusinessService;

    @RequestMapping(
      value = "/ndc/log4j2", 
      method = RequestMethod.POST)
    public ResponseEntity<Investment> postPayment(
      @RequestBody Investment investment) {

        ThreadContext.push("tx.id=" + investment.getTransactionId());
        ThreadContext.push("tx.owner=" + investment.getOwner());

        log4j2BusinessService.transfer(investment.getAmount());

        ThreadContext.pop();
        ThreadContext.pop();

        ThreadContext.clearAll();

        return 
          new ResponseEntity<Investment>(investment, HttpStatus.OK);
    }
}
```

正如 Log4j 一样，让我们使用 Log4j 2 配置文件`log4j2.xml`中的`%x`选项:

```
<Configuration status="INFO">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout
              pattern="%-4r [%t] %5p %c{1} - %m -%x%n" />
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

日志输出:

```
204724 [http-nio-8080-exec-1]  INFO Log4J2InvestmentService 
  - Preparing to transfer 1500$. 
  - [tx.id=6, tx.owner=Samantha]
205455 [http-nio-8080-exec-2]  INFO Log4J2InvestmentService 
  - Preparing to transfer 2000$. 
  - [tx.id=4, tx.owner=Marc]
205525 [http-nio-8080-exec-1]  INFO Log4J2InvestmentService 
  - Has transfer of 1500$ completed successfully ? false. 
  - [tx.id=6, tx.owner=Samantha]
206064 [http-nio-8080-exec-2]  INFO Log4J2InvestmentService 
  - Has transfer of 2000$ completed successfully ? true. 
  - [tx.id=4, tx.owner=Marc]
...
```

## 6。NDC 在伐木门面(JBoss Logging)

像 SLF4J 这样的日志门面提供了与各种日志框架的集成。SLF4J 不支持 NDC(但包含在 slf4j-ext 模块中)。JBoss Logging 是一个日志桥，就像 SLF4J 一样。JBoss 日志支持 NDC。

默认情况下，JBoss Logging 将按照以下优先顺序搜索`ClassLoader`中后端/提供者的可用性:JBoss LogManager、Log4j 2、Log4j、SLF4J 和 JDK 日志。

JBoss LogManager 作为日志提供者通常用在 WildFly 应用服务器内部。在我们的例子中，JBoss 日志桥将按照优先顺序选择下一个(Log4j 2)作为日志提供者。

让我们从在`pom.xml`中添加所需的依赖项开始:

```
<dependency>
    <groupId>org.jboss.logging</groupId>
    <artifactId>jboss-logging</artifactId>
    <version>3.3.0.Final</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220815043922/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jboss.logging%22%20AND%20a%3A%22jboss-logging%22)查看。

让我们向 NDC 堆栈添加上下文信息:

```
import org.jboss.logging.NDC;

@RestController
public class JBossLoggingController {
    @Autowired
    @Qualifier("JBossLoggingInvestmentService")
    private InvestmentService jbossLoggingBusinessService;

    @RequestMapping(
      value = "/ndc/jboss-logging", 
      method = RequestMethod.POST)
    public ResponseEntity<Investment> postPayment(
      @RequestBody Investment investment) {

        NDC.push("tx.id=" + investment.getTransactionId());
        NDC.push("tx.owner=" + investment.getOwner());

        jbossLoggingBusinessService.transfer(investment.getAmount());

        NDC.pop();
        NDC.pop();

        NDC.clear();

        return 
          new ResponseEntity<Investment>(investment, HttpStatus.OK);
    }
}
```

日志输出:

```
17045 [http-nio-8080-exec-1]  INFO JBossLoggingInvestmentService 
  - Preparing to transfer 1,500$. 
  - [tx.id=6, tx.owner=Samantha]
17725 [http-nio-8080-exec-1]  INFO JBossLoggingInvestmentService 
  - Has transfer of 1,500$ completed successfully ? true. 
  - [tx.id=6, tx.owner=Samantha]
18257 [http-nio-8080-exec-2]  INFO JBossLoggingInvestmentService 
  - Preparing to transfer 2,000$. 
  - [tx.id=4, tx.owner=Marc]
18904 [http-nio-8080-exec-2]  INFO JBossLoggingInvestmentService 
  - Has transfer of 2,000$ completed successfully ? true. 
  - [tx.id=4, tx.owner=Marc]
...
```

## 7。结论

我们已经看到了诊断上下文如何以一种有意义的方式帮助关联日志——从业务角度以及出于调试目的。这是丰富日志记录的宝贵技术，尤其是在多线程应用程序中。

本文中使用的例子可以在 Github 项目中找到。