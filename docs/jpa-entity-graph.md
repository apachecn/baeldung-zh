# JPA 实体图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-entity-graph>

## 1.概观

JPA 2.1 引入了实体图特性，作为一种更复杂的处理性能加载的方法。

它允许通过对我们想要检索的相关持久性字段进行分组来定义模板，并允许我们在运行时选择图形类型。

在本教程中，我们将更详细地解释如何创建和使用这个功能。

## 2.实体图试图解决的问题

在 JPA 2.0 之前，为了加载实体关联，我们通常使用`FetchType.` `LAZY`和`FetchType.` `EAGER `作为获取策略`. `，这指示 JPA 提供者是否额外获取相关的关联。**不幸的是，这个元配置是静态的**，不允许在运行时在这两种策略之间切换。

JPA 实体图的主要目标是在加载实体的相关关联和基本字段时提高运行时性能。

简而言之，JPA 提供者在一个选择查询中加载所有的图，然后避免提取与更多选择查询的关联。这被认为是提高应用程序性能的好方法。

## 3.定义模型

在我们开始探索实体图之前，我们需要定义我们正在处理的模型实体。假设我们想要创建一个博客站点，用户可以在那里评论和分享帖子。

所以，首先我们会有一个`User`实体:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;

    //...
}
```

用户可以共享各种帖子，所以我们还需要一个`Post`实体:

```java
@Entity
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String subject;
    @OneToMany(mappedBy = "post")
    private List<Comment> comments = new ArrayList<>();

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    private User user;

    //...
}
```

用户还可以对共享的帖子发表评论，所以，最后，我们将添加一个`Comment`实体:

```java
@Entity
public class Comment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String reply;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    private Post post;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    private User user;

    //...
}
```

如我们所见，`Post`实体与`Comment`和`User`实体有关联。`Comment`实体与`Post`和`User`实体有关联。

目标是使用各种方式加载下图:

```java
Post  ->  user:User
      ->  comments:List<Comment>
            comments[0]:Comment -> user:User
            comments[1]:Comment -> user:User
```

## 4.用`FetchType`策略加载相关实体

`FetchType`方法定义了两种从数据库获取数据的策略:

*   **`FetchType.EAGER` :** 持久性提供者必须加载相关的带注释的字段或属性。这是`@Basic, @ManyToOne`和`@OneToOne `注释字段的默认行为。
*   **`FetchType.LAZY` :** 持久性提供者应该在第一次被访问时加载数据，但也可能被急切地加载。这是`@OneToMany, @ManyToMany`和`@ElementCollection-`注释字段的默认行为。

例如，当我们加载一个`Post`实体时，相关的`Comment`实体不会作为默认的`FetchType`被加载，因为`@OneToMany`是`LAZY.`，我们可以通过将`FetchType`改为`EAGER:`来覆盖这个行为

```java
@OneToMany(mappedBy = "post", fetch = FetchType.EAGER)
private List<Comment> comments = new ArrayList<>();
```

相比之下，当我们加载一个`Comment`实体时，他的`Post`父实体被加载为`@ManyToOne, `的默认模式，也就是`EAGER.`，我们也可以通过将此注释改为`LAZY:`来选择不加载`Post`实体

```java
@ManyToOne(fetch = FetchType.LAZY) 
@JoinColumn(name = "post_id") 
private Post post;
```

**注意，由于`LAZY`不是必需的，如果需要，持久性提供者仍然可以急切地加载`Post`实体。**因此，为了正确使用这个策略，我们应该回到相应的持久性提供者的官方文档。

现在，因为我们已经使用注释来描述我们的获取策略，**我们的定义是静态的，在运行时**没有办法在`LAZY`和`EAGER`之间切换。

这就是实体图发挥作用的地方，我们将在下一节中看到。

## 5.定义实体图

要定义一个实体图，我们可以使用实体上的注释，也可以使用 JPA API 以编程方式进行。

### 5.1.定义带有注释的实体图

当我们想要加载实体和相关的关联时,@ `NamedEntityGraph`注释允许指定要包含的属性。

所以让我们首先定义一个实体图，它加载了`Post` 和他的相关实体`User` 和`Comment` s:

```java
@NamedEntityGraph(
  name = "post-entity-graph",
  attributeNodes = {
    @NamedAttributeNode("subject"),
    @NamedAttributeNode("user"),
    @NamedAttributeNode("comments"),
  }
)
@Entity
public class Post {

    @OneToMany(mappedBy = "post")
    private List<Comment> comments = new ArrayList<>();

    //...
}
```

在这个例子中，我们使用了`@NamedAttributeNode`来定义在加载根实体时要加载的相关实体。

现在让我们定义一个更复杂的实体图，其中我们还希望加载与`Comment`相关的`User`。

为此，我们将使用`@NamedAttributeNode `子图属性。**这允许引用通过`@NamedSubgraph`注释:**定义的命名子图

```java
@NamedEntityGraph(
  name = "post-entity-graph-with-comment-users",
  attributeNodes = {
    @NamedAttributeNode("subject"),
    @NamedAttributeNode("user"),
    @NamedAttributeNode(value = "comments", subgraph = "comments-subgraph"),
  },
  subgraphs = {
    @NamedSubgraph(
      name = "comments-subgraph",
      attributeNodes = {
        @NamedAttributeNode("user")
      }
    )
  }
)
@Entity
public class Post {

    @OneToMany(mappedBy = "post")
    private List<Comment> comments = new ArrayList<>();
    //...
}
```

`@NamedSubgraph`注释的定义与@ `NamedEntityGraph`相似，允许指定相关关联的属性。这样做，我们可以构建一个完整的图形。

在上面的例子中，通过定义的'`post-entity-graph-with-comment-users'`图，我们可以加载与`Post,`相关的 `User,` 、`Comments`和与`Comments.`相关的`User`

