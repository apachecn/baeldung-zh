# Querydsl 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intro-to-querydsl>

## 1。简介

这是一篇介绍性文章，让您开始使用强大的用于数据持久性的 [Querydsl](https://web.archive.org/web/20221208143841/http://www.querydsl.com/) API。

这里的目标是为您提供实用的工具来将 Querydsl 添加到您的项目中，理解生成的类的结构和用途，并对如何为大多数常见场景编写类型安全的数据库查询有一个基本的了解。

## 2。Querydsl 的用途

对象关系映射框架是企业 Java 的核心。这些弥补了面向对象方法和关系数据库模型之间的不匹配。它们还允许开发人员编写更干净、更简洁的持久性代码和领域逻辑。

然而，ORM 框架最困难的设计选择之一是用于构建正确和类型安全查询的 API。

使用最广泛的 Java ORM 框架之一 Hibernate(也是密切相关的 JPA 标准)提出了一种非常类似于 SQL 的基于字符串的查询语言 HQL (JPQL)。这种方法的明显缺点是缺乏类型安全性和静态查询检查。此外，在更复杂的情况下(例如，当查询需要根据某些条件在运行时构建时)，构建 HQL 查询通常涉及字符串的连接，这通常非常不安全且容易出错。

JPA 2.0 标准以 [Criteria Query API](https://web.archive.org/web/20221208143841/https://docs.oracle.com/javaee/7/tutorial/persistence-criteria.htm#GJITV) 的形式带来了改进——这是一种新的类型安全的查询构建方法，利用了注释预处理过程中生成的元模型类。不幸的是，从本质上来说是开创性的，Criteria Query API 最终变得非常冗长，几乎不可读。这里有一个来自 Jakarta EE 教程的例子，用于生成一个像`SELECT p FROM Pet p`一样简单的查询:

```java
EntityManager em = ...;
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.select(pet);
TypedQuery<Pet> q = em.createQuery(cq);
List<Pet> allPets = q.getResultList();
```

难怪一个更合适的 Querydsl 库很快出现了，它基于相同的生成元数据类的思想，但是用流畅可读的 API 实现。

## 3。Querydsl 类生成

让我们从生成和探索神奇的元类开始，这些元类解释了 Querydsl 的流畅 API。

### 3.1。将 Querydsl 添加到 Maven 构建中

将 Querydsl 包含在项目中非常简单，只需在构建文件中添加几个依赖项，并配置一个插件来处理 JPA 注释。让我们从依赖项开始。Querydsl 库的版本应该被提取到`<project><properties>`部分中的一个单独的属性中，如下所示(对于 Querydsl 库的最新版本，请查看 [Maven Central](https://web.archive.org/web/20221208143841/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.querydsl%22) 资源库):

```java
<properties>
    <querydsl.version>4.1.3</querydsl.version>
</properties>
```

接下来，将以下依赖项添加到您的`pom.xml`文件的`<project><dependencies>`部分:

```java
<dependencies>

    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
        <version>${querydsl.version}</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
        <version>${querydsl.version}</version>
    </dependency>

</dependencies>
```

`querydsl-apt`依赖项是一个注释处理工具(APT)——实现相应的 Java API，允许在源文件中的注释进入编译阶段之前对其进行处理。这个工具生成所谓的 Q-types——与应用程序的实体类直接相关的类，但是以字母 Q 为前缀。例如，如果您的应用程序中有一个标有`@Entity`注释的`User`类，那么生成的 Q-type 将驻留在一个`QUser.java`源文件中。

`querydsl-apt`依赖项的`provided`范围意味着这个 jar 应该只在构建时可用，而不是包含在应用程序工件中。

querydsl-jpa 库是 querydsl 本身，设计用于与 jpa 应用程序一起使用。

要配置利用`querydsl-apt`的注释处理插件，将以下插件配置添加到 pom 中——在`<project><build><plugins>`元素内:

```java
<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这个插件确保在 Maven 构建的过程目标期间生成 Q 类型。`outputDirectory`配置属性指向将生成 Q 类型源文件的目录。这个属性的值将在以后您研究 Q 文件时有用。

如果您的 IDE 不能自动将此目录添加到项目的源文件夹中，那么您也应该将此目录添加到项目的源文件夹中—请查阅您最喜欢的 IDE 的文档，了解如何完成此操作。

对于本文，我们将使用一个简单的博客服务 JPA 模型，由`Users`和它们的`BlogPosts`组成，它们之间有一对多的关系:

```java
@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String login;

    private Boolean disabled;

    @OneToMany(cascade = CascadeType.PERSIST, mappedBy = "user")
    private Set<BlogPost> blogPosts = new HashSet<>(0);

    // getters and setters

}

@Entity
public class BlogPost {

    @Id
    @GeneratedValue
    private Long id;

    private String title;

    private String body;

    @ManyToOne
    private User user;

    // getters and setters

}
```

要为您的模型生成 Q 类型，只需运行:

```java
mvn compile
```

### 3.2。探索生成的类

现在转到 apt-maven-plugin 的`outputDirectory`属性中指定的目录(在我们的例子中是`target/generated-sources/java`)。您将看到一个直接反映您的域模型的包和类结构，除了所有的类都以字母 Q 开头(在我们的例子中是`QUser`和`QBlogPost`)。

打开文件`QUser.java`。这是构建所有以`User`为根实体的查询的入口点。首先你会注意到的是`@Generated`注释，这意味着这个文件是自动生成的，不应该手动编辑。如果您更改了您的任何域模型类，您将不得不再次运行`mvn compile`来重新生成所有相应的 Q-type。

除了这个文件中出现的几个`QUser`构造函数，您还应该注意到一个`QUser`类的公共静态最终实例:

```java
public static final QUser user = new QUser("user");
```

这是一个实例，您可以在对该实体的大多数 Querydsl 查询中使用它，除非您需要编写一些更复杂的查询，比如在一个查询中连接一个表的几个不同实例。

最后需要注意的是，对于实体类的每个字段，Q-type 中都有一个对应的`*Path`字段，就像`QUser`类中的`NumberPath id`、`StringPath login`和`SetPath blogPosts`(注意 `Set`对应的字段名称是复数)。这些字段被用作我们稍后将会遇到的 fluent 查询 API 的一部分。

## 4。用 Querydsl 查询

### 4.1。简单的查询和过滤

要构建一个查询，首先我们需要一个 [`JPAQueryFactory`](https://web.archive.org/web/20221208143841/http://www.querydsl.com/static/querydsl/4.1.3/apidocs/com/querydsl/jpa/impl/JPAQueryFactory.html) 的实例，这是开始构建过程的首选方式。`JPAQueryFactory`唯一需要的是一个`EntityManager`，它应该已经可以通过`EntityManagerFactory.createEntityManager()`调用或`@PersistenceContext`注入在您的 JPA 应用程序中获得。

```java
EntityManagerFactory emf = 
  Persistence.createEntityManagerFactory("com.baeldung.querydsl.intro");
EntityManager em = entityManagerFactory.createEntityManager();
JPAQueryFactory queryFactory = new JPAQueryFactory(em);
```

现在让我们创建第一个查询:

```java
QUser user = QUser.user;

User c = queryFactory.selectFrom(user)
  .where(user.login.eq("David"))
  .fetchOne();
```

注意，我们已经定义了一个局部变量`QUser` user，并用静态实例`QUser.user`对其进行了初始化。这样做纯粹是为了简洁，或者您可以导入静态的`QUser.user`字段。

`JPAQueryFactory`的`selectFrom`方法开始构建查询。我们将`QUser`实例传递给它，并继续用`.where()`方法构建查询的条件子句。`user.login`是对我们之前见过的`QUser`类的`StringPath`字段的引用。`StringPath`对象也有`.eq()`方法，允许通过指定字段相等条件来流畅地继续构建查询。

最后，为了将值从数据库提取到持久性上下文中，我们通过调用`fetchOne()`方法来结束构建链。如果找不到对象，该方法返回`null`，但是如果有多个实体满足`.where()`条件，则抛出一个`NonUniqueResultException`。

### 4.2。排序和分组

现在让我们获取一个列表中的所有用户，按照他们的登录名升序排序。

```java
List<User> c = queryFactory.selectFrom(user)
  .orderBy(user.login.asc())
  .fetch();
```

这种语法是可行的，因为`*Path`类有`.asc()`和`.desc()`方法。您还可以为`.orderBy()`方法指定几个参数，以便根据多个字段进行排序。

现在我们来试试更难的。假设我们需要按标题对所有文章进行分组，并对重复标题进行计数。这是通过`.groupBy()`子句完成的。我们还想根据出现次数对标题进行排序。

```java
NumberPath<Long> count = Expressions.numberPath(Long.class, "c");

List<Tuple> userTitleCounts = queryFactory.select(
  blogPost.title, blogPost.id.count().as(count))
  .from(blogPost)
  .groupBy(blogPost.title)
  .orderBy(count.desc())
  .fetch();
```

我们选择了博文标题和重复次数，按标题分组，然后按总次数排序。注意，我们首先为。`select()`子句，因为我们需要在`.orderBy()`子句中引用它。

### 4.3。带有连接和子查询的复杂查询

让我们找到所有写了标题为“Hello World！”的帖子的用户对于这样的查询，我们可以使用内部连接。注意，我们已经为连接的表创建了一个别名`blogPost`，以便在`.on()`子句中引用它:

```java
QBlogPost blogPost = QBlogPost.blogPost;

List<User> users = queryFactory.selectFrom(user)
  .innerJoin(user.blogPosts, blogPost)
  .on(blogPost.title.eq("Hello World!"))
  .fetch();
```

现在，让我们尝试用子查询实现同样的功能:

```java
List<User> users = queryFactory.selectFrom(user)
  .where(user.id.in(
    JPAExpressions.select(blogPost.user.id)
      .from(blogPost)
      .where(blogPost.title.eq("Hello World!"))))
  .fetch();
```

正如我们所看到的，子查询与查询非常相似，它们也非常可读，但是它们以`JPAExpressions`工厂方法开始。为了将子查询与主查询连接起来，我们总是引用前面定义和使用的别名。

### 4.4。修改数据

`JPAQueryFactory`不仅允许构造查询，还允许修改和删除记录。让我们更改用户的登录并禁用帐户:

```java
queryFactory.update(user)
  .where(user.login.eq("Ash"))
  .set(user.login, "Ash2")
  .set(user.disabled, true)
  .execute();
```

对于不同的字段，我们可以有任意数量的`.set()`子句。`.where()`子句是不必要的，所以我们可以一次更新所有记录。

要删除符合特定条件的记录，我们可以使用类似的语法:

```java
queryFactory.delete(user)
  .where(user.login.eq("David"))
  .execute();
```

`.where()`子句也不是必需的，但是要小心，因为省略`.where()`子句会导致删除某个类型的所有实体。

你可能想知道，为什么`JPAQueryFactory`没有`.insert()`方法。这是 JPA 查询接口的一个限制。底层的 [`javax.persistence.Query.executeUpdate()`](https://web.archive.org/web/20221208143841/https://docs.oracle.com/javaee/7/api/javax/persistence/Query.html#executeUpdate--) 方法能够执行更新和删除，但不能执行插入语句。要插入数据，只需用 EntityManager 持久化实体。

如果您仍然想利用类似的 Querydsl 语法来插入数据，您应该使用驻留在 querydsl-sql 库中的 [`SQLQueryFactory`](https://web.archive.org/web/20221208143841/http://www.querydsl.com/static/querydsl/4.1.3/apidocs/com/querydsl/sql/SQLQueryFactory.html) 类。

## 5。结论

在本文中，我们发现了由 Querydsl 提供的用于持久对象操作的强大且类型安全的 API。

我们已经学会了将 Querydsl 添加到 project 中，并研究了生成的 Q 类型。我们还涵盖了一些典型的用例，并享受它们的简明性和可读性。

示例的所有源代码都可以在 [github 库](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/persistence-modules/querydsl)中找到。

最后，Querydsl 当然还提供了更多的功能，包括处理原始 SQL、非持久集合、NoSQL 数据库和全文搜索——我们将在以后的文章中探讨其中的一些功能。