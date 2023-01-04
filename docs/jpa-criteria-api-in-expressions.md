# 标准 API–IN 表达式的一个例子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-criteria-api-in-expressions>

 ![](img/32a5c71767e350fae07c153e23d4432f.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220524002723/https://www.baeldung.com/lightrun-n-jpa)

## 1.概观

我们经常遇到这样的问题，我们需要根据单值属性是否是给定集合的成员来查询实体。

在本教程中，我们将学习如何在 [`Criteria` API](/web/20220524002723/https://www.baeldung.com/hibernate-criteria-queries) 的帮助下解决这个问题。

## 2.样本实体

在我们开始之前，让我们看一下我们将在文章中使用的实体。

我们有一个`DeptEmployee`类，它与一个`Department`类有一个[多对一关系](/web/20220524002723/https://www.baeldung.com/hibernate-one-to-many):

```java
@Entity
public class DeptEmployee {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String title;

    @ManyToOne
    private Department department;
}
```

同样，映射到多个`DeptEmployees`的`Department`实体:

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String name;

    @OneToMany(mappedBy="department")
    private List<DeptEmployee> employees;
}
```

## 3.`CriteriaBuilder.In`

首先，我们用一下`CriteriaBuilder`界面。**`in()`方法接受一个`Expression`并返回一个新的`CriteriaBuilder.In`类型的`Predicate``.`**它可以用来测试给定的表达式是否包含在值列表中:

```java
CriteriaQuery<DeptEmployee> criteriaQuery = 
  criteriaBuilder.createQuery(DeptEmployee.class);
Root<DeptEmployee> root = criteriaQuery.from(DeptEmployee.class);
In<String> inClause = criteriaBuilder.in(root.get("title"));
for (String title : titles) {
    inClause.value(title);
}
criteriaQuery.select(root).where(inClause);
```

## 4.`Expression.In`

或者，我们可以从 [`Expression`](https://web.archive.org/web/20220524002723/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/criteria/Expression.html) 接口中使用一组重载的`in()`方法:

```java
criteriaQuery.select(root)
  .where(root.get("title")
  .in(titles));
```

**与`CriteriaBuilder.` `in()`相反，`Expression.in()`接受一组值。**正如我们所见，它也稍微简化了我们的代码。

## 5.`IN`使用子查询的表达式

到目前为止，我们已经使用了具有预定义值的集合。现在，让我们看一个例子，一个集合是从子查询的输出中派生出来的。

例如，我们可以获取所有属于名称中带有指定关键字的`Department,`的`DeptEmployee`:

```java
Subquery<Department> subquery = criteriaQuery.subquery(Department.class);
Root<Department> dept = subquery.from(Department.class);
subquery.select(dept)
  .distinct(true)
  .where(criteriaBuilder.like(dept.get("name"), "%" + searchKey + "%"));

criteriaQuery.select(emp)
  .where(criteriaBuilder.in(emp.get("department")).value(subquery));
```

这里，我们创建了一个子查询，然后将它作为表达式传递给`value()`来搜索`Department`实体。

## 6.结论

在这篇简短的文章中，我们学习了使用 Criteria API 实现 In 操作的不同方法。我们还探讨了如何将 Criteria API 用于子查询。

最后，本教程的完整实现可以在 GitHub 上找到。