# JPA @嵌入式和@可嵌入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-embedded-embeddable>

## 1.概观

在本教程中，我们将了解如何将一个包含嵌入属性的实体映射到一个数据库表中。

因此，出于这个目的，我们将使用由 [Java 持久性 API (JPA)](/web/20220629014641/https://www.baeldung.com/jpa-hibernate-difference) 提供的`@Embeddable`和`@Embedded`注释。

## 2.数据模型上下文

首先我们来定义一个叫`company`的表。

`company`表将存储公司名称、地址和电话等基本信息，以及联系人的信息:

```java
public class Company {

    private Integer id;

    private String name;

    private String address;

    private String phone;

    private String contactFirstName;

    private String contactLastName;

    private String contactPhone;

    // standard getters, setters
}
```

然而，联系人似乎应该被抽象成一个单独的类。问题是**我们不想为这些细节创建一个单独的表。**那么，让我们看看我们能做些什么。

## 3.`@Embeddable`

**JPA 提供了`@Embeddable`注释来声明一个类将被其他实体嵌入。**

让我们定义一个类来提取联系人的详细信息:

```java
@Embeddable
public class ContactPerson {

    private String firstName;

    private String lastName;

    private String phone;

    // standard getters, setters
}
```

## 4.`@Embedded`

**JPA 注释`@Embedded`用于将一个类型嵌入另一个实体。**

接下来让我们修改我们的`Company`类。我们将添加 JPA 注释，我们还将使用`ContactPerson` 代替单独的字段:

```java
@Entity
public class Company {

    @Id
    @GeneratedValue
    private Integer id;

    private String name;

    private String address;

    private String phone;

    @Embedded
    private ContactPerson contactPerson;

    // standard getters, setters
}
```

结果，我们有了我们的实体`Company`，嵌入了联系人的详细信息，并映射到一个数据库表。

不过，我们还有一个问题，那就是 JPA 如何将这些字段映射到数据库列。

## 5.属性覆盖

事情是这样的，我们的字段在最初的`Company`类中被称为`contactFirstName`，现在在我们的`ContactPerson `类中被称为`firstName`。所以，JPA 会想把这些分别映射到`contact_first_name `和`first_name,` 。

除了不够理想之外，它还会让我们现在复制的`phone`专栏破产。

**因此，我们可以使用`@AttributeOverrides`和`@AttibuteOverride`来覆盖嵌入类型的列属性。**

让我们将它添加到我们的`Company `实体中的`ContactPerson `字段:

```java
@Embedded
@AttributeOverrides({
  @AttributeOverride( name = "firstName", column = @Column(name = "contact_first_name")),
  @AttributeOverride( name = "lastName", column = @Column(name = "contact_last_name")),
  @AttributeOverride( name = "phone", column = @Column(name = "contact_phone"))
})
private ContactPerson contactPerson;
```

注意，由于这些注释在字段上，我们可以为每个封闭实体设置不同的覆盖。

## 6.结论

在本教程中，我们用一些嵌入属性配置了一个实体，并将它们映射到与封闭实体相同的数据库表中。为此，我们使用了 Java 持久性 API 提供的`@Embedded`、`@Embeddable`、`@AttributeOverrides`和`@AttributeOverride`注释。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220629014641/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-annotations)