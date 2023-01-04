# 使用 JMS 和 ActiveMQ 的 Spring Remoting

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-remoting-jms>

## 1。概述

我们在[的上一篇文章](/web/20220625222544/https://www.baeldung.com/spring-remoting-amqp)中看到了`Spring Remoting`如何被用来在异步通道之上提供`RPC`作为`AMQP`队列。然而，我们也可以使用`JMS`获得相同的结果。

事实上，在本文中，我们将探索如何使用`Spring Remoting JMS`和`Apache ActiveMQ`作为消息中间件来设置远程调用。

## 2。启动 Apache ActiveMQ 代理

`Apache [ActiveMQ](https://web.archive.org/web/20220625222544/https://activemq.apache.org/)`是一个**开源消息代理**，使应用程序能够异步交换信息，它完全兼容`Java Message Service` `API`。

为了运行我们的实验，我们首先需要设置一个运行的实例`ActiveMQ`。我们可以从几种方法中选择:遵循[官方指南](https://web.archive.org/web/20220625222544/https://activemq.apache.org/getting-started.html)中描述的步骤，将其嵌入到`Java`应用程序中，或者更简单地用下面的命令构建一个`Docker`容器:

```java
docker run -p 61616:61616 -p 8161:8161 rmohr/activemq:5.14.3
```

这将启动一个`ActiveMQ`容器，在端口 8081 上显示一个简单的管理 web GUI，通过它我们可以检查可用的队列、连接的客户端和其他管理信息。`JMS`客户端将需要使用端口 61616 连接到代理并交换消息。

## 3。Maven 依赖关系

正如在之前的文章中涉及到的`Spring Remoting`，我们将设置一个服务器和一个客户端`Spring Boot`应用程序来展示`JMS Remoting`是如何工作的。

正如通常我们仔细选择`Spring Boot`起始依赖项一样，[在这里解释为](/web/20220625222544/https://www.baeldung.com/spring-boot-starters):

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

为了在类路径中没有与`Tomcat` 相关的`.jar`文件，我们显式地排除了`spring-boot-starter-tomcat`。

这反过来会阻止`Spring Boot`的自动配置机制在应用程序启动时启动嵌入式 web 服务器，因为我们不需要它。

## 4。服务器应用程序

### 4.1。公开服务

我们将建立一个服务器应用程序，公开客户端能够调用的`CabBookingService`。

第一步是声明一个 bean，它实现了我们想要向客户端公开的服务的接口。这是将在服务器上执行业务逻辑的 bean:

```java
@Bean 
CabBookingService bookingService() {
    return new CabBookingServiceImpl();
}
```

然后，让我们定义服务器将从中检索调用的队列，并在构造函数中指定其名称:

```java
@Bean 
Queue queue() {
    return new ActiveMQQueue("remotingQueue");
}
```

从前面的文章中我们已经知道，**`Spring Remoting`的一个主要概念是`Service Exporter`，这个组件收集来自某个源**的调用请求，在本例中是一个`ApacheMQ`队列，并调用服务实现上所需的方法。

为了使用`JMS`，我们定义了一个`JmsInvokerServiceExporter`:

```java
@Bean 
JmsInvokerServiceExporter exporter(CabBookingService implementation) {
    JmsInvokerServiceExporter exporter = new JmsInvokerServiceExporter();
    exporter.setServiceInterface(CabBookingService.class);
    exporter.setService(implementation);
    return exporter;
}
```

最后，我们需要定义一个负责消费消息的侦听器。**监听器充当`ApacheMQ`和** `**JmsInvokerServiceExporter**,`之间的桥梁，它监听队列上可用的调用消息，将调用转发给服务导出器，并序列化回结果:

```java
@Bean SimpleMessageListenerContainer listener(
  ConnectionFactory factory, 
  JmsInvokerServiceExporter exporter) {

    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
    container.setConnectionFactory(factory);
    container.setDestinationName("remotingQueue");
    container.setConcurrentConsumers(1);
    container.setMessageListener(exporter);
    return container;
}
```

### 4.2。配置

让我们记住设置`application.properties`文件以允许`Spring Boot`配置一些基本对象，例如监听器需要的`ConnectionFactory`。各种参数的值主要取决于

各种参数的值主要取决于`ApacheMQ`的安装方式，以下是在运行这些示例的同一台机器上运行的`Docker`容器的合理配置:

```java
spring.activemq.broker-url=tcp://localhost:61616
spring.activemq.packages.trusted=org.springframework.remoting.support,java.lang,com.baeldung.api
```

`spring.activemq.broker-url` 参数是对`AMQ`端口的引用。相反,`spring.activemq.packages.trusted parameter`需要更深入的解释。因为 5.12.2 版的 ActiveMQ 默认拒绝任何类型的消息

**从版本 5.12.2 开始，默认情况下，ActiveMQ 拒绝任何类型为`ObjectMessage`的消息，这种消息用于交换序列化的`Java`对象，因为在某些情况下，它被认为是安全攻击的潜在媒介。**

无论如何，可以指示`AMQ`接受指定包中的序列化对象。`org.springframework.remoting.support`是包含表示远程方法调用及其结果的主要消息的包。包裹

包`com.baeldung.api`包含我们服务的参数和结果。添加`java.lang`是因为表示出租车预订结果的对象引用了一个`String`，所以我们也需要序列化它。

## 5。客户端应用程序

### 5.1。调用远程服务

让我们现在处理客户。同样，我们需要定义调用消息将被写入的队列。我们需要仔细检查客户机和服务器是否使用了相同的名称。

```java
@Bean 
Queue queue() {
    return new ActiveMQQueue("remotingQueue");
}
```

然后我们需要建立一个出口商:

```java
@Bean 
FactoryBean invoker(ConnectionFactory factory, Queue queue) {
    JmsInvokerProxyFactoryBean factoryBean = new JmsInvokerProxyFactoryBean();
    factoryBean.setConnectionFactory(factory);
    factoryBean.setServiceInterface(CabBookingService.class);
    factoryBean.setQueue(queue);
    return factoryBean;
}
```

我们现在可以使用远程服务，就像它被声明为本地 bean 一样:

```java
CabBookingService service = context.getBean(CabBookingService.class);
out.println(service.bookRide("13 Seagate Blvd, Key Largo, FL 33037"));
```

### 5.2。运行示例

同样对于客户端应用程序，我们必须正确地选择应用程序中的值`.properties`文件。在常见的设置中，这些将与服务器端使用的完全匹配。

这应该足以演示通过`Apache AMQ`的远程调用。所以，让我们首先启动`ApacheMQ`，然后是服务器应用程序，最后是将调用远程服务的客户机应用程序。

## 6。结论

在这个快速教程中，我们看到了如何使用`Spring Remoting`在`JMS`系统之上提供`RPC`作为`AMQ`。

Spring Remoting 继续展示了不管底层通道如何，快速建立异步调用是多么容易。

像往常一样，你会在 GitHub 上找到这些资源[。](https://web.archive.org/web/20220625222544/https://github.com/eugenp/tutorials/tree/master/spring-remoting-modules/remoting-jms)