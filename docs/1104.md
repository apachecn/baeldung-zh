# 用 Spring 介绍 Jooq

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jooq-with-spring>

## 1。概述

本文将介绍 Jooq 面向对象查询——Jooq——以及一种与 Spring 框架协作设置它的简单方法。

大多数 Java 应用程序都有某种 SQL 持久性，并在 JPA 等高级工具的帮助下访问该层。虽然这很有用，但在某些情况下，您真的需要一个更好、更细致的工具来获取您的数据，或者真正利用底层 DB 所提供的一切。

Jooq 避免了一些典型的 ORM 模式，并生成允许我们构建类型安全查询的代码，并通过干净而强大的 fluent API 获得对生成的 SQL 的完全控制。

本文主要关注 Spring MVC。我们的文章 [Spring Boot 对 jOOQ 的支持](/web/20220627091424/https://www.baeldung.com/spring-boot-support-for-jooq)描述了如何在 Spring Boot 使用 jOOQ。

## 2。Maven 依赖关系

以下依赖项是运行本教程中的代码所必需的。

### 2.1 .jooq〔t1〕

```
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq</artifactId>
    <version>3.14.15</version>
</dependency>
```

### 2.2。弹簧

我们的例子需要几个 Spring 依赖项；然而，为了简单起见，我们只需要在 POM 文件中显式包含其中的两个:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

### 2.3。数据库

为了简化我们的示例，我们将使用 H2 嵌入式数据库:

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.191</version>
</dependency>
```

## 3。代码生成

### 3.1。数据库结构

让我们介绍一下贯穿本文的数据库结构。假设我们需要为出版商创建一个数据库来存储他们管理的书籍和作者的信息，其中一个作者可能写了许多书，一本书可能由许多作者合著。

为了简单起见，我们将只生成三个表:书籍的`book`、作者的`author`和另一个名为`author_book` 的表，以表示作者和书籍之间的多对多关系。`author`表有三列:`id`、`first_name`和`last_name.``book`表只包含一个`title`列和`id`主键。

存储在`intro_schema.sql`资源文件中的以下 SQL 查询将针对我们之前已经建立的数据库执行，以创建必要的表并用样本数据填充它们:

```
DROP TABLE IF EXISTS author_book, author, book;

CREATE TABLE author (
  id             INT          NOT NULL PRIMARY KEY,
  first_name     VARCHAR(50),
  last_name      VARCHAR(50)  NOT NULL
);

CREATE TABLE book (
  id             INT          NOT NULL PRIMARY KEY,
  title          VARCHAR(100) NOT NULL
);

CREATE TABLE author_book (
  author_id      INT          NOT NULL,
  book_id        INT          NOT NULL,

  PRIMARY KEY (author_id, book_id),
  CONSTRAINT fk_ab_author     FOREIGN KEY (author_id)  REFERENCES author (id)  
    ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT fk_ab_book       FOREIGN KEY (book_id)    REFERENCES book   (id)
);

INSERT INTO author VALUES 
  (1, 'Kathy', 'Sierra'), 
  (2, 'Bert', 'Bates'), 
  (3, 'Bryan', 'Basham');

INSERT INTO book VALUES 
  (1, 'Head First Java'), 
  (2, 'Head First Servlets and JSP'),
  (3, 'OCA/OCP Java SE 7 Programmer');

INSERT INTO author_book VALUES (1, 1), (1, 3), (2, 1);
```

### 3.2。属性 Maven 插件

我们将使用三个不同的 Maven 插件来生成 Jooq 代码。第一个是属性 Maven 插件。

这个插件用于从资源文件中读取配置数据。这不是必需的，因为数据可以直接添加到 POM 中，但是从外部管理这些属性是一个好主意。

在本节中，我们将在一个名为`intro_config.properties`的文件中定义数据库连接的属性，包括 JDBC 驱动程序类、数据库 URL、用户名和密码。将这些属性外部化使得切换数据库或仅仅更改配置数据变得容易。

这个插件的`read-project-properties`目标应该被绑定到一个早期阶段，以便配置数据可以被其他插件使用。在这种情况下，它被绑定到`initialize`阶段:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>properties-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
        <execution>
            <phase>initialize</phase>
            <goals>
                <goal>read-project-properties</goal>
            </goals>
            <configuration>
                <files>
                    <file>src/main/resources/intro_config.properties</file>
                </files>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 3.3。SQL Maven 插件

SQL Maven 插件用于执行 SQL 语句来创建和填充数据库表。它将利用 Properties Maven 插件从`intro_config.properties`文件中提取的属性，并从`intro_schema.sql`资源中获取 SQL 语句。

SQL Maven 插件配置如下:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>sql-maven-plugin</artifactId>
    <version>1.5</version>
    <executions>
        <execution>
            <phase>initialize</phase>
            <goals>
                <goal>execute</goal>
            </goals>
            <configuration>
                <driver>${db.driver}</driver>
                <url>${db.url}</url>
                <username>${db.username}</username>
                <password>${db.password}</password>
                <srcFiles>
                    <srcFile>src/main/resources/intro_schema.sql</srcFile>
                </srcFiles>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.191</version>
        </dependency>
    </dependencies>
</plugin>
```

注意，在 POM 文件中，这个插件必须放在 Properties Maven 插件之后，因为它们的执行目标都绑定到同一个阶段，Maven 将按照它们被列出的顺序执行它们。

### 3.4。jOOQ Codegen 插件

Jooq Codegen 插件从数据库表结构中生成 Java 代码。它的`generate`目标应该绑定到`generate-sources`阶段，以确保正确的执行顺序。插件元数据如下所示:

```
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <version>${org.jooq.version}</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <jdbc>
                    <driver>${db.driver}</driver>
                    <url>${db.url}</url>
                    <user>${db.username}</user>
                    <password>${db.password}</password>
                </jdbc>
                <generator>
                    <target>
                        <packageName>com.baeldung.jooq.introduction.db</packageName>
                        <directory>src/main/java</directory>
                    </target>
                </generator>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 3.5。生成代码

为了完成源代码生成的过程，我们需要运行 Maven `generate-sources`阶段。在 Eclipse 中，我们可以通过右键单击项目并选择`Run As`–>`Maven generate-sources`来做到这一点。该命令完成后，生成对应于`author`、`book`、`author_book`表(以及其他几个支持类)的源文件。

让我们深入研究表类，看看 Jooq 产生了什么。每个类都有一个与该类同名的静态字段，只是名称中的所有字母都是大写的。以下是从生成的类的定义中提取的代码片段:

`Author`类:

```
public class Author extends TableImpl<AuthorRecord> {
    public static final Author AUTHOR = new Author();

    // other class members
}
```

`Book`类:

```
public class Book extends TableImpl<BookRecord> {
    public static final Book BOOK = new Book();

    // other class members
}
```

`AuthorBook`类:

```
public class AuthorBook extends TableImpl<AuthorBookRecord> {
    public static final AuthorBook AUTHOR_BOOK = new AuthorBook();

    // other class members
}
```

当与项目中的其他层一起工作时，这些静态字段引用的实例将作为数据访问对象来表示相应的表。

## 4。弹簧配置

### 4.1。将 jOOQ 异常转换为 Spring

为了使 Jooq 执行抛出的异常与 Spring 对数据库访问的支持保持一致，我们需要将它们转换成`DataAccessException`类的子类型。

让我们定义一个`ExecuteListener`接口的实现来转换异常:

```
public class ExceptionTranslator extends DefaultExecuteListener {
    public void exception(ExecuteContext context) {
        SQLDialect dialect = context.configuration().dialect();
        SQLExceptionTranslator translator 
          = new SQLErrorCodeSQLExceptionTranslator(dialect.name());
        context.exception(translator
          .translate("Access database using Jooq", context.sql(), context.sqlException()));
    }
}
```

这个类将被 Spring 应用程序上下文使用。

### 4.2。配置弹簧

本节将介绍定义一个包含元数据和 beanss 的`PersistenceContext`的步骤，这些元数据和 bean 将在 Spring 应用程序上下文中使用。

让我们从对类应用必要的注释开始:

*   `@Configuration`:使类被识别为 beans 的容器
*   `@ComponentScan`:配置扫描指令，包括`value`选项来声明一个包名数组以搜索组件。在本教程中，要搜索的包是由 Jooq Codegen Maven 插件生成的包
*   `@EnableTransactionManagement`:启用 Spring 管理的事务
*   `@PropertySource`:表示要加载的属性文件的位置。本文中的值指向包含配置数据和数据库方言的文件，这恰好是 4.1 小节中提到的同一文件。

```
@Configuration
@ComponentScan({"com.baeldung.Jooq.introduction.db.public_.tables"})
@EnableTransactionManagement
@PropertySource("classpath:intro_config.properties")
public class PersistenceContext {
    // Other declarations
}
```

接下来，使用一个`Environment`对象来获取配置数据，然后用它来配置`DataSource` bean:

```
@Autowired
private Environment environment;

@Bean
public DataSource dataSource() {
    JdbcDataSource dataSource = new JdbcDataSource();

    dataSource.setUrl(environment.getRequiredProperty("db.url"));
    dataSource.setUser(environment.getRequiredProperty("db.username"));
    dataSource.setPassword(environment.getRequiredProperty("db.password"));
```

```
 return dataSource; 
}
```

现在我们定义几个 beans 来处理数据库访问操作:

```
@Bean
public TransactionAwareDataSourceProxy transactionAwareDataSource() {
    return new TransactionAwareDataSourceProxy(dataSource());
}

@Bean
public DataSourceTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSource());
}

