# Hibernate 的“非空属性引用空值或瞬态值”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-not-null-error>

## 1.**概述**

在本文中，我们将学习 Hibernate 的`PropertyValueException.` ，特别是，我们将考虑`“not-null property references a null or transient value”` 错误消息。

Hibernate 主要会在两种情况下抛出`PropertyValueException` :

*   为标有`nullable = false`的列保存`null`值时
*   当保存带有引用未保存实例的关联的实体时

## 2.Hibernate 的可空性检查

首先，我们来讨论 Hibernate 的 [`@Column(nullable = false)`](/web/20221208143856/https://www.baeldung.com/hibernate-notnull-vs-nullable) 注释。**如果没有其他 Bean 验证，我们可以依靠** **Hibernate 的可空性检查。**

此外，我们可以通过设置`hibernate.check_nullability = true.` 来实施这种验证。为了重现下面的例子，我们需要启用可空性检查。

## 3.为非空列保存一个`null` 值

现在，让我们利用 Hibernate 的验证机制来重现一个简单用例的错误。我们将尝试保存一个`@Entity`而不设置它的强制字段。对于这个例子，我们将使用简单的 *Book* 类:

```java
@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false)
    private String title;

    // getters and setters
}
```

*标题*列的`nullable` 标志被设置为 `false.`,我们现在可以保存一个`Book` 对象，而不用设置它的标题并断言`PropertyValueException`被抛出:

```java
@Test
public void whenSavingEntityWithNullMandatoryField_thenThrowPropertyValueException() {    
    Book book = new Book();

    assertThatThrownBy(() -> session.save(book))
      .isInstanceOf(PropertyValueException.class)
      .hasMessageContaining("not-null property references a null or transient value");
} 
```

因此，我们只需要在保存实体之前设置必填字段，就可以解决这个问题:`book.setTitle(“Clean Code”)`。

## 4.保存引用未保存实例的关联

在这一节中，我们将探索一个更复杂设置的常见场景。对于这个例子，我们将使用共享双向关系的`Author`和`Article`实体:

```java
@Entity
public class Author {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    private String name;

    @OneToMany
    @Cascade(CascadeType.ALL)
    private List<Article> articles;

    // constructor, getters and setters
}
```

`articles` 字段具有`@Cascade(CascadeType.` ALL)注释。因此，当我们保存一个`Author` 实体时，操作将通过所有的`Article` 对象传播:

```java
@Entity
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    private String title;

    @ManyToOne(optional = false)
    private Author author;

    // constructor, getters and setters
}
```

现在，让我们试着保存一个`Author `和一些`Articles`，看看会发生什么:

```java
@Test
public void whenSavingBidirectionalEntityiesithCorrectParent_thenDoNotThrowException() {
    Author author = new Author("John Doe");
    author.setArticles(asList(new Article("Java tutorial"), new Article("What's new in JUnit5")));

    assertThatThrownBy(() -> session.save(author))
      .isInstanceOf(PropertyValueException.class)
      .hasMessageContaining("not-null property references a null or transient value");
} 
```

当我们处理双向关系时，我们可能会犯一个常见的错误，即忘记更新双方的赋值。如果我们改变`Author`类中的`setter`来更新所有的子`articles` ，我们就可以避免这种情况。

为了说明本文中出现的所有用例，我们将为此创建一个不同的方法。但是，从父实体的`setter`设置这些字段是一个好的做法:

```java
public void addArticles(List<Article> articles) {
    this.articles = articles;
    articles.forEach(article -> article.setAuthor(this));
}
```

我们现在可以使用新方法来设置赋值，并且不会出现错误:

```java
@Test
public void whenSavingBidirectionalEntitesWithCorrectParent_thenDoNotThrowException() {
    Author author = new Author("John Doe");
    author.addArticles(asList(new Article("Java tutorial"), new Article("What's new in JUnit5")));

    session.save(author);
}
```

## 5.结论

在本文中，我们看到了 Hibernate 的验证机制是如何工作的。首先，我们发现了如何在项目中启用`nullability checking` 。之后举例说明了造成`PropertyValueException` 的主要原因，并学习了如何修复。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-exceptions)