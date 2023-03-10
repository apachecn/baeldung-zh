# Java 中的 DAO 模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dao-pattern>

## 1。概述

数据访问对象(DAO)模式是一种结构模式，它允许我们使用抽象 API 将应用程序/业务层与持久层(通常是关系数据库，但也可以是任何其他的持久化机制)隔离开来。

API 对应用程序隐藏了在底层存储机制中执行 CRUD 操作的所有复杂性。这允许两层在不了解对方的情况下独立发展。

在本教程中，我们将深入探讨该模式的实现，并学习如何使用它来抽象对 [JPA 实体管理器](https://web.archive.org/web/20220926183157/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html)的调用。

## 延伸阅读:

## [Spring Data JPA 简介](/web/20220926183157/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

Introduction to Spring Data JPA with Spring 4 - the Spring config, the DAO, manual and generated queries and transaction management.[Read more](/web/20220926183157/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) →

## [JPA/Hibernate 级联类型概述](/web/20220926183157/https://www.baeldung.com/jpa-cascade-types)

A quick and practical overview of JPA/Hibernate Cascade Types.[Read more](/web/20220926183157/https://www.baeldung.com/jpa-cascade-types) →

## 2。一个简单的实现

为了理解 DAO 模式是如何工作的，让我们创建一个基本的例子。

假设我们想开发一个管理用户的应用程序。我们希望保持应用程序的域模型对数据库完全不可知。因此，我们将创建**一个简单的 DAO 类，它将负责保持这些组件整齐地相互解耦。**

### 2.1。域类

因为我们的应用程序将与用户一起工作，所以我们只需要定义一个类来实现它的域模型:

```java
public class User {

    private String name;
    private String email;

    // constructors / standard setters / getters
}
```

`User`类只是一个普通的用户数据容器，所以它没有实现任何其他值得强调的行为。

当然，这里重要的设计选择是如何让使用这个类的应用程序与任何可以实现的持久性机制隔离开来。

这正是 DAO 模式试图解决的问题。

### 2.2。DAO API

让我们定义一个基本的 DAO 层，这样我们就可以看到它如何**保持领域模型与持久层完全分离。**

下面是 DAO API:

```java
public interface Dao<T> {

    Optional<T> get(long id);

    List<T> getAll();

    void save(T t);

    void update(T t, String[] params);

    void delete(T t);
}
```

从鸟瞰图来看，`Dao`接口显然定义了一个抽象 API，它对类型`T`的对象执行 CRUD 操作。

由于接口提供的高度抽象，很容易创建一个具体的、细粒度的实现来处理`User`对象。

### 2.3。`UserDao`班

让我们定义一个特定于用户的`Dao`接口实现:

```java
public class UserDao implements Dao<User> {

    private List<User> users = new ArrayList<>();

    public UserDao() {
        users.add(new User("John", "[[email protected]](/web/20220926183157/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
        users.add(new User("Susan", "[[email protected]](/web/20220926183157/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    }

    @Override
    public Optional<User> get(long id) {
        return Optional.ofNullable(users.get((int) id));
    }

    @Override
    public List<User> getAll() {
        return users;
    }

    @Override
    public void save(User user) {
        users.add(user);
    }

    @Override
    public void update(User user, String[] params) {
        user.setName(Objects.requireNonNull(
          params[0], "Name cannot be null"));
        user.setEmail(Objects.requireNonNull(
          params[1], "Email cannot be null"));

        users.add(user);
    }

    @Override
    public void delete(User user) {
        users.remove(user);
    }
}
```

`UserDao`类实现了获取、更新和删除`User`对象所需的所有功能。

**为了简单起见，`users List`就像一个内存中的数据库，由构造函数中的几个`User`对象填充。**

当然，重构其他方法是很容易的，这样它们就可以工作，例如，处理关系数据库。

虽然`User`和`UserDao`类独立地共存于同一个应用程序中，但是我们仍然需要了解如何使用后者来保持持久层对应用程序逻辑隐藏:

```java
public class UserApplication {

    private static Dao<User> userDao;

    public static void main(String[] args) {
        userDao = new UserDao();

        User user1 = getUser(0);
        System.out.println(user1);
        userDao.update(user1, new String[]{"Jake", "[[email protected]](/web/20220926183157/https://www.baeldung.com/cdn-cgi/l/email-protection)"});

        User user2 = getUser(1);
        userDao.delete(user2);
        userDao.save(new User("Julie", "[[email protected]](/web/20220926183157/https://www.baeldung.com/cdn-cgi/l/email-protection)"));

        userDao.getAll().forEach(user -> System.out.println(user.getName()));
    }

    private static User getUser(long id) {
        Optional<User> user = userDao.get(id);

        return user.orElseGet(
          () -> new User("non-existing user", "no-email"));
    }
}
```

这个例子是人为设计的，但是它简单地展示了 DAO 模式背后的动机。在这种情况下，`main`方法只是使用一个`UserDao`实例对几个`User`对象执行 CRUD 操作。

**这个过程最相关的方面是`UserDao`如何向应用程序隐藏关于对象如何被持久化、更新和删除的所有底层细节。**

## 3。使用带有 JPA 的模式

开发人员倾向于认为 JPA 的发布降低了 DAO 模式的功能。该模式成为 JPA 的实体管理器所提供的抽象和复杂性的又一层。

在某些情况下确实如此。尽管如此，我们有时只想向我们的应用程序公开实体管理器 API 的一些特定领域的方法。道模式在这种情况下有它的位置。

### 3.1。`JpaUserDao`班

让我们创建一个新的`Dao`接口实现，看看它如何封装 JPA 的实体管理器提供的现成功能:

```java
public class JpaUserDao implements Dao<User> {

    private EntityManager entityManager;

    // standard constructors

    @Override
    public Optional<User> get(long id) {
        return Optional.ofNullable(entityManager.find(User.class, id));
    }

    @Override
    public List<User> getAll() {
        Query query = entityManager.createQuery("SELECT e FROM User e");
        return query.getResultList();
    }

    @Override
    public void save(User user) {
        executeInsideTransaction(entityManager -> entityManager.persist(user));
    }

    @Override
    public void update(User user, String[] params) {
        user.setName(Objects.requireNonNull(params[0], "Name cannot be null"));
        user.setEmail(Objects.requireNonNull(params[1], "Email cannot be null"));
        executeInsideTransaction(entityManager -> entityManager.merge(user));
    }

    @Override 
    public void delete(User user) {
        executeInsideTransaction(entityManager -> entityManager.remove(user));
    }

    private void executeInsideTransaction(Consumer<EntityManager> action) {
        EntityTransaction tx = entityManager.getTransaction();
        try {
            tx.begin();
            action.accept(entityManager);
            tx.commit(); 
        }
        catch (RuntimeException e) {
            tx.rollback();
            throw e;
        }
    }
}
```

**`JpaUserDao`类可以处理 JPA 实现支持的任何关系数据库。**

同样，如果我们仔细观察这个类，我们会意识到如何使用[组合](https://web.archive.org/web/20220926183157/https://en.wikipedia.org/wiki/Composition_over_inheritance)和[依赖注入](https://web.archive.org/web/20220926183157/https://en.wikipedia.org/wiki/Dependency_injection)来允许我们只调用应用程序所需的实体管理器方法。

简单地说，我们有一个特定领域定制的 API，而不是整个实体管理器的 API。

### 3.2。重构`User`类

在这种情况下，我们将使用 Hibernate 作为 JPA 的默认实现，因此我们将相应地重构`User`类:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String name;
    private String email;

    // standard constructors / setters / getters
}
```

### 3.3。以编程方式引导 JPA 实体管理器

 **假设我们已经有了一个运行在本地或远程的 MySQL 实例和一个填充了一些用户记录的数据库表`“users”`，我们需要一个 JPA 实体管理器，这样我们就可以使用`JpaUserDao`类在数据库中执行 CRUD 操作。

在大多数情况下，我们通过典型的`persistence.xml`文件来完成，这是标准的方法。

在这种情况下，我们将采用一种无 XML 的方法，通过 Hibernate 的便利的`[EntityManagerFactoryBuilderImpl](https://web.archive.org/web/20220926183157/https://docs.jboss.org/hibernate/orm/5.0/javadocs/org/hibernate/jpa/boot/internal/EntityManagerFactoryBuilderImpl.html)`类用普通 Java 获得实体管理器。

关于如何用 Java 引导 JPA 实现的详细解释，请查看本文。

### 3.4。`UserApplication`班

最后，让我们重构初始的`UserApplication`类，这样它就可以使用`JpaUserDao`实例并在`User`实体上运行 CRUD 操作:

```java
public class UserApplication {

    private static Dao<User> jpaUserDao;

    // standard constructors

    public static void main(String[] args) {
        User user1 = getUser(1);
        System.out.println(user1);
        updateUser(user1, new String[]{"Jake", "[[email protected]](/web/20220926183157/https://www.baeldung.com/cdn-cgi/l/email-protection)"});
        saveUser(new User("Monica", "[[email protected]](/web/20220926183157/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
        deleteUser(getUser(2));
        getAllUsers().forEach(user -> System.out.println(user.getName()));
    }

    public static User getUser(long id) {
        Optional<User> user = jpaUserDao.get(id);

        return user.orElseGet(
          () -> new User("non-existing user", "no-email"));
    }

    public static List<User> getAllUsers() {
        return jpaUserDao.getAll();
    }

    public static void updateUser(User user, String[] params) {
        jpaUserDao.update(user, params);
    }

    public static void saveUser(User user) {
        jpaUserDao.save(user);
    }

    public static void deleteUser(User user) {
        jpaUserDao.delete(user);
    }
}
```

这里的例子非常有限。但是它对于展示如何将 DAO 模式的功能与实体管理器提供的功能集成在一起非常有用。

在大多数应用程序中，有阿迪框架，它负责将一个`JpaUserDao`实例注入到`UserApplication`类中。为了简单起见，我们省略了这个过程的细节。

这里要强调的最相关的一点是**的 `JpaUserDao`类如何帮助`UserApplication`类完全不知道持久层如何执行 CRUD 操作。**

此外，我们可以将 MySQL 替换为任何其他 RDBMS(甚至替换为平面数据库),我们的应用程序将继续按预期工作，这要感谢由`Dao`接口和实体管理器提供的抽象级别。

## 4。结论

在本文中，我们深入研究了 DAO 模式的关键概念。我们看到了如何用 Java 实现它，以及如何在 JPA 的实体管理器上使用它。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926183157/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-architectural)**