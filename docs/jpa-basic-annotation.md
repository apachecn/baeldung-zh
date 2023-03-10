# JPA @Basic 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-basic-annotation>

## 1.概观

在这个快速教程中，我们将探索 JPA `@Basic`注释。我们还将讨论`@Basic`和`@Column` JPA 注释的区别。

## 2.基本类型

JPA 支持各种 Java 数据类型作为实体的持久字段，通常称为基本类型。

**一个基本类型直接映射到数据库中的一列。**这些包括 Java 原语和它们的包装类`String`、`java.math.BigInteger`和` java.math.BigDecimal`，各种可用的日期时间类、枚举和任何其他实现`java.io.Serializable`的类型。

和其他 ORM 供应商一样，Hibernate 维护一个基本类型的注册表，并使用它来解析列的特定`org.hibernate.type.Type`。

## 3.`@Basic`注释

我们可以使用 `@Basic`注释来标记一个基本的类型属性:

```java
@Entity
public class Course {

    @Basic
    @Id
    private int id;

    @Basic
    private String name;
    ...
}
```

换句话说，**字段或属性上的`@Basic`注释表示它是一个基本类型，Hibernate 应该使用标准映射来保证它的持久性。**

**注意，这是一个可选的注释。**因此，我们可以将我们的`Course`实体重写为:

```java
@Entity
public class Course {

    @Id
    private int id;

    private String name;
    ...
}
```

**当我们没有为一个基本类型属性指定`@Basic`注释时，它是隐式假定的，并且应用这个注释的默认值。**

## 4.为什么要用`@Basic`标注？

`@Basic`注释有两个属性，`optional`和`fetch`。让我们仔细看看每一个。

**`optional`属性是一个`boolean`参数，定义被标记的字段或属性是否允许`null`。**默认为`true`。因此，如果字段不是基本类型，默认情况下，基础列被假定为`nullable` 。

`fetch`属性接受枚举`Fetch`的一个成员，枚举`Fetch`的成员**指定被标记的字段或属性是应该被延迟加载还是被急切获取。**默认为`FetchType.EAGER`，但是我们可以通过将其设置为`FetchType.LAZY.` 来允许延迟加载

只有当我们将一个大的`Serializable`对象映射为基本类型时，延迟加载才有意义，因为在这种情况下，字段访问成本可能很高。

我们有一个详细的教程，涵盖了 Hibernate 中的[急/懒加载，对这个主题进行了更深入的探讨。](/web/20220701015525/https://www.baeldung.com/hibernate-lazy-eager-loading)

现在，假设我们不想让`Course`的`name`使用`nulls`，并且也想惰性地加载那个属性。然后，我们将我们的`Course`实体定义为:

```java
@Entity
public class Course {

    @Id
    private int id;

    @Basic(optional = false, fetch = FetchType.LAZY)
    private String name;
    ...
}
```

**当愿意偏离`optional`和`fetch`** 参数的默认值时，我们应该明确使用`@Basic`注释。根据我们的需要，我们可以指定其中一个或两个属性。

## 5.JPA `@Basic` vs `@Column`

让我们来看看`@Basic`和`@Column`注释的区别:

*   注释的属性应用于 JPA 实体，而属性应用于数据库列
*   `@Basic`批注的`optional`属性定义实体字段是否可以为`null`；另一方面，`@Column`注释的`nullable`属性指定相应的数据库列是否可以是`null`
*   我们可以使用`@Basic`来表示一个字段应该被延迟加载
*   `@Column`注释允许我们指定映射的数据库列的`name`

## 6.结论

在本文中，我们学习了何时以及如何使用 JPA 的`@Basic`注释。我们还讨论了它与`@Column`注释的不同之处。

像往常一样，Github 上有代码示例。