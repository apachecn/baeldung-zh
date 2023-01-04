# 在 JDBC 返回生成的密钥

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-returning-generated-keys>

## 1.概观

在这个快速教程中，我们将看到如何用纯 JDBC 获得最后的自动生成的密钥。

## 2.设置

为了能够执行 SQL 查询，我们将使用内存中的 [H2](https://web.archive.org/web/20220524124950/https://search.maven.org/artifact/com.h2database/h2) 数据库。

第一步，让我们添加它的 Maven 依赖项:

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

此外，我们将使用一个只有两列的非常简单的表:

```
public class JdbcInsertIdIntegrationTest {

    private static Connection connection;

    @BeforeClass
    public static void setUp() throws Exception {
        connection = DriverManager.getConnection("jdbc:h2:mem:generated-keys", "sa", "");
        connection
          .createStatement()
          .execute("create table persons(id bigint auto_increment, name varchar(255))");
    }

    @AfterClass
    public static void tearDown() throws SQLException {
        connection
          .createStatement()
          .execute("drop table persons");
        connection.close();
    }

    // omitted
}
```

这里，我们连接到`generated-keys` 内存数据库，并在其中创建一个名为`persons` 的表。

## 3.返回生成的密钥标志

**在自动生成之后获取密钥的一种方法是将`[Statement.RETURN_GENERATED_KEYS](https://web.archive.org/web/20220524124950/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Statement.html#RETURN_GENERATED_KEYS) `传递给`[prepareStatement()](https://web.archive.org/web/20220524124950/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html#prepareStatement(java.lang.String,int)) `方法:**

```
String QUERY = "insert into persons (name) values (?)";
try (PreparedStatement statement = connection.prepareStatement(QUERY, Statement.RETURN_GENERATED_KEYS)) {
    statement.setString(1, "Foo");
    int affectedRows = statement.executeUpdate();
    assertThat(affectedRows).isPositive();

    // omitted
} catch (SQLException e) {
    // handle the database related exception appropriately
}
```

在准备和执行查询之后，我们可以调用`PreparedStatement `上的`[getGeneratedKeys()](https://web.archive.org/web/20220524124950/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Statement.html#getGeneratedKeys()) `方法来获得 id:

```
try (ResultSet keys = statement.getGeneratedKeys()) {
    assertThat(keys.next()).isTrue();
    assertThat(keys.getLong(1)).isGreaterThanOrEqualTo(1);
}
```

如上图，我们首先调用`next() `方法来移动结果光标。然后我们使用`getLong() `方法获取第一列，同时将其转换为`long `。

此外，也可以对普通的`Statement` s 使用相同的技术:

```
try (Statement statement = connection.createStatement()) {
    String query = "insert into persons (name) values ('Foo')";
    int affectedRows = statement.executeUpdate(query, Statement.RETURN_GENERATED_KEYS);
    assertThat(affectedRows).isPositive();

    try (ResultSet keys = statement.getGeneratedKeys()) {
        assertThat(keys.next()).isTrue();
        assertThat(keys.getLong(1)).isGreaterThanOrEqualTo(1);
    }
}
```

此外，值得一提的是，我们广泛使用了`[try-with-resources](/web/20220524124950/https://www.baeldung.com/java-try-with-resources) `来让编译器在我们之后进行清理。

## 4.返回列

事实证明，**我们还可以要求 JDBC 在发出查询**后返回特定的列。为此，我们只需传递一组列名:

```
try (PreparedStatement statement = connection.prepareStatement(QUERY, new String[] { "id" })) {
    statement.setString(1, "Foo");
    int affectedRows = statement.executeUpdate();
    assertThat(affectedRows).isPositive();

    // omitted
}
```

如上所示，我们告诉 JDBC 在执行给定查询后返回`id `列的值。类似于前面的例子，我们可以在之后获取`id `:

```
try (ResultSet keys = statement.getGeneratedKeys()) {
    assertThat(keys.next()).isTrue();
    assertThat(keys.getLong(1)).isGreaterThanOrEqualTo(1);
}
```

我们也可以对简单的`Statement`使用相同的方法:

```
try (Statement statement = connection.createStatement()) {
    int affectedRows = statement.executeUpdate("insert into persons (name) values ('Foo')", 
      new String[] { "id" });
    assertThat(affectedRows).isPositive();

    try (ResultSet keys = statement.getGeneratedKeys()) {
        assertThat(keys.next()).isTrue();
        assertThat(keys.getLong(1)).isGreaterThanOrEqualTo(1);
    }
}
```

## 5.结论

在这个快速教程中，我们看到了如何使用纯 JDBC 在查询执行后获取生成的键。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524124950/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)