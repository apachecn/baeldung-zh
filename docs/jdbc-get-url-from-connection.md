# 从 JDBC 连接对象获取数据库 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-get-url-from-connection>

## 1.概观

在这个快速教程中，我们将讨论如何从 JDBC `Connection`对象获取数据库 URL。

## 2.示例类

为了演示这一点，我们将使用方法`getConnection`创建一个`DBConfiguration`类:

```java
public class DBConfiguration {

    public static Connection getConnection() throws Exception {
        Class.forName("org.h2.Driver");
        String url = "jdbc:h2:mem:testdb";
        return DriverManager.getConnection(url, "user", "password");
    }
}
```

## 3.`DatabaseMetaData#getURL`法

我们可以通过使用`[DatabaseMetaData#getURL](https://web.archive.org/web/20220524020832/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/DatabaseMetaData.html#getURL())`方法获得数据库 URL:

```java
@Test
void givenConnectionObject_whenExtractMetaData_thenGetDbURL() throws Exception {
    Connection connection = DBConfiguration.getConnection();
    String dbUrl = connection.getMetaData().getURL();
    assertEquals("jdbc:h2:mem:testdb", dbUrl);
}
```

在上面的例子中，我们首先获得了`Connection`实例。

然后，我们调用我们的`Connection`上的`getMetaData`方法来得到 [`DatabaseMetaData`](/web/20220524020832/https://www.baeldung.com/jdbc-database-metadata#databasemetadata-interface) 。

最后，我们在 [`DatabaseMetaData`](/web/20220524020832/https://www.baeldung.com/jdbc-database-metadata#databasemetadata-interface) 实例上调用`getURL`方法。正如我们所料，它返回我们数据库的 URL。

## 4.结论

在本教程中，我们已经看到了如何从 JDBC `Connection`对象中获取数据库 URL。

和往常一样，这个例子的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524020832/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)