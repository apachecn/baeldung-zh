# Spring Data JPA 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa>

## 1。概述

本教程将重点关注**将 Spring 数据 JPA 引入 Spring 项目，**并完全配置持久层。关于使用基于 Java 的配置和项目的基本 Maven pom 建立 Spring 上下文的分步介绍，请参见[这篇文章](/web/20220926191213/https://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration "Bootstrapping a web application with Spring 3.1 and Java based Configuration, part 1")。

## 延伸阅读:

## [带 Spring 的 JPA 指南](/web/20220926191213/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)

Setup JPA with Spring - how to set up the EntityManager factory and use the raw JPA APIs.[Read more](/web/20220926191213/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) →

## [Spring 数据中的 CrudRepository、JpaRepository 和 paging and sorting repository](/web/20220926191213/https://www.baeldung.com/spring-data-repositories)

Learn about the different flavours of repositories offered by Spring Data.[Read more](/web/20220926191213/https://www.baeldung.com/spring-data-repositories) →

## [用 Spring 和 Java 泛型简化 DAO](/web/20220926191213/https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics)

Simplify the Data Access Layer by using a single, generified DAO, which will result in **elegant data access**, no unnecessary clutter.[Read more](/web/20220926191213/https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics) →

## 2。Spring 数据生成 DAO——不再有 DAO 实现

正如我们在之前的文章中所讨论的，[DAO 层](/web/20220926191213/https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics "Removing the boilerplate from the DAO")通常由许多样板代码组成，这些代码可以而且应该被简化。这种简化的优点有很多:减少了我们需要定义和维护的工件的数量，数据访问模式的一致性，以及配置的一致性。

Spring Data 将这种简化推进了一步，**使得完全移除 DAO 实现成为可能**。DAO 的接口现在是我们唯一需要显式定义的工件。

为了开始利用 JPA 的 Spring 数据编程模型，DAO 接口需要扩展 JPA 特定的`Repository`接口，`JpaRepository`。这将使 Spring Data 能够找到这个接口，并自动为它创建一个实现。

通过扩展接口，我们获得了标准 DAO 中最相关的标准数据访问的 CRUD 方法。

## 3。自定义访问方法和查询

如前所述，**通过实现其中一个`Repository`接口，DAO 已经定义并实现了一些基本的 CRUD 方法(和查询)**。

为了定义更具体的访问方法，Spring JPA 支持很多选项:

*   只需**在接口中定义一个新方法**
*   通过使用`@Query`注释来提供实际的 **JPQL 查询**
*   在 Spring 数据中使用更高级的**规范和 Querydsl 支持**
*   通过 JPA 命名查询定义**自定义查询**

[第三个选项](https://web.archive.org/web/20220926191213/https://spring.io/blog/2011/04/26/advanced-spring-data-jpa-specifications-and-querydsl/)，规范和 Querydsl 支持，类似于 JPA 标准，但是使用了更加灵活和方便的 API。这使得整个操作更具可读性和可重用性。当处理大量固定查询时，这个 API 的优势将变得更加明显，因为我们可以通过更少的可重用块来更简洁地表达这些查询。

最后一个选项的缺点是，它要么涉及 XML，要么给域类增加了查询负担。

### 3.1。自动定制查询

当 Spring Data 创建一个新的`Repository`实现时，它会分析接口定义的所有方法，并尝试**从方法名**中自动生成查询。虽然这有一些限制，但它是一种非常强大和优雅的方式，可以毫不费力地定义新的自定义访问方法。

让我们看一个例子。如果实体有一个`name` 字段(以及 Java Bean 标准的`getName`和`setName`方法)**，我们将在 DAO 接口中定义`findByName`方法。**这将自动生成正确的查询:

```java
public interface IFooDAO extends JpaRepository<Foo, Long> {

    Foo findByName(String name);

}
```

这是一个比较简单的例子。查询创建机制支持[更多的关键字](https://web.archive.org/web/20220926191213/https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#jpa.query-methods.query-creation "Spring Data JPA - Query creation")。

如果解析器无法将属性与域对象字段相匹配，我们将看到以下异常:

```java
java.lang.IllegalArgumentException: No property nam found for type class com.baeldung.spring.data.persistence.model.Foo
```

### 3.2。手动自定义查询

现在让我们来看一个自定义查询，我们将通过`@Query`注释来定义它:

```java
@Query("SELECT f FROM Foo f WHERE LOWER(f.name) = LOWER(:name)")
Foo retrieveByName(@Param("name") String name);
```

对于更细粒度的查询创建控制，比如使用命名参数或修改现有查询，[引用](https://web.archive.org/web/20220926191213/https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#jpa.named-parameters "Spring Data JPA - Query creation additional options")是一个很好的起点。

## 4。交易配置

Spring 管理的 DAO 的实际实现确实是隐藏的，因为我们不直接使用它。然而，这是一个足够简单的实现，**`SimpleJpaRepository,`使用注释**定义了事务语义。

更明确地说，这在类级别使用了一个只读的`@Transactional`注释，然后为非只读的方法覆盖它。其余的事务语义是默认的，但是这些可以很容易地被每个方法手动覆盖。

### 4.1。异常转换仍然存在并且运行良好

现在的问题变成了:既然 Spring 数据 JPA 不依赖于旧的 ORM 模板(`JpaTemplate`，`HibernateTemplate`)，而且它们从 Spring 5 开始就被移除了，我们还会让我们的 JPA 异常被转换到 Spring 的`DataAccessException`层次结构吗？

答案当然是，我们是。**异常翻译仍然可以通过使用 DAO** 上的`@Repository`注释来实现。这个注释使 Spring bean 后处理器能够用在容器中找到的所有`PersistenceExceptionTranslator`实例通知所有的`@Repository`bean，并像以前一样提供异常翻译。

让我们用一个集成测试来验证异常转换:

```java
@Test(expected = DataIntegrityViolationException.class)
public void whenInvalidEntityIsCreated_thenDataException() {
    service.create(new Foo());
}
```

记住**异常转换是通过代理完成的。**为了让 Spring 能够围绕 DAO 类创建代理，这些类不能被声明`final`。

## 5。Spring 数据 JPA 存储库配置

为了激活 Spring JPA 存储库支持，我们可以使用`@EnableJpaRepositories`注释并指定包含 DAO 接口的包:

```java
@EnableJpaRepositories(basePackages = "com.baeldung.spring.data.persistence.repository") 
public class PersistenceConfig { 
    ...
}
```

我们可以用 XML 配置做同样的事情:

```java
<jpa:repositories base-package="com.baeldung.spring.data.persistence.repository" />
```

## 6。Java 或 XML 配置

在之前的文章中，我们已经详细讨论了如何在 Spring 中[配置 JPA。Spring 数据还利用了 Spring 对 JPA `@PersistenceContext`注释的支持。它用这个将`EntityManager`连接到负责创建实际 DAO 实现的 Spring factory bean】。](/web/20220926191213/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)

除了已经讨论过的配置，如果我们使用 XML，我们还需要包括 Spring 数据 XML 配置:

```java
@Configuration
@EnableTransactionManagement
@ImportResource("classpath*:*springDataConfig.xml")
public class PersistenceJPAConfig {
    ...
}
```

## 7 .**。Maven 依赖关系**

除了 JPA 的 Maven 配置，就像在[之前的文章](/web/20220926191213/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)中一样，我们将添加[的`spring-data-jpa`依赖项](https://web.archive.org/web/20220926191213/https://search.maven.org/search?q=g:org.springframework.data%20a:spring-data-jpa):

```java
<dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-jpa</artifactId>
   <version>2.4.0</version>
</dependency>
```

## 8.使用 Spring Boot

**我们也可以使用 [Spring Boot 启动器数据 JPA](https://web.archive.org/web/20220926191213/https://search.maven.org/search?q=a:spring-boot-starter-data-jpa) 依赖项，它会自动为我们配置`DataSource`。**

我们需要确保我们想要使用的数据库存在于类路径中。在我们的示例中，我们添加了 H2 内存数据库:

```java
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
   <version>2.7.2</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

因此，只需完成这些依赖项，我们的应用程序就可以启动并运行，并且我们可以将它用于其他数据库操作。

标准 Spring 应用程序的显式配置现在作为 Spring Boot 自动配置的一部分。

当然，我们可以通过添加定制的显式配置来修改自动配置。

Spring Boot 提供了一种简单的方法，使用`application.properties`文件中的属性来实现这一点。让我们看一个更改连接 URL 和凭据的示例:

```java
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

## 9.Spring 数据的有用工具 JPA

所有主流 Java IDEs 都支持 Spring Data JPA。让我们看看 Eclipse 和 IntelliJ IDEA 有哪些有用的工具。

如果你使用 Eclipse 作为你的 IDE，你可以安装 [Dali Java 持久性工具](https://web.archive.org/web/20220926191213/https://www.eclipse.org/webtools/dali/downloads.php)插件。它提供了 JPA 实体的 ER 图、初始化模式的 DDL 生成以及基本的逆向工程功能。此外，您可以使用 Eclipse Spring 工具套件(STS)。这将有助于验证 Spring Data JPA 存储库中的查询方法名称。

如果使用 IntelliJ IDEA，有两个选项。

IntelliJ IDEA Ultimate 支持 ER 图、用于测试 JPQL 语句的 JPA 控制台和有价值的检查。但是，这些功能在社区版中不可用。

**为了提高 IntelliJ 的工作效率，你可以安装 [JPA Buddy](https://web.archive.org/web/20220926191213/https://plugins.jetbrains.com/plugin/15075-jpa-buddy) 插件，**它提供了许多功能，包括生成 JPA 实体、Spring Data JPA 存储库、dto、初始化 DDL 脚本、Flyway 版本化迁移、Liquibase changelogs 等。另外，JPA Buddy 提供了一个高级的逆向工程工具。

最后，JPA Buddy 插件支持社区版和旗舰版。

## 10。结论

在本文中，我们使用基于 XML 和 Java 的配置，介绍了 Spring 5、JPA 2 和 Spring Data JPA(Spring Data umbrella 项目的一部分)的持久层的配置和实现。

我们讨论了定义更多**高级定制查询**，以及**事务语义**，以及带有新`jpa`名称空间的**配置的方法。最终结果是用 Spring 对数据访问进行了一次新的优雅的尝试，几乎没有实际的实现工作。**

这个 Spring Data JPA 教程的实现可以在[GitHub 项目](https://web.archive.org/web/20220926191213/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo-2 "Spring Data JPA example project")中找到。