# 测试时在 Spring Boot 设置日志级别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-testing-log-level>

## 1。概述

在本教程中，我们将学习如何在运行 Spring Boot 应用程序的测试时**设置日志级别。**

虽然我们在测试通过时可以忽略日志，但是如果我们需要**诊断失败的测试**，选择正确的日志级别是至关重要的。

## 2。日志级别的重要性

正确配置日志级别可以为我们节省大量时间。

例如，如果测试在 CI 服务器上失败，但是在我们的开发机器上通过，**我们将不能诊断失败的测试，除非我们有足够的日志输出**。相反，如果我们记录了太多的细节，可能就更难找到有用的信息。

为了获得正确的细节，我们可以微调应用程序包的日志级别。如果我们发现一个 Java 包对我们的测试更重要，我们可以给它一个较低的级别，比如`DEBUG`。类似地，为了避免我们的日志中有太多的噪音，我们可以配置一个更高的级别，比如为不太重要的包配置`INFO`或`ERROR,`。

让我们探索设置日志记录级别的各种方法。

## 3。`application.properties` 中的记录设置

如果我们想在测试中用**修改日志级别**，我们可以在`src/test/resources/` `application.properties`中设置一个属性:

```java
logging.level.com.baeldung.testloglevel=DEBUG
```

该属性将**为`com.baeldung.testloglevel` 包专门设置** **日志级别**。

类似地，我们可以通过**设置根日志级别**来改变所有包的日志级别:

```java
logging.level.root=INFO
```

现在，让我们通过添加一个写一些日志的 REST 端点来尝试我们的日志设置:

```java
@RestController
public class TestLogLevelController {

    private static final Logger LOG = LoggerFactory.getLogger(TestLogLevelController.class);

    @Autowired
    private OtherComponent otherComponent;

    @GetMapping("/testLogLevel")
    public String testLogLevel() {
        LOG.trace("This is a TRACE log");
        LOG.debug("This is a DEBUG log");
        LOG.info("This is an INFO log");
        LOG.error("This is an ERROR log");

        otherComponent.processData();

        return "Added some log output to console...";
    }

}
```

正如所料，如果我们在测试中调用这个端点`, ` **，我们将能够看到来自`TestLogLevelController`的`DEBUG`日志**:

```java
2019-04-01 14:08:27.545 DEBUG 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is a DEBUG log
2019-04-01 14:08:27.545  INFO 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an INFO log
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an ERROR log
2019-04-01 14:08:27.546  INFO 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an INFO log from another package
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an ERROR log from another package 
```

像这样设置日志级别是非常容易的，如果我们的测试用`@SpringBootTest`注释，我们肯定应该这样做。但是，如果我们不使用该注释，我们将不得不以不同的方式配置日志级别。

### 3.1。基于配置文件的日志记录设置

尽管在大多数情况下，将设置放入`src/test/application.properties `中是可行的，但在某些情况下，我们可能希望**为一个测试或一组测试**设置不同的设置。

