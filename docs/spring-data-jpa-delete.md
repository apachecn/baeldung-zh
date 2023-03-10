# Spring 数据 JPA 删除和关系

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-delete>

## 1.概观

在本教程中，我们将看看如何在 [Spring Data JPA](https://web.archive.org/web/20220727020746/https://spring.io/projects/spring-data-jpa) 中删除。

## 2.样本实体

从 Spring Data JPA 参考文档中我们知道，存储库接口为我们提供了一些基本的实体支持。

假设我们有一个实体，比如一个`Book`:

```java
@Entity
public class Book {

    @Id
    @GeneratedValue
    private Long id;
    private String title;

    // standard constructors

    // standard getters and setters
}
```

然后我们可以扩展 Spring Data JPA 的`CrudRepository` 来访问`Book`上的 CRUD 操作:

```java
@Repository
public interface BookRepository extends CrudRepository<Book, Long> {}
```

## 3.从存储库中删除

其中，`CrudRepository`包含两个方法:`deleteById`和`deleteAll`。

让我们直接从我们的`BookRepository`开始测试这些方法:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {Application.class})
public class DeleteFromRepositoryUnitTest {

    @Autowired
    private BookRepository repository;

    Book book1;
    Book book2;
    List<Book> books;

    // data initialization

    @Test
    public void whenDeleteByIdFromRepository_thenDeletingShouldBeSuccessful() {
        repository.deleteById(book1.getId());
        assertThat(repository.count()).isEqualTo(1);
    }

    @Test
    public void whenDeleteAllFromRepository_thenRepositoryShouldBeEmpty() {
        repository.deleteAll();
        assertThat(repository.count()).isEqualTo(0);
    }
}
```

尽管我们使用的是`CrudRepository`，但是请注意，这些相同的方法也存在于其他 Spring 数据 JPA 接口，比如`JpaRepository`或`PagingAndSortingRepository`。

## 4.派生删除查询

我们还可以派生出删除实体的查询方法。写它们有一套规则，但让我们只关注最简单的例子。

**派生的删除查询必须以`deleteBy`开头，后跟选择标准的名称。**这些标准必须在方法调用中提供。

假设我们想通过`title`删除`Book` s。使用命名约定，我们将从`deleteBy` 开始，并将`title`列为我们的标准:

```java
@Repository
public interface BookRepository extends CrudRepository<Book, Long> {
    long deleteByTitle(String title);
}
```

类型为`long`的返回值表明该方法删除了多少条记录。

让我们写一个测试并确保它是正确的:

```java
@Test
@Transactional
public void whenDeleteFromDerivedQuery_thenDeletingShouldBeSuccessful() {
    long deletedRecords = repository.deleteByTitle("The Hobbit");
    assertThat(deletedRecords).isEqualTo(1);
}
```

**在 JPA 中持久化和删除对象需要一个事务。这就是为什么我们应该在使用这些[派生的删除查询](/web/20220727020746/https://www.baeldung.com/spring-data-jpa-deleteby)时使用`@Transactional`注释，以确保事务正在运行。**这在[带 Spring 的 ORM 文档](https://web.archive.org/web/20220727020746/https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#orm)中有详细解释。

## 5.自定义删除查询

派生查询的方法名可能会很长，并且仅限于一个表。

当我们需要更复杂的东西时，我们可以一起使用`@Query` 和`[@Modifying](/web/20220727020746/https://www.baeldung.com/spring-data-jpa-modifying-annotation)`来编写定制查询。

让我们检查前面派生方法的等价代码:

```java
@Modifying
@Query("delete from Book b where b.title=:title")
void deleteBooks(@Param("title") String title);
```

同样，我们可以通过一个简单的测试来验证它的有效性:

```java
@Test
@Transactional
public void whenDeleteFromCustomQuery_thenDeletingShouldBeSuccessful() {
    repository.deleteBooks("The Hobbit");
    assertThat(repository.count()).isEqualTo(1);
}
```

上面给出的两种解决方案是相似的，并且实现了相同的结果。然而，他们采取了稍微不同的方法。

**`@Query`方法创建一个针对数据库的 JPQL 查询。相比之下，`deleteBy`方法执行一个读查询，然后逐个删除每个条目。**

## 6.在关系中删除

现在让我们看看当我们与其他实体有**关系时会发生什么。**

假设我们有一个与`Book`实体有`OneToMany`关联的`Category`实体:

```java
@Entity
public class Category {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Book> books;

    // standard constructors

    // standard getters and setters
}
```

`CategoryRepository`可以只是一个扩展`CrudRepository`的空接口:

```java
@Repository
public interface CategoryRepository extends CrudRepository<Category, Long> {}
```

我们还应该修改`Book`实体来反映这种关联:

```java
@ManyToOne
private Category category;
```

现在让我们添加两个类别，并将它们与我们当前拥有的书籍相关联。

**现在，如果我们尝试删除类别，书籍也会被删除**:

```java
@Test
public void whenDeletingCategories_thenBooksShouldAlsoBeDeleted() {
    categoryRepository.deleteAll();
    assertThat(bookRepository.count()).isEqualTo(0);
    assertThat(categoryRepository.count()).isEqualTo(0);
}
```

**这不是双向的，尽管**，这意味着如果我们删除书籍，类别仍然存在:

```java
@Test
public void whenDeletingBooks_thenCategoriesShouldAlsoBeDeleted() {
    bookRepository.deleteAll();
    assertThat(bookRepository.count()).isEqualTo(0);
    assertThat(categoryRepository.count()).isEqualTo(2);
}
```

我们可以通过改变关系的属性来改变这种行为，比如 [`CascadeType`](/web/20220727020746/https://www.baeldung.com/delete-with-hibernate) 。

## 7.结论

在本文中，我们看到了在 Spring Data JPA 中删除实体的不同方法。

我们查看了从`CrudRepository`提供的删除方法，以及使用`@Query`注释的派生查询或自定义查询。

我们也看到了删除是如何在关系中完成的。

和往常一样，本文中提到的所有代码片段都可以在我们的 GitHub 库中找到。