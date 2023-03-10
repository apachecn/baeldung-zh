# ArangoDB 的春季数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-arangodb>

## 1.介绍

在本教程中，我们将学习如何使用 [Spring 数据](/web/20220707143816/https://www.baeldung.com/spring-data)模块和 [ArangoDB](https://web.archive.org/web/20220707143816/https://www.arangodb.com/) 数据库。ArangoDB 是一个免费的开源多模型数据库系统。它支持键值、文档和图形数据模型，具有一个数据库核心和一种统一的查询语言: [AQL (ArangoDB 查询语言)。](https://web.archive.org/web/20220707143816/https://www.arangodb.com/docs/stable/aql/)

我们将涵盖所需的配置、基本的 CRUD 操作、定制查询和实体关系。

## 2.ArangoDB 设置

要安装 ArangoDB，我们首先需要从 ArangoDB 官方网站的[下载](https://web.archive.org/web/20220707143816/https://www.arangodb.com/download/)页面下载这个包。

出于本教程的目的，我们将安装 ArangoDB 的社区版。详细的安装步骤可以在这里找到[。](https://web.archive.org/web/20220707143816/https://www.arangodb.com/docs/stable/installation.html)

默认安装包含一个名为`_system`的数据库和一个能够访问所有数据库的`root`用户。

根据软件包的不同，安装程序会在安装过程中询问 root 密码，或者设置一个随机密码。

在默认配置下，我们将看到 ArangoDB 服务器运行在`8529`端口上。

一旦设置完成，我们就可以使用在`http://localhost:8529`上可访问的 Web 界面与服务器进行交互。在本教程的后面部分，我们将使用这个主机和端口进行 Spring 数据配置。

我们也可以使用同步 shell】与服务器交互。

让我们从启动`arangosh`开始，创建一个名为`baeldung-database`的新数据库和一个可以访问这个新创建的数据库的用户`baeldung`。

```java
arangosh> db._createDatabase("baeldung-database", {}, [{ username: "baeldung", passwd: "password", active: true}]);
```

## 3.属国

为了在我们的应用程序中使用带有 ArangoDB 的 Spring 数据，我们需要在依赖关系之后的[:](https://web.archive.org/web/20220707143816/https://search.maven.org/search?q=a:arangodb-spring-data)

```java
<dependency>
    <groupId>com.arangodb</groupId>
    <artifactId>arangodb-spring-data</artifactId>
    <version>3.5.0</version>
</dependency>
```

## 4.配置

在我们开始处理数据之前，我们需要建立到`ArangoDB`的连接。我们应该通过创建一个实现`ArangoConfiguration`接口的配置类来实现它:

```java
@Configuration
public class ArangoDbConfiguration implements ArangoConfiguration {}
```

在内部，我们需要实现两个方法。第一个应该创建`ArangoDB.Builder`对象，该对象将生成到我们数据库的接口:

```java
@Override
public ArangoDB.Builder arango() {
    return new ArangoDB.Builder()
      .host("127.0.0.1", 8529)
      .user("baeldung").password("password"); }
```

创建连接需要四个参数:主机、端口、用户名和密码。

或者，我们可以跳过在配置类中设置这些参数:

```java
@Override
public ArangoDB.Builder arango() {
    return new ArangoDB.Builder();
}
```

因为我们可以将它们存储在`arango.properties`资源文件中:

```java
arangodb.host=127.0.0.1
arangodb.port=8529
arangodb.user=baeldung
arangodb.password=password
```

这是阿兰戈寻找的默认位置。可以通过向自定义属性文件传递一个`InputStream`来覆盖它:

```java
InputStream in = MyClass.class.getResourceAsStream("my.properties");
ArangoDB.Builder arango = new ArangoDB.Builder()
  .loadProperties(in);
```

我们必须实现的第二个方法是简单地提供我们在应用程序中需要的数据库名称:

```java
@Override
public String database() {
    return "baeldung-database";
}
```

此外，配置类需要 `@EnableArangoRepositories` 注释来告诉 Spring Data 在哪里寻找 ArangoDB 存储库:

```java
@EnableArangoRepositories(basePackages = {"com.baeldung"})
```

## 5.数据模型

下一步，我们将创建一个数据模型。对于这篇文章，我们将使用一个带有`name`、`author`和`publishDate`字段的文章表示:

```java
@Document("articles")
public class Article {

    @Id
    private String id;

    @ArangoId
    private String arangoId;

    private String name;
    private String author;
    private ZonedDateTime publishDate;

    // constructors
}
```

`ArangoDB`实体必须有 `@Document` 注释，该注释将集合名称作为参数。默认情况下，它是一个非大写的类名。

接下来，我们有两个 id 字段。一个带有弹簧的 `@Id` 注释，第二个带有阿兰戈的 `@ArangoId` 注释。第一个存储生成的实体 id。第二个在数据库中的适当位置存储相同的 id 螺母。在我们的例子中，这些值可以相应地是 `1` 和 `articles/1` 。

现在，当我们定义了实体后，我们可以为数据访问创建一个存储库接口:

```java
@Repository
public interface ArticleRepository extends ArangoRepository<Article, String> {}
```

它应该用两个通用参数扩展`ArangoRepository`接口。在我们的例子中，它是一个 id 类型为`String`的`Article`类。

## 6.CRUD 操作

最后，我们可以创建一些具体的数据。

首先，我们需要一个对文章存储库的依赖:

```java
@Autowired
ArticleRepository articleRepository;
```

和一个简单的`Article`类实例:

```java
Article newArticle = new Article(
  "ArangoDb with Spring Data",
  "Baeldung Writer",
  ZonedDateTime.now()
);
```

现在，如果我们想将这篇文章存储在我们的数据库中，我们应该简单地调用 `save` 方法:

```java
Article savedArticle = articleRepository.save(newArticle);
```

之后，我们可以确保生成了 `id` 和 `arangoId` 字段:

```java
assertNotNull(savedArticle.getId());
assertNotNull(savedArticle.getArangoId());
```

要从数据库中获取文章，我们需要首先获取它的 id:

```java
String articleId = savedArticle.getId();
```

然后简单地调用`findById`方法:

```java
Optional<Article> articleOpt = articleRepository.findById(articleId);
assertTrue(articleOpt.isPresent());
```

有了文章实体，我们可以改变它的属性:

```java
Article article = articleOpt.get();
article.setName("New Article Name");
articleRepository.save(article);
```

最后，再次调用`save`方法来更新数据库条目。它不会创建新条目，因为 id 已经分配给实体。

删除条目也是一个简单的操作。我们简单地调用存储库的`delete`方法:

```java
articleRepository.delete(article)
```

通过 id 删除它也是可能的:

```java
articleRepository.deleteById(articleId)
```

## 7.自定义查询

有了`Spring Data`和`ArangoDB,`,我们可以利用[派生的库](/web/20220707143816/https://www.baeldung.com/spring-data-derived-queries),并简单地通过方法名定义查询:

```java
@Repository
public interface ArticleRepository extends ArangoRepository<Article, String> {
    Iterable<Article> findByAuthor(String author);
}
```

第二个选项是使用 [AQL (ArangoDb 查询语言)](https://web.archive.org/web/20220707143816/https://www.arangodb.com/docs/stable/aql/)。这是一种定制的语法语言，我们可以用 [`@Query`注解](/web/20220707143816/https://www.baeldung.com/spring-data-jpa-query)来应用它。

现在，让我们来看一个基本的 AQL 查询，该查询将查找给定作者的所有文章，并按发表日期对它们进行排序:

```java
@Query("FOR a IN articles FILTER a.author == @author SORT a.publishDate ASC RETURN a")
Iterable<Article> getByAuthor(@Param("author") String author);
```

## 8.关系

`ArangoDB`给出在实体间创建关系的可能性。

作为一个例子，让我们创建一个`Author`类和它的文章之间的关系。

为此，我们需要用`@Relations`注释定义一个新的集合属性，该属性将包含给定作者撰写的每篇文章的链接:

```java
@Relations(edges = ArticleLink.class, lazy = true)
private Collection<Article> articles;
```

正如我们所见，`ArangoDB`中的关系是通过一个单独的用@Edge 注释的类定义的:

```java
@Edge
public class ArticleLink {

    @From
    private Article article;

    @To
    private Author author;

    // constructor, getters and setters
}
```

它带有两个标注有 `@From` 和`@To` `.` 的字段，它们定义了传入和传出的关系。

## 9.结论

在本教程中，我们学习了如何配置 `ArangoDB` 并将其用于 Spring 数据。我们已经介绍了基本的 CRUD 操作、定制查询和实体关系。

与往常一样，所有的源代码都可以在 GitHub 上找到。