# 用不同的名称公开 GraphQL 字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/graphql-field-name>

## 1.概观

GraphQL 已经被广泛用作 web 服务中的一种通信模式。**graph QL 的基本前提是被客户端应用灵活使用。**

在本教程中，我们将研究灵活性的另一个方面。我们还将探索如何用不同的名称公开 GraphQL 字段。

## 2.GraphQL 模式

让我们以一个[博客](/web/20220817142632/https://www.baeldung.com/spring-graphql)拥有不同`Authors`的`Posts`为例。GraphQL 模式如下所示:

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
query {
    recentPosts(count: 1,offset: 0){
        id
        title
        text
        category
        author{
            id
            name
            thumbnail
        }
    }
} 
```

在这里我们可以获取最近的帖子。**每一个`post` 都会伴随着它的`author`。**查询的结果如下:

```java
{
    "data": {
        "recentPosts": [
            {
                "id": "Post00",
                "title": "Post 0:0",
                "text": "Post 0 + by author 0",
                "category": null,
                "author": {
                    "id": "Author0",
                    "name": "Author 0",
                    "thumbnail": "http://example.com/authors/0"
                }
            }
        ]
    }
}
```

## 3.使用不同的名称公开 GraphQL 字段

客户端应用程序可能需要使用字段`first_author.`，现在它正在使用 `author`。为了满足这一要求，我们有两种解决方案:

*   更改 GraphQL 服务器中模式的定义
*   利用 GraphQL 中的别名概念

让我们一个一个来看。

### 3.1.更改模式

让我们更新 `post`的模式定义:

```java
type Post {
    id: ID!
    title: String!
    text: String!
    category: String
    first_author: Author!
} 
```

`author`不是一个无足轻重的领域。这是个复杂的问题。我们还必须更新相应的解析器来适应这种变化。

**`PostResolver,` 中的方法`getAuthor(Post post),` 将更新为`getFirst_author(Post post)`。**

以下是查询:

```java
query{
    recentPosts(count: 1,offset: 0){
        id
        title
        text
        category
        first_author{
            id
            name
            thumbnail
        }
    }
} 
```

上述查询的结果如下:

```java
{
    "data": {
        "recentPosts": [
            {
                "id": "Post00",
                "title": "Post 0:0",
                "text": "Post 0 + by author 0",
                "category": null,
                "first_author": {
                    "id": "Author0",
                    "name": "Author 0",
                    "thumbnail": "http://example.com/authors/0"
                }
            }
        ]
    }
} 
```

这种解决方案有两个主要问题:

*   它引入了对模式和服务器端实现的更改
*   它迫使其他客户端应用程序遵循这个更新的模式定义

这些问题与 GraphQL 提供的灵活性相矛盾。

### 3.2 .GraphQL 别名

别名，在 GraphQL 中，让我们在不改变模式定义的情况下，将字段的结果重命名为我们想要的任何名称。要在查询中引入别名，别名和冒号符号(:)必须在 GraphQL 字段之前。

下面是查询的演示:

```java
query{
    recentPosts(count: 1,offset: 0){
        id
        title
        text
        category
        first_author:author{
            id
            name
            thumbnail
        }
    }
} 
```

上述查询的结果如下:

```java
{
    "data": {
        "recentPosts": [
            {
                "id": "Post00",
                "title": "Post 0:0",
                "text": "Post 0 + by author 0",
                "category": null,
                "first_author": {
                    "id": "Author0",
                    "name": "Author 0",
                    "thumbnail": "http://example.com/authors/0"
                }
            }
        ]
    }
} 
```

让我们注意，查询本身正在请求第一个帖子。另一个客户端应用程序可能会请求使用`first_post` 而不是`recentPosts.`,别名将再次派上用场。

```java
query{
    first_post: recentPosts(count: 1,offset: 0){
        id
        title
        text
        category
        author{
            id
            name
            thumbnail
        }
    }
} 
```

上述查询的结果如下:

```java
{
    "data": {
        "first_post": [
            {
                "id": "Post00",
                "title": "Post 0:0",
                "text": "Post 0 + by author 0",
                "category": null,
                "author": {
                    "id": "Author0",
                    "name": "Author 0",
                    "thumbnail": "http://example.com/authors/0"
                }
            }
        ]
    }
} 
```

这两个例子清楚地显示了使用 GraphQL 的灵活性。每个客户端应用程序都可以根据需要进行自我更新。同时，服务器端模式定义和实现保持不变。

## 4.结论

在本文中，我们研究了用不同的名称公开 graphQL 字段的两种方法。我们已经通过例子介绍了别名的概念，并解释了它是如何正确使用的。

和往常一样，本文的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220817142632/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries)