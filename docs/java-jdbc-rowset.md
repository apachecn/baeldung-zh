# Java 中的 JDBC 行集接口介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jdbc-rowset>

## 1.概观

在本文中，我们将回顾 JDBC `RowSet`界面**。JDBC `RowSet`对象保存表格数据的方式比结果集更具适应性，使用起来也更简单。**

Oracle 为最常用的一个`RowSet:`定义了五个`RowSet`接口

*   `JdbcRowSet`
*   `CachedRowSet`
*   `WebRowSet`
*   `JoinRowSet`
*   `FilteredRowSet`

在本教程中，我们将回顾如何使用这些`RowSet`接口。

## 2.`JdbcRowSet`

让我们从`JdbcRowSet`开始——我们将简单地通过向`JdbcRowSetImpl`传递一个`Connection` 对象来创建一个:

```java
JdbcRowSet jdbcRS = new JdbcRowSetImpl(conn);
jdbcRS.setType(ResultSet.TYPE_SCROLL_INSENSITIVE);
String sql = "SELECT * FROM customers";
jdbcRS.setCommand(sql);
jdbcRS.execute();
jdbcRS.addRowSetListener(new ExampleListener());
while (jdbcRS.next()) {
    // each call to next, generates a cursorMoved event
    System.out.println("id = " + jdbcRS.getString(1));
    System.out.println("name = " + jdbcRS.getString(2));
}
```

在上面的例子中， `jdbcRs`不包含任何数据，直到我们用方法`setCommand` 定义了 SQL 语句，然后运行方法`execute`。

还要注意，为了执行事件处理，我们在`JdbcRowSet.`中添加了一个`RowSetListener`

`JdbcRowSet`不同于其他四个`RowSet`实现——因为**总是连接到数据库**,因此它最类似于 `ResultSet`对象。

## 3.`CachedRowSet`

一个`CachedRowSet`对象是唯一的，因为它可以在不连接到数据源的情况下运行。我们称之为“断开的`RowSet`对象”。

`CachedRowSet`之所以得名，是因为它将数据缓存在内存中，这样它就可以操作自己的数据，而不是存储在数据库中的数据。

由于`CachedRowSet`接口是所有断开连接的行集对象的**超级接口，我们下面查看的代码同样适用于`WebRowSe` t、`JoinRowSet`或`FilteredRowSe` t:**

```java
CachedRowSet crs = new CachedRowSetImpl();
crs.setUsername(username);
crs.setPassword(password);
crs.setUrl(url);
crs.setCommand(sql);
crs.execute();
crs.addRowSetListener(new ExampleListener());
while (crs.next()) {
    if (crs.getInt("id") == 1) {
        System.out.println("CRS found customer1 and will remove the record.");
        crs.deleteRow();
        break;
    }
}
```

## 4.`WebRowSet`

接下来，我们来看看`WebRowSet`。

这也是独一无二的，因为除了提供`CachedRowSet`对象的功能之外，**还可以将自身写入 XML 文档** t，并且还可以读取 XML 文档以将其自身转换回`WebRowSet`:

```java
WebRowSet wrs = new WebRowSetImpl();
wrs.setUsername(username);
wrs.setPassword(password);
wrs.setUrl(url);
wrs.setCommand(sql);
wrs.execute();
FileOutputStream ostream = new FileOutputStream("customers.xml");
wrs.writeXml(ostream);
```

使用 `writeXml`方法，我们将`WebRowSet`对象的当前状态写入 XML 文档。

通过向`writeXml`方法传递一个`OutputStream`对象，我们用字节而不是字符来写，这对于处理所有形式的数据非常有帮助。

## 5.`JoinRowSet`

让我们在内存中的`RowSet` 对象之间创建一个 SQL `JOIN` 。这很重要，因为它为我们节省了创建一个或多个连接的开销:

```java
CachedRowSetImpl customers = new CachedRowSetImpl();
// configuration of settings for CachedRowSet
CachedRowSetImpl associates = new CachedRowSetImpl();
// configuration of settings for this CachedRowSet            
JoinRowSet jrs = new JoinRowSetImpl();
jrs.addRowSet(customers,ID);
jrs.addRowSet(associates,ID);
```

因为添加到`JoinRowSet`对象的每个`RowSet`对象都需要一个匹配列，SQL `JOIN`所基于的列，所以我们在`addRowSet`方法中指定了`“id”` 。

注意，除了使用列名，我们还可以使用列号。

## 6.`FilteredRowSet`

最后，`FilteredRowSet` **让我们减少`RowSet`对象中可见的行数**，这样我们就可以只处理与我们正在做的事情相关的数据。

我们决定如何使用`Predicate`接口的实现来“过滤”数据:

```java
public class FilterExample implements Predicate {

    private Pattern pattern;

    public FilterExample(String regexQuery) {
        if (regexQuery != null && !regexQuery.isEmpty()) {
            pattern = Pattern.compile(regexQuery);
        }
    }

    public boolean evaluate(RowSet rs) {
        try {
            if (!rs.isAfterLast()) {
                String name = rs.getString("name");
                System.out.println(String.format(
                  "Searching for pattern '%s' in %s", pattern.toString(),
                  name));
                Matcher matcher = pattern.matcher(name);
                return matcher.matches();
            } else
                return false;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    // methods for handling errors
}
```

现在我们将滤镜应用于一个`FilteredRowSet`对象:

```java
RowSetFactory rsf = RowSetProvider.newFactory();
FilteredRowSet frs = rsf.createFilteredRowSet();
frs.setCommand("select * from customers");
frs.execute(conn);
frs.setFilter(new FilterExample("^[A-C].*"));

ResultSetMetaData rsmd = frs.getMetaData();
int columncount = rsmd.getColumnCount();
while (frs.next()) {
    for (int i = 1; i <= columncount; i++) {
        System.out.println(
          rsmd.getColumnLabel(i)
          + " = "
          + frs.getObject(i) + " ");
        }
    }
```

## 7.结论

这个快速教程涵盖了 JDK 中可用的`RowSet`接口的五个标准实现。

我们讨论了每个实现的配置，并提到了它们之间的差异。

正如我们指出的，只有一个`RowSet`实现是连接的`RowSet`对象——T2。另外四个是断开的`RowSet`物体。

和往常一样，这篇文章的完整代码可以在 Github 上找到[。](https://web.archive.org/web/20221206003849/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)