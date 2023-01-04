# Apache Cayenne 中的高级查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-cayenne-query>

## 1。概述

[在之前的](/web/20220625225228/https://www.baeldung.com/apache-cayenne-orm)中，我们重点介绍了如何开始使用 Apache Cayenne。

在本文中，我们将介绍如何用 ORM 编写简单和高级的查询。

## 2。设置

设置类似于前一篇文章中使用的设置。

此外，在每次测试之前，我们会保存三位作者，在测试结束时，我们会删除他们:

*   保罗·泽维尔
*   保罗·史密斯
*   维琪萨拉

## 3。`ObjectSelect`

让我们从简单的开始，看看我们如何能得到名字包含“Paul”的所有作者:

```java
@Test
public void whenContainsObjS_thenWeGetOneRecord() {
    List<Author> authors = ObjectSelect.query(Author.class)
      .where(Author.NAME.contains("Paul"))
      .select(context);

    assertEquals(authors.size(), 1);
}
```

接下来，让我们看看如何对作者姓名列应用不区分大小写的 LIKE 类型的查询:

```java
@Test
void whenLikeObjS_thenWeGetTwoAuthors() {
    List<Author> authors = ObjectSelect.query(Author.class)
      .where(Author.NAME.likeIgnoreCase("Paul%"))
      .select(context);

    assertEquals(authors.size(), 2);
}
```

接下来，`endsWith()`表达式将只返回一条记录，因为只有一个作者有匹配的名字:

```java
@Test
void whenEndsWithObjS_thenWeGetOrderedAuthors() {
    List<Author> authors = ObjectSelect.query(Author.class)
      .where(Author.NAME.endsWith("Sarra"))
      .select(context);
    Author firstAuthor = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(firstAuthor.getName(), "Vicky Sarra");
}
```

一个更复杂的方法是查询名字在列表中的作者:

```java
@Test
void whenInObjS_thenWeGetAuthors() {
    List names = Arrays.asList(
      "Paul Xavier", "pAuL Smith", "Vicky Sarra");

    List<Author> authors = ObjectSelect.query(Author.class)
      .where(Author.NAME.in(names))
      .select(context);

    assertEquals(authors.size(), 3);
}
```

`nin`一个是相反的，这里只有“Vicky”会出现在结果中:

```java
@Test
void whenNinObjS_thenWeGetAuthors() {
    List names = Arrays.asList(
      "Paul Xavier", "pAuL Smith");
    List<Author> authors = ObjectSelect.query(Author.class)
      .where(Author.NAME.nin(names))
      .select(context);
    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "Vicky Sarra");
}
```

请注意，以下两个代码是相同的，因为它们都将使用相同的参数创建相同类型的表达式:

```java
Expression qualifier = ExpressionFactory
  .containsIgnoreCaseExp(Author.NAME.getName(), "Paul");
```

```java
Author.NAME.containsIgnoreCase("Paul");
```

下面是`[Expression](https://web.archive.org/web/20220625225228/https://cayenne.apache.org/docs/4.0/api/org/apache/cayenne/exp/Expression.html)` 和 [`ExpressionFactory`](https://web.archive.org/web/20220625225228/https://cayenne.apache.org/docs/4.0/api/org/apache/cayenne/exp/ExpressionFactory.html) 类`:`中一些可用的表达列表

*   `likeExp`:用于建立喜欢的表情
*   `likeIgnoreCaseExp`:用于构建 LIKE_IGNORE_CASE 表达式
*   `containsExp`:LIKE 查询的表达式，其模式匹配字符串中的任何位置
*   `containsIgnoreCaseExp`:与`containsExp`相同，但使用不区分大小写的方法
*   `startsWithExp`:模式应该匹配字符串的开头
*   `startsWithIgnoreCaseExp`:类似于`startsWithExp`，但是使用不区分大小写的方法
*   `endsWithExp`:匹配字符串末尾的表达式
*   `endsWithIgnoreCaseExp`:使用不区分大小写的方法匹配字符串结尾的表达式
*   `expTrue`:布尔`true`表达式
*   `expFalse`:布尔`false`表达式
*   `andExp`:用`and`运算符将两个表达式链接起来
*   `orExp`:使用`or`操作符链接两个表达式

更多的书面测试可在文章的代码源中找到，请查看 [Github](https://web.archive.org/web/20220625225228/https://github.com/eugenp/tutorials/tree/master/persistence-modules/apache-cayenne) 资源库。

## 4。`SelectQuery`

这是用户应用程序中使用最广泛的查询类型。`SelectQuery`描述了一个简单而强大的 API，其行为类似于 SQL 语法，但仍然使用 Java 对象和方法，并遵循构建器模式来构建更复杂的表达式。

这里我们讨论的是一种表达式语言，其中我们使用`Expression` (构建表达式)aka 限定符和`Ordering` (对结果进行排序)类来构建查询，这些类接下来被 ORM 转换为原生 SQL。

为了了解这一点，我们将一些测试放在一起，展示如何在实践中构建一些表达式并对数据进行排序。

让我们应用一个 LIKE 查询来获取姓名为" Paul" **:** 的作者

```java
@Test
void whenLikeSltQry_thenWeGetOneAuthor() {
    Expression qualifier 
      = ExpressionFactory.likeExp(Author.NAME.getName(), "Paul%");
    SelectQuery query 
      = new SelectQuery(Author.class, qualifier);

    List<Author> authorsTwo = context.performQuery(query);

    assertEquals(authorsTwo.size(), 1);
}
```

这意味着如果您不为查询(`SelectQuery`)提供任何表达式，结果将是 Author 表的所有记录。

可以使用`containsIgnoreCaseExp` 表达式执行一个类似的查询，以获得姓名中包含 Paul 的所有作者，而不考虑字母的大小写:

```java
@Test
void whenCtnsIgnorCaseSltQry_thenWeGetTwoAuthors() {
    Expression qualifier = ExpressionFactory
      .containsIgnoreCaseExp(Author.NAME.getName(), "Paul");
    SelectQuery query 
      = new SelectQuery(Author.class, qualifier);

    List<Author> authors = context.performQuery(query);

    assertEquals(authors.size(), 2);
}
```

类似地，让我们以不区分大小写的方式(`containsIgnoreCaseExp`)获取姓名中包含“Paul”的作者，并且姓名以字母 h 结尾(`endsWithExp`):

```java
@Test
void whenCtnsIgnorCaseEndsWSltQry_thenWeGetTwoAuthors() {
    Expression qualifier = ExpressionFactory
      .containsIgnoreCaseExp(Author.NAME.getName(), "Paul")
      .andExp(ExpressionFactory
        .endsWithExp(Author.NAME.getName(), "h"));
    SelectQuery query = new SelectQuery(
      Author.class, qualifier);
    List<Author> authors = context.performQuery(query);

    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "pAuL Smith");
}
```

可以使用`Ordering`类执行升序排序:

```java
@Test
void whenAscOrdering_thenWeGetOrderedAuthors() {
    SelectQuery query = new SelectQuery(Author.class);
    query.addOrdering(Author.NAME.asc());

    List<Author> authors = query.select(context);
    Author firstAuthor = authors.get(0);

    assertEquals(authors.size(), 3);
    assertEquals(firstAuthor.getName(), "Paul Xavier");
}
```

这里不使用`query.addOrdering(Author.NAME.asc()),` ，我们也可以只使用`SortOrder` 类来获得升序:

```java
query.addOrdering(Author.NAME.getName(), SortOrder.ASCENDING);
```

相对来说，是降序排列:

```java
@Test
void whenDescOrderingSltQry_thenWeGetOrderedAuthors() {
    SelectQuery query = new SelectQuery(Author.class);
    query.addOrdering(Author.NAME.desc());

    List<Author> authors = query.select(context);
    Author firstAuthor = authors.get(0);

    assertEquals(authors.size(), 3);
    assertEquals(firstAuthor.getName(), "pAuL Smith");
}
```

正如我们在前面的例子中看到的，设置这种顺序的另一种方法是:

```java
query.addOrdering(Author.NAME.getName(), SortOrder.DESCENDING);
```

## 5。`SQLTemplate`

`SQLTemplate` 也是 Cayenne 不使用对象样式查询的一种替代方法。

用`SQLTemplate`构建查询与用一些参数编写原生 SQL 语句直接相关。让我们实现一些快速的例子。

以下是我们在每次测试后删除所有作者的方法:

```java
@After
void deleteAllAuthors() {
    SQLTemplate deleteAuthors = new SQLTemplate(
      Author.class, "delete from author");
    context.performGenericQuery(deleteAuthors);
}
```

要找到所有记录的作者，我们只需应用 SQL 查询`select * from Author` ，我们将直接看到结果是正确的，因为我们正好有三个保存的作者:

```java
@Test
void givenAuthors_whenFindAllSQLTmplt_thenWeGetThreeAuthors() {
    SQLTemplate select = new SQLTemplate(
      Author.class, "select * from Author");
    List<Author> authors = context.performQuery(select);

    assertEquals(authors.size(), 3);
}
```

接下来，让我们来看看作者的名字“维姬·萨拉”:

```java
@Test
void givenAuthors_whenFindByNameSQLTmplt_thenWeGetOneAuthor() {
    SQLTemplate select = new SQLTemplate(
      Author.class, "select * from Author where name = 'Vicky Sarra'");
    List<Author> authors = context.performQuery(select);
    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "Vicky Sarra");
}
```

## 6。`EJBQLQuery`

接下来，让我们通过`EJBQLQuery,` 查询数据，它是在 Cayenne 中采用 Java 持久性 API 的实验的一部分。

这里，使用参数化的对象样式应用查询；让我们来看一些实际的例子。

首先，对所有已保存作者的搜索如下所示:

```java
@Test
void givenAuthors_whenFindAllEJBQL_thenWeGetThreeAuthors() {
    EJBQLQuery query = new EJBQLQuery("select a FROM Author a");
    List<Author> authors = context.performQuery(query);

    assertEquals(authors.size(), 3);
}
```

让我们用名字“Vicky Sarra”再次搜索作者，但现在用`EJBQLQuery`:

```java
@Test
void givenAuthors_whenFindByNameEJBQL_thenWeGetOneAuthor() {
    EJBQLQuery query = new EJBQLQuery(
      "select a FROM Author a WHERE a.name = 'Vicky Sarra'");
    List<Author> authors = context.performQuery(query);
    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "Vicky Sarra");
}
```

一个更好的例子是更新作者:

```java
@Test
void whenUpdadingByNameEJBQL_thenWeGetTheUpdatedAuthor() {
    EJBQLQuery query = new EJBQLQuery(
      "UPDATE Author AS a SET a.name "
      + "= 'Vicky Edison' WHERE a.name = 'Vicky Sarra'");
    QueryResponse queryResponse = context.performGenericQuery(query);

    EJBQLQuery queryUpdatedAuthor = new EJBQLQuery(
      "select a FROM Author a WHERE a.name = 'Vicky Edison'");
    List<Author> authors = context.performQuery(queryUpdatedAuthor);
    Author author = authors.get(0);

    assertNotNull(author);
}
```

如果我们只想选择一列，我们应该使用这个查询`“select a.name FROM Author a”`。关于 Github 的文章的源代码中有更多的例子。

## 7。`SQLExec`

`SQLExec`也是从 Cayenne 的 M4 版本引入的新的流畅查询 API。

一个简单的插入看起来像这样:

```java
@Test
void whenInsertingSQLExec_thenWeGetNewAuthor() {
    int inserted = SQLExec
      .query("INSERT INTO Author (name) VALUES ('Baeldung')")
      .update(context);

    assertEquals(inserted, 1);
}
```

接下来，我们可以根据作者的名字更新他:

```java
@Test
void whenUpdatingSQLExec_thenItsUpdated() {
    int updated = SQLExec.query(
      "UPDATE Author SET name = 'Baeldung' "
      + "WHERE name = 'Vicky Sarra'")
      .update(context);

    assertEquals(updated, 1);
}
```

我们可以从[文档](https://web.archive.org/web/20220625225228/https://cayenne.apache.org/docs/4.0/index.html)中获得更多细节。

## 8。结论

在本文中，我们研究了使用 Cayenne 编写简单和更高级查询的多种方法。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625225228/https://github.com/eugenp/tutorials/tree/master/persistence-modules/apache-cayenne)