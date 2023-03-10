# Hibernate 命名查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-named-query>

## 1。概述

将 HQL 和 SQL 分散在数据访问对象中的一个主要缺点是，这会使代码不可读。因此，将所有 HQL 和 SQL 放在一个地方并在实际的数据访问代码中只使用它们的引用可能是有意义的。幸运的是，Hibernate 允许我们通过命名查询做到这一点。

命名查询是静态定义的查询，具有预定义的不可更改的查询字符串。它们在创建会话工厂时得到验证，从而使应用程序在出错时快速失败。

**在本文中，我们将看到如何使用`@NamedQuery`和`@NamedNativeQuery`注释来定义和使用 Hibernate 命名查询。**

## 2。实体

让我们首先来看看我们将在本文中使用的实体:

```java
@Entity
public class DeptEmployee {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String employeeNumber;

    private String designation;

    private String name;

    @ManyToOne
    private Department department;

    // getters and setters
}
```

在我们的示例中，我们将根据员工编号检索员工。

## 3。命名查询

为了将它定义为一个命名查询，我们将使用`org.hibernate.annotations.NamedQuery` 注释。它用 Hibernate 特性扩展了 javax `.persistence.NamedQuery`。

我们将其定义为`DeptEmployee`类的注释:

```java
@org.hibernate.annotations.NamedQuery(name = "DeptEmployee_findByEmployeeNumber", 
  query = "from DeptEmployee where employeeNumber = :employeeNo") 
```

**值得注意的是，每个`@NamedQuery `注释都被附加到一个实体类或映射的超类。但是，** **由于命名查询的范围是整个持久化单元，我们应该仔细选择查询名以避免冲突。**我们通过使用实体名称作为前缀实现了这一点。

如果一个实体有多个命名查询，我们将使用`@NamedQueries`注释对它们进行分组:

```java
@org.hibernate.annotations.NamedQueries({
    @org.hibernate.annotations.NamedQuery(name = "DeptEmployee_FindByEmployeeNumber", 
      query = "from DeptEmployee where employeeNumber = :employeeNo"),
    @org.hibernate.annotations.NamedQuery(name = "DeptEmployee_FindAllByDesgination", 
      query = "from DeptEmployee where designation = :designation"),
    @org.hibernate.annotations.NamedQuery(name = "DeptEmployee_UpdateEmployeeDepartment", 
      query = "Update DeptEmployee set department = :newDepartment where employeeNumber = :employeeNo"),
...
})
```

注意，HQL 查询可以是 DML 风格的操作。所以，它不需要仅仅是一个`select` 语句。例如，我们可以有一个更新查询，如上面的`DeptEmployee_UpdateEmployeeDesignation` 所示。

### 3.1。配置查询功能

我们可以用`@NamedQuery `注释设置各种查询特性。让我们看一个例子:

```java
@org.hibernate.annotations.NamedQuery(
  name = "DeptEmployee_FindAllByDepartment", 
  query = "from DeptEmployee where department = :department",
  timeout = 1,
  fetchSize = 10
)
```

这里，我们已经配置了超时间隔和获取大小。除了这两个，我们还可以设置如下功能:

*   `cacheable`–查询(结果)是否可缓存
*   `cacheMode`–本次查询使用的缓存模式；这可以是`GET, IGNORE, NORMAL, PUT,`或`REFRESH`中的一个
*   `cacheRegion`–如果查询结果可缓存，命名要使用的查询缓存区域
*   `comment`–添加到生成的 SQL 查询中的注释；面向数据库管理员
*   `flushMode`–本次查询的刷新模式，`ALWAYS, AUTO, COMMIT, MANUAL,`或`PERSISTENCE_CONTEXT`之一

### 3.2。使用命名查询

现在我们已经定义了命名查询，让我们用它来检索一个雇员:

```java
Query<DeptEmployee> query = session.createNamedQuery("DeptEmployee_FindByEmployeeNumber", 
  DeptEmployee.class);
query.setParameter("employeeNo", "001");
DeptEmployee result = query.getSingleResult(); 
```

这里，我们使用了`createNamedQuery`方法。它接受查询的名称并返回一个`org.hibernate.query.Query `对象。

## 4。命名本地查询

除了 HQL 查询，我们还可以将原生 SQL 定义为命名查询。为此，我们可以使用`@NamedNativeQuery`注释。虽然它与`@NamedQuery`相似，但它需要更多的配置。

让我们用一个例子来研究这个注释:

```java
@org.hibernate.annotations.NamedNativeQueries(
    @org.hibernate.annotations.NamedNativeQuery(name = "DeptEmployee_GetEmployeeByName", 
      query = "select * from deptemployee emp where name=:name",
      resultClass = DeptEmployee.class)
)
```

**由于这是一个本地查询，我们必须告诉 Hibernate 将结果映射到哪个实体类。因此，我们使用了`resultClass `属性来做这件事。**

**映射结果的另一种方式是使用`resultSetMapping `属性。在这里，我们可以指定一个预定义的`SQLResultSetMapping`的名称。**

注意，我们只能使用`resultClass`和`resultSetMapping`中的一个。

### 4.1。使用命名的本地查询

为了使用命名的本地查询，我们可以使用`Session.createNamedQuery()`:

```java
Query<DeptEmployee> query = session.createNamedQuery("DeptEmployee_FindByEmployeeName", DeptEmployee.class);
query.setParameter("name", "John Wayne");
DeptEmployee result = query.getSingleResult();
```

还是那个`Session.getNamedNativeQuery()`:

```java
NativeQuery query = session.getNamedNativeQuery("DeptEmployee_FindByEmployeeName");
query.setParameter("name", "John Wayne");
DeptEmployee result = (DeptEmployee) query.getSingleResult();
```

这两种方法之间的唯一区别是返回类型。第二种方法返回一个`NativeQuery, `，它是`Query`的子类。

## 5。存储过程和函数

我们也可以使用`@NamedNativeQuery`注释来定义对存储过程和函数的调用:

```java
@org.hibernate.annotations.NamedNativeQuery(
  name = "DeptEmployee_UpdateEmployeeDesignation", 
  query = "call UPDATE_EMPLOYEE_DESIGNATION(:employeeNumber, :newDesignation)", 
  resultClass = DeptEmployee.class)
```

注意，虽然这是一个更新查询，但是我们已经使用了`resultClass `属性。这是因为 Hibernate 不支持纯本地标量查询。解决这个问题的方法是设置一个`resultClass `或者一个`resultSetMapping.`

## 6。结论

在本文中，我们看到了如何定义和使用命名 HQL 和本地查询。

GitHub 上的[提供了源代码。](https://web.archive.org/web/20220703153709/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-queries)