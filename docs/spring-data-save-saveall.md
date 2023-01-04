# Spring 数据中 save()和 saveAll()的性能差异

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-save-saveall>

## 1.概观

在这个快速教程中，我们将了解 Spring 数据中的`save()`和`saveAll()`方法之间的性能差异。

## 2.应用

为了测试性能，我们需要一个带有实体和存储库的 Spring 应用程序。

让我们创建一个图书实体:

```
@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String title;
    private String author;

    // constructors, standard getters and setters
}
```

此外，让我们为它创建一个存储库:

```
public interface BookRepository extends JpaRepository<Book, Long> {
}
```

## 3.表演

为了测试性能，我们将使用这两种方法保存 10，000 本书。

首先，我们将使用 `save()`方法:

```
for(int i = 0; i < bookCount; i++) {
    bookRepository.save(new Book("Book " + i, "Author " + i));
}
```

然后，我们将创建一个图书列表，并使用`saveAll()`方法一次性保存所有图书:

```
List<Book> bookList = new ArrayList<>();
for (int i = 0; i < bookCount; i++) {
    bookList.add(new Book("Book " + i, "Author " + i));
}

bookRepository.saveAll(bookList);
```

在我们的测试中，我们注意到第一种方法花费了大约 2 秒，第二种方法花费了大约 0.3 秒。

此外，当我们启用 JPA 批量插入时，我们观察到`save()`方法的性能降低了 10%,而`saveAll()`方法的性能提高了 60%。

## 4.差异

查看这两个方法的实现，我们可以看到 `saveAll()`遍历每个元素，并在每次迭代中使用`save()`方法。这意味着它不应该有这么大的性能差异。

仔细观察，我们发现这两种方法都用`@Transactional`进行了注释。

此外，默认的事务传播类型是`REQUIRED,` ，这意味着，如果没有提供**，则每次调用**时都会创建一个新的事务。

在我们的例子中，每次我们调用`save()`方法时，都会创建一个新的事务，而当我们调用`saveAll()`时，只创建一个事务，它稍后会被 `save()`重用。

这种开销转化为我们之前注意到的性能差异。

最后，当启用批处理时，开销会更大，因为它是在事务级完成的。

## 5.结论

在本文中，我们已经了解了 Spring 数据中的`save()`和`saveAll()`方法之间的性能差异。

最终，选择是否使用一种方法会对应用程序的性能产生很大的影响。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220617075734/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo-2)