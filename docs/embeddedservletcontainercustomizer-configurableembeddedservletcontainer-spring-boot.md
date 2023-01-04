# Spring Boot 集装箱配置 2

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/embeddedservletcontainercustomizer-configurableembeddedservletcontainer-spring-boot>

## 1。概述

在这个快速教程中，我们将看看如何替换 Spring Boot 2 中的`EmbeddedServletContainerCustomizer`和`ConfigurableEmbeddedServletContainer`。

这些类是 Spring Boot 早期版本的一部分，但是从 Spring Boot 2 开始已经被删除了。当然，**的功能仍然可以通过接口`WebServerFactoryCustomizer`和类** `**ConfigurableServletWebServerFactory**.`获得

让我们来看看如何使用这些。

## 2。在 Spring Boot 2 之前

首先，让我们看一下使用旧类和接口的配置，我们需要替换它:

```java
@Component
public class CustomContainer implements EmbeddedServletContainerCustomizer {

    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.setPort(8080);
        container.setContextPath("");
     }
}
```

这里，我们定制 servlet 容器的端口和上下文路径。

实现这一点的另一种可能性是对 Tomcat 这样的容器类型使用更具体的子类`ConfigurableEmbeddedServletContainer,`:

```java
@Component
public class CustomContainer implements EmbeddedServletContainerCustomizer {

    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        if (container instanceof TomcatEmbeddedServletContainerFactory) {
            TomcatEmbeddedServletContainerFactory tomcatContainer = 
              (TomcatEmbeddedServletContainerFactory) container;
            tomcatContainer.setPort(8080);
            tomcatContainer.setContextPath("");
        }
    }
}
```

## 3。升级到 Spring Boot 2

在《Spring Boot 2》中，**的`EmbeddedServletContainerCustomizer`界面被替换为`WebServerFactoryCustomizer,`，而`ConfigurableEmbeddedServletContainer`职业被替换为`ConfigurableServletWebServerFactory.`**

让我们为 Spring Boot 新协议项目改写前面的例子:

```java
public class CustomContainer implements 
  WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    public void customize(ConfigurableServletWebServerFactory factory) {
        factory.setPort(8080);
        factory.setContextPath("");
     }
}
```

第二个例子现在将使用一个`TomcatServletWebServerFactory:`

```java
@Component
public class CustomContainer implements 
  WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        factory.setContextPath("");
        factory.setPort(8080);
    }
}
```

类似地，我们将`JettyServletWebServerFactory`和`UndertowServletWebServerFactory`作为被移除的`JettyEmbeddedServletContainerFactory`和`UndertowEmbeddedServletContainerFactory.`的等价物

## 4.结论

这篇简短的文章展示了如何修复我们在将 Spring Boot 应用程序升级到 2.x 版本时可能会遇到的问题

我们的 [GitHub 库](https://web.archive.org/web/20220628105250/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization-2)中有一个 Spring Boot 2 项目的例子。