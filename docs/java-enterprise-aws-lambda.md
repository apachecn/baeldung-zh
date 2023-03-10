# 用 Java 编写企业级 AWS Lambda

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enterprise-aws-lambda>

## 1.概观

用 Java 编写一个基本的 AWS Lambda 不需要太多代码。为了保持简洁，我们通常创建没有框架支持的无服务器应用程序。

然而，如果我们需要以企业级的质量部署和监控我们的软件，我们需要解决许多用 Spring 这样的框架现成解决的问题。

在本教程中，我们将看看如何在 AWS Lambda 中包含**配置和日志功能，以及减少样板代码的库，同时仍然保持轻量级。**

## 2.构建一个示例

### 2.1.框架选项

像 Spring Boot 这样的框架不能用来创建 AWS Lambdas。Lambda 具有不同于服务器应用程序的生命周期，它与 AWS 运行时接口，而不直接使用 HTTP。

Spring 提供了 [Spring Cloud Function](/web/20221129012300/https://www.baeldung.com/spring-cloud-function) ，可以帮助我们创建一个 AWS Lambda，但是我们经常需要更小更简单的东西。

我们将从 [DropWizard](/web/20221129012300/https://www.baeldung.com/java-dropwizard) 中获得灵感，它的功能集比 Spring 小，但仍然支持通用标准，包括可配置性、日志记录和依赖注入。

虽然我们可能不需要从一个 Lambda 到下一个 Lambda 的所有这些特性，但我们将构建一个解决所有这些问题的示例，这样我们就可以选择在未来的开发中使用哪些技术。

### 2.2.示例问题

让我们创建一个每隔几分钟运行一次的应用程序。它会查看“待办事项列表”，找到没有标记为完成的最早的工作，然后创建一个博客帖子作为提醒。它还会生成有用的日志，允许 CloudWatch 警报发出错误警报。

我们将使用 [JsonPlaceholder](https://web.archive.org/web/20221129012300/https://jsonplaceholder.typicode.com/) 上的 API 作为我们的后端，我们将使应用程序对于 API 的基本 URL 和我们将在该环境中使用的凭证都是可配置的。

### 2.3.基本设置

我们将使用 [AWS SAM CLI](https://web.archive.org/web/20221129012300/https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) 创建一个基本的 [`Hello World Example`](/web/20221129012300/https://www.baeldung.com/java-aws-lambda-hibernate#2-creating-the-sam-template) 。

然后，我们将把默认的`App`类(其中有一个示例 API 处理程序)改为一个简单的在启动时记录的`RequestStreamHandler`:

```java
public class App implements RequestStreamHandler {

    @Override
    public void handleRequest(
      InputStream inputStream, 
      OutputStream outputStream, 
      Context context) throws IOException {
        context.getLogger().log("App starting\n");
    }
}
```

因为我们的例子不是一个 API 处理器，所以我们不需要读取任何输入或者产生任何输出。现在，我们正在使用传递给我们函数的`Context`中的`LambdaLogger`进行日志记录，不过稍后，我们将看看如何使用 [`Log4j`](/web/20221129012300/https://www.baeldung.com/log4j2-appenders-layouts-filters) 和`[Slf4j](/web/20221129012300/https://www.baeldung.com/slf4j-with-log4j2-logback).`

让我们快速测试一下:

```java
$ sam build
$ sam local invoke

Mounting todo-reminder/.aws-sam/build/ToDoFunction as /var/task:ro,delegated inside runtime container
App starting
END RequestId: 2aaf6041-cf57-4414-816d-76a63c7109fd
REPORT RequestId: 2aaf6041-cf57-4414-816d-76a63c7109fd  Init Duration: 0.12 ms  Duration: 121.70 ms
  Billed Duration: 200 ms Memory Size: 512 MB     Max Memory Used: 512 MB 
```

我们的存根应用程序已经启动，并且将`“App starting”`记录到日志中。

## 3.配置

由于我们可能将应用程序部署到多个环境中，或者希望将凭证之类的东西与代码分开，所以我们需要能够在部署或运行时传入配置值。这通常通过设置环境变量来实现。

### 3.1.向模板添加环境变量

**`template.yaml `文件包含λ**的设置。我们可以使用`AWS::Serverless::Function`部分下的`Environment`部分将环境变量添加到我们的函数中:

```java
Environment: 
  Variables:
    PARAM1: VALUE
```

生成的示例模板有一个硬编码的环境变量`PARAM1`，但是我们需要在部署时设置我们的环境变量。

假设我们希望应用程序知道变量`ENV_NAME`中的环境名称。

首先，让我们用一个默认的环境名在`template.yaml`文件的最顶端添加一个参数:

```java
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: todo-reminder application

Parameters:
  EnvironmentName:
    Type: String
    Default: dev
```

接下来，让我们将该参数连接到`AWS::Serverless::Function`部分中的环境变量:

```java
Environment: 
  Variables: 
    ENV_NAME: !Ref EnvironmentName
```

现在，我们准备在运行时读取环境变量。

### 3.2.读取环境变量

让[在构造我们的`App`对象时读取环境变量](/web/20221129012300/https://www.baeldung.com/java-system-get-property-vs-system-getenv#using-systemgetenv) `ENV_NAME`:

```java
private String environmentName = System.getenv("ENV_NAME");
```

我们还可以在调用`handleRequest`时记录环境:

```java
context.getLogger().log("Environment: " + environmentName + "\n");
```

日志消息必须以`“\n”`结尾，以分隔日志行。我们可以看到输出:

```java
$ sam build
$ sam local invoke

START RequestId: 12fb0c05-f222-4352-a26d-28c7b6e55ac6 Version: $LATEST
App starting
Environment: dev
```

在这里，我们看到环境已经从`template.yaml`中的缺省值进行了设置。

### 3.3.更改参数值

我们可以**使用参数覆盖在运行时**或部署时提供不同的值:

```java
$ sam local invoke --parameter-overrides "ParameterKey=EnvironmentName,ParameterValue=test"

START RequestId: 18460a04-4f8b-46cb-9aca-e15ce959f6fa Version: $LATEST
App starting
Environment: test
```

### 3.4.使用环境变量的单元测试

由于环境变量对应用程序来说是全局的，我们可能想用一个`private static final`常量来初始化它。然而，这使得单元测试非常困难。

由于在应用程序的整个生命周期中，处理程序类都是由 AWS Lambda 运行时作为单例进行初始化的，因此最好使用处理程序的实例变量来存储运行时状态。

我们可以使用[系统存根](/web/20221129012300/https://www.baeldung.com/java-system-stubs)来设置一个环境变量，并使用 [Mockito 深存根](/web/20221129012300/https://www.baeldung.com/mockito-fluent-apis#deep-mocking)来使我们的`LambdaLogger`在`Context`中可测试。首先，我们必须将`MockitoJUnitRunner`添加到测试中:

```java
@RunWith(MockitoJUnitRunner.class)
public class AppTest {

    @Mock(answer = Answers.RETURNS_DEEP_STUBS)
    private Context mockContext;

    // ...
}
```

接下来，我们可以在创建`App`对象之前使用`EnvironmentVariablesRule`来控制环境变量:

```java
@Rule
public EnvironmentVariablesRule environmentVariablesRule = 
  new EnvironmentVariablesRule();
```

现在，我们可以编写测试:

```java
environmentVariablesRule.set("ENV_NAME", "unitTest");
new App().handleRequest(fakeInputStream, fakeOutputStream, mockContext);

verify(mockContext.getLogger()).log("Environment: unitTest\n");
```

随着我们的 lambdas 变得越来越复杂，能够对 handler 类进行单元测试是非常有用的，包括它加载配置的方式。

## 4.处理复杂的配置

对于我们的例子，我们需要 API 的端点地址，以及环境的名称。端点可能在测试时有所不同，但是它有一个默认值。

我们可以多次使用`System.getenv`，甚至使用`Optional`和`orElse`来降低到一个默认值:

```java
String setting = Optional.ofNullable(System.getenv("SETTING"))
  .orElse("default");
```

然而，这可能需要大量重复的代码和许多个体`String`的协调。

### 4.1.将配置表示为 POJO

如果我们构建一个 Java 类来包含我们的配置，我们可以与需要它的服务共享它:

```java
public class Config {
    private String toDoEndpoint;
    private String postEndpoint;
    private String environmentName;

    // getters and setters
}
```

现在，我们可以用当前配置来构造我们的运行时组件:

```java
public class ToDoReaderService {
    public ToDoReaderService(Config configuration) {
        // ...
    }
}
```

该服务可以从`Config`对象获取它需要的任何配置值。我们甚至可以将配置建模为对象的层次结构，如果我们有像凭据这样的重复结构，这可能会很有用:

```java
private Credentials toDoCredentials;
private Credentials postCredentials;
```

到目前为止，这只是一个设计模式。让我们看看如何在实践中加载这些值。

### 4.2.配置加载程序

我们可以**使用[轻量级配置](https://web.archive.org/web/20221129012300/https://github.com/webcompere/lightweight-config)从我们资源中的`.yml`文件**加载我们的配置。

让我们将[依赖项](https://web.archive.org/web/20221129012300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22uk.org.webcompere%22%20AND%20a%3A%22lightweight-config%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>lightweight-config</artifactId>
    <version>1.1.0</version>
</dependency>
```

然后，让我们将一个`configuration.yml`文件添加到我们的`src/main/resources`目录。这个文件反映了我们的配置 POJO 的结构，包含硬编码的值、从环境变量填充的占位符和默认值:

```java
toDoEndpoint: https://jsonplaceholder.typicode.com/todos
postEndpoint: https://jsonplaceholder.typicode.com/posts
environmentName: ${ENV_NAME}
toDoCredentials:
  username: baeldung
  password: ${TODO_PASSWORD:-password}
postCredentials:
  username: baeldung
  password: ${POST_PASSWORD:-password}
```

我们可以使用`ConfigLoader`将这些设置加载到 POJO 中:

```java
Config config = ConfigLoader.loadYmlConfigFromResource("configuration.yml", Config.class);
```

这将填充来自环境变量的占位符表达式，在`:-`表达式后应用默认值。它非常类似于 DropWizard 内置的配置加载器。

### 4.3.在某处保持上下文

如果我们有几个组件——包括配置——要在 lambda 第一次启动时加载，将它们放在一个中心位置会很有用。

让我们创建一个名为`ExecutionContext`的类，`App`可以用它来创建对象:

```java
public class ExecutionContext {
    private Config config;
    private ToDoReaderService toDoReaderService;

    public ExecutionContext() {
        this.config = 
          ConfigLoader.loadYmlConfigFromResource("configuration.yml", Config.class);
        this.toDoReaderService = new ToDoReaderService(config);
    }
}
```

`App`可以在其初始化列表中创建其中一个:

```java
private ExecutionContext executionContext = new ExecutionContext();
```

现在，当`App`需要一个“bean”时，它可以从这个对象中获得它。

## 5.更好的记录

到目前为止，我们对`LambdaLogger`的使用还是非常基础的。如果我们引入由[执行日志](/web/20221129012300/https://www.baeldung.com/java-logging-intro)的库，那么**可能会期望`Log4j`或`Slf4j`出现**。理想情况下，我们的日志行将有时间戳和其他有用的上下文信息。

最重要的是，当我们遇到错误时，我们应该记录大量有用的信息，而且`Logger.error`通常比自制代码做得更好。

### 5.1.添加 AWS Log4j 库

我们可以通过向我们的`pom.xml`添加依赖项来启用 [AWS lambda `Log4j`运行时](https://web.archive.org/web/20221129012300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.amazonaws%22%20AND%20a%3A%22aws-lambda-java-log4j2%22):

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-log4j2</artifactId>
    <version>1.2.0</version>
</dependency>
```

我们还需要在`src/main/resources`中配置一个`log4j2.xml`文件来使用这个记录器:

```java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration packages="com.amazonaws.services.lambda.runtime.log4j2">
    <Appenders>
        <Lambda name="Lambda">
            <PatternLayout>
                <pattern>%d{yyyy-MM-dd HH:mm:ss} %X{AWSRequestId} %-5p %c{1} - %m%n</pattern>
            </PatternLayout>
        </Lambda>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Lambda" />
        </Root>
    </Loggers>
</Configuration>
```

### 5.2.编写日志记录语句

现在，我们将标准的`Log4j` `Logger`样板文件添加到我们的类中:

```java
public class ToDoReaderService {
    private static final Logger LOGGER = LogManager.getLogger(ToDoReaderService.class);

    public ToDoReaderService(Config configuration) {
        LOGGER.info("ToDo Endpoint on: {}", configuration.getToDoEndpoint());
        // ...
    }

    // ...
}
```

然后我们可以从命令行测试它:

```java
$ sam build
$ sam local invoke

START RequestId: acb34989-980c-42e5-b8e4-965d9f497d93 Version: $LATEST
2021-05-23 20:57:15  INFO  ToDoReaderService - ToDo Endpoint on: https://jsonplaceholder.typicode.com/todos 
```

### 5.3.单元测试日志输出

在测试日志输出很重要的情况下，我们可以使用系统存根来完成。我们的配置针对 AWS Lambda 进行了优化，将日志输出定向到`System.out`，我们可以利用它:

```java
@Rule
public SystemOutRule systemOutRule = new SystemOutRule();

@Test
public void whenTheServiceStarts_thenItOutputsEndpoint() {
    Config config = new Config();
    config.setToDoEndpoint("https://todo-endpoint.com");
    ToDoReaderService service = new ToDoReaderService(config);

    assertThat(systemOutRule.getLinesNormalized())
      .contains("ToDo Endpoint on: https://todo-endpoint.com");
}
```

### 5.4.添加 Slf4j 支持

我们可以通过添加依赖关系来添加`[Slf4j](https://web.archive.org/web/20221129012300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.logging.log4j%22%20AND%20a%3A%22log4j-slf4j-impl%22)`:

```java
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.13.2</version>
</dependency>
```

这允许我们查看来自`Slf4j`启用库的日志消息。我们也可以直接使用它:

```java
public class ExecutionContext {
    private static final Logger LOGGER =
      LoggerFactory.getLogger(ExecutionContext.class);

    public ExecutionContext() {
        LOGGER.info("Loading configuration");
        // ...
    }

    // ...
}
```

`Slf4j`日志记录通过 AWS `Log4j`运行时路由:

```java
$ sam local invoke

START RequestId: 60b2efad-bc77-475b-93f6-6fa7ddfc9f88 Version: $LATEST
2021-05-23 21:13:19  INFO  ExecutionContext - Loading configuration 
```

## 6.使用 Feign 使用 REST API

如果我们的 Lambda 使用 REST 服务，我们可以直接使用 Java HTTP 库。然而，使用轻量级框架也有好处。

OpenFeign 是一个很好的选择。它允许我们为 HTTP 客户端、日志、JSON 解析等等插入我们选择的组件。

### 6.1.添加假装

在这个例子中，我们将使用[假装](https://web.archive.org/web/20221129012300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-core%22)默认客户端，尽管 [Java 11 客户端](https://web.archive.org/web/20221129012300/https://github.com/OpenFeign/feign/tree/master/java11)也是一个非常好的选择，并且基于 Amazon Corretto 与 Lambda `java11`运行时一起工作。

另外，我们将使用 [`Slf4j`](https://web.archive.org/web/20221129012300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-slf4j%22) 日志和 [`Gson`](https://web.archive.org/web/20221129012300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-gson%22) 作为我们的 JSON 库:

```java
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-core</artifactId>
    <version>11.2</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-slf4j</artifactId>
    <version>11.2</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-gson</artifactId>
    <version>11.2</version>
</dependency>
```

我们在这里使用`Gson`作为我们的 JSON 库，因为 [`Gson`比`Jackson`](/web/20221129012300/https://www.baeldung.com/jackson-vs-gson) 小得多。我们可以使用 [`Jackson`](/web/20221129012300/https://www.baeldung.com/jackson) ，但是这会使启动时间变慢。还有使用 [`Jackson-jr`](https://web.archive.org/web/20221129012300/https://github.com/OpenFeign/feign/tree/master/jackson-jr) 的选项，尽管这仍然是实验性的。

### 6.2.定义一个伪装接口

首先，我们用一个接口来描述我们将要调用的 API:

```java
public interface ToDoApi {
    @RequestLine("GET /todos")
    List<ToDoItem> getAllTodos();
}
```

这描述了 API 中的路径和任何将从 JSON 响应中产生的对象。让我们创建`ToDoItem`来模拟 API 的响应:

```java
public class ToDoItem {
    private int userId;
    private int id;
    private String title;
    private boolean completed;

    // getters and setters
}
```

### 6.3.从界面定义客户端

接下来，我们使用`Feign.Builder`将`interface`转换成客户端:

```java
ToDoApi toDoApi = Feign.builder()
  .decoder(new GsonDecoder())
  .logger(new Slf4jLogger())
  .target(ToDoApi.class, config.getToDoEndpoint());
```

在我们的例子中，我们还使用了凭证。假设这些是通过基本认证提供的，这将要求我们在`target`调用之前添加一个`BasicAuthRequestInterceptor`:

```java
.requestInterceptor(
   new BasicAuthRequestInterceptor(
     config.getToDoCredentials().getUsername(),
     config.getToDoCredentials().getPassword()))
```

## 7.将物体连接在一起

到目前为止，我们已经为我们的应用程序创建了配置和 beans，但是我们还没有将它们连接在一起。对此我们有两种选择。我们要么使用普通的 Java 将对象连接在一起，要么使用某种依赖注入解决方案。

### 7.1.构造函数注入

因为一切都是普通的 Java 对象，而且我们已经构建了`ExecutionContext`类来协调构造，所以我们可以在它的构造函数中完成所有的工作。

我们可能希望扩展构造函数来按顺序构建所有的 beans:

```java
this.config = ... // load config
this.toDoApi = ... // build api
this.postApi = ... // build post API
this.toDoReaderService = new ToDoReaderService(toDoApi);
this.postService = new PostService(postApi);
```

这是最简单的解决方案。它鼓励定义良好的组件，这些组件在运行时既可测试又易于组合。

然而，超过一定数量的组件，这就开始变得冗长和难以管理。

### 7.2.引入依赖注入框架

DropWizard 使用 [Guice](/web/20221129012300/https://www.baeldung.com/guice) 进行依赖注入。这个库相对较小，可以帮助管理 AWS Lambda 中的组件。

让我们添加它的[依赖关系](https://web.archive.org/web/20221129012300/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.inject%22%20AND%20a%3A%22guice%22):

```java
<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>5.0.1</version>
</dependency>
```

### 7.3.在容易注射的地方注射

我们可以用 **`@Inject`注释来注释从其他 bean 构造的 bean，使它们可以自动注入**:

```java
public class PostService {
    private PostApi postApi;

    @Inject
    public PostService(PostApi postApi) {
        this.postApi = postApi;
    }

    // other functions
}
```

### 7.4.创建自定义进样模块

对于任何我们必须使用定制装载或构造代码的 beans，我们可以**使用`Module` 作为工厂**:

```java
public class Services extends AbstractModule {
    @Override
    protected void configure() {
        Config config = 
          ConfigLoader.loadYmlConfigFromResource("configuration.yml", Config.class);

        ToDoApi toDoApi = Feign.builder()
          .decoder(new GsonDecoder())
          .logger(new Slf4jLogger())
          .logLevel(FULL)
          .requestInterceptor(... // omitted
          .target(ToDoApi.class, config.getToDoEndpoint());

        PostApi postApi = Feign.builder()
          .encoder(new GsonEncoder())
          .logger(new Slf4jLogger())
          .logLevel(FULL)
          .requestInterceptor(... // omitted
          .target(PostApi.class, config.getPostEndpoint());

        bind(Config.class).toInstance(config);
        bind(ToDoApi.class).toInstance(toDoApi);
        bind(PostApi.class).toInstance(postApi);
    }
}
```

然后我们通过一个`Injector`在`ExecutionContext`中使用这个模块:

```java
public ExecutionContext() {
    LOGGER.info("Loading configuration");

    try {
        Injector injector = Guice.createInjector(new Services());
        this.toDoReaderService = injector.getInstance(ToDoReaderService.class);
        this.postService = injector.getInstance(PostService.class);
    } catch (Exception e) {
        LOGGER.error("Could not start", e);
    }
}
```

这种方法伸缩性很好，因为它将 bean 依赖关系局限于最接近每个 bean 的类。有了构建每个 bean 的中央配置类，任何依赖关系的改变总是需要在那里进行改变。

我们还应该注意到**记录启动过程中发生的错误**很重要——如果失败，Lambda 将无法运行。

### 7.5.一起使用这些对象

现在我们有了一个包含服务的`ExecutionContext` ，服务内部有 API，由`Config`配置，让我们完成我们的处理程序:

```java
@Override
public void handleRequest(InputStream inputStream, 
  OutputStream outputStream, Context context) throws IOException {

    PostService postService = executionContext.getPostService();
    executionContext.getToDoReaderService()
      .getOldestToDo()
      .ifPresent(postService::makePost);
}
```

让我们来测试一下:

```java
$ sam build
$ sam local invoke

Mounting /Users/ashleyfrieze/dev/tutorials/aws-lambda/todo-reminder/.aws-sam/build/ToDoFunction as /var/task:ro,delegated inside runtime container
2021-05-23 22:29:43  INFO  ExecutionContext - Loading configuration
2021-05-23 22:29:44  INFO  ToDoReaderService - ToDo Endpoint on: https://jsonplaceholder.typicode.com
App starting
Environment: dev
2021-05-23 22:29:44 73264c34-ca48-4c3e-a2b4-5e7e74e13960 INFO  PostService - Posting about: ToDoItem{userId=1, id=1, title='delectus aut autem', completed=false}
2021-05-23 22:29:44 73264c34-ca48-4c3e-a2b4-5e7e74e13960 INFO  PostService - Post: PostItem{title='To Do is Out Of Date: 1', body='Not done: delectus aut autem', userId=1}
END RequestId: 73264c34-ca48-4c3e-a2b4-5e7e74e13960 
```

## 8.结论

在本文中，我们研究了在使用 Java 构建企业级 AWS Lambda 时，配置和日志记录等特性的重要性。我们看到了像 Spring 和 DropWizard 这样的框架是如何默认提供这些工具的。

我们探讨了如何使用环境变量来控制配置，以及如何构建我们的代码以使单元测试成为可能。

然后，我们查看了用于加载配置、构建 REST 客户端、编组 JSON 数据以及将我们的对象连接在一起的库，重点是选择较小的库来使我们的 Lambda 尽快启动。

和往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129012300/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-lambda)