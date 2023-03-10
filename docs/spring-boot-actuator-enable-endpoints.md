# 如何启用 Spring Boot 执行器中的所有端点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-actuator-enable-endpoints>

## 1.概观

在本教程中，我们将学习如何启用 [Spring Boot 致动器中的所有端点。](/web/20220628065013/https://www.baeldung.com/spring-boot-actuators)我们将从必要的 Maven 依赖项开始。从那里，我们将看看如何通过我们的属性文件来控制我们的端点。最后，我们将概述如何保护我们的终端。

就致动器端点的配置而言，Spring Boot 1.x 和 Spring Boot 2.x 之间有几处[变化。这些问题出现时，我们会记录下来。](/web/20220628065013/https://www.baeldung.com/spring-boot-actuators)

## 2.设置

为了使用致动器，我们需要在我们的 Maven 配置中包括`[spring-boot-starter-actuator](https://web.archive.org/web/20220628065013/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-actuator)`:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.5.1</version>
</dependency>
```

此外，从 Spring Boot 2.0 开始，**如果我们想通过 HTTP** 公开我们的端点，我们需要包含 [web starter](https://web.archive.org/web/20220628065013/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-web) :

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.1</version>
</dependency>
```

## 3.启用和公开端点

从 Spring Boot 2 开始，**我们必须启用并公开我们的端点**。默认情况下，除了`/shutdown`之外的所有端点都被启用，只有`/health` 和`/info`被暴露。即使我们为应用程序配置了不同的根上下文，所有的端点都在`/actuator`处找到。

这意味着一旦我们在 Maven 配置中添加了合适的启动器，我们就可以在`http://localhost:8080/actuator/health `和`http://localhost:8080/actuator/info`访问`/health`和`/info`端点。

让我们转到`http://localhost:8080/actuator`并查看可用端点列表，因为致动器端点已启用 HATEOS。我们应该看到`/health`和`/info`。

```java
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},
"health":{"href":"http://localhost:8080/actuator/health","templated":false},
"info":{"href":"http://localhost:8080/actuator/info","templated":false}}}
```

### 3.1.公开所有端点

现在，让我们通过修改我们的`application.properties`文件来公开除了`/shutdown`之外的所有端点:

```java
management.endpoints.web.exposure.include=*
```

一旦我们重启了服务器并再次访问了`/actuator`端点，我们应该会看到除了`/shutdown:`之外的其他可用端点

```java
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},
"beans":{"href":"http://localhost:8080/actuator/beans","templated":false},
"caches":{"href":"http://localhost:8080/actuator/caches","templated":false},
"health":{"href":"http://localhost:8080/actuator/health","templated":false},
"info":{"href":"http://localhost:8080/actuator/info","templated":false},
"conditions":{"href":"http://localhost:8080/actuator/conditions","templated":false},
"configprops":{"href":"http://localhost:8080/actuator/configprops","templated":false},
"env":{"href":"http://localhost:8080/actuator/env","templated":false},
"loggers":{"href":"http://localhost:8080/actuator/loggers","templated":false},
"heapdump":{"href":"http://localhost:8080/actuator/heapdump","templated":false},
"threaddump":{"href":"http://localhost:8080/actuator/threaddump","templated":false},
"metrics":{"href":"http://localhost:8080/actuator/metrics","templated":false},
"scheduledtasks":{"href":"http://localhost:8080/actuator/scheduledtasks","templated":false},
"mappings":{"href":"http://localhost:8080/actuator/mappings","templated":false}}}
```

### 3.2.公开特定端点

一些端点可以公开敏感数据，所以让我们学习如何更细粒度地公开哪些端点。

`management.endpoints.web.exposure.include`属性也可以接受逗号分隔的端点列表。所以，我们只曝光`/beans`和`/loggers`:

```java
management.endpoints.web.exposure.include=beans, loggers
```

除了用属性包含某些端点之外，我们还可以排除端点。让我们公开除了`/threaddump`之外的所有端点:

```java
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=threaddump
```

`include`和`exclude`属性都接受端点列表。**`exclude`房产优先于`include`** 。

### 3.3.启用特定端点

接下来，让我们了解如何更精确地了解我们启用了哪些终端。

首先，我们需要关闭启用所有端点的默认设置:

```java
management.endpoints.enabled-by-default=false
```

接下来，让我们只启用和公开`/health`端点:

```java
management.endpoint.health.enabled=true
management.endpoints.web.exposure.include=health
```

使用这种配置，我们只能访问`/health`端点。

### 3.4.启用关机

由于其敏感性，**`/shutdown`端点在默认情况下被禁用**。

让我们现在通过在我们的`application.properties`文件中添加一行来启用它:

```java
management.endpoint.shutdown.enabled=true
```

现在，当我们查询`/actuator`端点时，我们应该看到它被列出。**`/shutdown`端点只接受`POST`请求**，所以让我们优雅地关闭我们的应用程序:

```java
curl -X POST http://localhost:8080/actuator/shutdown
```

## 4.保护端点

在现实世界的应用程序中，我们最有可能在我们的应用程序上有安全性。记住这一点，让我们保护我们的执行器端点。

首先，让我们通过添加[安全启动器 Maven 依赖项](https://web.archive.org/web/20220628065013/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-security)来增加应用程序的安全性:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.5.1</version>
</dependency>
```

为了最基本的安全，这就是我们要做的。通过添加安全启动器，我们已经自动将基本身份验证应用于除了`/info`和`/health`之外的所有公开的端点。

现在，让我们定制我们的安全性，将`/actuator`端点限制为一个`ADMIN`角色。

让我们从排除默认安全配置开始:

```java
@SpringBootApplication(exclude = { 
    SecurityAutoConfiguration.class, 
    ManagementWebSecurityAutoConfiguration.class 
})
```

让我们记下`ManagementWebSecurityAutoConfiguration.class`，因为这将让我们将自己的安全配置应用到`/actuator`。

在我们的配置类中，让我们配置几个用户和角色，这样我们就有一个`ADMIN`角色可以使用:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
    auth
      .inMemoryAuthentication()
      .withUser("user")
      .password(encoder.encode("password"))
      .roles("USER")
      .and()
      .withUser("admin")
      .password(encoder.encode("admin"))
      .roles("USER", "ADMIN");
}
```

SpringBoot 为我们提供了一个方便的请求匹配器，用于我们的执行器端点。

让我们用它将我们的`/actuator`锁定在`ADMIN`角色上:

```java
http.requestMatcher(EndpointRequest.toAnyEndpoint())
  .authorizeRequests((requests) -> requests.anyRequest().hasRole("ADMIN"));
```

## 5.结论

在本教程中，我们学习了 Spring Boot 如何默认配置执行器。之后，我们定制在我们的`application.properties`文件中启用、禁用和公开哪些端点。因为默认情况下 Spring Boot 配置`/shutdown`端点的方式不同，所以我们学习了如何单独启用它。

在学习了基础知识之后，我们接着学习了如何配置执行器安全性。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220628065013/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-actuator)