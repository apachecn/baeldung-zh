# Spring Boot @ rest client test 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/restclienttest-in-spring-boot>

## 1。简介

本文是对`@RestClientTest`注释的快速介绍。

新的注释有助于简化和加速 Spring 应用程序中 REST 客户机的测试。

## 2。Spring Boot 1.4 版之前的 REST 客户端支持

Spring Boot 是一个方便的框架，它提供了许多具有典型设置的自动配置的 Spring beans，允许您更少地关注 Spring 应用程序的配置，而更多地关注您的代码和业务逻辑。

但是在 1.3 版本中，当我们想要创建或测试 REST 服务客户端时，我们没有得到很多帮助。它对 REST 客户端的支持不是很深。

为 REST API 创建一个客户端——通常使用一个`RestTemplate`实例。通常它必须在使用前进行配置，其配置可能会有所不同，所以 Spring Boot 不提供任何通用配置的`RestTemplate` bean。

测试 REST 客户端也是如此。在 Spring Boot 1.4.0 之前，测试 Spring REST 客户端的过程与任何其他基于 Spring 的应用程序没有太大的不同。您将创建一个`MockRestServiceServer`实例，将它绑定到测试中的`RestTemplate`实例，并向它提供对请求的模拟响应，如下所示:

```java
RestTemplate restTemplate = new RestTemplate();

MockRestServiceServer mockServer =
  MockRestServiceServer.bindTo(restTemplate).build();
mockServer.expect(requestTo("/greeting"))
  .andRespond(withSuccess());

// Test code that uses the above RestTemplate ...

mockServer.verify();
```

您还必须初始化 Spring 容器，并确保只将需要的组件加载到上下文中，以加快上下文加载时间(从而加快测试执行时间)。

## 3。Spring Boot 1.4 版新增 REST 客户端特性

在 Spring Boot 1.4 中，团队为简化和加速 REST 客户端的创建和测试付出了巨大的努力。

那么，让我们来看看新的功能。

### 3.1。将 Spring Boot 添加到您的项目中

首先，您需要确保您的项目使用的是 Spring Boot 1.4.x 或更高版本:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies> 
```

最新发布的版本可以在这里找到。

### 3.2。`RestTemplateBuilder`

Spring Boot 带来了自动配置的`RestTemplateBuilder`来简化创建`RestTemplates`，以及匹配的`@RestClientTest`注释来测试用`RestTemplateBuilder`构建的客户端。下面是如何创建一个简单的 REST 客户端，并自动注入`RestTemplateBuilder`:

```java
@Service
public class DetailsServiceClient {

    private final RestTemplate restTemplate;

    public DetailsServiceClient(RestTemplateBuilder restTemplateBuilder) {
        restTemplate = restTemplateBuilder.build();
    }

    public Details getUserDetails(String name) {
        return restTemplate.getForObject("/{name}/details",
          Details.class, name);
    }
}
```

注意，我们没有将`RestTemplateBuilder`实例显式连接到构造函数。这要归功于 Spring 的一个新特性，叫做隐式构造函数注入，这在本文的[中讨论过。](/web/20220926195643/https://www.baeldung.com/whats-new-in-spring-4-3)

提供注册消息转换器、错误处理程序、URI 模板处理程序、基本授权的便利方法，还可以使用任何你需要的定制程序。

### 3.3。`@RestClientTest`

为了测试这样一个用`RestTemplateBuilder`构建的 REST 客户端，您可以使用一个用`@RestClientTest`注释的`SpringRunner`执行的测试类。该注释禁用完全自动配置，仅应用与 REST 客户端测试相关的配置，即 Jackson 或 GSON 自动配置和`@JsonComponent`bean，而不是常规的`@Component`bean。

`@RestClientTest`确保 Jackson 和 GSON 支持是自动配置的，还将预先配置的`RestTemplateBuilder`和`MockRestServiceServer`实例添加到上下文中。被测 bean 用`@RestClientTest`注释的`value`或`components`属性指定:

```java
@RunWith(SpringRunner.class)
@RestClientTest(DetailsServiceClient.class)
public class DetailsServiceClientTest {

    @Autowired
    private DetailsServiceClient client;

    @Autowired
    private MockRestServiceServer server;

    @Autowired
    private ObjectMapper objectMapper;

    @Before
    public void setUp() throws Exception {
        String detailsString = 
          objectMapper.writeValueAsString(new Details("John Smith", "john"));

        this.server.expect(requestTo("/john/details"))
          .andRespond(withSuccess(detailsString, MediaType.APPLICATION_JSON));
    }

    @Test
    public void whenCallingGetUserDetails_thenClientMakesCorrectCall() 
      throws Exception {

        Details details = this.client.getUserDetails("john");

        assertThat(details.getLogin()).isEqualTo("john");
        assertThat(details.getName()).isEqualTo("John Smith");
    }
}
```

首先，我们需要通过添加`@RunWith(SpringRunner.class)`注释来确保这个测试是用`SpringRunner` 运行的。

那么，有什么新消息吗？

**第一个**—`@RestClientTest`注释允许我们指定测试中的确切服务——在我们的例子中是`DetailsServiceClient`类。这个服务将被加载到测试上下文中，而其他的都被过滤掉了。

这允许我们在测试中自动连接`DetailsServiceClient`实例，将其他所有东西留在测试之外，这加快了上下文的加载。

**第二个**——由于`MockRestServiceServer`实例也被配置用于一个`@RestClientTest`注释的测试(并为我们绑定到`DetailsServiceClient`实例)，我们可以简单地注入并使用它。

**最后**–JSON 对`@RestClientTest`的支持允许我们注入 Jackson 的`ObjectMapper`实例来准备`MockRestServiceServer’s`模拟答案值。

剩下要做的就是执行对我们服务的调用并验证结果。

## 4。结论

在本文中，我们讨论了新的`@RestClientTest`注释，它允许简单快速地测试用 Spring 构建的 REST 客户端。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220926195643/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-client)