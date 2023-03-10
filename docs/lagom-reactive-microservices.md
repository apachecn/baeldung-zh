# 使用 Lagom 框架的反应式微服务指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lagom-reactive-microservices>

## 1。概述

在本文中，我们将探索 [Lagom 框架](https://web.archive.org/web/20221205142529/https://www.lightbend.com/platform/development/lagom-framework)和**使用反应式微服务驱动架构实现一个示例应用**。

简单地说，反应式软件应用依赖于消息驱动的异步通信，本质上是高度`Responsive`、`Resilient`和`Elastic`。

通过微服务驱动的架构，我们意味着将系统分成协作服务之间的边界，以实现`Isolation`、`Autonomy`、`Single Responsibility`、`Mobility`等目标。关于这两个概念的进一步阅读，请参考[反应宣言](https://web.archive.org/web/20221205142529/http://www.reactivemanifesto.org/)和[反应微服务架构](https://web.archive.org/web/20221205142529/https://info.lightbend.com/COLL-20XX-Reactive-Microservices-Architecture-RES-LP.html)。

## 2。为什么是我？

Lagom 是一个开源框架，构建时考虑到了从单片到微服务驱动的应用程序架构的转变。它抽象了构建、运行和监控微服务驱动的应用程序的复杂性。

在幕后，Lagom framework 使用 [Play Framework](https://web.archive.org/web/20221205142529/https://www.playframework.com/) ，一个 [Akka](https://web.archive.org/web/20221205142529/http://akka.io/) 消息驱动的运行时， [Kafka](https://web.archive.org/web/20221205142529/https://kafka.apache.org/) 用于解耦服务，[事件源](https://web.archive.org/web/20221205142529/https://martinfowler.com/eaaDev/EventSourcing.html)和 [CQRS](https://web.archive.org/web/20221205142529/https://martinfowler.com/bliki/CQRS.html) 模式，[行为](https://web.archive.org/web/20221205142529/https://www.lagomframework.com/documentation/1.3.x/java/ConductR.html)支持在容器环境中监控和扩展微服务。

## 3。拉各斯的 hello World

我们将创建一个 Lagom 应用程序来处理来自用户的问候请求，并回复一条问候消息以及当天的天气统计数据。

我们将开发两个独立的微服务:`Greeting`和`Weather.`

`Greeting` 将专注于处理问候请求，与天气服务交互以回复用户。`Weather` 微服务将为今天的天气统计请求提供服务。

在现有用户与`Greeting`微服务交互的情况下，将向用户显示不同的问候消息。

### 3.1。先决条件

1.  从[这里](https://web.archive.org/web/20221205142529/https://www.scala-lang.org/download/2.11.8.html)安装`Scala`(我们现在用的是 2.11.8 版本)
2.  从[这里](https://web.archive.org/web/20221205142529/http://www.scala-sbt.org/download.html)安装`sbt`构建工具(我们目前使用的是 0.13.11)

## 4。项目设置

现在让我们快速看一下建立一个工作 Lagom 系统的步骤。

### 4.1。SBT 建造

创建一个项目文件夹`lagom-hello-world`，后面是构建文件 **`build.sbt`** 。Lagom 系统通常由一组`sbt`构建组成，每个构建对应一组相关的服务:

```java
organization in ThisBuild := "com.baeldung"

scalaVersion in ThisBuild := "2.11.8"

lagomKafkaEnabled in ThisBuild := false

lazy val greetingApi = project("greeting-api")
  .settings(
    version := "1.0-SNAPSHOT",
    libraryDependencies ++= Seq(
      lagomJavadslApi
    )
  )

lazy val greetingImpl = project("greeting-impl")
  .enablePlugins(LagomJava)
  .settings(
    version := "1.0-SNAPSHOT",
    libraryDependencies ++= Seq(
      lagomJavadslPersistenceCassandra
    )
  )
  .dependsOn(greetingApi, weatherApi)

lazy val weatherApi = project("weather-api")
  .settings(
    version := "1.0-SNAPSHOT",
    libraryDependencies ++= Seq(
      lagomJavadslApi
    )
  )

lazy val weatherImpl = project("weather-impl")
  .enablePlugins(LagomJava)
  .settings(
    version := "1.0-SNAPSHOT"
  )
  .dependsOn(weatherApi)

def project(id: String) = Project(id, base = file(id))
```

首先，我们已经为当前项目指定了组织细节、`scala`版本和禁用`Kafka`。 **Lagom 遵循每个微服务有两个独立项目的惯例** : API 项目和实现项目。

API 项目包含实现所依赖的服务接口。

我们已经为相关的 Lagom 模块添加了依赖项，如`lagomJavadslApi`、`lagomJavadslPersistenceCassandra` ，分别用于在我们的微服务中使用 Lagom Java API 和在`Cassandra,`中存储与持久实体相关的事件。

此外，`greeting-impl`项目依赖于`weather-api`项目在问候用户时获取和提供天气统计信息。

通过创建一个包含 **`plugins.sbt`文件的插件文件夹，添加对 Lagom 插件的支持，**包含一个 Lagom 插件条目。它为构建、运行和部署我们的应用程序提供了所有必要的支持。

另外，如果我们在这个项目中使用 Eclipse IDE 的话,`sbteclipse`插件会很方便。下面的代码显示了两个插件的内容:

```java
addSbtPlugin("com.lightbend.lagom" % "lagom-sbt-plugin" % "1.3.1")
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "3.0.0")
```

创建 **`project/build.properties`** 文件并指定要使用的`sbt`版本:

```java
sbt.version=0.13.11
```

### 4.2。项目生成

从项目根目录运行`sbt`命令将生成以下项目模板:

1.  问候语-api
2.  问候-impl
3.  天气-空气污染指数
4.  天气影响

在我们开始实现微服务之前，让我们在每个项目中添加`src/main/java`和`src/main/java/resources`文件夹，以遵循类似 Maven 的项目目录布局。

另外，在`project-root/target/lagom-dynamic-projects`中生成了两个动态项目:

1.  lagom-内部-元项目-cassandra
2.  lagom-内部元项目服务定位器

这些项目由 Lagom 内部使用。

## 5。服务接口

在`greeting-api`项目中，我们指定了以下接口:

```java
public interface GreetingService extends Service {

    public ServiceCall<NotUsed, String> handleGreetFrom(String user);

    @Override
    default Descriptor descriptor() {
        return named("greetingservice")
          .withCalls(restCall(Method.GET, "/api/greeting/:fromUser",
            this::handleGreetFrom))
          .withAutoAcl(true);
    }
}
```

`GreetingService`暴露`handleGreetFrom()`来处理来自用户的问候请求。一个`ServiceCall` API 被用作这些方法的返回类型。`ServiceCall`带两个类型参数`Request`和`Response`。

`Request`参数是传入请求消息的类型，`Response`参数是传出响应消息的类型。

在上面的例子中，我们没有使用请求有效载荷，请求类型是`NotUsed`，而`Response`类型是`String`问候消息。

通过提供`Service.descriptor()`方法的默认实现，`GreetingService`还指定了到调用期间使用的实际传输的映射。名为`greetingservice`的服务被返回。

`handleGreetFrom()`服务调用使用 Rest 标识符映射:`GET`方法类型和路径标识符`/api/greeting/:fromUser`映射到`handleGreetFrom()`方法。查看[此链接](https://web.archive.org/web/20221205142529/https://www.lagomframework.com/documentation/1.3.x/java/ServiceDescriptors.html)了解服务标识符的更多详情。

同样，我们在`weather-api`项目中定义了`WeatherService`接口。`weatherStatsForToday()`方法和`descriptor()`方法几乎是不言自明的:

```java
public interface WeatherService extends Service {

    public ServiceCall<NotUsed, WeatherStats> weatherStatsForToday();

    @Override
    default Descriptor descriptor() {
        return named("weatherservice")
          .withCalls(
            restCall(Method.GET, "/api/weather",
              this::weatherStatsForToday))
          .withAutoAcl(true);
    }
};
```

`WeatherStats`定义为一个枚举，具有不同天气的样本值和随机查找以返回当天的天气预报:

```java
public enum WeatherStats {

    STATS_RAINY("Going to Rain, Take Umbrella"), 
    STATS_HUMID("Going to be very humid, Take Water");

    public static WeatherStats forToday() {
        return VALUES.get(RANDOM.nextInt(SIZE));
    }
}
```

## 6. **Lagom 持久性–事件采购**

简单地说，在一个利用`Event Sourcing`的系统中，我们将能够捕获所有的变化，作为一个接一个附加的不可变域事件。**当前状态是通过重放和处理事件得出的。**该操作本质上是函数式编程范例中已知的`foldLeft`操作。

事件源通过追加事件并避免更新和删除现有事件来帮助实现高写入性能。

现在让我们看看 greeting-impl 项目中的持久实体，`GreetingEntity`:

```java
public class GreetingEntity extends 
  PersistentEntity<GreetingCommand, GreetingEvent, GreetingState> {

      @Override
      public Behavior initialBehavior(
        Optional<GreetingState> snapshotState) {
            BehaviorBuilder b 
              = newBehaviorBuilder(new GreetingState("Hello "));

            b.setCommandHandler(
              ReceivedGreetingCommand.class,
              (cmd, ctx) -> {
                  String fromUser = cmd.getFromUser();
                  String currentGreeting = state().getMessage();
                  return ctx.thenPersist(
                    new ReceivedGreetingEvent(fromUser),
                    evt -> ctx.reply(
                      currentGreeting + fromUser + "!"));
              });

            b.setEventHandler(
              ReceivedGreetingEvent.class,
              evt -> state().withMessage("Hello Again "));

            return b.build();
      }
}
```

Lagom 提供了`[PersistentEntity<Command, Entity, Event>](https://web.archive.org/web/20221205142529/https://www.lagomframework.com/documentation/1.1.x/java/api/com/lightbend/lagom/javadsl/persistence/PersistentEntity.html)` API，用于通过`setCommandHandler()`方法处理`Command`类型的传入事件，并将状态更改作为`Event`类型的事件持久化。通过使用`setEventHandler()`方法将事件应用到当前状态来更新域对象状态。`The initialBehavior()`抽象方法定义了实体的`Behavior`。

在`initialBehavior()`中，我们构建原始的`GreetingState`“Hello”文本。然后我们可以定义一个`ReceivedGreetingCommand`命令处理程序——它产生一个`ReceivedGreetingEvent`事件并保存在事件日志中。

`GreetingState`被`ReceivedGreetingEvent`事件处理程序方法重新计算为“Hello Again”。**如前所述，我们没有调用 setters——相反，我们从正在处理的当前事件**中创建了一个新的`State`实例。

Lagom 遵循`GreetingCommand`和`GreetingEvent`接口的约定，将所有支持的命令和事件放在一起:

```java
public interface GreetingCommand extends Jsonable {

    @JsonDeserialize
    public class ReceivedGreetingCommand implements 
      GreetingCommand, 
      CompressedJsonable, 
      PersistentEntity.ReplyType<String> {      
          @JsonCreator
          public ReceivedGreetingCommand(String fromUser) {
              this.fromUser = Preconditions.checkNotNull(
                fromUser, "fromUser");
          }
    }
}
```

```java
public interface GreetingEvent extends Jsonable {
    class ReceivedGreetingEvent implements GreetingEvent {

        @JsonCreator
        public ReceivedGreetingEvent(String fromUser) {
            this.fromUser = fromUser;
        }
    }
}
```

## 7。服务实施

### 7.1。问候服务

```java
public class GreetingServiceImpl implements GreetingService {

    @Inject
    public GreetingServiceImpl(
      PersistentEntityRegistry persistentEntityRegistry, 
      WeatherService weatherService) {
          this.persistentEntityRegistry = persistentEntityRegistry;
          this.weatherService = weatherService;
          persistentEntityRegistry.register(GreetingEntity.class);
      }

    @Override
    public ServiceCall<NotUsed, String> handleGreetFrom(String user) {
        return request -> {
            PersistentEntityRef<GreetingCommand> ref
              = persistentEntityRegistry.refFor(
                GreetingEntity.class, user);
            CompletableFuture<String> greetingResponse 
              = ref.ask(new ReceivedGreetingCommand(user))
                .toCompletableFuture();
            CompletableFuture<WeatherStats> todaysWeatherInfo
              = (CompletableFuture<WeatherStats>) weatherService
                .weatherStatsForToday().invoke();

            try {
                return CompletableFuture.completedFuture(
                  greetingResponse.get() + " Today's weather stats: "
                    + todaysWeatherInfo.get().getMessage());
            } catch (InterruptedException | ExecutionException e) {
                return CompletableFuture.completedFuture(
                  "Sorry Some Error at our end, working on it");
            }
        };
    }
}
```

简单地说，我们使用`@Inject`(由`Guice`框架提供)注入`PersistentEntityRegistry`和`WeatherService`依赖项，并注册持久化的`GreetingEntity`。

`handleGreetFrom()`实现使用`CompletionStage` API 的`CompletableFuture`实现将`ReceivedGreetingCommand`发送给`GreetingEntity`处理并异步返回问候字符串。

类似地，我们异步调用`Weather`微服务来获取今天的天气统计。

最后，我们连接两个输出，并将最终结果返回给用户。

为了向 Lagom 注册服务描述符接口`GreetingService`的实现，让我们创建扩展`AbstractModule`并实现`ServiceGuiceSupport`的`GreetingServiceModule`类:

```java
public class GreetingServiceModule extends AbstractModule 
  implements ServiceGuiceSupport {

      @Override
      protected void configure() {
          bindServices(
            serviceBinding(GreetingService.class, GreetingServiceImpl.class));
          bindClient(WeatherService.class);
    }
} 
```

另外，Lagom 内部使用 Play 框架。因此，我们可以在`src/main/resources/application.conf`文件中将我们的模块添加到玩家的已启用模块列表中:

```java
play.modules.enabled
  += com.baeldung.lagom.helloworld.greeting.impl.GreetingServiceModule
```

### 7.2。气象服务

看完`GreetingServiceImpl`，`WeatherServiceImpl`非常简单明了:

```java
public class WeatherServiceImpl implements WeatherService {

    @Override
    public ServiceCall<NotUsed, WeatherStats> weatherStatsForToday() {
        return req -> 
          CompletableFuture.completedFuture(WeatherStats.forToday());
    }
}
```

我们按照与上面问候模块相同的步骤向 Lagom 注册天气模块:

```java
public class WeatherServiceModule 
  extends AbstractModule 
  implements ServiceGuiceSupport {

      @Override
      protected void configure() {
          bindServices(serviceBinding(
            WeatherService.class, 
            WeatherServiceImpl.class));
      }
}
```

另外，将天气模块注册到 Play 的框架启用模块列表中:

```java
play.modules.enabled
  += com.baeldung.lagom.helloworld.weather.impl.WeatherServiceModule
```

## 8。运行项目

Lagom 允许用一个命令运行任意数量的服务。

我们可以通过点击下面的命令来开始我们的项目:

```java
sbt lagom:runAll
```

这将启动嵌入式 `Service Locator`，嵌入式`Cassandra`，然后并行启动微服务。当代码改变时，同样的命令也重新加载我们的单个微服务，这样我们**就不必手动重启它们**。

我们可以专注于我们的逻辑，Lagom 处理编译和重新加载。一旦成功启动，我们将看到以下输出:

```java
................
[info] Cassandra server running at 127.0.0.1:4000
[info] Service locator is running at http://localhost:8000
[info] Service gateway is running at http://localhost:9000
[info] Service weather-impl listening for HTTP on 0:0:0:0:0:0:0:0:56231 and how the services interact via
[info] Service greeting-impl listening for HTTP on 0:0:0:0:0:0:0:0:49356
[info] (Services started, press enter to stop and go back to the console...)
```

一旦成功启动，我们可以发出 curl 请求问候:

```java
curl http://localhost:9000/api/greeting/Amit
```

我们将在控制台上看到以下输出:

```java
Hello Amit! Today's weather stats: Going to Rain, Take Umbrella
```

为现有用户运行相同的 curl 请求将会更改问候语:

```java
Hello Again Amit! Today's weather stats: Going to Rain, Take Umbrella
```

## 9。结论

在本文中，我们介绍了如何使用 Lagom 框架创建两个异步交互的微服务。

GitHub 项目中的[提供了本文的完整源代码和所有代码片段。](https://web.archive.org/web/20221205142529/https://github.com/eugenp/tutorials/tree/master/lagom)