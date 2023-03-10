# 莫奇托附加答案介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-additionalanswers>

## 1.概观

在本教程中，我们将熟悉 Mockito 的`AdditionalAnswers`类及其方法。

## 2.返回参数

`AdditionalAnswers`类的主要目的是返回传递给模拟方法的参数。

例如，当更新一个对象时，被模仿的方法通常只返回更新后的对象。使用来自`AdditionalAnswers`的方法，我们可以改为**根据它在参数列表**中的位置，返回一个作为参数传递给方法的特定参数。

此外，`AdditionalAnswers `拥有`Answer`类的不同实现。

为了开始我们的演示，让我们创建一个库项目。

首先，我们将创建一个简单的模型:

```java
public class Book {

    private Long bookId;
    private String title;
    private String author;
    private int numberOfPages;

    // constructors, getters and setters

}
```

此外，我们需要一个用于图书检索的存储库类:

```java
public class BookRepository {
    public Book getByBookId(Long bookId) {
        return new Book(bookId, "To Kill a Mocking Bird", "Harper Lee", 256);
    }

    public Book save(Book book) {
        return new Book(book.getBookId(), book.getTitle(), book.getAuthor(), book.getNumberOfPages());
    }

    public Book selectRandomBook(Book bookOne, Book bookTwo, Book bookThree) {
        List<Book> selection = new ArrayList<>();
        selection.add(bookOne);
        selection.add(bookTwo);
        selection.add(bookThree);
        Random random = new Random();
        return selection.get(random.nextInt(selection.size()));
    }
}
```

相应地，我们有一个调用我们的存储库方法的服务类:

```java
public class BookService {
    private final BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public Book getByBookId(Long id) {
        return bookRepository.getByBookId(id);
    }

    public Book save(Book book) {
        return bookRepository.save(book);
    }

    public Book selectRandomBook(Book book1, Book book2, Book book3) {
        return bookRepository.selectRandomBook(book1, book2, book3);
    }
}
```

记住这一点，让我们创建一些测试。

### 2.1.返回第一个参数

对于我们的测试类，我们需要通过**用`MockitoJUnitRunner`** 注释 JUnit 测试类来启用对 Mockito 测试的注释。此外，我们需要**模仿我们的服务和存储库类**:

```java
@RunWith(MockitoJUnitRunner.class)
public class BookServiceUnitTest {
    @InjectMocks
    private BookService bookService;

    @Mock
    private BookRepository bookRepository;

    // test methods

}
```

首先，让我们创建一个返回第一个参数的测试:

```java
@Test
public void givenSaveMethodMocked_whenSaveInvoked_ThenReturnFirstArgument_UnitTest() {
    Book book = new Book("To Kill a Mocking Bird", "Harper Lee", 256);
    Mockito.when(bookRepository.save(any(Book.class))).then(AdditionalAnswers.returnsFirstArg());

    Book savedBook = bookService.save(book);

    assertEquals(savedBook, book);
}
```

换句话说，我们将从接受`Book`对象的`BookRepository`类中模仿`save`方法。

**当我们运行这个测试时，它确实会返回第一个参数**，它等于我们保存的`Book`对象。

### 2.2.返回第二个参数

其次，我们使用`AdditionalAnswers.returnsSecondArg()`创建一个测试:

```java
@Test
public void givenCheckifEqualsMethodMocked_whenCheckifEqualsInvoked_ThenReturnSecondArgument_UnitTest() {
    Book book1 = new Book(1L, "The Stranger", "Albert Camus", 456);
    Book book2 = new Book(2L, "Animal Farm", "George Orwell", 300);
    Book book3 = new Book(3L, "Romeo and Juliet", "William Shakespeare", 200);

    Mockito.when(bookRepository.selectRandomBook(any(Book.class), any(Book.class),
      any(Book.class))).then(AdditionalAnswers.returnsSecondArg());

    Book secondBook = bookService.selectRandomBook(book1, book2, book3);

    assertEquals(secondBook, book2);
}
```

在这种情况下，当我们的 *selectRandomBook* 方法执行时，该方法将返回第二本书。

### 2.3.返回最后一个参数

类似地，我们可以使用`AdditionalAnswers.returnsLastArg()`来获取传递给我们的方法的最后一个参数:

