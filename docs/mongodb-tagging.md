# 用 MongoDB 实现简单的标记

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-tagging>

[This article is part of a series:](javascript:void(0);)[• A Simple Tagging Implementation with Elasticsearch](/web/20220625175311/https://www.baeldung.com/elasticsearch-tagging)
[• A Simple Tagging Implementation with JPA](/web/20220625175311/https://www.baeldung.com/jpa-tagging)
[• An Advanced Tagging Implementation with JPA](/web/20220625175311/https://www.baeldung.com/jpa-tagging-advanced)
• A Simple Tagging Implementation with MongoDB (current article)

## 1。概述

在本教程中，我们将看看一个使用 Java 和 MongoDB 的简单标记实现。

对于那些不熟悉这个概念的人来说，**标签是一个关键字，用作将文档分组到不同类别的“标签”。**这允许用户快速浏览相似的内容，在处理大量数据时尤其有用。

也就是说，这种技术在博客中非常普遍也就不足为奇了。在这种情况下，根据所涉及的主题，每篇文章都有一个或多个标签。当用户完成阅读后，他可以跟随其中一个标签来查看与该主题相关的更多内容。

让我们看看如何实现这个场景。

## 2。依赖性

为了查询数据库，我们必须在我们的`pom.xml`中包含 MongoDB 驱动程序依赖性:

```java
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.6.3</version>
</dependency>
```

这个依赖关系的当前版本可以在[这里](https://web.archive.org/web/20220625175311/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.mongodb%22%20AND%20a%3A%22mongo-java-driver%22)找到。

## 3。数据模型

首先，让我们从规划一个帖子文档应该是什么样子开始。

为了简单起见，我们的数据模型将只有一个标题，我们还将使用它作为文档 id、作者和一些标签。

我们将标签存储在一个数组中，因为一篇文章可能不止一个:

```java
{
    "_id" : "Java 8 and MongoDB",
    "author" : "Donato Rimenti",
    "tags" : ["Java", "MongoDB", "Java 8", "Stream API"]
}
```

我们还将创建相应的 Java 模型类:

```java
public class Post {
    private String title;
    private String author;
    private List<String> tags;

    // getters and setters
}
```

## 4。更新标签

既然我们已经建立了数据库并插入了几篇示例文章，让我们看看如何更新它们。

我们的存储库类将包含两个方法，通过使用标题来查找标签来处理标签的添加和删除。我们还将返回一个布尔值来表明查询是否更新了一个元素:

```java
public boolean addTags(String title, List<String> tags) {
    UpdateResult result = collection.updateOne(
      new BasicDBObject(DBCollection.ID_FIELD_NAME, title), 
      Updates.addEachToSet(TAGS_FIELD, tags));
    return result.getModifiedCount() == 1;
}

public boolean removeTags(String title, List<String> tags) {
    UpdateResult result = collection.updateOne(
      new BasicDBObject(DBCollection.ID_FIELD_NAME, title), 
      Updates.pullAll(TAGS_FIELD, tags));
    return result.getModifiedCount() == 1;
}
```

我们使用`addEachToSet` 方法代替`push` 进行添加，这样如果标签已经存在，我们就不会再添加它们。

还要注意,`addToSet` 操作符也不起作用，因为它会将新标签作为嵌套数组添加，这不是我们想要的。

**我们执行更新的另一种方式是通过 Mongo shell。**例如，让我们更新帖子`JUnit5 with Java.`，特别是，我们要添加标签`Java`和 J `Unit5`，删除标签`Spring`和`REST`:

```java
db.posts.updateOne(
    { _id : "JUnit 5 with Java" }, 
    { $addToSet : 
        { "tags" : 
            { $each : ["Java", "JUnit5"] }
        }
});

db.posts.updateOne(
    {_id : "JUnit 5 with Java" },
    { $pull : 
        { "tags" : { $in : ["Spring", "REST"] }
    }
});
```

## 5。查询

最后但同样重要的是，让我们浏览一些在使用标签时我们可能感兴趣的最常见的查询。为此，我们将特别利用三个数组运算符:

*   `**$in –**` 返回文档，其中**字段包含指定数组的任意值**
*   `**$nin –**`返回文档，其中**字段不包含指定数组的任何值**
*   `**$all –**`返回文档，其中**字段包含指定数组的所有值**

**我们将定义三种方法来查询与作为参数**传递的标签集合相关的帖子。它们将返回与至少一个标签匹配的帖子，所有标签匹配的帖子，没有标签匹配的帖子。我们还将使用 Java 8 的流 API 创建一个映射方法来处理文档和模型之间的转换:

```java
public List<Post> postsWithAtLeastOneTag(String... tags) {
    FindIterable<Document> results = collection
      .find(Filters.in(TAGS_FIELD, tags));
    return StreamSupport.stream(results.spliterator(), false)
      .map(TagRepository::documentToPost)
      .collect(Collectors.toList());
}

public List<Post> postsWithAllTags(String... tags) {
    FindIterable<Document> results = collection
      .find(Filters.all(TAGS_FIELD, tags));
    return StreamSupport.stream(results.spliterator(), false)
      .map(TagRepository::documentToPost)
      .collect(Collectors.toList());
}

public List<Post> postsWithoutTags(String... tags) {
    FindIterable<Document> results = collection
      .find(Filters.nin(TAGS_FIELD, tags));
    return StreamSupport.stream(results.spliterator(), false)
      .map(TagRepository::documentToPost)
      .collect(Collectors.toList());
}

private static Post documentToPost(Document document) {
    Post post = new Post();
    post.setTitle(document.getString(DBCollection.ID_FIELD_NAME));
    post.setAuthor(document.getString("author"));
    post.setTags((List<String>) document.get(TAGS_FIELD));
    return post;
}
```

同样，**让我们也来看看 shell 等价查询**。我们将获取三个不同的 post 集合，分别标记为`MongoDB`或`Stream` API，标记为`Java 8`和`JUnit 5`，没有标记为`Groovy`或`Scala`:

```java
db.posts.find({
    "tags" : { $in : ["MongoDB", "Stream API" ] } 
});

db.posts.find({
    "tags" : { $all : ["Java 8", "JUnit 5" ] } 
});

db.posts.find({
    "tags" : { $nin : ["Groovy", "Scala" ] } 
});
```

## 6。结论

在本文中，我们展示了如何构建标记机制。当然，除了博客之外，我们还可以将这种方法用于其他目的。

如果您有兴趣进一步学习 MongoDB，[我们鼓励您阅读这篇介绍性文章](/web/20220625175311/https://www.baeldung.com/java-mongodb)。

和往常一样，示例中的所有代码都可以在 Github 项目中的[处获得。](https://web.archive.org/web/20220625175311/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb)

上一篇[一篇高级标记实现同 JPA](/web/20220625175311/https://www.baeldung.com/jpa-tagging-advanced)