# GraphQL 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/graphql>

## 1。概述

GraphQL 是一种查询语言，由脸书创建，目的是基于直观和灵活的语法构建客户端应用程序，用于描述数据需求和交互。

传统 REST 调用的主要挑战之一是客户端无法请求定制的(有限或扩展的)数据集。在大多数情况下，一旦客户机向服务器请求信息，它要么获得所有字段，要么不获得任何字段。

另一个困难是工作和维护多个端点。随着平台的发展，数量也会增加。因此，客户端经常需要从不同的端点请求数据。

在构建 GraphQL 服务器时，只需要有一个 URL 用于所有的数据获取和变更。因此，客户机可以通过向服务器发送一个查询字符串来请求一组数据，该字符串描述了它们想要的内容。

## 2。基本图表命名法

让我们来看看 GraphQL 的基本术语。

*   **查询:**是向 GraphQL 服务器请求的只读操作
*   **突变:**是对 GraphQL 服务器的读写操作请求
*   **Resolver:** 在 GraphQL 中，`Resolver`负责映射运行在负责处理请求的后端上的操作和代码。它类似于 RESTFul 应用程序中的 MVC 后端
*   **类型:** A `Type`定义了可以从 GraphQL 服务器返回的响应数据的形状，包括作为其他`Types`的边的字段
*   **输入:**类似于`Type,`，但是定义了发送到 GraphQL 服务器的输入数据的形状
*   **标量:**为原语`Type`，如`String`、`Int`、`Boolean`、`Float`等
*   接口:一个接口将存储字段的名称和它们的参数，所以 GraphQL 对象可以继承它，确保特定字段的使用
*   **模式:**在 GraphQL 中，模式管理查询和变异，定义了允许在 GraphQL 服务器中执行什么

### 2.1。模式加载

将模式加载到 GraphQL 服务器有两种方式:

1.  通过使用 GraphQL 的接口定义语言(IDL)
2.  通过使用一种受支持的编程语言

让我们用 IDL 来演示一个例子:

```
type User {
    firstName: String
}
```

现在，一个使用 Java 代码的模式定义的例子:

```
GraphQLObjectType userType = newObject()
  .name("User")  
  .field(newFieldDefinition()
    .name("firstName")
    .type(GraphQLString))
  .build();
```

## 3。接口定义语言

接口定义语言(IDL)或模式定义语言(SDL)是指定 GraphQL 模式的最简洁的方式。该语法定义良好，将被正式的 GraphQL 规范采用。

例如，让我们为用户/电子邮件创建一个 GraphQL 模式，可以这样指定:

```
schema {
    query: QueryType
}

enum Gender {
    MALE
    FEMALE
}

type User {
    id: String!
    firstName: String!
    lastName: String!
    createdAt: DateTime!
    age: Int! @default(value: 0)
    gender: [Gender]!
    emails: [Email!]! @relation(name: "Emails")
}

type Email {
    id: String!
    email: String!
    default: Int! @default(value: 0)
    user: User @relation(name: "Emails")
}
```

## 4。GraphQL-java

