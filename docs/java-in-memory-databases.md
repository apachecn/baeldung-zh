# 内存数据库列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-in-memory-databases>

## 1。概述

内存数据库依靠系统内存而不是磁盘空间来存储数据。因为内存访问比磁盘访问快，所以这些数据库自然更快。

当然，我们只能在不需要持久化数据或者为了更快地执行测试的应用程序和场景中使用内存数据库。它们通常作为嵌入式数据库运行，这意味着它们在流程开始时创建，在流程结束时丢弃，这对于测试来说非常方便，因为您不需要设置外部数据库。

在接下来的章节中，**我们将看看 Java 环境中一些最常用的内存数据库，以及每个数据库所需的配置**。

## 2。H2 数据库

`H2`是一个用 Java 编写的开源数据库，支持嵌入式和独立数据库的标准 SQL。它非常快，包含在一个只有大约 1.5 MB 的 JAR 中。

### 2.1。Maven 依赖关系

要在应用程序中使用`H2`数据库，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.194</version>
</dependency>
```

最新版本的 [`H2`数据库](https://web.archive.org/web/20220926184245/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22)可以从 Maven Central 下载。

### 2.2。配置

为了连接到一个`H2`内存中的数据库，我们可以使用一个连接`String`，协议为`mem,` ，后跟数据库名称。`driverClassName, URL, username`和`password`属性可以放在一个`.properties`文件中，由我们的应用程序读取:

```java
driverClassName=org.h2.Driver
url=jdbc:h2:mem:myDb;DB_CLOSE_DELAY=-1
username=sa
password=sa
```

这些属性确保在应用程序启动时自动创建`myDb`数据库。

默认情况下，当关闭与数据库的连接时，数据库也会关闭。如果我们希望数据库在 JVM 运行期间一直存在，我们可以指定属性`DB_CLOSE_DELAY=-1`

如果我们将数据库与 Hibernate 一起使用，我们还需要指定 Hibernate 方言:

```java
hibernate.dialect=org.hibernate.dialect.H2Dialect
```

`H2`数据库定期维护并提供关于[h2database.com](https://web.archive.org/web/20220926184245/http://www.h2database.com/html/main.html)的更详细的文件。

## 3。`HSQLDB` ( `HyperSQL`数据库)

`HSQLDB`是一个开源项目，也是用 Java 写的，代表一个关系数据库。它遵循 SQL 和 JDBC 标准，并支持 SQL 特性，如存储过程和触发器。

它可以在内存模式下使用，也可以配置为使用磁盘存储。

### 3.1。Maven 依赖关系

要使用`HSQLDB`开发应用程序，我们需要 Maven 依赖关系:

```java
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.3.4</version>
</dependency>
```

你可以在 Maven Central 上找到最新版本的`[HSQLDB](https://web.archive.org/web/20220926184245/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22hsqldb%22%20AND%20g%3A%22org.hsqldb%22)` 。

### 3.2。配置

我们需要的连接属性具有以下格式:

```java
driverClassName=org.hsqldb.jdbc.JDBCDriver
url=jdbc:hsqldb:mem:myDb
username=sa
password=sa
```

这确保了数据库将在启动时自动创建，在应用程序运行期间驻留在内存中，并在进程结束时被删除。

`HSQLDB`的`Hibernate`方言属性为:

```java
hibernate.dialect=org.hibernate.dialect.HSQLDialect
```

JAR 文件还包含一个带有 GUI 的数据库管理器。更多信息可以在 hsqldb.org 网站上找到。

## 4。Apache Derby 数据库

`Apache Derby`是另一个开源项目，包含由`Apache Software Foundation`创建的关系数据库管理系统。

`Derby`基于 SQL 和 JDBC 标准，主要用作嵌入式数据库，但也可以通过使用`Derby Network Server`框架在客户机-服务器模式下运行。

### 4.1。Maven 依赖关系

要在应用程序中使用`Derby`数据库，我们需要添加以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.derby</groupId>
    <artifactId>derby</artifactId>
    <version>10.13.1.1</version>
</dependency>
```

最新版本的 [`Derby`数据库](https://web.archive.org/web/20220926184245/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22derby%22%20AND%20g%3A%22org.apache.derby%22)可以在 Maven Central 上找到。

### 4.2。配置

连接字符串使用`memory`协议:

```java
driverClassName=org.apache.derby.jdbc.EmbeddedDriver
url=jdbc:derby:memory:myDb;create=true
username=sa
password=sa
```

对于启动时自动创建的数据库，我们必须在连接字符串中指定`create=true`。默认情况下，数据库在 JVM 退出时被关闭和删除。

如果使用带有`Hibernate`的数据库，我们需要定义方言:

```java
hibernate.dialect=org.hibernate.dialect.DerbyDialect
```

你可以在[db.apache.org/derby](https://web.archive.org/web/20220926184245/https://db.apache.org/derby/)了解更多关于`Derby`数据库的信息。

## 5。SQLite 数据库

`SQLite`是一个只在嵌入式模式下运行的 SQL 数据库，要么在内存中运行，要么保存为文件。它是用 C 语言编写的，但是也可以和 Java 一起使用。

### 5.1。Maven 依赖关系

要使用一个`SQLite`数据库，我们需要添加 JDBC 驱动程序 JAR:

```java
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.16.1</version>
</dependency>
```

可以从 Maven Central 下载 [sqlite-jdbc](https://web.archive.org/web/20220926184245/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22sqlite-jdbc%22%20AND%20g%3A%22org.xerial%22) 依赖项。

### 5.2。配置

连接属性将`org.sqlite.JDBC`驱动程序类和`memory`协议用于连接字符串:

```java
driverClassName=org.sqlite.JDBC
url=jdbc:sqlite:memory:myDb
username=sa
password=sa
```

如果`myDb`数据库不存在，这将自动创建该数据库。

目前，`Hibernate`还没有为`SQLite`提供方言，尽管将来很可能会提供。如果你想用`Hibernate`来使用`SQLite`，你必须创建你的`HibernateDialect`类。

要了解更多关于`SQLite`的信息，请访问[sqlite.org](https://web.archive.org/web/20220926184245/https://www.sqlite.org/index.html)。

## 6。Spring Boot 的内存数据库

Spring Boot 让使用内存数据库变得特别容易——因为它可以自动为`H2`、`HSQLDB,`和`Derby`创建配置。

要使用 Spring Boot 三种类型之一的数据库，我们需要做的就是将它的依赖项添加到`pom.xml`中。当框架遇到对类路径的依赖时，它会自动配置数据库。

## 7。结论

在本文中，我们快速浏览了 Java 生态系统中最常用的内存数据库，以及它们的基本配置。尽管它们对于测试很有用，但是请记住，在许多情况下，它们并不提供与最初的独立版本完全相同的功能。

你可以在 Github 上找到本文[中使用的代码示例。](https://web.archive.org/web/20220926184245/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-2)