@Bean
public DataSourceConnectionProvider connectionProvider() {
    return new DataSourceConnectionProvider(transactionAwareDataSource());
}

@Bean
public ExceptionTranslator exceptionTransformer() {
    return new ExceptionTranslator();
}

@Bean
public DefaultDSLContext dsl() {
    return new DefaultDSLContext(configuration());
}
```

最后，我们提供了一个 Jooq `Configuration`实现，并将其声明为由`DSLContext`类使用的 Spring bean:

```
@Bean
public DefaultConfiguration configuration() {
    DefaultConfiguration JooqConfiguration = new DefaultConfiguration();
    jooqConfiguration.set(connectionProvider());
    jooqConfiguration.set(new DefaultExecuteListenerProvider(exceptionTransformer()));

    String sqlDialectName = environment.getRequiredProperty("jooq.sql.dialect");
    SQLDialect dialect = SQLDialect.valueOf(sqlDialectName);
    jooqConfiguration.set(dialect);

    return jooqConfiguration;
}
```

## 5。使用 jOOQ 和 Spring

本节演示 Jooq 在常见数据库访问查询中的使用。对于每种类型的“写”操作，包括插入、更新和删除数据，有两个测试，一个用于提交，一个用于回滚。当选择数据来验证“写”查询时，示出了“读”操作的使用。

我们将首先声明一个自动连接的`DSLContext`对象和 Jooq 生成的类的实例，供所有测试方法使用:

```
@Autowired
private DSLContext dsl;