最后，请注意，我们也可以使用`orm.xml`部署描述符添加实体图的定义:

```java
<entity-mappings>
  <entity class="com.baeldung.jpa.entitygraph.Post" name="Post">
    ...
    <named-entity-graph name="post-entity-graph">
            <named-attribute-node name="comments" />
    </named-entity-graph>
  </entity>
  ...
</entity-mappings>
```

### 5.2.用 JPA API 定义实体图

我们也可以通过调用`createEntityGraph()`方法，通过`EntityManager` API 定义实体图:

```java
EntityGraph<Post> entityGraph = entityManager.createEntityGraph(Post.class);
```

为了指定根实体的属性，我们使用了`addAttributeNodes()`方法。

```java
entityGraph.addAttributeNodes("subject");
entityGraph.addAttributeNodes("user");
```

类似地，为了包含相关实体的属性，我们使用`addSubgraph()`来构建一个嵌入的实体图，然后像上面一样使用`addAttributeNodes() `。

```java
entityGraph.addSubgraph("comments")
  .addAttributeNodes("user");
```

既然我们已经看到了如何创建实体图，我们将在下一节探索如何使用它。

## 6.使用实体图

### 6.1.实体图的类型

JPA 定义了两个属性或提示，持久性提供者可以通过它们进行选择，以便在运行时加载或获取实体图:

*   `javax.persistence.fetchgraph – `仅从数据库中检索指定的属性。**由于我们在本教程中使用 Hibernate，我们可以注意到，与 JPA 规范相反，静态配置为`EAGER`的属性也被加载。**
*   `javax.persistence.loadgraph – `除了指定的属性，还检索静态配置为`EAGER`的属性。

**在任何一种情况下，主键和版本(如果有的话)总是被加载。**

### 6.2.加载实体图

我们可以使用各种方法检索实体图。

**让我们从使用`EntityManager.find`()方法开始。**正如我们已经展示的，默认模式是基于静态元策略`FetchType.EAGER`和`FetchType.LAZY`。

因此，让我们调用`find()`方法并检查日志:

```java
Post post = entityManager.find(Post.class, 1L);
```

下面是 Hibernate 实现提供的日志:

```java
select
    post0_.id as id1_1_0_,
    post0_.subject as subject2_1_0_,
    post0_.user_id as user_id3_1_0_ 
from
    Post post0_ 
where
    post0_.id=?
```

正如我们从日志中看到的，没有加载`User`和`Comment`实体。

我们可以通过调用重载的`find()`方法来覆盖这个默认行为，该方法将提示作为`Map.`接受，然后**可以提供我们想要加载的图形类型:**

```java
EntityGraph entityGraph = entityManager.getEntityGraph("post-entity-graph");
Map<String, Object> properties = new HashMap<>();
properties.put("javax.persistence.fetchgraph", entityGraph);
Post post = entityManager.find(Post.class, id, properties);
```

如果我们再次查看日志，我们可以看到这些实体现在已经加载，并且只在一个 select 查询中加载:

```java
select
    post0_.id as id1_1_0_,
    post0_.subject as subject2_1_0_,
    post0_.user_id as user_id3_1_0_,
    comments1_.post_id as post_id3_0_1_,
    comments1_.id as id1_0_1_,
    comments1_.id as id1_0_2_,
    comments1_.post_id as post_id3_0_2_,
    comments1_.reply as reply2_0_2_,
    comments1_.user_id as user_id4_0_2_,
    user2_.id as id1_2_3_,
    user2_.email as email2_2_3_,
    user2_.name as name3_2_3_ 
from
    Post post0_ 
left outer join
    Comment comments1_ 
        on post0_.id=comments1_.post_id 
left outer join
    User user2_ 
        on post0_.user_id=user2_.id 
where
    post0_.id=?
```

**让我们看看如何使用 JPQL 实现同样的事情:**

```java
EntityGraph entityGraph = entityManager.getEntityGraph("post-entity-graph-with-comment-users");
Post post = entityManager.createQuery("select p from Post p where p.id = :id", Post.class)
  .setParameter("id", id)
  .setHint("javax.persistence.fetchgraph", entityGraph)
  .getSingleResult();
```

**最后，我们来看一个`Criteria` API 的例子:**

```java
EntityGraph entityGraph = entityManager.getEntityGraph("post-entity-graph-with-comment-users");
CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
CriteriaQuery<Post> criteriaQuery = criteriaBuilder.createQuery(Post.class);
Root<Post> root = criteriaQuery.from(Post.class);
criteriaQuery.where(criteriaBuilder.equal(root.<Long>get("id"), id));
TypedQuery<Post> typedQuery = entityManager.createQuery(criteriaQuery);
typedQuery.setHint("javax.persistence.loadgraph", entityGraph);
Post post = typedQuery.getSingleResult();
```

在每一个例子中，**图的类型是作为提示给出的**。在第一个例子中，我们使用了`Map,`,而在后面的两个例子中，我们使用了`setHint()`方法。

## 7.结论

在本文中，我们探索了如何使用 JPA 实体图动态获取一个`Entity`及其关联。

决定是在运行时做出的，在运行时我们选择加载或不加载相关的关联。

性能显然是设计 JPA 实体时要考虑的一个关键因素。JPA 文档建议尽可能使用`FetchType.LAZY`策略，当我们需要加载关联时使用实体图。

像往常一样，所有的代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20220626203230/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)