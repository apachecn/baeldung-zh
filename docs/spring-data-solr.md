# Spring Data Solr 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-solr>

## 1。概述

在本文中，**我们将以实用的方式探索 Spring Data Solr** 的基础知识。

Apache Solr 是一个开源的、随时可以部署的企业全文搜索引擎。你可以在[官网](https://web.archive.org/web/20220529023744/https://lucene.apache.org/solr/resources.html)找到更多关于 Solr 的特性。

我们将展示如何进行简单的 Solr 配置，当然还有如何与服务器交互。

首先，我们需要启动一个 Solr 服务器并创建一个核心来存储数据(默认情况下，Solr 将以无模式模式创建)。

## 2。春季数据

正如任何其他 Spring 数据项目一样，Spring Data Solr 有一个明确的目标，那就是移除样板代码，我们肯定会利用这一点。

### 2.1。Maven 依赖关系

让我们从将 Spring 数据 Solr 依赖项添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-solr</artifactId>
    <version>4.3.14</version>
</dependency>
```

你可以在这里找到最新的依赖关系。

### 2.2。定义文档

让我们定义一个名为`Product`的文档:

```java
@SolrDocument(solrCoreName = "product")
public class Product {

    @Id
    @Indexed(name = "id", type = "string")
    private String id;

    @Indexed(name = "name", type = "string")
    private String name;

}
```

`@SolrDocument`注释表明`Product`类是一个 Solr 文档，并被索引到名为`product`的内核。用`@Indexed` 标注的字段在 Solr 中被索引，并且是可搜索的。

### 2.3。定义存储库接口

接下来，我们需要通过扩展 Spring Data Solr 提供的存储库来创建一个存储库接口。我们将自然地用`Product`和`String`作为我们的实体 id 对此进行参数化:

```java
public interface ProductRepository extends SolrCrudRepository<Product, String> {

    public List<Product> findByName(String name);

    @Query("id:*?0* OR name:*?0*")
    public Page<Product> findByCustomQuery(String searchTerm, Pageable pageable);

    @Query(name = "Product.findByNamedQuery")
    public Page<Product> findByNamedQuery(String searchTerm, Pageable pageable);

}
```

注意我们是如何在这里定义三个方法的，在由`SolrCrudRepository`提供的 API 之上。我们将在接下来的几节中讨论这些。

还要注意，`Product.findByNamedQuery`属性是在类路径文件夹中的 Solr 命名查询文件`solr-named-queries.properties`中定义的:

```java
Product.findByNamedQuery=id:*?0* OR name:*?0*
```

### 2.4。Java 配置

现在我们将探索 Solr 持久层的 Spring 配置:

```java
@Configuration
@EnableSolrRepositories(
  basePackages = "com.baeldung.spring.data.solr.repository",
  namedQueriesLocation = "classpath:solr-named-queries.properties")
@ComponentScan
public class SolrConfig {

    @Bean
    public SolrClient solrClient() {
        return new HttpSolrClient.Builder("http://localhost:8983/solr").build();
    }

    @Bean
    public SolrTemplate solrTemplate(SolrClient client) throws Exception {
        return new SolrTemplate(client);
    }
}
```

我们使用`@EnableSolrRepositories`来扫描包中的存储库。请注意，我们已经指定了命名查询属性文件的位置，并启用了多核支持。

如果没有启用多核，那么默认情况下，Spring 数据将假设 Solr 配置是针对单核的。我们在这里启用多核，仅供参考。

## 3。带 Spring Boot 的弹簧数据 Solr】

在这一节中，我们将看看 Spring Boot 应用程序的设置是什么样子的。

让我们从将 Spring Boot 启动器数据 Solr 依赖项添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-solr</artifactId>
    <version>2.4.12</version>
</dependency>
```

你可以在这里找到依赖关系[的最新版本。](https://web.archive.org/web/20220529023744/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-data-solr%22)

我们还必须**用 Solr URL 的值定义`application.properties`文件中的属性`spring.data.solr.host`** :

```java
spring.data.solr.host=http://localhost:8983/solr 
```

确保 Solr 正在指定的 URL 上运行。

这是我们在 Spring Boot 应用程序中设置 Spring Data Solr 所需的所有配置，因为在类路径中有 starter 将加载自动配置。

## 4。索引、更新和删除

为了在 Solr 中搜索文档，应该将文档索引到 Solr 存储库中。

下面的例子通过使用`SolrCrudRepository's` save 方法简单地索引 Solr 存储库中的产品文档:

```java
Product phone = new Product();
phone.setId("P0001");
phone.setName("Phone");
productRepository.save(phone);
```

现在让我们检索和更新一个文档:

```java
Product retrievedProduct = productRepository.findById("P0001").get();
retrievedProduct.setName("Smart Phone");
productRepository.save(retrievedProduct);
```

只需调用 delete 方法就可以删除文档:

```java
productRepository.delete(retrievedProduct);
```

## 5。查询

现在让我们探索 Spring Data Solr API 提供的不同查询技术。

### 5.1。方法名查询生成

基于方法名的查询是通过解析方法名以生成要执行的预期查询来生成的:

```java
public List<Product> findByName(String name);
```

在我们的存储库接口中，我们有基于方法名生成查询的`findByName`方法:

```java
List<Product> retrievedProducts = productRepository.findByName("Phone");
```

### 5.2。带`@Query`标注的查询

Solr 搜索查询可以通过将查询放在方法的`@Query`注释中来创建。在我们的例子中,`findByCustomQuery`是用`@Query`标注的:

```java
@Query("id:*?0* OR name:*?0*")
public Page<Product> findByCustomQuery(String searchTerm, Pageable pageable);
```

让我们用这种方法来检索文档:

```java
Page<Product> result 
  = productRepository.findByCustomQuery("Phone", PageRequest.of(0, 10));
```

通过调用`findByCustomQuery(“Phone”, PageRequest.of(0, 10))`,我们获得了产品文档的第一页，该页在其字段`id`或`name`中包含单词“Phone”。

### 5.3。命名查询

命名查询类似于带有`@Query`注释的查询，只是这些查询是在单独的属性文件中声明的:

```java
@Query(name = "Product.findByNamedQuery")
public Page<Product> findByNamedQuery(String searchTerm, Pageable pageable);
```

注意，如果属性文件中查询的关键字(`findByNamedQuery`)与方法名匹配，则不需要`@Query`注释。

让我们使用命名查询方法检索一些文档:

```java
Page<Product> result 
  = productRepository.findByNamedQuery("one", PageRequest.of(0, 10));
```

## 6。结论

本文是对 Spring Data Solr 的快速实用的介绍，涵盖了基本配置、定义存储库和自然查询。

和往常一样，这里使用的例子可以从 Github 上的[示例项目中获得。](https://web.archive.org/web/20220529023744/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-solr)