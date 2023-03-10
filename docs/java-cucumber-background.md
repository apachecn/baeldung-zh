# 黄瓜背景

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cucumber-background>

## 1.概观

在这个简短的教程中，我们将学习 [Cucumber](/web/20220627083314/https://www.baeldung.com/cucumber-scenario-outline) 背景，这是一个允许我们为 Cucumber 特性的每个测试执行一些句子的特性。

## 2.黄瓜背景

先来说明一下[黄瓜背景](https://web.archive.org/web/20220627083314/https://cucumber.io/docs/gherkin/reference/#background)是什么？它的目的是在每次测试一个特性之前执行一个或多个句子。

但是我们在这里试图解决什么问题呢？

假设我们有一个想要用 Cucumber 测试的书店应用程序。首先，让我们创建这个应用程序，它只是一个 Java 类:

```java
public class BookStore {
    private List<Book> books = new ArrayList<>();

    public void addBook(Book book) {
        books.add(book);
    }

    public List<Book> booksByAuthor(String author) {
        return books.stream()
          .filter(book -> Objects.equals(author, book.getAuthor()))
          .collect(Collectors.toList());
    }

    public Optional<Book> bookByTitle(String title) {
        return books.stream()
          .filter(book -> book.getTitle().equals(title))
          .findFirst();
    }
}
```

正如我们所见，可以在商店中添加和搜索书籍。现在，让我们创建几个黄瓜句子来与书店互动:

```java
public class BookStoreRunSteps {
    private BookStore store;
    private List<Book> foundBooks;
    private Book foundBook;

    @Before
    public void setUp() {
        store = new BookStore();
        foundBooks = new ArrayList<>();
    }

    @Given("^I have the following books in the store$")
    public void haveBooksInTheStore(DataTable table) {
        List<List<String>> rows = table.asLists(String.class);

        for (List<String> columns: rows) {
            store.addBook(new Book(columns.get(0), columns.get(1)));
        }
    }

    @When("^I search for books by author (.+)$")
    public void searchForBooksByAuthor(String author) {
        foundBooks = store.booksByAuthor(author);
    }

    @When("^I search for a book titled (.+)$")
    public void searchForBookByTitle(String title) {
        foundBook = store.bookByTitle(title).orElse(null);
    }

    @Then("^I find (\\d+) books$")
    public void findBooks(int count) {
        assertEquals(count, foundBooks.size());
    }

    @Then("^I find a book$")
    public void findABook() {
        assertNotNull(foundBook);
    }

    @Then("^I find no book$")
    public void findNoBook() {
        assertNull(foundBook);
    }
}
```

有了那些句子，我们可以添加书籍，按作者或书名搜索，检查是否找到。

现在，一切都准备好了，我们来创建我们的功能。我们将根据作者以及书名来搜索书籍:

```java
Feature: Book Store Without Background
  Scenario: Find books by author
    Given I have the following books in the store
      | The Devil in the White City          | Erik Larson |
      | The Lion, the Witch and the Wardrobe | C.S. Lewis  |
      | In the Garden of Beasts              | Erik Larson |
    When I search for books by author Erik Larson
    Then I find 2 books

  Scenario: Find books by author, but isn't there
    Given I have the following books in the store
      | The Devil in the White City          | Erik Larson |
      | The Lion, the Witch and the Wardrobe | C.S. Lewis  |
      | In the Garden of Beasts              | Erik Larson |
    When I search for books by author Marcel Proust
    Then I find 0 books

  Scenario: Find book by title
    Given I have the following books in the store
      | The Devil in the White City          | Erik Larson |
      | The Lion, the Witch and the Wardrobe | C.S. Lewis  |
      | In the Garden of Beasts              | Erik Larson |
    When I search for a book titled The Lion, the Witch and the Wardrobe
    Then I find a book

  Scenario: Find book by title, but isn't there
    Given I have the following books in the store
      | The Devil in the White City          | Erik Larson |
      | The Lion, the Witch and the Wardrobe | C.S. Lewis  |
      | In the Garden of Beasts              | Erik Larson |
    When I search for a book titled Swann's Way
    Then I find no book
```

这个特性工作得很好，但是**有点冗长，因为我们为每个测试**初始化存储。这不仅创建了许多行，而且如果我们必须更新存储，我们必须为每个测试都这样做。这时黄瓜背景就派上用场了。

## 3.例子

那么，如何为这个功能创建一个创建商店的背景呢？**要做到这一点，我们必须使用关键字`Background`，像对`Scenario`一样给它一个标题，并定义要执行的句子:**

```java
Background: The Book Store
  Given I have the following books in the store
    | The Devil in the White City          | Erik Larson |
    | The Lion, the Witch and the Wardrobe | C.S. Lewis  |
    | In the Garden of Beasts              | Erik Larson |
```

当我们做到这一点时，我们可以在测试中去掉这句话，让他们专注于他们的特殊性:

```java
Scenario: Find books by author
  When I search for books by author Erik Larson
  Then I find 2 books

Scenario: Find books by author, but isn't there
  When I search for books by author Marcel Proust
  Then I find 0 books

Scenario: Find book by title
  When I search for a book titled The Lion, the Witch and the Wardrobe
  Then I find a book

Scenario: Find book by title, but isn't there
  When I search for a book titled Swann's Way
  Then I find no book
```

正如我们所见，**场景比之前的**要短得多，剩下的句子关注于我们试图测试的东西，而不是设置数据。

## 4.与`@Before`的差异

现在，让我们讨论一下黄瓜背景和[`@Before`钩子](https://web.archive.org/web/20220627083314/https://cucumber.io/docs/cucumber/api/#before)的区别。钩子也允许我们在一个场景之前执行代码，但是这些代码对于那些只阅读特征文件的人来说是隐藏的。另一方面，背景由在特征文件中可见的句子组成。

## 5.结论

在这篇短文中，我们学习了如何使用黄瓜背景功能。它允许我们在一个故事的每个场景之前执行一些句子。我们还讨论了这个特性和`@Before`钩子之间的区别。

像往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627083314/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries)