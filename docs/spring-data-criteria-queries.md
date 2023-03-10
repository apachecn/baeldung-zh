# 在 Spring 数据应用程序中使用条件查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-criteria-queries>

## 1.介绍

Spring Data JPA 提供了很多处理实体的方法，包括[查询方法](/web/20221115111200/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)和自定义 [JPQL 查询](/web/20221115111200/https://www.baeldung.com/spring-data-jpa-query)。但是有时候，我们需要更程序化的方法，比如[标准 API](/web/20221115111200/https://www.baeldung.com/hibernate-criteria-queries) 或者 [QueryDSL](/web/20221115111200/https://www.baeldung.com/querydsl-with-jpa-tutorial) 。

**Criteria API 提供了一种创建类型化查询**的编程方式，这有助于我们避免语法错误。此外，当我们将它与元模型 API 一起使用时，它会进行编译时检查，以确认我们是否使用了正确的字段名和类型。

然而，它也有不利的一面；我们必须编写充斥着样板代码的冗长逻辑。

在本教程中，我们将学习如何使用标准查询来实现我们的定制 DAO 逻辑。我们还将说明 Spring 如何帮助减少样板代码。

## 2.示例应用程序

为了简化示例，我们将以多种方式实现同一个查询:根据作者姓名和包含`String`的书名查找书籍。

这里是`Book`实体:

```java
@Entity
class Book {

    @Id
    Long id;
    String title;
    String author;

    // getters and setters

}
```

因为我们想保持简单，所以在本教程中我们不会使用元模型 API。

## 3.`@Repository`阶级

我们知道，在 Spring 组件模型中，**我们应该将我们的数据访问逻辑放在`@Repository`bean**中。当然，这个逻辑可以使用任何实现，比如 Criteria API。

为此，我们只需要一个`EntityManager`实例，我们可以自动连接它:

```java
@Repository
class BookDao {

    EntityManager em;

    // constructor

    List<Book> findBooksByAuthorNameAndTitle(String authorName, String title) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Book> cq = cb.createQuery(Book.class);

        Root<Book> book = cq.from(Book.class);
        Predicate authorNamePredicate = cb.equal(book.get("author"), authorName);
        Predicate titlePredicate = cb.like(book.get("title"), "%" + title + "%");
        cq.where(authorNamePredicate, titlePredicate);

        TypedQuery<Book> query = em.createQuery(cq);
        return query.getResultList();
    }

}
```

上面的代码遵循标准的 API 工作流程:

*   首先，我们得到一个`CriteriaBuilder`引用，我们可以用它来创建查询的不同部分。
*   使用`CriteriaBuilder`，我们创建一个`CriteriaQuery<Book>`，它描述了我们想要在查询中做什么。它还声明结果中行的类型。
*   我们用`CriteriaQuery<Book>,`声明查询的起点(`Book`实体)，并将其存储在`book`变量中以备后用。
*   接下来，我们用`CriteriaBuilder,`创建针对我们的`Book`实体的谓词。注意，这些谓词还没有任何效果。
*   我们将两个谓词都应用到我们的`CriteriaQuery.` `CriteriaQuery.where(Predicate…)`中，将它的参数组合成一个逻辑`and`。这就是我们将这些谓词与查询联系起来的时候。
*   之后，我们从我们的`CriteriaQuery.`创建一个`TypedQuery<Book>`实例
*   最后，我们返回所有匹配的`Book`实体。

注意，因为我们用`@Repository`，**标记了 DAO 类，Spring 为这个类启用了异常翻译**。

## 4.用自定义方法扩展存储库

拥有[自动定制查询](/web/20221115111200/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#customquery)是一个强大的 Spring 数据特性。然而，有时我们需要更复杂的逻辑，这是我们无法用自动查询方法创建的。

我们可以在单独的 DAO 类中实现这些查询(就像上一节一样)。

或者，**如果我们想要一个`@Repository`接口有一个自定义实现的方法，我们可以使用[可组合库](/web/20221115111200/https://www.baeldung.com/spring-data-composable-repositories)** 。

自定义界面如下所示:

```java
interface BookRepositoryCustom {
    List<Book> findBooksByAuthorNameAndTitle(String authorName, String title);
}
```

这里是`@Repository`界面:

```java
interface BookRepository extends JpaRepository<Book, Long>, BookRepositoryCustom {}
```

我们还必须修改之前的 DAO 类来实现`BookRepositoryCustom,`，并将其重命名为`BookRepositoryImpl`:

```java
@Repository
class BookRepositoryImpl implements BookRepositoryCustom {

    EntityManager em;

    // constructor

    @Override
    List<Book> findBooksByAuthorNameAndTitle(String authorName, String title) {
        // implementation
    }

}
```

当我们将`BookRepository`声明为依赖项时，Spring 会找到`BookRepositoryImpl`并在我们调用定制方法时使用它。

假设我们想要选择在查询中使用哪些谓词。比如，当我们不想按作者和书名查找书籍时，只需要作者匹配即可。

有多种方法可以做到这一点，比如只有当传递的参数不是`null`时才应用谓词:

```java
@Override
List<Book> findBooksByAuthorNameAndTitle(String authorName, String title) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Book> cq = cb.createQuery(Book.class);

    Root<Book> book = cq.from(Book.class);
    List<Predicate> predicates = new ArrayList<>();

    if (authorName != null) {
        predicates.add(cb.equal(book.get("author"), authorName));
    }
    if (title != null) {
        predicates.add(cb.like(book.get("title"), "%" + title + "%"));
    }
    cq.where(predicates.toArray(new Predicate[0]));

    return em.createQuery(cq).getResultList();
}
```

然而，这种方法**使得代码很难维护**，特别是如果我们有许多谓词，并且希望它们是可选的。

将这些谓词外部化将是一个实用的解决方案。有了 JPA 规范，我们可以做到这一点，甚至更多。

## 5.使用 JPA 规范

Spring Data 引入了`org.springframework.data.jpa.domain.Specification`接口来封装单个谓词:

```java
interface Specification<T> {
    Predicate toPredicate(Root<T> root, CriteriaQuery query, CriteriaBuilder cb);
}
```

我们可以提供创建`Specification`实例的方法:

```java
static Specification<Book> hasAuthor(String author) {
    return (book, cq, cb) -> cb.equal(book.get("author"), author);
}

static Specification<Book> titleContains(String title) {
    return (book, cq, cb) -> cb.like(book.get("title"), "%" + title + "%");
} 
```

为了使用它们，我们需要我们的存储库来扩展`org.springframework.data.jpa.repository.JpaSpecificationExecutor<T>`:

```java
interface BookRepository extends JpaRepository<Book, Long>, JpaSpecificationExecutor<Book> {}
```

这个接口**声明了使用规范**的简便方法。例如，现在我们可以用这个一行程序找到指定作者的所有`Book`实例:

```java
bookRepository.findAll(hasAuthor(author));
```

不幸的是，我们没有任何可以传递多个`Specification`参数的方法。相反，我们在`org.springframework.data.jpa.domain.Specification`接口中获得实用方法。

例如，我们可以用一个逻辑`and`组合两个`Specification`实例:

```java
bookRepository.findAll(where(hasAuthor(author)).and(titleContains(title)));
```

在上面的例子中，`where()`是`Specification`类的静态方法。

这样，我们可以使我们的查询模块化。此外，我们不必编写标准 API 样板文件，因为 Spring 为我们提供了它。

请注意，这并不意味着我们不再需要编写标准样板文件；这种方法只能处理我们看到的工作流，即选择满足所提供条件的实体。

一个查询可以有很多它不支持的结构，包括分组，返回一个不同于我们选择的类，或者子查询。

## 6.结论

在本文中，我们讨论了在 Spring 应用程序中使用条件查询的三种方式:

*   创建一个 DAO 类是最直接、最灵活的方法。
*   扩展一个`@Repository`接口以无缝集成自动查询
*   在`Specification`实例中使用谓词，使简单的情况更加简洁

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221115111200/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-2)