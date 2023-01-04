# Apache Derby 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-derby>

## 1.概观

在本教程中，我们将了解 Apache Derby 数据库引擎，这是一个基于 Java 的关系数据库引擎，由 Apache 软件基金会作为开源项目开发。

我们将从安装和配置它开始，然后看看它提供的与它交互的工具。创建示例数据库后，我们将学习如何使用 Derby 的命令行工具执行 SQL 命令。最后，我们将了解如何使用普通 JDBC 并通过 Spring Boot 应用程序以编程方式连接到数据库。

## 2.部署模式

Apache Derby 有两个基本的部署选项:一个简单的嵌入式选项和一个客户机/服务器选项。

在嵌入式模式下，Derby 作为一个简单的单用户 Java 应用程序运行在同一个 Java 虚拟机(JVM)中。由于其自动启动和关闭，最终用户通常看不到它，也不需要管理员干预。在这种模式下，我们可以建立一个临时数据库，而不必管理它。

在客户机/服务器模式下，Derby 在 Java 虚拟机(JVM)中运行，当由一个提供跨网络多用户连接的应用程序启动时，该虚拟机托管服务器。

## 3.安装和配置

让我们看看如何安装 Apache Derby。

### 3.1.装置

首先，我们可以从[这里](https://web.archive.org/web/20220524032452/https://db.apache.org/derby/derby_downloads.html)下载最新版本的 Apache Derby。之后，我们提取下载的文件。提取的安装包含几个子目录:

```java
$ ls 
bin  demo  docs  index.html  javadoc  KEYS  lib  LICENSE  NOTICE  RELEASE-NOTES.html  test 
```

*   `bin`包含执行实用程序和设置环境的脚本
*   `demo`包含示例程序
*   `javadoc`包含 API 文档
*   包含 Apache Derby 文档
*   `lib`包含了阿帕奇德比。jar 文件
*   包含 Apache Derby 的回归测试

### 3.2.配置

在启动数据库引擎之前，我们需要配置一些东西。

首先，我们将把`DERBY_HOME`环境变量设置为我们提取 Apache Derby `bin.` 的位置。例如，如果 Apache Derby 安装在`/opt/derby-db`目录中，我们可以使用下面的命令:

```java
export DERBY_HOME=/opt/derby-db 
```

然后，我们将`DERBY_HOME/bin`目录添加到 PATH 环境变量中，这样我们就可以从任何目录运行 Derby 脚本:

```java
export PATH="$DERBY_HOME/bin:$PATH"
```

### 3.3.德比图书馆

在`lib`目录中有 Apache Derby 提供的各种库:

```java
$ ls
derbyclient.jar     derbynet.jar            derbyshared.jar
derby.jar           derbyoptionaltools.jar  derbytools.jar
derbyrun.jar        derby.war               derbyLocale_XX.jar ... 
```

每个库描述如下:

*   `derby.jar`:嵌入式环境需要。对于客户机/服务器环境，我们只需要服务器上的这个库。
*   所有配置都需要，无论我们运行的是嵌入式引擎、网络服务器、远程客户端还是数据库工具。
*   需要运行所有的 Apache Derby 工具(IJ、DBLook 和导入/导出)。如果我们运行的是网络服务器，或者我们的应用程序直接引用了 JDBC 驱动程序，这也是必需的。
*   `derbyrun.jar:` 用于启动 Apache Derby 工具
*   `derbynet.jar: `需要启动 Apache Derby 网络服务器
*   `derbyclient.jar:` 需要使用 Apache Derby 网络客户端驱动程序
*   需要本地化 Apache Derby 的消息

## 4.工具

Apache Derby 为不同的应用程序提供了许多工具。我们可以使用缩写名称运行 Apache Derby 工具和带有`derbyrun.jar`的实用程序，并且我们不需要设置 java CLASSPATH 环境变量。**`derbyrun.jar` 文件必须和其他 Derby JAR 文件在同一个文件夹中。**

### 4.1.颈内

IJ 是一个基于 JDBC 的 Java 命令行应用程序。它的主要目的是允许以交互方式或通过脚本执行 Derby SQL 语句。

首先，让我们运行 IJ 工具:

```java
$ $DERBY_HOME/bin/ij
ij version 10.13
ij> 
```

我们也可以使用下面的命令来执行它:

```java
$ java -jar $DERBY_HOME/lib/derbyrun.jar ij
ij version 10.13
ij> 
```

注意，所有命令都以分号结尾。如果我们执行一个不带分号的命令，命令行将移动到下一行。

在使用 SQL 语句一节中，我们将看到如何使用 IJ 执行几个命令。

此外，IJ 可以使用`properties`来执行命令。这可以帮助我们保存 一些 重复性 工作 由 利用属性 由IJ

我们可以通过以下方式设置 IJ 属性:

*   通过在命令行上使用`-p property-file`选项指定一个属性文件
*   通过使用命令行上的`-D`选项

我们可以创建一个属性文件，添加所有需要的属性，然后运行以下命令:

```java
$ java -jar derbyrun.jar ij -p file-name.properties
```

**如果当前目录下不存在`file-name.properties`文件，我们会得到一个 [`java.io.FileNotFoundException.`](/web/20220524032452/https://www.baeldung.com/java-filenotfound-exception)**

例如，假设我们想要创建一个到具有特定名称的特定数据库的连接。我们可以用`ij.database`属性来实现:

```java
$ java -jar -Dij.protocol=jdbc:derby: -Dij.database=baeldung derbyrun.jar ij
ij version 10.13
CONNECTION0* - 	jdbc:derby:baeldung
* = current connection 
```

### 4.2.DBLook

`dblook`工具提供了数据库的 DDL(数据定义语言)。例如，我们可以将`baeldung`数据库的 DDL 写入控制台:

```java
$ $DERBY_HOME/bin/dblook -d jdbc:derby:baeldung
-- Timestamp: 2021-08-23 01:29:48.529
-- Source database is: baeldung
-- Connection URL is: jdbc:derby:baeldung
-- appendLogs: false

-- ----------------------------------------------
-- DDL Statements for schemas
-- ----------------------------------------------

CREATE SCHEMA "basic_users";

-- ----------------------------------------------
-- DDL Statements for tables
-- ----------------------------------------------

CREATE TABLE "APP"."authors" ("id" INTEGER NOT NULL, "first_name" VARCHAR(255) , "last_name" VARCHAR(255));

-- ----------------------------------------------
-- DDL Statements for keys
-- ----------------------------------------------

-- PRIMARY/UNIQUE
ALTER TABLE "APP"."authors" ADD CONSTRAINT "SQL0000000000-582f8014-017b-6e26-ada1-00000644e000" PRIMARY KEY ("id"); 
```

`dblook`有其他选项，如下所述。

我们可以使用`-o`将 DDL 写到类似`baeldung.sql`的文件中:

```java
$ sudo $DERBY_HOME/bin/dblook -d jdbc:derby:baeldung -o baeldung.sql 
```

我们也可以用`-z`指定模式，用`-t:`指定表

```java
$ sudo $DERBY_HOME/bin/dblook -d jdbc:derby:baeldung -o baeldung.sql -z SCHEMA_NAME -t "TABLE_NAME"
```

### 4.3.Sysinfo

Apache Derby `sysinfo`工具显示关于我们的 Java 环境和 Derby 版本的信息。此外，`sysinfo`工具在控制台上显示系统信息:

```java
$ java -jar $DERBY_HOME/lib/derbyrun.jar sysinfo

------------------ Java Information ------------------
Java Version:    11.0.11
Java Vendor:     Ubuntu
Java home:       /usr/lib/jvm/java-11-openjdk-amd64
Java classpath:  /opt/derby-db/lib/derbyrun.jar
OS name:         Linux
OS architecture: amd64
OS version:      5.11.0-27-generic
Java user name:  arash
Java user home:  /home/arash
Java user dir:   /opt/derby-db
java.specification.name: Java Platform API Specification
java.specification.version: 11
java.runtime.version: 11.0.11+9-Ubuntu-0ubuntu2.20.04
--------- Derby Information --------
[/opt/derby-db/lib/derby.jar] 10.13.1.1 - (1873585)
[/opt/derby-db/lib/derbytools.jar] 10.13.1.1 - (1873585)
[/opt/derby-db/lib/derbynet.jar] 10.13.1.1 - (1873585)
[/opt/derby-db/lib/derbyclient.jar] 10.13.1.1 - (1873585)
[/opt/derby-db/lib/derbyshared.jar] 10.13.1.1 - (1873585)
[/opt/derby-db/lib/derbyoptionaltools.jar] 10.13.1.1 - (1873585)
```

## 5.Apache Derby 中的 SQL 语句

在这里 我们将 研究一下Apache Derby 提供的一些基本 SQL 语句。我们将通过一个例子来看看每个语句的语法。

### 5.1.创建数据库

我们可以用连接字符串中的`connect`命令和`create=true`属性创建一个新的数据库:

```java
ij> connect 'jdbc:derby:databaseName;create=true';
```

### 5.2.连接到数据库

当我们与 Derby 数据库交互时，IJ 会根据 URL 语法自动加载适当的驱动程序:

```java
ij> connect 'jdbc:derby:baeldung' user 'user1' password 'pass123';
```

### 5.3.创建模式

`CREATE SCHEMA`语句定义了一个模式，这是一种为一组对象标识特定名称空间的方法:

```java
CREATE SCHEMA schema_name AUTHORIZATION userName;
```

模式名不能超过 128 个字符，并且在数据库中必须是唯一的。此外，模式名不能以前缀 SYS 开头。

下面是一个名为`baeldung_authors`的模式示例:

```java
ij> CREATE SCHEMA baeldung_authors;
```

除了 ，我们可以为`baeldung_authors`创建一个模式，只有 ID 为`arash`的 特定用户可以访问:

```java
ij> CREATE SCHEMA baeldung_authors AUTHORIZATION arash;
```

### 5.4.删除架构

我们可以使用`DROP SCHEMA`语句通过 删除 一个 模式，同样，目标模式必须为空， `DROP SCHEMA`才能成功。我们不能删除`APP`模式(默认用户模式)或`SYS`模式:

```java
DROP SCHEMA schema_name RESTRICT;
```

**通过使用`RESTRICT`关键字我们 可以强制执行 的规则，即不能 成为中定义的任何对象**

让我们看一个删除模式的示例:

```java
ij> DROP SCHEMA baeldung_authors RESTRICT;
```

### 5.5.创建表格

我们可以用`CREATE TABLE`语句给 创建一个表，其中 包含 列和

```java
CREATE TABLE table_name (
   column_name1 column_data_type1 constraint (optional),
   column_name2 column_data_type2 constraint (optional),
);
```

让我们看一个例子:

```java
ij> CREATE TABLE posts(post_id INT NOT NULL, publish_date DATE NOT NULL,
    view_number INT DEFAULT 0, PRIMARY KEY (post_id));
```

**如果我们 做 不做 指定默认值 ，NULL则 插入 中的 列 为**

用于 CRUD 操作的其他 SQL 语句，如 INSERT、UPDATE、DELETE 和 SELECT，与标准 SQL 相似。

## 6.用 Apache Derby 进行 JDBC 编程

在这里，我们将学习如何创建一个使用 Apache Derby 作为数据库引擎的 Java 应用程序。

### 6.1.Maven 依赖性

有两个 德比 驱动 在[Maven](/web/20220524032452/https://www.baeldung.com/maven-guide):`derby`和`derbynet.` 前者是 用于嵌入式应用而 用于后者

让我们为 [`derby`](https://web.archive.org/web/20220524032452/https://mvnrepository.com/artifact/org.apache.derby/derby) 添加一个 Maven 依赖:

```java
<dependency>
    <groupId>org.apache.derby</groupId>
    <artifactId>derby</artifactId>
    <version>10.13.1.1</version>
</dependency> 
```

同样，我们为 [`derbyclient`](https://web.archive.org/web/20220524032452/https://mvnrepository.com/artifact/org.apache.derby/derbyclient) 添加 Maven 依赖:

```java
<dependency>
    <groupId>org.apache.derby</groupId>
    <artifactId>derbyclient</artifactId>
    <version>10.13.1.1</version>
</dependency> 
```

### 6.2.JDBC URL 连接

我们可以用 不同的 连接字符串参数 连接到数据库，用于 客户/服务器和嵌入式应用。

在下面的语法中，我们可以将其用于嵌入式模式:

```java
jdbc:derby:[subsubprotocol:][databaseName][;attribute=value]
```

此外，我们可以将下面的语法用于客户端/服务器模式:

```java
jdbc:derby://server[:port]/databaseName[;attribute=value]
```

在嵌入式模式下，数据库 URL 不包含主机名和端口号。例如:

```java
String urlConnection = "jdbc:derby:baeldung;create=true";
```

### 6.3.在嵌入式模式下使用 Apache Derby

让我们以嵌入式模式连接到一个 Apache Derby 数据库，如果它不存在，就在当前目录中创建它，创建一个表，并使用 SQL 语句将行插入到表中:

```java
String urlConnection = "jdbc:derby:baeldung;create=true";
Connection con = DriverManager.getConnection(urlConnection);
Statement statement = con.createStatement();
String sql = "CREATE TABLE authors (id INT PRIMARY KEY,first_name VARCHAR(255),last_name VARCHAR(255))";
statement.execute(sql);
sql = "INSERT INTO authors VALUES (1, 'arash','ariani')";
statement.execute(sql);
```

### 6.4.在客户机/服务器模式下使用 Apache Derby

首先，我们运行下面的命令以客户机/服务器模式启动 Apache Derby:

```java
$ java -jar $DERBY_HOME/lib/derbyrun.jar server start 
Sat Aug 28 20:47:58 IRDT 2021 : Security manager installed using the Basic server security policy.
Sat Aug 28 20:47:58 IRDT 2021 : Apache Derby Network Server - 10.13.1.1 -
(1873585) started and ready to accept connections on port 1527
```

我们使用 JDBC API 连接到本地主机上的 Apache Derby 服务器，并从数据库`baeldung`的`authors`表中选择所有条目:

```java
String urlConnection = "jdbc:derby://localhost:1527/baeldung";
   try (Connection con = DriverManager.getConnection(urlConnection)) {
       Statement statement = con.createStatement();
       String sql = "SELECT * FROM authors";
       ResultSet result = statement.executeQuery(sql);
         while (result.next()) {
           // We can print or use ResultSet here
         }
   } catch (SQLException ex) {
       ex.printStackTrace();
 }
```

## 7.与 Spring Boot 的阿帕奇德比

本节将不会详细解释如何创建基于 [Spring Boot](/web/20220524032452/https://www.baeldung.com/spring-boot) 的应用程序，而只解释与 Apache Derby 数据库交互相关的设置。

### 7.1.Apache Derby 的依赖性

我们只需将来自 Spring Boot 的 [Spring 数据](/web/20220524032452/https://www.baeldung.com/spring-data)和 Apache Derby 依赖项添加到项目中，以将 Apache Derby 整合到其中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.derby</groupId>
    <artifactId>derbyclient</artifactId>
    <version>10.13.1.1</version>
</dependency>
```

### 7.2.Spring Boot 构型

通过添加这些应用程序属性，我们可以将 Derby 用作我们的持久数据库:

```java
spring.datasource.url=jdbc:derby://localhost:1527/baeldung 
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.DerbyTenSevenDialect 
spring.jpa.hibernate.ddl-auto=update
spring.datasource.driver-class-name=org.apache.derby.jdbc.ClientDriver
```

## 8.结论

在本教程中，我们研究了 Apache Derby 的安装和配置。之后，我们对工具和一些最重要的 SQL 语句进行了概述。我们用 Java 代码片段讲述了 JDBC 编程，并最终了解了如何配置 Spring Boot 以使用 Apache Derby 作为持久数据库。

和往常一样，本文中使用的例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524032452/https://github.com/eugenp/tutorials/tree/master/persistence-modules/apache-derby)