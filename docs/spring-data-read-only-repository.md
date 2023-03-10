# 用 Spring 数据创建只读存储库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-read-only-repository>

## 1.概观

在这个简短的教程中，**我们将讨论如何创建只读的[弹簧数据](/web/20221006111820/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)**

有时有必要在不修改数据的情况下从数据库中读取数据。在这种情况下，拥有一个只读的`Repository`接口将是完美的。

它将提供读取数据的能力，而没有任何人更改数据的风险。

## 2.延伸`Repository`

让我们从一个包含 [`spring-boot-starter-data-jpa`依赖项](https://web.archive.org/web/20221006111820/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa)的 Spring Boot 项目开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.4.3</version>
</dependency>
```

这个依赖项包括 Spring Data 流行的`CrudRepository`接口，它提供了大多数应用程序需要的所有基本 CRUD 操作(创建、读取、更新、删除)的方法。然而，它包括几个修改数据的方法，我们需要一个只具有读取数据能力的存储库。

`CrudRepository`实际上扩展了另一个叫做`Repository`的接口。我们还可以扩展这个接口来满足我们的需求。

让我们创建一个扩展了`Repository`的新接口:

```java
@NoRepositoryBean
public interface ReadOnlyRepository<T, ID> extends Repository<T, ID> {
    Optional<T> findById(ID id);
    List<T> findAll();
}
```

这里，我们只定义了两个只读方法。由该存储库访问的实体将不会受到任何修改。

同样需要注意的是，我们必须使用`@NoRepositoryBean`注释来告诉 Spring 我们希望这个存储库保持通用。**这允许我们为任意多的不同实体重用我们的只读存储库。**

接下来，我们将看到如何将一个实体绑定到新的`ReadOnlyRepository`。

## 3.延伸`ReadOnlyRepository`

假设我们有一个想要访问的简单的`Book`实体:

```java
@Entity
public class Book {
    @Id
    @GeneratedValue
    private Long id;
    private String author;
    private String title;

    //getters and setters
}
```

现在我们有了一个持久化的实体，我们可以创建一个从我们的`ReadOnlyRepository`继承的存储库接口:

```java
public interface BookReadOnlyRepository extends ReadOnlyRepository<Book, Long> {
    List<Book> findByAuthor(String author);
    List<Book> findByTitle(String title);
}
```

除了它继承的两个方法之外，我们还添加了两个特定于书籍的只读方法:`findByAuthor()`和`findByTitle()`。总的来说，这个存储库可以访问四个只读方法。

最后，让我们编写一个测试来确保我们的`BookReadOnlyRepository`的功能:

```java
@Test
public void givenBooks_whenUsingReadOnlyRepository_thenGetThem() {
    Book aChristmasCarolCharlesDickens = new Book();
    aChristmasCarolCharlesDickens.setTitle("A Christmas Carol");
    aChristmasCarolCharlesDickens.setAuthor("Charles Dickens");
    bookRepository.save(aChristmasCarolCharlesDickens);

    Book greatExpectationsCharlesDickens = new Book();
    greatExpectationsCharlesDickens.setTitle("Great Expectations");
    greatExpectationsCharlesDickens.setAuthor("Charles Dickens");
    bookRepository.save(greatExpectationsCharlesDickens);

    Book greatExpectationsKathyAcker = new Book();
    greatExpectationsKathyAcker.setTitle("Great Expectations");
    greatExpectationsKathyAcker.setAuthor("Kathy Acker");
    bookRepository.save(greatExpectationsKathyAcker);

    List<Book> charlesDickensBooks = bookReadOnlyRepository.findByAuthor("Charles Dickens");
    Assertions.assertEquals(2, charlesDickensBooks.size());

    List<Book> greatExpectationsBooks = bookReadOnlyRepository.findByTitle("Great Expectations");
    Assertions.assertEquals(2, greatExpectationsBooks.size());

    List<Book> allBooks = bookReadOnlyRepository.findAll();
    Assertions.assertEquals(3, allBooks.size());

    Long bookId = allBooks.get(0).getId();
    Book book = bookReadOnlyRepository.findById(bookId).orElseThrow(NoSuchElementException::new);
    Assertions.assertNotNull(book);
}
```

为了在读回之前将书籍保存到数据库中，我们创建了一个在测试范围内扩展了`CrudRepository`的`BookRepository`。这个存储库在主项目范围内是不需要的，但是在这个测试中是必须的。

```java
public interface BookRepository
  extends BookReadOnlyRepository, CrudRepository<Book, Long> {}
```

我们能够测试所有四个只读方法，并且现在可以为其他实体重用`ReadOnlyRepository`接口。

## 4.结论

我们学习了如何扩展 Spring Data 的`Repository`接口，以便创建一个可重用的只读存储库。之后，我们将它绑定到一个简单的`Book`实体上，并编写了一个测试，证明它的功能如我们预期的那样工作。

与往常一样，可以在 GitHub 上找到这个代码的工作示例[。](https://web.archive.org/web/20221006111820/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2)