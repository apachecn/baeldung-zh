# Akka 简介 HTTP

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/akka-http>

## 1.概观

在本教程中，在 Akka 的[演员](/web/20220926194208/https://www.baeldung.com/akka-actors-java) & [流](/web/20220926194208/https://www.baeldung.com/akka-streams)模型的帮助下，我们将学习如何设置 Akka 来创建一个提供基本 CRUD 操作的 HTTP API。

## 2.Maven 依赖性

首先，让我们看看开始使用 Akka HTTP:

```
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-http_2.12</artifactId>
    <version>10.0.11</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-stream_2.12</artifactId>
    <version>2.5.11</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-http-jackson_2.12</artifactId>
    <version>10.0.11</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-http-testkit_2.12</artifactId>
    <version>10.0.11</version>
    <scope>test</scope>
</dependency>
```

当然，我们可以在 Maven Central 上找到这些 Akka 库的最新版本。

## 3.创建演员

例如，我们将构建一个允许我们管理用户资源的 HTTP API。API 将支持两种操作:

*   创建新用户
*   加载现有用户

在我们提供 HTTP API 之前，**我们需要实现一个 actor 来提供我们需要的操作:**

```
class UserActor extends AbstractActor {

  private UserService userService = new UserService();

  static Props props() {
    return Props.create(UserActor.class);
  }

  @Override
  public Receive createReceive() {
    return receiveBuilder()
      .match(CreateUserMessage.class, handleCreateUser())
      .match(GetUserMessage.class, handleGetUser())
      .build();
  }

  private FI.UnitApply<CreateUserMessage> handleCreateUser() {
    return createUserMessage -> {
      userService.createUser(createUserMessage.getUser());
      sender()
        .tell(new ActionPerformed(
           String.format("User %s created.", createUserMessage.getUser().getName())), getSelf());
    };
  }

  private FI.UnitApply<GetUserMessage> handleGetUser() {
    return getUserMessage -> {
      sender().tell(userService.getUser(getUserMessage.getUserId()), getSelf());
    };
  }
}
```

基本上，我们扩展了`AbstractActor`类并实现了它的`createReceive()`方法。

在`createReceive()`中，我们**将传入的消息类型**映射到处理相应类型消息的方法。

**消息类型是简单的可序列化容器类，带有一些描述特定操作的字段**。`GetUserMessage`并有一个单独的字段`userId`来标识要加载的用户。`CreateUserMessage`包含一个`User`对象，其中包含我们创建新用户所需的用户数据。

稍后，我们将看到如何将传入的 HTTP 请求转换成这些消息。

最终，我们将所有消息委托给一个`UserService`实例，该实例提供了管理持久用户对象所必需的业务逻辑。

另外，请注意`props()`方法。**虽然`props() `方法不是扩展** `**AbstractActor**,` 所必需的，但它会在稍后创建`ActorSystem`时派上用场。

关于演员更深入的讨论，看看我们的[对 Akka 演员的介绍](/web/20220926194208/https://www.baeldung.com/akka-actors-java)。

## 4.定义 HTTP 路由

有了为我们做实际工作的 actor，我们剩下要做的就是提供一个 HTTP API，将传入的 HTTP 请求委托给我们的 actor。

Akka 使用路由的概念来描述 HTTP API。对于每个操作，我们需要一条路线。

为了创建 HTTP 服务器，我们扩展了框架类`HttpApp`并实现了`routes`方法:

```
class UserServer extends HttpApp {

  private final ActorRef userActor;

  Timeout timeout = new Timeout(Duration.create(5, TimeUnit.SECONDS));

  UserServer(ActorRef userActor) {
    this.userActor = userActor;
  }

  @Override
  public Route routes() {
    return path("users", this::postUser)
      .orElse(path(segment("users").slash(longSegment()), id -> route(getUser(id))));
  }

  private Route getUser(Long id) {
    return get(() -> {
      CompletionStage<Optional<User>> user = 
        PatternsCS.ask(userActor, new GetUserMessage(id), timeout)
          .thenApply(obj -> (Optional<User>) obj);

      return onSuccess(() -> user, performed -> {
        if (performed.isPresent())
          return complete(StatusCodes.OK, performed.get(), Jackson.marshaller());
        else
          return complete(StatusCodes.NOT_FOUND);
      });
    });
  }

  private Route postUser() {
    return route(post(() -> entity(Jackson.unmarshaller(User.class), user -> {
      CompletionStage<ActionPerformed> userCreated = 
        PatternsCS.ask(userActor, new CreateUserMessage(user), timeout)
          .thenApply(obj -> (ActionPerformed) obj);

      return onSuccess(() -> userCreated, performed -> {
        return complete(StatusCodes.CREATED, performed, Jackson.marshaller());
      });
    })));
  }
} 
```

现在，这里有相当多的样板文件，但是请注意，我们遵循与之前相同的**映射操作模式，这次是路由。**我们来稍微分解一下。

在`getUser()`中，我们简单地将传入的用户 id 包装在类型为`GetUserMessage`的消息中，并将该消息转发给我们的`userActor`。

一旦参与者处理了消息，就会调用`onSuccess`处理程序，其中我们通过发送具有特定 HTTP 状态和特定 JSON 主体的响应来`complete`HTTP 请求。我们使用 [Jackson](/web/20220926194208/https://www.baeldung.com/jackson-object-mapper-tutorial) 编组器将演员给出的答案序列化为 JSON 字符串。

在`postUser()`中，我们做的事情略有不同，因为我们期望 HTTP 请求中有一个 JSON 主体。我们使用`entity()`方法将传入的 JSON 主体映射到一个`User`对象，然后将其封装到一个`CreateUserMessage`中并传递给我们的 actor。同样，我们使用 Jackson 在 Java 和 JSON 之间进行映射，反之亦然。

**因为`HttpApp`期望我们提供一个单独的`Route`对象，我们在`routes`方法中将两个路径合并成一个。**这里，我们使用`path`指令来最终提供我们的 API 应该可用的 URL 路径。

我们将由`postUser()`提供的路由绑定到路径`/users`。如果传入的请求不是 POST 请求，Akka 将自动进入`orElse`分支，并期望路径为`/users/<id>` ，HTTP 方法为 GET。

如果 HTTP 方法是 GET，请求将被转发到`getUser()`路由。如果用户不存在，Akka 将返回 HTTP 状态 404(未找到)。如果方法既不是 POST 也不是 GET，Akka 将返回 HTTP 状态 405(不允许使用方法)。

关于如何用 Akka 定义 HTTP 路由的更多信息，请看一下 [Akka 文档](https://web.archive.org/web/20220926194208/https://doc.akka.io/docs/akka-http/current/routing-dsl/routes.html)。

## 5.启动服务器

一旦我们创建了如上所示的`HttpApp`实现，我们就可以用几行代码启动我们的 HTTP 服务器:

```
public static void main(String[] args) throws Exception {
  ActorSystem system = ActorSystem.create("userServer");
  ActorRef userActor = system.actorOf(UserActor.props(), "userActor");
  UserServer server = new UserServer(userActor);
  server.startServer("localhost", 8080, system);
}
```

**我们简单地创建一个只有一个类型为`UserActor`的角色的`ActorSystem`，并在`localhost`上启动服务器。**

## 6.结论

在本文中，我们学习了 Akka HTTP 的基础知识，并通过一个示例展示了如何设置 HTTP 服务器并公开端点来创建和加载资源，类似于 REST API。

像往常一样，这里展示的源代码可以在 [GitHub](https://web.archive.org/web/20220926194208/https://github.com/eugenp/tutorials/tree/master/akka-modules/akka-http) 上找到。