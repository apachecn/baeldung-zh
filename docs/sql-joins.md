# SQL 联接的类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sql-joins>

## 1。简介

在本教程中，我们将展示不同类型的 SQL 连接，以及如何在 Java 中轻松实现它们。

## 2。定义模型

让我们首先创建两个简单的表:

```
CREATE TABLE AUTHOR
(
  ID int NOT NULL PRIMARY KEY,
  FIRST_NAME varchar(255),
  LAST_NAME varchar(255)
);

CREATE TABLE ARTICLE
(
  ID int NOT NULL PRIMARY KEY,
  TITLE varchar(255) NOT NULL,
  AUTHOR_ID int,
  FOREIGN KEY(AUTHOR_ID) REFERENCES AUTHOR(ID)
); 
```

并用一些测试数据填充它们:

```
INSERT INTO AUTHOR VALUES 
(1, 'Siena', 'Kerr'),
(2, 'Daniele', 'Ferguson'),
(3, 'Luciano', 'Wise'),
(4, 'Jonas', 'Lugo');

INSERT INTO ARTICLE VALUES
(1, 'First steps in Java', 1),
(2, 'SpringBoot tutorial', 1),
(3, 'Java 12 insights', null),
(4, 'SQL JOINS', 2),
(5, 'Introduction to Spring Security', 3);
```

注意，在我们的样本数据集中，并不是所有的作者都有文章，反之亦然。这将在我们的示例中发挥重要作用，我们稍后会看到。

让我们还定义一个 POJO，我们将在整个教程中使用它来存储连接操作的结果:

```
class ArticleWithAuthor {

    private String title;
    private String authorFirstName;
    private String authorLastName;

    // standard constructor, setters and getters
}
```

在我们的示例中，我们将从文章表中提取标题，从作者表中提取作者数据。

## 3。配置

对于我们的示例，我们将使用运行在端口 5432 上的外部 PostgreSQL 数据库。除了 MySQL 和 H2 都不支持的完全连接之外，所有提供的片段都可以与任何 SQL 提供者一起工作。

