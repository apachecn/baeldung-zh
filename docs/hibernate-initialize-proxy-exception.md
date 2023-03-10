# Hibernate 无法初始化代理-没有会话

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-initialize-proxy-exception>

## 1.概观

使用 Hibernate 时，我们可能会遇到这样的错误:`org.hibernate.LazyInitializationException : could not initialize proxy – no Session` `.`

在这个快速教程中，我们将仔细看看错误的根本原因，并学习如何避免它。

## 2.理解错误

在打开的 Hibernate 会话的上下文之外访问延迟加载的对象将导致这个异常。

理解**什么是会话**、**惰性初始化、**、**和**、**代理对象**以及它们如何在`Hibernate`框架中结合在一起很重要:

*   `Session` 是一个持久上下文，表示应用程序和数据库之间的对话。
*   `Lazy Loading`意味着对象不会被加载到`Session` 上下文中，直到它在代码中被访问。
*   Hibernate 创建了一个动态的`Proxy Object`子类，只有当我们第一次使用这个对象时，它才会访问数据库。

当我们试图使用代理对象从数据库中获取一个延迟加载的对象，但是 Hibernate 会话已经关闭时，就会出现这个错误。

## 3.`LazyInitializationException`的例子

让我们看看具体场景中的异常。

我们想要创建一个简单的`User`对象和相关的角色。我们将使用 JUnit 来演示`LazyInitializationException` 错误。

### 3.1.Hibernate 实用程序类

首先，让我们定义一个 *HibernateUtil* 类来创建一个带有配置的`SessionFactory`。

我们将使用内存中的`HSQLDB`数据库。

### 3.2.实体

这是我们的`User`实体:

```java
@Entity
@Table(name = "user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @OneToMany
    private Set<Role> roles;

} 
```

以及相关联的`Role`实体:

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "role_name")
    private String roleName;
}
```

正如我们所看到的，在`User`和`Role`之间存在一对多的关系。

### 3.3.创建具有角色的用户

然后我们将创建两个`Role`对象:

```java
Role admin = new Role("Admin");
Role dba = new Role("DBA");
```

我们还将创建一个包含角色的`User`:

```java
User user = new User("Bob", "Smith");
user.addRole(admin);
user.addRole(dba);
```

最后，我们可以打开一个会话并持久保存对象:

```java
Session session = sessionFactory.openSession();
session.beginTransaction();
user.getRoles().forEach(role -> session.save(role));
session.save(user);
session.getTransaction().commit();
session.close();
```

### 3.4.获取角色

在第一个场景中，我们将看到如何以适当的方式获取用户角色:

```java
@Test
public void whenAccessUserRolesInsideSession_thenSuccess() {

    User detachedUser = createUserWithRoles();

    Session session = sessionFactory.openSession();
    session.beginTransaction();

    User persistentUser = session.find(User.class, detachedUser.getId());

    Assert.assertEquals(2, persistentUser.getRoles().size());

    session.getTransaction().commit();
    session.close();
}
```

这里我们在会话中访问对象，因此，没有错误。

### 3.5.提取角色失败

在第二个场景中，我们将在会话外调用一个`getRoles`方法:

```java
@Test
public void whenAccessUserRolesOutsideSession_thenThrownException() {

    User detachedUser = createUserWithRoles();

    Session session = sessionFactory.openSession();
    session.beginTransaction();

    User persistentUser = session.find(User.class, detachedUser.getId());

    session.getTransaction().commit();
    session.close();

    thrown.expect(LazyInitializationException.class);
    System.out.println(persistentUser.getRoles().size());
}
```

在这种情况下，我们试图在会话关闭后访问角色，结果代码抛出一个`LazyInitializationException` `.`

## 4.如何避免错误

现在，让我们来看看克服这一错误的四种不同的解决方案。

### 4.1.上层中的开放会话

最佳实践是在持久层中打开一个会话，例如使用 [DAO 模式](/web/20220712012000/https://www.baeldung.com/java-dao-pattern)。

我们可以在上层打开会话，以安全的方式访问相关的对象。例如，我们可以在`View`层打开会话。

结果，我们将看到响应时间增加，这将影响应用程序的性能。

根据关注点分离原则，这个解决方案是一个反模式。此外，它还会导致数据完整性违规和长时间运行的事务。

### 4.2.开启 **`enable_lazy_load_no_trans`** 属性

这个 Hibernate 属性用于声明惰性加载对象获取的全局策略。

默认情况下，该属性为`false`。打开它意味着对关联的惰性加载实体的每次访问都将被包装在新事务中运行的新会话中:

```java
<property name="hibernate.enable_lazy_load_no_trans" value="true"/>
```

不推荐使用这个属性来避免`LazyInitializationException` 错误，因为它会降低应用程序的性能。这是因为我们将**以一个 [n + 1 问题](/web/20220712012000/https://www.baeldung.com/hibernate-common-performance-problems-in-logs)结束。**简单来说，就是对`User`进行一次选择，再进行 N 次选择来获取每个用户的角色。

这种方法效率不高，也被认为是一种反模式。

### 4.3.使用  `FetchType.EAGER`策略

我们可以将这个策略与`@OneToMany`注释一起使用:

```java
@OneToMany(fetch = FetchType.EAGER)
@JoinColumn(name = "user_id")
private Set<Role> roles;
```

当我们需要为大多数用例获取相关的集合时，这是一种折衷的解决方案。

声明 [`EAGER` 获取类型](/web/20220712012000/https://www.baeldung.com/hibernate-fetchmode)要容易得多，而不是为大多数不同的业务流显式获取集合。

### 4.4.使用联接抓取

我们还可以使用`JPQL`中的`JOIN FETCH`指令按需获取相关的集合:

```java
SELECT u FROM User u JOIN FETCH u.roles
```

或者我们可以使用 Hibernate Criteria API:

```java
Criteria criteria = session.createCriteria(User.class);
criteria.setFetchMode("roles", FetchMode.EAGER);
```

这里，我们指定应该从数据库中获取的相关集合，以及同一个往返行程中的*用户*对象。使用这个查询提高了迭代的效率，因为它消除了单独检索相关对象的需要。

**这是避免`LazyInitializationException` 错误的最有效和细粒度的解决方案。**

## 5.结论

在本文中，我们学习了如何处理`org.hibernate.LazyInitializationException : could not initialize proxy – no Session` 错误`.`

我们探索了不同的方法，以及性能问题。我们还讨论了使用简单高效的解决方案以避免影响性能的重要性。

最后，我们演示了连接获取方法是如何避免错误的好方法。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220712012000/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-exceptions)