# 使用 Hibernate 进行动态映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-dynamic-mapping>

## 1。简介

在本文中，我们将通过`@Formula`、`@Where`、`@Filter`和`@Any`注释探索 Hibernate 的一些动态映射功能。

注意，虽然 Hibernate 实现了 JPA 规范，但是这里描述的注释只在 Hibernate 中可用，不能直接移植到其他 JPA 实现中。

## 2。项目设置

为了演示这些特性，我们只需要 hibernate-core 库和一个后台 H2 数据库:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.12.Final</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.194</version>
</dependency>
```

对于当前版本的`hibernate-core`库，请前往 [Maven Central](https://web.archive.org/web/20220630141258/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22) 。

## 3。计算列用`@Formula`

假设我们想根据一些其他属性计算一个实体字段值。一种方法是在我们的 Java 实体中定义一个计算只读字段:

```java
@Entity
public class Employee implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private long grossIncome;

    private int taxInPercents;

    public long getTaxJavaWay() {
        return grossIncome * taxInPercents / 100;
    }

}
```

明显的缺点是，每次 getter 访问这个虚拟字段时，我们都必须重新计算。

从数据库中获取已经计算出的值会容易得多。这可以通过`@Formula`注释来完成:

```java
@Entity
public class Employee implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private long grossIncome;

    private int taxInPercents;

    @Formula("grossIncome * taxInPercents / 100")
    private long tax;

}
```

**使用`@Formula`，我们可以使用子查询，调用本地数据库函数和存储过程，并且基本上可以做任何不会破坏该字段的 SQL select 子句语法的事情。**

Hibernate 足够聪明，可以解析我们提供的 SQL 并插入正确的表和字段别名。需要注意的是，由于注释的值是原始 SQL，这可能会使我们的映射依赖于数据库。

另外，请记住**该值是在从数据库**中提取实体时计算的。因此，当我们持久化或更新实体时，直到该实体从上下文中被逐出并被再次加载，该值才会被重新计算:

```java
Employee employee = new Employee(10_000L, 25);
session.save(employee);

session.flush();
session.clear();

employee = session.get(Employee.class, employee.getId());
assertThat(employee.getTax()).isEqualTo(2_500L);
```

## 4。用`@Where` 过滤实体

假设每当我们请求某个实体时，我们想为查询提供一个附加条件。

例如，我们需要实现“软删除”。这意味着实体永远不会从数据库中删除，而只是用一个`boolean`字段标记为已删除。

我们必须非常注意应用程序中所有现有的和未来的查询。我们必须为每个查询提供这个附加条件。幸运的是，Hibernate 在一个地方提供了实现这一点的方法:

```java
@Entity
@Where(clause = "deleted = false")
public class Employee implements Serializable {

    // ...
}
```

方法上的`@Where`注释包含一个 SQL 子句，该子句将被添加到该实体的任何查询或子查询中:

```java
employee.setDeleted(true);

session.flush();
session.clear();

employee = session.find(Employee.class, employee.getId());
assertThat(employee).isNull();
```

与`@Formula`注释的情况一样，**因为我们正在处理原始 SQL，所以`@Where`条件不会被重新评估，直到我们将实体刷新到数据库并将其从上下文**中逐出。

在此之前，实体将留在上下文中，并且可以通过`id`进行查询和查找。

`@Where`注释也可以用于集合字段。假设我们有一个可删除电话的列表:

```java
@Entity
public class Phone implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private boolean deleted;

    private String number;

}
```

然后，从`Employee`端，我们可以映射一个可删除的集合`phones`，如下所示:

```java
public class Employee implements Serializable {

    // ...

    @OneToMany
    @JoinColumn(name = "employee_id")
    @Where(clause = "deleted = false")
    private Set<Phone> phones = new HashSet<>(0);

}
```

不同的是，`Employee.phones`集合总是被过滤，但是我们仍然可以通过直接查询获得所有的电话，包括被删除的电话:

```java
employee.getPhones().iterator().next().setDeleted(true);
session.flush();
session.clear();

employee = session.find(Employee.class, employee.getId());
assertThat(employee.getPhones()).hasSize(1);

List<Phone> fullPhoneList 
  = session.createQuery("from Phone").getResultList();
