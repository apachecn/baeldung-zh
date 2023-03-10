# 在 Spring Boot 使用 application.yml 与 application.properties

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-yaml-vs-properties>

## 1.概观

在 Spring Boot，通常的做法是使用一个[外部配置来定义我们的属性](/web/20220630021618/https://www.baeldung.com/properties-with-spring)。这允许我们在不同的环境中使用相同的应用程序代码。

我们可以使用属性文件、YAML 文件、环境变量和命令行参数。

在这个简短的教程中，我们将探索属性和 YAML 文件之间的主要区别。

## 2.属性配置

默认情况下，Spring Boot 可以访问在`application.properties`文件中设置的配置，该文件使用键值格式:

```java
spring.datasource.url=jdbc:h2:dev
spring.datasource.username=SA
spring.datasource.password=password
```

这里每一行都是一个单独的配置，所以我们需要通过使用相同的关键字前缀来表达分层数据。在这个例子中，每个键都属于`spring.datasource`。

### 2.1.属性中的占位符

在我们的值中，我们可以使用带有`${}`语法的占位符来引用其他键、系统属性或环境变量的内容:

```java
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

### 2.2.表结构

如果我们有不同值的同类属性，我们可以用数组索引来表示列表结构:

```java
application.servers[0].ip=127.0.0.1
application.servers[0].path=/path1
application.servers[1].ip=127.0.0.2
application.servers[1].path=/path2
application.servers[2].ip=127.0.0.3
application.servers[2].path=/path3
```

### 2.3.多个配置文件

从版本 2.4.0 开始，Spring Boot 支持创建多文档属性文件。简单地说，我们可以将一个物理文件分割成多个逻辑文档。

这允许我们为我们需要声明的每个概要文件定义一个文档，所有的都在同一个文件中:

```java
logging.file.name=myapplication.log
bael.property=defaultValue
#---
spring.config.activate.on-profile=dev
spring.datasource.password=password
spring.datasource.url=jdbc:h2:dev
spring.datasource.username=SA
bael.property=devValue
#---
spring.config.activate.on-profile=prod
spring.datasource.password=password
spring.datasource.url=jdbc:h2:prod
spring.datasource.username=prodUser
bael.property=prodValue
```

注意，我们使用“# -”符号来表示我们想要拆分文档的位置。

在这个例子中，我们有两个标记了不同的`profiles`的`spring`部分。此外，我们可以在根级别拥有一组公共属性——在这种情况下,`logging.file.name`属性在所有概要文件中都是相同的。

### 2.4.跨多个文件的配置文件

作为在同一个文件中拥有不同概要文件的替代方法，我们可以在不同的文件中存储多个概要文件。在版本 2.4.0 之前，这是唯一可用于`properties`文件的方法。

我们通过将概要文件的名称放入文件名中来实现这一点——例如，`application-dev.yml` 或`application-dev.properties`。

## 3.YAML 构型

### 3.1.YAML 格式

除了 Java 属性文件，我们还可以在我们的 Spring Boot 应用程序中使用基于 [YAML](/web/20220630021618/https://www.baeldung.com/spring-yaml) 的配置文件。 **YAML 是一种用于指定分层配置数据的便捷格式。**

现在，让我们从我们的属性文件中取出相同的示例，并将其转换为 YAML:

```java
spring:
    datasource:
        password: password
        url: jdbc:h2:dev
        username: SA
```

因为它不包含重复的前缀，所以比它的属性文件更具可读性。

### 3.2.表结构

YAML 有一个更简洁的格式来表示列表:

```java
application:
    servers:
    -   ip: '127.0.0.1'
        path: '/path1'
    -   ip: '127.0.0.2'
        path: '/path2'
    -   ip: '127.0.0.3'
        path: '/path3'
```

### 3.3.多个配置文件

与属性文件不同，YAML 通过设计支持多文档文件，这样，无论我们使用哪个版本的 Spring Boot，我们都可以在同一个文件中存储多个概要文件。

然而，在这种情况下，规范指示我们必须使用三个破折号来表示新文档的开始:

```java
logging:
  file:
    name: myapplication.log
---
spring:
  config:
    activate:
      on-profile: staging
  datasource:
    password: 'password'
    url: jdbc:h2:staging
    username: SA
bael:
  property: stagingValue
```

注意:我们通常不希望在项目中同时包含标准的`application.properties`和`application.yml `文件，因为这可能会导致意想不到的结果。

例如，如果我们将上面显示的属性(在一个`application.yml`文件中)与第 2.3 节描述的属性结合起来。，那么`bael.property`将被赋予`defaultValue `，而不是特定于配置文件的值。这仅仅是因为`application.properties `稍后被加载，覆盖了之前分配的值。

## 4.Spring Boot 用法

现在我们已经定义了我们的配置，让我们看看如何访问它们。

### 4.1.`Value`注释

我们可以使用`@Value`注释注入我们的属性值:

```java
@Value("${key.something}")
private String injectedProperty;
```

这里，属性`key.something`通过字段注入被注入到我们的一个对象中。

### 4.2.`Environment`抽象

我们还可以使用`Environment` API 获得属性的值:

```java
@Autowired
private Environment env;

public String getSomeKey(){
    return env.getProperty("key.something");
} 
```

### 4.3.`ConfigurationProperties`注释

最后，我们还可以使用`@ConfigurationProperties`注释将我们的属性绑定到类型安全的结构化对象:

```java
@ConfigurationProperties(prefix = "mail")
public class ConfigProperties {
    String name;
    String description;
...
```

## 5.结论

在本文中，我们已经看到了`properties`和`yml` Spring Boot 配置文件之间的一些差异。我们还看到了它们的值如何引用其他属性。最后，我们看了如何将这些值注入我们的运行时。

和往常一样，GitHub 上的所有代码示例[都是可用的。](https://web.archive.org/web/20220630021618/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-3)