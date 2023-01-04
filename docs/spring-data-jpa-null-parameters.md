# Spring 数据 JPA 和空参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-null-parameters>

## 1。概述

在本教程中，我们将展示如何处理 [Spring 数据 JPA](/web/20220628114545/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 中的`null`参数。

在某些情况下，当我们通过参数**搜索记录时，我们希望找到字段值为`null`的行。**其他时候，我们想要忽略一个`null`并且**在我们的查询中跳过那个字段。**

下面我们将展示如何实现这些。

## 2。快速示例

假设我们有一个`Customer`实体:

```java
@Entity
public class Customer {

    @Id
    @GeneratedValue
    private long id;
    private String name;
    private String email;

    public Customer(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // getters/setters

}
```

此外，我们还有一个 JPA 存储库:

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> { 

   // method1
   // method2
}
```

我们想通过`name`和`email`来搜索客户。

为此，我们将编写两个不同地处理`null`参数的方法。

## 3。处理`Null`参数的方法

首先，我们将创建一个方法，将参数的`null`值解释为`IS NULL`。然后，我们将创建一个忽略`null`参数并将其从 WHERE 子句中排除的方法。

### 3.1。`IS NULL`查询

第一种方法创建起来非常简单，因为查询方法中的`null`参数默认被解释为 `IS NULL`。

让我们创建方法:

```java
List<Customer> findByNameAndEmail(String name, String email);
```

现在，如果我们传递一封`null`电子邮件，生成的 JPQL 将包含`IS NULL`条件:

```java
customer0_.email is null
```

为了演示这一点，让我们创建一个测试。

首先，我们将一些客户添加到存储库中:

```java
@Before
public void before() {
    entityManager.persist(new Customer("A", "[[email protected]](/web/20220628114545/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    entityManager.persist(new Customer("D", null));
    entityManager.persist(new Customer("D", "[[email protected]](/web/20220628114545/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
}
```

现在让我们将`“D”` 作为参数`name` 的值，将`null`作为参数`email`的值传递给我们的查询方法。

我们可以看到，只找到一个客户:

```java
List<Customer> customers = repository.findByNameAndEmail("D", null);

assertEquals(1, customers.size());

Customer actual = customers.get(0);

assertEquals(null, actual.getEmail());
assertEquals("D", actual.getName());
```

### 3.2。用替代方法避免`null`参数

有时候我们想忽略一些参数，不在`WHERE`子句中包含它们对应的字段。

我们可以向我们的存储库添加更多的查询方法。

例如，要忽略`email`，我们可以添加一个只接受`name`的方法:

```java
 List<Customer> findByName(String name);
```

但是这种忽略我们的一个列的方式随着数量的增加伸缩性很差，因为我们必须添加许多方法来实现所有的组合。

### 3.3。使用`@Query`注释忽略`null`参数

我们可以通过使用 `@Query`注释并给 JPQL 语句添加一个小的复杂性来避免创建额外的方法:

```java
@Query("SELECT c FROM Customer c WHERE (:name is null or c.name = :name) and (:email is null"
  + " or c.email = :email)")
List<Customer> findCustomerByNameAndEmail(@Param("name") String name, @Param("email") String email);
```

注意，如果`email`参数是`null`，那么子句总是`true`，因此不会影响整个`WHERE`子句:

```java
:email is null or s.email = :email
```

让我们确保这是可行的:

```java
List<Customer> customers = repository.findCustomerByNameAndEmail("D", null);

assertEquals(2, customers.size());
```

我们发现名为 `“D”`的两位客户忽略了他们的电子邮件。

生成的 JPQL WHERE 子句如下所示:

```java
where (? is null or customer0_.name=?) and (? is null or customer0_.email=?)
```

通过这种方法，我们相信数据库服务器能够识别关于查询参数为`null`的子句，并优化查询的执行计划，这样就不会有很大的性能开销。对于一些查询或数据库服务器，尤其是涉及到大量的表扫描时，可能会有性能开销。

## 4。结论

在本文中，我们演示了 Spring Data JPA 如何解释查询方法中的`null`参数，以及如何更改默认行为。

也许在将来，我们能够使用 [`@NullMeans`](https://web.archive.org/web/20220628114545/https://jira.spring.io/browse/DATAJPA-209) 注释来指定如何解释`null`参数。请注意，这是目前提议的功能，仍在考虑中。

总而言之，有两种主要的方法来解释`null`参数，并且它们都由建议的`@NullMeans`注释提供:

*   `IS (is null) `–第 3.1 节中演示的默认选项。
*   `IGNORED`(从`WHERE`子句中排除一个`null`参数)——通过额外的查询方法实现(第 3.2 节)。)或使用变通方法(第 3.3 节)。)

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628114545/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-filtering)