# Apache Camel 条件路由

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-apache-camel-conditional-routing>

## 1。概述

Apache Camel 是一个强大的开源集成框架，实现了几种已知的 T2 企业集成模式 T3。

通常，当使用 Camel 处理消息路由时，我们会希望有一种方法根据消息的内容以不同的方式处理消息。为此，Camel 从 [EIP 模式](https://web.archive.org/web/20221029120649/https://camel.apache.org/components/3.18.x/eips/enterprise-integration-patterns.html)的集合中提供了一个强大的特性，叫做[基于内容的路由器](https://web.archive.org/web/20221029120649/https://camel.apache.org/components/3.18.x/eips/choice-eip.html)。

在本教程中，**我们将看看基于某些条件路由消息的几种方法。**

## 2。依赖性

我们需要做的就是将`[camel-spring-boot-starter](https://web.archive.org/web/20221029120649/https://search.maven.org/search?q=a:camel-spring-boot-starter)`添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
     <artifactId>camel-spring-boot-starter</artifactId>
     <version>3.18.1</version>
</dependency>
```

然后，我们需要将`[camel-test-spring-junit5](https://web.archive.org/web/20221029120649/https://search.maven.org/search?q=g:org.apache.camel%20a:camel-spring-boot-starter)`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring-junit5</artifactId>
    <version>3.18.1</version>
</dependency>
```

顾名思义，这种依赖是专门针对我们的单元测试的。

## 3。定义一个简单的骆驼 Spring Boot 应用程序

在本教程中，我们示例的重点将是一个简单的 Apache Camel Spring Boot 应用程序。

因此，让我们从定义应用程序入口点开始:

```java
@SpringBootApplication
public class ConditionalRoutingSpringApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConditionalRoutingSpringApplication.class, args);
    }
}
```

正如我们所看到的，这是一个标准的 Spring Boot 应用程序。

## 4。条件路由

简单回顾一下，Apache Camel 中的 [route](https://web.archive.org/web/20221029120649/https://camel.apache.org/manual/routes.html) 是一个基本的构建块，通常由一系列步骤组成，由 Camel 按顺序执行，消费和处理消息。

例如，一个路由通常会接收一条消息，使用一个可能来自磁盘上的文件或消息队列的消费者。然后，Camel 执行路由中的其余步骤，要么以某种方式处理消息，要么将其发送到其他端点。

毫无疑问，**我们需要一种基于某些事实有条件地路由消息的方法。为此，Camel 提供了`choice`和`when`构造**。我们可以认为这相当于 Java 中的 [`if-else`语句](/web/20221029120649/https://www.baeldung.com/java-if-else)。

记住这一点，让我们继续用一些条件逻辑创建我们的第一条路线。

## 5。创建路线

在本例中，我们将根据收到的消息正文的内容，用一些条件逻辑定义一个基本路由:

```java
@Component
public class ConditionalBodyRouter extends RouteBuilder {

    @Override
    public void configure() throws Exception {

        from("direct:start-conditional")
          .routeId("conditional-body-route")
          .choice()
            .when(body().contains("Baeldung"))
              .setBody(simple("Goodbye, Baeldung!"))
              .to("mock:result-body")
            .otherwise()
              .to("mock:result-body")
          .end();
    }
}
```

正如我们在这个小例子中看到的，我们配置我们的路由来消费来自名为`start-conditional`的[直接端点](https://web.archive.org/web/20221029120649/https://camel.apache.org/components/next/direct-component.html)的消息。

现在让我们来看看路线的关键部分:

*   首先，我们使用`choice()`方法开始路线—**这告诉 Camel 接下来的线路将包含一些条件来评估**
*   接下来，`when()`方法指出了要评估的新条件——在本例中，我们只是检查消息体是否包含字符串 Baeldung。我们可以根据需要添加任意数量的条件
*   为了总结我们的路线，我们使用`otherwise()`方法来定义当前面的 when 条件都不满足时要做什么。
*   最后，使用关闭选择块的`end()` 方法终止路由。

总而言之，当我们运行我们的路由时，如果我们的消息体包含字符串`Baeldung`，我们将设置消息体为`Goodbye, Baeldung!`，并将结果发送到一个叫做`result-body`的[模拟端点](https://web.archive.org/web/20221029120649/https://camel.apache.org/components/3.18.x/mock-component.html)。

或者，我们只将原始消息路由到我们的模拟端点。

## 6.测试路线

记住最后一节，让我们继续编写一个单元测试来探索我们的路由是如何工作的:

```java
@SpringBootTest
@CamelSpringBootTest
class ConditionalBodyRouterUnitTest {

