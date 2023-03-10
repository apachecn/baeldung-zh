# Spring 数据 JPA 中 save()和 saveAndFlush()的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-save-saveandflush>

## 1。概述

在这个快速教程中，我们将讨论 [Spring 数据 JPA](/web/20220703153110/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 中的`save()`和`saveAndFlush()`方法的区别。

尽管我们使用这两种方法将实体保存到数据库中，但还是有一些基本的区别。

## 2。应用示例

首先，让我们通过一个例子来看看如何使用`save() `和`saveAndFlush()` 方法。我们将首先创建一个实体类:

```java
@Entity
public class Employee {

    @Id
    private Long id;
    private String name;

    // constructors
    // standard getters and setters
}
```

接下来，我们将为*雇员*实体类上的 CRUD 操作创建一个 [JPA 存储库](/web/20220703153110/https://www.baeldung.com/spring-data-repositories):

```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
}
```

## 3。`save()` 法

顾名思义， [`save()`](/web/20220703153110/https://www.baeldung.com/spring-data-crud-repository-save) 方法允许我们**将一个实体保存到 DB** 。属于 Spring 数据定义的`CrudRepository`接口。让我们看看如何使用它:

```java
employeeRepository.save(new Employee(1L, "John"));
```

通常，Hibernate 将持久状态保存在内存中。将此状态与底层数据库同步的过程称为刷新。

当我们使用`save()`方法时，**与保存操作相关的数据不会被刷新到 DB，除非并且直到对`flush()`** **或`commit()`方法进行了显式调用**。

如果我们使用像 Hibernate 这样的 JPA 实现，那么这个特定的实现将管理刷新和提交操作。

这里我们必须记住的一点是，如果我们决定自己刷新数据而不提交它，那么外部[事务](/web/20220703153110/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)将看不到这些更改，除非在这个事务中进行了提交调用，或者外部事务的隔离级别是`READ_UNCOMMITTED`。

## 4。`saveAndFlush()` 法

与`save()`不同的是，`saveAndFlush()`方法**在执行过程中会立即刷新数据。**该方法属于 Spring Data JPA 的`JpaRepository`接口。我们是这样使用它的:

```java
employeeRepository.saveAndFlush(new Employee(2L, "Alice"));
```

通常，当我们的业务逻辑需要在同一个事务中的稍后时刻，但在提交之前读取保存的更改时，我们使用这种方法。

例如，想象一个场景，我们必须执行一个存储过程，该存储过程需要我们将要保存的实体的一个属性。在这种情况下，`save()`方法不起作用，因为更改与数据库不同步，存储过程不知道这些更改。`saveAndFlush()`方法非常适合这种场景。

## 5。结论

在这篇简短的文章中，我们主要关注 Spring 数据 JPA `save()`和`saveAndFlush()`方法之间的区别。

在大多数情况下，我们将使用`save()`方法。但是偶尔，对于特定的用例，我们可能也需要使用`saveAndFlush()`方法。

像往常一样，我们在这里讨论的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220703153110/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-crud)