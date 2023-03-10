# Statement 和 PreparedStatement 之间的差异

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-statement-preparedstatement>

## 1。概述

在本教程中，我们将探讨 [JDBC](/web/20220626082541/https://www.baeldung.com/java-jdbc) 的 `Statement` 和 `PreparedStatement` 接口的区别。我们不会讨论`CallableStatement`，一个用于执行存储过程的 JDBC API 接口 。

## 2。JDBC API 接口

` Statement`和 `PreparedStatement` 都可以用来执行 SQL 查询。这些接口看起来非常相似。然而，它们在特性和性能上有很大的不同:

*   `Statement`–**用于执行基于字符串的 SQL** **查询**
*   `PreparedStatement`–**用于执行参数化的 SQL 查询**

为了能够使用 `Statement` 和 `PreparedStatement` 在我们的示例中， 我们将把`h2` JDBC 连接器声明为我们的`pom.xml`文件中的依赖项:

```java
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <version>1.4.200</version>
</dependency>
```

让我们定义一个贯穿本文的实体:

```java
public class PersonEntity {
    private int id;
    private String name;

    // standard setters and getters
}
```

## 3。`Statement`

首先， `Statement` 接口 接受字符串作为 SQL 查询。因此，当我们连接 SQL 字符串:时，**代码变得可读性更差**

```java
public void insert(PersonEntity personEntity) {
    String query = "INSERT INTO persons(id, name) VALUES(" + personEntity.getId() + ", '"
      + personEntity.getName() + "')";

    Statement statement = connection.createStatement();
    statement.executeUpdate(query);
}
```

其次，**它容易受到**的攻击。接下来的例子说明了这个弱点。

在第一行中，update 会将所有行上的列“`name`”设置为“`hacker`”，因为“—”之后的任何内容都被解释为 SQL 中的注释，update 语句的条件将被忽略。在第二行中，insert 将失败，因为“`name`”列上的引号没有被转义:

```java
dao.update(new PersonEntity(1, "hacker' --"));
dao.insert(new PersonEntity(1, "O'Brien"))
```

第三， **JDBC 将带有内联值的查询传递给数据库**。因此，没有查询优化，最重要的是，**数据库引擎必须确保所有的检查**。此外，该查询对数据库来说不会显示为相同，并且**它将阻止缓存使用**。同样，批量更新需要单独执行:

```java
public void insert(List<PersonEntity> personEntities) {
    for (PersonEntity personEntity: personEntities) {
        insert(personEntity);
    }
}
```

第四，****`Statement`界面适用于 DDL 查询类似 创建 ， 修改 ， 删除** :**

```java
public void createTables() {
    String query = "create table if not exists PERSONS (ID INT, NAME VARCHAR(45))";
    connection.createStatement().executeUpdate(query);
}
```

最后，****`Statement`****接口不能用于存储和检索文件和数组**。**

 **## 4。`PreparedStatement`

首先， `PreparedStatement` 扩展了`Statement`接口。它有绑定各种对象类型的方法，包括文件和数组。因此，**代码变得** **易于理解** :

```java
public void insert(PersonEntity personEntity) {
    String query = "INSERT INTO persons(id, name) VALUES( ?, ?)";

    PreparedStatement preparedStatement = connection.prepareStatement(query);
    preparedStatement.setInt(1, personEntity.getId());
    preparedStatement.setString(2, personEntity.getName());
    preparedStatement.executeUpdate();
}
```

其次，**它通过对提供的所有参数值文本进行转义来防范 SQL 注入**:

```java
@Test 
void whenInsertAPersonWithQuoteInText_thenItNeverThrowsAnException() {
    assertDoesNotThrow(() -> dao.insert(new PersonEntity(1, "O'Brien")));
}

@Test 
void whenAHackerUpdateAPerson_thenItUpdatesTheTargetedPerson() throws SQLException {

    dao.insert(Arrays.asList(new PersonEntity(1, "john"), new PersonEntity(2, "skeet")));
    dao.update(new PersonEntity(1, "hacker' --"));

    List<PersonEntity> result = dao.getAll();
    assertEquals(Arrays.asList(
      new PersonEntity(1, "hacker' --"), 
      new PersonEntity(2, "skeet")), result);
}
```

第三， **`PreparedStatement`** **采用预编译**。数据库一收到查询，就会在预编译查询之前检查缓存。因此，**如果它没有被缓存，数据库引擎将保存它供下次使用。**

此外，**这个特性通过非 SQL 二进制协议加速了数据库和 JVM** 之间的通信。也就是说，数据包中的数据更少，因此服务器之间的通信速度更快。

第四， **`PreparedStatement`** **提供了一个[批处理](/web/20220626082541/https://www.baeldung.com/jdbc-batch-processing)执行期间的单个数据库连接**。让我们来看看实际情况:

```java
public void insert(List<PersonEntity> personEntities) throws SQLException {
    String query = "INSERT INTO persons(id, name) VALUES( ?, ?)";
    PreparedStatement preparedStatement = connection.prepareStatement(query);
    for (PersonEntity personEntity: personEntities) {
        preparedStatement.setInt(1, personEntity.getId());
        preparedStatement.setString(2, personEntity.getName());
        preparedStatement.addBatch();
    }
    preparedStatement.executeBatch();
}
```

接下来，**`PreparedStatement`****通过使用`BLOB`和`CLOB`数据类型**提供了一种存储和检索文件的简单方法。同样，通过将 `java.sql.Array` 转换为 SQL 数组来存储列表也很有帮助。

最后， `PreparedStatement` 实现类似于 `getMetadata()` 的方法，这些方法包含关于返回结果的信息。

## 5。结论

在本教程中，我们介绍了`PreparedStatement`和`Statement`的主要区别。两个接口都提供了执行 SQL 查询的方法，但是对于 DDL 查询使用`Statement`和对于 DML 查询使用`PreparedStatement`更合适。

像往常一样，GitHub 上的所有代码示例[都可用。](https://web.archive.org/web/20220626082541/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)****