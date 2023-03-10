# 使用 HTTP 调用程序的 Spring Remoting 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-remoting-http-invoker>

## 1。概述

在某些情况下，我们需要将一个系统分解成几个进程，每个进程负责应用程序的不同方面。在这些场景中，一个流程需要从另一个流程同步获取数据的情况并不少见。

Spring 框架提供了一系列被称为`Spring Remoting`的工具，允许我们调用远程服务，就好像它们至少在某种程度上是本地可用的一样。

在本文中，我们将建立一个基于 Spring 的 `HTTP invoker`的应用程序，它利用本地 Java 序列化和 HTTP 来提供客户端和服务器应用程序之间的远程方法调用。

## 2。服务定义

让我们假设我们必须实现一个允许用户在出租车上预定行程的系统。

让我们假设我们选择构建**两个不同的应用程序**来实现这个目标:

*   预订引擎应用程序，用于检查是否可以满足出租车请求，以及
*   一个前端 web 应用程序，允许客户预订他们的乘坐，确保出租车的可用性已得到确认

### 2.1。服务接口

当我们使用`Spring Remoting`和`HTTP invoker,`时，我们必须通过一个接口定义我们的远程可调用服务，让 Spring 在客户端和服务器端创建代理，封装远程调用的技术细节。让我们从允许我们预订出租车的服务接口开始:

```java
public interface CabBookingService {
    Booking bookRide(String pickUpLocation) throws BookingException;
}
```

当服务能够分配 cab 时，它返回一个带有预订代码的`Booking`对象。`Booking`必须是可序列化的，因为 Spring 的 HTTP invoker 必须将其实例从服务器传输到客户端:

```java
public class Booking implements Serializable {
    private String bookingCode;

    @Override public String toString() {
        return format("Ride confirmed: code '%s'.", bookingCode);
    }

    // standard getters/setters and a constructor
}
```

如果服务无法预订出租车，就会抛出一个`BookingException`。在这种情况下，没有必要将该类标记为`Serializable`，因为`Exception`已经实现了它:

```java
public class BookingException extends Exception {
    public BookingException(String message) {
        super(message);
    }
}
```

### 2.2。打包服务

服务接口以及所有用作参数的自定义类、返回类型和异常必须在客户端和服务器的类路径中都可用。最有效的方法之一是将它们打包在一个`.jar`文件中，这个文件可以作为一个依赖项包含在服务器和客户机的`pom.xml`中。

因此，让我们将所有代码放在一个专用的 Maven 模块中，称为“API”；对于这个例子，我们将使用下面的 Maven 坐标:

```java
<groupId>com.baeldung</groupId>
<artifactId>api</artifactId>
<version>1.0-SNAPSHOT</version>
```

## 3。服务器应用程序

让我们使用 Spring Boot 构建预订引擎应用程序来公开服务。

### 3.1。Maven 依赖关系

首先，您需要确保您的项目使用 Spring Boot:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version>
</parent>
```

你可以在这里找到最新的 Spring Boot 版本。然后我们需要 Web starter 模块:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

我们需要在上一步中组装的服务定义模块:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### 3.2。服务实施

我们首先定义一个实现服务接口的类:

```java
public class CabBookingServiceImpl implements CabBookingService {

    @Override public Booking bookPickUp(String pickUpLocation) throws BookingException {
        if (random() < 0.3) throw new BookingException("Cab unavailable");
        return new Booking(randomUUID().toString());
    }
}
```

让我们假设这是一个可能的实现。使用带有随机值的测试，我们将能够再现两种成功的场景——当找到可用的出租车并返回预订代码时——以及失败的场景——当抛出 BookingException 以指示没有任何可用的出租车时。

### 3.3。公开服务

然后我们需要在上下文中定义一个带有类型为`HttpInvokerServiceExporter` 的 bean 的应用程序。它将负责在 web 应用程序中公开一个 HTTP 入口点，供客户端调用:

```java
@Configuration
@ComponentScan
@EnableAutoConfiguration
public class Server {

    @Bean(name = "/booking") HttpInvokerServiceExporter accountService() {
        HttpInvokerServiceExporter exporter = new HttpInvokerServiceExporter();
        exporter.setService( new CabBookingServiceImpl() );
        exporter.setServiceInterface( CabBookingService.class );
        return exporter;
    }

