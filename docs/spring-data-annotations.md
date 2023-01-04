# Spring 数据注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-annotations>

[This article is part of a series:](javascript:void(0);)[• Spring Core Annotations](/web/20220630020130/https://www.baeldung.com/spring-core-annotations)
[• Spring Web Annotations](/web/20220630020130/https://www.baeldung.com/spring-mvc-annotations)
[• Spring Boot Annotations](/web/20220630020130/https://www.baeldung.com/spring-boot-annotations)
[• Spring Scheduling Annotations](/web/20220630020130/https://www.baeldung.com/spring-scheduling-annotations)
• Spring Data Annotations (current article)[• Spring Bean Annotations](/web/20220630020130/https://www.baeldung.com/spring-bean-annotations)

## 1.介绍

Spring Data 提供了对数据存储技术的抽象。因此，我们的业务逻辑代码可以更加独立于底层的持久性实现。此外，Spring 简化了与实现相关的数据存储细节的处理。

在本教程中，我们将看到 Spring Data、Spring Data JPA 和 Spring Data MongoDB 项目的最常见注释。

## 2.常见的 Spring 数据注释

### 2.1.`@Transactional`

当我们想要**配置方法**的事务行为时，我们可以使用:

```java
@Transactional
void pay() {}
```

如果我们在类级别应用这个注释，那么它对类中的所有方法都有效。然而，我们可以通过将它应用于特定的方法来覆盖它的效果。

它有很多配置选项，可以在[这篇文章](/web/20220630020130/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)中找到。

### 2.2.`@NoRepositoryBean`

有时我们想要创建存储库接口，唯一的目标是为子存储库提供公共方法。

当然，我们不希望 Spring 创建这些存储库的 bean，因为我们不会将它们注入任何地方。`@NoRepositoryBean `就是这样做的:当我们标记一个`org.springframework.data.repository.Repository`的子接口时，Spring 不会用它创建一个 bean。

例如，如果我们想要在我们所有的存储库中有一个`Optional<T> findById(ID id) `方法，我们可以创建一个基础存储库:

```java
@NoRepositoryBean
interface MyUtilityRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
    Optional<T> findById(ID id);
}
```

此注释不影响子接口；因此，Spring 将为以下存储库接口创建一个 bean:

```java
@Repository
interface PersonRepository extends MyUtilityRepository<Person, Long> {}
```

注意，上面的例子是不必要的，因为 Spring Data version 2 包含了这个方法，取代了旧的`T findOne(ID id)`。

### 2.3.`@Param`

我们可以使用`@Param`将命名参数传递给我们的查询:

```java
@Query("FROM Person p WHERE p.name = :name")
Person findByName(@Param("name") String name);
```

注意，我们用`:name `语法引用参数。

更多例子，请访问[这篇文章](/web/20220630020130/https://www.baeldung.com/spring-data-jpa-query)。

### 2.4.`@Id`

`@Id `将模型类中的字段标记为主键:

```java
class Person {

    @Id
    Long id;

    // ...

}
```

因为它是独立于实现的，所以它使得模型类易于与多个数据存储引擎一起使用。

### 2.5.`@Transient`

我们可以使用这个注释将模型类中的字段标记为瞬态的。因此，数据存储引擎不会读取或写入该字段的值:

```java
class Person {

    // ...

    @Transient
    int age;

    // ...

}
```

与`@Id`一样，`@Transient `也是独立于实现的，这使得它可以方便地用于多个数据存储实现。

### 2.6.`@CreatedBy`、`@LastModifiedBy`、`@CreatedDate`、`@LastModifiedDate`

有了这些注释，我们可以审计我们的模型类:Spring 自动用创建对象的主体、最后修改对象的主体、创建日期和最后修改日期填充注释字段:

```java
public class Person {

    // ...

    @CreatedBy
    User creator;

    @LastModifiedBy
    User modifier;

    @CreatedDate
    Date createdAt;

    @LastModifiedDate
    Date modifiedAt;

    // ...

}
```

注意，如果我们希望 Spring 填充主体，我们也需要使用 Spring 安全性。

如需更详细的描述，请访问本文。

## 3.Spring 数据 JPA 注释

### 3.1.`@Query`

使用`@Query`，我们可以为存储库方法提供一个 JPQL 实现:

```java
@Query("SELECT COUNT(*) FROM Person p")
long getPersonCount();
```

此外，我们可以使用命名参数:

```java
@Query("FROM Person p WHERE p.name = :name")
Person findByName(@Param("name") String name);
```

此外，我们可以使用本地 SQL 查询，如果我们将参数`nativeQuery `设置为`true`:

```java
@Query(value = "SELECT AVG(p.age) FROM person p", nativeQuery = true)
int getAverageAge();
```

欲了解更多信息，请访问[这篇文章](/web/20220630020130/https://www.baeldung.com/spring-data-jpa-query)。

### 3.2.`@Procedure`

有了 Spring Data JPA，我们可以轻松地从存储库中调用存储过程。

首先，我们需要使用标准 JPA 注释在实体类上声明存储库:

```java
@NamedStoredProcedureQueries({ 
    @NamedStoredProcedureQuery(
        name = "count_by_name", 
        procedureName = "person.count_by_name", 
        parameters = { 
            @StoredProcedureParameter(
                mode = ParameterMode.IN, 
                name = "name", 
                type = String.class),
            @StoredProcedureParameter(
                mode = ParameterMode.OUT, 
                name = "count", 
                type = Long.class) 
            }
    ) 
})

class Person {}
```

此后，我们可以在存储库中用我们在`name `参数中声明的名称来引用它:

```java
@Procedure(name = "count_by_name")
long getCountByName(@Param("name") String name);
```

### 3.3.`@Lock`

我们可以在执行存储库查询方法时配置锁定模式:

```java
@Lock(LockModeType.NONE)
@Query("SELECT COUNT(*) FROM Person p")
long getPersonCount();
```

可用的锁定模式:

*   `READ`
*   `WRITE`
*   `OPTIMISTIC`
*   `OPTIMISTIC_FORCE_INCREMENT`
*   `PESSIMISTIC_READ`
*   `PESSIMISTIC_WRITE`
*   `PESSIMISTIC_FORCE_INCREMENT`
*   `NONE`

### 3.4.`@Modifying`

如果我们用`@Modifying`注释数据，我们可以用存储库方法修改数据:

```java
@Modifying
@Query("UPDATE Person p SET p.name = :name WHERE p.id = :id")
void changeName(@Param("id") long id, @Param("name") String name);
```

欲了解更多信息，请访问[这篇文章](/web/20220630020130/https://www.baeldung.com/spring-data-jpa-query)。

### 3.5.`@EnableJpaRepositories`

要使用 JPA 存储库，我们必须将它指示给 Spring。我们可以用`@EnableJpaRepositories.`来做这件事

注意，我们必须将这个注释与`@Configuration`一起使用:

```java
@Configuration
@EnableJpaRepositories
class PersistenceJPAConfig {}
```

Spring 将在这个`@Configuration `类的子包中寻找存储库。

我们可以用`basePackages `参数改变这种行为:

```java
@Configuration
@EnableJpaRepositories(basePackages = "com.baeldung.persistence.dao")
class PersistenceJPAConfig {}
```

还要注意，如果 Spring Boot 在类路径中找到 Spring 数据 JPA，它会自动这么做。

## 4.Spring 数据 Mongo 注释

Spring 数据使得使用 MongoDB 更加容易。在接下来的小节中，我们将探索 Spring Data MongoDB 的最基本的特性。

更多信息，请访问我们关于 Spring Data MongoDB 的[文章。](/web/20220630020130/https://www.baeldung.com/spring-data-mongodb-tutorial)

### 4.1.`@Document`

该注释将一个类标记为我们希望保存到数据库中的域对象:

```java
@Document
class User {}
```

它还允许我们选择想要使用的集合的名称:

```java
@Document(collection = "user")
class User {}
```

注意，这个注释是 JPA 中`@Entity `的 Mongo 等价物。

### 4.2.`@Field`

使用`@Field`，我们可以配置当 MongoDB 持久化文档时我们想要使用的字段的名称:

```java
@Document
class User {

    // ...

    @Field("email")
    String emailAddress;

    // ...

}
```

注意，这个注释是 JPA 中`@Column `的 Mongo 等价物。

### 4.3.`@Query`

使用`@Query`，我们可以在 MongoDB 存储库方法上提供一个 finder 查询:

```java
@Query("{ 'name' : ?0 }")
List<User> findUsersByName(String name);
```

### 4.4.`@EnableMongoRepositories`

要使用 MongoDB 存储库，我们必须将它指示给 Spring。我们可以用`@EnableMongoRepositories.`来做这件事

注意，我们必须将这个注释与`@Configuration`一起使用:

```java
@Configuration
@EnableMongoRepositories
class MongoConfig {}
```

Spring 将在这个`@Configuration `类的子包中寻找存储库。我们可以用`basePackages `参数改变这种行为:

```java
@Configuration
@EnableMongoRepositories(basePackages = "com.baeldung.repository")
class MongoConfig {}
```

还要注意，如果 Spring Boot 在类路径中找到了 Spring 数据 MongoDB，它会自动这么做。

## 5.结论

在本文中，我们看到了使用 Spring 处理数据时需要的最重要的注释。此外，我们研究了最常见的 JPA 和 MongoDB 注释。

像往常一样，GitHub 上的例子[这里](https://web.archive.org/web/20220630020130/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa)是关于 common 和 JPA 注释的，这里[是关于 MongoDB 注释的。](https://web.archive.org/web/20220630020130/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb)

Next **»**[Spring Bean Annotations](/web/20220630020130/https://www.baeldung.com/spring-bean-annotations)**«** Previous[Spring Scheduling Annotations](/web/20220630020130/https://www.baeldung.com/spring-scheduling-annotations)