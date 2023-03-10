# GraphQL 和 Spring Boot 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-graphql>

## 1。简介

GraphQL 是来自脸书的一个相对较新的概念，被宣传为 Web APIs 的 REST 替代方案。

在本教程中，我们将学习如何使用 Spring Boot 设置一个 GraphQL 服务器，这样我们就可以将它添加到现有的应用程序中，或者在新的应用程序中使用它。

## 2。什么是`GraphQL`？

传统的 REST APIs 使用服务器管理的资源的概念。我们可以遵循各种 HTTP 动词，以一些标准的方式操作这些资源。只要我们的 API 符合资源概念，这种方法就能很好地工作，但是当我们需要背离它时，它很快就会崩溃。

当客户端同时需要来自多个资源的数据时，比如请求一篇博客文章和评论，这也会受到影响。通常，解决这一问题的方法是让客户端发出多个请求，或者让服务器提供可能并不总是需要的额外数据，从而导致更大的响应大小。

GraphQL 为这两个问题提供了解决方案。它允许客户端准确地指定它需要的数据，包括在单个请求中导航子资源，并允许在单个请求中进行多个查询。

它还以一种更 RPC 的方式工作，使用命名查询和变异，而不是一组标准的强制动作。这将控件放在它应该在的地方，API 开发者指定什么是可能的，API 消费者指定什么是期望的。

例如，博客可能允许以下查询:

```java
query {
    recentPosts(count: 10, offset: 0) {
        id
        title
        category
        author {
            id
            name
            thumbnail
        }
    }
}
```

该查询将:

*   请求十个最近的帖子
*   对于每个帖子，请求 ID、标题和类别
*   对于每篇文章，请求作者，返回 ID、姓名和缩略图

在传统的 REST API 中，要么需要 11 个请求，一个针对帖子，10 个针对作者，要么需要在帖子细节中包含作者细节。

### 2.1 .图 QL 方案〔t1〕

GraphQL 服务器公开了描述 API 的模式。这个方案由类型定义组成。每种类型都有一个或多个字段，每个字段接受零个或多个参数，并返回特定的类型。

该图源自这些字段相互嵌套的方式。请注意，该图不必是非循环的，循环是完全可以接受的，但它是有向的。也就是说，客户端可以从一个字段获取其子字段，但是它不能自动返回到父字段，除非模式明确定义了这一点。

博客的示例 GraphQL 模式可以包含描述帖子、帖子的作者以及获取博客上最新帖子的根查询的以下定义:

```java
type Post {
    id: ID!
    title: String!
    text: String!
    category: String
    author: Author!
}

type Author {
    id: ID!
    name: String!
    thumbnail: String
    posts: [Post]!
}

# The Root Query for the application
type Query {
    recentPosts(count: Int, offset: Int): [Post]!
}

# The Root Mutation for the application
type Mutation {
    writePost(title: String!, text: String!, category: String) : Post!
}
```

“！”在一些名字的末尾表示它是一个不可空的类型。任何没有这个的类型在服务器的响应中都可以是 null。GraphQL 服务可以正确地处理这些问题，允许我们安全地请求可空类型的子字段。

GraphQL 服务还使用一组标准字段公开模式本身，允许任何客户机提前查询模式定义。

这允许客户端自动检测模式何时改变，并允许客户端动态适应模式的工作方式。一个非常有用的例子是 GraphQL 工具，它允许我们与任何 GraphQL API 进行交互。

## 3。介绍 GraphQL Spring Boot 启动器

