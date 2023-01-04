# 使用 MSF4J 介绍 Java 微服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/msf4j>

## 1。概述

在本教程中，我们将展示使用 MSF4J **框架**的**微服务开发。**

这是一个轻量级的工具，它提供了一种简单的方法来构建各种侧重于高性能的服务。

## 2。Maven 依赖关系

我们需要比平常更多的 Maven 配置来构建基于 MSF4J 的微服务。这个框架的简单和强大是有代价的:**基本上，我们需要定义一个父工件**，以及主类:

```
<parent>
    <groupId>org.wso2.msf4j</groupId>
    <artifactId>msf4j-service</artifactId>
    <version>2.6.0</version>
</parent>

<properties>
    <microservice.mainClass>
        com.baeldung.msf4j.Application
    </microservice.mainClass>
</properties>
```

最新版本的 [msf4j-service](https://web.archive.org/web/20220525131127/https://search.maven.org/classic/#search%7Cga%7C1%7Cmsf4j-service) 可以在 Maven Central 上找到。

接下来，我们将展示三种不同的微服务场景。首先是一个极简示例，然后是一个 RESTful API，最后是一个 Spring 集成示例。

## 3。基本项目

### 3.1。简单 API

我们将发布一个简单的网络资源。

该服务提供了一个使用一些注释的类，其中每个方法处理一个请求。通过这些注释，我们设置了每个请求所需的方法、路径和参数。

返回的内容类型只是纯文本:

```
@Path("/")
public class SimpleService {

    @GET
    public String index() {
        return "Default content";
    }

    @GET
    @Path("/say/{name}")
    public String say(@PathParam("name") String name) {
        return "Hello " + name;
    }
}
```

请记住，所有使用的类和注释都只是标准的 JAX-RS 元素，我们已经在本文中介绍过 T2。

### 3.2。应用程序

**我们可以使用这个主类**启动微服务，在这里我们设置、部署和运行之前定义的服务:

```
public class Application {
    public static void main(String[] args) {
        new MicroservicesRunner()
          .deploy(new SimpleService())
          .start();
    }
}
```

如果我们愿意，我们可以在这里链接`deploy`调用来同时运行几个服务:

```
new MicroservicesRunner()
  .deploy(new SimpleService())
  .deploy(new ComplexService())
  .start()
```

### 3.3。运行微服务

要运行 MSF4J 微服务，我们有几个选项:

1.  在 IDE 上，作为 Java 应用程序运行
2.  运行生成的 jar 包

一旦开始，你可以在`http://localhost:9090`看到结果。

### 3.4。启动配置

我们可以通过在启动代码中添加一些子句的方式来调整配置。

例如，我们可以为请求添加任何类型的拦截器:

```
new MicroservicesRunner()
  .addInterceptor(new MetricsInterceptor())
  .deploy(new SimpleService())
  .start();
```

或者，我们可以添加一个全局拦截器，比如用于身份验证的拦截器:

```
new MicroservicesRunner()
  .addGlobalRequestInterceptor(newUsernamePasswordSecurityInterceptor())
  .deploy(new SimpleService())
  .start();
```

或者，如果我们需要会话管理，我们可以设置一个会话管理器:

```
new MicroservicesRunner()
  .deploy(new SimpleService())
  .setSessionManager(new PersistentSessionManager()) 
  .start();
```

要了解每个场景的更多细节并查看一些工作示例，请查看 MSF4J 的官方 GitHub repo 。

## 4。构建 API 微服务

我们已经展示了最简单的例子。现在我们将转向一个更现实的项目。

这一次，我们将展示如何用所有典型的 CRUD 操作构建一个 API 来管理膳食存储库。

### 4.1。型号

这个模型只是一个简单的 POJO，代表一顿饭:

```
public class Meal {
    private String name;
    private Float price;

    // getters and setters
}
```

### 4.2。API

我们将 API 构建为 web 控制器。使用标准注释，我们用以下内容设置每个函数:

*   path
*   HTTP 方法:GET、POST 等。
*   输入(`@Consumes`)内容类型
*   输出(`@Produces`)内容类型

因此，让我们为每个标准 CRUD 操作创建一个方法:

```
@Path("/menu")
public class MenuService {

    private List<Meal> meals = new ArrayList<Meal>();

    @GET
    @Path("/")
    @Produces({ "application/json" })
    public Response index() {
        return Response.ok()
          .entity(meals)
          .build();
    }

    @GET
    @Path("/{id}")
    @Produces({ "application/json" })
    public Response meal(@PathParam("id") int id) {
        return Response.ok()
          .entity(meals.get(id))
          .build();
    }

    @POST
    @Path("/")
    @Consumes("application/json")
    @Produces({ "application/json" })
    public Response create(Meal meal) {
        meals.add(meal);
        return Response.ok()
          .entity(meal)
          .build();
    }

    // ... other CRUD operations
}
```

### 4.3。数据转换功能

**MSF4J 提供对不同数据转换库**的支持，比如 GSON(默认提供)和 Jackson(通过 [msf4j-feature](https://web.archive.org/web/20220525131127/https://search.maven.org/classic/#search%7Cga%7C1%7Cmsf4j-feature) 依赖)。例如，我们可以显式地使用 GSON:

```
@GET
@Path("/{id}")
@Produces({ "application/json" })
public String meal(@PathParam("id") int id) {
    Gson gson = new Gson();
    return gson.toJson(meals.get(id));
}
```

顺便说一下，注意我们在`@Consumes`和`@Produces`注释中都使用了花括号，所以我们可以设置不止一个 mime 类型。

### 4.4。运行 API 微服务

我们通过发布`MenuService`的`Application`类来运行微服务，就像我们在前面的例子中所做的一样。

一旦启动，您可以在 http://localhost:9090/menu 看到结果。

## 5。MSF4J 和弹簧

**我们也可以在基于 MSF4J 的微服务**中应用 Spring，从中我们可以获得它的依赖注入特性。

### 5.1。Maven 依赖关系

我们必须将适当的依赖项添加到先前的 Maven 配置中，以添加 Spring 和 Mustache 支持:

```
<dependencies>
    <dependency>
        <groupId>org.wso2.msf4j</groupId>
        <artifactId>msf4j-spring</artifactId>
        <version>2.6.1</version>
    </dependency>
    <dependency>
        <groupId>org.wso2.msf4j</groupId>
        <artifactId>msf4j-mustache-template</artifactId>
        <version>2.6.1</version>
    </dependency>
</dependencies>
```

最新版本的 [msf4j-spring](https://web.archive.org/web/20220525131127/https://search.maven.org/classic/#search%7Cga%7C1%7Cmsf4j-spring) 和 [msf4j-mustache-template](https://web.archive.org/web/20220525131127/https://search.maven.org/classic/#search%7Cga%7C1%7Cmsf4j-mustache-template) 可以在 Maven Central 上找到。

### 5.2。膳食 API

这个 API 只是一个简单的服务，使用了一个模拟餐存储库。**注意我们如何使用 Spring 注释进行自动连接**并将这个类设置为 Spring 服务组件。

```
@Service
public class MealService {

    @Autowired
    private MealRepository mealRepository;

    public Meal find(int id) {
        return mealRepository.find(id);
    }

    public List<Meal> findAll() {
        return mealRepository.findAll();
    }

    public void create(Meal meal) {
        mealRepository.create(meal);
    }
}
```

### 5.3。控制器

我们将控制器声明为一个组件，Spring 通过自动连接提供服务。第一个方法展示了如何提供 Mustache 模板，第二个方法展示了 JSON 资源:

```
@Component
@Path("/meal")
public class MealResource {

    @Autowired
    private MealService mealService;

    @GET
    @Path("/")
    public Response all() {
        Map map = Collections.singletonMap("meals", mealService.findAll());
        String html = MustacheTemplateEngine.instance()
          .render("meals.mustache", map);
        return Response.ok()
          .type(MediaType.TEXT_HTML)
          .entity(html)
          .build();
    }

    @GET
    @Path("/{id}")
    @Produces({ "application/json" })
    public Response meal(@PathParam("id") int id) {
        return Response.ok()
          .entity(mealService.find(id))
          .build();
    }

}
```

### 5.4。主程序

在 Spring 场景中，我们是这样启动微服务的:

```
public class Application {

    public static void main(String[] args) {
        MSF4JSpringApplication.run(Application.class, args);
    }
}
```

一旦开始，我们可以在 http://localhost:8080/meals 看到结果。默认端口在 Spring 项目中有所不同，但是我们可以将它设置为我们想要的任何端口。

### 5.5。配置 bean

为了启用特定的设置，包括拦截器和会话管理，我们可以添加配置 beans。

例如，这将更改微服务的默认端口:

```
@Configuration
public class PortConfiguration {

    @Bean
    public HTTPTransportConfig http() {
        return new HTTPTransportConfig(9090);
    }

}
```

## 6.**结论**

在本文中，我们介绍了 MSF4J 框架，应用不同的场景来构建基于 Java 的微服务。

围绕这个概念有很多争论，但是[一些理论背景已经被设定](https://web.archive.org/web/20220525131127/https://martinfowler.com/articles/microservices.html)，并且 MSF4J 提供了一个方便和标准化的方法来应用这个模式。

此外，为了进一步阅读，看看[用 Eclipse Microprofile](/web/20220525131127/https://www.baeldung.com/eclipse-microprofile) 构建微服务，当然还有我们关于[用 Spring Boot 和 Spring Cloud](/web/20220525131127/https://www.baeldung.com/spring-microservices-guide) 构建 Spring 微服务的指南。

最后，这里的所有例子都可以在 GitHub repo 中找到[。](https://web.archive.org/web/20220525131127/https://github.com/eugenp/tutorials/tree/master/msf4j)