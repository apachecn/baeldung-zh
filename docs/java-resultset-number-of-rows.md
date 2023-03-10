# 获取结果集中的行数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-resultset-number-of-rows>

## 1.概观

在这篇文章中，我们将看看我们可以用不同的方法**计算一个`ResultSet`**的行数。

## 2.计数`ResultSet`行

计算一个`ResultSet`的行数并不简单，因为没有 API 方法提供这个信息。这样做的原因是**一个 JDBC 查询不会立即获取所有的结果**。每次我们使用`ResultSet.next`方法请求行结果时，都会从数据库中加载它们。

当我们执行 JDBC 查询时，我们事先不知道会有多少结果。相反，我们需要遍历所有的行，只有当我们到达末尾时，我们才能确定可用的行数。

有两种方法可以做到这一点，要么使用标准的，要么使用可滚动的`ResultSet`。

## 3.标准`ResultSet`

统计查询结果最直接的方法是**遍历所有结果，并为每个结果增加一个计数器变量**。

让我们为数据库连接创建一个带有单个参数的`StandardRowCounter`类:

```java
class StandardRowCounter {
    Connection conn;

    StandardRowCounter(Connection conn) {
        this.conn = conn;
    }
}
```

我们的类将包含一个方法，该方法将一个 SQL 查询作为一个`String`并将通过迭代`ResultSet`返回行数，为每个结果递增一个计数器变量。

让我们将我们的计数器方法命名为`getQueryRowCount`:

```java
int getQueryRowCount(String query) throws SQLException {
    try (Statement statement = conn.createStatement();
        ResultSet standardRS = statement.executeQuery(query)) {
        int size = 0;
        while (standardRS.next()) {
            size++;
        }
        return size;
    }
}
```

注意，我们使用一个 [`try-with-resources`](/web/20220524064309/https://www.baeldung.com/java-try-with-resources) 块来自动关闭 JDBC 资源。

为了测试我们的实现，我们将利用一个[内存数据库](/web/20220524064309/https://www.baeldung.com/java-in-memory-databases)来快速生成一个包含 3 个条目的表。

有了这些，让我们用一个简单的`main`方法创建一个`RowCounterApp`:

```java
class RowCounterApp {

    public static void main(String[] args) throws SQLException {
        Connection conn = createDummyDB();

        String selectQuery = "SELECT * FROM STORAGE";

        StandardRowCounter standardCounter = new StandardRowCounter(conn);
        assert standardCounter.getQueryRowCount(selectQuery) == 3;
    }

    static Connection createDummyDB() throws SQLException {
        ...
    }

}
```

上述方法**将在任何数据库**中工作。但是，如果数据库驱动程序支持，我们可以使用一些更高级的 API 来达到同样的效果。

## 4.可滚动`ResultSet`

通过使用重载的`Statement` 方法`createStatement,`,我们可以要求在查询执行后创建一个可滚动的`ResultSet`。使用可滚动版本，我们可以使用更高级的遍历方法，如`previous`来向后移动。在我们的例子中，我们将使用`last`方法移动到`Result`T5 的末尾，并获得最后一个条目的行号，这是由`getRow`方法给出的。

让我们创建一个`ScrollableRowCounter`类:

```java
class ScrollableRowCounter {
    Connection conn;

    ScrollableRowCounter(Connection conn) {
        this.conn = conn;
    }
}
```

和我们的`StandardRowCounter`一样，我们将使用的唯一字段是数据库*连接*。

同样，我们将使用一个`getQueryRowCount`方法:

```java
int getQueryRowCount(String query) throws SQLException {
    try (Statement statement = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
        ResultSet scrollableRS = statement.executeQuery(query)) {
        scrollableRS.last();
        return scrollableRS.getRow();
    }
}
```

为了获得可滚动的`ResultSet,`，我们必须向`createStatement`方法提供`ResultSet.TYPE_SCROLL_INSENSITIVE`常量。此外，我们必须为并发模式提供一个值，但是因为它与我们的情况无关，所以我们使用默认的`ResultSet.` CONCUR_READ_ONLY 常量。如果 JDBC 驱动程序不支持这种操作模式，它会抛出一个异常。

让我们用`RowCountApp`测试我们的新实现:

```java
ScrollableRowCounter scrollableCounter = new ScrollableRowCounter(conn);
assert scrollableCounter.getQueryRowCount(selectQuery) == 3;
```

## 5.性能考虑因素

虽然上面的实现很简单，**但是由于对`ResultSet`的强制遍历，它们没有最好的性能**。出于这个原因，通常建议对行数操作使用`COUNT`类型的查询。

一个简单的例子是:

`SELECT COUNT(*) FROM STORAGE`

这将返回一行一列，其中包含`STORAGE`表中的行数。

## 6.结论

在本文中，我们了解了获取`ResultSet`中行数的不同方法。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524064309/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)