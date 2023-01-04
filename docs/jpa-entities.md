# 定义 JPA 实体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-entities>

## 1.介绍

在本教程中，我们将学习实体的基础知识，以及在 JPA 中定义和定制实体的各种注释。

## 2.实体

JPA 中的实体只不过是表示可以持久存储到数据库中的数据的 POJOs。实体表示存储在数据库中的表。实体的每个实例代表表中的一行。

### 2.1.`Entity`注解

假设我们有一个名为`Student,` 的 POJO，它代表一个学生的数据，我们希望将它存储在数据库中:

```java
public class Student {

    // fields, getters and setters

}
```

为了做到这一点，我们应该定义一个实体，以便 JPA 知道它。

所以让我们通过使用`@Entity`注释来定义它。我们必须在类级别指定这个注释。**我们还必须确保实体有一个无参数的构造函数和一个主键:**

```java
@Entity
public class Student {

    // fields, getters and setters

}
```

实体名默认为类名。我们可以使用`name`元素更改它的名称:

```java
@Entity(name="student")
public class Student {

    // fields, getters and setters

}
```

因为各种 JPA 实现会尝试对我们的实体进行子类化以提供它们的功能，**实体类不能被声明为`final`。**

### 2.2.`Id`注解

每个 JPA 实体必须有一个唯一标识它的主键。[`@Id`标注](/web/20221124000447/http://www.baeldung.com/hibernate-identifiers)定义主键。我们可以用不同的方式生成标识符，这由`@GeneratedValue`注释指定。

使用`strategy`元素，我们可以从四种 id 生成策略中进行选择。**该值可以是`AUTO, TABLE, SEQUENCE,` 或 `IDENTITY:`或**

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private String name;

    // getters and setters
}
```

如果我们指定`GenerationType`。`AUTO`，JPA 提供者将使用任何它想要的策略来生成标识符。

如果我们注释实体的字段，JPA 提供者将使用这些字段来获取和设置实体的状态。除了字段访问，我们还可以进行属性访问或混合访问，这使我们能够在同一个实体中同时使用字段和属性访问**。**

### 2.3.`Table`注解

在大多数情况下，**数据库中的表名和实体名不会相同。**

在这些情况下，我们可以使用`@Table`注释来指定表名:

```java
@Entity
@Table(name="STUDENT")
public class Student {

    // fields, getters and setters

}
```

我们还可以使用`schema`元素提到模式:

```java
@Entity
@Table(name="STUDENT", schema="SCHOOL")
public class Student {

    // fields, getters and setters

}
```

模式名有助于区分一组表和另一组表。

如果我们不使用`@Table`注释，表的名称将是实体的名称。

### 2.4.`Column`注解

就像`@Table`注释一样，我们可以使用`@Column`注释来提及表中某一列的详细信息。

`@Column`注释有许多元素，如`name, length, nullable, and unique`:

```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;

    // other fields, getters and setters
}
```

元素指定了表中列的名称。元素指定了它的长度。`nullable`元素指定该列是否可以为空，而`unique`元素指定该列是否唯一。

如果我们不指定这个注释，表中列的名称将是字段的名称。

### 2.5.`Transient`注解

有时，我们可能希望**使一个字段不持久。**我们可以使用`@Transient`注释来这样做。它指定字段不会被持久化。

例如，我们可以从出生日期计算学生的年龄。

所以让我们用`@Transient`注释来注释字段`age`:

```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Column(name="STUDENT_NAME", length=50, nullable=false)
    private String name;

    @Transient
    private Integer age;

    // other fields, getters and setters
}
```

因此，字段`age` 不会被保存到表中。

### 2.6.`Temporal`注解

在某些情况下，我们可能需要在表中保存临时值。

对于这一点，我们有[`@Temporal`注解](/web/20221124000447/http://www.baeldung.com/hibernate-date-time):

```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;

    @Transient
    private Integer age;

    @Temporal(TemporalType.DATE)
    private Date birthDate;

    // other fields, getters and setters
}
```

然而，在 JPA 2.2 中，我们也支持`java.time.LocalDate, java.time.LocalTime, java.time.LocalDateTime, java.time.OffsetTime` 和`java.time.OffsetDateTime.`

### 2.7.`Enumerated`注解

有时，我们可能想要持久化一个 Java `enum`类型。

我们可以使用`@Enumerated`注释来指定`enum`应该按名称还是按序号持久化(默认):

```java
public enum Gender {
    MALE, 
    FEMALE
} 
```

```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;

    @Transient
    private Integer age;

    @Temporal(TemporalType.DATE)
    private Date birthDate;

    @Enumerated(EnumType.STRING)
    private Gender gender;

    // other fields, getters and setters
}
```

实际上，**如果我们要通过`enum`的序号来持久化`Gender`，我们根本不需要指定`@Enumerated`注释。**

然而，为了通过`enum` 名称持久化`Gender`，我们已经用`EnumType.STRING.`配置了注释

## 3.结论

在本文中，我们学习了什么是 JPA 实体以及如何创建它们。我们还了解了可以用来进一步定制实体的不同注释。

这篇文章的完整代码可以在 Github 上找到[。](https://web.archive.org/web/20221124000447/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)