# 使用 Hibernate 从表中获取所有数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-select-all>

## 1。概述

在这个快速教程中，我们将看看如何使用 JPQL 或 Criteria API 从 Hibernate 的一个表中获取所有数据。

JPQL 为我们提供了更快更简单的实现，而使用 Criteria API 则更加动态和健壮。

## 2。JPQL

JPQL 提供了一种简单直接的方法来从表中获取所有实体。

让我们看看使用 JPQL 从一个表中检索所有学生是什么样子:

```java
public List<Student> findAllStudentsWithJpql() {
    return session.createQuery("SELECT a FROM Student a", Student.class).getResultList();      
} 
```

我们的 Hibernate 会话的`createQuery()`方法接收一个类型化的查询字符串作为第一个参数，实体的类型作为第二个参数。我们通过调用`getResultList()`方法来执行查询，该方法将结果作为类型化的`List`返回。

简单是这种方法的优点。 JPQL 非常接近 SQL，因此更容易编写和理解。

## 3。标准 API

**[标准 API](/web/20220523231412/https://www.baeldung.com/hibernate-criteria-queries) 提供了构建 JPA 查询的动态方法。**

它允许我们通过实例化表示查询元素的 Java 对象来构建查询。如果查询是由许多可选字段构成的，这是一个更干净的解决方案，因为它消除了许多字符串连接。

我们刚刚看到了一个使用 JPQL 的全选查询。让我们使用 Criteria API 来看看它的对等物:

```java
public List<Student> findAllStudentsWithCriteriaQuery() {
    CriteriaBuilder cb = session.getCriteriaBuilder();
    CriteriaQuery<Student> cq = cb.createQuery(Student.class);
    Root<Student> rootEntry = cq.from(Student.class);
    CriteriaQuery<Student> all = cq.select(rootEntry);

    TypedQuery<Student> allQuery = session.createQuery(all);
    return allQuery.getResultList();
} 
```

首先，我们得到一个`CriteriaBuilder`，我们用它来创建一个类型化的`Criteria` `Query`。稍后，我们为查询设置根条目。最后，我们用一个`getResultList()`方法来执行它。

现在，这种方法类似于我们之前所做的。但是，它让我们能够完全接触到 Java 语言，在表达查询时表达更细微的差别。

除了相似之外，JPQL 查询和基于 JPA 标准的查询性能相当。

## 4。结论

在本文中，我们演示了如何使用 JPQL 或 Criteria API 从表中获取所有实体。

该示例的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220523231412/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-queries)