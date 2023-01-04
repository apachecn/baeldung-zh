# Morphia 简介 MongoDB 的 Java ODM

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-morphia>

## 1.概观

在本教程中，我们将了解如何使用 Java 中 MongoDB 的对象文档映射器(ODM)[Morphia](https://web.archive.org/web/20221225164858/https://morphia.dev/)。

在这个过程中，我们还将了解什么是 ODM，以及它如何促进 MongoDB 的使用。

## 2.什么是`ODM`？

对于那些不熟悉这个领域的人来说， **[MongoDB](https://web.archive.org/web/20221225164858/https://www.mongodb.com/) 是一个面向文档的数据库，是为自然分布而构建的**。简单地说，面向文档的数据库管理文档，这只不过是一种组织半结构化数据的无模式方式**。它们属于 NoSQL 数据库的一个范围更广、定义更松散的大伞下，以它们明显偏离 SQL 数据库的传统组织而命名。**

MongoDB 为 Java 等几乎所有流行的编程语言提供了 [**驱动。这些驱动程序为使用 MongoDB 提供了一个抽象层，因此我们不会直接使用有线协议。可以认为这是 Oracle 为他们的关系数据库提供了 JDBC 驱动程序的实现。**](/web/20221225164858/https://www.baeldung.com/java-mongodb)

然而，如果我们直接回忆与 JDBC 一起工作的日子，我们会意识到它会变得多么混乱——尤其是在面向对象的范例中。幸运的是，我们有 Hibernate 这样的对象关系映射(ORM)框架来拯救我们。对于 MongoDB 来说没有太大的不同。

虽然我们当然可以使用底层驱动程序，但要完成这项任务，需要更多的样板文件。这里，我们有了一个与 ORM 类似的概念，叫做对象文档映射器(ODM) 。Morphia 完全填补了 Java 编程语言的空白，并在 MongoDB 的 Java 驱动程序之上工作。

## 3.设置相关性

我们已经看到足够的理论让我们进入一些代码。对于我们的例子，我们将对一个图书馆建模，并看看如何使用 Morphia 在 MongoDB 中管理它。

但是在开始之前，我们需要设置一些依赖项。

### 3.1.MongoDB

我们需要一个 MongoDB 的运行实例。有几种方法可以实现这一点，最简单的是在我们的本地机器上下载并安装社区版。

我们应该保留所有默认配置，包括运行 MongoDB 的端口。

### 3.2.吗啡

我们可以从 [Maven Central](https://web.archive.org/web/20221225164858/https://search.maven.org/search?q=g:dev.morphia.morphia%20AND%20a:core) 下载 Morphia 的预构建 jar，并在我们的 Java 项目中使用它们。

然而，最简单的方法是使用像 Maven 这样的依赖管理工具:

```java
<dependency>
    <groupId>dev.morphia.morphia</groupId>
    <artifactId>core</artifactId>
    <version>1.5.3</version>
</dependency>
```

## 4.如何使用 Morphia 连接？

现在我们已经安装并运行了 MongoDB，并在 Java 项目中设置了 Morphia，我们准备使用 Morphia 连接到 MongoDB。

让我们看看如何实现这一点:

```java
Morphia morphia = new Morphia();
morphia.mapPackage("com.baeldung.morphia");
Datastore datastore = morphia.createDatastore(new MongoClient(), "library");
datastore.ensureIndexes();
```

差不多就是这样！让我们更好地理解这一点。我们的映射操作需要两件事情:

1.  映射器:它负责将我们的 Java POJOs 映射到 MongoDB 集合。在上面的代码片段中，`Morphia`是负责这个的类。注意我们是如何配置包的，它应该在哪里寻找我们的 POJOs。
2.  连接:这是到 MongoDB 数据库的连接，映射器可以在这个数据库上执行不同的操作。类`Datastore` 将`MongoClient`(来自 Java MongoDB 驱动程序)的实例和 MongoDB 数据库的名称**作为参数，返回一个活动连接以与**一起工作。

所以，我们都准备好使用这个`Datastore`并与我们的实体一起工作。

## 5.如何处理实体？

在我们可以使用新创建的`Datastore`之前，我们需要定义一些要使用的域实体。

### 5.1.简单实体

让我们首先定义一个简单的带有一些属性的`Book`实体:

```java
@Entity("Books")
public class Book {
    @Id
    private String isbn;
    private String title;
    private String author;
    @Property("price")
    private double cost;
    // constructors, getters, setters and hashCode, equals, toString implementations
}
```

这里有几件有趣的事情需要注意:

*   请注意注释 **@ `Entity`，它通过 Morphia 将这个 POJO 限定为 ODM 映射**
*   默认情况下，Morphia 通过类的名称将一个实体映射到 MongoDB 中的一个集合，但是我们可以显式地覆盖它(就像我们在这里对实体`Book`所做的那样)
*   默认情况下，Morphia 通过变量的名称将实体中的变量映射到 MongoDB 集合中的键，但是我们也可以覆盖这一点(就像我们在这里对变量`cost`所做的那样)
*   最后，我们需要通过注释@ `Id` 在实体中**标记一个变量作为主键(就像我们在这里使用 ISBN)**

### 5.2.有关系的实体

然而，在现实世界中，实体很难像它们看起来那样简单，它们之间有着复杂的关系。例如，我们的简单实体`Book`可以有一个`Publisher`，并且可以引用其他配套书籍。我们如何为它们建模？

MongoDB 提供了两种机制来建立关系——引用和嵌入。顾名思义，通过引用，MongoDB 将相关数据作为单独的文档存储在相同或不同的集合中，并使用其 id 引用它。

相反，通过嵌入，MongoDB 将关系存储或嵌入到父文档本身中。

让我们看看如何使用它们。让我们从在我们的`Book`中嵌入`Publisher` 开始:

```java
@Embedded
private Publisher publisher;
```

很简单。现在让我们继续添加对其他书籍的引用:

```java
@Reference
private List<Book> companionBooks;
```

就是这样——Morphia 为 MongoDB 支持的模型关系提供了方便的注释。然而，选择**引用还是嵌入，应该从数据模型的复杂性、冗余性和一致性**以及其他考虑因素中得出。

这个练习类似于关系数据库中的规范化。

现在，我们准备使用`Datastore`对`Book`执行一些操作。

## 6.一些基本操作

让我们看看如何使用 Morphia 进行一些基本操作。

### 6.1.救援

让我们从最简单的操作开始，在我们的 MongoDB 数据库`library`中创建一个`Book`实例:

```java
Publisher publisher = new Publisher(new ObjectId(), "Awsome Publisher");

Book book = new Book("9781565927186", "Learning Java", "Tom Kirkman", 3.95, publisher);
Book companionBook = new Book("9789332575103", "Java Performance Companion", 
  "Tom Kirkman", 1.95, publisher);

book.addCompanionBooks(companionBook);

datastore.save(companionBook);
datastore.save(book);
```

这足以让 Morphia 在我们的 MongoDB 数据库中创建一个集合，如果它不存在的话，并执行一个 upsert 操作。

### 6.2.询问

让我们看看能否在 MongoDB 中查询我们刚刚创建的图书:

```java
List<Book> books = datastore.createQuery(Book.class)
  .field("title")
  .contains("Learning Java")
  .find()
  .toList();

assertEquals(1, books.size());

assertEquals(book, books.get(0));
```

在 Morphia 中查询文档首先使用`Datastore`创建一个查询，然后声明性地添加过滤器，这让喜欢函数式编程的人很高兴！

Morphia 用过滤器和操作符支持更多复杂的查询构造。此外，Morphia 允许限制、跳过和排序查询中的结果。

此外，Morphia 允许我们使用用 MongoDB 的 Java 驱动程序编写的原始查询来获得更多的控制，如果需要的话。

### 6.3.更新

虽然如果主键匹配，保存操作可以处理更新，但是 Morphia 提供了有选择地更新文档的方法:

```java
Query<Book> query = datastore.createQuery(Book.class)
  .field("title")
  .contains("Learning Java");

UpdateOperations<Book> updates = datastore.createUpdateOperations(Book.class)
  .inc("price", 1);

datastore.update(query, updates);

List<Book> books = datastore.createQuery(Book.class)
  .field("title")
  .contains("Learning Java")
  .find()
  .toList();

assertEquals(4.95, books.get(0).getCost());
```

这里，我们构建一个查询和一个更新操作，将查询返回的所有书籍的价格增加一。

### 6.4.删除

最后，已经创建的必须删除！同样，Morphia 非常直观:

```java
Query<Book> query = datastore.createQuery(Book.class)
  .field("title")
  .contains("Learning Java");

datastore.delete(query);

List<Book> books = datastore.createQuery(Book.class)
  .field("title")
  .contains("Learning Java")
  .find()
  .toList();

assertEquals(0, books.size());
```

我们像前面一样创建查询，并在`Datastore`上运行删除操作。

## 7.高级用法

MongoDB 有一些高级操作，如聚合、索引和许多其他操作。虽然不可能使用 Morphia 完成所有这些，但是肯定可以实现其中的一些。遗憾的是，对于其他人，我们将不得不求助于 MongoDB 的 Java 驱动程序。

让我们关注一些我们可以通过 Morphia 执行的高级操作。

### 7.1.聚合

MongoDB 中的聚合允许我们在管道中定义一系列操作**，这些操作可以对一组文档进行操作并产生聚合输出**。

Morphia 有一个 API 来支持这样的聚合管道。

让我们假设我们希望以这样一种方式聚集我们的图书馆数据，即我们将所有的书按它们的作者分组:

```java
Iterator<Author> iterator = datastore.createAggregation(Book.class)
  .group("author", grouping("books", push("title")))
  .out(Author.class);
```

那么，这是如何工作的呢？我们首先使用相同的旧`Datastore`创建一个聚合管道。我们必须提供希望对其执行聚合操作的实体，例如这里的`Book`。

接下来，我们希望按照“作者”对文档进行分组，并将它们的“标题”聚合到一个名为“书籍”的关键字下。最后，我们在这里与一个 ODM 合作。因此，我们必须定义一个实体来收集我们的聚合数据—在我们的例子中，它是`Author`。

当然，我们必须用一个名为 books 的变量定义一个名为`Author`的实体:

```java
@Entity
public class Author {
    @Id
    private String name;
    private List<String> books;
    // other necessary getters and setters
}
```

当然，这只是触及了 MongoDB 提供的一个非常强大的构造的表面，可以进一步探索细节。

### 7.2.推断

MongoDB 中的 Projection 允许我们在查询中只选择我们想要从文档中获取的字段。如果文档结构很复杂，当我们只需要几个字段时，这真的很有用。

假设我们只需要在查询中获取带有书名的书籍:

```java
List<Book> books = datastore.createQuery(Book.class)
  .field("title")
  .contains("Learning Java")
  .project("title", true)
  .find()
  .toList();

assertEquals("Learning Java", books.get(0).getTitle());
assertNull(books.get(0).getAuthor());
```

在这里，正如我们所看到的，我们只在我们的结果中得到标题，而不是作者和其他字段。然而，我们应该小心地使用预计的输出来保存回 MongoDB。这可能会导致数据丢失！

### 7.3.索引

索引在数据库的查询优化中起着非常重要的作用，无论是关系数据库还是许多非关系数据库。

MongoDB **在集合级别定义索引，默认情况下在主键上创建一个惟一索引**。此外，MongoDB 允许在文档中的任何字段或子字段上创建索引。我们应该根据我们想要创建的查询来选择在一个键上创建索引。

例如，在我们的例子中，我们可能希望在字段`Book`的“title”上创建一个索引，因为我们经常最终查询它:

```java
@Indexes({
  @Index(
    fields = @Field("title"),
    options = @IndexOptions(name = "book_title")
  )
})
public class Book {
    // ...
    @Property
    private String title;
    // ...
}
```

当然，我们可以传递额外的索引选项来定制创建的索引的细微差别。请注意，该字段应使用@ `Property`进行注释，以便在索引中使用。

此外，除了类级别的索引，Morphia 还有一个注释来定义字段级别的索引。

### 7.4.模式验证

我们可以选择为 MongoDB 在执行更新或插入操作时可以使用的集合提供**数据验证规则。Morphia 通过他们的 API 支持这一点。**

假设我们不想插入没有有效价格的书。我们可以利用模式验证来实现这一点:

```java
@Validation("{ price : { $gt : 0 } }")
public class Book {
    // ...
    @Property("price")
    private double cost;
    // ...
}
```

这里可以使用 MongoDB 提供的一组丰富的验证。

## 8.替代 MongoDB ODMs

Morphia 不是唯一可用的 MongoDB ODM。我们可以考虑在应用程序中使用其他几种方法。这里不可能讨论与 Morphia 的比较，但了解我们的选项总是有用的:

*   [Spring Data](/web/20221225164858/https://www.baeldung.com/spring-data-mongodb-guide) :为使用 MongoDB 提供了一个基于 Spring 的编程模型
*   [MongoJack](https://web.archive.org/web/20221225164858/https://mongojack.org/) :提供从 JSON 到 MongoDB 对象的直接映射

这不是一个完整的 MongoDB ODMs 列表，但是有一些有趣的替代选项！

## 9.结论

在本文中，我们了解了 MongoDB 的基本细节，以及如何使用 ODM 从 Java 等编程语言连接和操作 MongoDB。我们进一步探讨了 Morphia 作为一个 MongoDB ODM for Java 及其各种功能。

和往常一样，代码可以在 GitHub 上找到。