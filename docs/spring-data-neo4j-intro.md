# Spring 数据 Neo4j 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-neo4j-intro>

## 1。概述

本文**是对流行的图形数据库 Spring Data Neo4j** 的介绍。

Spring Data Neo4j 支持基于 POJO 的 Neo4j 图形数据库开发，并使用熟悉的 Spring 概念，如核心 API 使用的模板类，并提供基于注释的编程模型。

此外，很多开发人员并不知道 Neo4j 是否真的能满足他们的特定需求；这里是 Stackoverflow 上的一个坚实的概述，讨论为什么要使用 Neo4j 及其利弊。

## 2。Maven 依赖关系

让我们从在`pom.xml.` 中声明 Spring 数据 Neo4j 依赖关系开始，下面提到的 Spring 模块也是 Spring 数据 Neo4j 所需要的:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-neo4j</artifactId>
    <version>5.0.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.neo4j</groupId>
    <artifactId>neo4j-ogm-test</artifactId>
    <version>3.1.2</version>
    <scope>test</scope>
</dependency>
```

这些依赖项还包括测试所需的模块。

请注意，最后一个依赖项的范围是“test”。但是还要注意，在实际的应用程序开发中，您更有可能运行完整的 Neo4J 服务器。

如果我们想使用嵌入式服务器，我们还必须添加依赖关系:

```java
<dependency>
    <groupId>org.neo4j</groupId>
    <artifactId>neo4j-ogm-embedded-driver</artifactId>
    <version>3.1.2</version>
</dependency>
```

Maven Central 上提供了 [spring-data-neo4j](https://web.archive.org/web/20220625180249/https://search.maven.org/search?q=a:spring-data-neo4j) 、 [neo4j-ogm-test](https://web.archive.org/web/20220625180249/https://search.maven.org/search?q=a:neo4j-ogm-test) 和[neo4j-ogm-embedded-driver](https://web.archive.org/web/20220625180249/https://search.maven.org/search?q=a:neo4j-ogm-embedded-driver)依赖项。

## 3。Neo4Jj 配置

Neo4j 配置非常简单，它定义了应用程序连接到服务器的连接设置。与大多数其他 spring 数据模块类似，这是一个 spring 配置，可以定义为 XML 或 Java 配置。

在本教程中，我们将只使用基于 Java 的配置:

```java
public static final String URL = 
  System.getenv("NEO4J_URL") != null ? 
  System.getenv("NEO4J_URL") : "http://neo4j:[[email protected]](/web/20220625180249/https://www.baeldung.com/cdn-cgi/l/email-protection):7474";

@Bean
public org.neo4j.ogm.config.Configuration getConfiguration() {
    return new Builder().uri(URL).build();
}

@Bean
public SessionFactory getSessionFactory() {
    return new SessionFactory(getConfiguration(), 
      "com.baeldung.spring.data.neo4j.domain");
}

@Bean
public Neo4jTransactionManager transactionManager() {
    return new Neo4jTransactionManager(getSessionFactory());
}
```

如上所述，配置很简单，只包含两个设置。首先—`SessionFactory is` ,引用我们创建的代表数据对象的模型。然后，服务器端点和访问凭证的连接属性。

Neo4j 将根据 URI 的协议推断驱动程序类，在我们的例子中是“http”。

请注意，在本例中，连接相关的属性是直接配置给服务器的；然而，在生产应用程序中，这些应该被适当地具体化，并成为项目标准配置的一部分。

## 4。Neo4j 储存库

与 Spring 数据框架一致，Neo4j 支持 Spring 数据存储库抽象行为。这意味着访问底层持久化机制是在内置的`Neo4jRepository` 中抽象的，项目可以直接扩展它并使用提供的开箱即用的操作。

储存库可通过带注释的、命名的或派生的查找器方法来扩展。对 Spring 数据 Neo4j 存储库的支持也基于`Neo4jTemplate`，所以底层功能是相同的。

### 4.1。创建`MovieRepository` & `PersonRepository`

在本教程中，我们使用两个存储库来实现数据持久性:

```java
@Repository
public interface MovieRepository extends Neo4jRepository<Movie, Long> {

    Movie findByTitle(@Param("title") String title);

    @Query("MATCH (m:Movie) WHERE m.title =~ ('(?i).*'+{title}+'.*') RETURN m")
    Collection<Movie> 
      findByTitleContaining(@Param("title") String title);

    @Query("MATCH (m:Movie)<-[:ACTED_IN]-(a:Person) 
      RETURN m.title as movie, collect(a.name) as cast LIMIT {limit}")
    List<Map<String,Object>> graph(@Param("limit") int limit);
} 
```

正如您所看到的，存储库包含一些自定义操作以及从基类继承的标准操作。

接下来我们有更简单的`PersonRepository`，它只有标准操作:

```java
@Repository
public interface PersonRepository extends Neo4jRepository <Person, Long> {
    //
}
```

你可能已经注意到了`PersonRepository`只是标准的 Spring 数据接口。这是因为，在这个简单的例子中，基本上使用内置操作就足够了，因为我们的操作集与`Movie`实体相关。但是，您可以在这里添加自定义操作，这可能会包装单个/多个内置操作。

### 4.2。配置 Neo4j`Repositories`

下一步，我们必须让 Spring 知道在第 3 节创建的`Neo4jConfiguration`类中指示它的相关存储库:

```java
@Configuration
@ComponentScan("com.baeldung.spring.data.neo4j")
@EnableNeo4jRepositories(
  basePackages = "com.baeldung.spring.data.neo4j.repository")
public class MovieDatabaseNeo4jConfiguration {
    //
}
```

## 5。完整的数据模型

我们已经开始查看数据模型，所以现在让我们把它全部展开——完整的`Movie, Role` 和`Person`。`Person`实体通过`Role` 关系引用`Movie` 实体。

```java
@NodeEntity
public class Movie {

    @Id @GeneratedValue
    Long id;

    private String title;

    private int released;

    private String tagline;

    @Relationship(type="ACTED_IN", direction = Relationship.INCOMING)

    private List<Role> roles;

    // standard constructor, getters and setters 
}
```

注意我们是如何用`@NodeEntity`注释`Movie` 来表示这个类直接映射到 Neo4j 中的一个节点。

```java
@JsonIdentityInfo(generator=JSOGGenerator.class)
@NodeEntity
public class Person {

    @Id @GeneratedValue
    Long id;

    private String name;

    private int born;

    @Relationship(type = "ACTED_IN")
    private List<Movie> movies;

    // standard constructor, getters and setters 
}

@JsonIdentityInfo(generator=JSOGGenerator.class)
@RelationshipEntity(type = "ACTED_IN")
public class Role {

    @Id @GeneratedValue
    Long id;

    private Collection<String> roles;

    @StartNode
    private Person person;

    @EndNode
    private Movie movie;

    // standard constructor, getters and setters 
}
```

当然，这最后几个类也有类似的注释，并且-movies `–`引用通过“ACTED_IN”关系将`Person`链接到`Movie` 类。

## `**6\. Data Access Using MovieRepository**`

### 6.1。保存新的电影对象

让我们保存一些数据——首先是一部新电影，然后是一个人，当然还有一个角色——包括我们拥有的所有关系数据:

```java
Movie italianJob = new Movie();
italianJob.setTitle("The Italian Job");
italianJob.setReleased(1999);
movieRepository.save(italianJob);

Person mark = new Person();
mark.setName("Mark Wahlberg");
personRepository.save(mark);

Role charlie = new Role();
charlie.setMovie(italianJob);
charlie.setPerson(mark);
Collection<String> roleNames = new HashSet();
roleNames.add("Charlie Croker");
charlie.setRoles(roleNames);
List<Role> roles = new ArrayList();
roles.add(charlie);
italianJob.setRoles(roles);
movieRepository.save(italianJob);
```

### 6.2。通过标题检索现有电影对象

现在，让我们通过使用定义的标题检索插入的电影来验证它，这是一个自定义操作:

```java
Movie result = movieRepository.findByTitle(title);
```

### 6.3。通过标题的一部分检索现有的电影对象

可以使用标题的一部分搜索现有电影:

```java
Collection<Movie> result = movieRepository.findByTitleContaining("Italian");
```

### 6.4。检索所有电影

所有电影都可以检索一次，并且可以检查正确的计数:

```java
Collection<Movie> result = (Collection<Movie>) movieRepository.findAll();
```

然而，有许多 find 方法提供了默认行为，这对于定制需求是有用的，这里没有全部描述。

### 6.5。统计现有的电影对象

插入几个电影对象后，我们可以得到退出的电影数量:

```java
long movieCount = movieRepository.count();
```

### 6.6。删除现有电影

```java
movieRepository.delete(movieRepository.findByTitle("The Italian Job"));
```

删除插入的电影后，我们可以搜索电影对象并验证结果为空:

```java
assertNull(movieRepository.findByTitle("The Italian Job"));
```

### 6.7。删除所有插入的数据

可以删除数据库中的所有元素，使数据库变空:

```java
movieRepository.deleteAll();
```

该操作的结果是从表中快速删除所有数据。

## 7 .**。结论**

在本教程中，我们使用一个非常简单的例子来介绍 Spring 数据 Neo4j 的基础知识。

然而，Neo4j 能够满足非常高级和复杂的应用程序的需求，这些应用程序具有大量的关系和网络。Spring Data Neo4j 还提供了将带注释的实体类映射到 Neo4j 图形数据库的高级特性。

上述代码片段和示例的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，因此应该很容易导入和运行。