# 使用 RMI 的 Spring Remoting

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-remoting-rmi>

## 1。概述

Java `Remote Method Invocation`允许调用驻留在不同的`Java Virtual Machine`中的对象。这是一项成熟的技术，但使用起来有点麻烦，我们可以在专门针对该主题的[官方甲骨文踪迹](https://web.archive.org/web/20221126231640/https://docs.oracle.com/javase/tutorial/rmi/index.html)中看到。

在这篇简短的文章中，我们将探索`Spring Remoting`如何允许以一种更简单、更干净的方式利用`RMI`。

本文也完成了对`Spring Remoting`的概述。您可以在以前的文章中找到关于其他支持技术的详细信息: [HTTP Invokers](/web/20221126231640/https://www.baeldung.com/spring-remoting-http-invoker) 、 [JMS](/web/20221126231640/https://www.baeldung.com/spring-remoting-jms) 、 [AMQP](/web/20221126231640/https://www.baeldung.com/spring-remoting-amqp) 、 [Hessian 和 Burlap](/web/20221126231640/https://www.baeldung.com/spring-remoting-hessian-burlap) 。

## 2。Maven 依赖关系

正如我们在以前的文章中所做的，我们将设置几个`Spring Boot`应用程序:一个公开远程可调用对象的服务器和一个调用公开服务的客户机。

我们需要的一切都在`spring-context` jar 中——所以我们可以使用我们喜欢的任何`Spring Boot`助手来引入它——因为我们的主要目标只是让主库可用。

现在让我们继续进行通常的`spring-boot-starter-web`——记住移除`Tomcat`依赖性以排除嵌入式 web 服务:

```java
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
```

## 3。服务器应用程序

我们将开始声明一个接口，该接口定义了一个预订出租车的服务，该服务最终将向客户端公开:

```java
public interface CabBookingService {
    Booking bookRide(String pickUpLocation) throws BookingException;
}
```

然后我们将定义一个实现接口的 bean。这是将在服务器上实际执行业务逻辑的 bean:

```java
@Bean 
CabBookingService bookingService() {
    return new CabBookingServiceImpl();
}
```

让我们继续声明使服务对客户端可用的`Exporter`。在这种情况下，我们将使用`RmiServiceExporter`:

```java
@Bean 
RmiServiceExporter exporter(CabBookingService implementation) {
    Class<CabBookingService> serviceInterface = CabBookingService.class;
    RmiServiceExporter exporter = new RmiServiceExporter();
    exporter.setServiceInterface(serviceInterface);
    exporter.setService(implementation);
    exporter.setServiceName(serviceInterface.getSimpleName());
    exporter.setRegistryPort(1099); 
    return exporter;
}
```

通过`setServiceInterface()`,我们提供了一个接口的引用，这个接口将被远程调用。

我们还应该用`setService()`提供对实际执行该方法的对象的引用。如果我们不想使用默认端口 1099，那么我们可以提供服务器运行机器上可用的端口`RMI registry`。

我们还应该设置一个服务名，它允许在`RMI`注册表中识别公开的服务。

使用给定的配置，客户端将能够通过以下 URL 联系到`CabBookingService`:`rmi://HOST:1199/CabBookingService`。

让我们最后启动服务器。我们甚至不需要自己启动 RMI 注册中心，因为如果这样的注册中心不可用的话,`Spring`会自动为我们启动。

## 4。客户端应用程序

现在让我们编写客户端应用程序。

我们开始声明`RmiProxyFactoryBean`,它将创建一个 bean，该 bean 具有由运行在服务器端的服务公开的相同接口，并且它将透明地将接收到的调用路由到服务器:

```java
@Bean 
RmiProxyFactoryBean service() {
    RmiProxyFactoryBean rmiProxyFactory = new RmiProxyFactoryBean();
    rmiProxyFactory.setServiceUrl("rmi://localhost:1099/CabBookingService");
    rmiProxyFactory.setServiceInterface(CabBookingService.class);
    return rmiProxyFactory;
}
```

接下来，让我们编写一个简单的代码来启动客户端应用程序，并使用上一步中定义的代理:

```java
public static void main(String[] args) throws BookingException {
    CabBookingService service = SpringApplication
      .run(RmiClient.class, args).getBean(CabBookingService.class);
    Booking bookingOutcome = service
      .bookRide("13 Seagate Blvd, Key Largo, FL 33037");
    System.out.println(bookingOutcome);
}
```

现在只需启动客户机来验证它是否调用了服务器公开的服务。

## 5。结论

在本教程中，我们看到了如何使用`Spring Remoting`来简化`RMI`的使用，否则将需要一系列繁琐的任务，其中包括使用大量使用检查异常的接口来创建注册中心和定义服务。

像往常一样，你可以在 GitHub 上找到这些资源。