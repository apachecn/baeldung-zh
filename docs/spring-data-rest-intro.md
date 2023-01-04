# Spring Data REST 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-rest-intro>

## 1。概述

本文将解释 [Spring Data REST](https://web.archive.org/web/20220617075814/https://spring.io/projects/spring-data-rest) 的基础知识，并展示如何使用它来构建一个简单的 REST API。

总的来说，Spring Data REST 是建立在 Spring Data 项目之上的，它使得构建连接到 Spring 数据仓库的超媒体驱动的 REST web 服务变得很容易——所有这些都使用 HAL 作为驱动超媒体类型。

它消除了通常与此类任务相关的大量手工工作，并使实现 web 应用程序的基本 CRUD 功能变得非常简单。

## 2。Maven 依赖关系

我们的简单应用程序需要以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId
    <artifactId>spring-boot-starter-data-rest</artifactId></dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

我们决定在这个例子中使用 Spring Boot，但是经典的 Spring 也可以。我们还选择使用 H2 嵌入式数据库，以避免任何额外的设置，但该示例可以应用于任何数据库。

## 3。编写应用程序

我们将首先编写一个域对象来表示我们网站的用户:

```java
@Entity
public class WebsiteUser {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String name;
    private String email;

    // standard getters and setters
}
```

每个用户都有一个名字和一个电子邮件，以及一个自动生成的 id。现在我们可以编写一个简单的存储库:

```java
@RepositoryRestResource(collectionResourceRel = "users", path = "users")
public interface UserRepository extends PagingAndSortingRepository<WebsiteUser, Long> {
    List<WebsiteUser> findByName(@Param("name") String name);
}
```

这是一个允许你用`WebsiteUser`对象执行各种操作的界面。我们还定义了一个自定义查询，它将根据给定的名称提供一个用户列表。

`@RepositoryRestResource`注释是可选的，用于定制 REST 端点。如果我们决定省略它，Spring 会自动在“`/websiteUsers`”而不是“`/users`”创建一个端点。

最后，我们将编写一个标准的 **Spring Boot 主类来初始化应用程序**:

```java
@SpringBootApplication
public class SpringDataRestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringDataRestApplication.class, args);
    }
}
```

就是这样！我们现在有了一个全功能的 REST API。让我们来看看它的运行情况。

## 4。访问 REST API

如果我们运行应用程序并在浏览器中转至 [http://localhost:8080/](https://web.archive.org/web/20220617075814/http://localhost:8080/) ，我们将收到以下 JSON:

```java
{
  "_links" : {
    "users" : {
      "href" : "http://localhost:8080/users{?page,size,sort}",
      "templated" : true
    },
    "profile" : {
      "href" : "http://localhost:8080/profile"
    }
  }
}
```

如您所见，有一个“`/users`”端点可用，并且它已经有了“`?page`”、“`?size`”和“`?sort`”选项。

还有一个标准的“`/profile`”端点，它提供应用程序元数据。值得注意的是，响应是以遵循 REST 架构风格的约束的方式构造的。具体来说，它提供了统一的接口和自描述消息。这意味着每条消息都包含足够的信息来描述如何处理该消息。

我们的应用程序中还没有用户，所以转到[http://localhost:8080/users](https://web.archive.org/web/20220617075814/http://localhost:8080/users)只会显示一个空的用户列表。让我们使用 curl 来添加一个用户。

```java
$ curl -i -X POST -H "Content-Type:application/json" -d '{  "name" : "Test", \ 
"email" : "[[email protected]](/web/20220617075814/https://www.baeldung.com/cdn-cgi/l/email-protection)" }' http://localhost:8080/users
{
  "name" : "test",
  "email" : "[[email protected]](/web/20220617075814/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users/1"
    },
    "websiteUser" : {
      "href" : "http://localhost:8080/users/1"
    }
  }
}
```

让我们看看响应头:

```java
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/users/1
Content-Type: application/hal+json;charset=UTF-8
Transfer-Encoding: chunked
```

您会注意到返回的内容类型是“`application/hal+json`”。 [HAL](https://web.archive.org/web/20220617075814/http://stateless.co/hal_specification.html) 是一种简单的格式，为 API 中的资源之间的超链接提供了一种一致且简单的方式。这个头还自动包含了`Location` 头，这是我们可以用来访问新创建的用户的地址。

我们现在可以通过[http://localhost:8080/users/1](https://web.archive.org/web/20220617075814/http://localhost:8080/users/1)访问该用户

```java
{
  "name" : "test",
  "email" : "te[[email protected]](/web/20220617075814/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users/1"
    },
    "websiteUser" : {
      "href" : "http://localhost:8080/users/1"
    }
  }
}
```

您还可以使用 curl 或任何其他 REST 客户端来发出 PUT、PATCH 和 DELETE 请求。还需要注意的是，Spring Data REST 自动遵循 HATEOAS 的原则。HATEOAS 是 REST 架构风格的约束之一，这意味着应该使用超文本来找到通过 API 的方法。

最后，让我们尝试访问我们之前编写的定制查询，并找到所有名为“test”的用户。这是通过进入[http://localhost:8080/users/search/find by name？名称=测试](https://web.archive.org/web/20220617075814/http://localhost:8080/users/search/findByName?name=test)

```java
{
  "_embedded" : {
    "users" : [ {
      "name" : "test",
      "email" : "[[email protected]](/web/20220617075814/https://www.baeldung.com/cdn-cgi/l/email-protection)",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/users/1"
        },
        "websiteUser" : {
          "href" : "http://localhost:8080/users/1"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users/search/findByName?name=test"
    }
  }
}
```

## 5。结论

本教程演示了使用 Spring Data REST 创建简单 REST API 的基础。本文中使用的例子可以在链接的 [GitHub 项目](https://web.archive.org/web/20220617075814/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)中找到。