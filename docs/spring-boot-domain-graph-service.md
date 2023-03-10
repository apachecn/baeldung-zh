# 领域图服务(DGS)框架介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-domain-graph-service>

## 1.概观

在过去几年中，关于客户机/服务器通信的最重要的范式变化之一是 [GraphQL](https://web.archive.org/web/20220707143855/https://graphql.org/) ，一种开源查询语言，以及用于操作 API 的运行时。我们可以使用它来请求我们需要的确切数据，从而限制我们需要的请求数量。

网飞创建了一个域图服务框架(DGS)服务器框架来使事情变得更加简单。在这个快速教程中，我们将介绍 DGS 框架的关键特性。我们将看到如何将这个框架添加到我们的应用程序中，并检查它的基本注释是如何工作的。要了解更多关于 GraphQL 本身的信息，请查看我们的[graph QL 简介文章](/web/20220707143855/https://www.baeldung.com/graphql)。

## 2.领域图服务框架

[网飞 DGS](https://web.archive.org/web/20220707143855/https://netflix.github.io/dgs/) (域图服务)是一个用 Kotlin 写的基于 Spring Boot 的 GraphQL 服务器框架。除了 Spring 框架之外，它被设计成具有最小的外部依赖性。

网飞 uses 框架使用一个基于注释的 GraphQL Java 库，构建在 Spring Boot 之上。除了基于注释的编程模型，它还提供了几个有用的特性。**它允许从 GraphQL 模式生成源代码。** **让我们总结一些关键特征**:

*   基于注释的 Spring Boot 编程模型
*   将查询测试编写为单元测试的测试框架
*   从模式创建类型的 Gradle/Maven 代码生成插件
*   与 GraphQL 联邦轻松集成
*   与 Spring Security 的集成
*   GraphQL 订阅(WebSockets 和 SSE)
*   文件上传
*   错误处理
*   许多扩展点

## 3.配置

首先，由于 DGS 框架基于 Spring Boot，[让我们创建一个 Spring Boot 应用](/web/20220707143855/https://www.baeldung.com/spring-boot-start)。然后，让我们将 [DGS 依赖项](https://web.archive.org/web/20220707143855/https://search.maven.org/search?q=g:com.netflix.graphql.dgs%20a:graphql-dgs)添加到我们的项目中:

```java
<dependency>
    <groupId>com.netflix.graphql.dgs</groupId>
    <artifactId>graphql-dgs-spring-boot-starter</artifactId>
    <version>4.9.16</version>
</dependency>
```

## 4\. (计划或理论的)纲要

### 4.1.发展途径

**DGS 框架支持两种开发方法——模式优先和代码优先。**但是推荐的方法是模式优先，主要是因为它更容易跟上数据模型的变化。Schema-first 表示我们首先为 GraphQL 服务定义模式，然后通过匹配模式中的定义来实现代码。默认情况下，框架会选择 `src/main/resources/schema`文件夹中的任何模式文件。

### 4.2.履行

让我们使用[模式定义语言(SDL)](https://web.archive.org/web/20220707143855/https://www.howtographql.com/basics/2-core-concepts/) 为我们的示例应用程序创建一个简单的 GraphQL 模式:

```java
type Query {
    albums(titleFilter: String): [Album]
}

type Album {
    title: String
    artist: String
    recordNo: Int
}
```

该模式允许查询相册列表，并且可选地通过`title`进行过滤。

## 5.基本注释

让我们从创建一个对应于我们的模式的`Album`类开始:

```java
public class Album {
    private final String title;
    private final String artist;
    private final Integer recordNo;

    public Album(String title, String artist, Integer recordNo) {
        this.title = title;
        this.recordNo = recordNo;
        this.artist = artist;
    }

    // standard getters
}
```

### 5.1.数据提取器

数据提取器负责为查询返回数据。**`@DgsQuery, @DgsMutation,`和`@DgsSubscription`** **注释是在`Query, Mutation`和`Subscription`类型上定义数据提取器的简写。**所有提及的注释等同于`@DgsData` 注释。我们可以在 Java 方法上使用这些注释之一，使该方法成为一个数据提取器，并定义一个带参数的类型。

### 5.2.履行

**因此，为了定义 DGS 数据提取器，我们需要在`@DgsComponent`类**中创建一个查询方法。在我们的例子中，我们想要查询一个列表`Albums`，所以让我们用`@DgsQuery`标记这个方法:

```java
private final List<Album> albums = Arrays.asList(
  new Album("Rumours", "Fleetwood Mac", 20),
  new Album("What's Going On", "Marvin Gaye", 10), 
  new Album("Pet Sounds", "The Beach Boys", 12)
  );

@DgsQuery
public List<Album> albums(@InputArgument String titleFilter) {
    if (titleFilter == null) {
        return albums;
    }
    return albums.stream()
      .filter(s -> s.getTitle().contains(titleFilter))
      .collect(Collectors.toList());
}
```

我们还用注释`@InputArgument`标记了该方法的参数。该注释将使用方法参数的名称来匹配查询中发送的输入参数的名称。

## 6.代码生成插件

DGS 还附带了一个 code-gen 插件，用于从 GraphQL 模式生成 Java 或 Kotlin 代码。代码生成通常与构建集成在一起。

**DGS 代码生成插件可供 Gradle 和 Maven 使用。**插件在我们项目的构建过程中基于我们的域图服务的 GraphQL 模式文件生成代码。**该插件可以为类型、输入类型、枚举和接口、样本数据提取器和类型安全查询 API 生成数据类型。**还有一个`DgsConstants`类，包含类型和字段的名称。

## 7.测试

查询我们的 API 的一种便捷方式是 [GraphiQL](https://web.archive.org/web/20220707143855/https://github.com/graphql/graphiql) 。 **GraphiQL 是一个与 DGS 框架一起开箱即用的查询编辑器。**让我们在默认的 Spring Boot 端口上启动应用程序，并检查 URL ` http://localhost:8080/graphiql`。让我们尝试以下查询并测试结果:

```java
{
    albums{
        title
    }
} 
```

注意，与 REST 不同，我们必须明确列出希望从查询中返回哪些字段。我们来看看回应:

```java
{
  "data": {
    "albums": [
      {
        "title": "Rumours"
      },
      {
        "title": "What's Going On"
      },
      {
        "title": "Pet Sounds"
      }
    ]
  }
}
```

## 8.结论

域图服务框架是使用 GraphQL 的一种简单且颇具吸引力的方式。它使用更高级别的构建块来处理查询执行等。DGS 框架通过一个方便的 Spring Boot 编程模型实现了所有这些。这个框架有一些有用的特性，我们将在本文中介绍。

我们讨论了在我们的应用程序中配置 DGS，并查看了它的一些基本注释。然后，我们编写了一个简单的应用程序来检查如何从模式中创建数据并查询它们。最后，我们使用 GraphiQL 测试了我们的 API。和往常一样，这个例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143855/https://github.com/eugenp/tutorials/tree/master/graphql-modules/graphql-dgs)