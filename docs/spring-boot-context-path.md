# Spring Boot 更改上下文路径

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-context-path>

## 1。概述

**默认情况下，** **在根上下文路径(“/”)**上提供内容。

虽然习惯比配置更好通常是个好主意，但是有些情况下我们确实希望有一个自定义的路径。

在这个快速教程中，我们将介绍配置它的不同方法。

## 2。设置属性

就像许多其他配置选项一样，可以通过设置属性`server.servlet.context-path`来更改 Spring Boot 中的上下文路径。

注意，这适用于 Spring Boot 2.x。对于 Boot 1.x，该属性为`server.context-path`。

设置该属性有多种方法，让我们一个一个来看。

### 2.1。使用`application.properties` / `yml`

更改上下文路径最直接的方法是在`application.properties` / `yml`文件中设置属性:

```java
server.servlet.context-path=/baeldung
```

除了将属性文件放在`src/main/resources`中，我们还可以将它保存在当前的工作目录中(在类路径之外)。

### 2.2。Java 系统属性

我们还可以在上下文初始化之前将上下文路径设置为 Java 系统属性:

```java
public static void main(String[] args) {
    System.setProperty("server.servlet.context-path", "/baeldung");
    SpringApplication.run(Application.class, args);
}
```

### 2.3。操作系统环境变量

Spring Boot 也可以依赖操作系统环境变量。在基于 Unix 的系统上，我们可以编写:

```java
$ export SERVER_SERVLET_CONTEXT_PATH=/baeldung
```

在 Windows 上，设置环境变量的命令是:

```java
> set SERVER_SERVLET_CONTEXT_PATH=/baeldung
```

上面的**环境变量是针对 Spring Boot 2 . x . x .**T3 的，如果我们有 1.x.x ，这个变量就是`**SERVER_CONTEXT_PATH**`。

### 2.4。命令行参数

我们也可以通过命令行参数动态设置属性:

```java
$ java -jar app.jar --server.servlet.context-path=/baeldung
```

## 3。使用 Java 配置

现在让我们通过用配置 bean 填充 bean 工厂来设置上下文路径。

对于 Spring Boot 2，我们可以使用`WebServerFactoryCustomizer`:

```java
@Bean
public WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>
  webServerFactoryCustomizer() {
    return factory -> factory.setContextPath("/baeldung");
}
```

使用 Spring Boot 1，我们可以创建一个`EmbeddedServletContainerCustomizer`的实例:

```java
@Bean
public EmbeddedServletContainerCustomizer
  embeddedServletContainerCustomizer() {
    return container -> container.setContextPath("/baeldung");
}
```

## 4。配置的优先顺序

有了这么多的选项，我们最终可能会对同一个属性有多个配置。

下面是按降序排列的**，**，Spring Boot 用它来选择有效配置:

1.  Java 配置
2.  命令行参数
3.  Java 系统属性
4.  操作系统环境变量
5.  `application.properties`在当前目录
6.  `application.properties`在类路径中(`src/main/resources`或打包的 jar 文件)

## 5。结论

在这篇简短的文章中，我们介绍了在 Spring Boot 应用程序中设置上下文路径或任何其他配置属性的不同方法。