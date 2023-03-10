# H2 的嵌入式数据库在哪里存储数据？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/h2-embedded-db-data-storage>

## 1.介绍

在本文中，我们将学习如何配置 Spring Boot 应用程序来使用嵌入式 [H2 数据库](https://web.archive.org/web/20220524040032/https://www.h2database.com/html/main.html)，然后看看 H2 的嵌入式数据库在哪里存储数据。

H2 数据库是一个轻量级的开源数据库，目前还没有商业支持。我们可以在各种模式下使用它:

*   服务器模式–用于通过 TCP/IP 使用 JDBC 或 ODBC 的远程连接
*   嵌入式模式–用于使用 JDBC 的本地连接
*   混合模式–这意味着我们可以将 H2 用于本地和远程连接

H2 可以配置为作为[内存数据库](/web/20220524040032/https://www.baeldung.com/java-in-memory-databases)运行，但它也可以是持久的，例如，它的数据将存储在磁盘上。出于本教程的目的，**我们将在嵌入式模式下使用 H2 数据库，并启用持久性，这样我们将在磁盘上拥有数据**。

## 2.嵌入式 H2 数据库

如果我们想使用 H2 数据库，我们需要将 [`h2`](https://web.archive.org/web/20220524040032/https://search.maven.org/search?q=g:com.h2database%20a:h2) 和 [`spring-boot-starter-data-jpa`](https://web.archive.org/web/20220524040032/https://search.maven.org/search?q=a:spring-boot-starter-data-jpa%20g:org.springframework.boot) Maven 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <versionId>1.4.200</versionId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <versionId>2.3.4.RELEASE</versionId>
</dependency>
```

## 3.H2 的嵌入式持久模式

我们已经提到，H2 可以使用文件系统来存储数据库数据。与内存中的方法相比，这种方法的最大优点是应用程序重启后数据库数据不会丢失。

我们能够通过我们的`application.properties`文件中的`spring.datasource.url`属性来配置存储模式。这样，我们可以通过在数据源 URL 中添加`mem`参数，后跟数据库名称，将 H2 数据库设置为使用内存方法:

`spring.datasource.url=jdbc:h2:mem:demodb`

如果我们使用基于文件的持久模式，我们将为磁盘位置设置一个可用选项，而不是参数`mem`。在下一节中，我们将讨论这些选项是什么。

让我们看看 H2 数据库创建了哪些文件:

*   `demodb.mv.db`–与其他文件不同，该文件总是被创建，它包含数据、事务日志和索引
*   `demodb.lock.db`–这是一个数据库锁文件，当数据库正在使用时，H2 会重新创建它
*   `demodb.trace.db`–该文件包含跟踪信息
*   `demodb.123.temp.db` –用于处理 blobs 或巨大的结果集
*   `demodb.newFile`–H2 使用此文件进行数据库压缩，它包含一个新的数据库存储文件
*   `demodb.oldFile`–H2 也使用此文件进行数据库压缩，它包含旧的数据库存储文件

## 4.H2 的嵌入式数据库存储位置

H2 在数据库文件的存储方面非常灵活。此时，我们可以将其存储目录配置为:

*   磁盘上的目录
*   当前用户目录
*   当前项目目录或工作目录

### 4.1.磁盘上的目录

我们可以设置存储数据库文件的特定目录位置:

```java
spring.datasource.url=jdbc:h2:file:C:/data/demodb
```

注意，在这个连接字符串中，**最后一个块指的是数据库名**。此外，即使我们错过了这个数据源连接 URL 中的文件关键字，H2 将管理它，并在提供的位置创建文件。

### 4.2.当前用户目录

如果我们希望将数据库文件存储在当前用户目录中，我们将使用在关键字`file`后包含波浪号`(~)`的数据源 URL:

```java
spring.datasource.url=jdbc:h2:file:~/demodb
```

比如在 Windows 系统中，这个目录会是 `C:/Users/<current user>`。

将数据库文件存储在当前用户目录的子目录中:

```java
spring.datasource.url=jdbc:h2:file:~/subdirectory/demodb
```

注意**如果子目录不存在，它将自动创建**。

### 4.3.当前工作目录

当前工作目录是应用程序启动的地方，用点(.)在数据源 URL 中。如果我们希望数据库文件在那里，我们将配置如下:

```java
spring.datasource.url=jdbc:h2:file:./demodb
```

将数据库文件存储在当前工作目录的子目录中:

```java
spring.datasource.url=jdbc:h2:file:./subdirectory/demodb
```

请注意，如果子目录不存在，将会自动创建。

## 5.结论

在这个简短的教程中，我们讨论了 H2 数据库的一些方面，并展示了 H2 的嵌入式数据库存储数据的位置。我们还学习了如何配置数据库文件的位置。

完整的代码样本可以在 [GitHub](https://web.archive.org/web/20220524040032/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-h2) 上找到。