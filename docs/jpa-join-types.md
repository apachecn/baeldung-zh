# JPA 连接类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-join-types>

## 1.概观

在本教程中，我们将看看 [JPA](/web/20221107131215/https://www.baeldung.com/jpa-hibernate-difference) 支持的不同连接类型。

为此，我们将使用 JPQL，[一种 JPA](/web/20221107131215/https://www.baeldung.com/jpa-queries) 的查询语言。

## 2.样本数据模型

让我们看看我们将在示例中使用的样本数据模型。

首先，我们将创建一个`Employee`实体:

```java
@Entity
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    private String name;

    private int age;

    @ManyToOne
    private Department department;

    @OneToMany(mappedBy = "employee")
    private List<Phone> phones;

    // getters and setters...
}
```

每个`Employee` 将只分配给一个`Department`:

```java
@Entity
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String name;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;

    // getters and setters...
}
```

最后，每个`Employee` 将有多个`Phone`:

```java
@Entity
public class Phone {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    private String number;

    @ManyToOne
    private Employee employee;

    // getters and setters...
}
```

## 3.内部联接

我们将从内部连接开始。当两个或多个实体进行内部联接时，结果中只收集符合联接条件的记录。

### 3.1.带有单值关联导航的隐式内部连接

内部连接可以是`implicit.`顾名思义，**开发者没有指定隐式内部连接**。每当我们导航单值关联时，JPA 会自动创建一个隐式连接:

```java
@Test
public void whenPathExpressionIsUsedForSingleValuedAssociation_thenCreatesImplicitInnerJoin() {
    TypedQuery<Department> query
      = entityManager.createQuery(
          "SELECT e.department FROM Employee e", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

这里，`Employee`实体与`Department`实体有多对一的关系。**如果我们从一个`Employee`实体导航到她指定`e.department,`** 的`Department,` ，我们将导航到一个单值关联。因此，JPA 将创建一个内部连接。此外，连接条件将从映射元数据中导出。

### 3.2.具有单值关联的显式内部连接

接下来我们将看看`explicit`内部连接，其中**我们在 [JPQL 查询](/web/20221107131215/https://www.baeldung.com/jpa-queries) :** 中使用了 JOIN 关键字

```java
@Test
public void whenJoinKeywordIsUsed_thenCreatesExplicitInnerJoin() {
    TypedQuery<Department> query
      = entityManager.createQuery(
          "SELECT d FROM Employee e JOIN e.department d", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

在这个查询中，**我们在 FROM 子句**中指定了一个 JOIN 关键字和相关的`Department`实体，而在前面的查询中它们根本没有被指定。然而，除了这种语法上的不同，产生的 SQL 查询将非常相似。

我们还可以指定一个可选的内部关键字:

```java
@Test
public void whenInnerJoinKeywordIsUsed_thenCreatesExplicitInnerJoin() {
    TypedQuery<Department> query
      = entityManager.createQuery(
          "SELECT d FROM Employee e INNER JOIN e.department d", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

那么既然 JPA 自动创建了一个隐式的内部连接，我们什么时候需要显式的呢？

首先， **JPA 只在我们指定路径表达式的时候创建一个隐式内连接。**例如，当我们想只选择有`Department,` 的`Employee`时，我们不使用像`e.department`这样的路径表达式，我们应该在查询中使用 JOIN 关键字。

第二，当我们很明确的时候，就更容易知道发生了什么。

### 3.3.具有集值关联的显式内部连接

我们需要明确的另一个地方是集合值关联。

如果我们看一下我们的数据模型，`Employee` 与`Phone`有一对多的关系。正如前面的例子一样，我们可以尝试编写一个类似的查询:

```java
SELECT e.phones FROM Employee e
```

**然而，这并不完全符合我们的预期**。由于选择的关联，`e.phones,` 是集合值的，**我们将得到一个`Collection`的列表，而不是`Phone` 实体**:

```java
@Test
public void whenCollectionValuedAssociationIsSpecifiedInSelect_ThenReturnsCollections() {
    TypedQuery<Collection> query 
      = entityManager.createQuery(
          "SELECT e.phones FROM Employee e", Collection.class);
    List<Collection> resultList = query.getResultList();

    //Assertions
}
```

此外，如果我们想在 WHERE 子句中过滤`Phone`实体，JPA 不允许这样做。这是因为**路径表达式不能从集合值关联**继续。所以比如说，**的`e.phones.number`就不是有效的**。

相反，我们应该创建一个显式的内部连接，并为`Phone`实体创建一个别名。然后我们可以在 SELECT 或 WHERE 子句中指定`Phone`实体:

```java
@Test
public void whenCollectionValuedAssociationIsJoined_ThenCanSelect() {
    TypedQuery<Phone> query 
      = entityManager.createQuery(
          "SELECT ph FROM Employee e JOIN e.phones ph WHERE ph LIKE '1%'", Phone.class);
    List<Phone> resultList = query.getResultList();

    // Assertions...
}
```

## 4.外部连接

当两个或多个实体外连接时，**满足连接条件的记录以及左侧实体中的记录将被收集在结果中:**

```java
@Test
public void whenLeftKeywordIsSpecified_thenCreatesOuterJoinAndIncludesNonMatched() {
    TypedQuery<Department> query 
      = entityManager.createQuery(
          "SELECT DISTINCT d FROM Department d LEFT JOIN d.employees e", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

这里，结果将包含有关联的`Employee`的`Department`和没有关联的`Employee`。

这也称为左外部联接。JPA 不提供右连接,我们也从右实体收集不匹配的记录。但是，我们可以通过交换 FROM 子句中的实体来模拟右连接。

## 5.WHERE 子句中的联接

### 5.1.有条件的

**我们可以在 FROM 子句和** **中列出两个实体，然后在 WHERE 子句**中指定连接条件。

这很方便，尤其是当数据库级别的外键不存在时:

```java
@Test
public void whenEntitiesAreListedInFromAndMatchedInWhere_ThenCreatesJoin() {
    TypedQuery<Department> query 
      = entityManager.createQuery(
          "SELECT d FROM Employee e, Department d WHERE e.department = d", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

这里，我们连接了`Employee`和`Department`实体，但是这次在 WHERE 子句中指定了一个条件。

### 5.2.无条件(笛卡尔积)

类似地，**我们可以在 FROM 子句中列出两个实体，而不指定任何连接条件**。在这种情况下，**我们会得到一个笛卡尔积返回**。这意味着第一个实体中的每条记录都与第二个实体中的每条记录成对出现:

```java
@Test
public void whenEntitiesAreListedInFrom_ThenCreatesCartesianProduct() {
    TypedQuery<Department> query
      = entityManager.createQuery(
          "SELECT d FROM Employee e, Department d", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

正如我们所猜测的，这类查询不会执行得很好。

## 6.多重连接

到目前为止，我们已经使用了两个实体来执行连接，但是这不是一个规则。**我们还可以在单个 JPQL 查询中连接多个实体**:

```java
@Test
public void whenMultipleEntitiesAreListedWithJoin_ThenCreatesMultipleJoins() {
    TypedQuery<Phone> query
      = entityManager.createQuery(
          "SELECT ph FROM Employee e
      JOIN e.department d
      JOIN e.phones ph
      WHERE d.name IS NOT NULL", Phone.class);
    List<Phone> resultList = query.getResultList();

    // Assertions...
}
```

这里我们选择了所有`Employees`中具有类似于其他内部连接的`Department.` 的所有`Phones`，我们没有指定条件，因为 JPA 从映射元数据中提取该信息。

## 7.提取连接

现在我们来谈谈 fetch 连接。它们的**主要用途是让[为当前查询](/web/20221107131215/https://www.baeldung.com/hibernate-lazy-eager-loading)**急切地获取懒惰加载的关联。

在这里我们将急切地载入`Employee`的联想:

```java
@Test
public void whenFetchKeywordIsSpecified_ThenCreatesFetchJoin() {
    TypedQuery<Department> query 
      = entityManager.createQuery(
          "SELECT d FROM Department d JOIN FETCH d.employees", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

虽然这个查询看起来与其他查询非常相似，但是有一点不同；**那些`Employee`们急切地装上**。这意味着一旦我们在上面的测试中调用了`getResultList`,`Department`实体就会加载它们的`employees` 字段，这样就省去了我们再次访问数据库的麻烦。

**然而，我们必须意识到内存权衡**。我们可能更有效率，因为我们只执行了一个查询，但是我们也将所有的`Department`及其雇员一次加载到内存中。

我们还可以以类似于外部连接的方式执行外部提取连接，从左侧实体中收集不匹配连接条件的记录。此外，它急切地加载指定的关联:

```java
@Test
public void whenLeftAndFetchKeywordsAreSpecified_ThenCreatesOuterFetchJoin() {
    TypedQuery<Department> query 
      = entityManager.createQuery(
          "SELECT d FROM Department d LEFT JOIN FETCH d.employees", Department.class);
    List<Department> resultList = query.getResultList();

    // Assertions...
}
```

## 8.摘要

在本文中，我们讨论了 JPA 连接类型。

和往常一样，本教程和其他教程的示例可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221107131215/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query)