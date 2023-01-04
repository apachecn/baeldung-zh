# 使用 Spring 的 Oracle 连接池

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-oracle-connection-pooling>

## 1.概观

Oracle 是大型生产环境中最受欢迎的数据库之一。因此，作为 Spring 开发人员，必须使用这些数据库是很常见的。

在本教程中，我们将讨论如何进行这种集成。

## 2.数据库

当然，我们首先需要的是数据库。如果我们没有安装，我们可以从 [Oracle 数据库软件下载](https://web.archive.org/web/20220630003853/https://www.oracle.com/database/technologies/oracle-database-software-downloads.html)中获取并安装任何可用的数据库。但是如果我们不想做任何安装，我们也可以为 Docker 构建任何 [Oracle 数据库映像。](https://web.archive.org/web/20220630003853/https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance)

在这种情况下，我们将使用一个`Oracle Database 12c Release 2 (12.2.0.2)`标准版 [Docker 图像](https://web.archive.org/web/20220630003853/https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance#building-oracle-database-docker-install-images)。因此，这使我们不必在电脑上安装新软件。

## 3.连接池

现在，我们已经为传入的连接准备好了数据库。接下来，让我们学习一些在 Spring 中使用连接池的不同方法。

### 3.1.HikariCP

Spring 连接池最简单的方法是使用自动配置。`spring-boot-starter-jdbc`依赖项包括 [HikariCP](/web/20220630003853/https://www.baeldung.com/hikaricp) 作为首选池数据源。因此，如果我们看一下我们的`pom.xml`,我们会看到:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

对于我们来说， [`spring-boot-starter-data-jpa`](https://web.archive.org/web/20220630003853/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-data-jpa) 依赖项包含了`spring-boot-starter-jdbc`依赖项。

现在我们只需[将我们的配置](/web/20220630003853/https://www.baeldung.com/spring-boot-hikari)添加到`application.properties`文件中:

```java
# OracleDB connection settings
spring.datasource.url=jdbc:oracle:thin:@//localhost:11521/ORCLPDB1
spring.datasource.username=books
spring.datasource.password=books
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

# HikariCP settings
spring.datasource.hikari.minimumIdle=5
spring.datasource.hikari.maximumPoolSize=20
spring.datasource.hikari.idleTimeout=30000
spring.datasource.hikari.maxLifetime=2000000
spring.datasource.hikari.connectionTimeout=30000
spring.datasource.hikari.poolName=HikariPoolBooks

# JPA settings
spring.jpa.database-platform=org.hibernate.dialect.Oracle12cDialect
spring.jpa.hibernate.use-new-id-generator-mappings=false
spring.jpa.hibernate.ddl-auto=create
```

如您所见，我们有三种不同的部分配置设置:

*   `OracleDB connection settings`部分是我们像往常一样配置 JDBC 连接属性的地方
*   `HikariCP settings`部分是我们配置 HikariCP 连接池的地方。如果我们需要高级配置，我们应该检查 [HikariCP 配置属性列表](https://web.archive.org/web/20220630003853/https://github.com/brettwooldridge/HikariCP#configuration-knobs-baby)
*   `JPA settings`部分是使用 Hibernate 的一些基本配置

这就是我们所需要的。这再简单不过了，不是吗？

### 3.2.Tomcat 和 Commons DBCP2 连接池

[Spring 推荐 HikariCP 性能](https://web.archive.org/web/20220630003853/https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-connect-to-production-database)。**另一方面，它也支持 Spring Boot 自动配置应用程序中的 Tomcat 和 Commons DBCP2。**

它试图使用 HikariCP。如果它不可用，则尝试使用 Tomcat 池。如果这两个都不可用，那么它会尝试使用 Commons DBCP2。

我们还可以指定要使用的连接池。在这种情况下，我们只需要向我们的`application.properties`文件添加一个新属性:

```java
spring.datasource.type=org.apache.tomcat.jdbc.pool.DataSource
```

如果我们需要配置特定的设置，我们可以使用它们的前缀:

*   `spring.datasource.hikari.*`对于 HikariCP 配置
*   `spring.datasource.tomcat.*`对于 Tomcat 池配置
*   `spring.datasource.dbcp2.*`对于 Commons DBC2 配置

实际上，我们可以将`spring.` `datasource.type`设置为任何其他的`DataSource`实现。没有必要是上面提到的三种中的任何一种。

但是在这种情况下，我们将只有一个基本的开箱即用配置。在很多情况下，我们需要一些高级配置。让我们看看其中的一些。

### 3.3.Oracle 通用连接池

如果我们想使用高级配置，我们可以在 application.properties 文件中声明 UCP 数据源并设置其余的属性。从 UCP 版本 21.1.0.0 开始，这是最简单的方法。

[适用于 JDBC 的 Oracle 通用连接池(UCP)](https://web.archive.org/web/20220630003853/https://docs.oracle.com/en/database/oracle/oracle-database/21/jjucp/)为缓存 JDBC 连接提供了一个全功能的实施方案。它重用连接，而不是创建新的连接。它还为我们提供了一组用于定制池行为的属性。

如果我们想使用 UCP，我们需要添加以下 Maven 依赖项:

```java
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
</dependency>
<dependency>
    <groupId>com.oracle.database.ha</groupId>
    <artifactId>ons</artifactId>
</dependency>
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ucp</artifactId>
</dependency>
```

现在我们只需将我们的配置添加到`application.properties`文件中:

```java
# UCP settings
spring.datasource.type=oracle.oracleucp.jdbc.UCPDataSource
spring.datasource.oracleucp.connection-factory-class-name=oracle.jdbc.pool.OracleDataSource 
spring.datasource.oracleucp.sql-for-validate-connection=select * from dual 
spring.datasource.oracleucp.connection-pool-name=UcpPoolBooks 
spring.datasource.oracleucp.initial-pool-size=5 
spring.datasource.oracleucp.min-pool-size=5 
spring.datasource.oracleucp.max-pool-size=10
```

在上面的示例中，我们定制了一些池属性:

*   `spring.datasource.oracleucp.initial-pool-size`指定池启动后创建的可用连接数
*   `spring.datasource.oracleucp.min-pool-size`指定我们的池维护的可用和借用连接的最小数量，以及
*   `spring.datasource.oracleucp.max-pool-size`指定我们的池维护的可用和借用连接的最大数量

如果我们需要添加更多的配置属性，我们应该查看 [`UCPDataSource` JavaDoc](https://web.archive.org/web/20220630003853/https://docs.oracle.com/en/database/oracle/oracle-database/21/jjuar/oracle/ucp/jdbc/UCPDataSource.html) 或[开发者指南](https://web.archive.org/web/20220630003853/https://docs.oracle.com/en/database/oracle/oracle-database/21/jjucp/)。

## 4.旧版本的 Oracle

**对于 11.2 之前的版本，如 Oracle 9i 或 10g** ，我们应该创建一个`OracleDataSource`，而不是使用 Oracle 的通用连接池。

在我们的`OracleDataSource` 实例中，我们通过`setConnectionCachingEnabled`打开连接缓存:

```java
@Configuration
@Profile("oracle")
public class OracleConfiguration {
    @Bean
    public DataSource dataSource() throws SQLException {
        OracleDataSource dataSource = new OracleDataSource();
        dataSource.setUser("books");
        dataSource.setPassword("books");
        dataSource.setURL("jdbc:oracle:thin:@//localhost:11521/ORCLPDB1");
        dataSource.setFastConnectionFailoverEnabled(true);
        dataSource.setImplicitCachingEnabled(true);
        dataSource.setConnectionCachingEnabled(true);
        return dataSource;
    }
}
```

在上面的例子中，我们为连接池创建了`OracleDataSource`并配置了一些参数。我们可以在 [`OracleDataSource` JavaDoc](https://web.archive.org/web/20220630003853/https://docs.oracle.com/en/database/oracle/oracle-database/12.2/jajdb/oracle/jdbc/pool/OracleDataSource.html) 上检查所有可配置的参数。

## 5.结论

如今，使用 Spring 配置 Oracle 数据库连接池是小菜一碟。

我们已经看到了如何通过自动配置和编程来实现。尽管 Spring 推荐使用 HikariCP，但其他选项也是可用的。我们应该小心谨慎，选择适合我们当前需求的正确实现。

和往常一样，完整的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220630003853/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-2)