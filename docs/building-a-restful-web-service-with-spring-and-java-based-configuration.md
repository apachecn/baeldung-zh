# 用 Spring 和 Java 配置构建一个 REST API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration>

## 1。概述

在本教程中，我们将学习如何**在 Spring 中设置 REST，**包括控制器和 HTTP 响应代码、有效载荷编组的配置以及内容协商。

## 延伸阅读:

## [使用 Spring @ResponseStatus 设置 HTTP 状态码](/web/20220824083901/https://www.baeldung.com/spring-response-status)

Have a look at the @ResponseStatus annotation and how to use it to set the response status code.[Read more](/web/20220824083901/https://www.baeldung.com/spring-response-status) →

## [Spring @ Controller 和@RestController 注释](/web/20220824083901/https://www.baeldung.com/spring-controller-vs-restcontroller)

Learn about the differences between @Controller and @RestController annotations in Spring MVC.[Read more](/web/20220824083901/https://www.baeldung.com/spring-controller-vs-restcontroller) →

## 2。 了解春天的休息

Spring 框架支持两种创建 RESTful 服务的方式:

*   使用 MVC 和`ModelAndView`
*   使用 HTTP 消息转换器

`ModelAndView`方法更老，文档也更好，但也更冗长，配置也更多。它试图将 REST 范式硬塞进旧模型，这并不是没有问题。Spring 团队明白这一点，并从 Spring 3.0 开始提供一流的 REST 支持。

**基于`HttpMessageConverter `和注释的新方法更加轻量级，也更容易实现。**配置是最少的，它为我们期望从 RESTful 服务中得到的东西提供了合理的默认值。

## 3。Java 配置

```java
@Configuration
@EnableWebMvc
public class WebConfig{
   //
}
```

新的`@EnableWebMvc`注释做了一些有用的事情；具体来说，在 REST 的情况下，它检测类路径上是否存在 Jackson 和 JAXB 2，并自动创建和注册默认的 JSON 和 XML 转换器。注释的功能等同于 XML 版本:

`<mvc:annotation-driven />`

这是一种捷径，尽管在许多情况下可能有用，但并不完美。当我们需要更复杂的配置时，可以去掉注释，直接扩展`WebMvcConfigurationSupport`。

### 3.1.使用 Spring Boot

如果我们使用的是`@SpringBootApplication`注释，并且`spring-webmvc `库在类路径中，那么`@EnableWebMvc`注释会自动添加，而[是默认的自动配置](https://web.archive.org/web/20220824083901/https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-auto-configuration)。

我们仍然可以通过在一个`@Configuration `注释类上实现`WebMvcConfigurer`接口来为这个配置添加 MVC 功能。我们还可以使用一个`WebMvcRegistrationsAdapter` 实例来提供我们自己的`RequestMappingHandlerMapping`、`RequestMappingHandlerAdapter`或`ExceptionHandlerExceptionResolver `实现。

最后，如果我们想放弃 Spring Boot 的 MVC 特性并声明一个定制配置，我们可以通过使用`@EnableWebMvc`注释来实现。

## 4。测试 Spring 上下文

从 Spring 3.1 开始，我们获得了对`@Configuration`类的一流测试支持:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration( 
  classes = {WebConfig.class, PersistenceConfig.class},
  loader = AnnotationConfigContextLoader.class)
public class SpringContextIntegrationTest {

   @Test
   public void contextLoads(){
      // When
   }
}
```

我们用`@ContextConfiguration`注释指定 Java 配置类。新的`AnnotationConfigContextLoader`从`@Configuration`类加载 bean 定义。

注意,`WebConfig`配置类没有包含在测试中，因为它需要在 Servlet 上下文中运行，但是没有提供。

### 4.1.使用 Spring Boot

Spring Boot 提供了几个注释，以更直观的方式为我们的测试设置 Spring `ApplicationContext` 。

我们可以只加载应用程序配置的特定部分，也可以模拟整个上下文启动过程。

例如，如果我们想在不启动服务器的情况下创建整个上下文，我们可以使用`@SpringBootTest`注释。

准备就绪后，我们可以添加`@AutoConfigureMockMvc`来注入一个`MockMvc `实例并发送 HTTP 请求`:`

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class FooControllerAppIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenTestApp_thenEmptyResponse() throws Exception {
        this.mockMvc.perform(get("/foos")
            .andExpect(status().isOk())
            .andExpect(...);
    }

}
```

为了避免创建整个上下文并只测试我们的 MVC 控制器，我们可以使用`@WebMvcTest:`

```java
@RunWith(SpringRunner.class)
@WebMvcTest(FooController.class)
public class FooControllerWebLayerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private IFooService service;

    @Test()
    public void whenTestMvcController_thenRetrieveExpectedResult() throws Exception {
        // ...

        this.mockMvc.perform(get("/foos")
            .andExpect(...);
    }
}
```

我们可以在[我们的“Spring Boot 测试”文章](/web/20220824083901/https://www.baeldung.com/spring-boot-testing)中找到关于这个主题的详细信息。

## 5。控制器

**`@RestController`是 RESTful API 的整个 Web 层的核心构件。**出于本文的目的，控制器正在建模一个简单的 REST 资源，`Foo`:

```java
@RestController
@RequestMapping("/foos")
class FooController {

    @Autowired
    private IFooService service;

    @GetMapping
    public List<Foo> findAll() {
        return service.findAll();
    }

    @GetMapping(value = "/{id}")
    public Foo findById(@PathVariable("id") Long id) {
        return RestPreconditions.checkFound(service.findById(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Long create(@RequestBody Foo resource) {
        Preconditions.checkNotNull(resource);
        return service.create(resource);
    }

    @PutMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void update(@PathVariable( "id" ) Long id, @RequestBody Foo resource) {
        Preconditions.checkNotNull(resource);
        RestPreconditions.checkNotNull(service.getById(resource.getId()));
        service.update(resource);
    }

    @DeleteMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void delete(@PathVariable("id") Long id) {
        service.deleteById(id);
    }

}
```

正如我们所看到的，我们正在使用一个简单的、番石榴风格的`RestPreconditions`实用程序:

```java
public class RestPreconditions {
    public static <T> T checkFound(T resource) {
        if (resource == null) {
            throw new MyResourceNotFoundException();
        }
        return resource;
    }
}
```

控制器实现是非公共的，因为它不需要是公共的。

通常，控制器是依赖链中的最后一个。它从 Spring front 控制器( `DispatcherServlet`)接收 HTTP 请求，并简单地将它们转发给服务层。如果没有必须通过直接引用来注入或操作控制器的用例，那么我们可能不愿意将它声明为公共的。

请求映射非常简单。**与任何控制器一样，映射的实际`value`以及 HTTP 方法决定了请求的目标方法。** @ `RequestBody`将方法的参数绑定到 HTTP 请求的主体，而`@ResponseBody`对响应和返回类型做同样的事情。

**`@RestController `是[的简写](/web/20220824083901/https://www.baeldung.com/spring-controller-vs-restcontroller)，包含了我们的`.`** 类中的`@ResponseBody`和`@Controller` 注释

它们还确保使用正确的 HTTP 转换器对资源进行编组和解组。将进行内容协商来选择将使用哪个活动转换器，主要基于`Accept`头，尽管也可以使用其他 HTTP 头来确定表示。

## 6。映射 HTTP 响应代码

HTTP 响应的状态代码是 REST 服务最重要的部分之一，这个主题很快会变得非常复杂。获得这些权利可能是服务成功或失败的关键。

### 6.1。未映射的请求

如果 Spring MVC 收到一个没有映射的请求，它会认为这个请求不被允许，并向客户端返回一个 405 方法不被允许。

当向客户端返回一个`405`来指定允许哪些操作时，包含`Allow` HTTP 头也是一个好的做法。这是 Spring MVC 的标准行为，不需要任何额外的配置。

### 6.2。有效的映射请求

对于任何有映射的请求，如果没有指定其他状态代码，Spring MVC 认为请求是有效的，并以 200 OK 响应。

正因为如此，控制器为`create`、`update`和`delete`动作声明了不同的`@ResponseStatus`，而没有为`get`声明，后者确实应该返回默认的 200 OK。

### 6.3。客户端错误

在客户端出错的情况下，自定义异常被定义并映射到适当的错误代码。

简单地从 web 层的任何一层抛出这些异常将确保 Spring 在 HTTP 响应上映射相应的状态代码:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BadRequestException extends RuntimeException {
   //
}
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
   //
}
```

这些异常是 REST API 的一部分，因此，我们应该只在与 REST 对应的适当层中使用它们；例如，如果 DAO/DAL 层存在，它不应该直接使用异常。

还要注意，这些不是检查异常，而是符合 Spring 实践和习惯用法的运行时异常。

### 6.4。使用`@ExceptionHandler`

将定制异常映射到特定状态代码的另一个选项是使用控制器中的`@ExceptionHandler`注释。这种方法的问题是注释只适用于定义它的控制器。这意味着我们需要在每个控制器中单独声明它们。

当然，在 Spring 和 Spring Boot 中有更多的处理错误的方式提供了更多的灵活性。

## 7 .**。附加美文依赖**

除了标准 web 应用程序所需的`spring-webmvc`依赖关系[之外，我们还需要为 REST API 设置内容编组和解组:](/web/20220824083901/https://www.baeldung.com/spring-with-maven#mvc "Spring Maven Web dependencies")

```java
<dependencies>
   <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.8</version>
   </dependency>
   <dependency>
      <groupId>javax.xml.bind</groupId>
      <artifactId>jaxb-api</artifactId>
      <version>2.3.1</version>
      <scope>runtime</scope>
   </dependency>
</dependencies>
```

这些是我们将用来将 REST 资源的表示转换成 JSON 或 XML 的库。

### 7.1.使用 Spring Boot

如果我们想要检索 JSON 格式的资源，Spring Boot 提供了对不同库的支持，即 Jackson、Gson 和 JSON-B

我们可以通过简单地在类路径中包含任何映射库来执行自动配置。

通常，如果我们正在开发一个 web 应用程序，**我们只需添加`spring-boot-starter-web`依赖项，并依靠它将所有必要的工件包含到我们的项目**中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.6.1</version>
</dependency>
```

Spring Boot 默认使用杰克森。

如果我们想以 XML 格式序列化我们的资源，我们必须将 Jackson XML 扩展(`jackson-dataformat-xml`)添加到我们的依赖项中，或者通过在我们的资源上使用`@XmlRootElement`注释退回到 JAXB 实现(在 JDK 中默认提供)。

## 8。结论

本文展示了如何使用 Spring 和基于 Java 的配置来实现和配置 REST 服务。

在本系列的下一篇文章中，我们将关注 API 的[可发现性、高级内容协商以及使用 a `Resource.`的附加表示](/web/20220824083901/https://www.baeldung.com/restful-web-service-discoverability "HATEOAS for the API")

本文中的所有代码都可以在 Github 上的[处获得。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20220824083901/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest)