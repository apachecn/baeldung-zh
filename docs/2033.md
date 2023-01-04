# Hibernate OGM 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-ogm>

## 1。概述

在本教程中，我们将介绍 [Hibernate 对象/网格映射器(OGM)](https://web.archive.org/web/20220628154233/http://hibernate.org/ogm/) 的基础知识。

Hibernate OGM 为 NoSQL 数据存储提供 Java 持久性 API (JPA)支持。NoSQL 是一个涵盖各种数据存储的总称。例如，这包括键值、文档、面向列和面向图形的数据存储。

## 2.Hibernate OGM 的体系结构

传统上，Hibernate 为关系数据库提供了一个对象关系映射(ORM)引擎。 **Hibernate OGM 引擎扩展了它的功能以支持 NoSQL 数据存储。**使用它的主要好处是跨关系和 NoSQL 数据存储的 JPA 接口的一致性。

Hibernate OGM 能够在许多 NoSQL 数据存储上提供抽象，因为有两个关键接口，`DatastoreProvider`和`GridDialect`。因此，它支持的每个新的 NoSQL 数据存储都带有这些接口的实现。

到目前为止，它并不支持所有的 NoSQL 数据存储，但它能够支持其中的许多数据存储，如 Infinispan 和 Ehcache(键值)、MongoDB 和 CouchDB(文档)以及 Neo4j(图形)。

**它还[完全支持交易](/web/20220628154233/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)，并且可以与标准 JTA 提供商合作。**首先，这可以通过 Jakarta EE 容器提供，无需任何显式配置。此外，我们可以在 Java SE 环境中使用独立的 JTA 事务管理器，如 Narayana。

## 3。设置

对于本教程，我们将使用 Maven 来提取所需的依赖项，以便与 Hibernate OGM 一起工作。我们还将使用 [MongoDB](/web/20220628154233/https://www.baeldung.com/java-mongodb) 。

为了澄清这一点，让我们看看如何在本教程中设置它们。

### 3.1.Maven 依赖性

让我们看看使用 Hibernate OGM 和 MongoDB 所需的依赖关系:

```
<dependency>
    <groupId>org.hibernate.ogm</groupId>
    <artifactId>hibernate-ogm-mongodb</artifactId>
    <version>5.4.0.Final</version>
</dependency>
<dependency>
    <groupId>org.jboss.narayana.jta</groupId>
    <artifactId>narayana-jta</artifactId>
    <version>5.9.2.Final</version>
</dependency>
```

在这里，我们通过 Maven 获取所需的依赖关系:

*   [针对 MongoDB 的 Hibernate OGM 方言](https://web.archive.org/web/20220628154233/https://search.maven.org/search?q=a:hibernate-ogm-mongodb%20AND%20g:org.hibernate.ogm)
*   [Narayana 交易经理](https://web.archive.org/web/20220628154233/https://search.maven.org/search?q=g:org.jboss.narayana.jta%20AND%20a:narayana-jta)(JTA 的实际提供者)

### 3.2.持久性单元

我们还必须**定义休眠`persistance.xml`中的数据存储细节**:

```
<persistence-unit name="ogm-mongodb" transaction-type="JTA">
    <provider>org.hibernate.ogm.jpa.HibernateOgmPersistence</provider>
    <properties>
        <property name="hibernate.ogm.datastore.provider" value="MONGODB" />
        <property name="hibernate.ogm.datastore.database" value="TestDB" />
        <property name="hibernate.ogm.datastore.create_database" value="true" />
    </properties>
</persistence-unit>
```

请注意我们在这里提供的定义:

*   属性 transaction-type 的值为“JTA”(这意味着我们需要一个来自`EntityManagerFactory)`的 JTA 实体管理器
*   提供者，对于 Hibernate OGM 是`HibernateOgmPersistence`
*   一些与数据库相关的附加细节(这些细节通常在不同的数据源之间有所不同)

默认情况下，该配置假设 MongoDB 正在运行并可访问。如果不是这样，我们可以随时[提供必要的细节](https://web.archive.org/web/20220628154233/https://docs.jboss.org/hibernate/stable/ogm/reference/en-US/html_single/#_configuring_mongodb)。我们之前的一篇文章也详细介绍了[如何设置 MongoDB](/web/20220628154233/https://www.baeldung.com/java-mongodb)。

## 4。实体定义

现在我们已经完成了基础知识，让我们定义一些实体。**如果我们以前使用过 Hibernate ORM 或 JPA，这就没什么可补充的了**。这是 Hibernate OGM 的根本前提。它**承诺让我们仅用 JPA** 的知识就能处理不同的 NoSQL 数据存储库。

对于本教程，我们将定义一个简单的对象模型:

[![Domain Model](img/787ad9ba5f52bbb3318f2e9a4bbc9fd2.png)](/web/20220628154233/https://www.baeldung.com/wp-content/uploads/2018/12/Domain-Model.jpg)

它定义了`Article`、`Author`和`Editor`类以及它们之间的关系。

让我们也用 Java 来定义它们:

```
@Entity
public class Article {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String articleId;

    private String articleTitle;

    @ManyToOne
    private Author author;

    // constructors, getters and setters...
}
```

```
@Entity
public class Author {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String authorId;

    private String authorName;

    @ManyToOne
    private Editor editor;

    @OneToMany(mappedBy = "author", cascade = CascadeType.PERSIST)
    private Set<Article> authoredArticles = new HashSet<>();

    // constructors, getters and setters...
}
```

```
@Entity
public class Editor {
    @Id
    @GeneratedValue(generator = "uuid")
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    private String editorId;

    private String editorName;
    @OneToMany(mappedBy = "editor", cascade = CascadeType.PERSIST)
    private Set<Author> assignedAuthors = new HashSet<>();

    // constructors, getters and setters...
}
```

我们现在已经定义了实体类，并用 JPA 标准注释对它们进行了注释:

*   `@Entity`将其建立为 JPA 实体
*   `@Id`为具有 UUIDs 的实体生成主键
*   `@OneToMany`和`@ManyToOne`建立实体间的双向关系

## 5。操作

现在我们已经创建了实体，让我们看看是否可以对它们执行一些操作。作为第一步，我们必须生成一些测试数据。在这里，我们将创建一个`Editor`，几个`Author`，和一些`Article.`，我们还将建立他们的关系。

此后，在我们执行任何操作之前，我们需要一个`EntityManagerFactory.`的实例，我们可以用它来创建`EntityManager`。与此同时，我们需要创建`TransactionManager`来处理事务边界。

让我们看看如何使用这些来保存和检索我们之前创建的实体:

```
private void persistTestData(EntityManagerFactory entityManagerFactory, Editor editor) 
  throws Exception {
    TransactionManager transactionManager = 
      com.arjuna.ats.jta.TransactionManager.transactionManager();
    transactionManager.begin();
    EntityManager entityManager = entityManagerFactory.createEntityManager();

    entityManager.persist(editor);
    entityManager.close();
    transactionManager.commit();
}
```

这里，我们使用`EntityManager`来持久化根实体，它级联到它的所有关系。我们还在定义的事务边界内执行这个操作。

现在，我们准备加载刚刚持久化的实体，并验证其内容。我们可以运行一个测试来验证这一点:

```
@Test
public void givenMongoDB_WhenEntitiesCreated_thenCanBeRetrieved() throws Exception {
    EntityManagerFactory entityManagerFactory = 
      Persistence.createEntityManagerFactory("ogm-mongodb");
    Editor editor = generateTestData();
    persistTestData(entityManagerFactory, editor);

    TransactionManager transactionManager = 
      com.arjuna.ats.jta.TransactionManager.transactionManager();  
    transactionManager.begin();
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    Editor loadedEditor = entityManager.find(Editor.class, editor.getEditorId());

    assertThat(loadedEditor).isNotNull();
    // Other assertions to verify the entities and relations
}
```

这里，我们再次使用`EntityManager`来查找数据并对其执行标准断言。当我们运行这个测试时，它实例化数据存储、持久化实体、检索它们并进行验证。

同样，我们已经**使用 JPA 来持久化实体及其关系**。类似地，**我们使用 JPA 将实体加载回**，这一切都运行良好，即使我们选择的数据库是 MongoDB 而不是传统的关系数据库。

## 6.切换后端

我们也可以切换我们的后端。现在让我们来看看做这件事有多难。

我们将把后端改为 [Neo4j，这恰好是一个流行的面向图形的数据存储库](/web/20220628154233/https://www.baeldung.com/java-neo4j)。

首先，让我们为 Neo4j 添加 [Maven 依赖项:](https://web.archive.org/web/20220628154233/https://search.maven.org/search?q=a:hibernate-ogm-neo4j)

```
<dependency>
    <groupId>org.hibernate.ogm</groupId>
    <artifactId>hibernate-ogm-neo4j</artifactId>
    <version>5.4.0.Final</version>
</dependency>
```

接下来，我们必须在我们的`persistence.xml`中添加相关的持久性单元:

```
<persistence-unit name="ogm-neo4j" transaction-type="JTA">
    <provider>org.hibernate.ogm.jpa.HibernateOgmPersistence</provider>
    <properties>
        <property name="hibernate.ogm.datastore.provider" value="NEO4J_EMBEDDED" />
        <property name="hibernate.ogm.datastore.database" value="TestDB" />
        <property name="hibernate.ogm.neo4j.database_path" value="target/test_data_dir" />
    </properties>
</persistence-unit>
```

简而言之，这些是 Neo4j 需要的非常基本的配置。这可以根据需要进一步详细说明[。](https://web.archive.org/web/20220628154233/https://docs.jboss.org/hibernate/stable/ogm/reference/en-US/html_single/#_configuring_neo4j)

嗯，这就是我们需要做的。当我们使用 Neo4j 作为后端数据存储运行相同的测试时，它可以非常无缝地工作。

请注意，我们已经将后端从 MongoDB(一种面向文档的数据存储)切换到 Neo4j(一种面向图形的数据存储)。**我们以最小的改动完成了这一切，并且不需要对我们的任何运营进行任何改动**。

## 7。结论

在本文中，我们已经介绍了 Hibernate OGM 的基础知识，包括它的架构。随后，我们实现了一个基本的领域模型，并使用各种数据库执行了各种操作。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628154233/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-ogm)