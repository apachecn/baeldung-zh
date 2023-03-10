# 将 Hibernate 查询映射到自定义类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-query-to-custom-class>

## 1。概述

当我们使用 Hibernate 从数据库中检索数据时，默认情况下，它使用检索到的数据为请求的对象构建整个对象图。但有时我们可能只想检索部分数据，最好是平面结构。

在这个快速教程中，我们将看到如何在 Hibernate 中使用一个自定义类来实现这一点。

## 2。实体

首先，让我们看看我们将用来检索数据的实体:

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

    // constructor, getters and setters 
} 

@Entity
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String name;

    @OneToMany(mappedBy="department")
    private List<DeptEmployee> employees;

    public Department(String name) {
        this.name = name;
    }

    // getters and setters 
}
```

这里，我们有两个实体——`DeptEmployee`和`Department`。为了简单起见，让我们假设一个`DeptEmployee`只能属于一个`Department.`

但是，一个`Department`可以有多个`DeptEmployees`。

## 3。自定义查询结果类

假设我们想要打印一份所有员工的列表，其中只包含他们的姓名和所在部门的名称。

通常，我们会使用如下查询检索这些数据:

```java
Query<DeptEmployee> query = session.createQuery("from com.baeldung.hibernate.entities.DeptEmployee");
List<DeptEmployee> deptEmployees = query.list();
```

这将检索所有雇员、他们的所有属性、关联的部门及其所有属性。

但是，在这个特殊的例子中，这可能有点贵，因为我们只需要雇员的名字和部门的名字。

检索我们需要的信息的一种方法是在 select 子句中指定属性。

但是，当我们这样做时，Hibernate 返回一个数组列表，而不是一个`Objects:`列表

```java
Query query = session.createQuery("select m.name, m.department.name from com.baeldung.hibernate.entities.DeptEmployee m");
List managers = query.list();
Object[] manager = (Object[]) managers.get(0);
assertEquals("John Smith", manager[0]);
assertEquals("Sales", manager[1]);
```

正如我们所看到的，返回的数据处理起来有点麻烦。但是，幸运的是，我们可以让 Hibernate 将这些数据填充到一个类中。

让我们看一下`Result`类，我们将使用它将检索到的数据填充到:

```java
public class Result {
    private String employeeName;

    private String departmentName;

    public Result(String employeeName, String departmentName) {
        this.employeeName = employeeName;
        this.departmentName = departmentName;
    }

    public Result() {
    }

    // getters and setters 
}
```

请注意，该类不是一个实体，而只是一个 POJO。然而，我们也可以使用一个实体，只要它有一个构造函数，这个构造函数将我们想要填充的所有属性作为参数。

我们将在下一节看到为什么构造函数很重要。

## 4。在 HQL 使用建造师

现在，让我们看看使用这个类的 HQL:

```java
Query<Result> query = session.createQuery("select new com.baeldung.hibernate.pojo.Result(m.name, m.department.name)" 
  + " from com.baeldung.hibernate.entities.DeptEmployee m");
List<Result> results = query.list();
Result result = results.get(0);
assertEquals("John Smith", result.getEmployeeName());
assertEquals("Sales", result.getDepartmentName());
```

这里，我们使用我们在`Result `类中定义的构造函数以及我们想要检索的属性。这将返回一个包含从列中填充的数据的`Result`对象列表。

正如我们所看到的，返回的列表比使用对象数组列表更容易处理。

需要注意的是，我们必须在查询中使用完全限定的类名。

## 5。使用 ResultTransformer

在 HQL 查询中使用构造函数的另一种方法是使用`ResultTransformer:`

```java
Query query = session.createQuery("select m.name as employeeName, m.department.name as departmentName" 
  + " from com.baeldung.hibernate.entities.DeptEmployee m");
query.setResultTransformer(Transformers.aliasToBean(Result.class));
List<Result> results = query.list();
Result result = results.get(0);
assertEquals("John Smith", result.getEmployeeName());
assertEquals("Sales", result.getDepartmentName());
```

我们使用`Transformers.` `aliasToBean() `方法使用检索到的数据来填充`Result `对象。

因此，我们必须确保 select 语句中的列名或它们的别名与`Result `类的属性相匹配。

注意[`Query.setResultTransformer(`result transformer](https://web.archive.org/web/20220523151100/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/Query.html#setResultTransformer-org.hibernate.transform.ResultTransformer-)`) `从 Hibernate 5.2 开始就不推荐使用了。

## 6。结论

在本文中，我们看到了如何使用自定义类以易读的形式检索数据。

本文附带的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220523151100/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)