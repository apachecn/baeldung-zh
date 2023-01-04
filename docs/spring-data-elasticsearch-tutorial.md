# Spring 数据弹性搜索简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-elasticsearch-tutorial>

## 1。概述

在本教程中，**我们将以注重代码和实用的方式探索 Spring 数据弹性搜索**的基础知识。

我们将学习如何使用 Spring 数据 Elasticsearch 在 Spring 应用程序中索引、搜索和查询 Elasticsearch。Spring Data Elasticseach 是一个 Spring 模块，它实现了 Spring 数据，从而提供了一种与流行的开源、基于 Lucene 的搜索引擎进行交互的方式。

虽然 Elasticsearch 可以在没有明确定义的模式的情况下工作，但设计一个模式并创建映射来指定我们在某些字段中期望的数据类型仍然是一种常见的做法。当文档被索引时，它的字段根据它们的类型被处理。例如，将根据映射规则对文本字段进行标记化和过滤。我们也可以创建自己的过滤器和标记器。

为了简单起见，我们将为我们的 Elasticsearch 实例使用 docker 映像，尽管**任何侦听端口 9200 的 Elasticsearch 实例都可以使用**。

我们将从启动我们的 Elasticsearch 实例开始:

```java
docker run -d --name es762 -p 9200:9200 -e "discovery.type=single-node" elasticsearch:7.6.2
```

## 2。春季数据

Spring 数据有助于避免样板代码。例如，如果我们定义一个存储库接口，它扩展了 Spring Data Elasticsearch 为相应的文档类提供的`,`接口，那么 CRUD 操作将默认可用。

此外，方法实现将简单地通过以预定义的格式声明方法名来为我们生成。没有必要编写存储库接口的实现。

