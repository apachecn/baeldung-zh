# JDBC 连接状态

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-connection-status>

## 1.概观

在本文中，我们将讨论 [JDBC](/web/20220524062622/https://www.baeldung.com/java-jdbc) 连接状态的一些方面。首先，我们将了解连接丢失的最常见原因。然后，我们将学习**如何确定连接状态**。

我们还将学习**如何在运行 SQL 语句**之前验证连接。

## 2.JDBC 连接

*连接*类负责与数据源通信。连接可能由于各种原因而丢失:

*   数据库服务器已关闭
*   网络连接
*   重用关闭的连接

**在连接丢失时运行任何数据库操作都会导致`SQLException`** 。此外，我们可以检查异常，以获得有关该问题的详细信息。

## 3.检查连接

有不同的方法来检查连接。我们将看看这些方法，以决定何时使用它们。

### 3.1.连接状态

**我们可以使用`isClosed()`方法**检查`Connection`状态。使用这种方法，不能授予 SQL 操作。但是，检查连接是否打开是有帮助的。

让我们在运行 SQL 语句之前创建一个状态条件:

```java
public static void runIfOpened(Connection connection) throws SQLException
{
    if (connection != null && !connection.isClosed()) {
        // run sql statements
    } else {
        // handle closed connection path
    }
}
```

### 3.2.连接验证

即使连接已打开，也可能由于上一节中描述的原因而丢失。因此，在运行任何 SQL 语句之前，可能需要**验证连接。**

从版本 1.6 `,`开始，`Connection`类提供了一个验证方法。**首先，它向数据库提交一个验证查询。其次，它使用`timeout` 参数作为操作的阈值。最后，如果在`timeout`内操作成功，连接被标记为有效。**

让我们看看在运行任何语句之前如何验证连接:

```java
public static void runIfValid(Connection connection)
        throws SQLException
{
    if (connection.isValid(5)) {
        // run sql statements
    }
    else {
        // handle invalid connection
    }
}
```

在这种情况下，`timeout`是 5 秒。零值表示超时不适用于验证。另一方面，小于零的值将抛出一个`SQLException`。

### 3.3.自定义验证

有充分的理由让**创建一个定制的验证方法**。例如，我们可以使用没有验证方法的遗留 JDBC。类似地，我们的项目可能需要一个定制的验证查询在所有语句之前运行。

让我们创建一个方法来运行预定义的验证查询:

```java
public static boolean isConnectionValid(Connection connection)
{
    try {
        if (connection != null && !connection.isClosed()) {
            // Running a simple validation query
            connection.prepareStatement("SELECT 1");
            return true;
        }
    }
    catch (SQLException e) {
        // log some useful data here
    }
    return false;
}
```

**首先，该方法检查连接状态。第二，它尝试运行验证查询，成功后返回`true`。最后，如果验证查询没有运行或失败，它将返回`false`。**

现在，我们可以在运行任何语句之前使用自定义验证:

```java
public static void runIfConnectionValid(Connection connection)
{
    if (isConnectionValid(connection)) {
        // run sql statements
    }
    else {
        // handle invalid connection
    }
}
```

当然，运行一个简单的查询是验证数据库连通性的一个好选择。然而，**根据目标驱动程序和数据库**，还有其他有用的方法:

*   自动提交–使用`connection.getAutocommit()`和`connection.setAutocommit()`
*   元数据–使用`connection.getMetaData()`

## 4.连接池

就资源而言，数据库连接是昂贵的。**C**连接池是管理和配置这些连接的好策略`.` 简而言之，它们可以降低连接生命周期的成本。

所有的 [Java 连接池](/web/20220524062622/https://www.baeldung.com/java-connection-pooling)框架都有自己的连接验证实现。此外，它们中的大多数使用可参数化的验证查询。

以下是一些最流行的框架:

*   DBCP 阿帕奇社区-`validationQuery, validationQueryTimeout`
*   光 CP-`connectionTestQuery, validationTimeout`
*   C3 P0—`preferredTestQuery`

## 5.结论

在本文中，我们了解了 JDBC 连接状态的基础知识。我们回顾了`Connection` 类的一些有用的方法。之后，我们描述了一些在运行 SQL 语句之前验证连接的替代方法。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524062622/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)