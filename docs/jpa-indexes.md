# 在 JPA 中定义索引

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-indexes>

## 1.介绍

在本教程中，我们将讨论使用 JPA 的`@Index`注释定义索引的**。通过例子，我们将学习如何使用 JPA 和 Hibernate 定义我们的第一个索引。之后，我们将修改定义，展示定制索引的其他方法。**

## 2.`@Index`注释

让我们先快速回顾一下。数据库索引是一种**数据结构，它以额外的写入和存储空间为代价，提高了对表**的数据检索操作的速度。大多数情况下，它是单个表中所选列数据的**副本。我们应该创建索引来提高持久层的性能。**

JPA 允许我们通过使用`@Index`从代码中定义索引来实现这一点。该注释由模式生成过程解释，自动创建工件。注意，没有必要为我们的实体指定任何索引。

现在，让我们来看一下定义。

### 2.1.`javax.persistence.Index`

索引支持最终由`[javax.persistence.Index](https://web.archive.org/web/20220630125253/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Index.html)`添加到 JPA 2.1 规范中。这个注释让我们为表定义一个索引，并相应地定制它:

```java
@Target({})
@Retention(RUNTIME)
public @interface Index {
    String name() default "";
    String columnList();
    boolean unique() default false;
}
```

正如我们所看到的，只有`columnList`属性是强制的，我们必须定义它。稍后，我们将通过示例更好地了解每个参数。

这里需要注意的一点是，注释不支持更改默认的索引算法——btree。

### 2.2.JPA 与 Hibernate