    public static void main(String[] args) {
        SpringApplication.run(Server.class, args);
    }
}
```

值得注意的是，Spring 的 `HTTP invoker` 使用`HttpInvokerServiceExporter` bean 的名称作为 HTTP 端点 URL 的相对路径。

现在，我们可以启动服务器应用程序，并在设置客户端应用程序时保持它的运行。

## 4。客户端应用程序

现在让我们编写客户端应用程序。

### 4.1。Maven 依赖关系

我们将使用与服务器端相同的服务定义和 Spring Boot 版本。我们仍然需要 web starter 依赖项，但是因为我们不需要自动启动嵌入式容器，所以我们可以从依赖项中排除 Tomcat starter:

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

### 4.2。客户端实现

让我们实现客户端:

```java
@Configuration
public class Client {

    @Bean
    public HttpInvokerProxyFactoryBean invoker() {
        HttpInvokerProxyFactoryBean invoker = new HttpInvokerProxyFactoryBean();
        invoker.setServiceUrl("http://localhost:8080/booking");
        invoker.setServiceInterface(CabBookingService.class);
        return invoker;
    }

    public static void main(String[] args) throws BookingException {
        CabBookingService service = SpringApplication
          .run(Client.class, args)
          .getBean(CabBookingService.class);
        out.println(service.bookRide("13 Seagate Blvd, Key Largo, FL 33037"));
    }
}
```

带注释的调用程序`()`方法创建了一个`HttpInvokerProxyFactoryBean`的实例。我们需要通过`setServiceUrl()`方法提供远程服务器响应的 URL。

类似于我们为服务器所做的，我们也应该通过`setServiceInterface()`方法提供我们想要远程调用的服务的接口。

`HttpInvokerProxyFactoryBean`实现弹簧的`FactoryBean`。一个`FactoryBean`被定义为一个 bean，但是 Spring IoC 容器将注入它创建的对象，而不是工厂本身。你可以在我们的[工厂豆文章](/web/20220812061910/https://www.baeldung.com/spring-factorybean)中找到更多关于`FactoryBean`的细节。

`main()`方法引导独立应用程序，并从上下文中获取`CabBookingService`的实例。实际上，这个对象只是由`HttpInvokerProxyFactoryBean`创建的一个代理，负责处理远程调用执行中涉及的所有技术问题。多亏了它，我们现在可以轻松地使用代理，就像服务实现在本地可用时一样。

让我们多次运行应用程序来执行几个远程调用，以验证客户机在 cab 可用和不可用时的行为。

## 5。买者自负

当我们使用允许远程调用的技术时，有一些陷阱我们应该非常清楚。

### 5.1。当心网络相关异常

当我们和一个不可靠的资源如网络一起工作时，我们应该总是预料不到的事情。

让我们假设客户端在无法联系到服务器时调用服务器——要么是因为网络问题，要么是因为服务器关闭——那么 Spring Remoting 将引发一个`RemoteAccessException` ,也就是一个`RuntimeException.`

编译器不会强迫我们在 try-catch 块中包含调用，但是我们应该总是考虑这样做，以正确地管理网络问题。

### 5.2。对象通过值传递，而不是通过引用

`Spring Remoting HTTP`整理方法参数和返回值以在网络上传输它们。这意味着服务器根据所提供的参数的副本进行操作，而客户端根据服务器创建的结果的副本进行操作。

所以我们不能期望，例如，在结果对象上调用一个方法会改变服务器端相同对象的状态，因为在客户机和服务器之间没有任何共享对象。

### 5.3。小心细粒度的接口

跨网络边界调用一个方法比在同一进程中对一个对象调用它要慢得多。

出于这个原因，定义应该用粗粒度接口远程调用的服务通常是一个好的实践，这些接口能够完成需要较少交互的业务事务，即使以更麻烦的接口为代价。

## 6。结论

通过这个例子，我们看到了 Spring Remoting 调用远程进程是多么容易。

该解决方案比其他广泛使用的机制(如 REST 或 web 服务)稍微开放一些，但是在所有组件都是用 Spring 开发的场景中，它可以代表一个可行且更快的替代方案。

像往常一样，你会在 GitHub 上找到这些资源[。](https://web.archive.org/web/20220812061910/https://github.com/eugenp/tutorials/tree/master/spring-remoting-modules/remoting-http)