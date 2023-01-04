# 在 JPA 中定义唯一约束

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-unique-constraints>

 ![announcement - icon](img/87f113593d335378c10992619b41d3b7.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220526054411/https://www.baeldung.com/lightrun-n-jpa)

## 1.介绍

在本教程中，我们将讨论使用 **JPA 和 Hibernate** 定义唯一约束的**。**

首先，我们将探索唯一约束以及它们与主键约束的不同之处。

然后，我们将看看 JPA 的重要注释,@ `Column(unique=true)` 和`@UniqueConstraint.` ,我们将实现它们来定义单个列和多个列上的唯一约束。

最后，我们将学习如何在被引用的表列上定义唯一约束。

## 2.唯一约束

让我们先快速回顾一下。唯一键是表中唯一标识数据库表中记录的一组单列或多列。

unique 和 primary key 约束都为一列或一组列的唯一性提供了保证。

### 2.1.它与主键约束有什么不同？

唯一约束确保一列或多列组合中的数据对于每一行都是唯一的。例如，表的主键充当隐式唯一约束。因此，**主键约束自动具有唯一约束。**

此外，每个表只能有一个 primary key 约束。但是，每个表可以有多个唯一约束。
**简单地说，除了主键映射的任何约束外，唯一约束也适用。**

我们定义的唯一约束在表创建期间用于生成适当的数据库约束，也可以在运行时用于排序`insert`、`update`或`delete `语句。

### 2.2。什么是单列和多列约束？

唯一约束可以是列约束，也可以是表约束。在表级别，我们可以定义跨多个列的唯一约束。

**JPA 允许我们使用`@Column(unique=true)`和`@UniqueConstraint`在代码中定义唯一的约束。**这些注释由模式生成过程解释，自动创建约束。

首先，我们应该强调**列级约束适用于单个列，表级约束适用于整个表。**

我们将在接下来的章节中更详细地讨论这一点。

## 3.建立一个实体

JPA 中的[实体](/web/20220526054411/https://www.baeldung.com/jpa-entities)表示存储在数据库中的一个表。实体的每个实例代表表中的一行。

让我们从创建一个域实体并将其映射到一个数据库表开始。对于这个例子，我们将创建一个*人*实体:

```
@Entity
@Table
public class Person implements Serializable {
    @Id
    @GeneratedValue
    private Long id;  
    private String name;
    private String password;
    private String email;
    private Long personNumber;
    private Boolean isActive;
    private String securityNumber;
    private String departmentCode;
    @JoinColumn(name = "addressId", referencedColumnName = "id")
    private Address address;
   //getters and setters
 } 
```

`address`字段是来自`Address`实体的引用字段:

```
@Entity
@Table
public class Address implements Serializable {
    @Id
    @GeneratedValue
    private Long id;
    private String streetAddress;
    //getters and setters
}
```

在整个教程中，我们将使用这个`Person`实体来演示我们的例子。

## 4.列约束

当我们准备好模型时，我们可以实现我们的第一个惟一约束。

让我们考虑一下保存个人信息的`Person`实体。我们有一个用于`id`列的主键。该实体还持有不包含任何重复值的`PersonNumber,` 。此外，我们不能定义主键，因为我们的表已经有了它。

在这种情况下，我们可以使用列唯一约束来确保在`PersonNumber` 字段中没有输入重复的值。JPA 允许我们使用带有`unique`属性的`@Column`注释来实现这一点。

在接下来的小节中，我们将看看`@Column`注释，然后学习如何实现它。

### 4.1.`@Column(unique=true)`

注释类型 [`Column`](https://web.archive.org/web/20220526054411/https://docs.jboss.org/hibernate/jpa/2.1/api/javax/persistence/Column.html) 用于指定持久属性或字段的映射列。

让我们来看看定义:

```
@Target(value={METHOD,FIELD})
@Retention(value=RUNTIME)
public @interface Column {
    boolean unique;
   //other elements
 } 
```

**`unique`属性指定该列是否是唯一键。这是`UniqueConstraint`注释的快捷方式，当唯一键约束只对应于一个列时非常有用。**

我们将在下一节中看到如何定义它。

### 4.2.定义列约束

**只要惟一约束只基于一个字段，我们就可以在该列上使用`@Column(unique=true)`。**

让我们在*人员编号*字段上定义一个唯一约束:

```
@Column(unique=true)
private Long personNumber;
```

当我们执行模式创建过程时，我们可以从日志中验证它:

```
[main] DEBUG org.hibernate.SQL -
    alter table Person add constraint UK_d44q5lfa9xx370jv2k7tsgsqt unique (personNumber)
```

类似地，如果我们想限制一个*人*注册一个唯一的电子邮件，我们可以在*电子邮件*字段上添加一个唯一的约束:

```
@Column(unique=true)
private String email;
```

让我们执行模式创建过程并检查约束条件:

```
[main] DEBUG org.hibernate.SQL -
    alter table Person add constraint UK_585qcyc8qh7bg1fwgm1pj4fus unique (email)
```

虽然当我们想要在单个列上放置唯一约束时这很有用，但有时我们可能想要在组合键(列的组合)上添加唯一约束。要定义一个组合唯一键，我们可以使用表约束。我们将在下一节讨论这一点。

## 5.表约束

组合唯一键是由列组合而成的唯一键。**要定义一个组合唯一键，我们可以在表上而不是在列上添加约束。JPA 使用`@UniqueConstraint`注释帮助我们实现这一点。**

### 5.1.`@UniqueConstraint`注释

注释类型 [`UniqueConstraint`](https://web.archive.org/web/20220526054411/https://docs.jboss.org/hibernate/jpa/2.1/api/javax/persistence/UniqueConstraint.html) 指定在为表生成的 DDL(数据定义语言)中包含唯一的约束。

让我们来看看定义:

```
@Target(value={})
@Retention(value=RUNTIME)
public @interface UniqueConstraint {
    String name() default "";
    String[] columnNames();
}
```

我们可以看到， `String`和`String[]`类型的**和`name`分别是可以为`UniqueConstraint`注释指定的注释元素。**

我们将在下一节通过例子更好地了解每个参数。

### 5.2.定义唯一约束

让我们考虑一下我们的`Person`实体。一个`Person`不应该有任何活动状态的重复记录。换句话说，包含`personNumber`和`isActive`的键不会有任何重复值。这里，我们需要添加跨多个列的唯一约束。

JPA 通过`@UniqueConstraint`注释帮助我们实现了这一点。我们在 *uniqueConstraints* 属性下的 [`@Table`](https://web.archive.org/web/20220526054411/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Table.html) 注释中使用它。让我们记住指定列的名称:

```
@Table(uniqueConstraints = { @UniqueConstraint(columnNames = { "personNumber", "isActive" }) })
```

一旦模式生成，我们就可以验证它:

```
[main] DEBUG org.hibernate.SQL -
    alter table Person add constraint UK5e0bv5arhh7jjhsls27bmqp4a unique (personNumber, isActive)
```

这里需要注意的一点是，如果我们不指定名称，它就是一个提供者生成的值。 **从 JPA 2.0 开始，我们可以为我们唯一的约束提供一个名字:**

```
@Table(uniqueConstraints = { @UniqueConstraint(name = "UniqueNumberAndStatus", columnNames = { "personNumber", "isActive" }) })
```

我们可以验证这一点:

```
[main] DEBUG org.hibernate.SQL -
    alter table Person add constraint UniqueNumberAndStatus unique (personNumber, isActive)
```

这里，我们在一组列上添加了唯一的约束。我们还可以添加多个唯一约束，即多组列上的唯一约束。我们将在下一节中这样做。

### 5.3.单个实体上的多个唯一约束

**一个表可以有多个唯一约束。**在上一节中，我们定义了组合键上的唯一约束:`personNumber`和`isActive`状态。在本节中，我们将在`securityNumber`和`departmentCode`的组合上添加约束。

让我们收集我们的唯一索引并立即指定它们。我们通过在大括号中重复`@UniqueConstraint` 注释来做到这一点，并用逗号分隔:

```
@Table(uniqueConstraints = {
   @UniqueConstraint(name = "UniqueNumberAndStatus", columnNames = {"personNumber", "isActive"}),
   @UniqueConstraint(name = "UniqueSecurityAndDepartment", columnNames = {"securityNumber", "departmentCode"})})
```

现在让我们看看日志，并检查约束条件:

```
[main] DEBUG org.hibernate.SQL -
    alter table Person add constraint UniqueNumberAndStatus unique (personNumber, isActive)
[main] DEBUG org.hibernate.SQL -
   alter table Person add constraint UniqueSecurityAndDepartment unique (securityNumber, departmentCode)
```

到目前为止，我们在同一个实体中的字段上定义了唯一的约束。但是，在某些情况下，我们可能引用了其他实体的字段，需要确保这些字段的唯一性。我们将在下一节讨论这一点。

## 6.被引用表列的唯一约束

当我们创建两个或更多彼此相关的表时，它们通常通过一个表中引用另一个表的主键的列来关联。该列称为“外键”例如，`Person`和`Address`实体通过`addressId`字段连接。因此，`addressId`充当被引用的表列。

我们可以在被引用的列上定义唯一约束`.` 我们将首先在单个列上实现它，然后在多个列上实现它。

### 6.1.单列约束

在我们的`Person`实体中，我们有一个引用`Address`实体的`address`字段。一个`person`应该有一个唯一的地址。

因此，让我们在`Person`的`address`字段上定义一个惟一的约束:

```
@Column(unique = true)
private Address address;
```

现在让我们快速检查一下这个约束:

```
[main] DEBUG org.hibernate.SQL -
   alter table Person add constraint UK_7xo3hsusabfaw1373oox9uqoe unique (address)
```

我们还可以在被引用的表列上定义多个列约束，这将在下一节中看到。

### 6.2.多列约束

我们可以在列的组合上指定唯一的约束。如前所述，我们可以使用表约束来做到这一点。

让我们定义对`personNumber`和`address,`的唯一约束，并将其添加到`uniqueConstraints`数组中:

```
@Entity
@Table(uniqueConstraints = 
  { //other constraints
  @UniqueConstraint(name = "UniqueNumberAndAddress", columnNames = { "personNumber", "address" })})
```

最后，让我们看看独特的约束条件:

```
[main] DEBUG org.hibernate.SQL -
    alter table Person add constraint UniqueNumberAndAddress unique (personNumber, address)
```

## 7.结论

唯一约束防止两个记录在一列或一组列中具有相同的值。

在本文中，我们学习了如何在 JPA 中定义惟一约束。首先，我们简单回顾一下独特的约束。然后我们讨论了`@Column(unique=true)` 和`@UniqueConstraint` 注释，分别定义了单个列和多个列上的惟一约束。

和往常一样，GitHub 上的[提供了这篇文章中的例子。](https://web.archive.org/web/20220526054411/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)