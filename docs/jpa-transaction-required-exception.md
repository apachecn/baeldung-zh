# TransactionRequiredException 错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-transaction-required-exception>

## 1。概述

在本教程中，我们将检查`TransactionRequiredException`错误的原因以及如何解决它。

## 2。`TransactionRequiredException`

**当我们试图在没有事务**的情况下执行修改数据库的数据库操作时，通常会出现这种错误。

例如，试图在没有事务的情况下更新记录:

```java
Query updateQuery
  = session.createQuery("UPDATE Post p SET p.title = ?1, p.body = ?2 WHERE p.id = ?3");
updateQuery.setParameter(1, title);
updateQuery.setParameter(2, body);
updateQuery.setParameter(3, id);
updateQuery.executeUpdate();
```

将引发异常，并显示如下消息:

```java
...
javax.persistence.TransactionRequiredException: Executing an update/delete query
  at org.hibernate.query.internal.AbstractProducedQuery.executeUpdate(AbstractProducedQuery.java:1586)
...
```

## 3。提供交易

**显而易见的解决方案是将任何数据库修改操作包装在一个事务中:**

```java
Transaction txn = session.beginTransaction();
Query updateQuery
  = session.createQuery("UPDATE Post p SET p.title = ?1, p.body = ?2 WHERE p.id = ?3");
updateQuery.setParameter(1, title);
updateQuery.setParameter(2, body);
updateQuery.setParameter(3, id);
updateQuery.executeUpdate();
txn.commit();
```

在上面的代码片段中，我们手动启动并提交事务。尽管**在 Spring Boot 环境中，我们可以通过使用`@Transactional `注释来实现这一点。**

## 4。春季交易支持

**如果想要更细粒度的控制，可以使用 Spring 的`TransactionTemplate`** 。因为这允许程序员在继续执行方法的代码之前立即触发对象的持久性。

例如，假设我们想在发送电子邮件通知之前更新帖子:

```java
public void update() {
    entityManager.createQuery("UPDATE Post p SET p.title = ?2, p.body = ?3 WHERE p.id = ?1")
      // parameters
      .executeUpdate();
    sendEmail();
}
```

**将`@Transactional `应用于上述方法可能会导致电子邮件被发送，尽管在更新过程中出现异常。**这是因为只有当方法退出并准备返回给调用者时，事务才会被提交。

因此，在`TransactionTemplate`中更新 post 将防止这种情况，因为它会立即提交操作:

```java
public void update() {
    transactionTemplate.execute(transactionStatus -> {
        entityManager.createQuery("UPDATE Post p SET p.title = ?2, p.body = ?3 WHERE p.id = ?1")
          // parameters
          .executeUpdate();
        transactionStatus.flush();
        return null;
    });
    sendEmail();
}
```

## 5。结论

总之，将数据库操作包装在一个事务中通常是一个好的做法。它有助于防止数据损坏。完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220630135928/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-exceptions)