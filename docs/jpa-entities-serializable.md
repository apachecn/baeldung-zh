# JPA 实体和可序列化接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-entities-serializable>

## 1.介绍

在本教程中，我们将讨论 JPA 实体和 Java `Serializable`接口是如何融合的。首先，我们将看看`java.io.Serializable` 接口以及我们为什么需要它。之后，我们将看看 JPA 规范，并将 Hibernate 作为其最流行的实现。

## 2.什么是`Serializable` 界面？

`Serializable` 是核心 Java 中为数不多的标记接口之一。[标记接口](/web/20221104165328/https://www.baeldung.com/java-marker-interfaces)是没有方法或常量的特殊情况接口。

**对象序列化是将 Java 对象转换成字节流的过程**。然后，我们可以通过网络传输这些字节流，或者将它们存储在永久存储器中。**反序列化是相反的过程**，我们将字节流转换回 Java 对象。为了允许对象序列化(或反序列化)，一个类必须实现`Serializable`接口。不然就碰到`java.io.NotSerializableException`了。 **[序列化](/web/20221104165328/https://www.baeldung.com/java-serialization)广泛应用于 RMI、JPA、EJB** 等技术中。

## 3.JPA 和`Serializable`

让我们看看 JPA 规范是如何描述`Serializable` 的，以及它如何适用于 Hibernate。

### 3.1.JPA 规范

JPA 的核心部分之一是实体类。我们将这些类标记为实体(用`@Entity`注释或 XML 描述符)。我们的实体类必须满足几个需求，根据 JPA 规范，我们最关心的需求[是:](https://web.archive.org/web/20221104165328/https://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf)

> 如果一个实体实例要作为一个分离的对象通过值传递(例如，通过一个远程接口)，那么 实体类必须实现 `Serializable` 接口。

实际上，**如果我们的对象要离开 JVM 的域，它将需要序列化**。

每个实体类都由持久字段和属性组成。该规范要求实体的字段可以是 Java 原语、Java 可序列化类型或用户定义的可序列化类型。

实体类还必须有一个主键。主键可以是基本的(单个持久字段)或复合的。多个规则适用于一个组合键，其中之一是要求一个**组合键是可序列化的**。

让我们使用 Hibernate、H2 内存数据库和一个以`UserId` 作为组合键的`User` 域对象创建一个简单的例子:

```java
@Entity
public class User {
    @EmbeddedId UserId userId;
    String email;

    // constructors, getters and setters
}

@Embeddable
public class UserId implements Serializable{
    private String name;
    private String lastName;

    // getters and setters
}
```

我们可以使用集成测试来测试我们的领域定义:

```java
@Test
public void givenUser_whenPersisted_thenOperationSuccessful() {
    UserId userId = new UserId();
    userId.setName("John");
    userId.setLastName("Doe");
    User user = new User(userId, "[[email protected]](/web/20221104165328/https://www.baeldung.com/cdn-cgi/l/email-protection)");

    entityManager.persist(user);

    User userDb = entityManager.find(User.class, userId);
    assertEquals(userDb.email, "[[email protected]](/web/20221104165328/https://www.baeldung.com/cdn-cgi/l/email-protection)");
}
```

如果我们的`UserId `类没有实现`Serializable`接口，我们将得到一个`MappingException`,其中包含一条具体消息，表明我们的组合键必须实现该接口。

### 3.2.休眠`@JoinColumn`注释

[Hibernate 官方文档](https://web.archive.org/web/20221104165328/https://hibernate.org/orm/documentation/)，在描述 Hibernate 中的映射时，注意到当我们从 [`@JoinColumn`](/web/20221104165328/https://www.baeldung.com/jpa-join-column) 注释中使用`referencedColumnName` 时，**引用的字段必须是可序列化的。通常，该字段是另一个实体中的主键。在复杂实体类的极少数情况下，我们的引用必须是可序列化的。**

让我们扩展前面的`User` 类，其中`email` 字段不再是一个`String`，而是一个独立的实体。此外，我们将添加`an Account` 类，它将引用一个用户，并有一个字段`type.` ，每个`User` 可以有多个不同类型的帐户。我们将通过`email`映射`Account` ，因为通过电子邮件地址搜索更自然:

```java
@Entity
public class User {
    @EmbeddedId private UserId userId;
    private Email email;
}

@Entity
public class Email implements Serializable {
    @Id
    private long id;
    private String name;
    private String domain;
}

@Entity
public class Account {
    @Id
    private long id;
    private String type;
    @ManyToOne
    @JoinColumn(referencedColumnName = "email")
    private User user;
}
```

为了测试我们的模型，我们将编写一个测试，为一个用户创建两个帐户，并通过一个电子邮件对象进行查询:

```java
@Test
public void givenAssociation_whenPersisted_thenMultipleAccountsWillBeFoundByEmail() {
    // object creation 

    entityManager.persist(user);
    entityManager.persist(account);
    entityManager.persist(account2);

    List userAccounts = entityManager.createQuery("select a from Account a join fetch a.user where a.user.email = :email")
      .setParameter("email", email)
      .getResultList();

    assertEquals(userAccounts.size(), 2);
}
```

如果`Email` 类没有实现`Serializable`接口，我们将再次得到`MappingException`，但是这一次带有一个有点隐晦的消息:“无法确定类型”。

### 3.3.向表示层公开实体

当使用 HTTP 通过网络发送对象时，我们通常为此创建特定的 dto(数据传输对象)。通过创建 dto，我们将内部域对象从外部服务中分离出来。如果我们想在没有 dto 的情况下直接向表示层公开我们的实体，那么实体必须是可序列化的。

我们使用`HttpSession` 对象来存储相关数据，这些数据帮助我们识别访问我们网站的多个页面的用户。web 服务器可以在正常关闭时将会话数据存储在磁盘上，或者将会话数据传输到集群环境中的另一个 web 服务器。如果一个实体是这个过程的一部分，那么它必须是可序列化的。不然就碰到`NotSerializableException`了。

## 4.结论

在本文中，我们介绍了 Java 序列化的基础知识，并了解了它如何在 JPA 中发挥作用。首先，我们回顾了 JPA 规范关于`Serializable`的要求。之后，我们研究了 Hibernate 作为 JPA 最流行的实现。最后，我们讲述了 JPA 实体如何与 web 服务器一起工作。

像往常一样，本文中的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221104165328/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-jpa)