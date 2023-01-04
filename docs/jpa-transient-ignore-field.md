# 忽略带有 JPA @Transient 注释的字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-transient-ignore-field>

## 1.介绍

当使用对象关系映射(ORM)框架将 Java 对象持久化到数据库记录中时，我们经常希望忽略某些字段。**如果框架符合 Java 持久性 API (JPA ),我们可以给这些字段添加`@Transient`注释。**

在本教程中，我们将演示`@Transient`注释的正确用法。我们还将看看它与 [Java 内置的`transient`关键字](/web/20220626200744/https://www.baeldung.com/java-transient-keyword)的关系。

## 2.`@Transient`注释与`transient`关键字

对于`@Transient`注释和 Java 内置的`transient`关键字之间的关系，通常会有一些混淆。`transient`关键字主要用于在 [Java 对象序列化](/web/20220626200744/https://www.baeldung.com/java-serialization)期间忽略字段，但是它也防止这些字段在使用 JPA 框架时被持久化。

**换句话说，当保存到数据库时，`transient`关键字与`@Transient`注释具有相同的效果。然而，`@Transient`注释并不影响 Java 对象的序列化。** 

## 3.JPA `@Transient`示例

假设我们有一个`User`类，它是一个 [JPA 实体](/web/20220626200744/https://www.baeldung.com/jpa-entities)，映射到数据库中的一个 Users 表。当用户登录时，我们从 Users 表中检索他们的记录，然后我们在`User`实体上设置一些额外的字段。这些额外的字段不对应于 Users 表中的任何列，因为我们不想保存这些值。

例如，我们将在`User`实体上设置一个时间戳，表示用户登录到当前会话的时间:

```java
@Entity
@Table(name = "Users")
public class User {

    @Id
    private Integer id;

    private String email;

    private String password;

    @Transient
    private Date loginTime;

    // getters and setters
}
```

**当我们使用 Hibernate 这样的 JPA 提供者将这个`User`对象保存到数据库时，提供者会因为`@Transient`注释而忽略`loginTime`字段。**

如果我们序列化这个`User`对象并将其传递给系统中的另一个服务，那么`loginTime`字段将包含在序列化中。如果我们不想包含这个字段，我们可以用关键字`transient`代替`@Transient`注释:

```java
@Entity
@Table(name = "Users")
public class User implements Serializable {

    @Id
    private Integer id;

    private String email;

    private String password;

    private transient Date loginTime;

    //getters and setters
}
```

**现在，`loginTime`字段在数据库持久化和对象序列化期间被忽略。**

## 4.结论

在本文中，我们研究了如何在典型用例中正确使用 JPA `@Transient`注释。请务必查看 JPA 上的其他文章，了解更多关于持久性的知识。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220626200744/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)