GraphQL-java 是基于规范和 [JavaScript 参考实现的实现。](https://web.archive.org/web/20221128111009/https://github.com/graphql/graphql-js)注意，至少需要 Java 8 才能正常运行。

### 4.1。GraphQL-java 注释

GraphQL 还使得使用 [Java 注释](https://web.archive.org/web/20221128111009/https://github.com/graphql-java/graphql-java-annotations)生成其模式定义成为可能，而无需使用传统 IDL 方法创建的所有样板代码。

### 4.2。依赖性

为了创建我们的示例，让我们首先开始导入依赖于 [Graphql-java-annotations](https://web.archive.org/web/20221128111009/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.graphql-java%22%20AND%20a%3A%22graphql-java-annotations%22) 模块的所需依赖项:

```
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-annotations</artifactId>
    <version>3.0.3</version>
</dependency>
```

我们还实现了一个 HTTP 库来简化应用程序的设置。我们将使用 [Ratpack](https://web.archive.org/web/20221128111009/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.ratpack%22%20AND%20a%3A%22ratpack-core%22) (虽然它也可以用 Vert.x、Spark、Dropwizard、Spring Boot 等实现。).

让我们也导入 Ratpack 依赖关系:

```
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.4.6</version>
</dependency>
```

### 4.3。实施

让我们创建一个例子:一个简单的 API，为用户提供一个“CRUDL”(创建、检索、更新、删除和列表)。首先，让我们创建我们的`User` POJO:

```
@GraphQLName("user")
public class User {

    @GraphQLField
    private Long id;

    @GraphQLField
    private String name;

    @GraphQLField
    private String email;

    // getters, setters, constructors, and helper methods omitted
}
```

在这个 POJO 中，我们可以看到`@GraphQLName(“user”)` 注释，这表明这个类是由 GraphQL 与每个用`@GraphQLField.`注释的字段一起映射的

接下来，我们将创建`UserHandler`类。这个类从选择的 HTTP 连接器库(在我们的例子中，Ratpack)继承了一个处理程序方法，它将管理和调用 GraphQL 的`Resolver` 特性。因此，将请求(JSON 有效负载)重定向到适当的查询或变异操作:

```
@Override
public void handle(Context context) throws Exception {
    context.parse(Map.class)
      .then(payload -> {
          Map<String, Object> parameters = (Map<String, Object>)
            payload.get("parameters");
          ExecutionResult executionResult = graphql
            .execute(payload.get(SchemaUtils.QUERY)
              .toString(), null, this, parameters);
          Map<String, Object> result = new LinkedHashMap<>();
          if (executionResult.getErrors().isEmpty()) {
              result.put(SchemaUtils.DATA, executionResult.getData());
          } else {
              result.put(SchemaUtils.ERRORS, executionResult.getErrors());
              LOGGER.warning("Errors: " + executionResult.getErrors());
          }
          context.render(json(result));
      });
}
```

现在，将支持查询操作的类，即`UserQuery.` 如前所述，所有从服务器向客户端检索数据的方法都由该类管理:

```
@GraphQLName("query")
public class UserQuery {

    @GraphQLField
    public static User retrieveUser(
     DataFetchingEnvironment env,
      @NotNull @GraphQLName("id") String id) {
        // return user
    }

    @GraphQLField
    public static List<User> listUsers(DataFetchingEnvironment env) {
        // return list of users
    }
}
```

类似于`UserQuery,`,现在我们创建`UserMutation,` ,它将管理所有旨在改变存储在服务器端的某些给定数据的操作:

```
@GraphQLName("mutation")
public class UserMutation {

    @GraphQLField
    public static User createUser(
      DataFetchingEnvironment env,
      @NotNull @GraphQLName("name") String name,
      @NotNull @GraphQLName("email") String email) {
      //create user information
    }
}
```

值得注意的是`UserQuery` 和`UserMutation` 类中的注释:`@GraphQLName(“query”)`和`@GraphQLName(“mutation”).` 这些注释分别用于定义查询和变异操作。

由于 GraphQL-java 服务器能够运行查询和变异操作，我们可以使用以下 JSON 有效负载来测试客户机对服务器的请求:

*   **对于创建操作:**

```
{
    "query": "mutation($name: String! $email: String!){
       createUser (name: $name email: $email) { id name email age } }",
    "parameters": {
        "name": "John",
        "email": "[[email protected]](/web/20221128111009/https://www.baeldung.com/cdn-cgi/l/email-protection)"
     }
} 
```

作为服务器对此操作的响应:

```
{
    "data": {
        "createUser": {
            "id": 1,
            "name": "John",
            "email": "[[email protected]](/web/20221128111009/https://www.baeldung.com/cdn-cgi/l/email-protection)"
        }
    } 
}
```

*   **对于检索操作:**

```
{
    "query": "query($id: String!){ retrieveUser (id: $id) {name email} }",
    "parameters": {
        "id": 1
    }
}
```

作为服务器对此操作的响应:

```
{
    "data": {
        "retrieveUser": {
            "name": "John",
            "email": "[[email protected]](/web/20221128111009/https://www.baeldung.com/cdn-cgi/l/email-protection)"
        }
    }
}
```

GraphQL 提供了客户端可以定制响应的特性。因此，在作为示例的最后一个检索操作中，我们可以只返回电子邮件，而不是返回姓名和电子邮件:

```
{
    "query": "query($id: String!){ retrieveUser (id: $id) {email} }",
    "parameters": {
        "id": 1
    }
}
```

因此，从 GraphQL 服务器返回的信息将只返回请求的数据:

```
{
    "data": {
        "retrieveUser": {
            "email": "[[email protected]](/web/20221128111009/https://www.baeldung.com/cdn-cgi/l/email-protection)"
        }
    }
}
```

## 5。结论

作为 REST APIs 的替代方法，GraphQL 是最小化客户机/服务器之间复杂性的一种简单且颇具吸引力的方法。

和往常一样，这个例子可以在我们的 [GitHub 库](https://web.archive.org/web/20221128111009/https://github.com/eugenp/tutorials/tree/master/graphql-modules/graphql-java)中找到。