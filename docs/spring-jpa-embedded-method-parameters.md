# Spring JPA @Embedded 和@EmbeddedId

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jpa-embedded-method-parameters>

## 1.概观

在本教程中，我们将介绍如何使用`@EmbeddedId`注释和`findBy`方法来查询基于组合键的 JPA 实体。

因此**我们将使用`@EmbeddeId`和`@Embeddable`注释来表示 JPA 实体**中的组合键。我们还需要使用 Spring `JpaRepository`来实现我们的目标。

我们将专注于通过部分主键查询对象。

## 2.需要为 **`@Embeddable`和`@EmbeddedId`**

在软件中，当我们需要一个复合主键来定义一个表中的条目时，我们会遇到许多用例。**复合主键是使用多列来唯一标识表中某一行的键**。

我们通过在类上使用`@Embeddable`注释来表示 Spring 数据中的复合主键。然后，通过在`@Embeddable`类型的字段上使用`@EmbeddedId`注释，将这个键作为复合主键嵌入到表的相应实体类中。

## 3.例子

考虑一个`book`表，其中一个`book`记录有一个由`author`和`name`组成的复合主键。有时候，我们可能想通过主键的一部分找到`books`。例如，用户可能希望仅通过特定的`author`来搜索书籍。我们将学习如何用 JPA 做到这一点。

我们的主要应用程序将由带有`@EmbeddedId BookId.`的`@Embeddable BookId`和`@Entity Book`组成

### 3.1.`@Embeddable`

让我们在这一节定义我们的`BookId`类。`author`和`name`将指定一个唯一的`BookId` —这个类是`Serializable`，并且实现了`equals`和`hashCode` 两种方法:

```java
@Embeddable
public class BookId implements Serializable {

    private String author;
    private String name;

    // standard getters and setters
}
```

### 3.2.`@Entity`和`@EmbeddedId`

我们的`Book`实体有`@EmbeddedId` `BookId` 和其他与一个`book`相关的字段。`BookId`告诉 JPA`Book`实体有一个组合键:

```java
@Entity
public class Book {

    @EmbeddedId
    private BookId id;
    private String genre;
    private Integer price;

    //standard getters and setters
}
```

### 3.3.JPA 存储库和方法命名

让我们通过用实体`Book`和`BookId`扩展`JpaRepository` 来快速定义我们的 JPA 存储库接口:

```java
@Repository
public interface BookRepository extends JpaRepository<Book, BookId> {

    List<Book> findByIdName(String name);

    List<Book> findByIdAuthor(String author);
}
```

我们使用部分`id`变量的字段名来派生我们的 Spring 数据查询方法。因此，JPA 将部分主键查询解释为:

```java
findByIdName -> directive "findBy" field "id.name"
findByIdAuthor -> directive "findBy" field "id.author"
```

## 4.结论

JPA 可以用来有效地映射组合键，并通过派生查询来查询它们。

在本文中，我们看到了一个运行部分 id 字段搜索的小例子。我们看了表示复合主键的`@Embeddable` 注释和在实体中插入复合键的`@EmbeddedId` 注释。

最后，我们看到了**如何使用`JpaRepository`** `**findBy**` **派生方法**来搜索部分 id 字段。

和往常一样，本教程的示例代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221205203259/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-annotations)