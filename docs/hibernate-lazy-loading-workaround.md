# Hibernate enable _ lazy _ load _ no _ trans 属性快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-lazy-loading-workaround>

## 1.概观

在 Hibernate 中使用延迟加载时，我们可能会遇到异常，比如没有会话。

在本教程中，我们将讨论如何解决这些懒惰的加载问题。为此，我们将使用 Spring Boot 来探索一个例子。

## 2.延迟加载问题

惰性加载的目的是通过在加载主对象时不将相关对象加载到内存中来节省资源。相反，我们将惰性实体的初始化推迟到需要它们的时候。Hibernate 使用代理和集合包装器来实现延迟加载。

当检索延迟加载的数据时，这个过程有两个步骤。首先，填充主对象，其次，检索其代理中的数据。**加载数据总是需要在休眠状态下打开`Session`。**

当第二个步骤发生在事务已经关闭**、**并导致`LazyInitializationException`之后时，问题就出现了。

推荐的方法是设计我们的应用程序以确保数据检索发生在单个事务中。但是，当在代码的另一部分使用惰性实体时，这有时会很困难，因为惰性实体无法确定哪些内容已经加载，哪些内容没有加载。

Hibernate 有一个变通方法，一个`enable_lazy_load_no_trans`属性。打开这个意味着**一个懒惰实体的每次获取将打开一个临时会话**并在一个单独的事务中运行。

## 3.延迟加载示例

让我们看看几种场景下的延迟加载行为。

### 3.1。建立实体和服务

假设我们有两个实体，`User`和`Document`。一个`User`可能有多个`Document`，我们将使用`@OneToMany`来描述这种关系。此外，为了提高效率，我们将使用`@Fetch(FetchMode.SUBSELECT)`。

我们应该注意到，默认情况下，`@OneToMany`有一个惰性获取类型。

现在让我们定义我们的`User`实体:

```java
@Entity
public class User {

    // other fields are omitted for brevity

    @OneToMany(mappedBy = "userId")
    @Fetch(FetchMode.SUBSELECT)
    private List<Document> docs = new ArrayList<>();
}
```

接下来，我们需要一个具有两种方法的服务层来说明不同的选项。其中一个标注为`@Transactional`。这里，两种方法通过计算来自所有用户的所有文档来执行相同的逻辑:

```java
@Service
public class ServiceLayer {

    @Autowired
    private UserRepository userRepository;

    @Transactional(readOnly = true)
    public long countAllDocsTransactional() {
        return countAllDocs();
    }

    public long countAllDocsNonTransactional() {
        return countAllDocs();
    }

    private long countAllDocs() {
        return userRepository.findAll()
            .stream()
            .map(User::getDocs)
            .mapToLong(Collection::size)
            .sum();
    }
}
```

现在，让我们仔细看看下面的三个例子。我们还将使用`SQLStatementCountValidator`来理解解决方案的效率，通过计算执行的查询数量。

### 3.2.周围事务的延迟加载

首先，我们用推荐的方式来使用懒加载。因此，我们将在服务层调用我们的`@Transactional` 方法:

```java
@Test
public void whenCallTransactionalMethodWithPropertyOff_thenTestPass() {
    SQLStatementCountValidator.reset();

    long docsCount = serviceLayer.countAllDocsTransactional();

    assertEquals(EXPECTED_DOCS_COLLECTION_SIZE, docsCount);
    SQLStatementCountValidator.assertSelectCount(2);
}
```

正如我们所看到的，这样做的结果是**到数据库**的两次往返。第一次往返选择用户，第二次选择他们的文档。

### 3.3.事务之外的延迟加载

现在，让我们调用一个非事务性的方法来模拟我们在没有周围事务的情况下得到的错误:

```java
@Test(expected = LazyInitializationException.class)
public void whenCallNonTransactionalMethodWithPropertyOff_thenThrowException() {
    serviceLayer.countAllDocsNonTransactional();
}
```

正如预测的那样，这个**会导致一个错误**，因为`User`的`getDocs`函数是在事务之外使用的。

### 3.4.自动事务延迟加载

要解决这个问题，我们可以启用属性:

```java
spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true
```

**随着属性打开，我们不再得到一个`LazyInitializationException`。**

然而，查询的计数显示对数据库进行了六次往返。这里，一次往返选择用户，五次往返为五个用户中的每一个选择文档:

```java
@Test
public void whenCallNonTransactionalMethodWithPropertyOn_thenGetNplusOne() {
    SQLStatementCountValidator.reset();

    long docsCount = serviceLayer.countAllDocsNonTransactional();

    assertEquals(EXPECTED_DOCS_COLLECTION_SIZE, docsCount);
    SQLStatementCountValidator.assertSelectCount(EXPECTED_USERS_COUNT + 1);
}
```

**我们遇到了臭名昭著的 N + 1 问题**，尽管我们设置了一个获取策略来避免它！

## 4.比较这些方法

下面简单讨论一下利弊。

**有了属性开启，我们就不用担心事务及其边界了。Hibernate 为我们管理这些。**

然而，该解决方案运行缓慢，因为 Hibernate 在每次读取时都会为我们启动一个事务。

当我们不关心性能问题时，它非常适合于演示。如果用于获取仅包含一个元素的集合，或者一对一关系中的单个相关对象，这可能没问题。

**没有了属性，我们可以对事务进行细粒度的控制，**并且我们不再面临性能问题。

**总的来说，这不是一个生产就绪的特性**，Hibernate 文档警告我们:

> 虽然启用这个配置可以让`LazyInitializationException`消失，但是最好使用一个获取计划，保证在会话关闭之前所有属性都被正确初始化。

## 5.结论

在本教程中，我们探讨了如何处理延迟加载。

我们尝试了 Hibernate 属性来帮助克服`LazyInitializationException`。我们也看到了它是如何降低效率的，并且可能只对有限数量的用例是可行的解决方案。

和往常一样，所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220630123235/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-h2)