Author author = Author.AUTHOR;
Book book = Book.BOOK;
AuthorBook authorBook = AuthorBook.AUTHOR_BOOK;
```

### 5.1。插入数据

第一步是将数据插入表格:

```
dsl.insertInto(author)
  .set(author.ID, 4)
  .set(author.FIRST_NAME, "Herbert")
  .set(author.LAST_NAME, "Schildt")
  .execute();
dsl.insertInto(book)
  .set(book.ID, 4)
  .set(book.TITLE, "A Beginner's Guide")
  .execute();
dsl.insertInto(authorBook)
  .set(authorBook.AUTHOR_ID, 4)
  .set(authorBook.BOOK_ID, 4)
  .execute();
```

提取数据的`SELECT`查询:

```
Result<Record3<Integer, String, Integer>> result = dsl
  .select(author.ID, author.LAST_NAME, DSL.count())
  .from(author)
  .join(authorBook)
  .on(author.ID.equal(authorBook.AUTHOR_ID))
  .join(book)
  .on(authorBook.BOOK_ID.equal(book.ID))
  .groupBy(author.LAST_NAME)
  .fetch();
```

上述查询产生以下输出:

```
+----+---------+-----+
|  ID|LAST_NAME|count|
+----+---------+-----+
|   1|Sierra   |    2|
|   2|Bates    |    1|
|   4|Schildt  |    1|
+----+---------+-----+
```

结果由`Assert` API 确认:

```
assertEquals(3, result.size());
assertEquals("Sierra", result.getValue(0, author.LAST_NAME));
assertEquals(Integer.valueOf(2), result.getValue(0, DSL.count()));
assertEquals("Schildt", result.getValue(2, author.LAST_NAME));
assertEquals(Integer.valueOf(1), result.getValue(2, DSL.count()));
```

当由于无效查询而导致失败时，会引发异常，并且事务会回滚。在以下示例中，`INSERT`查询违反了外键约束，导致异常:

```
@Test(expected = DataAccessException.class)
public void givenInvalidData_whenInserting_thenFail() {
    dsl.insertInto(authorBook)
      .set(authorBook.AUTHOR_ID, 4)
      .set(authorBook.BOOK_ID, 5)
      .execute();
}
```

### 5.2。更新数据

现在让我们更新现有数据:

```
dsl.update(author)
  .set(author.LAST_NAME, "Baeldung")
  .where(author.ID.equal(3))
  .execute();
