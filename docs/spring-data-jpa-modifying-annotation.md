# Spring 数据 JPA @修改注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-modifying-annotation>

## 1.介绍

在这个简短的教程中，**我们将学习如何用 Spring 数据 JPA `@Query`注释**创建更新查询。我们将通过使用`@Modifying`注释来实现这一点。

首先，为了刷新我们的记忆，我们可以阅读[如何使用 Spring 数据 JPA](/web/20220705092758/https://www.baeldung.com/spring-data-jpa-query) 进行查询。之后，我们将深入研究`@Query`和`@Modifying`注释的使用。最后，我们将讨论在使用修改查询时如何管理我们的持久性上下文的状态。

## 2.在 Spring 数据中查询 JPA

首先，让我们回顾一下 Spring Data JPA 为查询数据库中的数据提供的三种机制:

*   查询方法
*   `@Query`注释
*   自定义存储库实现

让我们创建一个`User`类和一个匹配的 Spring 数据 JPA 存储库来说明这些机制:

```java
@Entity
@Table(name = "users", schema = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;
    private LocalDate creationDate;
    private LocalDate lastLoginDate;
    private boolean active;
    private String email;

}
```

```java
public interface UserRepository extends JpaRepository<User, Integer> {}
```

查询方法机制允许我们通过从方法名派生查询来操作数据:

```java
List<User> findAllByName(String name);
void deleteAllByCreationDateAfter(LocalDate date);
```

在本例中，我们可以找到一个按用户姓名检索用户的查询，或者一个删除创建日期在某个日期之后的用户的查询。

至于`@Query`注释，**为我们提供了在`@Query`注释**中编写特定 JPQL 或 SQL 查询的机会:

```java
@Query("select u from User u where u.email like '%@gmail.com'")
List<User> findUsersWithGmailAddress();
```

在这个代码片段中，我们可以看到一个检索拥有`@gmail.com`电子邮件地址的用户的查询。

第一种机制使我们能够检索或删除数据。至于第二种机制，它允许我们执行几乎任何查询。但是，**为了更新查询，我们必须添加`@Modifying`注释**。这将是本教程的主题。

## 3.使用`@Modifying`注释

**[`@Modifying`注释](https://web.archive.org/web/20220705092758/https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Modifying.html)用于增强`@Query`注释，这样我们不仅可以执行`SELECT`查询，还可以执行`INSERT`、`UPDATE`、`DELETE`，甚至`DDL`查询。**

现在让我们稍微玩一下这个注释。

首先，让我们看一个`@Modifying`更新查询的例子:

```java
@Modifying
@Query("update User u set u.active = false where u.lastLoginDate < :date")
void deactivateUsersNotLoggedInSince(@Param("date") LocalDate date);
```

在这里，我们将停用自给定日期以来未登录的用户。

让我们试试另一个，我们将删除停用的用户:

```java
@Modifying
@Query("delete User u where u.active = false")
int deleteDeactivatedUsers();
```

正如我们看到的，这个方法返回一个整数。**这是 Spring Data JPA `@Modifying`查询的一个特性，它为我们提供了更新实体的数量。**

我们应该注意，使用`@Query`执行删除查询的工作方式不同于 Spring Data JPA 的`deleteBy`名称派生查询方法。后者首先从数据库中获取实体，然后逐个删除它们。这意味着生命周期方法`@PreRemove`将在那些实体上被调用。但是，对于前者，只对数据库执行一个查询。

最后，让我们用一个`DDL`查询向我们的`USERS`表添加一个`deleted`列:

```java
@Modifying
@Query(value = "alter table USERS.USERS add column deleted int(1) not null default 0", nativeQuery = true)
void addDeletedColumn();
```

不幸的是，使用修改查询会使底层持久性上下文过时。但是，这种情况是可以控制的。这是下一节的主题。

### 3.1.不使用`@Modifying`注释的结果

让我们看看当我们不将`@Modifying`注释放在删除查询上时会发生什么。

因此，我们需要创建另一种方法:

```java
@Query("delete User u where u.active = false")
int deleteDeactivatedUsersWithNoModifyingAnnotation();
```

请注意丢失的注释。

当我们执行上述方法时，我们得到一个`InvalidDataAccessApiUsage`异常:

```java
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.hql.internal.QueryExecutionRequestException: 
Not supported for DML operations [delete com.baeldung.boot.domain.User u where u.active = false]
(...) 
```

错误消息非常清楚；查询是`Not supported for DML operations`。

## 4.管理持久性上下文

如果我们的修改查询改变了包含在持久上下文中的实体，那么这个上下文就过时了。管理这种情况的一种方法是[清除持久性上下文](https://web.archive.org/web/20220705092758/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#clear--)。通过这样做，我们可以确保持久性上下文下次将从数据库中获取实体。

但是，我们不必在`EntityManager`上显式调用`clear()`方法。我们可以从`@Modifying`注释中使用 [`clearAutomatically`属性](https://web.archive.org/web/20220705092758/https://codingexplained.com/coding/java/spring-framework/updating-entities-with-update-query-spring-data-jpa):

```java
@Modifying(clearAutomatically = true)
```

通过这种方式，我们可以确保在查询执行之后，持久性上下文被清除。

然而，如果我们的持久性上下文包含未刷新的更改，清除它将意味着丢弃未保存的更改。幸运的是，在这种情况下，我们可以使用注释的另一个属性，`flushAutomatically`:

```java
@Modifying(flushAutomatically = true)
```

**现在，在执行我们的查询之前，`EntityManager`被刷新。**

## 5.结论

这篇关于`@Modifying`注释的简短文章到此结束。我们学习了如何使用这个注释来执行更新查询，比如`INSERT, UPDATE, DELETE,`甚至`DDL`。之后，我们讨论了如何用`clearAutomatically`和`flushAutomatically`属性管理持久性上下文的状态。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220705092758/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise)