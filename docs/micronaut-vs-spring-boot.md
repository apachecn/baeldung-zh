# 麦克诺特对 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/micronaut-vs-spring-boot>

## 1.概观

在本教程中，我们将比较[微诺特](https://web.archive.org/web/20220825173020/https://micronaut.io/)和 [Spring Boot](https://web.archive.org/web/20220825173020/https://spring.io/projects/spring-boot) 。 [Spring Boot](/web/20220825173020/https://www.baeldung.com/spring-boot) 是流行的 Spring 框架的一部分，用于快速启动和运行 Spring 应用程序。Micronaut 是一个基于 JVM 的框架，旨在解决 Spring/Spring Boot 的一些弱点。

我们将在几个方面比较这两个框架。首先，我们将比较创建新应用程序的容易程度、语言支持和其他配置选项。然后我们将看两个简单的 REST 应用程序。最后，我们将比较代码并测量性能差异。

## 2.特征

在接下来的小节中，我们将分解两个框架中的几个特性。

### 2.1.设置

首先，我们将比较在两种框架中启动和运行新应用程序的容易程度。

微诺特和 Spring Boot 都为创建新的应用程序提供了多种便捷的方法。例如，我们可以通过命令行界面使用任一框架创建一个新的应用程序。或者，我们可以为 Spring Boot 使用 [Spring Initializr](https://web.archive.org/web/20220825173020/https://start.spring.io/) ，或者为微机器人使用类似的工具 [Launch](https://web.archive.org/web/20220825173020/https://micronaut.io/launch/) 。

就 IDE 支持而言，我们可以为大多数流行的 IDE 使用 Spring Boot 插件，包括它的 Eclipse 风格、 [Eclipse Spring 工具套件](/web/20220825173020/https://www.baeldung.com/eclipse-sts-spring)。如果我们使用 IntelliJ，我们有一个 Micronaut 插件。

### 2.2.语言支持

当我们转向语言支持时，我们会发现 Spring Boot 和 Micronaut 几乎是一样的。对于这两种框架，我们可以在 Java、 [Groovy](/web/20220825173020/https://www.baeldung.com/spring-boot-groovy-web-app) 或 [Kotlin](/web/20220825173020/https://www.baeldung.com/kotlin/spring-boot-kotlin) 之间进行选择。如果选择 Java，两个框架都支持 Java 8、11、17。此外，我们可以在这两个框架中使用 Gradle 或 Maven。

### 2.3.Servlet 容器

使用 Spring Boot，我们的应用程序将默认使用 Tomcat。然而，我们也可以[配置 Spring Boot 使用突堤或者回流](/web/20220825173020/https://www.baeldung.com/spring-boot-servlet-containers)。

默认情况下，我们的 Micronaut 应用程序将运行在基于 Netty 的 HTTP 服务器上。但是，我们可以选择将应用程序切换到 Tomcat、Jetty 或 Undertow 上运行。

### 2.4.属性配置

对于 Spring Boot，我们可以在`application.properties`或`application.yml`中定义我们的[属性](/web/20220825173020/https://www.baeldung.com/properties-with-spring)。我们可以使用`application-{env}.properties`约定为不同的环境提供不同的属性。此外，我们可以使用系统属性、环境变量或 JNDI 属性来覆盖这些应用程序文件提供的属性。

我们可以使用`application.properties`、`application.yml`和`application.json`作为 Micronaut 中的属性文件。我们也可以使用相同的约定来提供特定于环境的属性文件。如果我们需要覆盖任何属性，我们可以使用系统属性或环境变量。

### 2.5.消息支持

如果我们使用与 Spring Boot 的消息传递，我们有[活动 MQ](/web/20220825173020/https://www.baeldung.com/spring-remoting-jms) 、Artemis、[兔子 MQ](/web/20220825173020/https://www.baeldung.com/spring-amqp) 和[阿帕奇卡夫卡](/web/20220825173020/https://www.baeldung.com/spring-kafka)可供我们使用。

在 Micronaut 方面，我们有 Apache Kafka、Rabbit MQ 和 Nats.io 作为选项。

### 2.6.安全性

Spring Boot 提供了五种授权策略:基本、表单登录、JWT、SAML 和 LDAP。如果我们使用 Micronaut，我们有相同的选项，只是少了 SAML。

这两个框架都为我们提供了支持。

就实际应用安全性而言，两种框架都允许我们使用注释来保护方法。

### 2.7.管理和监测

这两个框架都为我们提供了在应用程序中监控各种指标和统计数据的能力。我们可以在两个框架中定义自定义端点。我们还可以在两个框架中配置端点安全性。

然而， [Spring Boot 致动器](/web/20220825173020/https://www.baeldung.com/spring-boot-actuators)比 Micronaut 多提供了几个内置端点。

### 2.8.模板语言

我们可以使用这两种框架创建完整的全栈应用程序，使用提供的模板语言来呈现前端。

对于 Spring Boot，我们的选择是[百里香](/web/20220825173020/https://www.baeldung.com/spring-boot-crud-thymeleaf)、[阿帕奇飞人](/web/20220825173020/https://www.baeldung.com/freemarker-in-spring-mvc-tutorial)、[小胡子](/web/20220825173020/https://www.baeldung.com/spring-boot-mustache)和 Groovy。我们也可以使用 JSP，尽管不鼓励这样做。

我们在 Micronaut 有更多的选择:百里香叶，车把，阿帕奇速度，阿帕奇自由标记，摇杆，大豆/关闭，鹅卵石。

### 2.9.云支持

Spring Boot 应用依赖第三方库来实现许多特定于云的特性。

Micronaut 是为云微服务而生的。Micronaut 将为我们处理的云概念包括分布式配置、服务发现、客户端负载平衡、分布式跟踪和无服务器功能。

## 3.代码

既然我们已经比较了两个框架中的一些基本特性，让我们创建并比较两个应用程序。为了简单起见，我们将创建一个简单的 REST API 来解决基本的算术问题。我们的服务层将由一个实际上为我们计算的类组成。我们的控制器类将包含一个用于加法、减法、乘法和除法的端点。

在我们深入研究代码之前，让我们考虑一下 Spring Boot 和 Micronaut 之间的显著差异。尽管这两个框架都提供了依赖注入，但是它们的实现方式不同。我们的 Spring Boot 应用程序使用反射和代理在运行时处理依赖注入。相比之下，我们的 Micronaut 应用程序在编译时构建依赖注入数据。

### 3.1.Spring Boot 应用程序

首先，让我们在我们的 Spring Boot 应用程序中定义一个名为`ArithmeticService`的类:

```java
@Service
public class ArithmeticService {
    public float add(float number1, float number2) {
        return number1 + number2;
    }

    public float subtract(float number1, float number2) {
        return number1 - number2;
    }

    public float multiply(float number1, float number2) {
        return number1 * number2;
    }

    public float divide(float number1, float number2) {
        if (number2 == 0) {
            throw new IllegalArgumentException("'number2' cannot be zero");
        }
        return number1 / number2;
    }
}
```

接下来，让我们创建 REST 控制器:

```java
@RestController
@RequestMapping("/math")
public class ArithmeticController {
    @Autowired
    private ArithmeticService arithmeticService;

    @GetMapping("/sum/{number1}/{number2}")
    public float getSum(@PathVariable("number1") float number1, @PathVariable("number2") float number2) {
    	return arithmeticService.add(number1, number2);
    }

    @GetMapping("/subtract/{number1}/{number2}")
    public float getDifference(@PathVariable("number1") float number1, @PathVariable("number2") float number2) {
    	return arithmeticService.subtract(number1, number2);
    }

    @GetMapping("/multiply/{number1}/{number2}")
    public float getMultiplication(@PathVariable("number1") float number1, @PathVariable("number2") float number2) {
    	return arithmeticService.multiply(number1, number2);
    }

    @GetMapping("/divide/{number1}/{number2}")
    public float getDivision(@PathVariable("number1") float number1, @PathVariable("number2") float number2) {
    	return arithmeticService.divide(number1, number2);
    }
}
```

我们的控制器对于四个算术函数都有一个端点。

### 3.2.Micronaut 应用程序

现在，让我们创建 Micronaut 应用程序的服务层:

```java
@Singleton 
public class ArithmeticService {
    // implementation identical to the Spring Boot service layer
}
```

接下来，我们将使用与 Spring Boot 应用程序相同的四个端点来编写 REST 控制器:

```java
@Controller("/math")
public class ArithmeticController {
    @Inject
    private ArithmeticService arithmeticService;

    @Get("/sum/{number1}/{number2}")
    public float getSum(float number1, float number2) {
    	return arithmeticService.add(number1, number2);
    }

    @Get("/subtract/{number1}/{number2}")
    public float getDifference(float number1, float number2) {
    	return arithmeticService.subtract(number1, number2);
    }

    @Get("/multiply/{number1}/{number2}")
    public float getMultiplication(float number1, float number2) {
    	return arithmeticService.multiply(number1, number2);
    }

    @Get("/divide/{number1}/{number2}")
    public float getDivision(float number1, float number2) {
    	return arithmeticService.divide(number1, number2);
    }
}
```

我们可以看到我们非常简单的示例应用程序之间有许多相似之处。在差异方面，我们看到 Micronaut 利用 Java 的注释进行注入，而 Spring Boot 有自己的注释。此外，我们的 Micronaut REST 端点不需要对传递给方法的路径变量进行任何特殊的注释。

### 3.3.基本性能比较

Micronaut 宣传快速启动时间，所以让我们比较一下我们的两个应用程序。

首先，让我们启动 Spring Boot 应用程序，看看需要多长时间:

```java
[main] INFO  c.b.m.v.s.CompareApplication - Started CompareApplication in 3.179 seconds (JVM running for 4.164)
```

接下来，让我们看看 Micronaut 应用程序启动的速度:

```java
21:22:49.267 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 1278ms. Server Running: http://localhost:57535 
```

我们可以看到，我们的 Spring Boot 应用程序在三秒多一点的时间内启动，而在 Micronaut 中是一秒多一点。

既然我们已经了解了启动时间，让我们稍微练习一下我们的 API，然后检查一些基本的内存统计数据。启动应用程序时，我们将使用默认的内存设置。

我们将从 Spring Boot 应用程序开始。首先，让我们调用四个算术端点，然后调用内存端点:

```java
Initial: 0.25 GB 
Used: 0.02 GB 
Max: 4.00 GB 
Committed: 0.06 GB 
```

接下来，让我们对 Micronaut 应用程序进行同样的练习:

```java
Initial: 0.25 GB 
Used: 0.01 GB 
Max: 4.00 GB 
Committed: 0.03 GB
```

在这个有限的例子中，我们的两个应用程序都使用很少的内存，但是 Micronaut 使用的内存大约是 Spring Boot 应用程序的一半。

## 4.结论

在这篇文章中，我们比较了 Spring Boot 和 Micronaut。首先，我们从两个框架的概述开始。然后，我们浏览了几个特性并比较了选项。最后，我们对比了两个简单的示例应用程序。我们查看了这两个应用程序的代码，然后查看了启动和内存性能。

与往常一样，GitHub 上的示例代码可用于 [Spring Boot](https://web.archive.org/web/20220825173020/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-3) 和 [Micronaut](https://web.archive.org/web/20220825173020/https://github.com/eugenp/tutorials/tree/master/microservices-modules/micronaut) 应用程序。