    @Autowired
    private ProducerTemplate template;

    @EndpointInject("mock:result-body")
    private MockEndpoint mock;

    @Test
    void whenSendBodyWithBaeldung_thenGoodbyeMessageReceivedSuccessfully() throws InterruptedException {
        mock.expectedBodiesReceived("Goodbye, Baeldung!");

        template.sendBody("direct:start-conditional", "Hello Baeldung Readers!");

        mock.assertIsSatisfied();
    }
}
```

正如我们所看到的，我们的测试由三个简单的步骤组成:

*   首先，让我们设置一个期望，我们的模拟端点将接收给定的消息体
*   然后，我们将使用模板向我们的`direct:start-conditional`端点发送一条消息。注意，我们将确保我们的消息体包含字符串`Baeldung`
*   为了结束我们的测试，**我们使用`assertIsSatisfied`方法来验证我们对模拟端点的最初期望已经得到满足**

该测试确认我们的条件路由工作正常。厉害！

请务必阅读我们之前的教程，了解如何为我们的 Camel routes 编写可靠的、[自包含单元测试](/web/20221029120649/https://www.baeldung.com/spring-boot-apache-camel-routes-testing)。

## 7。构建其他条件谓词

到目前为止，我们已经探索了如何构建 when [谓词](https://web.archive.org/web/20221029120649/https://camel.apache.org/manual/predicate.html)的一个选项——检查我们的交换的消息体。然而，我们还有其他几种选择。

例如，**我们还可以通过检查给定消息头的值来控制我们的条件:**

```java
@Component
public class ConditionalHeaderRouter extends RouteBuilder {

    @Override
    public void configure() throws Exception {

        from("direct:start-conditional-header")
            .routeId("conditional-header-route")
            .choice()
              .when(header("fruit").isEqualTo("Apple"))
                .setHeader("favourite", simple("Apples"))
                .to("mock:result")
              .otherwise()
                .setHeader("favourite", header("fruit"))
                .to("mock:result")
            .end();
    }
}
```

这一次，我们修改了`when`方法来查看名为 fruit 的头的值。在我们的 when 条件中使用 Camel 提供的简单语言也是完全可能的。

## 8。使用 Java bean

此外，当我们想要在谓词中使用 Java 方法调用的结果时，我们也可以使用 [Camel Bean 语言](https://web.archive.org/web/20221029120649/https://camel.apache.org/components/3.18.x/languages/bean-language.html)。

首先，我们需要创建一个包含返回布尔值的方法的 Java bean:

```java
public class FruitBean {

    public static boolean isApple(Exchange exchange) {
        return "Apple".equals(exchange.getIn().getHeader("fruit"));
    }
}
```

**这里我们也可以选择添加`Exchange`作为参数，这样 Camel 会自动将`Exchange`传递给我们的方法。**

然后我们可以继续使用我们的`when`块中的`FruitBean`:

```java
@Component
public class ConditionalBeanRouter extends RouteBuilder {

    @Override
    public void configure() throws Exception {

        from("direct:start-conditional-bean")
            .routeId("conditional-bean-route")
            .choice()
              .when(method(FruitBean.class, "isApple"))
                .setHeader("favourite", simple("Apples"))
                .to("mock:result")
              .otherwise()
                .setHeader("favourite", header("fruit"))
                 .to("mock:result")
              .endChoice()
            .end();
    }
}
```

## 9.结论

在本文中，我们学习了如何根据路由中的某种条件来路由消息。首先，我们创建了一个简单的 Camel 应用程序，用一条路由来检查消息体。

然后，我们学习了使用消息头和 Java beans 在我们的路由中构建谓词的其他几种技术。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221029120649/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-camel)