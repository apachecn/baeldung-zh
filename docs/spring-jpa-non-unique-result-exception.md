# Spring Data JPA 中的 NonUniqueResultException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jpa-non-unique-result-exception>

## 1。简介

[Spring Data JPA](/web/20221231132733/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 为访问存储在各种关系数据库中的数据提供了一个简单而一致的接口，让开发者更容易编写与数据库无关的代码。它还消除了对大量样板代码的需要，允许开发人员专注于构建他们的应用程序的业务逻辑。

然而，我们仍然需要确保正确的返回类型，否则会抛出一个*异常*。在本教程中，我们将重点关注`NonUniqueResultException`。我们将了解导致它的原因，以及遇到它时如何修复我们的代码。

## 2。`NonUniqueResultException`

Spring Data JPA 框架抛出了一个运行时异常`NonUniqueResultException`,当一个查询方法被期望返回一个结果，但是却发现了不止一个结果。当使用 Spring Data JPA 的查询方法之一(如 `findById()` 、 ` findOne()` 、 或不返回集合的自定义方法 )执行查询时，会出现这种情况。

当抛出`NonUniqueResultException`时，意味着正在使用的方法被设计为返回单个结果，但是它找到了多个结果。这可能是由于错误的查询或数据库中不一致的数据造成的。

## 3。示例

让我们使用从我们的文章中了解到的这个`Entity`,[用 Spring Data JPA](/web/20221231132733/https://www.baeldung.com/spring-data-jpa-query-by-date) 按日期和时间查询实体:

```java
@Entity
public class Article {

    @Id
    @GeneratedValue
    private Integer id;

    @Temporal(TemporalType.DATE)
    private Date publicationDate;

    @Temporal(TemporalType.TIME)
    private Date publicationTime;

    @Temporal(TemporalType.TIMESTAMP)
    private Date creationDateTime;
    }
}
```

现在，让我们创建我们的` ArticleRepository`并添加两个方法:

```java
public interface ArticleRepository extends JpaRepository<Article, Integer> {

    List<Article> findAllByPublicationTimeBetween(Date publicationTimeStart, Date publicationTimeEnd);

    Article findByPublicationTimeBetween(Date publicationTimeStart, Date publicationTimeEnd);
} 
```

这两种方法的唯一区别是，`findAllByPublicationTimeBetween()`有一个`List` < `Article>`作为返回类型，`findByPublicationTimeBetween()`有一个单个的`Article` 作为返回类型。

当我们执行第一个方法`findAllByPublicationTimeBetween`时，我们总是会得到一个集合。根据数据库中的数据，我们可以得到一个空的`List`或一个带有一个或多个`Article`实例的`List`。

第二种方法`findByPublicationTimeBetween`在技术上也是可行的，假设数据库包含零个或一个匹配的`Article`。如果给定的查询没有单个条目，该方法将返回`null`。另一方面，如果有一个对应的`Article`，它将返回单个的`Article`。

但是，如果有多个`Article`与对`findByPublicationTimeBetween`的查询相匹配，该方法将抛出一个`NonUniqueResultException`，然后将其包装在一个`IncorrectResultSizeDataAccessException`中。

当像这样的异常在运行时随机出现时，这表明数据库设计或我们的方法实现有问题。在下一节中，我们将学习如何避免这种错误。

## 4。`NonUniqueResultException`避开 T2 的小技巧

为了避免`NonUniqueResultException`，仔细设计数据库查询并正确使用 Spring Data JPA 的查询方法非常重要。设计查询时，确保它总是返回预期数量的结果是很重要的。我们可以通过仔细指定查询标准来实现这一点，比如使用惟一键或其他标识信息。

在设计我们的查询方法时，我们应该遵循一些基本规则来避免`NonUniqueResultExceptions`:

*   **如果可能返回多个值，我们应该使用一个** **`List` 或 `Set`** 作为返回类型。
*   如果我们能够**通过数据库设计**确保只有一个返回值，我们就只使用一个返回值。当我们寻找一个唯一的键时，比如一个`Id`、`UUID`，或者，根据数据库的设计，它也可以是一个保证唯一的电子邮件或电话号码。
*   确保只有一个返回值的另一种方法是用 **[将返回](/web/20221231132733/https://www.baeldung.com/jpa-limit-query-results)限制为单个元素**。这可能是有用的，例如，如果我们总是想要最新的`Article`。

## 5。结论

`NonUniqueResultException`是使用 Spring 数据 JPA 时需要理解和避免的一个重要例外。 当一个查询应该返回一个结果，但却找到多个结果时，就会出现这种情况。我们可以通过确保我们的`JpaRepository`方法返回正确数量的元素并相应地指定正确的返回类型来防止这种情况。

通过理解并正确避免`NonUniqueResultException`，我们可以确保我们的应用程序能够一致且可靠地从数据库中访问数据。

和往常一样，这些例子也可以在 GitHub 上找到[。](https://web.archive.org/web/20221231132733/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-3)