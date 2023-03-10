# Hibernate EntityManager 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-entitymanager>

## 1.介绍

**`EntityManager`是 Java 持久性 API 的一部分。首先，它实现了由 JPA 2.0 规范定义的编程接口和生命周期规则。**

此外，我们可以通过使用`EntityManager`中的 API 来访问持久性上下文。

在本教程中，我们将看看`EntityManager`的配置、类型和各种 API。

## 2.Maven 依赖性

首先，我们需要包括 Hibernate 的依赖性:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.0.Final</version>
</dependency> 
```

根据我们使用的数据库，我们还必须包括驱动程序依赖关系:

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.13</version>
</dependency>
```

Maven Central 上提供了 [hibernate-core](https://web.archive.org/web/20220906015719/https://search.maven.org/search?q=a:hibernate-core%20AND%20g:org.hibernate) 和 [mysql-connector-java](https://web.archive.org/web/20220906015719/https://search.maven.org/search?q=a:mysql-connector-java%20AND%20g:mysql) 依赖项。

## 3.配置

现在让我们通过使用一个对应于数据库中电影表的`Movie`实体来演示`EntityManager`。

在本文中，我们将利用`EntityManager` API 来处理数据库中的`Movie`对象。

### 3.1.定义实体

让我们从使用`@Entity`注释创建对应于电影表的实体开始:

```java
@Entity
@Table(name = "MOVIE")
public class Movie {

    @Id
    private Long id;

    private String movieName;

    private Integer releaseYear;

    private String language;

    // standard constructor, getters, setters
}
```

### 3.2. `persistence.xml` 文件

当`EntityManagerFactory`被创建时，**持久性实现在类路径**中搜索`META-INF/persistence.xml`文件。

**该文件包含`EntityManager` :** 的配置

```java
<persistence-unit name="com.baeldung.movie_catalog">
    <description>Hibernate EntityManager Demo</description>
    <class>com.baeldung.hibernate.pojo.Movie</class> 
    <exclude-unlisted-classes>true</exclude-unlisted-classes>
    <properties>
        <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
        <property name="hibernate.hbm2ddl.auto" value="update"/>
        <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
        <property name="javax.persistence.jdbc.url" value="jdbc:mysql://127.0.0.1:3306/moviecatalog"/>
        <property name="javax.persistence.jdbc.user" value="root"/>
        <property name="javax.persistence.jdbc.password" value="root"/>
    </properties>
</persistence-unit>
```

正如我们所看到的，我们定义了持久化单元，它指定了由`EntityManager`管理的底层数据存储。

此外，我们定义了底层数据存储的方言和其他 JDBC 属性。Hibernate 是数据库无关的。基于这些属性，Hibernate 连接底层数据库。

## 4.容器和应用程序管理`EntityManager`

基本上，**有两种类型的`EntityManager`:容器管理的和应用程序管理的。**

让我们仔细看看每一种类型。

### 4.1.容器管理的`EntityManager`

这里，容器在我们的企业组件中注入了`EntityManager`。

换句话说，容器从`EntityManagerFactory`为我们创建了`EntityManager`:

```java
@PersistenceContext
EntityManager entityManager; 
```

这也意味着容器负责开始事务，以及提交或回滚事务。

类似地，容器负责关闭`EntityManager, `，因此使用是安全的，无需手动清理。即使我们试图`close` 一个容器管理的`EntityManager`，它也应该抛出一个`IllegalStateException.`

### 4.2.应用程序管理的`EntityManager`

相反，`EntityManager`的生命周期由应用程序管理。

事实上，我们将手动创建`EntityManager`，并管理它的生命周期。

首先，让我们创建`EntityManagerFactory:`

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("com.baeldung.movie_catalog");
```

为了创建一个`EntityManager`，我们必须在`EntityManagerFactory`中显式调用`createEntityManager()`:

```java
public static EntityManager getEntityManager() {
    return emf.createEntityManager();
}
```

因为我们负责创建`EntityManager `实例，所以关闭它们`. `也是我们的责任。因此，当我们使用完它们时，我们应该`close `每个`EntityManager`。

### 4.3.线程安全

**`EntityManagerFactory`实例以及 Hibernate 的`[SessionFactory](https://web.archive.org/web/20220906015719/https://github.com/hibernate/hibernate-orm/blob/716a8bac2080fdcce349550e1817b14a100f3b74/hibernate-core/src/main/java/org/hibernate/SessionFactory.java#L32)`实例都是线程安全的**。因此，在并发上下文中编写以下代码是完全安全的:

```java
EntityManagerFactory emf = // fetched from somewhere
EntityManager em = emf.createEntityManager();
```

另一方面，**`EntityManager`实例不是线程安全的，应该在线程受限的环境中使用**。这意味着每个线程都应该获得它的实例，使用它，并在最后关闭它。

当使用应用程序管理的`EntityManager`时，很容易创建线程限制的实例:

```java
EntityManagerFactory emf = // fetched from somewhere 
EntityManager em = emf.createEntityManager();
// use it in the current thread
```

然而，当使用容器管理的`EntityManager` s:

```java
@Service
public class MovieService {

    @PersistenceContext // or even @Autowired
    private EntityManager entityManager;

    // omitted
}
```

似乎所有操作都应该共享一个`EntityManager `实例。**然而，容器(JakartaEE 或 Spring)注入了一个特殊的代理，而不是这里的简单的`EntityManager `**。例如，Spring 注入了一个`[SharedEntityManagerCreator](https://web.archive.org/web/20220906015719/https://github.com/spring-projects/spring-framework/blob/master/spring-orm/src/main/java/org/springframework/orm/jpa/SharedEntityManagerCreator.java#L71). `类型的代理

每次我们使用注入的`EntityManager, `时，这个代理要么重用现有的`EntityManager `要么创建一个新的。重用通常发生在我们启用像`[Open Session/EntityManager in View](/web/20220906015719/https://www.baeldung.com/spring-open-session-in-view). `这样的东西的时候

无论哪种方式，**容器确保每个`EntityManager `被限制在一个线程**中。

## 5.休眠实体操作

`EntityManager` API 提供了一组方法。我们可以利用这些方法与数据库进行交互。

### 5.1.持久实体

为了让一个对象与 EntityManager 相关联，我们可以利用`persist()`方法:

```java
public void saveMovie() {
    EntityManager em = getEntityManager();

    em.getTransaction().begin();

    Movie movie = new Movie();
    movie.setId(1L);
    movie.setMovieName("The Godfather");
    movie.setReleaseYear(1972);
    movie.setLanguage("English");

    em.persist(movie);
    em.getTransaction().commit();
}
```

**一旦对象保存在数据库中，它就处于`persistent`状态。**

### 5.2.加载实体

**为了从数据库中检索对象，我们可以使用`find()`方法。**

这里，该方法通过主键进行搜索。事实上，该方法需要实体类类型和主键:

```java
public Movie getMovie(Long movieId) {
    EntityManager em = getEntityManager();
    Movie movie = em.find(Movie.class, new Long(movieId));
    em.detach(movie);
    return movie;
}
```

**但是，如果我们只需要引用实体，我们可以用`getReference()`** 的方法来代替。实际上，它向实体返回一个代理:

```java
Movie movieRef = em.getReference(Movie.class, new Long(movieId));
```

### 5.3.分离实体

如果我们需要从持久上下文中分离一个实体，**我们可以使用`detach()`方法**。我们将要分离的对象作为参数传递给方法:

```java
em.detach(movie);
```

一旦实体从持久性上下文中分离出来，它将处于分离状态。

### 5.4.合并实体

实际上，许多应用程序需要跨多个事务修改实体。例如，我们可能希望在一个事务中检索一个实体，以便呈现给 UI。然后，另一个事务将带来 UI 中所做的更改。

对于这种情况，我们可以使用`merge()`方法。**合并方法有助于将对分离实体所做的任何修改带入受管实体:**

```java
public void mergeMovie() {
    EntityManager em = getEntityManager();
    Movie movie = getMovie(1L);
    em.detach(movie);
    movie.setLanguage("Italian");
    em.getTransaction().begin();
    em.merge(movie);
    em.getTransaction().commit();
}
```

### 5.5.查询实体

此外，我们可以利用 JPQL 来查询实体。我们将调用`getResultList()`来执行它们。

当然，如果查询只返回一个对象，我们可以使用`getSingleResult()`:

```java
public List<?> queryForMovies() {
    EntityManager em = getEntityManager();
    List<?> movies = em.createQuery("SELECT movie from Movie movie where movie.language = ?1")
      .setParameter(1, "English")
      .getResultList();
    return movies;
}
```

### 5.6.删除实体

另外，**我们可以使用`remove()`方法**从数据库中删除一个实体。需要注意的是，对象不是分离的，而是被移除的。

在这里，实体的状态从持久变为新:

```java
public void removeMovie() {
    EntityManager em = HibernateOperations.getEntityManager();
    em.getTransaction().begin();
    Movie movie = em.find(Movie.class, new Long(1L));
    em.remove(movie);
    em.getTransaction().commit();
}
```

## 6.结论

在本文中，我们探索了 Hibernate 中的`EntityManager`。我们查看了类型和配置，并了解了 API 中用于处理持久性上下文的各种方法。

和往常一样，本文中使用的代码可以从 Github 上的[处获得。](https://web.archive.org/web/20220906015719/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)