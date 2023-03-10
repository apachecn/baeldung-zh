# @Size、@Length 和@Column 之间的差异(length=value)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-size-length-column-differences>

 ![announcement - icon](img/09a677d1478a9680c79e73e13487328a.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220525131809/https://www.baeldung.com/lightrun-n-jpa)

## 1.概观

在这个快速教程中，我们将看看 [JSR-330](/web/20220525131809/https://www.baeldung.com/javax-validation) 的`@Size`、[休眠](/web/20220525131809/https://www.baeldung.com/hibernate-4-spring)的`@Length`和 [JPA](/web/20220525131809/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 、`@Column`的`length` 属性。

乍一看，**这些可能看起来一样，但它们执行不同的功能**。让我们看看怎么做。

## 2.起源

简单地说，所有这些注释都是为了**传达一个字段的大小。**

`@Size` 和`@Length`类似。我们可以使用其中任何一个来验证字段的大小。第一个是 [Java 标准注释](https://web.archive.org/web/20220525131809/https://docs.oracle.com/javaee/7/tutorial/bean-validation001.htm)，第二个是特定于 Hibernate 的[。](https://web.archive.org/web/20220525131809/http://docs.jboss.org/ejb3/app-server/HibernateAnnotations/api/org/hibernate/validator/Length.html)

然而，`@Column`是一个 [JPA 注释](https://web.archive.org/web/20220525131809/https://docs.oracle.com/javaee/7/api/javax/persistence/Column.html)，我们用它来控制 DDL 语句。

现在，让我们详细地看一下它们。

## 3.`@Size`

对于验证，我们将使用`@Size` **，**一个 bean 验证注释。让我们使用用`@Size`注释的属性`middleName`来验证其在属性 *min* 和 *max:* 之间的值

```java
public class User {

    // ...

    @Size(min = 3, max = 15)
    private String middleName;

    // ...

}
```

最重要的是， **`@Size`使得 bean 独立于 JPA 及其供应商如 Hibernate** 。结果这比`@Length`更便携。

## 4.`@Length`

正如我们刚刚提到的，`@Length` 是`@Size.` 的 Hibernate 特定版本，让我们使用`@Length`来加强`lastName`的范围:

```java
@Entity
public class User {

    // ...

    @Length(min = 3, max = 15)
    private String lastName;

    // ...

}
```

## 5.`@Column(length=value)`

然而，情况完全不同。

我们将使用`@Column`到**来表示物理数据库列的具体特征。**让我们使用`@Column`注释的`[length](https://web.archive.org/web/20220525131809/http://java.sun.com/javaee/5/docs/api/javax/persistence/Column.html#length%28%29)`属性来指定字符串值列的长度:

```java
@Entity
public class User {

    @Column(length = 3)
    private String firstName;

    // ...

}
```

因此，生成的列将被生成为`VARCHAR(3)`，尝试插入更长的字符串将导致 SQL 错误。

注意，我们将只使用`@Column`来指定表列属性**，因为它不提供验证。**

当然，**我们可以将`@Column`与`@Size`** 一起使用，通过 bean 验证来指定数据库列属性。

```java
@Entity
public class User {

    // ... 

    @Column(length = 5)
    @Size(min = 3, max = 5)
    private String city;

    // ...

}
```

## 6.结论

在这篇文章中，我们了解了`@Size` 注释、`@Length`注释和`@Column`的`length`属性之间的区别。我们在它们的使用范围内分别检查了它们。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220525131809/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)