在这种情况下，**我们可以通过使用`ActiveProfiles`注释将[弹簧剖面](/web/20220628063812/https://www.baeldung.com/spring-profiles)添加到我们的测试**中:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT, classes = TestLogLevelApplication.class)
@EnableAutoConfiguration(exclude = SecurityAutoConfiguration.class)
@ActiveProfiles("logging-test")
public class TestLogLevelWithProfileIntegrationTest {

    // ...

}
```

我们的日志记录设置将保存在`src/test/resources`中的一个特殊的`application-logging-test.properties`文件中:

```java
logging.level.com.baeldung.testloglevel=TRACE
logging.level.root=ERROR
```

如果我们用描述的设置从我们的测试中调用`TestLogLevelController`，我们现在将从我们的控制器中看到`TRACE `日志，并且将不再有来自其他包的`INFO `日志:

```java
2019-04-01 14:08:27.545 DEBUG 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is a DEBUG log
2019-04-01 14:08:27.545  INFO 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an INFO log
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an ERROR log
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an ERROR log from another package
```

## 4。配置回退

如果我们使用在 Spring Boot 默认使用的[回退](/web/20220628063812/https://www.baeldung.com/logback)，我们可以**在`src/test/resources:`内设置`logback-test.xml`文件**中的日志级别

```java
<configuration>
    <include resource="/org/springframework/boot/logging/logback/base.xml"/>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
            </pattern>
        </encoder>
    </appender>
    <root level="error">
        <appender-ref ref="STDOUT"/>
    </root>
    <logger name="com.baeldung.testloglevel" level="debug"/>
</configuration>
```

上面的例子显示了如何在我们的测试日志配置中设置日志级别。根日志级别被设置为`INFO `，我们的`com.baeldung.testloglevel`包的日志级别被设置为`DEBUG`。

同样，让我们检查应用上述设置后的输出:

```java
2019-04-01 14:08:27.545 DEBUG 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is a DEBUG log
2019-04-01 14:08:27.545  INFO 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an INFO log
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an ERROR log
2019-04-01 14:08:27.546  INFO 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an INFO log from another package
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an ERROR log from another package 
```

### 4.1。基于配置文件的回退配置

**为我们的测试设置特定于概要文件的配置**的另一种方法是在`application.properties`中为我们的概要文件设置`logging.config `属性:

```java
logging.config=classpath:logback-testloglevel.xml
```

或者，**如果我们想在我们的类路径上有一个单独的日志回溯配置，**我们可以使用`logback.xml`中的`springProfile `元素:

```java
<configuration>
    <include resource="/org/springframework/boot/logging/logback/base.xml"/>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
            </pattern>
        </encoder>
    </appender>
    <root level="error">
        <appender-ref ref="STDOUT"/>
    </root>
    <springProfile name="logback-test1">
        <logger name="com.baeldung.testloglevel" level="info"/>
    </springProfile>
    <springProfile name="logback-test2">
        <logger name="com.baeldung.testloglevel" level="trace"/>
    </springProfile>
</configuration>
```

现在，如果我们在测试中用概要文件`logback-test1`调用`TestLogLevelController`，我们将得到以下输出:

```java
2019-04-01 14:08:27.545  INFO 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an INFO log
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an ERROR log
2019-04-01 14:08:27.546  INFO 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an INFO log from another package
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an ERROR log from another package 
```

相反，如果我们将配置文件更改为`logback-test2`，输出将为:

```java
2019-04-01 14:08:27.545 DEBUG 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is a DEBUG log
2019-04-01 14:08:27.545  INFO 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an INFO log
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an ERROR log
2019-04-01 14:08:27.546  INFO 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an INFO log from another package
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an ERROR log from another package 
```

## 5。一个 Log4J 替代方案

或者，如果我们使用 [Log4J2](/web/20220628063812/https://www.baeldung.com/log4j2-appenders-layouts-filters) ，我们可以**在`src/test/resources:`内设置`log4j2-spring.xml`文件**中的日志级别

```java
<Configuration>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout
                    pattern="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />
        </Console>
    </Appenders>

    <Loggers>
        <Logger name="com.baeldung.testloglevel" level="debug" />

        <Root level="info">
            <AppenderRef ref="Console" />
        </Root>
    </Loggers>
</Configuration>
```

我们可以通过设置`application.properties`中的`logging.config `属性来设置我们的`Log4J `配置的路径:

```java
logging.config=classpath:log4j-testloglevel.xml
```

最后，让我们检查应用上述设置后的输出:

```java
2019-04-01 14:08:27.545 DEBUG 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is a DEBUG log
2019-04-01 14:08:27.545  INFO 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an INFO log
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.testloglevel.TestLogLevelController  : This is an ERROR log
2019-04-01 14:08:27.546  INFO 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an INFO log from another package
2019-04-01 14:08:27.546 ERROR 56585 --- [nio-8080-exec-1] c.b.component.OtherComponent  : This is an ERROR log from another package 
```

## 6。结论

在本文中，我们学习了在测试 Spring Boot 应用程序时**如何设置日志级别。然后，我们探索了许多不同的配置方法。**

在 Spring Boot 的`application.properties` 中设置日志级别是最简单的选择，尤其是当我们使用`@SpringBootTest`注释时。

和往常一样，这些例子的源代码在 GitHub 上。