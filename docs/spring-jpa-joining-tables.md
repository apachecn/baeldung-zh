# 用 Spring 数据连接表 JPA 规范

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jpa-joining-tables>

## 1.概观

在这个简短的教程中，我们将讨论 [Spring Data JPA](/web/20220707143816/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 规范的一个更高级的特性，它允许我们在创建查询时连接表。

让我们先简要回顾一下 JPA 规范的用法。

## 2.JPA 规范

**Spring Data JPA 引入了`Specification `接口，允许我们用可重用的组件创建动态查询。**

对于本文中的代码示例，我们将使用`Author`和`Book`类:

```java
@Entity
public class Author {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstName;

    private String lastName;

    @OneToMany(cascade = CascadeType.ALL)
    private List<Book> books;

    // getters and setters
}
```

为了给`Author` 实体创建一个动态查询，我们可以使用`Specification`接口的实现:

```java
public class AuthorSpecifications {

    public static Specification<Author> hasFirstNameLike(String name) {
        return (root, query, criteriaBuilder) ->
          criteriaBuilder.like(root.<String>get("firstName"), "%" + name + "%");
    }

    public static Specification<Author> hasLastName(String name) {
        return (root, query, cb) ->
          cb.equal(root.<String>get("lastName"), name);
    }
}
```

最后，我们需要用`AuthorRepository`来扩展`JpaSpecificationExecutor`:

```java
@Repository
public interface AuthorsRepository extends JpaRepository<Author, Long>, JpaSpecificationExecutor<Author> {
}
```

因此，我们现在可以将这两个规范链接在一起，并用它们创建查询:

```java
@Test
public void whenFindByLastNameAndFirstNameLike_thenOneAuthorIsReturned() {

    Specification<Author> specification = hasLastName("Martin")
      .and(hasFirstNameLike("Robert"));

    List<Author> authors = repository.findAll(specification);

    assertThat(authors).hasSize(1);
}
```

## 3.使用 JPA 规范连接表

我们可以从我们的数据模型中观察到，`Author`实体与`Book`实体共享一对多关系:

```java
@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    // getters and setters
}
```

[标准查询 API](/web/20220707143816/https://www.baeldung.com/spring-data-criteria-queries) 允许我们在创建`Specification` `.` 时连接两个表。因此，我们将能够在查询中包含来自`Book`实体的字段:

```java
public static Specification<Author> hasBookWithTitle(String bookTitle) {
    return (root, query, criteriaBuilder) -> {
        Join<Book, Author> authorsBook = root.join("books");
        return criteriaBuilder.equal(authorsBook.get("title"), bookTitle);
    };
}
```

现在，让我们将这个新规范与以前创建的规范结合起来:

```java
@Test
public void whenSearchingByBookTitleAndAuthorName_thenOneAuthorIsReturned() {

    Specification<Author> specification = hasLastName("Martin")
      .and(hasBookWithTitle("Clean Code"));

    List<Author> authors = repository.findAll(specification);

    assertThat(authors).hasSize(1);
}
```

最后，让我们看一下生成的 SQL，看看`JOIN`子句:

```java
select 
  author0_.id as id1_1_, 
  author0_.first_name as first_na2_1_, 
  author0_.last_name as last_nam3_1_ 
from 
  author author0_ 
  inner join author_books books1_ on author0_.id = books1_.author_id 
  inner join book book2_ on books1_.books_id = book2_.id 
where 
  author0_.last_name = ? 
  and book2_.title = ?
```

## 4.结论

在本文中，我们已经了解了如何使用 JPA 规范来查询基于相关实体之一的表。

Spring Data JPA 的规范带来了一种流畅、动态和可重用的查询创建方式。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220707143816/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-3)