**[Spring Boot GraphQL Starter](https://web.archive.org/web/20220707052213/https://github.com/graphql-java/graphql-spring-boot)提供了一个奇妙的方法，让 graph QL 服务器在很短的时间内运行**。结合 [GraphQL Java Tools](https://web.archive.org/web/20220707052213/https://github.com/graphql-java/graphql-java-tools) 库，我们只需要为我们的服务编写必要的代码。

### 3.1。设置服务

要做到这一点，我们只需要正确的依赖关系:

```java
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-spring-boot-starter</artifactId>
    <version>5.0.2</version>
</dependency>
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-tools</artifactId>
    <version>5.2.4</version>
</dependency>
```

Spring Boot 会自动挑选这些，并设置适当的处理程序来工作。

默认情况下，这将在我们的应用程序的`/graphql`端点上公开 GraphQL 服务，并将接受包含 GraphQL 有效负载的 POST 请求。如果需要，我们可以在我们的`application.properties`文件中定制这个端点。

### 3.2。编写模式

GraphQL 工具库通过处理 GraphQL 模式文件来构建正确的结构，然后将特殊的 beans 连接到该结构。Spring Boot GraphQL 启动程序自动找到这些模式文件。

我们需要用扩展名“来保存这些文件。并且它们可以出现在类路径中的任何地方。我们也可以根据需要拥有尽可能多的这些文件，因此我们可以根据需要将方案划分成模块。

一个要求是必须有一个根查询，以及最多一个根变异。不像方案的其他部分，我们不能跨文件分割它。这是 GraphQL 模式定义本身的限制，而不是 Java 实现的限制。

### 3.3。根查询解析器

**根查询需要在 Spring 上下文中定义特殊的 beans 来处理这个根查询**中的各个字段。与模式定义不同，对于根查询字段，没有限制只能有一个 Spring bean。

唯一的要求是 beans 实现 *GraphQLQueryResolver，*并且来自 scheme 的根查询中的每个字段在这些类中有一个同名的方法:

```java
public class Query implements GraphQLQueryResolver {
    private PostDao postDao;
    public List<Post> getRecentPosts(int count, int offset) {
        return postsDao.getRecentPosts(count, offset);
    }
}
```

方法的名称必须是下列名称之一，按此顺序排列:

2.  `is<field>`–仅当字段类型为*布尔型*时
3.  获取

该方法必须具有与 GraphQL 模式中的任何参数相对应的参数，并且可以选择接受一个类型为 *DataFetchingEnvironment 的最终参数。*

该方法还必须为 GraphQL 方案中的类型返回正确的返回类型，就像我们将要看到的那样。我们可以使用任何简单的类型，`String, Int, List,`等等。，具有等效的 Java 类型，系统只是自动映射它们。

上面定义了方法`getRecentPosts,`，我们将使用它来处理前面定义的模式中的`recentPosts`字段的任何 GraphQL 查询。

### 3.4。使用 Beans 来表示类型

**graph QL 服务器中的每个复杂类型都由一个 Java bean 表示，**无论是从根查询还是从结构中的任何其他地方加载。相同的 Java 类必须总是表示相同的 GraphQL 类型，但是类名不是必需的。

**Java bean 中的字段将根据字段名称直接映射到 GraphQL 响应中的字段:**

```java
public class Post {
    private String id;
    private String title;
    private String category;
    private String authorId;
}
```

Java bean 上没有映射到 GraphQL 模式的任何字段或方法都将被忽略，但不会导致问题。这对于场分解器的工作很重要。

例如，这里的字段`authorId` 与我们之前定义的模式中的任何内容都不对应，但是它可以用于下一步。

### 3.5。复数值的字段解析器

有时，字段的值对于加载来说是很重要的。这可能涉及数据库查找、复杂的计算或其他任何事情。GraphQL 工具有一个用于此目的的字段解析器的概念。这些是可以代替数据 bean 提供值的 Spring beans。

字段解析器是 Spring 上下文中与数据 bean 同名的任何 bean，带有后缀`Resolver`，并实现了`GraphQLResolver`接口。字段解析器 bean 上的方法遵循与数据 bean 上所有相同的规则，但也提供数据 bean 本身作为第一个参数。

如果字段解析器和数据 bean 对于同一个 GraphQL 字段都有方法，则字段解析器优先:

```java
public class PostResolver implements GraphQLResolver<Post> {
    private AuthorDao authorDao;

    public Author getAuthor(Post post) {
        return authorDao.getAuthorById(post.getAuthorId());
    }
}
```

这些字段解析器从 Spring 上下文中加载是很重要的。这允许它们与任何其他 Spring 管理的 beans 一起工作，例如 DAOs。

重要的是，**如果客户端不请求字段，那么 GraphQL 服务器不会检索它**。这意味着，如果一个客户端检索一篇文章，并且没有询问作者，上面的`getAuthor()`方法就不会被执行，也不会进行 DAO 调用。

### 3.6。可空值

GraphQL 模式的概念是，有些类型是可空的，而有些不是。

我们在 Java 代码中通过直接使用空值来处理这个问题。相反，我们可以直接对可空类型使用 Java 8 中的新`Optional`类型，系统会对这些值做正确的事情。

这非常有用，因为这意味着我们的 Java 代码更明显地与方法定义中的 GraphQL 模式相同。

### 3.7。突变

到目前为止，我们所做的一切都是关于从服务器检索数据。GraphQL 还能够通过突变的方式更新存储在服务器上的数据。

从代码的角度来看，查询没有理由不能改变服务器上的数据。我们可以轻松地编写接受参数、保存新数据并返回这些更改的查询解析器。这样做会给 API 客户端带来意想不到的副作用，这被认为是一种不好的做法。

相反，应该使用**突变来通知客户端，这将导致正在存储的数据发生变化**。

通过使用实现`GraphQLMutationResolver,`而不是`GraphQLQueryResolver`的类在 Java 代码中定义突变。

否则，所有相同的规则都适用于查询。然后，变异字段的返回值将被视为与查询字段完全相同，也允许检索嵌套值:

```java
public class Mutation implements GraphQLMutationResolver {
    private PostDao postDao;

    public Post writePost(String title, String text, String category) {
        return postDao.savePost(title, text, category);
    }
}
```

## 4。介绍 GraphiQL

GraphQL 还有一个配套工具叫做 GraphiQL。这是一个能够与任何 GraphQL 服务器通信，并对其执行查询和变异的 UI。它的可下载版本以电子应用程序的形式存在，可以从[这里](https://web.archive.org/web/20220707052213/https://github.com/skevy/graphiql-app)检索。

通过添加 GraphiQL Spring Boot 启动器依赖项，还可以将基于 web 的 GraphiQL 版本自动包含在我们的应用程序中:

```java
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphiql-spring-boot-starter</artifactId>
    <version>5.0.2</version>
</dependency> 
```

只有当我们在默认端点`/graphql;`上托管我们的 GraphQL API 时，这才会起作用，否则我们将需要独立的应用程序。

## 5。总结

GraphQL 是一项非常令人兴奋的新技术，它可能会彻底改变我们开发 Web APIs 的方式。

Spring Boot GraphQL Starter 和 GraphQL Java 工具库的结合使得将这项技术添加到任何新的或现有的 Spring Boot 应用程序中变得非常容易。

代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220707052213/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries)