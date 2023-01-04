# JPA、Hibernate 和 EclipseLink 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-hibernate-difference>

## 1.介绍

在本教程中，我们将讨论 Hibernate 和 Java Persistence API(JPA)——重点是它们之间的区别。

我们将**从探索 JPA 是什么，它是如何使用的，以及它背后的核心概念**开始。

然后，我们将看一看 Hibernate 和 EclipseLink 是如何适应这幅画面的。

## 2.对象关系映射

在我们深入 JPA 之前，理解对象关系映射的概念很重要——也称为 ORM。

对象关系映射就是将任何 Java 对象直接保存到数据库表的过程。通常，被持久化的对象的名称成为表的名称，并且该对象中的每个字段成为一列。设置好表格后，每一行都对应于应用程序中的一条记录。

## 3.JPA 简介

Java 持久性 API(JPA)是一种规范，它定义了 Java 应用程序中关系数据的管理。API 映射出一组概念，这些概念定义了应用程序中的哪些对象应该持久化，以及应该如何持久化它们。

这里需要注意的是， **JPA 只是一个规范，它需要一个实现来工作**——但是更多的[在后面的](#implementations)中。

现在，让我们讨论一些实现必须涵盖的核心 JPA 概念。

### 3.1.实体

**`javax.persistence.Entity`类定义了哪些对象应该保存到数据库**中。对于每个持久化的实体，JPA 会在所选的数据库中创建一个新表。

此外，所有选择的实体都应该定义一个由`@Id `注释表示的主键。与`@GeneratedValue `注释一起，我们定义了当记录被保存到数据库时，应该自动生成主键。

让我们看一个由 JPA 描述的实体的简单例子。

```
@Entity
public class Car {

    @GeneratedValue
    @Id
    public long id;

    // getters and setters

} 
```

记住，这目前对应用程序没有影响——JPA 不提供任何实现代码。

### 3.2.场持久性

JPA 的另一个核心概念是字段持久性。当 Java 中的一个对象被定义为一个实体时，其中的所有字段都被自动保存为实体表中的不同列。

如果持久化对象中有一个字段我们`don't `想要持久化到数据库中，我们可以用`@Transient `注释声明这个字段是瞬态的。

### 3.3.关系

接下来， **JPA 指定了我们应该如何管理应用程序中不同数据库表**之间的关系。正如我们所见，JPA 用注释来处理这个问题。我们需要记住四个关系注释:

1.  `@OneToOne`
2.  `@OneToMany`
3.  `@ManyToOne`
4.  `@ManyToMany`

让我们来看看这是如何工作的:

```
@Entity
public class SteeringWheel {

    @OneToOne
    private Car car

    // getters and setters
}
```

在上面的例子中，`SteeringWheel `类描述了与前面的`Car `类的一对一关系。

### 3.4.实体经理

最后，`[javax.persistence.EntityManager](https://web.archive.org/web/20220926200213/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html) class `指定了对数据库的操作。 **`EntityManager`包含通用的创建、读取、更新和删除(CRUD)操作**，这些操作被[持久化](https://web.archive.org/web/20220926200213/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#persist-java.lang.Object-)到数据库。

## 4.JPA 实现

JPA 规范定义了我们应该如何持久化以及持久化什么，现在我们需要选择一个实现提供者来提供必要的代码。如果没有这样的提供者，我们将需要实现所有相关的类来符合 JPA，这是一项`lot`工作！

有大量的提供商可供选择，每个提供商都有自己的优缺点。**在决定使用哪一款**时，我们**应该考虑以下几点**:

1.  项目成熟度—`**how long has the provider been around**, and how well documented is it?`
2.  子项目—`**does the provider have any useful subprojects** for our new application?`
3.  社区支持—`is there **anyone to help us out when we end up with a critical bug**?`
4.  基准测试—`**how performant** is the implementation?`

虽然我们不会深入探讨不同 JPA 提供商的基准测试，但 [JPA 性能基准测试(JPAB)](https://web.archive.org/web/20220926200213/http://www.jpab.org/) 包含了对此的宝贵见解。

说完这些，让我们简单地看一下 JPA 的一些顶级提供商。

## 5.冬眠

在其核心， **Hibernate 是一个对象关系映射工具，它提供了 JPA** 的实现。 **Hibernate 是最成熟的 JPA 实现之一**，有一个巨大的社区支持这个项目。

它实现了我们在本文前面看到的所有`javax.persistence` 类，并提供了 JPA 之外的功能——[Hibernate 工具](https://web.archive.org/web/20220926200213/http://hibernate.org/tools/)、[验证](https://web.archive.org/web/20220926200213/http://hibernate.org/validator/)和[搜索](https://web.archive.org/web/20220926200213/http://hibernate.org/search/)。尽管这些特定于 Hibernate 的 API 可能很有用，但在只需要基本 JPA 功能的应用程序中并不需要它们。

让我们快速看一下 Hibernate 通过`@Entity`注释提供了什么。

在履行 JPA 合同的同时， **`@org.hibernate.annotations.Entity `增加了超出 JPA 规范的额外元数据。**这样做允许微调实体持久性。例如，让我们看看 [Hibernate 提供的一些注释，它们扩展了`@Entity` :](https://web.archive.org/web/20220926200213/https://docs.jboss.org/hibernate/annotations/3.5/reference/en/html/entity.html#entity-hibspec-entity) 的功能

1.  `@Table `–`allows us to **specify the name of the table** created for the entity`
2.  `@BatchSize`–`specifies the **batch size when retrieving entitie**s from the table`

同样值得注意的是 JPA 没有指定的一些额外特性，这些特性可能在较大的应用程序中很有用:

1.  带有`@SQLInsert, @SQLUpate and @SQLDelete `注释的可定制 CRUD 语句
2.  支持软删除
3.  带有`@Immutable `注释的不可变实体

要更深入地了解 Hibernate 和 Java 持久性——请阅读我们的 [Spring 持久性教程系列](/web/20220926200213/https://www.baeldung.com/persistence-with-spring-series)。

## 6.EclipseLink

由 Eclipse 基金会**构建的 EclipseLink 提供了一个开源的 JPA 实现**。此外，EclipseLink **支持许多其他持久性标准，比如 Java Architecture for XML Binding(JAXB)**。

简单地说，JAXB 不是将对象持久化到数据库行，而是将其映射到 XML 表示。

接下来，通过比较相同的`@Entity`注释实现，我们看到 EclipseLink 再次提供了[不同的扩展](https://web.archive.org/web/20220926200213/https://www.eclipse.org/eclipselink/documentation/2.7/jpa/extensions/annotations_ref.htm#CACGCEIJ)。虽然@ `BatchSize `没有注释，但正如我们前面看到的， **EclipseLink 提供了 Hibernate 没有的其他选项。**

例如:

1.  `@ReadOnly – specifies the entity to be persisted is read-only`
2.  @ `Struct ` `– defines the class to map to a database ‘struct' type`

要阅读更多关于 EclipseLink 的内容，请阅读我们关于 EclipseLink with Spring 的[指南。](/web/20220926200213/https://www.baeldung.com/spring-eclipselink)

## 7.结论

在本文中，我们已经了解了 Java 持久性 API，或 JPA。

最后，我们探索了**与 Hibernate 和 EclipseLink 的不同之处。**