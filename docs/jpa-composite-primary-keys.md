# JPA 中的复合主键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-composite-primary-keys>

## 1.介绍

在本教程中，我们将学习 JPA 中的复合主键和相应的注释。

## 延伸阅读:

## [Spring JPA @Embedded 和@EmbeddedId](/web/20220926131649/https://www.baeldung.com/spring-jpa-embedded-method-parameters)

Learn how to use the `@EmbeddeId` and `@Embeddable` annotations to represent composite keys in JPA entities.[Read more](/web/20220926131649/https://www.baeldung.com/spring-jpa-embedded-method-parameters) →

## [JPA 什么时候设置主键](/web/20220926131649/https://www.baeldung.com/jpa-strategies-when-set-primary-key)

Learn about the different strategies JPA uses to generate the primary key for an entity and at which moment each strategy sets the key value during persistence.[Read more](/web/20220926131649/https://www.baeldung.com/jpa-strategies-when-set-primary-key) →

## [用 JPA 返回自动生成的 Id](/web/20220926131649/https://www.baeldung.com/jpa-get-auto-generated-id)

In this tutorial, we'll discuss how we can handle auto-generated ids with JPA.[Read more](/web/20220926131649/https://www.baeldung.com/jpa-get-auto-generated-id) →

## 2.复合主键

复合主键也称为组合键，是两列或更多列的组合，构成表的主键。

在 JPA 中，我们有两个选项来定义组合键:`@IdClass`和`@EmbeddedId`注释。

**为了定义复合主键，我们应该遵循一些规则:**

*   复合主键类必须是公共的。
*   它必须有一个无参数构造函数。
*   它必须定义`equals()`和`hashCode()`方法。
*   必须是`S` `erializable.`

## 3.`IdClass`注解

假设我们有一个名为`Account`的表，它有两列，`accountNumber`和 `accountType,`组成了组合键。现在我们必须在 JPA 中映射它。

按照 JPA 规范，让我们用这些主键字段创建一个`AccountId`类:

```java
public class AccountId implements Serializable {
    private String accountNumber;

    private String accountType;

    // default constructor

    public AccountId(String accountNumber, String accountType) {
        this.accountNumber = accountNumber;
        this.accountType = accountType;
    }

    // equals() and hashCode()
}
```

接下来让我们将`AccountId`类与实体`Account`相关联。

为了做到这一点，我们需要用`[@IdClass](/web/20220926131649/https://www.baeldung.com/hibernate-identifiers)`注释来注释实体。我们还必须在实体*帐户*中声明来自`AccountId`类的字段，并用`@Id`对它们进行注释:

```java
@Entity
@IdClass(AccountId.class)
public class Account {
    @Id
    private String accountNumber;

    @Id
    private String accountType;

    // other fields, getters and setters
}
```

## 4.`EmbeddedId`注解

`@EmbeddedId`是`@IdClass`注释的替代选项。

让我们考虑另一个例子，在这个例子中，我们必须保存一个`Book`的一些信息，用`title` 和`language` 作为主键字段。

在这种情况下， ***的主键类，*的 BookId 必须用`@Embeddable`** 标注:

```java
@Embeddable
public class BookId implements Serializable {
    private String title;
    private String language;

    // default constructor

    public BookId(String title, String language) {
        this.title = title;
        this.language = language;
    }

    // getters, equals() and hashCode() methods
}
```

然后我们需要使用`[@EmbeddedId](/web/20220926131649/https://www.baeldung.com/jpa-many-to-many)`将这个类嵌入到`B` *ook* 实体中:

```java
@Entity
public class Book {
    @EmbeddedId
    private BookId bookId;

    // constructors, other fields, getters and setters
}
```

## 5.`@IdClass`对`@EmbeddedId`

正如我们所看到的，这两者表面上的区别是，使用`@IdClass`我们必须指定两次列，一次在`AccountId`中，另一次在`Account;` 中，但是使用`@EmbeddedId` 我们没有。

不过，还有一些其他的权衡。

例如，这些不同的结构会影响我们编写的 JPQL 查询。

使用`@IdClass`，查询稍微简单一点:

```java
SELECT account.accountNumber FROM Account account
```

使用`@EmbeddedId`，我们必须进行一次额外的遍历:

```java
SELECT book.bookId.title FROM Book book
```

此外， **`@IdClass`在我们** **使用我们不能修改的组合键类的地方非常有用。**

如果我们要单独访问组合键的各个部分，我们可以使用`@IdClass,` ，但是在我们经常使用完整标识符作为对象的地方，最好使用`@EmbeddedId`。

## 6.结论

在这篇简短的文章中，我们探讨了 JPA 中的复合主键。

和往常一样，本文的完整代码可以在 Github 上找到。