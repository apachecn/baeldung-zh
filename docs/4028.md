# 休眠错误“未设置所有命名参数”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-error-named-parameters-not-set>

## 1.介绍

当使用 [Hibernate](/web/20221207091825/https://www.baeldung.com/tag/hibernate/) 时，我们可以使用命名参数将数据安全地传递到 SQL 查询中。我们在运行时给查询参数赋值，使它们是动态的。更重要的是，这有助于防止 SQL 注入袭击。

但是，在使用命名参数时，我们可能会遇到错误。Hibernate 的独立库和 Hibernate JPA 实现中的两个比较常见的例子分别是:

*   `Not all named parameters have been set`
*   `Named parameter not bound`

尽管普通 Hibernate 和它的 JPA 实现之间的错误消息可能不同，但是根本原因是相同的。

在本教程中，我们将看看**是什么导致了这些错误，以及如何避免它们**。同时，我们将演示如何在 Hibernate 的独立库中使用命名参数。

## 2.什么导致了错误

在 Hibernate 中使用命名参数时，我们必须在执行查询之前给每个命名参数赋值。

让我们看一个使用命名参数的查询示例:

```
Query<Event> query = session.createQuery("from Event E WHERE E.title = :eventTitle", Event.class);
```

在这个例子中，我们有一个命名参数，由占位符`:eventTitle`表示。Hibernate 希望在我们执行查询之前设置这个参数。

但是，如果我们试图在不设置值的情况下执行查询:`eventTitle`:

```
List<Event> listOfEvents = query.list();
```

当我们运行 Hibernate 时，它会抛出`org.hibernate.QueryException`，我们会得到错误:

```
Not all named parameters have been set
```

## 3.修复错误

为了修复错误，我们只需在执行查询之前为命名参数提供一个值:

```
Query<Event> query = session.createQuery("from Event E WHERE E.title = :eventTitle", Event.class);
query.setParameter("eventTitle", "Event 1");

assertEquals(1, query.list().size());
```

通过使用`query`对象的`setParameter(String, String) `方法，我们告诉 Hibernate 我们想要为命名参数使用哪个值。

## 4.结论

在本文中，我们研究了命名参数以及如何在 Hibernate 中使用它们。我们还展示了如何修复我们可能遇到的一个命名查询错误。

像往常一样，所有的代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221207091825/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-enterprise)