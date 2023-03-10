# 在 Spring 数据仓库上测试@Cacheable

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-testing-cacheable>

## 1.概观

除了实现之外，我们可以**使用 [Spring 的声明式缓存机制](/web/20221128052619/https://www.baeldung.com/spring-cache-tutorial)来注释接口**。例如，我们可以在 Spring 数据存储库上声明缓存。

在本教程中，我们将展示如何测试这样一个场景。

## 2.入门指南

首先，让我们创建一个简单的模型:

```java
@Entity
public class Book {

    @Id
    private UUID id;
    private String title;

}
```

然后，让我们添加一个具有`@Cacheable`方法的存储库接口:

```java
public interface BookRepository extends CrudRepository<Book, UUID> {

    @Cacheable(value = "books", unless = "#a0=='Foundation'")
    Optional<Book> findFirstByTitle(String title);

}
```

这里的`unless` 条件不是强制的。它将帮助我们稍后测试一些缓存未命中场景。

另外，注意 SpEL 表达式`“#a0”` 而不是可读性更强的`“#title”`。我们这样做是因为代理不会保留参数名。所以，我们使用另一种`#root.arg[0], p0 or a0`符号。

## 3.测试

我们测试的目标是**确保缓存机制正常工作。**因此，我们**不打算涵盖 Spring 数据仓库实现**或持久性方面。

### 3.1.Spring Boot

让我们从一个简单的 Spring Boot 测试开始。

首先，我们将设置我们的测试依赖项，添加一些测试数据，并创建一个简单的实用方法来检查一本书是否在缓存中:

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = CacheApplication.class)
public class BookRepositoryIntegrationTest {

    @Autowired
    CacheManager cacheManager;

    @Autowired
    BookRepository repository;

    @BeforeEach
    void setUp() {
        repository.save(new Book(UUID.randomUUID(), "Dune"));
        repository.save(new Book(UUID.randomUUID(), "Foundation"));
    }

    private Optional<Book> getCachedBook(String title) {
        return ofNullable(cacheManager.getCache("books")).map(c -> c.get(title, Book.class));
    }
```

现在，让我们确保**在请求一本书之后，它被放入缓存**:

```java
 @Test
    void givenBookThatShouldBeCached_whenFindByTitle_thenResultShouldBePutInCache() {
        Optional<Book> dune = repository.findFirstByTitle("Dune");

        assertEquals(dune, getCachedBook("Dune"));
    }
```

还有，**一些书没有放在缓存中**:

```java
 @Test
    void givenBookThatShouldNotBeCached_whenFindByTitle_thenResultShouldNotBePutInCache() {
        repository.findFirstByTitle("Foundation");

        assertEquals(empty(), getCachedBook("Foundation"));
    }
```

在这个测试中，我们利用弹簧提供的`CacheManager`并检查**在每个`repository.findFirstByTitle` 操作后，根据`@Cacheable`规则`CacheManager`包含(或不包含)书籍**。

### 3.2.平原泉

现在让我们继续一个 Spring 集成测试。为了改变，这次让我们模仿我们的界面。然后我们将在不同的测试案例中验证与它的交互。

我们将从创建一个为我们的`BookRepository`提供`mock`实现的`@Configuration` 开始:

```java
@ContextConfiguration
@ExtendWith(SpringExtension.class)
public class BookRepositoryCachingIntegrationTest {

    private static final Book DUNE = new Book(UUID.randomUUID(), "Dune");
    private static final Book FOUNDATION = new Book(UUID.randomUUID(), "Foundation");

    private BookRepository mock;

    @Autowired
    private BookRepository bookRepository;

    @EnableCaching
    @Configuration
    public static class CachingTestConfig {

        @Bean
        public BookRepository bookRepositoryMockImplementation() {
            return mock(BookRepository.class);
        }

        @Bean
        public CacheManager cacheManager() {
            return new ConcurrentMapCacheManager("books");
        }

    } 
```

在继续设置我们的模拟行为之前，关于在这个上下文中成功使用`Mockito`有两个方面值得一提:

*   `BookRepository`是**在我们模拟周围的代理。**因此，为了使用`Mockito`验证，我们通过`AopTestUtils.getTargetObject`检索实际的模拟
*   我们**确保在测试之间`reset(mock)`** ，因为`CachingTestConfig`只加载一次

```java
 @BeforeEach
    void setUp() {
        mock = AopTestUtils.getTargetObject(bookRepository);

        reset(mock);

        when(mock.findFirstByTitle(eq("Foundation")))
                .thenReturn(of(FOUNDATION));

        when(mock.findFirstByTitle(eq("Dune")))
                .thenReturn(of(DUNE))
                .thenThrow(new RuntimeException("Book should be cached!"));
    }
```

现在，我们可以添加我们的测试方法。首先，我们将确保在将一本书放入缓存后，当稍后尝试检索该书时，不会再与存储库实现进行交互:

```java
 @Test
    void givenCachedBook_whenFindByTitle_thenRepositoryShouldNotBeHit() {
        assertEquals(of(DUNE), bookRepository.findFirstByTitle("Dune"));
        verify(mock).findFirstByTitle("Dune");

        assertEquals(of(DUNE), bookRepository.findFirstByTitle("Dune"));
        assertEquals(of(DUNE), bookRepository.findFirstByTitle("Dune"));

        verifyNoMoreInteractions(mock);
    }
```

我们还想检查**对于未缓存的书籍，我们每次都调用存储库**:

```java
 @Test
    void givenNotCachedBook_whenFindByTitle_thenRepositoryShouldBeHit() {
        assertEquals(of(FOUNDATION), bookRepository.findFirstByTitle("Foundation"));
        assertEquals(of(FOUNDATION), bookRepository.findFirstByTitle("Foundation"));
        assertEquals(of(FOUNDATION), bookRepository.findFirstByTitle("Foundation"));

        verify(mock, times(3)).findFirstByTitle("Foundation");
    }
```

## 4.摘要

总而言之，我们使用 Spring、Mockito 和 Spring Boot 实现了一系列集成测试，以确保应用于我们接口的缓存机制正常工作。

请注意，我们也可以组合上述方法。例如，没有什么可以阻止我们使用 Spring Boot 的模拟，或者在简单的 Spring 测试中对`CacheManager`进行检查。

完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221128052619/https://github.com/eugenp/tutorials/tree/master/spring-caching)