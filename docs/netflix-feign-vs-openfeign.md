# 网飞佯和开放佯的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/netflix-feign-vs-openfeign>

## 1.概观

在本教程中，我们将描述[春云网飞佯](https://web.archive.org/web/20220626202852/https://spring.io/projects/spring-cloud-netflix)和[春云开放佯](https://web.archive.org/web/20220626202852/https://spring.io/projects/spring-cloud-openfeign)之间的区别。

## 2.假装

**[Feign](/web/20220626202852/https://www.baeldung.com/intro-to-feign) 通过提供注释支持**使得编写 web 服务客户端变得更加容易，注释支持允许我们只使用接口来实现我们的客户端。

最初，Feign 是由网飞创建并发布的，作为他们的[网飞 OSS](https://web.archive.org/web/20220626202852/https://netflix.github.io/) 项目的一部分。今天，它是一个开源项目。

### 2.1.春云网飞佯道

Spring Cloud 网飞将网飞 OSS 产品集成到了 Spring Cloud 生态系统中。这包括 Feign、Eureka、Ribbon 和许多其他工具和实用程序。然而，Feign 被赋予了自己的 Spring Cloud Starter 来允许访问 just Feign。

### 2.2.OpenFeign

最终，网飞决定在内部停止使用 Feign，并停止了它的开发。由于这个决定，网飞将 Feign 完全转移到了一个名为 [OpenFeign](https://web.archive.org/web/20220626202852/https://github.com/OpenFeign/feign) 的新项目下的开源社区。

幸运的是，它继续从开源社区获得巨大的支持，并且看到了许多新的特性和更新。

### 2.3.春云开佯

与其前身类似，Spring Cloud OpenFeign 将前身项目整合到 Spring Cloud 生态系统中。

方便的是，这种集成增加了对 Spring MVC 注释的支持，并提供了相同的 [HttpMessageConverters](/web/20220626202852/https://www.baeldung.com/spring-httpmessageconverter-rest) 。

让我们比较一下在 [Spring Cloud OpenFeign](/web/20220626202852/https://www.baeldung.com/spring-cloud-openfeign) 中发现的 Feign 实现和使用 Spring Cloud 网飞 Feign 的实现。

## 3.属国

首先，我们必须将`[spring-cloud-starter-feign](https://web.archive.org/web/20220626202852/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-starter-feign)`和`[spring-cloud-dependencies](https://web.archive.org/web/20220626202852/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-dependencies)` 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <versionId>1.4.7.RELEASE</versionID>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>Hoxton.SR8</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

请注意，这个库只适用于 Spring Boot 1.4.7 或更早的版本。因此，我们的`pom.xml`必须使用任何 Spring Cloud 依赖项的兼容版本。

## 4.实现与春云网飞佯

现在，我们可以使用`@EnableFeignClients`为任何使用`@FeignClient`的接口启用组件扫描。

对于我们使用 Spring Cloud 网飞 Feign 项目开发的每个示例，我们使用以下导入:

```java
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.cloud.netflix.feign.EnableFeignClients;
```

新旧版本的所有特性的实现完全相同。

## 5.用 Spring Cloud OpenFeign 实现

相比之下，我们的 [Spring Cloud OpenFeign 教程](/web/20220626202852/https://www.baeldung.com/spring-cloud-openfeign)包含了与 Spring 网飞 Feign 实现相同的例子。

这里唯一的区别是我们的进口来自不同的包装:

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
```

其他一切都是一样的，由于这两个库之间的关系，这并不奇怪。

## 6.比较

基本上，Feign 的这两个实现是相同的。我们可以把这归因于网飞佯是开放佯的祖先。

但是，Spring Cloud OpenFeign 包含了 Spring Cloud 网飞 Feign 中没有的新选项和功能。

最近可以获得对[千分尺](https://web.archive.org/web/20220626202852/https://micrometer.io/)、[drowizard Metrics](https://web.archive.org/web/20220626202852/https://metrics.dropwizard.io/)、 [Apache HTTP Client 5](https://web.archive.org/web/20220626202852/https://hc.apache.org/httpcomponents-client-5.0.x/index.html) 、 [Google HTTP client](https://web.archive.org/web/20220626202852/https://googleapis.github.io/google-http-java-client) 等等的支持。

## 7.结论

本文比较了 OpenFeign 和网飞 Feign 的 Spring Cloud 集成。像往常一样，你会在 GitHub 上找到 [Spring Cloud OpenFeign](https://web.archive.org/web/20220626202852/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-openfeign) 和[网飞 Feign](https://web.archive.org/web/20220626202852/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-netflix-feign) 的源代码。