```java
@Test
public void givenCheckifEqualsMethodMocked_whenCheckifEqualsInvoked_ThenReturnLastArgument_UnitTest() {
    Book book1 = new Book(1L, "The Stranger", "Albert Camus", 456);
    Book book2 = new Book(2L, "Animal Farm", "George Orwell", 300);
    Book book3 = new Book(3L, "Romeo and Juliet", "William Shakespeare", 200);

    Mockito.when(bookRepository.selectRandomBook(any(Book.class), any(Book.class), 
      any(Book.class))).then(AdditionalAnswers.returnsLastArg());

    Book lastBook = bookService.selectRandomBook(book1, book2, book3);
    assertEquals(lastBook, book3);
}
```

这里，调用的方法将返回第三本书，因为它是最后一个参数。

### 2.4.返回索引处的参数

最后，让我们编写一个测试，使用方法**使我们能够返回给定索引**–`AdditionalAnswers.returnsArgAt(int index)`处的参数:

```java
@Test
public void givenCheckifEqualsMethodMocked_whenCheckifEqualsInvoked_ThenReturnArgumentAtIndex_UnitTest() {
    Book book1 = new Book(1L, "The Stranger", "Albert Camus", 456);
    Book book2 = new Book(2L, "Animal Farm", "George Orwell", 300);
    Book book3 = new Book(3L, "Romeo and Juliet", "William Shakespeare", 200);

    Mockito.when(bookRepository.selectRandomBook(any(Book.class), any(Book.class), 
      any(Book.class))).then(AdditionalAnswers.returnsArgAt(1));

    Book bookOnIndex = bookService.selectRandomBook(book1, book2, book3);

    assertEquals(bookOnIndex, book2);
}
```

最后，由于我们要求从索引 1 中得到参数，我们将得到第二个参数——具体地说，在本例中是`book2`。

## 3.从功能界面创建答案

`AdditionalAnswers`提供了一种**简洁而整洁的方式来从[功能接口](/web/20220706105622/https://www.baeldung.com/java-8-functional-interfaces)** 中创建答案。为此，它提供了两个方便的方法:`answer() and` `answerVoid().`

所以，让我们下兔子洞，看看如何在实践中使用它们。

请记住，**这两种方法都用`@Incubating`** 标注。这意味着它们可能会在以后根据社区反馈进行更改。

### 3.1.使用`AdditionalAnswers.answer()`

引入这个方法主要是为了在 Java 8 中使用函数接口创建强类型答案。

通常，Mockito 带有一组现成的通用接口，我们可以用它们来配置 mock 的答案。**例如，它为单个参数调用**提供了`Answer1<T,A0>`。

现在，让我们演示如何使用`AdditionalAnswers.answer()` 来创建一个返回`Book`对象的答案:

```java
@Test
public void givenMockedMethod_whenMethodInvoked_thenReturnBook() {
    Long id = 1L;
    when(bookRepository.getByBookId(anyLong())).thenAnswer(answer(BookServiceUnitTest::buildBook));

    assertNotNull(bookService.getByBookId(id));
    assertEquals("The Stranger", bookService.getByBookId(id).getTitle());
}

private static Book buildBook(Long bookId) {
    return new Book(bookId, "The Stranger", "Albert Camus", 456);
}
```

如上所示，我们使用了一个[方法引用](/web/20220706105622/https://www.baeldung.com/java-method-references)来表示`Answer1` 接口。

### 3.2.使用`AdditionalAnswers.answerVoid()`

类似地，我们可以使用`answerVoid()`为不返回任何内容的参数调用配置 mock 的答案。

接下来，让我们用一个测试用例来举例说明`AdditionalAnswers.answerVoid()` 方法的使用:

```java
@Test
public void givenMockedMethod_whenMethodInvoked_thenReturnVoid() {
    Long id = 2L;
    when(bookRepository.getByBookId(anyLong())).thenAnswer(answerVoid(BookServiceUnitTest::printBookId));
    bookService.getByBookId(id);

    verify(bookRepository, times(1)).getByBookId(id);
}

private static void printBookId(Long bookId) {
    System.out.println(bookId);
}
```

如我们所见，**我们使用了`VoidAnswer1<A0>`** **接口** **到** **为不返回任何内容的单个参数调用创建答案**。

`answer`方法指定了当我们与 mock 交互时执行的动作。在我们的例子中，我们只是打印传递的图书的 id。

## 4.结论

总之，本教程涵盖了 Mockito 的`AdditionalAnswers`类的方法。

GitHub 上的[提供了这些例子和代码片段的实现。](https://web.archive.org/web/20220706105622/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-2)