# 不同数据库的 JDBC URL 格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jdbc-url-format>

## 1.概观

当我们在 Java 中使用数据库时，通常我们用 [JDBC](/web/20221208143917/https://www.baeldung.com/java-jdbc) 连接到数据库。

JDBC URL 是在 Java 应用程序和数据库之间建立连接的一个重要参数。但是，对于不同的数据库系统，JDBC URL 格式可能会有所不同。

在本教程中，我们将仔细研究几种广泛使用的数据库的 JDBC URL 格式: [Oracle](https://web.archive.org/web/20221208143917/https://www.oracle.com/database/technologies/) 、 [MySQL](https://web.archive.org/web/20221208143917/https://www.mysql.com/) 、[微软 SQL Server](https://web.archive.org/web/20221208143917/https://www.microsoft.com/en-us/sql-server/sql-server-2019) 和 [PostgreSQL](https://web.archive.org/web/20221208143917/https://www.postgresql.org/) 。

## 2.Oracle 的 JDBC URL 格式

Oracle 数据库系统广泛应用于企业 Java 应用程序中。在我们查看连接 Oracle 数据库的 JDBC URL 的格式之前，我们应该首先确保 Oracle 瘦数据库驱动程序在我们的类路径中。

例如，如果我们的项目由 Maven 管理，我们需要在我们的`pom.xml`中添加 [`ojdbc8` 依赖关系](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:com.oracle.database.jdbc%20AND%20a:ojdbc8):

```
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>21.1.0.0</version>
</dependency> 
```

瘦驱动程序提供了几种 JDBC URL 格式:

*   连接到 [SID](https://web.archive.org/web/20221208143917/https://docs.oracle.com/cd/E11882_01/network.112/e41945/glossary.htm#BGBFBBAI)
*   连接到 Oracle [服务名](https://web.archive.org/web/20221208143917/https://docs.oracle.com/cd/E11882_01/network.112/e41945/glossary.htm#BGBGIHFG)
*   带 [`tnsnames.ora`](https://web.archive.org/web/20221208143917/https://docs.oracle.com/database/121/NETRF/tnsnames.htm#NETRF007) 条目的网址

接下来，我们将逐一介绍这些格式。

### 2.1.连接到 Oracle 数据库 SID

在一些早期版本的 Oracle 数据库中，数据库被定义为一个 SID。让我们看看连接到 SID 的 JDBC URL 格式:

```
jdbc:oracle:thin:[<user>/<password>]@<host>[:<port>]:<SID> 
```

例如，假设我们有一个 Oracle 数据库服务器主机“`myoracle.db.server:1521`”，SID 的名称是“`my_sid`”，我们可以按照上面的格式构建连接 URL 并连接到数据库:

```
@Test
public void givenOracleSID_thenCreateConnectionObject() {
    String oracleJdbcUrl = "jdbc:oracle:thin:@myoracle.db.server:1521:my_sid";
    String username = "dbUser";
    String password = "1234567";
    try (Connection conn = DriverManager.getConnection(oracleJdbcUrl, username, password)) {
        assertNotNull(conn);
    } catch (SQLException e) {
        System.err.format("SQL State: %s\n%s", e.getSQLState(), e.getMessage());
    }
}
```

### 2.2.连接到 Oracle 数据库服务名

通过服务名连接 Oracle 数据库的 JDBC URL 的格式与我们用来通过 SID 连接的格式非常相似:

```
jdbc:oracle:thin:[<user>/<password>]@//<host>[:<port>]/<service>
```

我们可以连接到 Oracle 数据库服务器“`myoracle.db.server:1521`”上的服务“`my_servicename`”:

```
@Test
public void givenOracleServiceName_thenCreateConnectionObject() {
    String oracleJdbcUrl = "jdbc:oracle:thin:@//myoracle.db.server:1521/my_servicename";
    ...
    try (Connection conn = DriverManager.getConnection(oracleJdbcUrl, username, password)) {
        assertNotNull(conn);
        ...
    }
    ...
} 
```

### 2.3.使用`tnsnames.ora`条目连接到 Oracle 数据库

我们还可以在 JDBC URL 中包含`tnsnames.ora`条目来连接 Oracle 数据库:

```
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<host>)(PORT=<port>))(CONNECT_DATA=(SERVICE_NAME=<service>)))
```

让我们看看如何使用来自`tnsnames.ora`文件的条目连接到我们的"`my_servicename`"服务:

```
@Test
public void givenOracleTnsnames_thenCreateConnectionObject() {
    String oracleJdbcUrl = "jdbc:oracle:thin:@" +
      "(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)" +
      "(HOST=myoracle.db.server)(PORT=1521))" +
      "(CONNECT_DATA=(SERVICE_NAME=my_servicename)))";
    ...
    try (Connection conn = DriverManager.getConnection(oracleJdbcUrl, username, password)) {
        assertNotNull(conn);
        ...
    }
    ...
}
```

## 3.MySQL 的 JDBC URL 格式

在这一节中，让我们讨论如何编写连接到 MySQL 数据库的 JDBC URL。

为了从我们的 Java 应用程序连接到 MySQL 数据库，让我们首先在我们的`pom.xml`中添加 JDBC 驱动程序 [`mysql-connector-java`依赖关系](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=a:mysql-connector-java%20g:mysql):

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.22</version>
</dependency> 
```

接下来，让我们看看 MySQL JDBC 驱动程序支持的连接 URL 的通用格式:

```
protocol//[hosts][/database][?properties]
```

让我们看一个连接到主机“`mysql.db.server`”上的 MySQL 数据库“`my_database`”的示例:

```
@Test
public void givenMysqlDb_thenCreateConnectionObject() {
    String jdbcUrl = "jdbc:mysql://mysql.db.server:3306/my_database?useSSL=false&serverTimezone;=UTC";    
    String username = "dbUser";
    String password = "1234567";
    try (Connection conn = DriverManager.getConnection(jdbcUrl, username, password)) {
        assertNotNull(conn);
    } catch (SQLException e) {
        System.err.format("SQL State: %s\n%s", e.getSQLState(), e.getMessage());
    }
} 
```

上面例子中的 JDBC 网址看起来很简单。它有四个组成部分:

*   `protocol`–`jdbc:mysql:`
*   `host` –`mysql.db.server:3306`
*   `database`–`my_database`
*   `properties`–`useSSL=false&serverTimezone;=UTC`

但是，有时候，我们可能会面临更复杂的情况，比如不同类型的连接或者多个 MySQL 主机等等。

接下来，我们将仔细看看每个构造块。

### 3.1.草案

**除了普通的“`jdbc:mysql:`”协议，`connector-java` JDBC 驱动仍然支持一些特殊连接的协议:**

*   [负载平衡 JDBC 连接](https://web.archive.org/web/20221208143917/https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-usagenotes-j2ee-concepts-managing-load-balanced-connections.html)–`jdbc:mysql:loadbalance:`
*   [JDBC 复制连接](https://web.archive.org/web/20221208143917/https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-source-replica-replication-connection.html)–`jdbc:mysql:replication: `

当我们谈到负载平衡和 JDBC 复制时，我们可能会意识到应该有多个 MySQL 主机。

接下来，我们来看看连接 URL 的另一部分— `hosts`的细节。

### 3.2.主机

在上一节中，我们已经看到了定义单个主机的 JDBC URL 示例—例如，`mysql.db.server:3306.`

但是，**如果我们需要处理多个主机，我们可以用逗号分隔的列表列出主机:`host1, host2,…,hostN`。**

**我们也可以用方括号将逗号分隔的主机列表括起来:`[host1, host2,…,hostN]`。**

让我们来看几个连接到多个 MySQL 服务器的 JDBC URL 示例:

*   `jdbc:mysql://myhost1:3306,myhost2:3307/db_name`
*   `jdbc:mysql://[myhost1:3306,myhost2:3307]/db_name`
*   `jdbc:mysql:loadbalance://myhost1:3306,myhost2:3307/db_name?user=dbUser&password;=1234567&loadBalanceConnectionGroup;=group_name&ha.enableJMX;=true`

如果我们仔细看看上面的最后一个例子，我们会发现在数据库名称之后，有一些属性和用户凭证的定义。我们接下来会看这些。

### 3.3.属性和用户凭据

**有效的全局属性将应用于所有主机。属性前面有一个问号“`?`”，并写成由“`&`”****符号**分隔的`key=value`对:

```
jdbc:mysql://myhost1:3306/db_name?prop1=value1&prop2;=value2
```

**我们也可以将用户凭证放在属性列表**中:

```
jdbc:mysql://myhost1:3306/db_name?user=root&password;=mypass
```

此外，**我们可以用格式为“`user:[[email protected]](/web/20221208143917/https://www.baeldung.com/cdn-cgi/l/email-protection)`”、**的用户凭证作为每个主机的前缀:

```
jdbc:mysql://root:[[email protected]](/web/20221208143917/https://www.baeldung.com/cdn-cgi/l/email-protection):3306/db_name
```

此外，**如果我们的 JDBC URL 包含主机列表，并且所有主机都使用相同的用户凭证，我们可以在主机列表前面加上前缀**:

```
jdbc:mysql://root:mypass[myhost1:3306,myhost2:3307]/db_name
```

毕竟，**也有可能在 JDBC URL** 之外提供用户凭证。

**当我们调用 [`DriverManager.getConnection(String url, String user, String password)`](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/DriverManager.html#getConnection(java.lang.String,java.lang.String,java.lang.String)) 方法来获得连接时，我们可以将用户名和密码传递给该方法。**

## 4.Microsoft SQL Server 的 JDBC URL 格式

Microsoft SQL Server 是另一种流行的数据库系统。要从 Java 应用程序连接 MS SQL Server 数据库，我们需要将 [`mssql-jdbc`依赖关系](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:com.microsoft.sqlserver%20a:mssql-jdbc)添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>8.4.1.jre11</version>
</dependency>
```

接下来，让我们看看如何构建 JDBC URL 来获得到 MS SQL Server 的连接。

用于连接 MS SQL Server 数据库的 JDBC URL 的一般格式为:

```
jdbc:sqlserver://[serverName[\instanceName][:portNumber]][;property=value[;property=value]]
```

让我们仔细看看格式的每个部分。

*   我们将要连接的服务器的地址；这可能是指向服务器的域名或 IP 地址
*   `instanceName –`在`serverName`上要连接的实例；这是一个可选字段，如果没有指定该字段，将选择默认实例
*   `portNumber`–这是在`serverName`上连接的端口(默认端口为`1433`)
*   `properties`–可以包含一个或多个可选的连接属性，这些属性必须用分号分隔，并且不允许有重复的属性名

现在，假设我们有一个运行在主机“`mssql.db.server`”上的 MS SQL Server 数据库，服务器上的`instanceName`是“`mssql_instance`”，我们要连接的数据库的名称是“`my_database`”。

让我们尝试获得到该数据库的连接:

```
@Test
public void givenMssqlDb_thenCreateConnectionObject() {
    String jdbcUrl = "jdbc:sqlserver://mssql.db.server\\mssql_instance;databaseName=my_database";
    String username = "dbUser";
    String password = "1234567";
    try (Connection conn = DriverManager.getConnection(jdbcUrl, username, password)) {
        assertNotNull(conn);
    } catch (SQLException e) {
        System.err.format("SQL State: %s\n%s", e.getSQLState(), e.getMessage());
    }
} 
```

## 5.PostgreSQL 的 JDBC URL 格式

PostgreSQL 是一个流行的开源数据库系统。为了使用 PostgreSQL，JDBC 驱动程序 [`postgresql`](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:org.postgresql%20AND%20a:postgresql) 应该作为一个依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.18</version>
</dependency>
```

连接到 PostgreSQL 的 JDBC URL 的一般形式是:

```
jdbc:postgresql://host:port/database?properties
```

现在，让我们来看看上述 JDBC 网址格式的每一部分。

`host`参数是数据库服务器的域名或 IP 地址。

**如果我们想指定一个 IPv6 地址，`host`参数必须用方括号括起来，例如`jdbc:postgresql://[::1]:5740/my_database`。mysql**

`port`参数指定 PostgreSQL 正在监听的端口号。**port 参数可选，默认端口号为`5432`。**

顾名思义，`database`参数定义了我们想要连接的数据库的名称。

`properties`参数可以包含一组由“`&`符号分隔的`key=value`对。

理解了 JDBC URL 格式中的参数后，让我们看一个如何获得到 PostgreSQL 数据库的连接的示例:

```
@Test
public void givenPostgreSqlDb_thenCreateConnectionObject() {
    String jdbcUrl = "jdbc:postgresql://postgresql.db.server:5430/my_database?ssl=true&loglevel;=2";
    String username = "dbUser";
    String password = "1234567";
    try (Connection conn = DriverManager.getConnection(jdbcUrl, username, password)) {
        assertNotNull(conn);
    } catch (SQLException e) {
        System.err.format("SQL State: %s\n%s", e.getSQLState(), e.getMessage());
    }
} 
```

在上面的示例中，我们通过以下方式连接到 PostgreSQL 数据库:

*   `host:port – postgresql.db.server:5430`
*   `database`–`my_database`
*   属性—`ssl=true&loglevel;=2`

## 6.结论

本文讨论了四种广泛使用的数据库系统的 JDBC URL 格式:Oracle、MySQL、Microsoft SQL Server 和 PostgreSQL。

我们还看到了构建 JDBC URL 字符串来连接这些数据库的不同例子。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)