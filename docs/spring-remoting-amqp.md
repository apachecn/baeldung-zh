# AMQP 的春季远程处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-remoting-amqp>

## 1。概述

在本系列的[前几期](/web/20220627075159/https://www.baeldung.com/spring-remoting-http-invoker)中，我们看到了如何利用`Spring Remoting`和相关技术在服务器和客户机之间的 HTTP 通道上实现同步`Remote Procedure Calls`。

在本文中，我们将在`AMQP`的基础上**探索`Spring Remoting`，这使得在利用本质上异步的介质**的同时执行同步`RPC`成为可能。

## 2。安装 RabbitMQ

我们可以使用各种与`AMQP`兼容的消息系统，我们选择`RabbitMQ`是因为它是一个成熟的平台，在`Spring –`完全受支持。这两种产品由同一家公司(Pivotal)管理。

如果你不熟悉`AMQP`或`RabbitMQ`，你可以看看我们的[快速介绍](/web/20220627075159/https://www.baeldung.com/rabbitmq)。

所以，第一步是安装并启动`RabbitMQ`。有各种各样的方法来安装它——只要按照官方指南中提到的[的步骤选择你喜欢的方法。](https://web.archive.org/web/20220627075159/https://www.rabbitmq.com/download.html)

## 3。Maven 依赖关系

我们将设置服务器和客户端`Spring Boot`应用程序来展示`AMQP Remoting`是如何工作的。正如`Spring Boot`经常出现的情况，我们只需选择并导入正确的启动依赖项，[，如下文](/web/20220627075159/https://www.baeldung.com/spring-boot-starters)所述:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

我们明确排除了`spring-boot-starter-tomcat`,因为我们不需要任何嵌入式`HTTP`服务器——如果我们允许`Maven`在类路径中导入所有可传递的依赖项，那么它将自动启动。

## 4。服务器应用程序

### 4.1。公开服务

正如我们在以前的文章中所展示的，我们将公开一个模拟可能的远程服务的`CabBookingService`。

让我们从声明一个 bean 开始，这个 bean 实现了我们想要远程调用的服务的接口。这是将在服务器端实际执行服务调用的 bean:

```java
@Bean 
CabBookingService bookingService() {
    return new CabBookingServiceImpl();
}
```

然后让我们定义服务器将从中检索调用的队列。在这种情况下，在构造函数中为它指定一个名称就足够了:

```java
@Bean 
Queue queue() {
    return new Queue("remotingQueue");
}
```

从前面的文章中我们已经知道，**`Spring Remoting`的一个主要概念是`Service Exporter`** ，这个组件实际上**从某个来源收集调用请求**──在本例中是一个`RabbitMQ`队列─ **,并在服务实现**上调用所需的方法。

在这种情况下，我们定义了一个`AmqpInvokerServiceExporter`，如您所见，它需要一个对`AmqpTemplate`的引用。`AmqpTemplate`类是由`Spring Framework`提供的，它简化了与`AMQP-`兼容的消息系统的处理，就像`JdbcTemplate`简化了处理数据库一样。

我们不会显式定义这样的`AmqpTemplate` bean，因为它将由`Spring Boot`的自动配置模块自动提供:

```java
@Bean AmqpInvokerServiceExporter exporter(
  CabBookingService implementation, AmqpTemplate template) {

    AmqpInvokerServiceExporter exporter = new AmqpInvokerServiceExporter();
    exporter.setServiceInterface(CabBookingService.class);
    exporter.setService(implementation);
    exporter.setAmqpTemplate(template);
    return exporter;
}
```

最后，我们需要**定义一个`container`，它负责消费队列中的消息，并将它们转发给某个指定的侦听器**。

然后我们将**将这个`container`连接到我们在上一步中创建的`service exporter,`****，以允许它接收排队的消息**。这里的`ConnectionFactory`是由`Spring Boot`自动提供的，与`AmqpTemplate`的方式相同:

```java
@Bean 
SimpleMessageListenerContainer listener(
  ConnectionFactory facotry, 
  AmqpInvokerServiceExporter exporter, 
  Queue queue) {

    SimpleMessageListenerContainer container
     = new SimpleMessageListenerContainer(facotry);
    container.setMessageListener(exporter);
    container.setQueueNames(queue.getName());
    return container;
}
```

### 4.2。配置

让我们记住设置`application.properties`文件以允许`Spring Boot`配置基本对象。显然，参数值也将取决于`RabbitMQ` 的安装方式。

例如，当`RabbitMQ`在运行本示例的同一台机器上运行时，以下配置可能是合理的:

```java
spring.rabbitmq.dynamic=true
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.host=localhost
```

## 5。客户端应用程序

### 5.1。调用远程服务

让我们现在处理客户。同样，我们需要**定义调用消息将被写入**的队列。我们需要仔细检查客户机和服务器是否使用了相同的名称。

```java
@Bean 
Queue queue() {
    return new Queue("remotingQueue");
}
```

在客户端，我们需要比服务器端稍微复杂一点的设置。事实上，我们需要**用相关的`Binding`定义一个`Exchange`** :

```java
@Bean 
Exchange directExchange(Queue someQueue) {
    DirectExchange exchange = new DirectExchange("remoting.exchange");
    BindingBuilder
      .bind(someQueue)
      .to(exchange)
      .with("remoting.binding");
    return exchange;
}
```

关于`RabbitMQ`作为`Exchanges`和`Bindings`的主要概念的一个很好的介绍可以在[这里](https://web.archive.org/web/20220627075159/https://www.rabbitmq.com/tutorials/tutorial-four-java.html)找到。

由于 `Spring Boot`不自动配置`AmqpTemplate`，我们必须自己设置一个，指定一个 r `outing key`。这样做时，我们需要再次检查`routing key`和`exchange`是否与上一步中用于定义`Exchange`的匹配:

```java
@Bean RabbitTemplate amqpTemplate(ConnectionFactory factory) {
    RabbitTemplate template = new RabbitTemplate(factory);
    template.setRoutingKey("remoting.binding");
    template.setExchange("remoting.exchange");
    return template;
}
```

然后，正如我们对其他`Spring Remoting`实现所做的那样，我们**定义了一个`FactoryBean`，它将产生远程公开的服务的本地代理**。这里没有什么太花哨的，我们只需要提供远程服务的接口:

```java
@Bean AmqpProxyFactoryBean amqpFactoryBean(AmqpTemplate amqpTemplate) {
    AmqpProxyFactoryBean factoryBean = new AmqpProxyFactoryBean();
    factoryBean.setServiceInterface(CabBookingService.class);
    factoryBean.setAmqpTemplate(amqpTemplate);
    return factoryBean;
}
```

**我们现在可以使用远程服务，就像它被声明为本地 bean 一样:**

```java
CabBookingService service = context.getBean(CabBookingService.class);
out.println(service.bookRide("13 Seagate Blvd, Key Largo, FL 33037"));
```

### 5.2。设置

同样对于客户端应用程序，我们必须正确选择`application.properties`文件中的值。在常见的设置中，这些将与服务器端使用的完全匹配。

### 5.3。运行示例

这应该足以演示通过`RabbitMQ`的远程调用。然后让我们启动 RabbitMQ、服务器应用程序和调用远程服务的客户机应用程序。

幕后发生的是， **`AmqpProxyFactoryBean`将构建一个实现`CabBookingService`** 的代理。

当在该代理上调用一个方法时，它在`RabbitMQ`上将一个消息排队，在其中指定调用的所有参数和用于发送回结果的队列的名称。

消息是从调用实际实现的`AmqpInvokerServiceExporter`中消费的。然后，它将结果收集到一条消息中，并将其放在传入消息中指定名称的队列中。

`AmqpProxyFactoryBean`接收回结果，并最终返回最初在服务器端生成的值。

## 6。结论

在本文中，我们看到了如何使用`Spring Remoting`在消息传递系统之上提供 RPC。

对于我们可能更喜欢利用`RabbitMQ`的异步性的主要场景来说，这可能不是正确的方法，但是在一些选定的和有限的场景中，同步调用可能更容易理解，开发起来也更快更简单。

像往常一样，你可以在 GitHub 的[上找到源代码。](https://web.archive.org/web/20220627075159/https://github.com/eugenp/tutorials/tree/master/spring-remoting-modules/remoting-amqp)