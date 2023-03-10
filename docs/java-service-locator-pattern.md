# 服务定位器模式和 Java 实现

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-service-locator-pattern>

## 1。简介

在本教程中，我们将学习 Java 中的**服务定位器设计模式。**

我们将描述这个概念，实现一个例子，并强调其使用的利与弊。

## 2。理解模式

服务定位器模式的目的是按需返回服务实例。这有助于将服务消费者从具体的类中分离出来。

实施将由以下组件组成:

*   客户端——客户端对象是服务消费者。它负责从服务定位器调用请求

*   服务定位器——是从缓存返回服务的通信入口点

*   缓存——存储服务引用以便以后重用的对象

*   初始化器——在缓存中创建和注册对服务的引用

*   服务——服务组件代表原始服务或它们的实现

定位器查找原始服务对象，并根据需要返回。

## 3。实施

现在，让我们从实际出发，通过一个例子来看看这些概念。

首先，我们将创建一个用不同方式发送消息的`MessagingService`接口:

```java
public interface MessagingService {

    String getMessageBody();
    String getServiceName();
}
```

接下来，我们将定义上述接口的两个实现，它们通过电子邮件和 SMS 发送消息:

```java
public class EmailService implements MessagingService {

    public String getMessageBody() {
        return "email message";
    }

    public String getServiceName() {
        return "EmailService";
    }
}
```

`SMSService`类的定义类似于`EmailService`类。

在定义了这两个服务之后，我们必须定义初始化它们的逻辑:

```java
public class InitialContext {
    public Object lookup(String serviceName) {
        if (serviceName.equalsIgnoreCase("EmailService")) {
            return new EmailService();
        } else if (serviceName.equalsIgnoreCase("SMSService")) {
            return new SMSService();
        }
        return null;
    }
}
```

在组装服务定位器对象之前，我们需要的最后一个组件是缓存。

在我们的例子中，这是一个具有`List`属性的简单类:

```java
public class Cache {
    private List<MessagingService> services = new ArrayList<>();

    public MessagingService getService(String serviceName) {
        // retrieve from the list
    }

    public void addService(MessagingService newService) {
        // add to the list
    }
} 
```

最后，我们可以实现我们的服务定位器类:

```java
public class ServiceLocator {

    private static Cache cache = new Cache();

    public static MessagingService getService(String serviceName) {

        MessagingService service = cache.getService(serviceName);

        if (service != null) {
            return service;
        }

        InitialContext context = new InitialContext();
        MessagingService service1 = (MessagingService) context
          .lookup(serviceName);
        cache.addService(service1);
        return service1;
    }
}
```

这里的逻辑相当简单。

该类保存了一个`Cache.`的实例，然后在`getService()`方法中，它将首先检查服务实例的缓存。

然后，如果那是`null,`，它将调用初始化逻辑，并将新对象添加到缓存中。

## 4。测试

现在让我们看看如何获得实例:

```java
MessagingService service 
  = ServiceLocator.getService("EmailService");
String email = service.getMessageBody();

MessagingService smsService 
  = ServiceLocator.getService("SMSService");
String sms = smsService.getMessageBody();

MessagingService emailService 
  = ServiceLocator.getService("EmailService");
String newEmail = emailService.getMessageBody();
```

**我们第一次从`ServiceLocator`获取`EmailService`时，一个新的实例被创建并返回**。然后，在下一次调用它之后,`EmailService`将从缓存中返回。

## 5。服务定位器 vs 依赖注入

乍一看，服务定位器模式可能看起来类似于另一个众所周知的模式——即依赖注入。

首先，需要注意的是**依赖注入和服务定位器模式都是控制反转概念**的实现。

在继续之前，在这篇[文章](/web/20220627092358/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)中学习更多关于依赖注入的知识。

**这里的关键区别是客户端对象仍然创建它的依赖关系**。它只是为此使用定位器，这意味着它需要一个对定位器对象的引用。

相比之下，当使用依赖注入时，类被赋予依赖。注入器只在启动时调用一次，将依赖项注入到类中。

最后，让我们考虑一些避免使用服务定位器模式的原因。

反对它的一个理由是它使单元测试变得困难。通过依赖注入，我们可以将依赖类的模拟对象传递给被测试的实例。另一方面，这是服务定位器模式的一个瓶颈。

另一个问题是，使用基于这种模式的 API 更加棘手。这样做的原因是依赖关系隐藏在类中，它们只在运行时被验证。

尽管如此，服务定位器模式易于编码和理解，是小型应用程序的最佳选择。

## 6。结论

本指南展示了如何以及为什么使用服务定位器设计模式。它讨论了服务定位器设计模式和依赖注入概念之间的主要区别。

一般来说，如何设计应用程序中的类取决于开发人员。

服务定位器模式是分离代码的一种简单模式。然而，如果在多个应用程序中使用这些类，依赖注入是一个正确的选择。

像往常一样，完整的代码可以在 [Github 项目](https://web.archive.org/web/20220627092358/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-architectural)中获得。