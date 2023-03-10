# 用 Spark Java 框架构建 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spark-framework-rest-api>

## 1。简介

在本文中，我们将快速介绍一下 [Spark 框架](https://web.archive.org/web/20220815033605/http://sparkjava.com/)。Spark framework 是一个快速开发的 web 框架，灵感来自于用于 Ruby 的 Sinatra 框架，它是围绕 Java 8 Lambda 表达式哲学构建的，这使得它比用其他 Java 框架编写的大多数应用程序更简洁。

如果你想在用 Java 开发 web API 或微服务时有一种`Node.js`般的体验，这是一个不错的选择。使用 Spark，您可以用不到十行代码就准备好一个 REST API 来服务 JSON。

我们将从一个“Hello World”示例开始，然后是一个简单的 REST API。

## 2。Maven 依赖关系

### 2.1。Spark 框架

在您的`pom.xml`中包含以下 Maven 依赖项:

```java
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-core</artifactId>
    <version>2.5.4</version>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20220815033605/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.sparkjava%22%20AND%20a%3A%22spark-core%22) 上找到最新版本的 Spark。

### 2.2。Gson 库

在示例的不同地方，我们将使用 Gson 库进行 JSON 操作。要在您的项目中包含 Gson，请在您的`pom.xml`中包含这个依赖项:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.0</version>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20220815033605/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.code.gson%22%20AND%20a%3A%22gson%22) 上找到 Gson 的最新版本。

## 3。Spark 框架入门

让我们看看 Spark 应用程序的基本构建块，并演示一个快速的 web 服务。

### 3.1。路线

Spark Java 中的 Web 服务是建立在路由及其处理程序之上的。路线是 Spark 中必不可少的元素。根据[文件](https://web.archive.org/web/20220815033605/http://sparkjava.com/documentation.html#routes)，每条路线由三个简单的部分组成——一个`verb`、一个`path`和一个`callback`。

1.  **动词**是对应于 HTTP 方法的方法。动词方法包括:`get, post, put, delete, head, trace, connect,` 和 `options`
2.  **路径**(也称为路由模式)决定了路由应该监听哪个(哪些)URI 并为其提供响应
3.  **回调**是一个为给定动词和路径调用的处理函数，以便生成并返回对相应 HTTP 请求的响应。回调将请求对象和响应对象作为参数

这里我们展示了使用`get`动词的路线的基本结构:

```java
get("/your-route-path/", (request, response) -> {
    // your callback code
});
```

### 3.2。Hello World API

让我们创建一个简单的 web 服务，它有两条 GET 请求路径，并返回“Hello”消息作为响应。这些路由使用`get`方法，这是从类`spark.Spark`的静态导入:

```java
import static spark.Spark.*;

public class HelloWorldService {
    public static void main(String[] args) {

        get("/hello", (req, res)->"Hello, world");

        get("/hello/:name", (req,res)->{
            return "Hello, "+ req.params(":name");
        });
    }
}
```

`get`方法的第一个参数是路由的路径。第一条路由包含一条静态路径，仅代表一个 URI ( `“/hello”`)。

第二条路线的路径(`“/hello/:name”`)包含一个用于`“name”`参数的占位符，这通过在参数前加一个冒号(":")来表示。该路由将被调用以响应对 URIs 的 GET 请求，例如`“/hello/Joe”`和`“/hello/Mary”`。

`get`方法的第二个参数是一个 [lambda 表达式](/web/20220815033605/https://www.baeldung.com/java-8-lambda-expressions-tips),为这个框架赋予了函数式编程的味道。

lambda 表达式将请求和响应作为参数，并帮助返回响应。我们将把我们的控制器逻辑放在 REST API 路由的 lambda 表达式中，我们将在本教程后面看到。

### 3.3。测试 Hello World API

将类`HelloWorldService`作为普通的 Java 类运行后，您将能够使用上面用`get`方法定义的路由在默认端口`4567`上访问服务。

让我们看看第一条路由的请求和响应:

**请求:**

```java
GET http://localhost:4567/hello
```

**响应:**

```java
Hello, world
```

让我们测试第二条路线，在其路径中传递`name`参数:

**请求:**

```java
GET http://localhost:4567/hello/baeldung
```

**响应:**

```java
Hello, baeldung
```

看看文本`“baeldung”`在 URI 中的位置是如何用于匹配路由模式`“/hello/:name”`的——导致第二个路由的回调处理函数被调用。

## 4。设计 RESTful 服务

在本节中，我们将为下面的`User`实体设计一个简单的 REST web 服务:

```java
public class User {
    private String id;
    private String firstName;
    private String lastName;
    private String email;

    // constructors, getters and setters
}
```

### 4.1。路线

让我们列出构成 API 的路由:

*   获取/用户—获取所有用户的列表
*   GET /users/:id —获取具有给定 id 的用户
*   POST /users/:id —添加用户
*   PUT /users/:id —编辑特定用户
*   OPTIONS/users/:id-检查具有给定 id 的用户是否存在
*   DELETE /users/:id —删除特定用户

### 4.2。 **用户服务**

下面是为`User`实体声明 CRUD 操作的`UserService`接口:

```java
public interface UserService {

    public void addUser (User user);

    public Collection<User> getUsers ();
    public User getUser (String id);

    public User editUser (User user) 
      throws UserException;

    public void deleteUser (String id);

    public boolean userExist (String id);
}
```

出于演示的目的，我们在 GitHub 代码中提供了这个`UserService`接口的`Map`实现来模拟持久性。**您可以为自己的实现提供您选择的数据库和持久层。**

### 4.3。JSON 响应结构

下面是 REST 服务中使用的响应的 JSON 结构:

```java
{
    status: <STATUS>
    message: <TEXT-MESSAGE>
    data: <JSON-OBJECT>
}
```

`status`字段值可以是`SUCCESS`或`ERROR`。`data`字段将包含返回数据的 JSON 表示，比如一个`User`或一个`Users`的集合。

当没有数据返回时，或者如果`status`是`ERROR`，我们将填充`message`字段来传达错误或缺少返回数据的原因。

让我们用一个 Java 类来表示上面的 JSON 结构:

```java
public class StandardResponse {

    private StatusResponse status;
    private String message;
    private JsonElement data;

    public StandardResponse(StatusResponse status) {
        // ...
    }
    public StandardResponse(StatusResponse status, String message) {
        // ...
    }
    public StandardResponse(StatusResponse status, JsonElement data) {
        // ...
    }

    // getters and setters
}
```

其中`StatusResponse`是定义如下的`enum`:

```java
public enum StatusResponse {
    SUCCESS ("Success"),
    ERROR ("Error");

    private String status;       
    // constructors, getters
}
```

## 5。实现 RESTful 服务

现在让我们实现 REST API 的路由和处理程序。

### 5.1。创建控制器

下面的 Java 类包含我们的 API 的路由，包括动词和路径以及每个路由的处理程序的概要:

```java
public class SparkRestExample {
    public static void main(String[] args) {
        post("/users", (request, response) -> {
            //...
        });
        get("/users", (request, response) -> {
            //...
        });
        get("/users/:id", (request, response) -> {
            //...
        });
        put("/users/:id", (request, response) -> {
            //...
        });
        delete("/users/:id", (request, response) -> {
            //...
        });
        options("/users/:id", (request, response) -> {
            //...
        });
    }
}
```

在接下来的小节中，我们将展示每个路由处理程序的完整实现。

### 5.2。添加用户

下面是将添加一个`User`的`post`方法响应处理程序:

```java
post("/users", (request, response) -> {
    response.type("application/json");
    User user = new Gson().fromJson(request.body(), User.class);
    userService.addUser(user);

    return new Gson()
      .toJson(new StandardResponse(StatusResponse.SUCCESS));
});
```

**注意:**在这个例子中，`User`对象的 JSON 表示作为 POST 请求的原始体传递。

让我们测试一下路线:

**请求:**

```java
POST http://localhost:4567/users
{
    "id": "1012", 
    "email": "[[email protected]](/web/20220815033605/https://www.baeldung.com/cdn-cgi/l/email-protection)", 
    "firstName": "Mac",
    "lastName": "Mason1"
}
```

**响应:**

```java
{
    "status":"SUCCESS"
}
```

### 5.3。获取所有用户

下面是从`UserService`返回所有用户的`get`方法响应处理器:

```java
get("/users", (request, response) -> {
    response.type("application/json");
    return new Gson().toJson(
      new StandardResponse(StatusResponse.SUCCESS,new Gson()
        .toJsonTree(userService.getUsers())));
});
```

现在让我们测试路线:

**请求:**

```java
GET http://localhost:4567/users
```

**响应:**

```java
{
    "status":"SUCCESS",
    "data":[
        {
            "id":"1014",
            "firstName":"John",
            "lastName":"Miller",
            "email":"[[email protected]](/web/20220815033605/https://www.baeldung.com/cdn-cgi/l/email-protection)"
        },
        {
            "id":"1012",
            "firstName":"Mac",
            "lastName":"Mason1",
            "email":"[[email protected]](/web/20220815033605/https://www.baeldung.com/cdn-cgi/l/email-protection)"
        }
    ]
}
```

### 5.4。通过 Id 获取用户

下面是用给定的`id`返回一个`User`的`get`方法响应处理器:

```java
get("/users/:id", (request, response) -> {
    response.type("application/json");
    return new Gson().toJson(
      new StandardResponse(StatusResponse.SUCCESS,new Gson()
        .toJsonTree(userService.getUser(request.params(":id")))));
});
```

现在让我们测试路线:

**请求:**

```java
GET http://localhost:4567/users/1012
```

**响应:**

```java
{
    "status":"SUCCESS",
    "data":{
        "id":"1012",
        "firstName":"Mac",
        "lastName":"Mason1",
        "email":"[[email protected]](/web/20220815033605/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    }
}
```

### 5.5。编辑用户

下面是`put` 方法响应处理程序，它编辑在路由模式中提供了`id`的用户:

```java
put("/users/:id", (request, response) -> {
    response.type("application/json");
    User toEdit = new Gson().fromJson(request.body(), User.class);
    User editedUser = userService.editUser(toEdit);

    if (editedUser != null) {
        return new Gson().toJson(
          new StandardResponse(StatusResponse.SUCCESS,new Gson()
            .toJsonTree(editedUser)));
    } else {
        return new Gson().toJson(
          new StandardResponse(StatusResponse.ERROR,new Gson()
            .toJson("User not found or error in edit")));
    }
});
```

**注意:**在这个例子中，数据在 POST 请求的原始主体中作为 JSON 对象传递，该对象的属性名与要编辑的`User`对象的字段相匹配。

让我们测试一下路线:

**请求:**

```java
PUT http://localhost:4567/users/1012
{
    "lastName": "Mason"
}
```

**响应:**

```java
{
    "status":"SUCCESS",
    "data":{
        "id":"1012",
        "firstName":"Mac",
        "lastName":"Mason",
        "email":"[[email protected]](/web/20220815033605/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    }
}
```

### 5.6。删除用户

下面是`delete` 方法响应处理程序，它将删除带有给定`id`的`User`:

```java
delete("/users/:id", (request, response) -> {
    response.type("application/json");
    userService.deleteUser(request.params(":id"));
    return new Gson().toJson(
      new StandardResponse(StatusResponse.SUCCESS, "user deleted"));
});
```

现在，让我们测试路线:

**请求:**

```java
DELETE http://localhost:4567/users/1012
```

**响应:**

```java
{
    "status":"SUCCESS",
    "message":"user deleted"
}
```

### 5.7。检查用户是否存在

对于条件检查来说，`options`方法是一个很好的选择。下面是`options` 方法响应处理器，它将检查带有给定`id`的`User`是否存在:

```java
options("/users/:id", (request, response) -> {
    response.type("application/json");
    return new Gson().toJson(
      new StandardResponse(StatusResponse.SUCCESS, 
        (userService.userExist(
          request.params(":id"))) ? "User exists" : "User does not exists" ));
});
```

现在让我们测试路线:

**请求:**

```java
OPTIONS http://localhost:4567/users/1012
```

**响应:**

```java
{
    "status":"SUCCESS",
    "message":"User exists"
}
```

## 6。结论

在本文中，我们快速介绍了用于快速 web 开发的 Spark 框架。

该框架主要用于在 Java 中生成微服务。想要利用基于 JVM 库构建的库的具有 Java 知识的开发人员应该可以轻松地使用这个框架。

和往常一样，你可以在 [Github 项目](https://web.archive.org/web/20220815033605/https://github.com/eugenp/tutorials/tree/master/spark-java)中找到本教程的所有源代码。