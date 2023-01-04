# 使用 Spring 数据 JPA 按日期和时间查询实体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-query-by-date>

## 1.介绍

在这个快速教程中，我们将看到如何使用 Spring 数据 JPA 按日期查询实体。

首先，我们将回顾一下如何用 JPA 映射日期和时间。然后，我们将创建一个带有日期和时间字段的实体，以及一个 Spring 数据存储库来查询这些实体。

## 2.用 JPA 映射日期和时间

首先，**我们将回顾一些关于用 JPA** 映射日期的理论。需要知道的是，我们需要决定是否要代表:

*   只有约会
*   只有一次
*   或者两者都有

除了(可选的)`@Column`注释之外，我们还需要添加`@Temporal`注释来指定字段代表什么。

这个注释接受一个值为`TemporalType enum:`的参数

*   `TemporalType.DATE`
*   `TemporalType.TIME`
*   `TemporalType.TIMESTAMP`

关于用 JPA 进行日期和时间映射的详细文章可以在[这里](/web/20220628234946/https://www.baeldung.com/hibernate-date-time)找到。

## 3.在实践中

在实践中，一旦我们的实体被正确设置，使用 Spring Data JPA 查询它们就没有太多工作要做了。我们只需使用`query methods,`或 `@Query` 注释。

**每个春季的数据 JPA 机制都会正常工作**。

让我们看几个用 Spring Data JPA 按日期和时间查询实体的例子。

### 3.1.建立一个实体

例如，假设我们有一个`Article`实体，它有发布日期、发布时间以及创建日期和时间:

```java
@Entity
public class Article {

    @Id
    @GeneratedValue
    Integer id;

    @Temporal(TemporalType.DATE)
    Date publicationDate;

    @Temporal(TemporalType.TIME)
    Date publicationTime;

    @Temporal(TemporalType.TIMESTAMP)
    Date creationDateTime;
}
```

出于演示目的，我们将发布日期和时间分成两个字段；这样我们就代表了三种时间类型。

### 3.2.查询实体

现在我们的实体已经设置好了，让我们创建一个 Spring 数据 `repository`来查询那些文章。

我们将使用几个 Spring Data JPA 特性创建三个方法:

```java
public interface ArticleRepository 
  extends JpaRepository<Article, Integer> {

    List<Article> findAllByPublicationDate(Date publicationDate);

    List<Article> findAllByPublicationTimeBetween(
      Date publicationTimeStart,
      Date publicationTimeEnd);

    @Query("select a from Article a where a.creationDateTime <= :creationDateTime")
    List<Article> findAllWithCreationDateTimeBefore(
      @Param("creationDateTime") Date creationDateTime);

}
```

所以我们定义了三种方法:

*   `findAllByPublicationDate`检索在给定日期发表的文章
*   `findAllByPublicationTimeBetween`检索两个小时内发表的文章
*   而`findAllWithCreationDateTimeBefore`检索在给定日期和时间之前创建的文章

前两种方法依赖于 Spring 数据的`query methods`机制，最后一种依赖于`@Query`注释。

最终，这不会改变约会被对待的方式。第一种方法将只考虑参数的日期部分。

第二个将只考虑参数的时间。最后一个将使用日期和时间。

### 3.3.测试查询

我们要做的最后一件事是设置一些测试来检查这些查询是否按预期工作。

我们将首先将数据导入到我们的数据库中，然后我们将创建测试类来检查存储库的每个方法:

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class ArticleRepositoryIntegrationTest {

    @Autowired
    private ArticleRepository repository;

    @Test
    public void whenFindByPublicationDate_thenArticles1And2Returned() {
        List<Article> result = repository.findAllByPublicationDate(
          new SimpleDateFormat("yyyy-MM-dd").parse("2018-01-01"));

        assertEquals(2, result.size());
        assertTrue(result.stream()
          .map(Article::getId)
          .allMatch(id -> Arrays.asList(1, 2).contains(id)));
    }

    @Test
    public void whenFindByPublicationTimeBetween_thenArticles2And3Returned() {
        List<Article> result = repository.findAllByPublicationTimeBetween(
          new SimpleDateFormat("HH:mm").parse("15:15"),
          new SimpleDateFormat("HH:mm").parse("16:30"));

        assertEquals(2, result.size());
        assertTrue(result.stream()
          .map(Article::getId)
          .allMatch(id -> Arrays.asList(2, 3).contains(id)));
    }

    @Test
    public void givenArticlesWhenFindWithCreationDateThenArticles2And3Returned() {
        List<Article> result = repository.findAllWithCreationDateTimeBefore(
          new SimpleDateFormat("yyyy-MM-dd HH:mm").parse("2017-12-15 10:00"));

        assertEquals(2, result.size());
        assertTrue(result.stream()
          .map(Article::getId)
          .allMatch(id -> Arrays.asList(2, 3).contains(id));
    }
}
```

每个测试都验证只检索符合条件的文章。

## 4.结论

在这篇简短的文章中，我们学习了如何使用 Spring Data JPA 通过日期和时间字段查询实体。

在使用 Spring 数据机制查询实体之前，我们讨论了一些理论。我们看到这些机制处理日期和时间的方式与处理其他类型数据的方式相同。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628234946/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-3)