我们知道 JPA 只是一个规范。为了正常工作，我们还需要指定一个持久性提供者。默认情况下，Hibernate 框架是 Spring 提供的 JPA 实现。关于它的更多内容，你可以在这里阅读[。](/web/20220630125253/https://www.baeldung.com/spring-boot-hibernate)

我们应该记得索引支持是很晚才添加到 JPA 中的。在此之前，许多 ORM 框架通过引入自己的定制实现来支持索引，这可能会有所不同。Hibernate 框架也这么做了，并引入了 [`org.hibernate.annotations.Index`](https://web.archive.org/web/20220630125253/https://docs.jboss.org/hibernate/orm/5.4/javadocs/org/hibernate/annotations/Index.html) 注释。在使用该框架时，我们必须小心，自从 JPA 2.1 规范支持以来，它已经被弃用，我们应该使用 JPA 的框架。

现在，当我们有了一些技术背景，我们可以通过例子，并在 JPA 中定义我们的第一个索引。

## 3.定义`@Index`

在本节中，我们将实现我们的索引。稍后，我们将尝试修改它，呈现不同的定制可能性。

在我们开始之前，我们需要[正确初始化我们的项目并定义一个模型](/web/20220630125253/https://www.baeldung.com/jpa-entities)。

让我们实现一个`Student`实体:

```java
@Entity
@Table
public class Student implements Serializable {
    @Id
    @GeneratedValue
    private Long id;
    private String firstName;
    private String lastName;

    // getters, setters
}
```

当我们有了模型后，让我们实现第一个索引。我们所要做的就是添加一个`@Index`注释。我们在`indexes`属性下的 [`@Table`](https://web.archive.org/web/20220630125253/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Table.html) 中标注。让我们记住指定列的名称:

```java
@Table(indexes = @Index(columnList = "firstName"))
```

我们已经使用`firstName`列声明了第一个索引。当我们执行模式创建过程时，我们可以验证它:

```java
[main] DEBUG org.hibernate.SQL -
  create index IDX2gdkcjo83j0c2svhvceabnnoh on Student (firstName)
```

现在，是时候修改我们的声明来显示额外的特性了。

### 3.1.`@Index`姓名

正如我们所看到的，我们的索引必须有一个名字。默认情况下，如果我们不这样指定，它是一个提供者生成的值。当我们想要一个自定义标签时，我们应该简单地添加`name`属性:

```java
@Index(name = "fn_index", columnList = "firstName")
```

此变体使用用户定义的名称创建索引:

```java
[main] DEBUG org.hibernate.SQL -
  create index fn_index on Student (firstName)
```

此外，通过在`name`中指定模式名，我们可以在不同的模式中创建索引:

`@Index(name = "schema2.fn_index", columnList = "firstName")`

### 3.2.多列`@Index`

现在，让我们仔细看看`columnList`语法:

```java
column ::= index_column [,index_column]*
index_column ::= column_name [ASC | DESC]
```

正如我们已经知道的，我们可以指定要包含在索引中的列名。当然，我们可以为单个索引指定多个列。我们通过用逗号分隔名称来做到这一点:

```java
@Index(name = "mulitIndex1", columnList = "firstName, lastName")

@Index(name = "mulitIndex2", columnList = "lastName, firstName")
```

```java
[main] DEBUG org.hibernate.SQL -
  create index mulitIndex1 on Student (firstName, lastName)

[main] DEBUG org.hibernate.SQL -
  create index mulitIndex2 on Student (lastName, firstName)
```

请注意，持久性提供程序必须遵守指定的列顺序。在我们的示例中，索引略有不同，即使它们指定了相同的一组列。

### 3.3.`@Index`订单

正如我们在上一节中回顾的语法，我们也可以在`column_name`后指定`ASC`(升序)和`DESC`(降序)值。我们用它来设置索引列中值的排序顺序:

```java
@Index(name = "mulitSortIndex", columnList = "firstName, lastName DESC")
```

```java
[main] DEBUG org.hibernate.SQL -
  create index mulitSortIndex on Student (firstName, lastName desc)
```

我们可以指定每列的顺序。如果没有，则假定升序。

### 3.4.`@Index`独特性

最后一个可选参数是一个`unique`属性，它定义索引是否惟一。唯一索引确保索引字段不会存储重复值。默认是`false`。如果我们想改变它，我们可以声明:

```java
@Index(name = "uniqueIndex", columnList = "firstName", unique = true)
```

```java
[main] DEBUG org.hibernate.SQL -
  alter table Student add constraint uniqueIndex unique (firstName)
```

当我们以这种方式创建一个索引时，我们在我们的列上添加了一个惟一性约束，类似地， [`@Column`](https://web.archive.org/web/20220630125253/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Column.html) 注释上的`unique`属性也是如此。`@Index`比`@Column`更有优势，因为它可以声明多列唯一约束:

```java
@Index(name = "uniqueMulitIndex", columnList = "firstName, lastName", unique = true)
```

### 3.5.单个实体上的多个`@Index`

到目前为止，我们已经实现了索引的不同变体。当然，我们并不局限于在实体上声明单个索引。让我们收集我们的声明并一次指定每个索引。我们通过在大括号中用逗号分隔重复`@Index`注释来做到这一点:

```java
@Entity
@Table(indexes = {
  @Index(columnList = "firstName"),
  @Index(name = "fn_index", columnList = "firstName"),
  @Index(name = "mulitIndex1", columnList = "firstName, lastName"),
  @Index(name = "mulitIndex2", columnList = "lastName, firstName"),
  @Index(name = "mulitSortIndex", columnList = "firstName, lastName DESC"),
  @Index(name = "uniqueIndex", columnList = "firstName", unique = true),
  @Index(name = "uniqueMulitIndex", columnList = "firstName, lastName", unique = true)
})
public class Student implements Serializable
```

此外，我们还可以为同一组列创建多个索引。

### 3.6.主关键字

当我们谈到索引时，我们必须在主键上停留一会儿。我们知道，由 [`EntityManager`](/web/20220630125253/https://www.baeldung.com/hibernate-entitymanager) 管理的每个实体必须指定一个映射到主键的标识符。

通常，主键是特定类型的唯一索引。值得补充的是，我们不必像前面介绍的那样声明这个键的定义。一切都是由 [`@Id`](https://web.archive.org/web/20220630125253/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/Id.html) 自动标注完成的。

### 3.7.非实体`@Index`

在我们学习了实现索引的不同方法之后，我们应该提到`@Table`并不是指定它们的唯一地方。同样，我们可以在`@SecondaryTable`、@CollectionTable、`@JoinTable`、`@TableGenerator`注释中声明索引。这些例子不在本文的讨论范围内。更多详情请查看[*javax . persistence*`JavaDoc`](https://web.archive.org/web/20220630125253/https://javaee.github.io/javaee-spec/javadocs/javax/persistence/package-summary.html)。

## 4.结论

在本文中，我们讨论了使用 JPA 声明索引。我们从复习关于它们的常识开始。后来，我们实现了我们的第一个索引，并通过示例，了解了如何通过更改名称、包含的列、顺序和惟一性来定制它。最后，我们讨论了主键以及可以声明它们的其他方法和位置。

和往常一样，文章中的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220630125253/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)