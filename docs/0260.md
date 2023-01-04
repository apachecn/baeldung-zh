# Blaze Persistence 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/blaze-persistence-tutorial>

## 1.介绍

在本教程中，我们将讨论在 Spring Boot 应用程序中使用 [Blaze Persistence](https://web.archive.org/web/20221211080952/https://github.com/Blazebit/blaze-persistence) 库。

该库为以编程方式创建 SQL 查询提供了丰富的标准 API。它允许我们应用各种过滤器、函数和逻辑条件。

我们将介绍项目设置，提供一些如何创建查询的例子，并了解如何将实体映射到 DTO 对象。

## 2.Maven 依赖性

为了在我们的项目中包含 Blaze Persistence core，我们需要在`pom.xml`文件中添加以下[三个依赖项](https://web.archive.org/web/20221211080952/https://search.maven.org/search?q=g:com.blazebit%20AND%20(a:blaze-persistence-core-api%20OR%20a:blaze-persistence-core-impl%20OR%20a:blaze-persistence-integration-hibernate-5.4)):

```
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-core-api</artifactId>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-core-impl</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-integration-hibernate-5.4</artifactId>
    <scope>runtime</scope>
</dependency>
```

根据我们使用的 Hibernate 版本，后一种依赖关系可能会有所不同。

## 3.实体模型

首先，让我们定义我们将在示例中使用的数据模型。为了自动创建表，我们将使用 Hibernate。

我们将有两个实体，`Person`和`Post`，它们使用一对多关系连接:

```
@Entity
public class Person {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    @OneToMany(mappedBy = "author")
    private Set<Post> posts = new HashSet<>();
}
```

```
@Entity
public class Post {

    @Id
    @GeneratedValue
    private Long id;

    private String title;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    private Person author;
}
```

## 4.标准 API

Blaze 持久性库是对 JPA 标准 API 的替代。这两个 API 都让我们能够在运行时定义动态查询。

然而，JPA Criteria API 在开发人员中不是很受欢迎，因为它很难读写。相比之下，Blaze Persistence 的设计更加用户友好，更易于使用。此外，它集成了各种 JPA 实现，并提供了广泛的查询功能。

### 4.1.配置

**为了使用 Blaze 持久性标准 API，我们需要在我们的配置类中定义`CriteriaBuilderFactory`bean:**

```
@Autowired
private EntityManagerFactory entityManagerFactory;

@Bean
public CriteriaBuilderFactory createCriteriaBuilderFactory() {
    CriteriaBuilderConfiguration config = Criteria.getDefault();
    return config.createCriteriaBuilderFactory(entityManagerFactory);
}
```

### 4.2.基本查询

现在，让我们从一个简单的查询开始，它从数据库中选择每一个`Post`。我们只需要两个方法调用来定义和执行一个查询:

```
List<Post> posts = builderFactory.create(entityManager, Post.class).getResultList();
```

`create`方法创建一个查询，而`getResultList`方法的调用返回查询返回的结果。

此外，在`create`方法中，`Post.class`参数有几个用途:

*   标识查询的结果类型
*   标识隐式查询根
*   为 Post 表添加隐式 SELECT 和 FROM 子句

一旦执行，该查询将生成以下 JPQL:

```
SELECT post
FROM Post post;
```

### 4.3.Where 子句

我们可以在标准构建器中添加 WHERE 子句，方法是在`create`方法之后调用`where`方法。

让我们来看看，如果一个人写了至少两篇帖子，并且年龄在 18 岁到 40 岁之间，我们会如何得到他写的帖子:

```
CriteriaBuilder<Person> personCriteriaBuilder = builderFactory.create(entityManager, Person.class, "p")
  .where("p.age")
    .betweenExpression("18")
    .andExpression("40")
  .where("SIZE(p.posts)").geExpression("2")
  .orderByAsc("p.name")
  .orderByAsc("p.id");
```

因为 Blaze Persistence 支持直接的函数调用语法，我们可以很容易地检索到与这个人相关的帖子的大小。

**此外，我们可以通过调用`whereAnd`或`whereOr`方法来定义复合谓词。它们返回构建器实例，我们可以通过一次或多次调用`where`方法来定义嵌套的复合谓词。一旦完成，我们需要调用`endAnd`或`endOr`方法来关闭复合谓词。**

例如，让我们创建一个查询，选择具有特定标题或作者姓名的文章:

```
CriteriaBuilder<Post> postCriteriaBuilder = builderFactory.create(entityManager, Post.class, "p")
  .whereOr()
    .where("p.title").like().value(title + "%").noEscape()
    .where("p.author.name").eq(authorName)
  .endOr();
```

### 4.4.From 子句

FROM 子句包含应该查询的实体。如前所述，我们可以在 create 方法中指定根实体。但是，我们可以定义 from 子句来指定根。这样，隐式创建的查询根将被删除:

```
CriteriaBuilder<Post> postCriteriaBuilder = builderFactory.create(entityManager, Post.class)
  .from(Person.class, "person")
  .select("person.posts");
```

在这个例子中，`Post.class`参数只定义了返回类型。

因为我们从不同的表中选择，所以构建器将在生成的查询中添加隐式连接:

```
SELECT posts_1 
FROM Person person 
LEFT JOIN person.posts posts_1;
```

## 5.实体视图模块

Blaze 持久化实体视图模块试图解决实体和 DTO 类之间的高效映射问题。使用这个模块，我们可以将 DTO 类定义为接口，并使用注释提供到实体类的映射。

### 5.1.Maven 依赖性

我们需要在我们的项目中包含额外的[实体-视图依赖关系](https://web.archive.org/web/20221211080952/https://search.maven.org/search?q=g:com.blazebit%20AND%20(a:blaze-persistence-entity-view-api%20OR%20a:blaze-persistence-entity-view-impl%20OR%20a:blaze-persistence-entity-view-processor)):

```
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-entity-view-api</artifactId>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-entity-view-impl</artifactId>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-entity-view-processor</artifactId>
</dependency>
```

### 5.2.配置

此外，我们需要一个注册了实体视图类的`EntityViewManager` bean:

```
@Bean
public EntityViewManager createEntityViewManager(
  CriteriaBuilderFactory criteriaBuilderFactory, EntityViewConfiguration entityViewConfiguration) {
    return entityViewConfiguration.createEntityViewManager(criteriaBuilderFactory);
}
```

**因为`EntityViewManager`同时绑定了`EntityManagerFactory`和`CriteriaBuilderFactory`，所以它的作用域应该是一样的。**

### 5.3.绘图

实体视图表示是一个简单的接口或抽象类，描述我们想要的投影结构。

让我们为`Post`类创建一个表示实体视图的接口:

```
@EntityView(Post.class)
public interface PostView {

    @IdMapping
    Long getId();

    String getTitle();

    String getContent();
}
```

**我们需要用`@EntityView`注释来注释我们的接口，并提供一个实体类。**

尽管这不是必需的，但我们应该尽可能使用`@IdMapping`注释来定义 id 映射。没有这种映射的实体视图会有考虑所有属性的`equals`和`hashCode`实现，而有了 id 映射，只考虑 id。

然而，如果我们想为 getter 方法使用不同的名称，我们可以添加一个`@Mapping`注释。使用这个注释，我们也可以定义整个表达式:

```
@Mapping("UPPER(title)")
String getTitle();
```

因此，映射将返回`Post`实体的大写标题。

此外，我们可以扩展实体视图。假设我们想要定义一个视图，该视图将返回一篇带有附加作者信息的文章。

首先，我们将定义一个`PersonView`接口:

```
@EntityView(Person.class)
public interface PersonView {

    @IdMapping
    Long getId();

    int getAge();

    String getName();
}
```

其次，让我们定义一个扩展`PostView`接口的新接口和一个返回`PersonView`信息的方法:

```
@EntityView(Post.class)
public interface PostWithAuthorView extends PostView {
    PersonView getAuthor();
}
```

最后，让我们通过标准构建器来使用视图。它们可以直接应用于查询。

我们可以定义一个基本查询，然后创建映射:

```
CriteriaBuilder<Post> postCriteriaBuilder = builderFactory.create(entityManager, Post.class, "p")
  .whereOr()
    .where("p.title").like().value("title%").noEscape()
    .where("p.author.name").eq(authorName)
  .endOr();

CriteriaBuilder<PostWithAuthorView> postWithAuthorViewCriteriaBuilder =
  viewManager.applySetting(EntityViewSetting.create(PostWithAuthorView.class), postCriteriaBuilder);
```

上面的代码将创建一个优化的查询，并基于结果构建我们的实体视图:

```
SELECT p.id AS PostWithAuthorView_id,
  p.author.id AS PostWithAuthorView_author_id,
  author_1.age AS PostWithAuthorView_author_age,
  author_1.name AS PostWithAuthorView_author_name,
  p.content AS PostWithAuthorView_content,
  UPPER(p.title) AS PostWithAuthorView_title
FROM com.baeldung.model.Post p
LEFT JOIN p.author author_1
WHERE p.title LIKE REPLACE(:param_0, '\\', '\\\\')
  OR author_1.name = :param_1
```

### 5.4.实体视图和弹簧数据

除了与 Spring 集成之外，Blaze Persistence 还提供了一个 Spring 数据集成模块，使得实体视图像使用实体一样方便。

此外，我们需要包含一个 [Spring 集成依赖项](https://web.archive.org/web/20221211080952/https://search.maven.org/search?q=g:com.blazebit%20AND%20a:blaze-persistence-integration-spring-data-2.4):

```
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-integration-spring-data-2.4</artifactId>
</dependency>
```

此外，为了启用 Spring 数据，我们需要用`@EnableBlazeRepositories`注释来注释配置类。或者，我们可以为存储库类扫描指定基础包。

**集成附带了一个基本的`EntityViewRepository`接口，我们可以用它来定义我们的存储库。**

现在，让我们定义一个与`PostWithAuthorView`一起工作的接口:

```
@Repository
@Transactional(readOnly = true)
public interface PostViewRepository extends EntityViewRepository<PostWithAuthorView, Long> {
}
```

这里，我们的接口继承了最常用的存储库方法，比如`findAll`、`findOne`和`exists`。如果需要，我们可以使用 Spring Data JPA 方法命名约定来定义自己的方法。

## 6.结论

在本文中，我们学习了如何使用 Blaze 持久性库配置和创建简单的查询。

像往常一样，所有的源代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221211080952/https://github.com/eugenp/tutorials/tree/master/persistence-modules/blaze-persistence)