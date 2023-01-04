# @使用度量和 AspectJ 的定时注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/timed-metrics-aspectj>

## 1.介绍

**监控对于发现 bug 和优化性能非常有帮助。我们可以手工编写代码来添加定时器和日志，但是这会导致很多令人分心的样板文件。**

另一方面，我们可以使用由注释驱动的监控框架，例如 [Dropwizard Metrics](/web/20221208143830/https://www.baeldung.com/dropwizard-metrics) 。

在本教程中，我们将使用 [Metrics AspectJ](https://web.archive.org/web/20221208143830/https://github.com/astefanutti/metrics-aspectj) 和 Dropwizard Metrics @ `Timed` 注释来检测一个简单的类。

## 2.Maven 设置

首先，让我们将度量 AspectJ Maven 依赖项添加到我们的项目中:

```
<dependency>
    <groupId>io.astefanutti.metrics.aspectj</groupId>
    <artifactId>metrics-aspectj</artifactId>
    <version>1.2.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.astefanutti.metrics.aspectj</groupId>
    <artifactId>metrics-aspectj-deps</artifactId>
    <version>1.2.0</version>
</dependency>
```

我们使用 [`metrics-aspectj`](https://web.archive.org/web/20221208143830/https://search.maven.org/search?q=a:metrics-aspectj%20AND%20g:io.astefanutti.metrics.aspectj) 通过面向方面编程提供`metrics`，使用 [`metrics-aspectj-deps`](https://web.archive.org/web/20221208143830/https://search.maven.org/search?q=a:metrics-aspectj-deps%20AND%20g:io.astefanutti.metrics.aspectj) 提供其依赖关系。

我们还需要 [`aspectj-maven-plugin`](https://web.archive.org/web/20221208143830/https://search.maven.org/search?q=g:org.codehaus.mojo%20AND%20a:aspectj-maven-plugin) 来设置度量注释的编译时处理:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.8</version>
    <configuration>
        <complianceLevel>1.8</complianceLevel>
        <source>1.8</source>
        <target>1.8</target>
        <aspectLibraries>
            <aspectLibrary>
                <groupId>io.astefanutti.metrics.aspectj</groupId>
                <artifactId>metrics-aspectj</artifactId>
            </aspectLibrary>
        </aspectLibraries>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

现在我们的项目已经准备好插入一些 Java 代码了。

## 3.注释工具

首先，让我们创建一个方法并用`@Timed`注释对其进行注释。我们还将用计时器的名称填充`name`属性:

```
import com.codahale.metrics.annotation.Timed;
import io.astefanutti.metrics.aspectj.Metrics;

@Metrics(registry = "objectRunnerRegistryName")
public class ObjectRunner {

    @Timed(name = "timerName")
    public void run() throws InterruptedException {
        Thread.sleep(1000L);
    }
}
```

我们在类级别使用 **`@Metrics`注释，让 Metrics AspectJ 框架知道这个类有要监控的方法。**我们将`@Timed`放在方法**上来创建计时器。**

此外， `@Metrics`使用提供的注册表名称——在本例中为`objectRunnerRegistryName`——创建一个注册表来存储指标。

我们的示例代码只是休眠一秒钟来模拟一个操作。

现在，让我们定义一个类来启动应用程序并配置我们的`MetricsRegistry`:

```
public class ApplicationMain {
    static final MetricRegistry registry = new MetricRegistry();

    public static void main(String args[]) throws InterruptedException {
        startReport();

        ObjectRunner runner = new ObjectRunner();

        for (int i = 0; i < 5; i++) {
            runner.run();
        }

        Thread.sleep(3000L);
    }

    static void startReport() {
        SharedMetricRegistries.add("objectRunnerRegistryName", registry);

        ConsoleReporter reporter = ConsoleReporter.forRegistry(registry)
                .convertRatesTo(TimeUnit.SECONDS)
                .convertDurationsTo(TimeUnit.MILLISECONDS)
                .outputTo(new PrintStream(System.out))
                .build();
        reporter.start(3, TimeUnit.SECONDS);
    }
}
```

在`ApplicationMain`的`startReport`方法中，我们使用与在`@Metrics`中使用的相同的注册表名称来设置`SharedMetricRegistries`的注册表实例。

之后，我们创建一个简单的`ConsoleReporter`来报告来自`@Timed`注释方法的度量。我们应该注意到还有其他类型的记者。

我们的应用程序将调用 timed 方法五次。让我们用 Maven 编译它，然后执行它:

```
-- Timers ----------------------------------------------------------------------
ObjectRunner.timerName
             count = 5
         mean rate = 0.86 calls/second
     1-minute rate = 0.80 calls/second
     5-minute rate = 0.80 calls/second
    15-minute rate = 0.80 calls/second
               min = 1000.49 milliseconds
               max = 1003.00 milliseconds
              mean = 1001.03 milliseconds
            stddev = 1.10 milliseconds
            median = 1000.54 milliseconds
              75% <= 1001.81 milliseconds
              95% <= 1003.00 milliseconds
              98% <= 1003.00 milliseconds
              99% <= 1003.00 milliseconds
            99.9% <= 1003.00 milliseconds
```

正如我们所看到的，度量框架为我们提供了详细的统计数据，只需对我们想要检测的方法进行很小的代码更改。

我们应该注意，在没有 Maven 构建的情况下运行应用程序——例如，通过 IDE——可能不会得到上面的输出。为了工作，我们需要确保 AspectJ 编译插件包含在构建中。

## 4.结论

在本教程中，我们研究了如何用 Metrics AspectJ 来测试一个简单的 Java 应用程序。

我们发现 Metrics AspectJ 注释是检测代码的好方法，不需要像 Spring、JEE 或 Dropwizard 这样的大型应用程序框架。相反，通过使用方面，我们能够在编译时添加拦截器。

与往常一样，该示例的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/metrics)