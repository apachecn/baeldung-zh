# spring Cloud——使用 Zipkin 的跟踪服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tracing-services-with-zipkin>

## 1。概述

在本文中，我们将把`Zipkin`添加到我们的 [spring cloud 项目](/web/20220627091739/https://www.baeldung.com/spring-cloud-securing-services)中。`Zipkin`是一个开源项目，提供了发送、接收、存储和可视化跟踪的机制。这使我们能够关联服务器之间的活动，并更清楚地了解我们的服务中到底发生了什么。

本文不是分布式跟踪或 spring cloud 的入门文章。如果你想了解更多关于分布式跟踪的信息，请阅读我们对 [spring sleuth](/web/20220627091739/https://www.baeldung.com/spring-cloud-sleuth-single-application) 的介绍。

## 2。Zipkin 服务

我们的服务将作为我们所有跨度的商店。每个跨度都被发送到该服务，并被收集到跟踪中以供将来识别。

### 2.1。设置

创建一个新的 Spring Boot 项目，并将这些依赖项添加到`pom.xml:`

```java
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-server</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-autoconfigure-ui</artifactId>
    <scope>runtime</scope>
</dependency>
```

供参考:你可以在`Maven Central` ( [zipkin-server](https://web.archive.org/web/20220627091739/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.zipkin.java%22%20AND%20a%3A%22zipkin-server%22) 、 [zipkin-autoconfigure-ui](https://web.archive.org/web/20220627091739/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.zipkin.java%22%20AND%20a%3A%22zipkin-autoconfigure-ui%22) 上找到最新版本。依赖关系的版本继承自[spring-boot-starter-parent](https://web.archive.org/web/20220627091739/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-parent%22)。

### 2.2。启用 Zipkin 服务器

为了启用`Zipkin`服务器，我们必须向主应用程序类添加一些注释:

```java
@SpringBootApplication
@EnableZipkinServer
public class ZipkinApplication {...}
```

新的注释`@EnableZipkinServer`将设置这个服务器来监听传入的 spans，并作为我们的查询 UI。

### 2.3。配置

首先，让我们在`src/main/resources`中创建一个名为`bootstrap.properties`的文件。请记住，这个文件是从配置服务器获取我们的配置所必需的。

让我们给它添加这些属性:

```java
spring.cloud.config.name=zipkin
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword

eureka.client.serviceUrl.defaultZone=
  http://discUser:[[email protected]](/web/20220627091739/https://www.baeldung.com/cdn-cgi/l/email-protection):8082/eureka/
```

现在让我们添加一个配置文件到我们的配置报告中，在 Windows 上位于`c:\Users\{username}\`或者在*nix 上位于`/home/{username}/`。

在这个目录中，我们添加一个名为`zipkin.properties`的文件，并添加以下内容:

```java
spring.application.name=zipkin
server.port=9411
eureka.client.region=default
eureka.client.registryFetchIntervalSeconds=5
logging.level.org.springframework.web=debug
```

请记住提交此目录中的更改，以便配置服务可以检测到这些更改并加载文件。

### 2.4。运行

现在让我们运行我们的应用程序并导航到 http://localhost:9411。迎接我们的应该是`Zipkin's`主页:

[![zipkinhomepage 1](img/d2f57bdf8d582425d46aa535944946b3.png)](/web/20220627091739/https://www.baeldung.com/wp-content/uploads/2017/03/zipkinhomepage-1-1.png)

太好了！现在，我们准备向我们想要跟踪的服务添加一些依赖项和配置。

## 3。服务配置

资源服务器的设置非常相似。在接下来的章节中，我们将详细介绍如何设置`book-service.`,我们将解释将这些更新应用到`rating-service`和`gateway-service**.**`所需的修改

### 3.1。设置

为了开始向我们的`Zipkin`服务器发送跨度，我们将把这个依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

供参考:你可以在`Maven Central`([spring-cloud-starter-zipkin](https://web.archive.org/web/20220627091739/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-zipkin%22)上找到最新版本。

### 3.2。弹簧配置

我们需要添加一些配置，以便**预订服务**将使用`Eureka` 找到我们的`Zipkin` 服务。打开`BookServiceApplication.java`并将这段代码添加到文件中:

```java
@Autowired
private EurekaClient eurekaClient;

@Autowired
private SpanMetricReporter spanMetricReporter;

@Autowired
private ZipkinProperties zipkinProperties;

@Value("${spring.sleuth.web.skipPattern}")
private String skipPattern;

// ... the main method goes here

@Bean
public ZipkinSpanReporter makeZipkinSpanReporter() {
    return new ZipkinSpanReporter() {
        private HttpZipkinSpanReporter delegate;
        private String baseUrl;

        @Override
        public void report(Span span) {

            InstanceInfo instance = eurekaClient
              .getNextServerFromEureka("zipkin", false);
            if (!(baseUrl != null && 
              instance.getHomePageUrl().equals(baseUrl))) {
                baseUrl = instance.getHomePageUrl();
                delegate = new HttpZipkinSpanReporter(new RestTemplate(), 
                  baseUrl, zipkinProperties.getFlushInterval(), spanMetricReporter);
                if (!span.name.matches(skipPattern)) delegate.report(span);
            }
        }
    };
}
```

上面的配置注册了一个从 eureka 获取 URL 的定制`ZipkinSpanReporter`。这段代码还跟踪现有的 URL，并且只在 URL 改变时更新`HttpZipkinSpanReporter`。这样，无论我们在哪里部署我们的 Zipkin 服务器，我们都可以在不重启服务的情况下找到它。

我们还导入由 spring boot 加载的默认 Zipkin 属性，并使用它们来管理我们的自定义报告器。

### 3.3。配置

现在让我们向配置存储库中的`book-service.properties`文件添加一些配置:

```java
spring.sleuth.sampler.percentage=1.0
spring.sleuth.web.skipPattern=(^cleanup.*)
```

`Zipkin`通过在服务器上采样动作来工作。通过将`spring.sleuth.sampler.percentage`设置为 1.0，我们将采样率设置为 100%。skip 模式只是一个正则表达式，用于排除名称匹配的 spans。

跳过模式将阻止报告所有以单词“cleanup”开头的范围。这是为了停止源自 spring 会话代码库的跨度。

### 3.4。评级服务

遵循上述`book-service`部分的相同步骤，将更改应用于`rating-service.`的等效文件

### 3.5。网关服务

遵循相同的步骤**预订服务**。但是当向网关*添加配置时。属性*改为添加这些:

```java
spring.sleuth.sampler.percentage=1.0
spring.sleuth.web.skipPattern=(^cleanup.*|.+favicon.*)
```

这将配置网关服务不发送关于 favicon 或 spring 会话的跨度。

### 3.6。运行

如果您还没有这样做，请启动**配置**、**发现**、**网关**、**预订**、**评级**和 **zipkin** 服务。

导航到 http://localhost:8080/book-service/books。

打开一个新选项卡，导航到 http://localhost:9411。选择 book-service，然后按“查找踪迹”按钮。您应该会在搜索结果中看到一个跟踪。单击打开它痕迹:

[![zipkintrace 1](img/bf12bf1151f13e3a0e8207f36025c9d4.png)](/web/20220627091739/https://www.baeldung.com/wp-content/uploads/2017/03/zipkintrace-1-1.png)

在跟踪页面上，我们可以看到按服务细分的请求。前两个跨度由`gateway`创建，最后一个跨度由`book-service.`创建。这向我们显示了请求在`book-service,`上花费了 18.379 毫秒，在`gateway,`上花费了 87.961 毫秒

## 4。结论

我们已经看到将`Zipkin`集成到我们的云应用程序中是多么容易。

这给了我们一些非常需要的洞察力，让我们了解通信是如何通过我们的应用程序进行的。随着我们的应用程序变得越来越复杂，Zipkin 可以为我们提供急需的信息，告诉我们请求在哪里花费了时间。这可以帮助我们确定哪些地方慢了下来，并指出我们的应用程序哪些方面需要改进。

和往常一样，你可以在 Github 上找到源代码[。](https://web.archive.org/web/20220627091739/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-bootstrap)