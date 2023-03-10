# 阿帕奇骆驼路线在 Spring Boot 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-apache-camel-routes-testing>

## 1。概述

Apache Camel 是一个强大的开源集成框架，实现了许多已知的 T2 企业集成模式。

在本教程中，**我们将学习如何为我们的骆驼路线**编写可靠的、自包含的单元测试。

首先，我们将使用 [Spring Boot](/web/20220707143856/https://www.baeldung.com/category/spring/spring-boot/) 创建一个基本的 Camel 应用程序。然后我们将看看如何使用 Camel 的 Spring 测试支持 API 和 [JUnit 5](/web/20220707143856/https://www.baeldung.com/junit-5) 来测试我们的应用程序。

## 2。依赖性

假设我们已经设置好项目[并配置好](/web/20220707143856/https://www.baeldung.com/apache-camel-spring-boot#maven-dependencies)与 Spring Boot 和骆驼一起工作。

然后，我们需要将`[camel-test-spring-junit5](https://web.archive.org/web/20220707143856/https://search.maven.org/search?q=g:org.apache.camel%20a:camel-test-spring-junit5)`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring-junit5</artifactId>
    <version>3.15.0</version>
    <scope>test</scope>
</dependency
```

顾名思义，这种依赖是专门针对我们的单元测试的。

## 3。定义一个简单的骆驼 Spring Boot 应用程序

在本教程中，我们测试的重点将是一个简单的 Apache Camel Spring Boot 应用程序。

因此，让我们从定义应用程序入口点开始:

```java
@SpringBootApplication
public class GreetingsFileSpringApplication {

    public static void main(String[] args) {
        SpringApplication.run(GreetingsFileSpringApplication.class, args);
    }
}
```

正如我们所看到的，这是一个标准的 Spring Boot 应用程序。

### 3.1。创建路线

接下来，我们将定义一条相当简单的路线:

```java
@Component
public class GreetingsFileRouter extends RouteBuilder {

    @Override
    public void configure() throws Exception {

        from("direct:start")
          .routeId("greetings-route")
          .setBody(constant("Hello Baeldung Readers!"))
          .to("file:output");
    }
}
```

简单概括一下，Apache Camel 中的路由是一个基本的构建块，通常由一系列步骤组成，由 Camel 按顺序执行，使用和处理消息。

正如我们在这个小例子中看到的，我们配置我们的路由来消费来自名为`start`的[直接端点](https://web.archive.org/web/20220707143856/https://camel.apache.org/components/next/direct-component.html)的消息。

然后，**我们将消息体设置为包含一个字符串`Hello Baeldung Readers!`，并使用[文件组件](https://web.archive.org/web/20220707143856/https://camel.apache.org/components/next/file-component.html)将我们的消息交换内容写入一个名为`output`的文件目录。**

我们也给我们的路线一个名为`greetings-route`的 id。在我们的路线中使用 ids 通常被认为是好的做法，并且可以在我们来测试我们的路线时帮助我们。

### 3.2。运行我们的应用程序

最后，如果我们运行我们的应用程序并向我们的直接消费者端点发送一条消息，我们应该在输出目录的一个文件中看到我们的问候文本。如果我们不指定文件名，Camel 会为我们创建一个:

```java
$ cat output/D97099B6B2958D2-0000000000000000 
Hello Baeldung Readers!
```

## 4。关于测试的一句话

一般来说，当编写干净的测试时，我们不应该依赖我们可能无法控制或可能突然停止工作的外部服务或文件系统。这可能会对我们的测试结果产生不利影响。

我们也不想在我们的路径中专门为我们的单元测试编写代码。谢天谢地，Camel 有一套专门用于测试的扩展和 API。所以我们可以把它看作是一种测试工具。

该工具包通过向 routes 发送消息并检查消息是否如预期的那样被接收，使得测试我们的 Camel 应用程序变得更加容易。

## 5。测试使用`@MockEndpoints`

记住最后一节，让我们继续编写我们的第一个单元测试:

```java
@SpringBootTest
@CamelSpringBootTest
@MockEndpoints("file:output")
class GreetingsFileRouterUnitTest {

    @Autowired
    private ProducerTemplate template;

    @EndpointInject("mock:file:output")
    private MockEndpoint mock;

    @Test
    void whenSendBody_thenGreetingReceivedSuccessfully() throws InterruptedException {
        mock.expectedBodiesReceived("Hello Baeldung Readers!");
        template.sendBody("direct:start", null);
        mock.assertIsSatisfied();
    }
}
```

让我们看一下测试的关键部分。

首先，我们开始用三个注释装饰我们的测试类:

*   `[@SpringBootTest](/web/20220707143856/https://www.baeldung.com/spring-boot-testing)`注释将确保我们的测试启动 Spring 应用程序上下文
*   我们还使用了`@CamelSpringBootRunner,` ,它为我们基于 Boot 的测试带来了基于 Spring 的 Camel 测试支持
*   **最后，我们添加了`@MockEndpoints` 注释，它告诉 Camel 我们想要模仿**的哪些端点

接下来，我们自动连接一个`ProducerTemplate`对象，这是一个允许我们向端点发送消息交换的接口。

另一个关键组件是我们使用`@EndpointInject`注入的`MockEndpoint`。**使用这个注释告诉 Camel 当路由开始时，我们想要注入我们的模拟端点。**

现在我们已经有了所有的测试设置，我们可以编写我们的测试了，它包括三个步骤:

*   首先，让我们设置一个期望，我们的模拟端点将接收给定的消息体
*   然后，我们将使用模板向我们的`direct:start`端点发送一条消息。注意，我们将发送一个`null`主体，因为我们的路由不处理传入的消息主体
*   为了结束我们的测试，**我们使用`assertIsSatisfied`方法来验证我们对模拟端点的最初期望已经得到满足**

这证实了我们的测试工作正常。厉害！我们现在有了一种使用 Camel 测试支持工具来编写自包含、独立的单元测试的方法。

## 6。结论

在本文中，我们学习了如何在 Spring Boot 测试我们的阿帕奇骆驼路线。首先，我们学习了如何使用 Spring Boot 创建一个简单的 Camel 应用程序。

然后了解了使用 Apache Camel 的内置测试支持项目中的一些可用特性来测试我们的路线的推荐方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220707143856/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-camel)