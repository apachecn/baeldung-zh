# 用粗麻布和粗麻布进行春季遥控

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-remoting-hessian-burlap>

## 1。概述

在[上一篇题为“HTTP Invokers 的 Spring Remoting 简介”](/web/20221129010015/https://www.baeldung.com/spring-remoting-http-invoker)的文章中，我们看到了通过`Spring Remoting`建立一个利用远程方法调用(RMI)的客户机/服务器应用程序是多么容易。

在本文中，我们将向**展示`Spring Remoting`如何使用`Hessian`和`Burlap`** 来支持 RMI 的实现。

## 2。Maven 依赖关系

`Hessian`和`Burlap` 都是由下面的库提供的，您需要将它们显式地包含在您的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.38</version>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20221129010015/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.caucho%22%20AND%20a%3A%22hessian%22) 上找到最新版本。

## 3。黑森

`Hessian`是来自`Caucho`的轻量级二进制协议，`Resin`应用服务器的制造商。`Hessian`存在多种平台和语言的实现，包括 Java。

在下面的小节中，我们将修改上一篇文章中的“出租车预订”示例，使客户机和服务器使用`Hessian`而不是基于`Spring Remote HTTP`的协议进行通信。

### 3.1。公开服务

让我们通过配置类型为`HessianServiceExporter`的`RemoteExporter`来公开服务，替换之前使用的`HttpInvokerServiceExporter`:

```java
@Bean(name = "/booking") 
RemoteExporter bookingService() {
    HessianServiceExporter exporter = new HessianServiceExporter();
    exporter.setService(new CabBookingServiceImpl());
    exporter.setServiceInterface( CabBookingService.class );
    return exporter;
}
```

我们现在可以启动服务器，并在准备客户机时保持它的活动状态。

### 3.2。客户端应用程序

让我们实现客户端。同样，修改非常简单——我们需要用一个`HessianProxyFactoryBean`替换`HttpInvokerProxyFactoryBean`:

```java
@Configuration
public class HessianClient {

    @Bean
    public HessianProxyFactoryBean hessianInvoker() {
        HessianProxyFactoryBean invoker = new HessianProxyFactoryBean();
        invoker.setServiceUrl("http://localhost:8080/booking");
        invoker.setServiceInterface(CabBookingService.class);
        return invoker;
    }

    public static void main(String[] args) throws BookingException {
        CabBookingService service
          = SpringApplication.run(HessianClient.class, args)
              .getBean(CabBookingService.class);
        out.println(
          service.bookRide("13 Seagate Blvd, Key Largo, FL 33037"));
    }
}
```

现在让我们使用`Hessian`运行客户端，使其连接到服务器。

## 4。粗麻布

`Burlap`是来自`Caucho`的另一个轻量级协议，基于`XML`。`Caucho`很久以前就停止了对它的维护，因此，尽管它已经存在，但在最新的 Spring 版本中它的支持已经被否决了。

因此，只有当您的应用程序已经被分发并且不容易迁移到另一个`Spring Remoting`实现时，您才应该合理地继续使用`Burlap`。

### 4.1。公开服务

我们可以像使用`Hessian`一样使用`Burlap`——我们只需要选择合适的实现:

```java
@Bean(name = "/booking") 
RemoteExporter burlapService() {
    BurlapServiceExporter exporter = new BurlapServiceExporter();
    exporter.setService(new CabBookingServiceImpl());
    exporter.setServiceInterface( CabBookingService.class );
    return exporter;
}
```

正如你所看到的，我们只是将导出器的类型从`HessianServiceExporter`改为`BurlapServiceExporter.`，所有的设置代码可以保持不变。

同样，让我们启动服务器，让它在我们处理客户端时保持运行。

### 4.2。客户端实现

我们同样可以在客户端将`Hessian`换成`Burlap`，将`HessianProxyFactoryBean`换成`BurlapProxyFactoryBean`:

```java
@Bean
public BurlapProxyFactoryBean burlapInvoker() {
    BurlapProxyFactoryBean invoker = new BurlapProxyFactoryBean();
    invoker.setServiceUrl("http://localhost:8080/booking");
    invoker.setServiceInterface(CabBookingService.class);
    return invoker;
}
```

我们现在可以运行客户机，看看它如何使用`Burlap`成功连接到服务器应用程序。

## 5。结论

通过这些简单的例子，我们展示了如何使用`Spring Remoting`在不同的技术中选择实现远程方法调用，以及如何开发一个完全不知道用于表示远程方法调用的协议的技术细节的应用程序。

像往常一样，你会在 GitHub 上找到源代码[，有客户端用于`Hessian`和`Burlap`以及`JUnit`测试`CabBookingServiceTest.java`，它们将负责运行服务器和客户端。](https://web.archive.org/web/20221129010015/https://github.com/eugenp/tutorials/tree/master/spring-remoting-modules/remoting-hessian-burlap)