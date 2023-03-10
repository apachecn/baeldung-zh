# DeltaSpike 数据模块指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/deltaspike-data-module>

## 1。概述

Apache DeltaSpike 是一个为 Java 项目提供 CDI 扩展的 T2 集合的项目；它要求 CDI 实现在运行时可用。

**当然，它可以与 CDI** 的不同实现一起工作——JBoss Weld 或 OpenWebBeans。它还在许多应用服务器上进行了测试。

在本教程中，我们将关注最著名和最有用的数据模块之一。

## 2。DeltaSpike 数据模块设置

Apache DeltaSpike 数据模块用于**简化存储库模式**的实现。它允许**通过为查询创建和执行提供集中逻辑来减少样板代码**。

这非常类似于[春季数据](/web/20221205135355/http://www.baeldung.com/spring-data)项目。为了查询一个数据库，我们需要定义一个方法声明(没有实现),它遵循已定义的命名约定或者包含`@Query`注释。CDI 扩展将为我们完成实现。

在接下来的小节中，我们将介绍如何在我们的应用程序中设置 Apache DeltaSpike 数据模块。

### 2.1。所需的依赖关系

为了在应用程序中使用 Apache DeltaSpike 数据模块，我们需要设置所需的依赖关系。

当 Maven 是我们的构建工具时，我们必须使用:

```java
<dependency>
    <groupId>org.apache.deltaspike.modules</groupId>
    <artifactId>deltaspike-data-module-api</artifactId>
    <version>1.8.2</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.apache.deltaspike.modules</groupId>
    <artifactId>deltaspike-data-module-impl</artifactId>
    <version>1.8.2</version>
    <scope>runtime</scope>
</dependency>
```

当我们使用 Gradle 时:

```java
runtime 'org.apache.deltaspike.modules:deltaspike-data-module-impl'
compile 'org.apache.deltaspike.modules:deltaspike-data-module-api' 
```

Apache DeltaSpike 数据模块工件可以在 Maven Central 上获得:

*   [delta spike-data-module-impl](https://web.archive.org/web/20221205135355/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22deltaspike-data-module-impl%22)
*   [delta spike-data-module-API](https://web.archive.org/web/20221205135355/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22deltaspike-data-module-api%22)

为了用数据模块运行应用程序，我们还需要在运行时可用的 JPA 和 CDI 实现。

尽管可以在 Java SE 应用程序中运行 Apache DeltaSpike，但在大多数情况下，它将部署在应用服务器上(例如 Wildfly 或 WebSphere)。

应用服务器完全支持 Jakarta EE，因此我们无需再做任何事情。在 Java SE 应用程序的情况下，我们必须提供这些实现(例如，通过添加对 Hibernate 和 JBoss Weld 的依赖)。

接下来，我们还将介绍`EntityManager`所需的配置。

### 2.2。实体管理器配置

**数据模块要求`EntityManager`通过 CDI** 注入。

我们可以通过使用 CDI 生成器来实现这一点:

```java
public class EntityManagerProducer {

    @PersistenceContext(unitName = "primary")
    private EntityManager entityManager;

    @ApplicationScoped
    @Produces
    public EntityManager getEntityManager() {
        return entityManager;
    }
}
```

上面的代码假设我们在`persistence.xml`文件中定义了名为`primary`的持久性单元。

下面我们来看一个定义的例子:

```java
<persistence-unit name="primary" transaction-type="JTA">
   <jta-data-source>java:jboss/datasources/baeldung-jee7-seedDS</jta-data-source>
   <properties>
      <property name="hibernate.hbm2ddl.auto" value="create-drop" />
      <property name="hibernate.show_sql" value="false" />
   </properties>
</persistence-unit>
```

我们示例中的持久性单元使用 JTA 事务类型，这意味着我们必须提供一个将要使用的事务策略。

### 2.3。交易策略

**如果我们使用 JTA 事务类型作为数据源，那么我们必须定义将在 Apache DeltaSpike 存储库中使用的事务策略**。我们可以在`apache-deltaspike.properties`文件中完成(在`META-INF`目录下):

```java
globalAlternatives.org.apache.deltaspike.jpa.spi.transaction.TransactionStrategy=org.apache.deltaspike.jpa.impl.transaction.ContainerManagedTransactionStrategy
```

我们可以定义四种类型的交易策略:

*   `BeanManagedUserTransactionStrategy`
*   `ResourceLocalTransactionStrategy`
*   `ContainerManagedTransactionStrategy`
*   `EnvironmentAwareTransactionStrategy`

全部实现`org.apache.deltaspike.jpa.spi.transaction.TransactionStrategy`。

这是我们的数据模块所需配置的最后一部分。

接下来，我们将展示如何实现存储库模式类。

## 3。储存库类别

当我们使用 Apache DeltaSpike 数据模块**时，任何抽象类或接口都可以成为存储库**类。

**我们所要做的就是** **添加一个`@Repository`** **注释**和一个`forEntity`属性，它定义了我们的存储库应该处理的 JPA 实体:

```java
@Entity
public class User {
    // ...
}  

@Repository(forEntity = User.class) 
public interface SimpleUserRepository { 
    // ... 
}
```

或者用一个抽象类:

```java
@Repository(forEntity = User.class)
public abstract class SimpleUserRepository { 
    // ... 
} 
```

数据模块发现带有这种注释的类(或接口),并处理其中的方法。

定义要执行的查询的可能性很小。我们将在接下来的小节中逐一介绍。

## 4。从方法名查询

定义查询的第一种可能性是使用遵循已定义命名约定的方法名。

它看起来像下面这样:

```java
(Entity|Optional<Entity>|List<Entity>|Stream<Entity>) (prefix)(Property[Comparator]){Operator Property [Comparator]} 
```

接下来，我们将关注这个定义的每一部分。

### 4.1。返回类型

**返回类型主要定义我们的查询可能返回多少个对象**。我们不能将单个实体类型定义为返回值，以防我们的查询可能返回多个结果。

如果有多个具有给定名称的`User`,下面的方法将抛出异常:

```java
public abstract User findByFirstName(String firstName);
```

反之则不然——我们可以将返回值定义为`Collection`,即使结果只是一个实体。

```java
public abstract Collection<User> findAnyByFirstName(String firstName);
```

在我们将返回值定义为`Collection`的情况下，建议将一个值作为返回类型的方法名前缀(例如`findAny`)被取消。

上面的查询将返回名字匹配的所有`Users`,即使方法名前缀暗示了不同的东西。

应该避免这种组合(`Collection`返回类型和一个表示单值返回的前缀)，因为代码变得不直观且难以理解。

下一节将展示更多关于方法名前缀的细节。

### 4.2。查询方法的前缀

**前缀定义了我们想要在存储库**上执行的动作。最有用的一种方法是找到符合给定搜索标准的实体。

这个动作有很多前缀，比如`findBy`、`findAny`、`findAll. `详细列表请查看官方 Apache DeltaSpike [文档](https://web.archive.org/web/20221205135355/https://deltaspike.apache.org/documentation/data.html#UsingMethodExpressions):

```java
public abstract User findAnyByLastName(String lastName);
```

但是，也有**其他方法模板用于计数和移除实体**。我们可以`count`表格中的所有行:

```java
public abstract int count();
```

另外，`remove`方法模板已经存在，我们可以将其添加到我们的存储库中:

```java
public abstract void remove(User user);
```

对`countBy`和`removeBy`方法前缀的支持将在 Apache DeltaSpike 1.9.0 的下一版本中添加。

下一节将展示如何向查询中添加更多的属性。

### 4.3。多属性查询

在查询中，我们可以使用**多属性结合`and`运算符**。

```java
public abstract Collection<User> findByFirstNameAndLastName(
  String firstName, String lastName);
public abstract Collection<User> findByFirstNameOrLastName(
  String firstName, String lastName); 
```

我们可以组合任意多的属性。还可以搜索嵌套属性，我们接下来将展示这一点。

### 4.4。具有嵌套属性的查询

**查询也可以使用嵌套属性**。

在下面的示例中，`User`实体具有类型为`Address`的地址属性，而`Address`实体具有类型为`city`的属性:

```java
@Entity
public class Address {
private String city;
    // ...
}
@Entity
public class User {
    @OneToOne 
    private Address address;
    // ...
}
public abstract Collection<User> findByAddress_city(String city);
```

### 4.5。订单查询中的

DeltaSpike 允许我们**定义结果返回的顺序**。我们可以定义升序和降序:

```java
public abstract List<User> findAllOrderByFirstNameAsc();
```

如上所示，我们所要做的就是给方法名添加一个部分，它包含我们想要排序的属性名和订单方向的简称。

我们可以轻松合并许多订单:

```java
public abstract List<User> findAllOrderByFirstNameAscLastNameDesc(); 
```

接下来，我们将展示如何限制查询结果的大小。

### 4.6。限制查询结果大小和分页

有些情况下，我们希望从整个结果中检索前几行。这就是所谓的查询限制。数据模块也很简单:

```java
public abstract Collection<User> findTop2OrderByFirstNameAsc();
public abstract Collection<User> findFirst2OrderByFirstNameAsc();
```

`First`和`top`可以互换使用。

然后我们可以通过提供两个额外的参数`@FirstResult`和`@MaxResult`**来启用查询分页:**

```java
public abstract Collection<User> findAllOrderByFirstNameAsc(@FirstResult int start, @MaxResults int size);
```

我们已经在库中定义了很多方法。其中一些是通用，应该定义一次，供每个存储库使用。

Apache DeltaSpike 提供了很少的基本类型，我们可以使用它们来获得大量现成的方法。

在下一节中，我们将重点讨论如何做到这一点。

## 5。基本存储库类型

为了**获得一些基本的存储库方法，我们的存储库应该扩展 Apache DeltaSpike** 提供的基本类型。有一些像`EntityRepository`、`FullEntityRepository,` 等。：

```java
@Repository
public interface UserRepository 
  extends FullEntityRepository<User, Long> {
    // ...
}
```

或者使用抽象类:

```java
@Repository
public abstract class UserRepository extends AbstractEntityRepository<User, Long> {
    // ...
} 
```

上面的实现为我们提供了许多方法，而无需编写额外的代码行，因此我们得到了我们想要的——我们大量减少了样板代码。

如果我们使用基本存储库类型，就不需要向我们的*@存储库*注释*传递额外的 *forEntity* 属性值。*

当我们使用抽象类而不是接口来存储库时，我们获得了创建自定义查询的额外可能性。

**抽象库基类，例如`AbstractEntityRepository`给我们提供了对字段(通过 getters)或实用方法的访问，我们可以使用它们来创建查询**:

```java
public List<User> findByFirstName(String firstName) {
    return typedQuery("select u from User u where u.firstName = ?1")
      .setParameter(1, firstName)
      .getResultList();
} 
```

在上面的例子中，我们使用了一个`typedQuery`工具方法来创建一个定制的实现。

创建查询的最后一种可能是使用`@Query`注释，我们将在下面展示。

## 6。`@Query`注解

要执行的 SQL **查询也可以用`@Query`注释**来定义。这非常类似于弹簧解决方案。我们必须用 SQL query 作为值向方法添加一个注释。

默认情况下，这是一个 JPQL 查询:

```java
@Query("select u from User u where u.firstName = ?1")
public abstract Collection<User> findUsersWithFirstName(String firstName); 
```

如上例所示，我们可以通过索引轻松地将参数传递给查询。

如果我们想通过原生 SQL 而不是 JPQL 传递查询，我们需要定义额外的查询属性–`isNative`以及真值:

```java
@Query(value = "select * from User where firstName = ?1", isNative = true)
public abstract Collection<User> findUsersWithFirstNameNative(String firstName);
```

## 7。结论

在本文中，我们介绍了 Apache DeltaSpike 的基本定义，并重点介绍了激动人心的部分——数据模块。这与 Spring Data 项目非常相似。

我们探讨了如何实现存储库模式。我们还介绍了如何定义要执行的查询的三种可能性。

与往常一样，本文中使用的完整代码示例可以在 Github 上的[处获得。](https://web.archive.org/web/20221205135355/https://github.com/eugenp/tutorials/tree/master/persistence-modules/deltaspike)