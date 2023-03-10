# 使用聚合函数定制 JPA 查询的结果

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-queries-custom-result-with-aggregation-functions>

## 1.概观

虽然 Spring Data JPA 可以抽象查询的创建，以便在特定情况下从数据库中检索实体，但是我们有时需要定制我们的查询，例如当我们添加聚合函数时。

在本教程中，我们将关注如何将这些查询的结果转换成一个对象。我们将探索两种不同的解决方案，一种涉及 JPA 规范和 POJO，另一种使用 Spring 数据投影。

## 2.JPA 查询和聚集问题

JPA 查询通常以映射实体的实例形式产生结果。然而，**带有聚合函数的查询通常返回结果为`Object[]`** 。

为了理解问题，让我们根据帖子和评论的关系定义一个领域模型:

```java
@Entity
public class Post {
    @Id
    private Integer id;
    private String title;
    private String content;
    @OneToMany(mappedBy = "post")
    private List comments;

    // additional properties
    // standard constructors, getters, and setters
}

@Entity
public class Comment {
    @Id
    private Integer id;
    private Integer year;
    private boolean approved;
    private String content;
    @ManyToOne
    private Post post;

    // additional properties
    // standard constructors, getters, and setters
}
```

我们的模型定义了一篇文章可以有很多评论，每个评论属于一篇文章。让我们在这个模型中使用一个 [Spring 数据仓库](/web/20221205063231/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa):

```java
@Repository
public interface CommentRepository extends JpaRepository<Comment, Integer> {
    // query methods
}
```

现在我们来统计一下按年份分组的评论:

```java
@Query("SELECT c.year, COUNT(c.year) FROM Comment AS c GROUP BY c.year ORDER BY c.year DESC")
List<Object[]> countTotalCommentsByYear();
```

**前一个 JPA 查询的结果不能加载到`Comment`的实例中，因为结果是不同的形状。**查询中指定的`year`和`COUNT`与我们的实体对象不匹配。

虽然我们仍然可以访问列表中返回的通用的`Object[]`中的结果，但是这样做会导致混乱的、容易出错的代码。

## 3.用类构造函数定制结果

JPA 规范允许我们以面向对象的方式定制结果。因此，我们可以使用一个 JPQL 构造函数表达式来设置结果:

```java
@Query("SELECT new com.baeldung.aggregation.model.custom.CommentCount(c.year, COUNT(c.year)) "
  + "FROM Comment AS c GROUP BY c.year ORDER BY c.year DESC")
List<CommentCount> countTotalCommentsByYearClass();
```

这将`SELECT`语句的输出绑定到一个 POJO。**指定的类需要有一个与投影属性完全匹配的构造函数，但不要求用`@Entity`注释。**

我们还可以看到，JPQL 中声明的构造函数必须有一个完全限定的名称:

```java
package com.baeldung.aggregation.model.custom;

public class CommentCount {
    private Integer year;
    private Long total;

    public CommentCount(Integer year, Long total) {
        this.year = year;
        this.total = total;
    }
    // getters and setters
}
```

## 4.使用弹簧数据投影自定义结果

另一个可能的解决方案是用 [Spring 数据投影](/web/20221205063231/https://www.baeldung.com/spring-data-jpa-projections)定制 JPA 查询的结果。这个功能**允许我们用少得多的代码投射查询结果**。

### 4.1.定制 JPA 查询的结果

要使用基于接口的投影，我们必须定义一个由匹配投影属性名的 getter 方法组成的 Java 接口。让我们为查询结果定义一个接口:

```java
public interface ICommentCount {
    Integer getYearComment();
    Long getTotalComment();
}
```

现在让我们用返回的结果`List<ICommentCount>`来表达我们的查询:

```java
@Query("SELECT c.year AS yearComment, COUNT(c.year) AS totalComment "
  + "FROM Comment AS c GROUP BY c.year ORDER BY c.year DESC")
List<ICommentCount> countTotalCommentsByYearInterface();
```

为了允许 Spring 将投影值绑定到我们的接口，我们需要用接口中的属性名为每个投影属性提供别名。

然后，Spring Data 将动态地构造结果，并为结果的每一行返回一个代理实例。

### 4.2.自定义本机查询的结果

我们可能会遇到 JPA 查询没有原生 SQL 快，或者不能使用我们数据库引擎的特定特性的情况。为了解决这个问题，我们将使用本地查询。

**基于接口的投影的一个优点是我们可以将其用于本地查询。**让我们再次使用`ICommentCount`，并将其绑定到一个 SQL 查询:

```java
@Query(value = "SELECT c.year AS yearComment, COUNT(c.*) AS totalComment "
  + "FROM comment AS c GROUP BY c.year ORDER BY c.year DESC", nativeQuery = true)
List<ICommentCount> countTotalCommentsByYearNative();
```

这与 JPQL 查询的工作方式相同。

## 5.结论

在本文中，**我们评估了两种不同的解决方案来解决用聚合函数映射 JPA 查询结果的问题。**首先，我们使用了包含 POJO 类的 JPA 标准。

对于第二个解决方案，我们使用带有接口的轻量级 Spring 数据投影。Spring 数据投影允许我们用 Java 和 JPQL 编写更少的代码。

和往常一样，本文的示例代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221205063231/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query)