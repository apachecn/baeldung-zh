# JPA 什么时候设置主键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-strategies-when-set-primary-key>

## 1.概观

在本教程中，我们将举例说明 JPA 给主键赋值的时刻**。我们将阐明 JPA 规范说了什么，然后，我们将展示使用各种 JPA 策略生成主键的例子。**

## 2.问题陈述

我们知道，JPA (Java Persistence API)使用`EntityManager`来管理`Entity`的生命周期。在某些时候，JPA 提供者需要给主键赋值。因此，我们可能会问自己，这是什么时候发生的？说明这一点的文件在哪里？

JPA 规范规定:

> 通过调用实体实例上的 persist 方法或级联 persist 操作，新的实体实例既成为托管的又成为持久的。

因此，我们将在本文中重点关注`EntityManager.persist()`方法。

## 3.创造价值战略

当我们调用`EntityManager.persist()` 方法时，实体的状态会根据 JPA 规范而改变:

> 如果 X 是一个新实体，它将被管理。实体 X 将在事务提交时或之前或作为刷新操作的结果被输入数据库。

这意味着有多种方法可以生成主键。一般来说，有两种解决方案:

*   预分配主键
*   在数据库中持久化后分配主键

To be more specific, JPA offers **four strategies to generate the primary key**:

*   `GenerationType.AUTO`
*   `GenerationType.IDENTITY`
*   `GenerationType.SEQUENCE`
*   `GenerationType.TABLE`

Let's take a look at them one by one.

### 3.1.`GenerationType.AUTO`

*自动*是`@GeneratedValue` 的**默认策略。如果我们只想有一个主键，我们可以使用`AUTO`策略。JPA 提供者将为底层数据库选择适当的策略:**

```java
@Entity
@Table(name = "app_admin")
public class Admin {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "admin_name")
    private String name;

    // standard getters and setters
}
```

### 3.2.`GenerationType.IDENTITY`

**`IDENTITY`策略依赖于数据库自动递增列**。数据库在每次插入操作后都会生成主键。JPA 在执行插入操作或事务提交后分配主键值:

```java
@Entity
@Table(name = "app_user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "user_name")
    private String name;

    // standard getters and setters
}
```

这里，我们验证事务提交前后的`id`值:

```java
@Test
public void givenIdentityStrategy_whenCommitTransction_thenReturnPrimaryKey() {
    User user = new User();
    user.setName("TestName");

    entityManager.getTransaction().begin();
    entityManager.persist(user);
    Assert.assertNull(user.getId());
    entityManager.getTransaction().commit();

    Long expectPrimaryKey = 1L;
    Assert.assertEquals(expectPrimaryKey, user.getId());
}
```

MySQL、SQL Server、PostgreSQL、DB2、Derby 和 Sybase 都支持`IDENTITY`策略。

### 3.3.`GenerationType.SEQUENCE`

通过使用`SEQUENCE`策略， **JPA 使用数据库序列**生成主键。在应用这个策略之前，我们首先需要在数据库端创建一个序列:

```java
CREATE SEQUENCE article_seq
  MINVALUE 1
  START WITH 50
  INCREMENT BY 50
```

JPA 在我们调用`EntityManager.persist()`方法之后和提交事务之前设置主键**。**

让我们用`SEQUENCE`策略定义一个`Article`实体:

```java
@Entity
@Table(name = "article")
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "article_gen")
    @SequenceGenerator(name="article_gen", sequenceName="article_seq")
    private Long id;

    @Column(name = "article_name")
    private String name

    // standard getters and setters
}
```

序列从 50 开始，所以第一个`id`将是下一个值 51。

现在，让我们测试一下`SEQUENCE`策略:

```java
@Test
public void givenSequenceStrategy_whenPersist_thenReturnPrimaryKey() {
    Article article = new Article();
    article.setName("Test Name");

    entityManager.getTransaction().begin();
    entityManager.persist(article);
    Long expectPrimaryKey = 51L;
    Assert.assertEquals(expectPrimaryKey, article.getId());

    entityManager.getTransaction().commit();
}
```

Oracle、PostgreSQL 和 DB2 支持`SEQUENCE`策略。

### 3.4.`GenerationType.TABLE`

`TABLE`策略从表中生成主键，不管底层数据库是什么，它都是一样的。

我们需要**在数据库端创建一个生成器表来生成主键**。该表至少应该有两列:一列表示生成器的名称，另一列存储主键值。

首先，让我们创建一个生成器表:

```java
@Table(name = "id_gen")
@Entity
public class IdGenerator {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "gen_name")
    private String gen_name;

    @Column(name = "gen_value")
    private Long gen_value;

    // standard getters and setters
}
```

然后，我们需要向生成器表中插入两个初始值:

```java
INSERT INTO id_gen (gen_name, gen_val) VALUES ('id_generator', 0);
INSERT INTO id_gen (gen_name, gen_val) VALUES ('task_gen', 10000);
```

**JPA 在调用`EntityManager.persist()`方法之后和事务提交之前分配主键值。**

现在让我们使用带有`TABLE`策略的生成器表。我们可以使用`allocationSize`来预分配一些主键:

```java
@Entity
@Table(name = "task")
public class Task {

    @TableGenerator(name = "id_generator", table = "id_gen", pkColumnName = "gen_name", valueColumnName = "gen_value",
        pkColumnValue="task_gen", initialValue=10000, allocationSize=10)
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "id_generator")
    private Long id;

    @Column(name = "name")
    private String name;

    // standard getters and setters
}
```

在我们调用`persist`方法后，`id`从 10，000 开始:

```java
@Test
public void givenTableStrategy_whenPersist_thenReturnPrimaryKey() {
    Task task = new Task();
    task.setName("Test Task");

    entityManager.getTransaction().begin();
    entityManager.persist(task);
    Long expectPrimaryKey = 10000L;
    Assert.assertEquals(expectPrimaryKey, task.getId());

    entityManager.getTransaction().commit();
}
```

## 4.结论

本文举例说明了 JPA 在不同策略下设置主键的时刻。此外，我们还通过示例了解了这些策略的用法。

完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205203259/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)