对于我们的 Java 实现，我们需要一个 [PostgreSQL 驱动程序](https://web.archive.org/web/20220629002650/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.postgresql%22%20AND%20a%3A%22postgresql%22):

```
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.5</version>
    <scope>test</scope>
</dependency>
```

让我们首先配置一个`java.sql.Connection`来使用我们的数据库:

```
Class.forName("org.postgresql.Driver");
Connection connection = DriverManager.
  getConnection("jdbc:postgresql://localhost:5432/myDb", "user", "pass");
```

接下来，让我们创建一个 DAO 类和一些实用方法:

```
class ArticleWithAuthorDAO {

    private final Connection connection;

    // constructor

    private List<ArticleWithAuthor> executeQuery(String query) {
        try (Statement statement = connection.createStatement()) {
            ResultSet resultSet = statement.executeQuery(query);
            return mapToList(resultSet);
        } catch (SQLException e) {
            e.printStackTrace();
        }
            return new ArrayList<>();
    }

    private List<ArticleWithAuthor> mapToList(ResultSet resultSet) throws SQLException {
        List<ArticleWithAuthor> list = new ArrayList<>();
        while (resultSet.next()) {
            ArticleWithAuthor articleWithAuthor = new ArticleWithAuthor(
              resultSet.getString("TITLE"),
              resultSet.getString("FIRST_NAME"),
              resultSet.getString("LAST_NAME")
            );
            list.add(articleWithAuthor);
        }
        return list;
    }
}
```

在本文中，我们不会深究关于使用`[ResultSet](/web/20220629002650/https://www.baeldung.com/jdbc-resultset), Statement,` 和`Connection.` 的细节，这些话题在我们的 [JDBC](/web/20220629002650/https://www.baeldung.com/java-jdbc) 相关文章中有所涉及。

让我们在下面的小节中开始探索 SQL 连接。

## 4。内部连接

让我们从最简单的连接类型开始。内部联接是一种从两个表中选择匹配给定条件的行的操作。该查询至少由三部分组成:选择列、连接表和连接条件。

记住这一点，语法本身变得非常简单:

```
SELECT ARTICLE.TITLE, AUTHOR.LAST_NAME, AUTHOR.FIRST_NAME
  FROM ARTICLE INNER JOIN AUTHOR 
  ON AUTHOR.ID=ARTICLE.AUTHOR_ID
```

我们也可以将**内连接的结果作为相交集的公共部分来说明:**

[![inner join](img/3f31cfaa53c21baa5fc2192c5add735c.png)](/web/20220629002650/https://www.baeldung.com/wp-content/uploads/2019/04/inner_join.png)

现在让我们在`ArticleWithAuthorDAO`类中实现内部连接的方法:

```
List<ArticleWithAuthor> articleInnerJoinAuthor() {
    String query = "SELECT ARTICLE.TITLE, AUTHOR.LAST_NAME, AUTHOR.FIRST_NAME "
      + "FROM ARTICLE INNER JOIN AUTHOR ON AUTHOR.ID=ARTICLE.AUTHOR_ID";
    return executeQuery(query);
}
```

并测试它:

```
@Test
public void whenQueryWithInnerJoin_thenShouldReturnProperRows() {
    List<ArticleWithAuthor> articleWithAuthorList = articleWithAuthorDAO.articleInnerJoinAuthor();

    assertThat(articleWithAuthorList).hasSize(4);
    assertThat(articleWithAuthorList)
      .noneMatch(row -> row.getAuthorFirstName() == null || row.getTitle() == null);
}
```

正如我们前面提到的，内部连接只根据提供的条件选择公共行。查看我们的插入，我们看到我们有一篇没有作者的文章和一个没有文章的作者。这些行被跳过，因为它们不满足提供的条件。因此，我们检索了四个连接的结果，它们都没有空的作者数据和空的标题。

## 5。左连接

接下来，我们来关注一下左连接。这种联接选择第一个表中的所有行，并匹配第二个表中的相应行。当不匹配时，用`null` 值`.`填充列

在我们深入 Java 实现之前，让我们看一下左连接的图形表示:

[![left join](img/a21c5855aced4404307a009f99f49ee4.png)](/web/20220629002650/https://www.baeldung.com/wp-content/uploads/2019/04/left_join.png)

在这种情况下， **LEFT JOIN 的结果包括表示第一个表的集合中的每条记录，以及第二个表中的相交值。**

现在，让我们转到 Java 实现:

```
List<ArticleWithAuthor> articleLeftJoinAuthor() {
    String query = "SELECT ARTICLE.TITLE, AUTHOR.LAST_NAME, AUTHOR.FIRST_NAME "
      + "FROM ARTICLE LEFT JOIN AUTHOR ON AUTHOR.ID=ARTICLE.AUTHOR_ID";
    return executeQuery(query);
}
```

与前一个例子的唯一区别是我们使用了 LEFT 关键字而不是 INNER 关键字。

在测试左连接方法之前，让我们再看一下我们的插入。在这种情况下，我们将从 ARTICLE 表中接收所有记录，并从 AUTHOR 表中接收它们的匹配行。正如我们之前提到的，并不是每篇文章都有作者，所以我们希望用`null`值来代替作者数据:

```
@Test
public void whenQueryWithLeftJoin_thenShouldReturnProperRows() {
    List<ArticleWithAuthor> articleWithAuthorList = articleWithAuthorDAO.articleLeftJoinAuthor();

    assertThat(articleWithAuthorList).hasSize(5);
    assertThat(articleWithAuthorList).anyMatch(row -> row.getAuthorFirstName() == null);
}
```

## 6。右连接

**右连接很像左连接，但是它返回第二个表中的所有行，并匹配第一个表中的行。**就像在左连接的情况下，空匹配被替换为`null`值。

这种连接的图形表示是我们为左连接所展示的连接的镜像:

[![right join](img/5b2b5c1a5c5d3c17e4fc4ab8e73ed207.png)](/web/20220629002650/https://www.baeldung.com/wp-content/uploads/2019/04/right_join.png)

让我们用 Java 实现正确的连接:

```
List<ArticleWithAuthor> articleRightJoinAuthor() {
    String query = "SELECT ARTICLE.TITLE, AUTHOR.LAST_NAME, AUTHOR.FIRST_NAME "
      + "FROM ARTICLE RIGHT JOIN AUTHOR ON AUTHOR.ID=ARTICLE.AUTHOR_ID";
    return executeQuery(query);
}
```

同样，让我们看看我们的测试数据。因为这个连接操作从第二个表中检索所有记录，我们期望检索五行，并且因为不是每个作者都已经写了一篇文章，我们期望 TITLE 列中有一些`null`值:

```
@Test
public void whenQueryWithRightJoin_thenShouldReturnProperRows() {
    List<ArticleWithAuthor> articleWithAuthorList = articleWithAuthorDAO.articleRightJoinAuthor();

    assertThat(articleWithAuthorList).hasSize(5);
    assertThat(articleWithAuthorList).anyMatch(row -> row.getTitle() == null);
}
```

## 7。完全外部连接

这个连接操作可能是最棘手的。**无论条件是否满足，完全连接都会选择第一个和第二个表中的所有行。**

我们也可以将相同的概念表示为每个相交集合中的所有值:

[![full join](img/41d0e96ad52169003375b829fae5da0d.png)](/web/20220629002650/https://www.baeldung.com/wp-content/uploads/2019/04/full_join.png)

让我们来看看 Java 实现:

```
List<ArticleWithAuthor> articleOuterJoinAuthor() {
    String query = "SELECT ARTICLE.TITLE, AUTHOR.LAST_NAME, AUTHOR.FIRST_NAME "
      + "FROM ARTICLE FULL JOIN AUTHOR ON AUTHOR.ID=ARTICLE.AUTHOR_ID";
    return executeQuery(query);
}
```

现在，我们可以测试我们的方法:

```
@Test
public void whenQueryWithFullJoin_thenShouldReturnProperRows() {
    List<ArticleWithAuthor> articleWithAuthorList = articleWithAuthorDAO.articleOuterJoinAuthor();

    assertThat(articleWithAuthorList).hasSize(6);
    assertThat(articleWithAuthorList).anyMatch(row -> row.getTitle() == null);
    assertThat(articleWithAuthorList).anyMatch(row -> row.getAuthorFirstName() == null);
}
```

再一次，让我们看看测试数据。我们有五篇不同的文章，其中一篇没有作者，四位作者，其中一篇没有指定的文章。作为完全连接的结果，我们期望检索六行。其中四个互相匹配，其余两个不匹配。出于这个原因，我们还假设在两个作者数据列中至少有一行具有`null`值，在标题列中至少有一行具有`null`值。

## 8。结论

在本文中，我们探讨了 SQL 连接的基本类型。我们看了四种类型的连接的例子，以及它们如何在 Java 中实现。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220629002650/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)