关于 [Spring Data](/web/20220815043426/https://www.baeldung.com/spring-data) 的 Baeldung 指南提供了关于这个主题的入门要点。

### 2.1。Maven 依赖关系

Spring Data Elasticsearch 为搜索引擎提供了一个 Java API。为了使用它，我们需要向`pom.xml`添加一个新的依赖项:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
    <version>4.0.0.RELEASE</version>
</dependency>
```

### 2.2。定义存储库接口

为了定义新的存储库，我们将扩展一个提供的存储库接口，用我们实际的文档和主键类型替换通用类型。

值得注意的是，`ElasticsearchRepository`从`PagingAndSortingRepository.` 扩展而来，这允许对分页和排序的内置支持。

在我们的示例中，我们将在自定义搜索方法中使用分页功能:

```java
public interface ArticleRepository extends ElasticsearchRepository<Article, String> {

    Page<Article> findByAuthorsName(String name, Pageable pageable);

    @Query("{\"bool\": {\"must\": [{\"match\": {\"authors.name\": \"?0\"}}]}}")
    Page<Article> findByAuthorsNameUsingCustomQuery(String name, Pageable pageable);
}
```

使用`findByAuthorsName`方法，存储库代理将基于方法名创建一个实现。解析算法将确定它需要访问`authors`属性，然后搜索每个项目的`name`属性。

第二种方法是`findByAuthorsNameUsingCustomQuery`，它使用一个定制的 Elasticsearch 布尔查询，这个查询是使用 `@Query`注释定义的，它要求作者的名字和提供的`name`参数之间严格匹配。

### 2.3。Java 配置

在我们的 Java 应用程序中配置 Elasticsearch 时，我们需要定义如何连接到 Elasticsearch 实例。为此，我们将使用 Elasticsearch 依赖项提供的`RestHighLevelClient,`:

```java
@Configuration
@EnableElasticsearchRepositories(basePackages = "com.baeldung.spring.data.es.repository")
@ComponentScan(basePackages = { "com.baeldung.spring.data.es.service" })
public class Config {

    @Bean
    public RestHighLevelClient client() {
        ClientConfiguration clientConfiguration 
            = ClientConfiguration.builder()
                .connectedTo("localhost:9200")
                .build();

        return RestClients.create(clientConfiguration).rest();
    }

    @Bean
    public ElasticsearchOperations elasticsearchTemplate() {
        return new ElasticsearchRestTemplate(client());
    }
}
```

我们使用标准的支持 Spring 的样式注释。`@EnableElasticsearchRepositories`将使 Spring Data Elasticsearch 扫描为 Spring 数据仓库提供的包。

为了与我们的 Elasticsearch 服务器通信，我们将使用一个简单的`[RestHighLevelClient](https://web.archive.org/web/20220815043426/https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high.html)` `.`虽然 Elasticsearch 提供了多种类型的客户端，但是使用`RestHighLevelClient`是一个保证与服务器通信的好方法。

最后，我们将设置一个`ElasticsearchOperations` bean 来在我们的服务器上执行操作。在这种情况下，我们实例化一个`ElasticsearchRestTemplate`。

## 3。映射

我们使用映射来定义文档的模式。通过为我们的文档定义一个模式，我们保护它们免受不希望的结果，比如映射到不希望的类型。

我们的实体是一个简单的文档，`Article,`，其中`id`的类型是`String`。我们还将指定这样的文档必须存储在`article`类型中名为`blog`的索引中。

```java
@Document(indexName = "blog", type = "article")
public class Article {

    @Id
    private String id;

    private String title;

    @Field(type = FieldType.Nested, includeInParent = true)
    private List<Author> authors;

    // standard getters and setters
}
```

索引可以有几种类型，我们可以用它们来实现层次结构。

我们将把 `authors`字段标记为`FieldType.Nested`。这允许我们单独定义`Author`类，但是当它在 Elasticsearch 中被索引时，仍然有 author 的单个实例嵌入在一个`Article`文档中。

## 4。索引文件

Spring Data Elasticsearch 通常会根据项目中的实体自动创建索引。但是，我们也可以通过客户端模板以编程方式创建索引:

```java
elasticsearchTemplate.indexOps(Article.class).create();
```

然后我们可以将文档添加到索引中:

```java
Article article = new Article("Spring Data Elasticsearch");
article.setAuthors(asList(new Author("John Smith"), new Author("John Doe")));
articleRepository.save(article);
```

## 5。查询

### 5.1。基于方法名的查询

当我们使用基于方法名的查询时，我们编写方法来定义我们想要执行的查询。在设置过程中，Spring Data 将解析方法签名并相应地创建查询:

```java
String nameToFind = "John Smith";
Page<Article> articleByAuthorName
  = articleRepository.findByAuthorsName(nameToFind, PageRequest.of(0, 10));
```

通过用一个`PageRequest`对象调用`findByAuthorsName`，我们将获得结果的第一页(页码从零开始)，该页最多包含 10 篇文章。page 对象还提供查询的总命中数，以及其他方便的分页信息。

### 5.2。自定义查询

有几种方法可以为 Spring Data Elasticsearch 存储库定义定制查询。一种方法是使用`@Query`注释，如 2.2 节所示。

另一个选择是使用查询构建器来创建我们的自定义查询。

如果我们想搜索标题中有单词“`data`”的文章，我们可以在`title:`上创建一个带有过滤器的`NativeSearchQueryBuilder`

```java
Query searchQuery = new NativeSearchQueryBuilder()
   .withFilter(regexpQuery("title", ".*data.*"))
   .build();
SearchHits<Article> articles = 
   elasticsearchTemplate.search(searchQuery, Article.class, IndexCoordinates.of("blog");
```

## 6。更新和删除

为了更新文档，我们必须首先检索它:

```java
String articleTitle = "Spring Data Elasticsearch";
Query searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title", articleTitle).minimumShouldMatch("75%"))
  .build();

SearchHits<Article> articles = 
   elasticsearchTemplate.search(searchQuery, Article.class, IndexCoordinates.of("blog");
Article article = articles.getSearchHit(0).getContent();
```

然后，我们可以通过使用对象的评估器编辑对象的内容来对文档进行更改:

```java
article.setTitle("Getting started with Search Engines");
articleRepository.save(article);
```

至于删除，有几种选择。我们可以检索文档并使用`delete`方法删除它:

```java
articleRepository.delete(article);
```

一旦我们知道了，我们也可以通过`id`删除它:

```java
articleRepository.deleteById("article_id");
```

还可以创建定制的`deleteBy`查询，并利用 Elasticsearch 提供的批量删除功能:

```java
articleRepository.deleteByTitle("title");
```

## 7。结论

在本文中，我们探讨了如何连接和利用 Spring 数据弹性搜索。我们讨论了如何查询、更新和删除文档。最后，我们学习了如果 Spring Data Elasticsearch 提供的内容不符合我们的需求，如何创建自定义查询。

像往常一样，本文中使用的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220815043426/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-elasticsearch)