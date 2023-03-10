# Hibernate @LazyCollection 批注的用法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-lazycollection>

## 1.概观

管理应用程序中的 SQL 语句是我们需要注意的最重要的事情之一，因为它对性能有巨大的影响。当处理对象之间的关系时，有两种主要的提取设计模式。第一种是懒惰的方法，另一种是渴望的方法。

在本文中，我们将对它们进行概述。此外，我们将讨论 Hibernate 中的`@LazyCollection` 注释。

## 2.懒惰抓取

当我们想要推迟数据初始化直到我们需要它的时候，我们使用[惰性抓取](/web/20220525123331/https://www.baeldung.com/hibernate-lazy-eager-loading)。让我们看一个例子来更好地理解这个想法。

假设我们有一家公司在这个城市有多个分支机构。每个分公司都有自己的员工。从数据库的角度来看，这意味着我们在分行和它的雇员之间有一对多的关系。

在惰性获取方法中，一旦获取了分支对象，我们就不会获取雇员。我们只获取分支对象的数据，并推迟加载雇员列表，直到我们调用`getEmployees()`方法。此时，将执行另一个数据库查询来获取雇员。

这种方法的好处是我们减少了最初加载的数据量。原因是我们可能不需要该分支机构的员工，而且加载他们也没有意义，因为我们不打算马上使用他们。

## 3.渴望获取

当数据需要立即加载时，我们使用渴望获取。让我们以公司、分公司和员工为例来解释这个想法。一旦我们从数据库中加载了某个分支对象，我们将使用相同的数据库查询立即加载其雇员列表。

使用急切抓取的主要问题是我们加载了大量可能不需要的数据。因此，我们应该只在我们确信一旦加载了它的对象，急切获取的数据将总是被使用时才使用它。

## 4.`@LazyCollection`注解

当我们需要关注应用程序的性能时，我们使用`@LazyCollection`注释。从 Hibernate 3.0 开始，`@LazyCollection`默认开启。**使用`@LazyCollection`的主要目的是控制获取数据应该使用懒惰方法还是急切方法。**

使用 `@LazyCollection`时，`LazyCollectionOption`设置有三种配置选项:`TRUE`、`FALSE`和`EXTRA`。我们分别独立讨论一下。

### 4.1.使用`LazyCollectionOption.TRUE`

该选项为指定的字段启用了惰性抓取方法，并且是从 Hibernate 版本开始的默认选项。因此，我们不需要显式设置该选项。然而，为了更好地解释这个想法，我们将举一个设置这个选项的例子。

在这个例子中，我们有一个由`id`、`name`和与`Employee`实体的`@OneToMany`关系组成的`Branch`实体。我们可以注意到，在本例中，我们将`@LazyCollection`选项显式设置为`true`:

```java
@Entity
public class Branch {

    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "branch")
    @LazyCollection(LazyCollectionOption.TRUE)
    private List<Employee> employees;

    // getters and setters
}
```

现在，让我们看看由`id`、`name`、`address`以及与`Branch`实体的`@ManyToOne`关系组成的`Employee`实体:

```java
@Entity
public class Employee {

    @Id
    private Long id;

    private String name;

    private String address;

    @ManyToOne
    @JoinColumn(name = "BRANCH_ID") 
    private Branch branch; 

    // getters and setters 
}
```

在上面的例子中，**当我们得到一个分支对象时，我们不会立即加载雇员列表**。相反，这个操作将被推迟，直到我们调用`getEmployees()`方法。

### 4.2.使用`LazyCollectionOption.FALSE`

当我们将这个选项设置为`FALSE`时，我们启用了急切获取方法。在这种情况下，我们需要显式地指定这个选项，因为我们将覆盖 Hibernate 的默认值。让我们看另一个例子。

在这种情况下，我们有一个`Branch`实体，它包含`id`、`name`，以及一个与`Employee`实体的`@OneToMany`关系。注意，我们将`@LazyCollection`的选项设置为`FALSE`:

```java
@Entity
public class Branch {

    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "branch")
    @LazyCollection(LazyCollectionOption.FALSE)
    private List<Employee> employees;

    // getters and setters
}
```

在上面的例子中，**当我们得到一个分支对象时，我们将立即加载雇员列表的分支**。

### 4.3.使用`LazyCollectionOption.EXTRA`

有时，我们只关心集合的属性，并不马上需要其中的对象。

例如，回到`Branch`和`Employee`的例子，我们可能只需要分支机构中的雇员数量，而不关心实际雇员的实体。在这种情况下，我们考虑使用`EXTRA` 选项。让我们更新我们的例子来处理这种情况。

与之前的情况类似，`Branch`实体与`Employee`实体有一个`id`、`name`和`@OneToMany`关系。但是，我们将`@LazyCollection`的选项设置为`EXTRA`:

```java
@Entity
public class Branch {

    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "branch")
    @LazyCollection(LazyCollectionOption.EXTRA)
    @OrderColumn(name = "order_id")
    private List<Employee> employees;

    // getters and setters

    public Branch addEmployee(Employee employee) {
        employees.add(employee);
        employee.setBranch(this);
        return this;
    }
}
```

**我们注意到在这个例子中我们使用了`@OrderColumn` 注释。原因是`EXTRA` 选项仅用于索引列表集合。这意味着如果我们没有用`@OrderColumn,`注释这个字段，那么`EXTRA` 选项将会给我们带来与 lazy 相同的行为，并且在第一次访问时将会获取集合。**

此外，我们还定义了`addEmployee()` 方法，因为我们需要从两端同步`Branch`和`Employee`。如果我们添加一个新的`Employee`并为他设置一个分支，我们需要更新`Branch`实体中的员工列表。

现在，当持久化一个有三个关联雇员的`Branch`实体时，我们需要将代码写成:

```java
entityManager.persist(
  new Branch().setId(1L).setName("Branch-1")

    .addEmployee(
      new Employee()
        .setId(1L)
        .setName("Employee-1")
        .setAddress("Employee-1 address"))

    .addEmployee(
      new Employee()
        .setId(2L)
        .setName("Employee-2")
        .setAddress("Employee-2 address"))

    .addEmployee(
      new Employee()
        .setId(3L)
        .setName("Employee-3")
        .setAddress("Employee-3 address"))
); 
```

如果我们看一下执行的查询，我们会注意到 Hibernate 将首先为 Branch-1 插入一个新的`Branch`。然后，它将插入雇员 1、雇员 2、雇员 3。

**我们可以看出这是一种自然行为。然而，`EXTRA` 选项中的不良行为是，在刷新上述查询后，它将执行另外三个查询——每添加一个`Employee`就执行一个:**

```java
UPDATE EMPLOYEES
SET
    order_id = 0
WHERE
    id = 1

UPDATE EMPLOYEES
SET
    order_id = 1
WHERE
    id = 2

UPDATE EMPLOYEES
SET
    order_id = 2
WHERE
    id = 3
```

**执行`UPDATE`语句来设置`List`入口索引。这是一个被称为 [`N` +1 查询问题](/web/20220525123331/https://www.baeldung.com/hibernate-common-performance-problems-in-logs)的例子，这意味着我们执行`N`额外的 SQL 语句来更新我们创建的相同数据。**

正如我们从例子中注意到的，当使用`EXTRA`选项时，我们可能会遇到`N` +1 查询问题。

另一方面，使用这个选项的好处是当我们需要获得每个分支机构的雇员列表的大小时:

```java
int employeesCount = branch.getEmployees().size();
```

当我们调用这条语句时，它只会执行这条 SQL 语句:

```java
SELECT
    COUNT(ID)
FROM
    EMPLOYEES
WHERE
    BRANCH_ID = :ID
```

**正如我们所看到的，我们不需要在内存中存储雇员列表来获取它的大小。然而，我们建议避免使用`EXTRA`选项，因为它会执行额外的查询。**

这里值得注意的是，其他数据访问技术也可能遇到`N` +1 查询问题，因为它不仅限于 JPA 和 Hibernate。

## 5.结论

在本文中，我们讨论了使用 Hibernate 从数据库获取对象属性的不同方法。

首先，我们用一个例子讨论了惰性抓取。然后，我们更新了这个例子来使用渴望获取，并讨论了不同之处。

最后，我们展示了另一种获取数据的方法，并解释了它的优缺点。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220525123331/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-annotations)