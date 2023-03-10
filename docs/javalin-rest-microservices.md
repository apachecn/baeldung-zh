# 用 Javalin 创建 REST 微服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javalin-rest-microservices>

## 1。简介

Javalin 是一个为 Java 和 Kotlin 编写的轻量级 web 框架。它是在 Jetty web 服务器之上编写的，这使得它具有很高的性能。Javalin 是严格模仿 [koa.js](https://web.archive.org/web/20220627185513/http://koajs.com/) 的，这意味着它是从头开始编写的，易于理解和构建。

在本教程中，我们将逐步使用这个轻量级框架构建一个基本的 REST 微服务。

## 2。添加依赖关系

要创建一个基本的应用程序，我们只需要一个依赖项——Java Lin 本身:

```java
<dependency>
    <groupId>io.javalin</groupId>
    <artifactId>javalin</artifactId>
    <version>1.6.1</version>
</dependency>
```

当前版本可以在这里找到[。](https://web.archive.org/web/20220627185513/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.javalin%22%20AND%20a%3A%22javalin%22)

## 3。设置 Javalin

Javalin 使设置一个基本的应用程序变得容易。我们将从定义我们的主类开始，并建立一个简单的“Hello World”应用程序。

让我们在基础包中创建一个名为`JavalinApp.java`的新文件。

在这个文件中，我们创建一个 main 方法，并添加以下内容来设置一个基本的应用程序:

```java
Javalin app = Javalin.create()
  .port(7000)
  .start();
app.get("/hello", ctx -> ctx.html("Hello, Javalin!"));
```

我们正在创建一个新的 Javalin 实例，让它监听端口 7000，然后启动应用程序。

我们还设置了第一个端点，在/ `hello`端点监听`GET`请求。

让我们运行这个应用程序并访问[http://localhost:7000/hello](https://web.archive.org/web/20220627185513/http://localhost:7000/hello)来查看结果。

## 4。创造一个`UserController`

一个“Hello World”的例子对于引入一个主题来说是很棒的，但是对于一个真正的应用程序来说并没有什么好处。现在让我们来看一个更现实的 Javalin 用例。

首先，我们需要创建一个我们正在处理的对象的模型。我们首先在根项目下创建一个名为`user`的包。

然后，我们添加一个新的`User`类:

```java
public class User {
    public final int id;
    public final String name;

    // constructors
}
```

此外，我们需要设置我们的数据访问对象(DAO)。在这个例子中，我们将使用一个内存对象来存储我们的用户。

我们在打包的`user`中创建了一个名为`UserDao.java:`的新类

```java
class UserDao {

    private List<User> users = Arrays.asList(
      new User(0, "Steve Rogers"),
      new User(1, "Tony Stark"),
      new User(2, "Carol Danvers")
    );

    private static UserDao userDao = null;

    private UserDao() {
    }

    static UserDao instance() {
        if (userDao == null) {
            userDao = new UserDao();
        }
        return userDao;
    }

    Optional<User> getUserById(int id) {
        return users.stream()
          .filter(u -> u.id == id)
          .findAny();
    }

    Iterable<String> getAllUsernames() {
        return users.stream()
          .map(user -> user.name)
          .collect(Collectors.toList());
    }
}
```

将我们的 DAO 实现为单例使得它在示例中更容易使用。如果我们愿意，我们也可以将它声明为主类的静态成员，或者从 Guice 这样的库中使用依赖注入。

最后，我们想创建我们的控制器类。Javalin 允许我们在声明路由处理程序时非常灵活，所以这只是定义它们的一种方式。

我们在`user`包中创建了一个名为`UserController.java`的新类:

```java
public class UserController {
    public static Handler fetchAllUsernames = ctx -> {
        UserDao dao = UserDao.instance();
        Iterable<String> allUsers = dao.getAllUsernames();
        ctx.json(allUsers);
    };

    public static Handler fetchById = ctx -> {
        int id = Integer.parseInt(Objects.requireNonNull(ctx.param("id")));
        UserDao dao = UserDao.instance();
        User user = dao.getUserById(id);
        if (user == null) {
            ctx.html("Not Found");
        } else {
            ctx.json(user);
        }
    };
}
```

通过将处理程序声明为静态，我们确保控制器本身不持有任何状态。但是，在更复杂的应用程序中，我们可能希望存储请求之间的状态，在这种情况下，我们需要删除 static 修饰符。

还要注意，静态方法的单元测试更难，所以如果我们想要那种级别的测试，我们需要使用非静态方法。

## 5。添加路线

我们现在有多种方式从模型中获取数据。最后一步是通过 REST 端点公开这些数据。我们需要在主应用程序中注册两条新的路由。

让我们将它们添加到我们的主应用程序类中:

```java
app.get("/users", UserController.fetchAllUsernames);
app.get("/users/:id", UserController.fetchById);
```

在编译和运行应用程序之后，我们可以向每个新的端点发出请求。调用[http://localhost:7000/users](https://web.archive.org/web/20220627185513/http://localhost:7000/users)将列出所有用户，调用[http://localhost:7000/users/0](https://web.archive.org/web/20220627185513/http://localhost:7000/users/0)将获得 id 为 0 的单用户 JSON 对象。我们现在有一个微服务，可以让我们检索`User`数据。

## 6。延伸路线

检索数据是大多数微服务的重要任务。

然而，我们还需要能够在我们的数据存储中存储数据。Javalin 提供了构建服务所需的全套路径处理程序。

我们在上面看到了一个`GET`的例子，但是`PATCH, POST, DELETE, `和`PUT` 也是可能的。

此外，如果我们包含 Jackson 作为依赖项，我们可以将 JSON 请求体自动解析到我们的模型类中。例如:

```java
app.post("/") { ctx ->
  User user = ctx.bodyAsClass(User.class);
}
```

将允许我们从请求体中获取 JSON `User`对象，并将其转换为`User`模型对象。

## 7。结论

我们可以结合所有这些技术来打造我们的微服务。

在本文中，我们看到了如何设置 Javalin 并构建一个简单的应用程序。我们还讨论了如何使用不同的 HTTP 方法类型让客户端与我们的服务进行交互。

关于如何使用 Javalin 的更多高级示例，请务必查看[文档](https://web.archive.org/web/20220627185513/https://javalin.io/documentation)。

同样，和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627185513/https://github.com/eugenp/tutorials/tree/master/libraries-http)