assertThat(fullPhoneList).hasSize(2);
```

## 5。用`@Filter` 进行参数化滤波

`@Where`注释的问题在于，它只允许我们指定一个没有参数的静态查询，并且不能根据需求禁用或启用。

**`@Filter`注释的工作方式与`@Where`相同，但是它也可以在会话级别上启用或禁用，也可以参数化。**

### 5.1。`@Filter`定义

为了演示`@Filter`如何工作，让我们首先将下面的过滤器定义添加到`Employee`实体中:

```java
@FilterDef(
    name = "incomeLevelFilter", 
    parameters = @ParamDef(name = "incomeLimit", type = "int")
)
@Filter(
    name = "incomeLevelFilter", 
    condition = "grossIncome > :incomeLimit"
)
public class Employee implements Serializable {
```

`@FilterDef`注释定义了过滤器名称和一组将参与查询的参数。参数的类型是 Hibernate 类型之一的名称([类型](https://web.archive.org/web/20220630141258/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/type/Type.html)、[用户类型](https://web.archive.org/web/20220630141258/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/usertype/UserType.html)或[复合用户类型](https://web.archive.org/web/20220630141258/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/usertype/CompositeUserType.html))，在我们的例子中是一个`int`。

`The @FilterDef`注释可以放在类型上，也可以放在包装上。注意，它没有指定过滤条件本身(尽管我们可以指定`defaultCondition`参数)。

这意味着我们可以在一个地方定义过滤器(它的名称和参数集),然后在多个不同的地方定义过滤器的条件。

这可以通过`@Filter`注释来完成。在我们的例子中，为了简单起见，我们把它放在同一个类中。该条件的语法是一个原始 SQL，参数名以冒号开头。

### 5.2。访问过滤的实体

`@Filter`与`@Where`的另一个区别是`@Filter`默认不启用。我们必须在会话级别手动启用它，并为它提供参数值:

```java
session.enableFilter("incomeLevelFilter")
  .setParameter("incomeLimit", 11_000);
```

现在假设数据库中有以下三名员工:

```java
session.save(new Employee(10_000, 25));
session.save(new Employee(12_000, 25));
session.save(new Employee(15_000, 25));
```

然后启用过滤器，如上所示，通过查询只能看到其中的两个:

```java
List<Employee> employees = session.createQuery("from Employee")
  .getResultList();
assertThat(employees).hasSize(2);
```

请注意，启用的过滤器及其参数值仅在当前会话中应用。在没有启用过滤器的新会话中，我们将看到所有三名员工:

```java
session = HibernateUtil.getSessionFactory().openSession();
employees = session.createQuery("from Employee").getResultList();
assertThat(employees).hasSize(3);
```

此外，当按 id 直接提取实体时，不应用过滤器:

```java
Employee employee = session.get(Employee.class, 1);
assertThat(employee.getGrossIncome()).isEqualTo(10_000);
```

### 5.3。`@Filter`和二级缓存

如果我们有一个高负载的应用程序，那么我们肯定希望启用 Hibernate 二级缓存，这可以带来巨大的性能优势。我们应该记住，**`@Filter`注释不能很好地处理缓存。**

**二级缓存只保留完整的未过滤集合**。如果不是这样，那么我们可以在启用过滤器的一个会话中读取一个集合，然后在禁用过滤器的另一个会话中获得相同的缓存过滤集合。

**这就是为什么`@Filter`注释基本上禁用了实体的缓存。**

## 6。用`@Any` 映射任何实体引用

有时我们希望将一个引用映射到多个实体类型中的任何一个，即使它们不是基于单个的`@MappedSuperclass`。它们甚至可以映射到不同的不相关的表。我们可以通过`@Any`注释来实现这一点。

在我们的示例**中，我们需要为我们的持久单元**中的每个实体附加一些描述，即`Employee`和`Phone`。仅仅为了做到这一点而从一个抽象超类继承所有实体是不合理的。

### 6.1。与`@Any`的映射关系

下面是我们如何定义对任何实现`Serializable`的实体的引用(即，对任何实体的引用):

```java
@Entity
public class EntityDescription implements Serializable {

    private String description;

    @Any(
        metaDef = "EntityDescriptionMetaDef",
        metaColumn = @Column(name = "entity_type"))
    @JoinColumn(name = "entity_id")
    private Serializable entity;

}
```

`metaDef`属性是定义的名称，`metaColumn`是用于区分实体类型的列的名称(与单个表层次结构映射中的 discriminator 列类似)。

我们还指定了将引用实体的`id`的列。值得注意的是**这个列不会是外键**，因为它可以引用我们想要的任何表。

`entity_id`列通常也不能是唯一的，因为不同的表可能有重复的标识符。

**然而，`entity_type` / `entity_id`对应该是唯一的，因为它唯一地描述了我们所引用的实体。**

### 6.2。用`@AnyMetaDef` 定义`@Any`映射

现在，Hibernate 不知道如何区分不同的实体类型，因为我们没有指定`entity_type`列可以包含什么。

为了实现这一点，我们需要添加带有`@AnyMetaDef`注释的映射的元定义。放置它的最佳位置是包级别，这样我们可以在其他映射中重用它。

下面是带有`@AnyMetaDef`注释的`package-info.java`文件的样子:

```java
@AnyMetaDef(
    name = "EntityDescriptionMetaDef", 
    metaType = "string", 
    idType = "int",
    metaValues = {
        @MetaValue(value = "Employee", targetEntity = Employee.class),
        @MetaValue(value = "Phone", targetEntity = Phone.class)
    }
)
package com.baeldung.hibernate.pojo;
```

这里我们已经指定了`entity_type`列的类型(`string`)、`entity_id`列的类型(`int`)、`entity_type`列中可接受的值(`“Employee”`和`“Phone”`)以及相应的实体类型。

现在，假设我们有一名员工有两部手机，描述如下:

```java
Employee employee = new Employee();
Phone phone1 = new Phone("555-45-67");
Phone phone2 = new Phone("555-89-01");
employee.getPhones().add(phone1);
employee.getPhones().add(phone2);
```

现在，我们可以向所有三个实体添加描述性元数据，即使它们具有不同的不相关类型:

```java
EntityDescription employeeDescription = new EntityDescription(
  "Send to conference next year", employee);
EntityDescription phone1Description = new EntityDescription(
  "Home phone (do not call after 10PM)", phone1);
EntityDescription phone2Description = new EntityDescription(
  "Work phone", phone1);
```

## 7。结论

在本文中，我们研究了 Hibernate 的一些注释，这些注释允许使用原始 SQL 对实体映射进行微调。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630141258/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)