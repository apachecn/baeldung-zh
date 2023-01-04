# Spring 数据 JPA @Query

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-query>

## 1。概述

Spring Data 提供了许多方法来定义我们可以执行的查询。其中之一是`@Query`注释。

在本教程中，我们将演示**如何在 Spring Data JPA 中使用`@Query`注释来执行 JPQL 和原生 SQL 查询。**

我们还将展示当`@Query`注释不够时如何构建动态查询。

## 延伸阅读:

## [Spring 数据 JPA 库中派生的查询方法](/web/20220826180223/https://www.baeldung.com/spring-data-derived-queries)

Explore the query derivation mechanism in Spring Data JPA.[Read more](/web/20220826180223/https://www.baeldung.com/spring-data-derived-queries) →

## [Spring 数据 JPA @修改注释](/web/20220826180223/https://www.baeldung.com/spring-data-jpa-modifying-annotation)

Create DML and DDL queries in Spring Data JPA by combining the @Query and @Modifying annotations[Read more](/web/20220826180223/https://www.baeldung.com/spring-data-jpa-modifying-annotation) →

## 2。选择查询

为了定义要为 Spring 数据存储库方法执行的 SQL，我们可以用`@Query`注释对该方法进行**注释——它的`value`属性包含要执行的 JPQL 或 SQL。**

`@Query`注释优先于命名查询，后者用`@NamedQuery`注释或在`orm.xml`文件中定义。

将查询定义放在存储库内的方法之上，而不是作为命名查询放在我们的域模型内，这是一个好方法。存储库负责持久性，所以它是存储这些定义的更好的地方。

### 2.1。JPQL

**默认情况下，查询定义使用 JPQL。**

让我们看一个简单的存储库方法，它从数据库返回活动的`User`实体:

```java
@Query("SELECT u FROM User u WHERE u.status = 1")
Collection<User> findAllActiveUsers(); 
```

### 2.2。原生

我们也可以使用本地 SQL 来定义我们的查询。我们所要做的就是**将`nativeQuery`属性的值设置为`true`** ，并在注释的`value`属性中定义原生 SQL 查询:

```java
@Query(
  value = "SELECT * FROM USERS u WHERE u.status = 1", 
  nativeQuery = true)
Collection<User> findAllActiveUsersNative(); 
```

## 3。在查询中定义订单

我们可以将类型为`Sort `的附加参数传递给具有`@Query`注释的 Spring 数据方法声明。它将被翻译成传递给数据库的`ORDER BY`子句。

### 3.1。JPA 提供和派生方法的排序

对于我们开箱即用的方法，比如`findAll(Sort)`或者通过解析方法签名生成的方法，**我们只能使用对象属性来定义我们的排序**:

```java
userRepository.findAll(Sort.by(Sort.Direction.ASC, "name")); 
```

现在假设我们想按名称属性的长度进行排序:

```java
userRepository.findAll(Sort.by("LENGTH(name)")); 
```

当我们执行上面的代码时，我们会收到一个异常:

> org . spring framework . data . mapping . propertyreferenceexception:未找到类型 User 的属性长度(名称)!

### 3.2。JPQL

**当我们使用 JPQL 进行查询定义时，Spring 数据可以毫无问题地处理排序**——我们所要做的就是添加一个类型为`Sort`的方法参数:

```java
@Query(value = "SELECT u FROM User u")
List<User> findAllUsers(Sort sort); 
```

我们可以调用这个方法并传递一个`Sort`参数，它将根据`User`对象的`name`属性对结果进行排序:

```java
userRepository.findAllUsers(Sort.by("name"));
```

因为我们使用了`@Query`注释，所以我们可以使用相同的方法根据名称的长度得到`Users`的排序列表:

```java
userRepository.findAllUsers(JpaSort.unsafe("LENGTH(name)")); 
```

**使用`JpaSort.unsafe()`创建`Sort`对象实例是至关重要的。**

当我们使用:

```java
Sort.by("LENGTH(name)"); 
```

然后我们会收到与上面看到的`findAll()`方法完全相同的异常。

当 Spring Data 发现使用了`@Query`注释的方法的不安全的`Sort`顺序时，它只是将 sort 子句附加到查询中——它跳过了检查排序所依据的属性是否属于域模型。

### 3.3。原生

**当`@Query`注释使用本地 SQL 时，就不可能定义`Sort`。**

如果这样做，我们将收到一个异常:

> org . spring framework . data . JPA . repository . query . invalidjpaquerymethodexception:不能将本机查询用于动态排序和/或分页

正如异常所说，本机查询不支持排序。错误消息提示我们分页也会导致异常。

但是，有一种变通方法可以实现分页，我们将在下一节中介绍它。

## 4。分页

分页允许我们在一个`Page`中返回整个结果的一个子集。例如，这在浏览网页上的几页数据时很有用。

分页的另一个优点是从服务器发送到客户端的数据量被最小化。通过发送较小的数据片段，我们通常可以看到性能的提高。

### 4.1。JPQL

在 JPQL 查询定义中使用分页非常简单:

```java
@Query(value = "SELECT u FROM User u ORDER BY id")
Page<User> findAllUsersWithPagination(Pageable pageable); 
```

**我们可以通过一个`PageRequest`参数来获取一页数据。**

本地查询也支持分页，但是需要做一些额外的工作。

### 4.2。原生

我们可以通过声明一个额外的属性`countQuery`来为本地查询启用分页。

这定义了要执行的 SQL 以计算整个结果中的行数:

```java
@Query(
  value = "SELECT * FROM Users ORDER BY id", 
  countQuery = "SELECT count(*) FROM Users", 
  nativeQuery = true)
Page<User> findAllUsersWithPagination(Pageable pageable);
```

### 4.3。2.0.4 之前的 Spring 数据 JPA 版本

上述原生查询解决方案适用于 Spring Data JPA 2 . 0 . 4 版和更高版本。

在该版本之前，当我们试图执行这样的查询时，我们将会收到与上一节中描述的排序相同的异常。

我们可以通过在查询中添加额外的分页参数来解决这个问题:

```java
@Query(
  value = "SELECT * FROM Users ORDER BY id \n-- #pageable\n",
  countQuery = "SELECT count(*) FROM Users",
  nativeQuery = true)
Page<User> findAllUsersWithPagination(Pageable pageable);
```

在上面的示例中，我们添加“\ n –# pageable \ n”作为分页参数的占位符。这告诉 Spring Data JPA 如何解析查询并注入可分页参数。这个解决方案适用于`H2`数据库。

我们已经介绍了如何通过 JPQL 和原生 SQL 创建简单的选择查询。接下来，我们将展示如何定义附加参数。

## 5。索引查询参数

有两种方法可以将方法参数传递给查询:索引参数和命名参数。

在这一节中，我们将讨论索引参数。

### 5.1。JPQL

对于 JPQL 中的索引参数，Spring 数据将**按照方法参数在方法声明**中出现的顺序将它们传递给查询:

```java
@Query("SELECT u FROM User u WHERE u.status = ?1")
User findUserByStatus(Integer status);

@Query("SELECT u FROM User u WHERE u.status = ?1 and u.name = ?2")
User findUserByStatusAndName(Integer status, String name); 
```

对于上述查询，`status`方法参数将被分配给索引为`1,`的查询参数，`name`方法参数将被分配给索引为`2`的查询参数。

### 5.2。原生

本地查询的索引参数的工作方式与 JPQL 完全相同:

```java
@Query(
  value = "SELECT * FROM Users u WHERE u.status = ?1", 
  nativeQuery = true)
User findUserByStatusNative(Integer status);
```

在下一节中，我们将展示一种不同的方法:通过名称传递参数。

## 6。命名参数

我们还可以使用命名参数将方法参数传递给查询。我们使用存储库方法声明中的`@Param`注释来定义这些。

用`@Param`标注的每个参数必须有一个与相应的 JPQL 或 SQL 查询参数名匹配的值字符串。带有命名参数的查询更容易阅读，并且在需要重构查询时不容易出错。

### 6.1。JPQL

如上所述，我们在方法声明中使用`@Param`注释来匹配 JPQL 中 name 定义的参数和方法声明中的参数:

```java
@Query("SELECT u FROM User u WHERE u.status = :status and u.name = :name")
User findUserByStatusAndNameNamedParams(
  @Param("status") Integer status, 
  @Param("name") String name); 
```

请注意，在上面的示例中，我们将 SQL 查询和方法参数定义为具有相同的名称，但是只要值字符串相同，这不是必需的:

```java
@Query("SELECT u FROM User u WHERE u.status = :status and u.name = :name")
User findUserByUserStatusAndUserName(@Param("status") Integer userStatus, 
  @Param("name") String userName); 
```

### 6.2。原生

对于原生查询定义，与 JPQL 相比，我们通过名称向查询传递参数的方式没有区别——我们使用`@Param`注释:

```java
@Query(value = "SELECT * FROM Users u WHERE u.status = :status and u.name = :name", 
  nativeQuery = true)
User findUserByStatusAndNameNamedParamsNative(
  @Param("status") Integer status, @Param("name") String name);
```

## 7。集合参数

让我们考虑一下 JPQL 或 SQL 查询的`where`子句包含`IN`(或`NOT IN`)关键字的情况:

```java
SELECT u FROM User u WHERE u.name IN :names
```

在这种情况下，我们可以定义一个以`Collection `为参数的查询方法:

```java
@Query(value = "SELECT u FROM User u WHERE u.name IN :names")
List<User> findUserByNameList(@Param("names") Collection<String> names);
```

由于参数是一个`Collection`，它可以和`List, HashSet`等一起使用。

接下来，我们将展示如何用@ `Modifying`注释修改数据。

## 8。用`@Modifying` 更新查询

我们可以**使用@ `Query`注释来修改数据库的状态，方法是将@ `Modifying`注释**添加到存储库方法中。

### 8.1。JPQL

与`select`查询相比，修改数据的存储库方法有两个不同之处——它有`@Modifying`注释，当然，JPQL 查询使用`update`而不是`select`:

```java
@Modifying
@Query("update User u set u.status = :status where u.name = :name")
int updateUserSetStatusForName(@Param("status") Integer status, 
  @Param("name") String name); 
```

返回值定义查询的执行更新了多少行。索引参数和命名参数都可以在更新查询中使用。

### 8.2。原生

我们还可以用本地查询修改数据库的状态。我们只需要添加`@Modifying`注释:

```java
@Modifying
@Query(value = "update Users u set u.status = ? where u.name = ?", 
  nativeQuery = true)
int updateUserSetStatusForNameNative(Integer status, String name);
```

### 8.3。插页

为了执行插入操作，我们必须同时应用`@Modifying`和使用本地查询，因为[插入不是 JPA 接口](/web/20220826180223/https://www.baeldung.com/jpa-insert)的一部分:

```java
@Modifying
@Query(
  value = 
    "insert into Users (name, age, email, status) values (:name, :age, :email, :status)",
  nativeQuery = true)
void insertUser(@Param("name") String name, @Param("age") Integer age, 
  @Param("status") Integer status, @Param("email") String email);
```

## 9。动态查询

通常，我们会遇到基于条件或数据集构建 SQL 语句的需求，这些条件或数据集的值只有在运行时才知道。在这些情况下，我们不能只使用静态查询。

### 9.1。动态查询的示例

例如，让我们设想这样一种情况，我们需要从运行时定义的集合中选择电子邮件为`LIKE` one 的所有用户— `email1`、`email2`、…、`emailn`:

```java
SELECT u FROM User u WHERE u.email LIKE '%email1%' 
    or  u.email LIKE '%email2%'
    ... 
    or  u.email LIKE '%emailn%'
```

由于集合是动态构造的，我们无法在编译时知道要添加多少个`LIKE`子句。

在这种情况下，**我们不能只使用`@Query`注释，因为我们不能提供静态 SQL 语句。**

相反，通过实现定制的复合存储库，我们可以扩展基本的`JpaRepository`功能，并为构建动态查询提供我们自己的逻辑。让我们来看看如何做到这一点。

### 9.2。定制存储库和 JPA 标准 API

幸运的是， **Spring 提供了一种通过使用定制片段接口来扩展基本存储库的方法。**然后我们可以将它们链接在一起，创建一个[复合存储库](/web/20220826180223/https://www.baeldung.com/spring-data-composable-repositories)。

我们将从创建一个自定义片段接口开始:

```java
public interface UserRepositoryCustom {
    List<User> findUserByEmails(Set<String> emails);
}
```

然后我们将实现它:

```java
public class UserRepositoryCustomImpl implements UserRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<User> findUserByEmails(Set<String> emails) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);

        Path<String> emailPath = user.get("email");

        List<Predicate> predicates = new ArrayList<>();
        for (String email : emails) {
            predicates.add(cb.like(emailPath, email));
        }
        query.select(user)
            .where(cb.or(predicates.toArray(new Predicate[predicates.size()])));

        return entityManager.createQuery(query)
            .getResultList();
    }
}
```

**如上所示，我们利用了 [JPA Criteria API](/web/20220826180223/https://www.baeldung.com/hibernate-criteria-queries) 来构建我们的动态查询。**

**此外，我们需要确保在类名中包含`Impl`后缀。**春将搜索到的`UserRepositoryCustom`实现为`UserRepositoryCustomImpl`。由于片段本身不是存储库，Spring 依靠这种机制来找到片段实现。

### 9.3。扩展现有存储库

注意，从第 2 节到第 7 节的所有查询方法都在`UserRepository`中。

因此，现在我们将通过扩展`UserRepository`中的新接口来集成我们的片段:

```java
public interface UserRepository extends JpaRepository<User, Integer>, UserRepositoryCustom {
    //  query methods from section 2 - section 7
}
```

### 9.4。使用存储库

最后，我们可以调用我们的动态查询方法:

```java
Set<String> emails = new HashSet<>();
// filling the set with any number of items

userRepository.findUserByEmails(emails); 
```

我们已经成功地创建了一个复合存储库，并调用了我们的自定义方法。

## 10。结论

在本文中，我们介绍了使用`@Query`注释在 Spring Data JPA 存储库方法中定义查询的几种方式。

我们还学习了如何实现一个定制的存储库和创建一个动态查询。

与往常一样，本文中使用的完整代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220826180223/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-2)