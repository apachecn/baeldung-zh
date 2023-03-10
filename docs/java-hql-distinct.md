# HQL 的独特查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hql-distinct>

## 1。概述

在本文中，我们将讨论不同的 HQL 查询，以及如何避免在 SQL 查询中添加不必要的关键字`distinct`。

## 2.理解问题

首先，让我们看看我们的数据模型，并确定我们试图完成什么。

我们将使用`Post`和`Comment`实体对象，它们共享一对多关系。我们想要检索一个包含所有相关评论的帖子列表。

让我们首先尝试下面的 HQL 查询:

```java
String hql = "SELECT p FROM Post p LEFT JOIN FETCH p.comments";
List<Post> posts = session.createQuery(hql, Post.class).getResultList();
```

这将生成以下 SQL 查询:

```java
select
    post0_.id as id1_3_0_,
    comment2_.id as id1_0_1_,
    post0_.title as title2_3_0_,
    comment2_.text as text2_0_1_,
    comments1_.Post_id as Post_id1_4_0__,
    comments1_.comments_id as comments2_4_0__
from
    Post post0_
left outer join
    Post_Comment comments1_
        on post0_.id=comments1_.Post_id
left outer join
    Comment comment2_
        on comments1_.comments_id=comment2_.id
```

**结果将包含重复项。** **一个`Post`会被显示多少次，有三个`Comments`的关联`Comments`–**`Post`会在结果列表中出现三次。

## 3.在 HQL 查询中使用`distinct`

我们需要在 HQL 查询中使用关键字`distinct` 来消除重复:

```java
String hql = "SELECT DISTINCT p FROM Post p LEFT JOIN FETCH p.comments";
List<Post> posts = session.createQuery(hql, Post.class).getResultList();
```

现在，我们得到了正确的结果:不再有重复的`Post`对象。让我们来看看 Hibernate 生成的 SQL 语句:

```java
select
    distinct post0_.id as id1_3_0_,
    comment2_.id as id1_0_1_,
    post0_.title as title2_3_0_,
    comment2_.text as text2_0_1_,
    comments1_.Post_id as Post_id1_4_0__,
    comments1_.comments_id as comments2_4_0__
from
    Post post0_
left outer join
    Post_Comment comments1_
        on post0_.id=comments1_.Post_id
left outer join
    Comment comment2_
        on comments1_.comments_id=comment2_.id 
```

我们可以注意到，`distinct`关键字不仅被 Hibernate 使用，也包含在 SQL 查询中。**我们应该避免这样做，因为这是不必要的，会导致性能问题。**

## 4.使用`QueryHint` 来停止传递`distinct`关键字

**从 Hibernate 5.2 开始，我们可以利用`pass-distinct-through`机制不再传递 SQL 语句的 HQL/JPQL `distinct`子句。**

要禁用`pass-distinct-through`，我们需要在查询中添加值为`false`的提示`QueryHint.PASS_DISTINCT_THROUGH,` :

```java
String hql = "SELECT DISTINCT p FROM Post p LEFT JOIN FETCH p.comments";
List<Post> posts = session.createQuery(hql, Post.class)
  .setHint(QueryHints.PASS_DISTINCT_THROUGH, false)
  .getResultList();
```

如果我们检查结果，我们将看到没有重复的实体。此外，SQL 语句中没有使用`distinct`子句:

```java
select
    post0_.id as id1_3_0_,
    comment2_.id as id1_0_1_,
    post0_.title as title2_3_0_,
    comment2_.text as text2_0_1_,
    comments1_.Post_id as Post_id1_4_0__,
    comments1_.comments_id as comments2_4_0__ 
from
    Post post0_ 
left outer join
    Post_Comment comments1_ 
        on post0_.id=comments1_.Post_id 
left outer join
    Comment comment2_ 
        on comments1_.comments_id=comment2_.id
```

## 5.结论

在本文中，我们发现 SQL 查询中出现的关键字`distinct `可能是不必要的，并且它会影响性能。之后，我们学习了如何使用`PASS_DISTINCT_THROUGH`查询提示来避免这种行为。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-queries)