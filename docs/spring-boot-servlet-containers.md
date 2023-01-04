# 比较 Spring Boot 的嵌入式 Servlet 容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-servlet-containers>

## 1.介绍

云原生应用和微服务的日益流行产生了对嵌入式 servlet 容器的需求。Spring Boot 允许开发人员使用 3 种最成熟的容器轻松构建应用程序或服务:Tomcat、Undertow 和 Jetty。

在本教程中，我们将演示一种方法，使用启动时和某些负载下获得的指标快速比较容器实现。

## 2.属国

我们对每个可用容器实现的设置总是要求我们在`pom.xml`中声明对`spring-boot-starter-web`的依赖。

一般来说，我们希望将父节点指定为`spring-boot-starter-parent`，然后包含我们想要的启动器:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies> 
```

### 2.1.雄猫

使用 Tomcat 时不需要更多的依赖项，因为在使用`spring-boot-starter-web`时默认包含它。

### 2.2.码头

为了使用 Jetty，我们首先需要从`spring-boot-starter-web`中排除`spring-boot-starter-tomcat`。

然后，我们简单地声明对`spring-boot-starter-jetty`的依赖:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency> 
```

### 2.3.逆流

除了我们使用`spring-boot-starter-undertow`作为依赖项之外，设置回流与 Jetty 相同:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

### 2.4.执行机构

我们将使用 Spring Boot 的致动器作为一种方便的方式来对系统施加压力和查询指标。

查看[这篇文章](/web/20220626194614/https://www.baeldung.com/spring-boot-actuators)了解致动器的详细信息。我们只需在我们的`pom`中添加一个依赖项，使其可用:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2.5.阿帕奇长凳

Apache Bench 是一个开源负载测试工具，与 Apache web 服务器捆绑在一起。

Windows 用户可以从链接[的第三方供应商下载 Apache。如果 Apache 已经安装在您的 Windows 机器上，您应该能够在您的`apache/bin`目录中找到`ab.exe`。](https://web.archive.org/web/20220626194614/https://httpd.apache.org/docs/current/platform/windows.html#down)

如果您在 Linux 机器上，`ab`可以使用`apt-get`安装:

```
$ apt-get install apache2-utils
```

## 3.启动指标

### 3.1.收藏品

为了收集我们的启动指标，我们将注册一个事件处理程序来触发 Spring Boot 的`ApplicationReadyEvent`。

我们将通过直接使用致动器组件使用的`MeterRegistry`以编程方式提取我们感兴趣的指标:

```
@Component
public class StartupEventHandler {

    // logger, constructor

    private String[] METRICS = {
      "jvm.memory.used", 
      "jvm.classes.loaded", 
      "jvm.threads.live"};
    private String METRIC_MSG_FORMAT = "Startup Metric >> {}={}";

    private MeterRegistry meterRegistry;

    @EventListener
    public void getAndLogStartupMetrics(
      ApplicationReadyEvent event) {
        Arrays.asList(METRICS)
          .forEach(this::getAndLogActuatorMetric);
    }

    private void processMetric(String metric) {
        Meter meter = meterRegistry.find(metric).meter();
        Map<Statistic, Double> stats = getSamples(meter);

        logger.info(METRIC_MSG_FORMAT, metric, stats.get(Statistic.VALUE).longValue());
    }

    // other methods
}
```

通过在启动时在事件处理程序中记录感兴趣的指标，我们避免了手动查询 Actuator REST 端点或运行独立的 JMX 控制台的需要。

### 3.2.选择

Actuator 提供了大量现成的指标。我们选择了 3 个指标来帮助获得服务器启动后关键运行时特征的高级概述:

*   `jvm.memory.used`–自启动以来 JVM 使用的总内存
*   `jvm.classes.loaded`–加载的类别总数
*   `jvm.threads.live`–活动线程的总数。在我们的测试中，这个值可以被视为“静态”的线程数

## 4.运行时指标

### 4.1.收藏品

除了提供启动度量之外，当我们运行 Apache Bench 时，我们将使用执行器公开的 */metrics* 端点作为目标 URL，以便让应用程序处于负载之下。

为了在负载下测试一个真实的应用程序，我们可以使用应用程序提供的端点。

一旦服务器启动，我们将得到一个命令提示符并执行`ab`:

```
ab -n 10000 -c 10 http://localhost:8080/actuator/metrics
```

在上面的命令中，我们使用 10 个并发线程指定了总共 10，000 个请求。

### 4.2.选择

Apache Bench 能够非常快速地为我们提供一些有用的信息，包括连接时间和在特定时间内得到服务的请求的百分比。

出于我们的目的，**我们关注每秒请求数和每次请求的时间(平均值)。**

## 5.结果

启动时，我们发现**Tomcat、Jetty 和 under flow 的内存占用与**相当，under flow 需要的内存比其他两个稍多，Jetty 需要的内存最少。

对于我们的基准测试，我们发现**Tomcat、Jetty 和 under flow 的性能与**相当，但是**under flow 显然是最快的，Jetty 只是稍慢一些。**

| 公制的 | 雄猫 | 码头 | 逆流 |
| 已用内存(MB) | One hundred and sixty-eight | One hundred and fifty-five | One hundred and sixty-four |
| jvm.classes.loaded | Nine thousand eight hundred and sixty-nine | Nine thousand seven hundred and eighty-four | Nine thousand seven hundred and eighty-seven |
| jvm.threads.live | Twenty-five | Seventeen | Nineteen |
| 每秒请求数 | One thousand five hundred and forty-two | One thousand six hundred and twenty-seven | One thousand six hundred and fifty |
| 每个请求的平均时间(毫秒) | Six point four eight three | Six point one four eight | Six point zero five nine |

请注意，度量标准自然代表了基本项目；您自己的应用程序的指标肯定会有所不同。

## 6.基准讨论

开发适当的基准测试来执行服务器实现的全面比较可能会变得复杂。为了提取最相关的信息，**关键是要清楚地了解什么对于所讨论的用例**是重要的。

值得注意的是，本例中收集的基准测量是使用一个非常特殊的工作负载进行的，该工作负载由对执行器端点的 HTTP GET 请求组成。

**预计不同的工作负载可能会导致不同容器实现之间不同的相对度量**。如果需要更健壮或者更精确的测量，那么建立一个与生产用例更加匹配的测试计划将是一个非常好的主意。

此外，更复杂的基准测试解决方案，如 [JMeter](/web/20220626194614/https://www.baeldung.com/jmeter) 或 [Gatling](/web/20220626194614/https://www.baeldung.com/introduction-to-gatling) 可能会产生更有价值的见解。

## 7.选择容器

**选择正确的容器实现应该基于许多因素，这些因素不能仅仅用一些指标来概括**。舒适度、功能、可用配置选项和策略通常同等重要，甚至更重要。

## 8.结论

在本文中，我们研究了 Tomcat、Jetty 和 Undertow 嵌入式 servlet 容器的实现。我们通过查看 Actuator 组件公开的指标，检查了每个容器在默认配置下启动时的运行时特征。

我们针对正在运行的系统执行了一个人为的工作负载，然后使用 Apache Bench 测量了性能。

最后，我们讨论了这种策略的优点，并提到了在比较实现基准时需要记住的一些事情。和往常一样，所有源代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220626194614/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-deployment)