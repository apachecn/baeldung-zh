# @JoinColumn 和 mappedBy 之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-joincolumn-vs-mappedby>

## 1.概观

JPA 关系可以是单向的，也可以是双向的。这仅仅意味着我们可以将它们建模为关联实体之一或两者的属性。

定义实体之间关系的方向对数据库映射没有影响。它仅仅定义了我们在领域模型中使用这种关系的方向。

对于双向关系，我们通常定义为

*   拥有方
*   反向或参考侧

[`@JoinColumn`](/web/20220926180736/https://www.baeldung.com/jpa-join-column) 注释帮助我们指定用于加入实体关联或元素集合的列。另一方面，`mappedBy`属性用于定义关系的引用方(非拥有方)。

在这个快速教程中，我们将看看 JPA 中的**`@JoinColumn`和`mappedBy`** **的区别。我们还将介绍如何在一对多关联中使用它们。**

## 2.初始设置

按照本教程，假设我们有两个实体:`Employee`和`Email`。

显然，一个员工可以有多个电子邮件地址。但是，给定的电子邮件地址可以完全属于一个员工。

这意味着它们共享一对多关联:

[![Employee - Email](img/17d14eeb63ac4775cad1ceafc2695a1b.png)](/web/20220926180736/https://www.baeldung.com/wp-content/uploads/2018/11/12345789.png)

同样在我们的 RDBMS 模型中，我们将在我们的`Email`实体中有一个外键`employee_id`来引用`Employee`的`id` 属性。

## 3.`@JoinColumn`注释

在一对多/多对一关系中，**拥有方通常定义在关系的`many`方。**通常是拥有外键的一方。

`@JoinColumn`注释定义了拥有方的实际物理映射:

```java
@Entity
public class Email {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "employee_id")
    private Employee employee;

    // ...

}
```

这仅仅意味着我们的`Email`实体将有一个名为`employee_id`的外键列，引用我们的`Employee` 实体的主属性`id`。

## 4.`mappedBy`属性

一旦我们定义了关系的拥有方，Hibernate 就已经拥有了在数据库中映射该关系所需的所有信息。

为了使这种关联是双向的，我们要做的就是定义引用方。反向或引用端只是映射到拥有端。

我们可以很容易地使用`@OneToMany`注释的`mappedBy`属性来做到这一点。

因此，让我们定义我们的`Employee`实体:

```java
@Entity
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @OneToMany(fetch = FetchType.LAZY, mappedBy = "employee")
    private List<Email> emails;

    // ...
}
```

这里，**`mappedBy`的值是拥有方的关联映射属性的名称。**这样，我们现在已经在我们的`Employee`和`Email`实体之间建立了一个双向关联。

## 5.结论

在本文中，我们研究了`@JoinColumn`和`mappedBy`之间的区别，以及如何在一对多的双向关系中使用它们。

`@JoinColumn`注释定义了拥有方的实际物理映射。另一方面，引用侧是使用`@OneToMany`注释的`mappedBy`属性定义的。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220926180736/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-annotations)