dsl.update(book)
  .set(book.TITLE, "Building your REST API with Spring")
  .where(book.ID.equal(3))
  .execute();
dsl.insertInto(authorBook)
  .set(authorBook.AUTHOR_ID, 3)
  .set(authorBook.BOOK_ID, 3)
  .execute();
```

获取必要的数据:

```
Result<Record3<Integer, String, String>> result = dsl
  .select(author.ID, author.LAST_NAME, book.TITLE)
  .from(author)
  .join(authorBook)
  .on(author.ID.equal(authorBook.AUTHOR_ID))
  .join(book)
  .on(authorBook.BOOK_ID.equal(book.ID))
  .where(author.ID.equal(3))
  .fetch();
```

输出应该是:

```
+----+---------+----------------------------------+
|  ID|LAST_NAME|TITLE                             |
+----+---------+----------------------------------+
|   3|Baeldung |Building your REST API with Spring|
+----+---------+----------------------------------+
```

以下测试将验证 Jooq 是否按预期工作:

```
assertEquals(1, result.size());
assertEquals(Integer.valueOf(3), result.getValue(0, author.ID));
assertEquals("Baeldung", result.getValue(0, author.LAST_NAME));
assertEquals("Building your REST API with Spring", result.getValue(0, book.TITLE));
```

在失败的情况下，抛出一个异常，事务回滚，我们通过测试来确认这一点:

```
@Test(expected = DataAccessException.class)
public void givenInvalidData_whenUpdating_thenFail() {
    dsl.update(authorBook)
      .set(authorBook.AUTHOR_ID, 4)
      .set(authorBook.BOOK_ID, 5)
      .execute();
}
```

### 5.3。删除数据

以下方法删除一些数据:

```
dsl.delete(author)
  .where(author.ID.lt(3))
  .execute();
```

以下是读取受影响的表的查询:

```
Result<Record3<Integer, String, String>> result = dsl
  .select(author.ID, author.FIRST_NAME, author.LAST_NAME)
  .from(author)
  .fetch();
```

查询输出:

```
+----+----------+---------+
|  ID|FIRST_NAME|LAST_NAME|
+----+----------+---------+
|   3|Bryan     |Basham   |
+----+----------+---------+
```

以下测试验证删除:

```
assertEquals(1, result.size());
assertEquals("Bryan", result.getValue(0, author.FIRST_NAME));
assertEquals("Basham", result.getValue(0, author.LAST_NAME));
```

另一方面，如果一个查询无效，它将抛出一个异常，事务回滚。以下测试将证明:

```
@Test(expected = DataAccessException.class)
public void givenInvalidData_whenDeleting_thenFail() {
    dsl.delete(book)
      .where(book.ID.equal(1))
      .execute();
}
```

## 6。结论

本教程介绍了 Jooq 的基础知识，Jooq 是一个用于处理数据库的 Java 库。它涵盖了从数据库结构生成源代码的步骤，以及如何使用新创建的类与数据库进行交互。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。