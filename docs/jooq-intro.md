# jOOQ 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jooq-intro>

## 1.介绍

在本教程中，我们将快速浏览使用 [jOOQ](https://web.archive.org/web/20220525142108/https://www.jooq.org/) (Java 面向对象查询)运行应用程序。该库基于数据库表生成 Java 类，并允许我们通过其 fluent API 创建类型安全的 SQL 查询。

我们将介绍整个设置、PostgreSQL 数据库连接和一些 CRUD 操作的例子。

## 2。Maven 依赖关系

对于 jOOQ 库，我们需要以下[三个 jOOQ 依赖项](https://web.archive.org/web/20220525142108/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jooq%22%20AND%20(a%3A%22jooq%22%20OR%20a%3A%22jooq-meta%22%20OR%20a%3A%22jooq-codegen%22)):

```java
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq</artifactId>
    <version>3.13.4</version>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-meta</artifactId>
    <version>3.13.4</version>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen</artifactId>
    <version>3.13.4</version>
</dependency>
```

我们还需要一个对 [PostgreSQL 驱动程序](https://web.archive.org/web/20220525142108/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.postgresql%22%20AND%20a%3A%22postgresql%22)的依赖:

```java
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.16</version>
</dependency>
```

## 3。数据库结构

在开始之前，让我们为示例创建一个简单的 DB 模式。我们将使用一个简单的`Author`和一个`Article`关系:

```java
create table AUTHOR
(
    ID         integer PRIMARY KEY,
    FIRST_NAME varchar(255),
    LAST_NAME  varchar(255),
    AGE        integer
);

create table ARTICLE
(
    ID          integer PRIMARY KEY,
    TITLE       varchar(255) not null,
    DESCRIPTION varchar(255),
    AUTHOR_ID   integer
        CONSTRAINT fk_author_id REFERENCES AUTHOR
);
```

## 4.数据库连接

现在，让我们看看我们将如何[连接到我们的数据库](/web/20220525142108/https://www.baeldung.com/java-jdbc)。

首先，我们需要提供用户、密码和数据库的完整 URL。我们将使用这些属性通过使用`DriverManager`和它的`getConnection`方法来创建一个`Connection`对象:

```java
String userName = "user";
String password = "pass";
String url = "jdbc:postgresql://db_host:5432/baeldung";
Connection conn = DriverManager.getConnection(url, userName, password); 
```

接下来，我们需要创建一个 [`DSLContext`](https://web.archive.org/web/20220525142108/https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/DSLContext.html) 的实例。这个对象将成为 jOOQ 接口的入口点:

```java
DSLContext context = DSL.using(conn, SQLDialect.POSTGRES);
```

在我们的例子中，我们正在传递`POSTGRES`方言，但是很少有其他方言可用，比如 H2、MySQL、SQLite 等等。

## 5.代码生成

要为我们的数据库表生成 Java 类，我们需要下面的`jooq-config.xml`文件:

```java
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration >

    <jdbc>
        <driver>org.postgresql.Driver</driver>
        <url>jdbc:postgresql://db_url:5432/baeldung_database</url>
        <user>username</user>
        <password>password</password>
    </jdbc>

    <generator>
        <name>org.jooq.codegen.JavaGenerator</name>

        <database>
            <name>org.jooq.meta.postgres.PostgresDatabase</name>
            <inputSchema>public</inputSchema>
            <includes>.*</includes>
            <excludes></excludes>
        </database>

        <target>
            <packageName>com.baeldung.jooq.model</packageName>
            <directory>C:/projects/baeldung/tutorials/jooq-examples/src/main/java</directory>
        </target>
    </generator>
</configuration>
```

定制配置需要在放置数据库凭证的`<jdbc>`部分和为将要生成的类配置包名和位置目录的`<target>`部分进行更改。

要执行 jOOQ 代码生成工具，我们需要运行以下代码:

```java
GenerationTool.generate(
  Files.readString(
    Path.of("jooq-config.xml")
  )    
);
```

生成完成后，我们将获得以下两个类，每个类都对应于其数据库表:

```java
com.baeldung.model.generated.tables.Article;
com.baeldung.model.generated.tables.Author;
```

## 6.`CRUD`操作

现在，让我们来看看可以用 jOOQ 库执行的一些基本 CRUD 操作。

### 6.1.创造

首先，让我们创建一个新的`Article`记录。为此，我们需要调用`newRecord`方法，并使用适当的表引用作为参数:

```java
ArticleRecord article = context.newRecord(Article.ARTICLE);
```

`Article.ARTICLE` 变量是`ARTICLE` 数据库表的引用实例。它是由 jOOQ 在代码生成过程中自动创建的。

接下来，我们可以为所有需要的属性设置值:

```java
article.setId(2);
article.setTitle("jOOQ examples");
article.setDescription("A few examples of jOOQ CRUD operations");
article.setAuthorId(1);
```

最后，我们需要对记录调用`store`方法，以将其保存在数据库中:

```java
article.store();
```

### 6.2.阅读

现在，让我们看看如何从数据库中读取值。例如，让我们选择所有作者:

```java
Result<Record> authors = context.select()
  .from(Author.AUTHOR)
  .fetch();
```

这里，我们使用结合了`from`子句的`select`方法来指示我们想要从哪个表中读取。调用`fetch`方法执行 SQL 查询并返回生成的结果。

`Result`对象实现了`Iterable`接口，因此很容易迭代每个元素。在访问单个记录时，我们可以通过使用带有适当字段引用的`getValue`方法来获取它的参数:

```java
authors.forEach(author -> {
    Integer id = author.getValue(Author.AUTHOR.ID);
    String firstName = author.getValue(Author.AUTHOR.FIRST_NAME);
    String lastName = author.getValue(Author.AUTHOR.LAST_NAME);
    Integer age = author.getValue(Author.AUTHOR.AGE);

    System.out.printf("Author %s %s has id: %d and age: %d%n", firstName, lastName, id, age);
});
```

我们可以将选择查询限制到一组特定的字段。让我们只获取文章 id 和标题:

```java
Result<Record2<Integer, String>> articles = context.select(Article.ARTICLE.ID, Article.ARTICLE.TITLE)
  .from(Author.AUTHOR)
  .fetch();
```

我们也可以用`fetchOne`方法选择单个对象。这个函数的参数是表引用和匹配正确记录的条件。

在我们的例子中，让我们选择一个 id 等于 1 的`Author`:

```java
AuthorRecord author = context.fetchOne(Author.AUTHOR, Author.AUTHOR.ID.eq(1))
```

如果没有符合条件的记录，`fetchOne`方法将返回`null`。

### 6.3.更新

为了更新给定的记录，我们可以使用来自`DSLContext`对象的`update` 方法，并结合对每个需要更改的字段的`set`方法调用。该语句后面应该有一个带有适当匹配条件的`where`子句:

```java
context.update(Author.AUTHOR)
  .set(Author.AUTHOR.FIRST_NAME, "David")
  .set(Author.AUTHOR.LAST_NAME, "Brown")
  .where(Author.AUTHOR.ID.eq(1))
  .execute();
```

只有在我们调用了`execute`方法之后，更新查询才会运行。作为返回值，我们将得到一个等于更新记录数的整数。

也可以通过执行它的`store` 方法来更新已经获取的记录:

```java
ArticleRecord article = context.fetchOne(Article.ARTICLE, Article.ARTICLE.ID.eq(1));
article.setTitle("A New Article Title");
article.store();
```

如果操作成功，则`store` 方法将返回`1`,如果不需要更新，则返回`0`。例如，没有与条件匹配的内容。

### 6.4。删除

要删除一个给定的记录，我们可以从`DSLContext`对象中使用`delete`方法。删除条件应该作为参数在下面的`where`子句中传递:

```java
context.delete(Article.ARTICLE)
  .where(Article.ARTICLE.ID.eq(1))
  .execute();
```

只有在我们调用了`execute`方法之后，删除查询才会运行。作为返回值，我们将得到一个等于已删除记录数的整数。

也可以通过执行它的`delete`方法来删除已经获取的记录:

```java
ArticleRecord articleRecord = context.fetchOne(Article.ARTICLE, Article.ARTICLE.ID.eq(1));
articleRecord.delete();
```

如果操作成功，则`delete` 方法将返回`1`,如果不需要删除，则返回`0`。例如，当没有与条件匹配的内容时。

## 7 .。结论

在本文中，我们学习了如何使用 jOOQ 框架配置和创建一个简单的 CRUD 应用程序。像往常一样，所有的源代码都可以在 GitHub 上获得。