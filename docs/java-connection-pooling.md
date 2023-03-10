# Java 连接池的简单指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-connection-pooling>

## 1。概述

连接池是一种众所周知的数据访问模式。它的主要目的是减少执行数据库连接和读/写数据库操作的开销。

**在最基本的层面上，** **连接池是一个数据库连接缓存实现**，可以对其进行配置以满足特定的需求。

在本教程中，我们将讨论一些流行的连接池框架。然后我们将从头开始学习如何实现我们自己的连接池。

## 2。为什么是连接池？

当然，这个问题是反问句。

如果我们分析一个典型的数据库连接生命周期中涉及的一系列步骤，我们就会明白为什么:

1.  使用数据库驱动程序打开到数据库的连接
2.  打开 [TCP 套接字](https://web.archive.org/web/20221126232712/https://en.wikipedia.org/wiki/Network_socket)进行数据读写
3.  通过套接字读取/写入数据
4.  关闭连接
5.  关闭插座

很明显，**数据库连接是相当昂贵的操作**，因此，在每一个可能的用例中应该减少到最小(在边缘情况下，只是避免)。

这就是连接池实现发挥作用的地方。

通过简单地实现一个数据库连接容器，它允许我们重用一些现有的连接，我们可以有效地节省执行大量昂贵的数据库访问的成本。这提高了我们数据库驱动的应用程序的整体性能。

## 3。JDBC 连接池框架

从实用的角度来看，考虑到已经可用的“企业就绪”连接池框架的数量，从头实现连接池是没有意义的。

从说教的角度来看，这是这篇文章的目标，它不是。

尽管如此，在我们学习如何实现一个基本的连接池之前，我们将首先展示一些流行的连接池框架。

### 3.1。DBCP 阿帕奇社区

让我们从 Apache Commons DBCP 组件开始，这是一个全功能的连接池 JDBC 框架:

```java
public class DBCPDataSource {

    private static BasicDataSource ds = new BasicDataSource();

    static {
        ds.setUrl("jdbc:h2:mem:test");
        ds.setUsername("user");
        ds.setPassword("password");
        ds.setMinIdle(5);
        ds.setMaxIdle(10);
        ds.setMaxOpenPreparedStatements(100);
    }

    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    private DBCPDataSource(){ }
}
```

在这种情况下，我们使用一个带有静态块的包装类来轻松配置 DBCP 的属性。

下面是如何获得与`DBCPDataSource`类的池连接:

```java
Connection con = DBCPDataSource.getConnection();
```

### 3.2。hikarcp

现在让我们来看看 [HikariCP](https://web.archive.org/web/20221126232712/https://github.com/brettwooldridge/HikariCP) ，一个由 [Brett Wooldridge](https://web.archive.org/web/20221126232712/https://github.com/brettwooldridge) 创建的闪电般快速的 JDBC 连接池框架(关于如何配置和充分利用 HikariCP 的完整细节，请查看[这篇文章](/web/20221126232712/https://www.baeldung.com/hikaricp)):

```java
public class HikariCPDataSource {

    private static HikariConfig config = new HikariConfig();
    private static HikariDataSource ds;

    static {
        config.setJdbcUrl("jdbc:h2:mem:test");
        config.setUsername("user");
        config.setPassword("password");
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        ds = new HikariDataSource(config);
    }

    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    private HikariCPDataSource(){}
}
```

类似地，下面是如何获得与`HikariCPDataSource`类的池连接:

```java
Connection con = HikariCPDataSource.getConnection();
```

### 3.3。C3P0

这篇评论的最后一篇是由 Steve Waldman 开发的强大的 JDBC4 连接和语句池框架:

```java
public class C3p0DataSource {

    private static ComboPooledDataSource cpds = new ComboPooledDataSource();

    static {
        try {
            cpds.setDriverClass("org.h2.Driver");
            cpds.setJdbcUrl("jdbc:h2:mem:test");
            cpds.setUser("user");
            cpds.setPassword("password");
        } catch (PropertyVetoException e) {
            // handle the exception
        }
    }

    public static Connection getConnection() throws SQLException {
        return cpds.getConnection();
    }

    private C3p0DataSource(){}
}
```

正如所料，用`C3p0DataSource`类获得一个池连接类似于前面的例子:

```java
Connection con = C3p0DataSource.getConnection();
```

## 4。一个简单的实现

为了更好地理解连接池的底层逻辑，让我们创建一个简单的实现。

我们将从基于单一接口的松散耦合设计开始:

```java
public interface ConnectionPool {
    Connection getConnection();
    boolean releaseConnection(Connection connection);
    String getUrl();
    String getUser();
    String getPassword();
}
```

`ConnectionPool`接口定义了一个基本连接池的公共 API。

现在，让我们创建一个提供一些基本功能的实现，包括获取和释放池连接:

```java
public class BasicConnectionPool 
  implements ConnectionPool {

    private String url;
    private String user;
    private String password;
    private List<Connection> connectionPool;
    private List<Connection> usedConnections = new ArrayList<>();
    private static int INITIAL_POOL_SIZE = 10;

    public static BasicConnectionPool create(
      String url, String user, 
      String password) throws SQLException {

        List<Connection> pool = new ArrayList<>(INITIAL_POOL_SIZE);
        for (int i = 0; i < INITIAL_POOL_SIZE; i++) {
            pool.add(createConnection(url, user, password));
        }
        return new BasicConnectionPool(url, user, password, pool);
    }

    // standard constructors

    @Override
    public Connection getConnection() {
        Connection connection = connectionPool
          .remove(connectionPool.size() - 1);
        usedConnections.add(connection);
        return connection;
    }

    @Override
    public boolean releaseConnection(Connection connection) {
        connectionPool.add(connection);
        return usedConnections.remove(connection);
    }

    private static Connection createConnection(
      String url, String user, String password) 
      throws SQLException {
        return DriverManager.getConnection(url, user, password);
    }

    public int getSize() {
        return connectionPool.size() + usedConnections.size();
    }

    // standard getters
}
```

虽然很幼稚，但是`BasicConnectionPool`类提供了我们期望从典型的连接池实现中得到的最小功能。

简而言之，该类基于一个存储 10 个连接的`ArrayList`初始化一个连接池，这个连接池可以很容易地被重用。

**还可以创建与 [`DriverManager`类](https://web.archive.org/web/20221126232712/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/DriverManager.html)和[数据源](https://web.archive.org/web/20221126232712/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/javax/sql/DataSource.html)实现**的 JDBC 连接。

因为保持连接数据库的创建不可知要好得多，所以我们在`create()`静态工厂方法中使用前者。

在这种情况下，我们将方法放在了`BasicConnectionPool`中，因为这是接口的唯一实现。

在一个更复杂的设计中，有多个`ConnectionPool`实现，最好把它放在接口中，这样可以得到更灵活的设计和更高水平的内聚性。

这里需要强调的最相关的一点是，一旦创建了池，就会从池中获取**个连接，因此不需要创建新的**。

此外，**当一个连接被释放时，它实际上被返回到池中，因此其他客户机可以重用它**。

没有与底层数据库的进一步交互，比如对`Connection's close()`方法的显式调用。

## 5。使用`BasicConnectionPool`类

正如所料，使用我们的`BasicConnectionPool`类很简单。

让我们创建一个简单的单元测试，并获得一个内存池 [H2](https://web.archive.org/web/20221126232712/http://www.h2database.com/html/main.html) 连接:

```java
@Test
public whenCalledgetConnection_thenCorrect() {
    ConnectionPool connectionPool = BasicConnectionPool
      .create("jdbc:h2:mem:test", "user", "password");

    assertTrue(connectionPool.getConnection().isValid(1));
}
```

## 6。进一步的改进和重构

当然，还有足够的空间来调整/扩展我们的连接池实现的当前功能。

例如，我们可以重构`getConnection()`方法并增加对最大池大小的支持。如果所有可用连接都被占用，并且当前池大小小于配置的最大值，则该方法将创建一个新连接。

我们还可以在将从池中获得的连接传递给客户端之前，验证它是否仍处于活动状态:

```java
@Override
public Connection getConnection() throws SQLException {
    if (connectionPool.isEmpty()) {
        if (usedConnections.size() < MAX_POOL_SIZE) {
            connectionPool.add(createConnection(url, user, password));
        } else {
            throw new RuntimeException(
              "Maximum pool size reached, no available connections!");
        }
    }

    Connection connection = connectionPool
      .remove(connectionPool.size() - 1);

    if(!connection.isValid(MAX_TIMEOUT)){
        connection = createConnection(url, user, password);
    }

    usedConnections.add(connection);
    return connection;
} 
```

注意，该方法现在抛出了`SQLException`，这意味着我们也必须更新接口签名。

或者我们可以添加一个方法来优雅地关闭我们的连接池实例:

```java
public void shutdown() throws SQLException {
    usedConnections.forEach(this::releaseConnection);
    for (Connection c : connectionPool) {
        c.close();
    }
    connectionPool.clear();
}
```

在生产就绪的实现中，连接池应该提供一系列额外的功能，比如跟踪当前正在使用的连接的能力，对准备好的语句池的支持，等等。

为了使本文简单，我们将省略如何实现这些额外的特性，并且为了清楚起见，保持实现非线程安全。

## 7。结论

在本文中，我们深入了解了什么是连接池，并学习了如何实现我们自己的连接池。

当然，我们不必每次都要从零开始，向我们的应用程序添加一个全功能的连接池层。

这就是为什么我们从探索一些最流行的连接池框架开始，这样我们就清楚地知道如何使用它们，并选择最适合我们需求的一个。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221126232712/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)